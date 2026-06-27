# Example council run

> A **real, unedited** run of this skill — five Claude advisors, an anonymized
> peer-review round, and a chairman synthesis. Reproduced here so you can see what
> `/council` actually produces before you install it.
>
> Reminder of what this is: a **single-model** technique. All five advisors are
> Claude in different roles. Read the chairman's bias caveat — it's the whole
> point of the honest framing.
>
> _Run date: 2026-06-27_

---

## The decision

> An ~8-person engineering team maintains a 3-year-old monolithic web application
> that currently works and serves real paying users. They must choose between:
> **(A)** incrementally rewrite it into microservices over the coming year, or
> **(B)** keep it as a single deployable application and invest the same effort in
> modularizing it internally — clear module boundaries, but one deploy.
>
> Factors in play: deployment complexity, team-coordination overhead, independent
> scalability, hiring/onboarding, operational burden.

---

## Round 1 — the five advisors (independent, in parallel)

### 🔴 The Contrarian
Microservices is the wrong tool for an 8-person team, and the failure mode is
brutally predictable: you will spend a year building distributed-systems plumbing
— service discovery, inter-service auth, network retries, distributed tracing,
schema-versioned contracts, a CI/CD matrix — instead of shipping anything
customers pay for. The single most likely way this kills you: feature velocity
craters mid-migration while you live in the worst-of-both-worlds state — a
half-split system where one logical change now spans three repos, two deploy
pipelines, and a cross-service transaction you used to do in one SQL statement.
Conway's Law is decisive here: microservices presuppose enough teams to own
services independently. Eight people cannot staff 6+ services; every engineer
becomes on-call for systems they didn't write, and a "network" now sits between
functions that used to be a method call — turning compile-time errors into 2am
production incidents.

Who gets hurt: the paying users, when distributed failures (partial outages,
eventual-consistency bugs, cascading timeouts) replace a monolith that simply
worked. Then the team, drowning in operational toil. Then the business, which
paid a year of salaries for a rewrite that delivered negative customer value and
a higher AWS bill.

The honest data point: most famous microservice successes (Amazon, Netflix)
adopted them at hundreds of engineers, and several high-profile teams have
reversed course back to monoliths citing cost and complexity. Option B gets you
90% of the modularity benefit at 10% of the risk, fully reversible.

`CONFIDENCE: 85%`
`CHANGE-MY-MIND: A concrete, near-term scaling or compliance requirement that genuinely demands independent deploy/scale of one component (e.g., a regulatory data-isolation mandate, or one workload whose scaling profile is 100x the rest) — which a modular monolith cannot satisfy.`

### 🟢 The First-Principles Thinker
Strip it down: what is actually being solved? Not "monolith vs microservices."
The real problem is that ~8 people feel coordination pain shipping changes —
merge conflicts, fear of deploys, unclear ownership, blast radius. Both options
are answers to a question that hasn't been verified. The first thing to establish
is *where the pain actually is*, because the two options treat radically
different diseases.

Microservices solve an organizational scaling problem — many teams needing to
deploy independently. With 8 engineers you have one or two teams. You do not have
the team topology that microservices exist to serve, and you will import their
entire tax (network failures, distributed transactions, observability, deploy
orchestration) to solve a problem you don't have. That is paying enterprise costs
for a startup-sized org.

Modularization (B) is strictly the better default *if* the real pain is internal
entanglement, and it's reversible — clean module boundaries are exactly the
precondition that makes a *later* extraction cheap if a genuine scaling need
appears (one component with a real, divergent scale profile). So the rebuilt
answer is: do B's boundary work regardless, then extract individual services
*only* where a measured constraint forces it. Don't rewrite "into microservices"
as a program; let architecture follow demonstrated load.

But verify first: profile the actual bottleneck. If the app "works" and scales
fine, possibly neither option is the priority and the year of effort belongs on
product or reliability instead.

`CONFIDENCE: 82%`
`CHANGE-MY-MIND: Evidence of a specific component with a genuinely divergent scaling or fault-isolation profile (e.g. one CPU-bound path that must scale 50x independently) — that would justify extracting that service now, not the whole rewrite.`

