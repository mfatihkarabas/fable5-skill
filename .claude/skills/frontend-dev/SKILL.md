---
name: frontend-dev
description: Frontend engineering as Fable 5 practices it — building and modifying UI components, pages, forms, client-side state management, data fetching, styling/CSS, and accessibility. Use when implementing or changing anything a user sees or interacts with in a browser or app: components, layouts, forms, loading/error states, client caching, responsive design, or debugging visual/interaction/state bugs.
---

# frontend-dev — client-side work, the Fable 5 way

*From the handoff archive. The generic model treats frontend as "make the
component look right." That framing produces demos. The senior framing:
frontend is a state management problem wearing a visual costume, running on
hardware and networks worse than yours, operated by hands and eyes unlike
yours. The procedures below encode that framing mechanically.*

## Stance — the priors

1. **State is the hard part; pixels are the easy part.** Most frontend bugs
   are a piece of state living in the wrong home or existing in two places.
   File the state correctly first and half the bugs never exist.
2. **The UI is a function of ALL the states, not the happy one.** A view isn't
   designed when the data renders; it's designed when empty, loading, error,
   partial, one-item, thousand-item, and slow all render on purpose.
3. **You cannot see JSX.** Reading component code confirms your intent, not
   the pixels or the behavior. Nothing frontend is done until it was rendered
   and operated — clicked, typed into, tabbed through, resized.
4. **Derive, don't sync.** Never store what you can compute from existing
   state. Two copies of one fact will disagree; the only question is when.
   (Every `useEffect` that copies state A into state B is this bug, pre-built.)
5. **The user's machine is worse than yours.** Slower CPU, worse network,
   smaller screen, maybe no mouse, maybe a screen reader. Every "works for me"
   carries that asterisk until the verification pass says otherwise.

## Boot ritual — before writing any UI code

Write these four answers:

1. **STATE FILING.** List every piece of state the task touches and file each
   into its home. Misfiling is the #1 frontend bug source; this table is the
   whole game:

   | Kind | Recognize it by | Home | Cardinal sin |
   |---|---|---|---|
   | **Server data** | Another system owns the truth; you hold a copy | Query/cache layer (React Query, SWR, loader) with an explicit staleness policy | Copying it into component/global state "for convenience" — now it silently goes stale |
   | **UI state** | Ephemeral; nobody cries if a refresh loses it (open dropdown, hover, active tab) | Local component state, lowest component that needs it | Hoisting it to a global store "in case" — now everything re-renders and nothing is encapsulated |
   | **Form state** | A draft with a validation lifecycle | Form state (local or form lib), *initialized from* server data, never live-synced with it | Two-way syncing draft ↔ server cache |
   | **URL state** | Losing it on refresh/share would annoy (route, filters, page, selected item, open modal with identity) | The URL — params and query | Burying it in memory so refresh/back/share silently break |
   | **Global app state** | Genuinely cross-cutting: session, theme, feature flags | One small global store | The dumping ground — global is the last resort, proven, not the default |

2. **DERIVATION CHECK.** Anything you're about to store that is computable
   from what's already stored (filtered lists, counts, `isValid`, selected
   object when you have id + collection) → compute it at render. Memoize only
   if a profile later says so.

3. **STATE ENUMERATION for each view:** empty / loading / error / partial /
   one / many-thousands / slow. One line each on what the user sees. The
   empty state is the most-skipped and the first one a new user meets;
   the error state is the difference between "retry" and a support ticket.

4. **INTERACTION CONTRACT:** what is clickable, what every action does, what
   the keyboard does, where focus goes after each transition (open, close,
   submit, delete). Written now, because retrofitted focus management is
   archaeology.

Modification tasks add the general rule: render the *current* behavior first
and operate it before changing it — the frontend equivalent of reproducing
before fixing.

## BUILD — components and pages

1. **Interface before body.** Write the component's props/data signature and
   the list of states it renders, before implementing. A component whose
   props can't be written crisply is two components.
