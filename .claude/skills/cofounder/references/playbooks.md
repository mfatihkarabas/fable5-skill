# Playbooks — idea types and enhancement operators

## Part 1 — The taxonomy: ideas fail the way their type fails

Classify every idea into one (sometimes two) of these before evaluating. The
value of the type is that it tells you **where to point the skepticism first** —
the dominant failure mode and the first question. Asking a marketplace about its
product quality, or a B2B SaaS about its virality, is examining the organ that
wasn't going to fail.

### Marketplace / two-sided platform
- **Dies of**: liquidity. Not demand, not product — the cold-start chicken-egg.
  Both sides show up only if the other side is already there.
- **First question**: *Can you fake the supply side manually, in one city or one
  niche, starting Monday?* (Recruit 20 providers by hand; be the matchmaker over
  a spreadsheet.) If supply can't be hand-assembled at toy scale, it won't
  assemble at real scale.
- **Watch for**: "we'll do both sides at once" (pick the harder side and do it
  by hand); take-rate fantasies (real take rates: 10–20%, and disintermediation
  pressure rises with transaction size and repeat relationships).

### B2B SaaS
- **Dies of**: distribution, almost never product. The graveyard is full of
  better products that nobody heard about.
- **First question**: *Who exactly champions this inside the buying company, and
  what does winning look like for them personally?* No named champion role = no
  sales motion.
- **Watch for**: the dead-zone price (roughly $2k–$10k/yr: too expensive for a
  credit card, too cheap to fund the sales calls it requires — push price up or
  friction down, not neither); "we integrate with everything" at v1; pilots that
  never convert because the pilot was free.

### Consumer app
- **Dies of**: retention (first) and CAC (second). Downloads are vanity;
  day-30 return is the pulse.
- **First question**: *What is the day-30 habit loop — what trigger, in the
  user's existing week, brings them back without a push notification?*
- **Watch for**: "the demo is the product" (novelty ≠ habit); ad-funded plans
  before 10M users; social features on a product with no graph yet.

### AI wrapper / copilot
- **Dies of**: moat-lessness plus platform risk. The model improving helps your
  clone as much as it helps you; the platform shipping your feature helps only
  them.
- **First question**: *What do you own after 12 months of operation that a new
  entrant with the same model API doesn't get on day one?* Acceptable answers:
  proprietary data loop, workflow lock-in, distribution asset, domain eval set.
  "Prompt engineering" is not an answer.
- **Watch for**: the demo-to-product gap (90% accuracy demos brilliantly and
  fails in production 1 time in 10 — ask what the cost of the failure case is);
  per-seat pricing on usage-shaped costs.

### Hardware / physical product
- **Dies of**: capital cycles and margin math. Every iteration is months and
  money; software instincts ("ship and fix") kill here.
- **First question**: *What are landed cost, channel margin, and return rate —
  and does 2–2.5× markup to retail still leave a price customers pay?*
- **Watch for**: crowdfunding treated as validation (it validates novelty, not
  repeat demand); support/logistics as an afterthought.

### Productized service / agency-with-software
- **Dies of**: the margin ceiling — humans in the loop scale linearly, and the
  "software will replace them later" step keeps not happening.
- **First question**: *What fraction of delivery cost is human hours today, and
  what — specifically, this quarter — removes the next 20 points of it?*
- **Watch for**: software-multiple dreams on service-margin reality. (Honest
  services businesses are fine businesses; the failure is pricing/raising like
  SaaS while operating like an agency.)

### Local / geographic (food, logistics, services with radius)
- **Dies of**: density economics. Unit economics that work at 30% market share
  in one zip code and nowhere else.
- **First question**: *Model one neighborhood at realistic (single-digit %)
  penetration — is a route/territory profitable?*
- **Watch for**: growth plans that are really "repeat the hard city launch 40
  times"; the playbook that worked in the launch city being the founder, not
  the model.

### Dev tool / API / infrastructure
- **Dies of**: adoption-path confusion — loved by individual developers who
  can't pay, needed by enterprises who won't adopt bottom-up.
- **First question**: *Does the individual developer both adopt AND have budget
  authority — and if not, what carries it from laptop to purchase order?*
- **Watch for**: open-source as strategy without a monetization boundary drawn
  in advance (what's free forever vs paid — decided now, not "later"); usage
  pricing that punishes your best users' growth.

### Content / community / audience
- **Dies of**: monetization ceilings and platform dependence.
- **First question**: *Who owns the relationship — do you have addresses
  (email/SMS/app) or renting reach from an algorithm?*
- **Watch for**: audience-first-then-product plans where the audience's topic
  doesn't match a purchase intent (a million followers for humor converts
  worse than 5k for tax software).

**Hybrid ideas** inherit the failure modes of *both* parents (a marketplace for
AI services dies of liquidity AND moat-lessness). Evaluate against both lists;
the binding constraint is whichever is weaker.

---

## Part 2 — Enhancement operators

Mechanical moves for ENHANCE mode. Each raises specific scoring dimensions and
risks others — that's why the re-score after applying is mandatory. Generate
variants by applying operators; don't wait for inspiration.