### 🟡 The Expansionist
Modularizing inside one deploy isn't the safe option — it's the *leverage*
option, because it makes the ambitious move *cheaper later*, not just safer now.
Here's the upside nobody named: pick (B) as the foundation, but treat each clean
module boundary as a pre-cut seam. The moment a single module shows a real,
asymmetric reason to split — it scales on a wildly different curve, it needs a
different language/runtime, or it becomes a product surface you'd sell or spin out
— you extract *that one* and only that one. You get microservices' best 10% (the
boundaries that actually pay) without buying the other 90% (distributed
transactions, network failure modes, an 8-person team running a fleet).

The bigger prize: a clean modular monolith is the ideal substrate for AI-assisted
development. Sharp boundaries + one deploy means an agent can reason about,
regenerate, and test a whole module in-process — fast feedback, no orchestration
tax. A premature microservice mesh is exactly where AI coding stalls. You're not
choosing "less ambitious," you're choosing the architecture that compounds
fastest over the next three years.

Adjacent option nobody named: instrument the seams now. Add per-module timing,
ownership tags, and dependency metrics so the *data* tells you which boundary to
extract first — let production decide, not the whiteboard. You earn the option to
go distributed exactly where it's worth it, and you can sell that optionality to
senior hires ("we extract on evidence") far better than either dogma.

`CONFIDENCE: 78%`
`CHANGE-MY-MIND: A concrete, near-term scaling or compliance boundary (e.g. one workload needs isolated infra/region, or a regulated data domain must be physically separated) that forces a real service split this year regardless of module hygiene.`

### 🔵 The Outsider
They have a thing that works and makes money, and the proposal is to spend a year
taking it apart. That's the part that jumps out at me first.

What's confusing: nobody said what's actually wrong today. "Microservices" is
offered as the goal, not as a fix for a named pain. Is deployment slow? Are teams
stepping on each other? Is one part of the app drowning under load while the rest
sleeps? I'm hearing a solution shopping for a problem.

The thing being assumed without justification: that an 8-person team can run
microservices. Splitting one app into many means networks between them, more
things to deploy, monitor, and debug at 3am, and someone owning that plumbing.
Eight people is small. Who babysits all those moving parts while still shipping
features to the paying customers?

On the listed factors: "independent scalability" only matters if one piece
genuinely needs to scale apart from the rest — otherwise it's a benefit you're
paying for and never using. "Hiring/onboarding" cuts the other way from how it's
usually sold: more services is more to learn, not less.

What a normal person would ask: Why not B? Clean internal boundaries get you most
of the organizational clarity with one deploy and a fraction of the risk — and if
a module later truly needs to split off, well-drawn boundaries make that easy. B
looks like A's first step anyway, minus the year-long gamble on a working,
revenue-generating product.

So default to B unless someone names a concrete pain that one-deploy can't solve.

`CONFIDENCE: 78%`
`CHANGE-MY-MIND: A specific, measured constraint that a single deployable genuinely cannot meet — e.g. one component must scale or be released independently and modularization provably won't deliver it.`

### ⚙️ The Executor
The Monday-morning question: what is the cheapest reversible first step? Option
A's first step is irreversible-ish — once you stand up a service mesh, CI/CD for N
services, and distributed tracing, you've absorbed the operational tax
permanently, and an 8-person team will spend its year paying it instead of
shipping features. Option B's first step is nearly free and fully reversible: pick
ONE painful seam in the monolith and enforce a module boundary behind an
interface. If it helps, do the next one. If it doesn't, you've lost a week, not a
year.

So start with B, concretely. Monday: turn on a module-dependency linter (e.g.
import-boundary rules), draw the current dependency graph, and pick the worst
tangle. Define 3-4 internal modules with explicit public interfaces and no
cross-imports of internals. One deploy, one CI pipeline, one on-call rotation —
unchanged.

