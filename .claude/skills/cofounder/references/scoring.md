# Scoring — gates and anchored dimensions

*The point of anchors: every score level is defined by an observable fact, not an
adjective. "Strong demand: 8/10" is a vibe and will come out different on every
run. "D2 = 2 because Competitor X charges $99/mo for a worse version (VERIFIED,
found pricing page)" reproduces. Score nothing without writing its evidence line
and class tag (`VERIFIED` / `REPORTED` / `ASSUMED`).*

Two hard rules, restated because they carry the system:

1. **Torn between two levels → take the lower.** Score inflation is the default
   failure mode of every evaluator; this rule is the correction.
2. **Never average past a 0.** A zero anywhere is the binding constraint. Report
   it as the finding; do not compute a total that dilutes it.

Totals are ordinal and session-local: they rank ideas scored in the same session
with the same rubric. A "14" is not a grade and means nothing across sessions.

---

## Gates — binary, checked first, in this order

A failed gate stops scoring. Verdict becomes `DEAD-UNLESS <the named condition>`.
Gates are ordered by how fast they kill: check cheap, fatal things first.

**G1 — Nameable customer.** Can the founder name 10 specific people or companies
who have this problem *this quarter* — names, or a list findable in an hour? Not
demographics ("busy professionals"), not personas. If they can't, nobody can be
interviewed, nobody can be sold to, and every other judgment is fiction.

**G2 — Existing pull.** Is anyone currently spending money or serious time on a
workaround for this problem? (A spreadsheet maintained weekly counts; a wish
expressed in a survey does not.) No workaround anywhere usually means the pain
is real but not acute — and the status quo, which charges nothing and requires
no change, wins against not-acute.

**G3 — Legal / platform survival.** Does the model survive one honest look at
law and platform policy? (Scrapes a platform that bans scraping; resells data it
can't license; regulated activity without the license; depends on an API whose
terms forbid the use.) "We'll deal with it later" fails the gate — later is when
it's expensive.

**G4 — Founder-feasible.** Can *this* founder, with their actual skills, money,
and time, get to a sellable v1 inside their runway? An idea that needs $2M and a
ML team is not a bad idea — it's a bad idea *for a solo founder with 6 months*.
Score the idea in the hands it's actually in.

