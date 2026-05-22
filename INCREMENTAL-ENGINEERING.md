# Incremental Engineering — The ComOS Methodology

> **Note on this public copy:** this document was authored inside the private ComOS
> ecosystem and is shared here in the spirit of disciplines being portable across teams.
> References to specific repositories (`comos-federation`, `comos-services`, `comai-portal`,
> `comai-plan`) point at internal projects where this methodology was validated — they are
> not files you can open from this repo, but they are real places where this work happened.
> The cross-references stay in the public version because they are *evidence* that this
> methodology has been run at scale on real production work, not just sketched.
>
> The accompanying `SKILL.md` in this same repo is the *portable* part — the operational
> implementation of this methodology at the agent layer. If you adopt nothing else, adopt
> that. This longer document explains *why* it works.

> The loop that, on 2026-05-21, compressed roughly three weeks of architectural work into
> an afternoon. Across three repositories (federation, portal/retail, services), produced a
> revised federation design with an implementation plan, an audit-driven correction to the
> portal naming convention, and a brand-new services platform with 47 passing tests against
> the federation contract. The methodology is not new. The substrate is.

## What this doc is

This is the *methodology* doc — how the work runs, not what work to do. It's intended to be
read by:

- Future-you starting a new piece of architectural work
- Any engineer or operator picking up the ComOS planning loop for the first time
- An agent (Claude, Codex, others) being asked to participate in the loop

If you're trying to *do* a specific piece of work, this doc points you at *how* to run it.
The specific work lives in the per-project planning seeds and decision logs.

## The lineage — why this is called incremental engineering

The methodology is mechanical engineering's incremental design discipline, applied to
software architecture, with the iteration substrate replaced. The phrase "incremental
engineering" is deliberate; the discipline predates agile, predates lean, predates every
software-methodology buzzword. It is what mechanical engineers do when they cannot solve
analytically for the optimal design in one shot — too many coupled variables, too much
real-world friction that diverges from the model. So they don't try to be brilliant once.
They ship something almost right, observe what it actually does, change one thing, observe
again. Convergence over optimality. Reproducibility over speed. The previous iteration is
sacred — never redesigned from scratch, only refined.

The Wright brothers' wind tunnel was not a search for the perfect wing. It was a
systematic single-variable sweep through hundreds of wing shapes, each one differing from
the last by one controlled change. They were not being smart. They were being
*unambiguous*. That's the lineage.

What changed in 2026 is not the discipline. The discipline is the same. What changed is
the *cost per iteration*. Mechanical engineering iterations were bounded by how long it
took to fabricate the part. Pure-thought iterations were bounded by how long it took a
human to write the draft. Both of those constraints relaxed when an agent could draft
against accumulated repo context at LLM speeds. The methodology finally has a substrate
that doesn't bottleneck on drafting time.

## The loop

```
   seed (ferments in your head for days, not paper)
         │
         ▼
   working doc  ◄──┐
         │         │  bounce until smooth
         ▼         │
    [review + push back, agent revises against pushback + repo context]
         │         │
         └─────────┘
         │
         ▼
   planning doc  ◄──┐
         │          │  bounce until round
         ▼          │
    [review + push back, plan tightens, decisions close]
         │          │
         └──────────┘
         │
         ▼
   implementation plan in the domain repo
         │
         ▼
   implementation (real code, tests, deploys)
         │
         ▼
   accumulated context flows back into the repos
         │
         ▼
   next seed enters the loop with a richer context base
```

Each arrow is a transition between *layers*. Each layer has its own bounce — the seed
bounces until it's smooth; the working doc bounces until it's round; the plan bounces
until it's implementation-ready. The bounces are not friction. **The bouncing is the
algorithm.** Every bounce sharpens one set of edges that would otherwise catch on
something downstream.

### Stage 1 — Fermentation (in your head, days)