The real blocker for A isn't code, it's operational maturity: do you have the
platform, observability, and on-call discipline to run 10 services? At 8 people,
almost certainly not, and microservices punish weak ops mercilessly. The real
blocker for B is discipline — boundaries rot without enforcement — which a linter
in CI solves cheaply.

Crucially, B is a prerequisite for A anyway. Clean internal modules are exactly
what you'd later carve out as services IF a real forcing function appears (a team
that must deploy independently, a component with a genuinely divergent scaling
profile). Modularize now; extract a service only when a specific module's pain
proves it earns its operational cost.

Microservices-first here is brilliant-but-impractical: it buys independent
scalability you don't need yet at a coordination cost you can't afford yet.

`CONFIDENCE: 82%`
`CHANGE-MY-MIND: Evidence that a specific module already has a divergent scaling or deploy-cadence profile the single deploy demonstrably can't serve — that turns one extraction from premature into justified.`

---

## Round 2 — anonymized peer review

Each advisor saw the other four responses relabeled **A–E** (identities hidden)
and answered three questions. Legend revealed at the end.

**Reviewer A's verdict:**
1. **C is strongest** — moves past "name a pain" to the concrete failure mode;
   Conway's Law makes 6+ services unstaffable at 8 people, and "worst-of-both
   mid-migration" names the real risk the others wave at. Mechanism-level.
2. **E is weakest/overconfident** — the "modular monolith as ideal AI-agent
   substrate" is speculative trend-grafting, not load-bearing for an 8-person
   revenue decision, smuggled in at 78%.
3. **Missing:** everyone assumes the team is asking in good faith about
   *architecture*. Nobody priced the human driver — résumé-driven development,
   boredom, a senior engineer's leverage play. The rewrite may be a
   retention/politics problem in a technical costume; no linter fixes that.

**Reviewer B's verdict:**
1. **D** — the only one that converts the consensus into a falsifiable Monday plan
   and nails that B is a strict prerequisite for A, making "modularize first" a
   free option rather than a stance.
2. **E** — lowest confidence yet biggest reach; the AI-substrate "prize" smuggles
   a trend into an architecture call. (C at 85% is the most confident but earns
   it.)
3. **Missing:** everyone treats this as pure engineering. Nobody asks WHO is
   pushing the rewrite or why; refusing it may cost your best engineers. And
   nobody priced the year's opportunity cost in customer/product terms.

**Reviewer C's verdict:**
1. **A is strongest** — it alone diagnoses that the real question is unverified
   coordination pain and reframes microservices as solving an org-scaling problem
   8 people don't have; carries the decision logic the others assume.
2. **E is weakest** — the AI-substrate claim is speculative and unscoped, an
   unproven thesis at 78%.
3. **Missing:** everyone treats this as architecture; nobody costs the people. No
   reversibility-of-talent — a year of plumbing can burn out or lose senior
   engineers, and the opportunity cost is the roadmap you didn't ship.

**Reviewer D's verdict:**
1. **C is strongest** — names concrete, dated failure modes and invokes Conway's
   Law correctly; specific and falsifiable, not just "default to B."
2. **E is most overconfident** — the "ideal substrate for AI-assisted
   development" claim is speculative and unmeasured at 78%.
3. **Missing:** everyone assumes B *succeeds*. Nobody costs the discipline B
   demands (modular monoliths rot without enforcement), or asks whether the
   rewrite is really a résumé/boredom signal no architecture fixes.

**Reviewer E's verdict:**
1. **A is strongest** — reframes the actual question: microservices solve an
   org-scaling problem 8 people don't have, so B's boundary work is right
   regardless. Diagnosis before prescription.
2. **C, at 85%, is most overconfident** — asserts "6+ services" and 3am failures
   as inevitable, catastrophizing rather than weighing.
3. **Missing:** everyone defaults to B and never steelmans the team's real motive
   — résumé appetite, hiring optics, morale. Nobody quantified the current pain or
   asked whether the year belongs on product, not architecture at all.

> **Legend:** A = First-Principles · B = Outsider · C = Contrarian ·
> D = Executor · E = Expansionist

---

## Round 3 — chairman synthesis