2. **Semantic skeleton first.** Real elements: `button` for actions, `a` for
   navigation, `form` with submit for forms, `label` wired to every input,
   headings in order. A `div` with `onClick` is not a style choice, it's a
   defect — it's invisible to keyboards, screen readers, and right-click.
   Getting the skeleton right makes half the accessibility pass free.
3. **Happy path, then immediately the state battery** (below) — same sitting,
   not "polish later." Later never comes, and the error state ships as a
   white screen.
4. **Extraction discipline: copy twice, extract on the third.** A premature
   shared component couples unrelated screens; its props grow booleans
   (`isCompact`, `hideFooter`, `variant2`) until nobody can change it. Three
   real usages tell you the true shape; two are a coincidence.
5. **Styling:** spacing and color from the project's tokens/scale — a magic
   number needs a comment saying why it can't be a token. Layout states its
   intent (flex row that wraps; grid with fixed sidebar) rather than nudging
   with margins. Responsive and dark-mode considered while building — as
   constraints they're cheap, as retrofits they're a rewrite.

## THE ASYNC BATTERY — every fetch and mutation, no exceptions

- **RACE** — two requests in flight; the *first* one returns *last*. Does
  stale data overwrite fresh? (Classic: type "ab", results for "a" arrive
  after results for "ab".) Fix: cancellation, or key responses to their
  request and ignore stale ones. A query library does this — which is the
  main reason to use one, not "convenience."
- **DOUBLE-FIRE** — the button mashed twice, the form submitted twice:
  disabled/pending state during flight. (The backend twin of this is the
  idempotency question — assume the backend did NOT handle it.)
- **REFRESH** — F5 right now: what's lost that shouldn't be? This audits the
  URL-state filing from the boot ritual.
- **STALE-AFTER-WRITE** — after this mutation succeeds, which queries on
  screen are now wrong, and how do they find out? Name the invalidations.
  The bug this catches: the edit saves, the list still shows the old name.
- **OPTIMISM** — optimistic updates have a rollback path and an error surface,
  or they aren't optimistic. Half-optimistic (update UI, ignore failure) is
  lying to the user.
- **SLOW** — on a throttled connection, what exists at 3 seconds? Reserved
  space (no layout shift when data lands), skeleton or spinner *with layout*,
  and interactive elements that either work or visibly wait — never a frozen
  button that took the click and did nothing visible.

## FORMS — the procedure

Forms are where state filing, async battery, and accessibility collide, which
is why they're always worse than estimated.

- **Validation timing:** validate a field on blur the first time, on change
  only *after* it has an error (so fixing feels responsive), everything again
  on submit. Never on first keystroke — yelling "invalid email" at someone
  who has typed `j` is hostile.
- **Errors live next to their fields**, wired with `aria-describedby`;
  a submit-level summary appears for multi-error cases. **Server errors map
  back to fields** — a toast saying "validation failed" while the fields sit
  green is the most common form bug in production.
- **Submit:** disabled while pending (double-fire), input preserved on
  failure (destroying a long form on a 500 is data loss), and success has an
  explicit next state — navigate, reset, or confirm; never silence.
- Initialize from server data, then the draft is the user's (see filing
  table): don't let a background refetch stomp half-typed edits.

## ACCESSIBILITY — a mechanical pass, not a virtue

Not an enhancement layer: a feature the keyboard can't reach is broken, the
same way an endpoint that 500s is broken. The pass, in order, ~10 minutes:

1. **Unplug the mouse.** Tab through the entire flow: everything interactive
   reachable, focus visible at all times, order follows the visual order,
   `Escape` closes what it should, `Enter` submits where it should.
2. **Focus management on transitions:** modal opens → focus moves in (and is
   trapped); closes → focus returns to the opener; route changes → focus
   resets to the new content; deleted item → focus lands somewhere sane, not
   on `body`.
