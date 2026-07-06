---
name: backend-dev
description: Backend engineering as Fable 5 practices it — designing and implementing APIs, services, data models, database schemas and migrations, background jobs, authentication/authorization, and integrations. Use when building or modifying server-side code, designing an endpoint or schema, writing a migration, debugging a server/database/concurrency issue, or reviewing backend changes.
---

# backend-dev — server-side work, the Fable 5 way

*From the handoff archive. This is not a style guide; it is the decision
procedure. The difference between senior and generic backend work is almost
never the code — it is what gets decided before the code, and which questions
get asked about every write path. Those questions are enumerable. Here they are.*

## Stance — the priors that drive everything below

1. **The data model outlives every function that touches it.** Handlers are
   rewritten weekly; schemas are forever (every bad column is load-bearing
   within a month). So design order is fixed and non-negotiable:
   **data → invariants → lifecycle → contract → handlers → optimization.**
   Starting at the handler — the generic model's reflex, because the handler is
   what the task names — is the root cause of most backend rework.
2. **All backend code is concurrent code.** Single-user thinking is the root
   backend sin. Every line runs while another copy of it runs, against the same
   rows. "That's unlikely" is not an answer; unlikely × production traffic =
   weekly.
3. **Everything that leaves the process will fail.** Network, disk, the other
   service, the database under load. The failure behavior is decided at write
   time, in the code, or it gets decided at 3am, in an incident.
4. **The database's guarantees beat your cleverness.** Uniqueness, referential
   integrity, atomic conditional updates, transactions — use the 50-year-old
   battle-tested mechanism before hand-rolling a check in application code.
   App-level checks are UX; constraints are truth.
5. **Boring wins.** The proven pattern over the interesting one. Novelty in a
   backend is a cost you pay to solve a problem the boring thing can't —
   name that problem or don't spend it.

## Boot ritual — before writing any backend code

Write these four answers (notes, comment block, anywhere durable):

1. **DATA TOUCHED:** which entities/tables, and — crucially — who *else* reads
   and writes them (other endpoints, jobs, services, reports). You are never
   the only client of your data.
2. **INVARIANTS:** what must be true after your change that was true before —
   uniqueness, sums that balance, states that only move forward, references
   that resolve. Each invariant then needs an *enforcer*: a constraint, a
   transaction boundary, or (weakest) a documented check.
3. **OPERATION CLASS:** read / idempotent write / non-idempotent write /
   long-running. Non-idempotent writes and long-running work get the full
   mutation battery below; reads get the performance questions.
4. **BLAST RADIUS:** who observes this change — API consumers, queue
   consumers, cron jobs, the mobile app two versions back. Is it backward
   compatible? If not, what is the migration sequence for *them*?

If the task is a modification, add the general rule: read the whole module and
grep the call sites before editing. Backend call sites include *non-code
callers*: cron entries, queue bindings, webhook registrations, other services'
HTTP calls. Grep configs and infra, not just source.

## BUILD — the design order, expanded

### 1. Data model first
- Schema written before any handler: types, nullability decided per column
  (nullable is a decision, not a default), and constraints.
- **Uniqueness and referential integrity live in the database.** An app-level
  "check if exists, then insert" is a race condition with extra steps; the
  unique constraint is the check. The app-level check may exist *in addition*,
  for a friendly error message.