The seed is allowed to be incomplete. It is *not* a spec. It is a hunch with a direction,
tested against everything you already know about the system, the market, the constraints.
The fermentation phase is what makes the seed arrive in the right *shape* when it finally
goes to paper. A seed dropped the moment it occurs to you is angular. A seed that has been
turning over in your head for three days has had the worst edges knocked off before any
agent ever sees it.

Without this phase, the seed needs more bouncing to converge. You're paying that cost
upfront, in your head, before any tooling is involved. That is correct and load-bearing.
Do not try to compress it.

### Stage 2 — Seed → working doc (minutes)

You write the seed as a short prompt — four sentences, a paragraph, sometimes less. You
pass it to an agent operating inside a repository with substantial accumulated context
(strategy docs, system learnings, competitive intel, audit methodologies, decision logs,
prior planning seeds). The agent does not read your mind. It fills the seed out *against
the repo's context*, which is what makes the output coherent rather than generic.

This is why the methodology fails on a codebase with no CLAUDE.md, no strategy docs, no
decision logs. The seed is the same shape, but the agent has nothing to fill it out
against. The 100x is partly the agent and partly *what the agent has to read*.

### Stage 3 — Working doc bouncing (smoothing)

The first working doc is angular. Assertions are made instead of verified. Decisions are
recommended instead of resolved. Load-bearing pieces are described in passing instead of
called out. Each bounce knocks off one set of edges:

- A claim that wasn't checked → caught, replaced with a citation or marked unverified
- A decision presented as obvious → questioned, written explicitly with rationale or alternatives
- A section that hand-waves over hard cases → expanded to address them or scope them out

You bounce until the doc is *smooth* — no remaining edge that catches on something
downstream. Most teams treat doc review as a nuisance step before the "real" work. In this
loop, **the bouncing is the proof-before-done discipline applied at the design layer**,
before code exists, where the cost of correction is a sentence rather than a week of
implementation.

### Stage 4 — Working doc → planning doc

The smoothed working doc becomes the seed for the *implementation planning* layer. A
domain engineer (or domain-specific agent) reads the working doc against their own repo's
accumulated context and produces a plan: ordered phases, file:line citations, decision
points to close before specific phases, gates between phases, a deliverables checklist.

The plan inherits the smoothness of the working doc. The engineer can be load-bearing on
the design without re-litigating it. They focus on *implementation* questions (signature
changes, file:line citations, test design) instead of *design* questions (was clause 4
really right? does the bypass belong in the session manager?). This is why the engineer's
plan comes back tight: most design questions are already settled.

### Stage 5 — Planning doc bouncing (rounding)

Same bouncing pattern, applied to the plan. Catches now look different — they're
implementation-specific:

- A phase that depends on a decision that hasn't closed → flagged, decision pulled forward
- A test claim that won't actually catch the bug it claims to catch → tightened or replaced
- A risk that's been mitigated but the table still shows "Medium" → reclassified
- A gate that the team could quietly skip → made non-negotiable in the gate language

You bounce until the plan is *round* — implementation-ready. Every decision has either
closed or has an explicit "decide before phase X" marker. Every gate has measurable
criteria. Every risk has either a mitigation or an accepted-with-reason note.

### Stage 6 — Implementation (or platform creation)

The round plan goes to the domain engineering loop. Implementation proceeds against the
plan as the contract. Where it deviates, the deviation is recorded in the decision log
with a reason. The plan is not a tablet — it's a contract that can be amended with
discipline.

If the work is creating a *new* platform (as happened with `comos-services` on 2026-05-21),
the planning loop runs again at the platform's own layer: a platform seed, bounced smooth,
produces a platform plan, bounced round, produces phase-by-phase implementation.

### Stage 7 — Accumulated context flows back

This is the part that makes the loop *learn over time*. Every implementation produces new
context that goes back into the repos:

- Decisions get written into decision logs
- Discoveries get written into audit reports
- Patterns get written into skill files
- Conventions get written into shared docs (like this one, or naming-conventions.md)

The next seed entering the loop reads against a *richer* context base than the previous
seed. That's why the loop accelerates rather than degrading. Each pass leaves the system
stronger than it found it.