**G5 — Unit sanity.** At a plausible price and honest cost per unit (including
the founder's time and any human-in-the-loop labor), is one unit
contribution-positive without invoking scale magic? "Margins improve at volume"
is allowed for manufacturing, not for services wearing a software costume.

---

## Dimensions — 0 to 3, every level anchored

### D1 — Problem intensity
- **0**: Nobody complains about this unprompted; the founder discovered the
  problem by imagining it.
- **1**: People agree it's annoying when asked, but no one has changed their
  behavior because of it.
- **2**: People complain unprompted (forum threads, reviews, support tickets —
  link them) and have tried something to fix it.
- **3**: People currently pay money or hours for a bad workaround — name the
  workaround and its price.

### D2 — Demand evidence
- **0**: Founder intuition only.
- **1**: Public complaint evidence found this session (links in the evidence line).
- **2**: Competitors or workarounds with real revenue exist for this problem
  (pricing page found, or paid-workaround named) — proof people pay for *the
  problem*, though not yet for *this solution*.
- **3**: Committed money for THIS product: pre-orders, LOIs, paid pilots, a
  waitlist that paid a deposit. (An unpaid waitlist is a 1, not a 3.)

### D3 — Why-now
- **0**: Nothing changed; the idea was equally buildable five years ago. (Then
  why isn't it built? Route back to "why not already.")
- **1**: A vague trend ("AI is getting better", "people care more about X").
- **2**: A specific enabling change within ~3 years, named and dated (cost
  threshold crossed, API opened, regulation passed, behavior shifted with
  evidence).
- **3**: A named change *plus* a closing window — a reason the opportunity is
  temporarily undefended (regulation with a compliance deadline, incumbent
  locked in a contract cycle, cost curve mid-fall so late entrants face
  incumbents with data advantages).

### D4 — Distribution
The dimension founders skip and the one that kills the most companies.
- **0**: "Marketing," "ads," "content," "going viral" — generic channel words
  with no mechanism.
- **1**: A specific plausible channel, untested (e.g., "vet-clinic management
  Facebook groups exist with 40k members" — named, but no foothold).
- **2**: The founder has privileged access to the channel today: an audience, a
  distribution partner in hand, a community they moderate, an existing customer
  base to cross-sell.
- **3**: The product distributes itself through use (each document sent recruits
  its recipient; each marketplace listing is SEO surface; team products spread
  seat-by-seat) — a mechanism, in the product, that you can point at.

### D5 — Moat trajectory
Moats aren't required at day one; a *path* to one is.
- **0**: A competent stranger could clone it in a weekend and nothing
  accumulates with usage. (Most "AI wrapper" ideas score 0 here — the model
  improving helps clones as much as it helps you.)
- **1**: Speed and execution are the only edge. Honest, common, and worth
  exactly eighteen months.
- **2**: Something accumulates with traction that a later entrant can't shortcut:
  proprietary data loops, switching costs, integrations, a supply side that took
  years to recruit. Name the accumulating asset.
- **3**: Structural: network effects with the density already forming, exclusive
  supply locked, regulatory license held, or a data asset that compounds.

### D6 — Economics shape
- **0**: Negative or unknown contribution margin per unit; or CAC is obviously
  unpayable at the price point (a $30/mo product that requires a sales call).
- **1**: Positive margin but a shape problem: long payback, heavy services
  content, capital-intensive growth, or price point in the dead zone (too
  expensive for self-serve, too cheap to fund a sales motion — roughly the
  $2k–$10k/yr B2B trap).
- **2**: Healthy margin and a coherent motion: price point matches the sales
  channel; payback under ~12 months; growth doesn't consume capital faster than
  it returns.
- **3**: Compounding economics: expansion revenue in the product's nature
  (usage-based, seat growth), near-zero marginal cost, or negative churn
  plausible. Show the mechanism, not the aspiration.

### D7 — Founder fit
The same idea is a different idea in different hands. Score the pairing.
- **0**: Tourist: no domain contact, no relevant skill, drawn by the market's
  size or heat.
- **1**: Genuine interest and general competence; would be learning the domain
  from zero in public.
- **2**: Earned insight: has worked in the domain, felt the problem personally,
  or possesses a specific observation about it that outsiders reliably get wrong.
- **3**: Unfair advantage: owns the audience, holds the relationships, has the
  license/dataset/technique, or has already built the hard part before. Name it.

---

## Output format — produce exactly this shape

```
IDEA: <one-sentence form from the boot ritual>
GATES: G1 ✓/✗ … G5 ✓/✗   (if any ✗ → verdict DEAD-UNLESS <condition>, stop)
D1 Problem:      <0-3> — <evidence line> [VERIFIED/REPORTED/ASSUMED]
D2 Demand:       <0-3> — …
D3 Why-now:      <0-3> — …
D4 Distribution: <0-3> — …
D5 Moat:         <0-3> — …
D6 Economics:    <0-3> — …
D7 Founder fit:  <0-3> — …
BINDING CONSTRAINT: <the lowest dimension and what it means>
CONFIDENCE: <which scores rest on ASSUMED/REPORTED evidence and could move>
VERDICT: PURSUE / TEST <named test, cost, flip-condition> / PIVOT <operator> / PASS <reopening condition>
```

---

## Worked example — follow the shape, not the content

Idea as brought: *"AI that writes meeting notes, but for veterinary clinics."*

```
IDEA: A small-practice vet clinic pays ~$150/mo for automatic visit
      documentation into their practice-management system, because vets spend
      2+ hrs/day on records, instead of typing notes after hours or hiring a
      scribe.
GATES: G1 ✓ (founder is married to a vet; can name 10 clinics through her
      network) G2 ✓ (clinics pay human scribes $15-20/hr and Talkatoo/ScribbleVet
      exist — workaround AND competitors verified) G3 ✓ G4 ✓ (founder ships
      SaaS for a living; 12-mo runway) G5 ✓ (ASR+LLM cost ≈ $0.4/visit vs $150/mo)

D1 Problem:      3 — vets on r/veterinary describe "pajama charting" nightly;
                 clinics pay scribes today [VERIFIED]
D2 Demand:       2 — competitors charge $99-249/mo and clinics pay; none of
                 THIS founder's prospects committed yet [VERIFIED]
D3 Why-now:      2 — medical-grade ASR accuracy/cost crossed usable threshold
                 ~2024; but the window partly closed: competitors exist [VERIFIED]
D4 Distribution: 1 — vet conferences + spouse's network is a start, not a
                 channel; competitors already at those conferences [ASSUMED]
D5 Moat:         1 — transcription is commodity; PMS integrations could become
                 a 2 but aren't built [ASSUMED]
D6 Economics:    2 — $150/mo self-serve-able; payback fast; churn unknown [REPORTED]
D7 Founder fit:  2 — domain access through spouse, ships SaaS; no unfair
                 distribution asset [VERIFIED]

BINDING CONSTRAINT: D4=1. This is now a distribution knife-fight against funded
incumbents, not a product bet. The product being good is table stakes.
CONFIDENCE: D5 and D4 could each move ±1 with a week of integration research
and 10 clinic calls.
VERDICT: TEST — get 5 clinics from the spouse's network to pre-commit $100/mo
for a manual-concierge version (founder documents visits with off-the-shelf
tools) within 3 weeks, <$300 cost. 5 yeses → PURSUE the wedge (and D2 becomes
a 3); fewer than 2 → the network advantage is imaginary, and against live
competitors that was the whole case: PASS, unless a PMS-integration moat
(operator O5: unbundle the incumbent's worst integration) changes the shape.
```

Note what the example does: cites evidence with tags, takes the lower score
when torn (D4 could be argued to 2; it isn't), names the binding constraint
instead of celebrating the 3s, and produces a test whose *failure* condition is
specified as precisely as its success.