### O1 — Narrow the wedge
Shrink the target segment until you can name 10 customers, then 10 more. The
vision stays; the entry point shrinks. "Project management for everyone" →
"shot-list management for wedding videographers."
- **Raises**: D1, D2 (specific pain is findable pain), D4 (narrow segments
  congregate — one forum, one conference), G1.
- **Risks**: D6 if the wedge is too small to sustain even a first business —
  check the bottom-up math still clears ~$1M ARR potential.
- **Use when**: G1 fails, or D1/D2 ≤ 1, or the anti-pattern "everyone is the
  customer" appears.

### O2 — Switch the payer
Keep the product; change who pays. Who suffers this problem *most expensively*?
Often it's not the user: consumer→employer, patient→clinic, individual→the
vendor who wants to reach them.
- **Raises**: D6 (B2B prices are 10–100× consumer), often D1.
- **Risks**: D4 (a new payer is a new sales motion); alignment (if payer ≠ user,
  the product must serve both or it becomes surveillance).
- **Use when**: D6 ≤ 1 on a consumer idea, or willingness-to-pay is the doubt.

### O3 — Move the price an order of magnitude
Usually up: 10× the price to 10× fewer, 10× more desperate customers. The
product then changes to earn it (service layer, guarantees, integration).
Occasionally down: collapse the price to zero and monetize the other side.
- **Raises**: D6 (escapes the dead zone), D1 (forces targeting the desperate).
- **Risks**: G4 (high-touch sales is a skill the founder may not have); D4
  motion changes entirely.
- **Use when**: price sits in the B2B dead zone, or unit economics fail at the
  reflexive "$29/mo" the founder anchored on.

### O4 — Move along the stack
Three positions for the same capability: sell the **tool** (customer does the
work), sell the **service** (you do the work with the tool), sell the
**outcome** (you charge on results). Each position has different margins,
moats, and buyer conversations. Founders default to "tool" reflexively;
"service first, tool later" often wins because the service phase generates the
data and workflow knowledge the tool needs (and it's revenue on day one).
- **Raises**: D2 (services are easier first sales), D5 (doing the work builds
  the proprietary dataset), D6 direction varies.
- **Risks**: G5/D6 (service margins), the productized-service trap — set the
  automation milestone in advance.
- **Use when**: the tool is clone-able (D5 = 0) or buyers "love it" but won't
  buy (they want the outcome, not homework).

### O5 — Unbundle an incumbent
Take the one feature the incumbent does worst — most-complained-about
(searchable: their reviews, forums, G2/Capterra one-stars) — and be 10× on it
for one segment. Suites are weakest at their edges.
- **Raises**: D2 (complaints are pre-verified demand — link them), D3 (the
  incumbent's neglect is the why-now), D1.
- **Risks**: D5 (if it works, the incumbent may fix the feature — need the
  wedge-to-platform sequel, O7); single-integration platform risk.
- **Use when**: entering a market with a dominant, disliked incumbent.

### O6 — Change the revenue model
Same product, different money shape: one-time → subscription; subscription →
usage; software fee → transaction take; product → the data exhaust as the
business. Each reshapes D6 and D5 without touching the product.
- **Raises**: D6 (recurring/expansion revenue), sometimes D5 (usage models
  deepen data loops).
- **Risks**: D2 (customers may resist the new shape — subscriptions fatigue,
  usage pricing frightens budget owners); check against the type's norms.
- **Use when**: D6 shape is the binding constraint but the product itself
  scores well.

### O7 — Design the sequence
Split the vision into a beachhead that pays now and the empire it earns later.
The test of a good beachhead: revenue within ~90 days, teaches something the
next stage needs, and is defensible enough to not be a dead end. "Day-one
revenue" is the constraint that keeps the sequence honest.
- **Raises**: G4 (fits runway), D2 (near-term testability), often D7.
- **Risks**: local-maximum trap — a comfortable beachhead nobody ever leaves.
  Write down, at design time, the trigger for stage two.
- **Use when**: the idea is good but too big (G4 fails), or PIVOT verdicts
  where the vision is sound and the entry is wrong.

### O8 — Attach to a live current
Reposition the idea onto a named, dated market change: a regulation with a
compliance deadline, a platform shift creating displaced users, a cost curve
that just crossed a threshold, a budget line that just appeared (e.g., "AI
transformation" budgets). Same product, new framing, urgent buyer.
- **Raises**: D3 (this IS the why-now dimension), D4 (currents carry attention
  — there are conferences, newsletters, budgets with the change's name on them).
- **Risks**: D5 (currents attract crowds — moat matters more, not less);
  faddishness (if the current recedes, does the product still matter? Check
  the idea survives the current's end).
- **Use when**: D3 ≤ 1, or the idea is timeless-but-ignorable ("vitamins")
  and needs a deadline to become a painkiller.

### Operator hygiene

- Apply operators **one at a time** and re-score; stacking three operators
  produces a variant whose score movement can't be attributed.
- Two to three variants is the right output of an ENHANCE pass, each scored,
  with the original kept as baseline. More is noise; one is a sales pitch.
- The winner is the variant whose **binding constraint improved** — not the one
  with the best story. If no variant moves the binding constraint, the honest
  ENHANCE output is "the constraint is intrinsic; verdict stands" — that
  finding is a success of the method, not a failure of imagination.
