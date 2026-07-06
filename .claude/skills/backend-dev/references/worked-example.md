# Worked example — "invite a user to a workspace"

The task as it arrives: *"Add an endpoint to invite someone to a workspace by
email. If they don't have an account yet, they get one when they accept."*

The obvious implementation — validate email, insert an `invites` row, send the
email, return 200 — is what a generic model ships. It contains at least three
production bugs. Here is the procedure catching them.

---

## Boot ritual

**DATA TOUCHED:** `invites` (new), `users`, `workspace_members`. Other
writers/readers: the accept-invite flow (to be built), the workspace members
page (reads membership), the billing job (counts seats!) — found by grepping
`workspace_members`. Billing is in the blast radius; seats-by-invite vs
seats-by-acceptance is a product decision, so it goes in the report, not into
a silent assumption.

**INVARIANTS:**
1. One *pending* invite per (workspace, email) — re-inviting should refresh,
   not duplicate.
2. A user is a member of a workspace at most once.
3. An accepted invite implies a membership row exists (cross-entity — needs a
   transaction, not a constraint).

**OPERATION CLASS:** non-idempotent write (creates invite, sends email) →
full mutation battery. Accept-invite is a second mutation → its own battery.

**BLAST RADIUS:** new endpoint (no API compat issue); email delivery (external
call battery); billing job semantics (flagged above).

**LIFECYCLE:** an invite is a state machine:
`pending → accepted | revoked | expired`. Legal transitions only from
`pending`. Expiry is a *read-time comparison* (`expires_at < now()`), not a
cron flipping rows — crons drift and race; comparisons don't.

**CONTRACT (errors enumerated):** 201 created; 200 refreshed (re-invite);
403 not an admin of *this* workspace; 409 already a member; 422 bad email.
Deliberate choice: inviting an email that already has a pending invite is a
success (refresh), not an error — callers retry, UIs double-fire.

---

## Schema — constraints carry the invariants

```sql
CREATE TABLE invites (
  id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),  -- opaque, not serial
  workspace_id  uuid NOT NULL REFERENCES workspaces(id),
  email         citext NOT NULL,          -- case-insensitive: Bob@ = bob@
  status        text NOT NULL DEFAULT 'pending'
                CHECK (status IN ('pending','accepted','revoked','expired')),
  token_hash    bytea NOT NULL,           -- hash only; raw token appears once, in the email
  invited_by    uuid NOT NULL REFERENCES users(id),
  expires_at    timestamptz NOT NULL,
  created_at    timestamptz NOT NULL DEFAULT now(),
  updated_at    timestamptz NOT NULL DEFAULT now()
);
-- Invariant 1, enforced by the database, not by "check then insert":
CREATE UNIQUE INDEX one_pending_invite
  ON invites (workspace_id, email) WHERE status = 'pending';
-- Invariant 2 lives on workspace_members as UNIQUE (workspace_id, user_id).
-- Index justified by a named query: "the workspace settings page lists
-- pending invites for a workspace":
CREATE INDEX invites_by_workspace ON invites (workspace_id) WHERE status = 'pending';
```

Note the decisions a generic implementation skips: `citext` (or lowercasing at
the parse boundary) so `Bob@x.com` and `bob@x.com` don't create two invites;
the **partial** unique index so *revoked* invites don't block re-inviting;
storing the token's **hash** so a database leak doesn't hand out live invite
links.

---

## Mutation battery — CREATE INVITE

- **CONCURRENT** — admin double-clicks; two identical requests race. The
  check-then-insert version creates two invites and sends two emails. Fix: no
  pre-check — `INSERT ... ON CONFLICT (workspace_id, email) WHERE status =
  'pending' DO UPDATE SET expires_at = ..., token_hash = ...` (upsert against
  the partial unique index). The race is absorbed by the constraint. ✓
- **RETRY** — client timeout + resend hits the same upsert → refreshes the
  same invite. Naturally idempotent. Second email may send; acceptable
  (documented) — both emails carry the same valid link. ✓
- **PARTIAL** — crash between the insert committing and the email sending:
  a pending invite exists that no one received. Email **cannot** join the DB
  transaction. Choice, in writing: enqueue the send via an outbox row in the
  same transaction (consumer retries); the poor-man's fallback is a
  "resend invite" button plus visible pending state — chosen only if no queue
  exists, and stated in the report. ✗→fixed
- **AUTHZ** — not "is logged in": *is an admin of this workspace*. Enforced in
  the query: `... FROM workspace_members WHERE workspace_id = ? AND user_id =
  ? AND role = 'admin'`. Also: does the response leak whether an email already
  has an account? It must not — 201/200 look identical either way. ✗→fixed
- **AUDIT** — `invited_by` on the row; log line with workspace, inviter,
  invite id (not the raw token, not the email in full if logs are shared). ✓

Three bugs the battery caught that the obvious version ships: the
double-click duplicate (CONCURRENT), the sent-nothing ghost invite (PARTIAL),
and the account-existence leak (AUTHZ).

## Mutation battery — ACCEPT INVITE (the second mutation, often forgotten)

- **CONCURRENT** — same invite accepted in two tabs, or accept races with
  revoke. Fix is the state machine enforced atomically:
  `UPDATE invites SET status='accepted' WHERE token_hash=? AND
  status='pending' AND expires_at > now()` — then check **rows affected**;
  0 rows = someone else transitioned it first; return the terminal state.
- **PARTIAL** — membership insert and invite update must be one transaction
  (invariant 3). User creation (if new) joins it too.
- **RETRY** — accepted twice: second `UPDATE` affects 0 rows → respond
  "already accepted", 200, not an error. Idempotent by the same mechanism
  that fixed the race — this is common: **the conditional update solves
  CONCURRENT and RETRY simultaneously**. That pattern (guarded transition +
  rows-affected check) is the single most reusable move in this file.

## External-call battery — the email send

Timeout set (email APIs hang); retries OK (send is idempotent-enough; dup
email accepted above); failure path chosen: invite exists + resend affordance,
so **degrade** — the endpoint does not 500 because the mail provider blinked;
slow case: sends are async (outbox/queue), so provider latency never holds a
request thread.

## Done gate

Exercised for real: created an invite via curl (201), re-sent (200, same row,
new expiry), double-clicked concurrently (one row — constraint held), accepted
with a stale token (contracted 409-shape, not a 500), checked the log line has
workspace/inviter/invite ids. Billing-seat question reported to the user as an
open product decision, not silently decided.

---

*The shape to copy: every one of those decisions was produced by a checklist
item, not by inspiration. The battery is short; run all of it, every write
path, and this quality becomes the floor instead of a good day.*
