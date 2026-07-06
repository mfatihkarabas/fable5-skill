# Worked example — "user table with search, filters, pagination, inline edit"

The task as it arrives: *"Add a users page: searchable, filterable by role,
paginated, and admins can edit a user's name inline."*

The obvious implementation — `useState` for everything, `useEffect` to fetch,
map over rows — is what a generic model ships. It contains at least five real
bugs. Here is the procedure catching them before they exist.

---

## Boot ritual

### 1. State filing

| State | Kind | Home | Note |
|---|---|---|---|
| The user list | **Server data** | Query layer, keyed by `(search, role, page)` | NOT copied into `useState` — bug #1 lives there |
| Search text, role filter, page number | **URL state** | Query params | Refresh/share/back must survive — bug #2 lives here |
| Search input's in-flight text | **UI state** | Local, debounced into the URL param | The input is the draft; the URL is the truth the fetch reads |
| "Which row is being edited" | **UI state** | Local: `editingUserId` | An id, not a copy of the row — derive the row |
| The inline-edit draft name | **Form state** | Local, initialized from the row when editing starts | Initialized-from, never synced-with — bug #3 |
| "Current user is admin" | **Server/global** | Session context, already exists | Gates the edit affordance; checked, not assumed |

Two derivation checks that delete state a generic version stores:
- ~~`filteredUsers`~~ — the server filters; the query key IS the filter.
- ~~`editingUser` object~~ — derived: `users.find(u => u.id === editingUserId)`.
  Storing the object means edits to a stale copy (bug #3's cousin).

### 2. View states enumerated
- **empty, no filters**: "No users yet" + invite affordance (first-run screen —
  a new workspace admin lands here first).
- **empty, with filters**: "No users match" + one-click clear-filters. A
  different state than the above — conflating them confuses users. (This is
  the state everyone skips.)
- **loading, first visit**: table-shaped skeleton, header visible — layout
  doesn't jump when data lands.
- **loading, filter change**: keep showing current rows dimmed + indicator.
  Blanking the table on every keystroke makes search feel broken.
- **error**: message + retry button that actually refetches. Not a toast that
  evaporates, not a white screen.
- **many**: 25/page; page count visible; the 10,000-user org is the design
  case, not the demo's 8 rows.
- **slow**: skeleton has the table's dimensions — zero layout shift.

### 3. Interaction contract
Row shows Edit (admins only) → click swaps name cell to input, **focus moves
into it, text selected** → `Enter` saves, `Escape` cancels and **returns focus
to the Edit button** → while saving: input disabled, row marked pending.
Sort/filter/pagination are links/controls reachable by keyboard in visual order.

---

## The bugs the batteries catch

**Bug #1 — stale copy (state filing).** The obvious version does
`useEffect(() => { fetchUsers().then(setUsers) }, [])`, then inline-edit
mutates the local copy. Another admin's changes never appear; a refetch stomps
an in-progress edit. Caught by the filing table: server data lives in the
query layer; components read the cache, never own a copy.

**Bug #2 — vanish on refresh (REFRESH battery / URL filing).** Search, filter,
and page in `useState`: F5 loses all three; the back button doesn't return to
page 3; the link a support agent sends a colleague ("look at this filtered
view") silently shows something else. Caught by the REFRESH item: those three
are URL state, so they were filed there before the first line of code.

**Bug #3 — the refetch stomps the draft (FORMS rule).** Edit draft initialized
from the row but kept in sync with it: a background refetch (window refocus —
query libraries do this by default) rewrites the input mid-typing. Caught by
"initialized-from, never synced-with": the draft is the user's from the moment
editing starts.

**Bug #4 — search race (RACE battery).** Type "ann", backspace to "an":
the "ann" response can land after the "an" response and win. The obvious
debounce does not fix arrival order. Caught by RACE: responses keyed to their
query `(search, role, page)`; a response for a key nobody is looking at is
ignored. (The query library does this — the reason it was chosen at filing
time, not for convenience.)

**Bug #5 — the lying list (STALE-AFTER-WRITE battery).** Save succeeds, the
row still shows the old name because page-2-with-filter is a differently-keyed
cached query. Caught by naming the invalidations at battery time: on save,
invalidate the users list family; optimistic update of the visible row with
rollback on error (OPTIMISM item: the rollback path exists or the update
isn't optimistic — here: restore previous cache value + put the row back in
edit mode with the draft intact + error next to the input, not a toast).

---

## Done gate, executed

Rendered and operated: searched, filtered, paginated, edited a name —
watched the network tab confirm one request per settled keystroke (debounce
works), a PATCH on save, and an ignored stale response (typed fast, watched
the out-of-order return get dropped). Forced the error state (offline in
devtools) — retry button refetches. Empty-with-filters shows clear-filters.
Phone width: table collapses per the project's pattern (cards), Edit still
reachable. Keyboard: full flow mouseless — Edit → input focused → Enter saves
→ focus back on Edit. Console: clean. F5 on page 3 with a filter: still on
page 3 with the filter.

---

*The shape to copy: five production bugs, zero cleverness. Every one was
caught by a table or a battery item that runs the same way every time. That —
not taste — is what makes the output stable.*