**The decision:** Spend the coming year rewriting a working, revenue-generating
monolith into microservices (A), or modularize it in place behind clean
boundaries on a single deploy (B)?

### Where they converged
All five advisors land on **B**: modularize in place now, and extract an
individual service *only* when a specific, measured constraint forces it. All five
also agree any future split should follow demonstrated load, not a whiteboard
plan. Confidence clustered tightly at 78–85%.

> ⚠️ **Shared-bias caveat (read this).** This is a single-model council — five
> Claude personas, not five independent minds. Near-unanimity here is **not**
> corroboration. Worse, "monolith-first / modular monolith" is also the current
> *fashionable* industry consensus (the well-publicized monolith reversals,
> "majestic monolith" advocacy), which a model's training data is saturated with.
> So this agreement may reflect a **shared prior absorbed from the same sources**,
> not five angles independently arriving at truth. Read it as "the conventional
> answer, stated confidently" — and pressure it against *your* specifics before
> trusting it.

### Live disagreements (preserved, not averaged)
- **Is modularization even the priority?** First-Principles and the Outsider say
  the current pain is *unverified* — maybe the year belongs on product or
  reliability, not architecture at all. The Executor counters that B's first step
  is so cheap and reversible it's worth doing regardless. (Unresolved — depends on
  data nobody has yet.)
- **Why B wins.** Two incompatible framings: B as the *low-risk default* (Outsider,
  Contrarian) vs. B as the *high-leverage / ambitious* play (Expansionist). Same
  recommendation, different theory — and the theory changes how aggressively you'd
  invest in the seams.
- **Is the Contrarian catastrophizing?** The Expansionist explicitly flagged the
  Contrarian's 85% certainty about "6+ services / 3am failures" as a strawman.
  The risk is real but its *inevitability* is contested.
- **Minority view, low weight:** the Expansionist's "modular monolith is the ideal
  substrate for AI-assisted development" was rejected as speculative by **four of
  five** reviewers. Noted, not relied upon.

### Blind spots & unknowns
- **The human/political driver (raised independently by all five reviewers).**
  Nobody knows *who* is pushing microservices or why — résumé-driven development,
  morale, retention, hiring optics. This is the council's single biggest collective
  miss, and it can't be assessed from the prompt. _Info gap: find out who is
  advocating and what outcome they actually want._
- **The pain is unquantified.** No data on deploy frequency, merge-conflict rate,
  incident load, or any component's scaling profile. _Info gap: measure before
  deciding._
- **Opportunity cost.** A year on architecture is a year of roadmap not shipped —
  unpriced by everyone.
- **B's discipline cost.** Module boundaries rot without enforcement; B silently
  fails if no one owns the CI guard.

### Recommendation
**Default to B** — modularize in place behind *enforced* boundaries, and extract a
single service only when a specific, measured constraint demands it.
`CONFIDENCE: 80%`
**Key assumption it rests on:** that *no* single component has a genuinely
divergent scaling, fault-isolation, or compliance profile requiring independent
deploy this year. If that assumption is false for even one component, extract
*that one* — and only that one — without buying the full rewrite.

### First action (cheapest reversible step)
Two near-free, fully reversible moves *this week*, before any architecture commit:
1. Spend an hour finding out **who wants microservices and what pain or goal they
   are actually naming.** (Addresses the council's biggest blind spot.)
2. Turn on an **import-boundary linter** and draw the current **dependency
   graph.** Commits you to neither A nor B; informs both.

### Tripwire (how you'll know you chose wrong)
- **B was insufficient if:** 2–3 months in, you repeatedly hit something one
  deploy genuinely cannot do — a component that must scale or release
  independently, or a regulatory data-isolation mandate. That's the signal for a
  *targeted extraction*, not a full rewrite.
- **B is failing on execution if:** the linter is ignored and boundaries decay —
  meaning the problem was ownership/discipline, and more architecture won't fix
  it.

---

_Generated by the [`council`](./SKILL.md) skill. Five Claude advisors, one model —
agreement is a prompt to verify, not a verdict to trust._