- Every index justified by a named query ("the dashboard filters by
  `(org_id, created_at)`"). No query, no index; no index, no query on a big
  table.
- IDs, money, and time: opaque IDs exposed externally (never enumerable
  integers), money as integer minor units (never floats), timestamps in UTC
  with timezone-aware types, and `created_at`/`updated_at` on everything —
  you will want them the first time anything goes wrong.

### 2. Lifecycle as a state machine
Anything with a status column is a state machine whether you admit it or not.
Admit it: enumerate the states, draw the legal transitions, and route every
transition through one function that rejects illegal moves. The bug this
prevents — an entity reaching a state it can't be in — is otherwise found in
production data months later, unexplainable.

### 3. Contract before implementation
- Endpoint shapes written first, **including the error responses** — errors
  are part of the contract, not an implementation detail. Enumerate them:
  what does the caller see on bad input, missing object, permission denied,
  conflict?
- **Pagination on every list endpoint from day one.** "We'll add it later" is
  a lie: later it's a breaking change for every consumer. Same for filtering
  contracts on anything that will grow.
- Authorization stated per endpoint at contract time: not "logged in" but
  *who may do this to this object* — object-level, not route-level.

### 4. Handlers: parse, don't validate
Convert raw input into typed, validated values **once, at the boundary**.
Interior code receives values that are already legal and never re-checks. The
alternative — defensive re-validation sprinkled everywhere — means nobody
knows which layer actually guarantees what, and that ambiguity is where
injection and null bugs live. Corollary: interior functions take parsed types
(an ID type, an email type), not strings.

### 5. Observability before "done" (the 3am test)
The standard: **a person woken at 3am can diagnose a failure of this feature
from its telemetry alone, without adding log lines and redeploying.**
- Structured log at each boundary crossing (request in, external call out,
  job start/end) carrying a correlation ID that survives across services.
- Every error log answers: which entity, which state, which caller. "Error
  processing request" is a lie of omission — you had the context in scope and
  dropped it.
- "Is it working right now?" must be answerable from metrics/health, not ssh.

## THE MUTATION BATTERY — run on every write path, no exceptions

This battery is the single highest-value habit in this file. For each
mutation, answer all five **in writing**:

- **CONCURRENT** — this handler runs twice at once for the same entity: what
  happens? Read-modify-write cycles must be atomic at the database:
  conditional `UPDATE ... WHERE status = 'expected'` (check rows affected),
  a unique constraint absorbing the race, or an explicit lock. If the answer
  relies on "requests won't overlap," the answer is wrong.
- **RETRY** — the client's request timed out and it sends the same mutation
  again: double charge? duplicate row? The options: naturally idempotent,
  idempotency key stored and checked, or *documented as neither* with the
  duplicate accepted. Choose one, in writing.
- **PARTIAL** — the process dies between step 2 and step 3: what state is left,
  and who cleans it? Draw the transaction boundary: what's atomic together.
  Side effects that cross systems (email, payment, queue publish) **cannot be
  in the transaction** — so it's the outbox pattern, an at-least-once consumer
  with dedup, or an explicitly accepted inconsistency, chosen consciously.
- **AUTHZ** — who may perform this mutation *on this specific object*?
  Route-level auth ("is logged in") plus a missing object-level check is the
  most common real-world vulnerability class (IDOR). The check is: does this
  actor own/have-role-on this entity — enforced in the query itself where
  possible (`WHERE id = ? AND org_id = ?`).
- **AUDIT** — after it happens, can you tell that it happened, when, and by
  whom? For anything touching money, permissions, or user data: yes is the
  only acceptable answer.

## THE EXTERNAL-CALL BATTERY — every call that leaves the process

- **Timeout on every call.** No unset timeouts, ever — the default is
  infinity, and infinity is how one slow dependency consumes every worker.
  The value is justified against the caller's own budget.
- **Retries only on idempotent operations**, with exponential backoff and a
  cap. Retrying a non-idempotent call is the RETRY battery item inflicted on
  someone else.
- **The failure path is chosen, not defaulted:** degrade (serve stale, skip
  the enrichment, queue for later) or fail loudly — per call site. The
  unchosen default is a 500 that takes your feature down because an optional
  dependency hiccuped.
- **The slow case, not just the down case:** at p99 latency of the dependency,
  what backs up? A queue with no consumer cap, a request pool, memory?

## MIGRATIONS — the procedure

The prime rule: **old code and new code run simultaneously during every
deploy.** Every migration must be safe with both versions live.

1. **Expand → migrate → contract**, each step deployed separately: add the new
   column/table (nullable or defaulted); write to both / backfill in batches;
   switch reads; only then — a full deploy later — drop the old.
2. Never in one step: rename in place (that's add + backfill + drop), add
   `NOT NULL` without a default to a populated table, change a column's type
   under load, or take a long lock on a hot table (know your database's
   locking behavior for `ALTER TABLE`).
3. Backfills are batched with progress logging and are resumable — a
   million-row `UPDATE` in one statement is an outage.
4. The rollback path is tested on a scratch database, not reasoned about.
   "Down migrations" that were never run are fiction.

## DEBUG — backend addendum to general debugging discipline

- Reproduce with the narrowest **real** artifact: the actual failing request,
  actual payload, actual database state — not a synthetic guess at them. Half
  of backend "bugs" are data-shape surprises, and synthetic repro hides them.
- Interrogate reality before theory: logs and traces first, then the actual DB
  state (`SELECT` the rows), *then* code. Reading code first anchors you on
  intent; the evidence is in what actually happened.
- Suspected concurrency/timing bug: write the interleaved timeline of two
  actors explicitly (T1 reads, T2 reads, T1 writes, T2 writes-over). If you
  can't construct the interleaving on paper, you haven't found it; if you can,
  you have a failing test.
- Heisenbugs that vanish under observation are load/timing — go to metrics
  and percentiles, not print statements.

## SECURITY PASS — before done, mechanically

- Every interpolation point (SQL, shell, path, template, header, log line)
  parameterized or escaped — grep for string-building near these sinks.
- Authn is not authz: each endpoint has both the "who are you" and the "may
  *you* touch *this*" check (see AUTHZ battery item).
- Secrets from environment/manager only; grep the diff for anything
  key-shaped before committing.
- Errors returned to clients leak nothing internal (stack traces, SQL, paths,
  other users' existence). Not-found and permission-denied are often
  deliberately the same response.
- Deny by default: new endpoints/queues/buckets start private.

## PERFORMANCE — discipline, not vibes

- **Count queries per request** in your own review. The tripwire is any query
  inside a loop — the N+1. If the ORM is doing the looping for you, make it
  show you: print/log the SQL it emits. "The ORM handles it" without knowing
  the emitted SQL is an ASSUMED-class claim about your own code.
- Optimize in this order: didn't-need-the-work (cache, precompute, don't
  fetch) → the query (index, shape) → the algorithm → the code. Micro-tuning
  code that sits on an unindexed query is polishing the doorknob of a
  collapsed house.
- Measure before optimizing, in an environment whose data volume resembles
  production — everything is fast on 100 rows.

## Done gate — backend additions to the general gate

1. Mutation battery answered in writing for every write path touched.
2. **The endpoint/job was actually exercised** — run the service, send the
   real request (curl/test), including at least one *failure* case: bad input
   produces the contracted error shape and status, not a 500.
3. If a migration: applied AND rolled back against a scratch database.
4. Logs checked by eye: your new path emits evidence a 3am responder can act on.
5. The security pass ran.

## Tripwires — thoughts that trigger mandatory actions

| The thought | The action |
|---|---|
| "This can't happen" | Then a constraint or assertion enforcing it costs nothing. Add it — production data always finds the can't-happen state. |
| "We'll paginate later" | Later is a breaking change. Now. |
| "The requests won't overlap" | They will. Run the CONCURRENT battery item honestly. |
| "The ORM handles that" | Name the SQL it emits. Can't? Print it and read it. |
| "It's internal, no auth needed" | Internal is one SSRF or leaked token away from external. Auth it. |
| "I'll just run this script against prod" | Stop. Scratch copy first, or wrapped in a transaction with the rollback path stated, with a second look at the WHERE clause. |
| "Quick fix: catch the exception and continue" | You just converted a loud failure into silent corruption. What state is half-written when you continue? |
| "We can store that in two places for convenience" | You now own a consistency protocol. Derive it or single-source it. |

## Worked example

`references/worked-example.md` walks one endpoint ("invite a user to a
workspace") through the boot ritual and the mutation battery, showing the
three real bugs the battery catches that the obvious implementation ships.
Follow its shape when the batteries feel abstract.