3. **Names:** every input labeled, every icon-only button `aria-label`ed,
   images alt-texted (or explicitly decorative).
4. **Not color alone:** any information carried by color has a second channel
   (icon, text, weight). Contrast sanity-check on new color pairs.
5. Anything more exotic (live regions, combobox semantics): reach for the
   established pattern/library rather than hand-rolling ARIA — wrong ARIA is
   worse than none.

## DEBUG — frontend addendum to general debugging discipline

- **Browser first, code second.** Reproduce with devtools open; the console
  and network tab are your logs — read them before reading source. The error,
  the failing request, or the 200-with-wrong-payload is usually sitting right
  there.
- **State bugs: inspect the actual state at the moment of wrongness** (React
  devtools, store devtools, or a temporary dump into the view). Never infer
  state from code — the whole class of stale-closure and misfiled-state bugs
  is invisible in source and obvious in the inspector.
- **CSS bugs: bisect live in the inspector** — toggle properties, delete
  nodes, edit values on the real page until the culprit rule is found; only
  then touch files. Editing stylesheets blind and reloading is guessing with
  a compile step. When stuck: outline everything (`* { outline: 1px solid
  red }`) to see the boxes you actually have, not the ones you think you have.
- **"Works on my machine" differences, in order of likelihood:** data shape
  (empty, huge, weird strings), viewport size, network timing (throttle it —
  races hide at localhost speed), extension/adblock interference, cache.

## PERFORMANCE — what actually matters, in order

1. **Ship less JavaScript.** Before adding a dependency, check what it drags
   in; the tripwire is importing a large library for one function you could
   write in five lines. Bundle weight is the performance budget everything
   else lives inside.
2. **Kill request waterfalls.** Fetch in parallel what doesn't depend;
   don't let component nesting serialize requests (parent fetches → renders
   child → child fetches). Hoist to the route/loader level.
3. **No layout shift:** reserve dimensions for everything async (images,
   ads, late data).
4. **Re-renders last, and only after profiling.** `memo`/`useMemo` sprinkled
   without a profile is noise that costs readability and often nothing else.
   With a profile, fix the actual cause — usually state filed too high,
   forcing a wide subtree to render.

## Done gate — frontend additions to the general gate

1. **Rendered and operated it**: dev server or screenshot; clicked the
   buttons, typed in the fields, submitted with bad input, watched the
   network tab do what you claimed.
2. **Resized it**: phone width, and the awkward middle (~800px). Text zoomed
   125% still usable.
3. **Keyboard pass done** (accessibility step 1 at minimum).
4. **State battery evidence**: saw the empty state, forced the error state
   (kill the network in devtools), watched the slow state (throttle).
5. **Console clean**: zero new errors or warnings — today's ignored warning
   is next month's unexplainable bug.

## Tripwires — thoughts that trigger mandatory actions

| The thought | The action |
|---|---|
| "I'll copy the server data into state so it's easier to work with" | You just scheduled a staleness bug. Use the cache layer; derive locally. |
| "I need a `useEffect` to keep these two states in sync" | You have duplicated state. Delete one; derive it. |
| "A div with onClick is fine here" | It's a `button`. It's always a `button`. |
| "I'll add loading/error states after it works" | Same sitting or they ship broken. The battery is the work, not the polish. |
| "Needs `memo`, it feels slow" | Profile first. It's usually a waterfall or misfiled state, not renders. |
| "I'll pull in {big library} for this" | Check the cost; it's often five lines of platform code. The platform got good. |
| "Works, shipping it" | At phone width? Keyboard? Empty list? Throttled network? That's the done gate, and it hasn't run. |
| "The design didn't specify the error state" | The design being silent doesn't make the state not exist. Propose one; don't ship a white screen. |

## Worked example

`references/worked-example.md` runs one feature (a filterable, paginated user
table with inline edit) through the boot ritual and batteries, showing the
five concrete bugs the procedure catches that the obvious implementation
ships. When the filing table feels abstract, copy its shape.