## Why this works — the analogies that explain the mechanics

These are framings that help reason about *why* the methodology produces the results it
does. They are not the methodology itself. The methodology is the loop above. But these
framings are useful when something is going wrong and you need to diagnose where.

### Proof-before-done as an activation function

In a neural network, an activation function is a per-neuron gate that decides whether a
signal flowing through is strong enough to propagate, and shapes how it propagates. ReLU
outputs zero below a threshold and linear above it. The non-linearity is what lets the
network learn complex patterns instead of just linear combinations.

Proof-before-done has exactly this shape. At each step of the loop, it gates whether the
signal "this is done" propagates forward. Below the threshold (any of the seven questions
fails): the signal doesn't propagate. The work is not done. It gets reworked. At the
threshold (all seven questions pass): the signal propagates cleanly. Downstream work can
depend on it.

The gate is **per-step, not per-task**. Every design decision, every code change, every
commit gets the same activation function applied. Without this per-step gate, *something*
propagates at every step — either real signal or noise — and the network cannot tell them
apart.

### The bouncing as backpropagation

Forward pass = produce a draft. Loss function = the gap between what the work claims and
what the work actually does (the proof-before-done seven questions). Backpropagation =
the loss signal travels backward through the prior decisions, adjusting weights at each
layer so future forward passes are less wrong.

When proof-before-done catches a failure mode — for example, the federation's prior
storage decision being wrong on 2026-05-21 — the signal that travels backward isn't
"the doc was wrong." It's more granular than that. It updates:

- *That specific assertion* (storage = own DB) → wrong
- *The reasoning chain that produced it* (didn't check existing code) → wrong
- *The convention layer* (no naming-conventions doc existed) → missing
- *The discipline layer* (asserted instead of verifying) → didn't run the gate

Each layer gets a credit-assigned correction. The naming-conventions doc gets *added*.
The discipline layer gets *reinforced*. The reasoning chain gets *updated*. That's
backprop, applied to engineering decisions.

### Documents as accumulated learned weights

Every decision log, every skill file, every convention doc is a *weight* the system has
learned. When the next iteration loads the repo context, it loads all of those weights.
The agent doesn't have to re-derive that retail uses database-per-scope, or that the
federation contract has clause 4, or that tenant handles bypass the blacklist — those
are weights baked into the network. The agent reads them and operates against them.

The implication: protect the weights. A decision log that gets out of date, a skill file
that gets contradicted by code, a convention doc that gets ignored — those are weights
that have decayed. The network starts producing worse outputs. The audit methodology
(in CLAUDE.md across the engineering repos) exists specifically to keep the weights
accurate.

### Tests as the reproducibility substrate

Incremental engineering only works when the system is *reproducible*. In mechanical
engineering, the wing is the wing — change one rib spacing and the rest is exactly as it
was. In software, change one line and the system can behave differently in ways that
aren't obvious because the state space is huge and the failure modes are silent.

Tests are the wind tunnel. When 47 tests pass on iteration N and 47 tests pass on
iteration N+1, you can interpret a behavioral difference as caused by your change, not
by some hidden state shift. The tests hold everything else constant so the change you
made is the only variable. Without tests, software is unreproducible and incremental
engineering does not converge.

This is why the test infrastructure trap from the retail audit was so dangerous —
"the test suites were lying" meant the wind tunnel was broken, and every iteration's
result was uninterpretable. Restoring the test infrastructure was restoring the
substrate that makes incremental engineering possible at all.

### Why the cost-per-iteration matters more than the per-step speed

A normal agent without this discipline might land 80% of a task in an hour, then spend a
week chasing the last 20%. With the discipline, the 80% lands the same hour, but the last
20% lands in the same hour too — because each step is verified before the next one starts
on a wrong foundation. Each step ends without debt. The compounding is what makes it 100x,
not the per-step speed.

The math is exponential in a specific sense: each verified step becomes load-bearing for
every subsequent step. Without verification, the inverse: each unverified step *reduces*
the value of every previous step, because the previous step now has to be re-examined
under the new doubt. Unverified work is not neutral. It actively erodes the value of the
work that came before.

## The division of labor — who does what

The methodology is collaborative. The roles are real and worth naming so that the loop
runs well across different operators.

### The human's role

**The judgment, the adversarial pressure, the fermentation.** The human:

- Holds the model of the system over time, across sessions
- Lets seeds ferment before submitting them
- Provides the gradient signal at the boundary the agent cannot see past — the question
  the agent didn't know to ask, the convention the agent didn't know to check
- Decides when a doc is "smooth" or a plan is "round" — the convergence judgment is human
- Pushes back when the agent has drifted into assertion instead of verification

This is irreducible. The agent cannot run this role on itself reliably. Every time the
loop has produced its best work, the human has been actively present at the gates.

### The agent's role

**The generative production, the recall against context, the running of the gate.** The agent:

- Drafts the next iteration of the doc against the seed plus the repo context
- Reads the accumulated weights (decision logs, skill files, conventions) without
  needing to be re-taught
- Runs proof-before-done on itself for the questions it knows to ask
- Surfaces options, recommends with rationale, leaves decisions to the human when
  judgment is required

The agent provides the *processing* and the *recall*. The human provides the
*judgment* and the *adversarial pressure*. Either alone produces maybe 5x. Together they
produce 100x.

### What this means for handing the methodology to someone else

If you wanted to teach this loop to someone other than yourself, the hardest part to
transfer is **the judgment**. You can write down the loop. You can write down the
analogies. You can show them the artifacts. What you cannot easily transfer is the
ability to push back on an agent's draft because something *smells off* — that's
pattern-matched against years of experience.

So the methodology is reproducible to the extent that the recipient has enough domain
expertise to be the loss function. Without that expertise, the bouncing degrades into
yes-and instead of sharpening, and the loop produces docs that are well-formatted but
not load-bearing.

The practical implication: when handing this loop to a new operator, they need *real
authority over the domain*. Not just process authority. They have to be the person who
can say "that's not actually true" with reasons.

## When this methodology applies — and when it doesn't

**Best applied to:**

- Architectural design across multiple systems or repos
- Methodology development (this doc is itself produced by this methodology)
- Strategic planning where the decision space is large and the cost of wrong decisions
  is high
- Audits and quality sweeps where the goal is finding systemic patterns, not isolated
  bugs
- Anything where the bottleneck is *thinking the problem through*, not typing the answer

**Less useful for:**

- One-off scripts and utility code
- Bug fixes where the cause is already located
- Mechanical work that is well-specified upfront (CRUD endpoints, schema changes,
  routine refactors)
- Anything where the iteration cost is dominated by external blockers (waiting for an
  API to respond, waiting for a deploy to finish)

The discipline is not free. Each bounce has a cost. The methodology pays off when the
iterations are cheap *and* the cost of getting it wrong is high. That intersection is
where multi-repo architectural work lives. That's why it works for ComOS specifically.

## What proves the methodology works — the 2026-05-21 evidence

This methodology was validated, at scale, in a single afternoon (with prior-evening
fermentation) on 2026-05-21. The session work spanned the federation, portal, retail, and
services repos. The proof points:

1. **Federation seed → plan → revision loop** caught five architectural mistakes that
   would have cost weeks of mid-implementation rework:
   - The `resolveTenantId()` seam — Phase 2 was not a one-line change
   - The dormancy baseline trap — fixture had to come from before Phase 0, not after
   - The `api_call` misframing — no gateway-side allowlist existed; Option B was a net-new
     security control, not a re-key
   - The `getAnyClient()` per-pool footgun — would have shipped an N-platform federation
     whose tool list was really one platform's
   - The refresh-storm vector — `federation_refresh_tools` needed auth or rate-limiting

2. **Portal naming-convention audit** surfaced two pre-existing gaps and documented them
   in SD9 (`comos-services/docs/decisions/multi-platform-federation.md`):
   - The `service` substring missing from `BLACKLISTED_HANDLES`
   - The pre-existing bug where `isValidTenantHandle` skips the blacklist entirely

3. **comos-services platform** went from zero to a stub MCP server with 47 passing
   tests, the full federation contract invariants locked, RFC 9728 metadata, deployment
   tooling, and Claude/Codex orientation files. The cost-per-iteration on the platform
   build was low enough that a wrong storage decision could be made *and corrected*
   within the same session, with the correction process itself producing
   `naming-conventions.md` as a permanent artifact.

4. **Confidence floor moved from ~80% to ~90%+** across the three production repos,
   because each black box cracked open during the session became a permanent
   documentation artifact. The next agent (or human) reading these repos has a sharper
   picture of *why* the system is structured the way it is — not just *what* it does.

The 100x framing is the product of two compounding effects: confidence compounding
(verified work raises the floor permanently) and cost-of-error compounding (caught-early
errors cost orders of magnitude less than caught-late errors). The combination is what
produces the order-of-magnitude result.

## How to start a new piece of work using this methodology

If you're picking up a new piece of architectural work and want to run this loop:

1. **Let the seed ferment.** Days, not minutes. Test it against what you already know.
   Resist the urge to write it down immediately.
2. **Write the seed short.** Four sentences to a paragraph. Direction over completeness.
3. **Submit to comai-plan** (or the appropriate planning repo) with no further
   instruction. Let the agent fill it out against the repo context.
4. **Bounce until smooth.** Push back on every assertion that should have been a
   verification. Cite specific weaknesses. Don't accept "smooth enough" until no
   downstream edge can catch.
5. **Hand the smooth doc to the implementation layer.** Federation engineer, services
   engineer, whoever owns the domain. Ask for a plan back.
6. **Bounce the plan until round.** Same discipline. Decisions close. Gates harden.
   Risks get classified honestly.
7. **Implementation proceeds against the round plan.** Deviations get recorded in the
   decision log. The plan is the contract.
8. **Accumulated context flows back.** New decision log entries, new audit reports, new
   skill files, new convention docs. The system is stronger than when you started.

If you find yourself skipping any of these steps under deadline pressure, the methodology
will degrade in exactly the way it's designed to catch. Run the loop. Trust the bouncing.

## What this doc is not

- Not a process replacement for thinking. The methodology only works because the human
  is genuinely engaged in the judgment loop. A human running this loop on autopilot
  produces autopilot results.
- Not a guarantee of correctness. The methodology raises the floor of what reliably ships;
  it does not eliminate the ceiling of failure. SD9-style gaps will continue to surface;
  what changes is whether they surface during planning (cheap) or production (expensive).
- Not an excuse to skip implementation. Bouncing a doc until it's round is *preparation*
  for the work, not the work itself. The plan is not the code.

## Sister docs and cross-references

- `comAI/CLAUDE.md` and `comai-portal/CLAUDE.md` — the codebase-audit methodology that
  produces the accumulated weights this loop reads against
- `.claude/skills/proof-before-done/SKILL.md` (in every ComOS engineering repo) — the
  seven-question gate that acts as the per-step activation function
- `comos-federation/docs/MULTI-PLATFORM-FEDERATION.md` + `MULTI-PLATFORM-FEDERATION-PLAN.md`
  — the canonical example of seed → working doc → plan, produced by this methodology
- `comos-services/docs/PLATFORM.md` + `docs/decisions/multi-platform-federation.md` —
  the canonical example of using this methodology to create an entire new platform
- `comos-services/docs/decisions/naming-conventions.md` — the artifact that resulted from
  a wrong assertion being caught and corrected during the loop; example of how the
  system gets stronger through use

---

*Captured 2026-05-21, the afternoon the methodology validated itself at scale. Author:
Ron, working with Claude. Subject to revision as the loop continues to run.*
