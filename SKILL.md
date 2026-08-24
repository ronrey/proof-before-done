---
name: proof-before-done
description: >-
  Use BEFORE declaring any engineering task complete — feature, refactor, bugfix, audit sweep, or design doc. The gatekeeper for everything: it fires on every done-claim, no opt-out. The operational implementation of incremental engineering at the agent layer — an activation-function gate that blocks unverified "done" claims from propagating, surfaces credit-assigned correction signals when work fails the gate, enforces the bounded-iteration discipline that makes work compound rather than decay, and blocks self-consistent verification by requiring test fixtures be captured from reality rather than invented. Depth scales with risk (routine work: a short written pass; hard-mandatory surfaces: the full eight questions) but the gate is never skipped. Triggers on marking TodoWrite tasks complete, writing "✅ pass" / "ship it" / "Phase X complete", authoring "What landed" / "Verification" / "Status" sections, handing work back with "done" / "ready for review".
---

<!--
================================================================================
TEMPLATE — FIRST-RUN CUSTOMIZATION CHECKLIST

This is a generic skill template. Before adopting, fill in the placeholders below
and delete this entire HTML comment block. The eight questions are universal and
should not be edited; everything else should be tuned to your project.

⚠️ Do not convert the gate to opt-in. See "When to engage" below — the escalation
criteria an invitation-based gate depends on require the agent to correctly assess
its own work, which is exactly what fails in the case the gate exists to catch.
Scale the DEPTH to the risk; never the attendance.

Required substitutions:
  {{PROJECT_NAME}}        Your codebase or organization name (e.g., "Acme Platform")
  {{PROJECT_STANDARD}}    The bar word your team holds (e.g., "amazing", "production-ready",
                          "ship-ready"). The skill contrasts this word against the weaker
                          "working" — pick a word that genuinely means more than "compiled".
  {{ORIGIN_INCIDENT}}     One sentence naming the audit or incident that motivated this
                          skill in your codebase. If you have no specific incident yet,
                          write: "anticipated failure modes from agents reaching the
                          plausible endpoint and reporting done."
  {{METHODOLOGY_DOC}}     Path to your team's methodology doc, if any. If you don't have
                          one, replace the entire "Cross-references" section's first bullet
                          with: "Your team's engineering standards doc."

Optional tuning:
  - The closing test references "a staff engineer." If your team uses different role
    language (principal, senior, tech lead), substitute it.
  - The anti-patterns table examples (24h, 90d, 0.95, the duplicate-email failure) are
    universal but can be replaced with your codebase's actual past failure modes for
    higher resonance.

Once filled in: delete this comment block. The skill should never ship with template
placeholders in production.
================================================================================
-->

# proof-before-done

## When to engage

**The gate fires on every done-claim. There is no opt-out.** It is the gatekeeper for everything — no artifact is declared complete without passing through it. What scales is *depth*, never *whether the gate runs*.

Why not opt-in: the escalation criteria a discipline-by-invitation depends on ("am I uncertain this works," "would rollback be painful") require the agent to correctly assess its own work. **An agent that has mis-assessed its work will also mis-assess whether it needs the gate** — so the case the gate exists to catch is precisely the case where it would not be invoked. A gatekeeper the guarded party decides whether to consult is not a gatekeeper.

### Depth scales with risk

**Routine work** — refactors, internal helpers, UI tweaks, doc updates, test additions, dependency bumps within a minor — passes with a **short written statement**: what was claimed, how it was verified, and (if the artifact is a checker) where its fixtures came from. One or two honest sentences. This is not ceremony; it is the minimum record that a claim was checked rather than assumed.

**Escalate to the full eight questions** whenever any of these holds:
- You're uncertain the change actually works (not just that it compiles)
- You changed an interface, contract, or shape other code consumes
- The change is hard to reverse once shipped
- A bug here would be silent rather than loud (data corruption > crash)

State the escalation explicitly: "Running the full gate because &lt;reason&gt;."

**User-invoked** — "PbD this," "prove it before done," "run the gate" — always means the full eight questions in writing. No exceptions, no abbreviation.

### Hard-mandatory surfaces (full gate, never abbreviated)

On these surfaces the full eight-question gate is mandatory regardless of how small the change looks or how confident the agent is. A one-line diff to an auth check gets the same gate as a rewrite. Tune this list to your system; the shape is "places where a silent failure is expensive or irreversible":

- **Auth & identity** — OAuth flows, token/session handling, RBAC and permissions, signing keys
- **Migrations & schema** — database migrations, schema changes, backfills, irreversible data transformations
- **Deploy & infra config** — deploy scripts, container definitions, cloud service config, IAM bindings, secret bindings, env vars that ship to production
- **Cross-service contracts** — API request/response shapes consumed by other services, event payload shapes, tool/function signatures, anything in a shared types or schemas directory
- **Money paths** — checkout, payment, billing, subscription, invoicing — anything that moves funds
- **First production writes** — the first time new code writes to a production data store, the gate fires even if the surface isn't otherwise flagged

Your project's contributor docs may extend this list. They may not shrink it.

## Why this exists

Agents reach the bar the human holds, not the bar they state. The standard in {{PROJECT_NAME}} is *{{PROJECT_STANDARD}}*, not *working*. "Working" means the code compiled and the happy path ran once. The standard of *{{PROJECT_STANDARD}}* means the counts match, the claims are tested not asserted, the magic numbers are justified, the failure modes are proven, and the doc tells the truth about what it does and does not do.

The pattern this skill targets: {{ORIGIN_INCIDENT}} The agent reported the work done, and the verification floor only checked that the code compiled. This skill closes that gap.

## What this skill actually is — the methodology framing

The name "proof-before-done" reads like a checklist. That undersells it. What this skill actually is, when injected at the right trigger points in an agent's reasoning loop, is **the operational implementation of incremental engineering at the agent layer.**

Three roles the eight questions play together, each load-bearing for why this discipline produces results that ordinary checklists don't:

### 1. Activation function — the per-step gate that blocks unverified propagation

In a neural network, an activation function intercepts signal at every layer boundary and decides whether it's strong enough to propagate. The triggers in this skill's frontmatter (TodoWrite complete, "✅ pass", "Phase X complete", "What landed", "done") are layer boundaries — points where signal would propagate forward to the next stage of work. The eight questions are the evaluation that decides whether the "done" signal passes through.

- Signal passes (all eight answered affirmatively in writing) → downstream work can safely depend on this step
- Signal blocks (any question is "no" or silent) → the step is reworked before propagation; the next layer never builds on a wrong foundation

The gate is **per-step, not per-task.** Every design decision, every commit, every doc section, every claim of completion gets the same gate applied. Without this per-step gate, *something* propagates at every step — either real signal or noise — and the system cannot tell them apart.

The trade-off is deliberate, and it is about **depth, not attendance**. The cost of eight written answers on a typo fix exceeds the value; the cost of skipping the gate on a migration is unbounded. So the typo fix gets one honest sentence and the migration gets the full eight — but both pass through. Scaling depth is what keeps the gate load-bearing; skipping it entirely is what lets an unverified claim propagate unnoticed, because the agent least equipped to judge its own work is the one deciding whether to invoke.

### 2. Backpropagation — credit-assigned correction when work fails the gate

When the gate fires (a question fails), the signal that travels backward is more granular than "the work was wrong." Questions 5 and 7 in particular — *what this does NOT solve* and *honest framing* — force the agent to surface where the work falls short of the claim. That gap is the loss signal, credit-assigned across multiple layers:

- *The specific assertion* (e.g., "the index handles the race") → wrong or unverified
- *The reasoning chain that produced it* → did not check the actual race condition
- *The convention layer* → no schema for the identifier the race depends on
- *The discipline layer* → asserted instead of running the gate

Each layer gets a correction. The convention gets *added* if it was missing. The discipline gets *reinforced* where it didn't fire. The next iteration starts with a sharper model of why the previous one failed. That's backprop applied to engineering decisions, not just to neural-network weights.

### 3. Bounded iteration — the discipline that makes work compound

Mechanical engineering's incremental design discipline only converges when each iteration is *bounded* — a single, scoped change against the previous iteration's known state. Without bounds, iterations thrash; the agent (or the engineer) tries to do too much and the result is ambiguous about which change caused which effect.

Question 5 is the bounding mechanism: *what does my work NOT solve?* By forcing the agent to explicitly state what is out of scope, the skill prevents the failure mode where each iteration overclaims. If the "what this doesn't solve" list is empty, you're either iterating perfectly (rare) or you haven't actually thought about the bounds (common). Most agents fail this question by trying to claim too much; the gate makes that visible.

The result: each iteration is bounded, scoped, and recorded. Future iterations land against a clear seam. That's the Wright-brothers single-variable-change discipline, enforced by the agent's own gate rather than by a human standing over the work.

### Why the skill scales without the human in the room

The full methodology requires a human in the loop — the human provides judgment, adversarial pressure, and fermentation. The skill is the part of the methodology that **operates regardless of whether a human is present at every step.** It is what keeps the activation function in place during autonomous work, during multi-step background tasks, during overnight runs, during the parts of the loop the human can't supervise directly.

That's why this skill is load-bearing for the codebase-audit methodology and for any iterative-engineering loop. It's not a quality check appended to the work. It is the part of the discipline that survives without you.

## The eight-question gate

Before declaring any task complete, answer all eight in writing. Any "no," "didn't check," or silence blocks completion until resolved. The cost of running this gate during the work is one extra paragraph per design decision and one extra test per assertion. The cost of skipping it is rewriting docs, finding bugs in already-shipped code, and burning the user's trust in completion claims.

### 1. Does my count match my list?

If the doc says "11 agents changed" and the table has 18 rows, one of those numbers is wrong. Recount from ground truth (`grep`, `ls`, `find`) — never from memory, never from what the previous step said. **Show the command and the output in the doc.**

Counts are the cheapest possible verification. Getting them wrong telegraphs that nothing else was verified either.

### 2. Did I test what I asserted, or did I assert it?

Every claim of the form *"X works because Y"* needs either:
- A passing test that demonstrates Y, OR
- An explicit acknowledgment that Y is unverified

> "The unique index handles the race" → assertion.
> "Test `concurrent-write.spec.ts` proves two simultaneous inserts produce one row" → verification.

The assertion is fine if the doc says "unverified, deferred to Phase 3." Silent assertion is what reads as confidence and lands as a production bug.

### 3. Are my magic numbers justified at the call site?

`24h` cooldown, `90d` expiry, `0.95` confidence threshold, `5`-failure circuit breaker — every numeric constant needs either:
- A comment one line away explaining why this number (not 12h, not 365d), OR
- A link to the decision artifact (issue, doc, conversation date)

Numbers without rationale rot the moment someone tries to tune them. Future-you (or future-agent) cannot safely change the value without re-deriving the reasoning from scratch.

### 4. Are my string-encoded keys schemaed?

If the code composes identifiers from interpolation (`` `${accountId}:${threshold}d` ``), there is exactly **one** helper that encodes and **one** that decodes. Not two callers each composing the string inline. Not eighteen callers each composing the string inline.

The day someone changes the format in one place — `<threshold>d` to `<thresholdDays>` — every consumer breaks silently. That's a duplicate-email-to-customer bug, not a compile error. The tests pass. The deploy succeeds. The customer receives the second email.

### 5. What does my work NOT solve?

Write the "Limitations" / "Does NOT solve" section **before** writing the "What landed" section. If you cannot articulate what is still broken after your fix, you do not understand the boundary of what you fixed.

Name explicitly: hot retries, cross-process races, downstream dedup, race windows shorter than the TTL monitor cadence, retry storms, scope you punted to a later phase.

A "What landed" section without a "What this does NOT solve" section reads as complete and is almost never complete.

### 6. Does my verification floor match my claim?

`tsc --noEmit` and `eslint` prove the code compiled. They do not prove the cooldown engages, the index enforces uniqueness, or the API does not fire twice.

If the claim is "agents no longer duplicate emails," the verification floor needs a test that runs the agent twice and asserts one email. If the only verification is type-check, **downgrade the claim** to "code compiles and ships; runtime behavior unverified."

Match the claim to the floor. Don't ship a claim the floor can't support.

### 7. Is my framing honest about what I did?

Re-read the opening sentence of the doc. Does it claim more than the test floor proves?

> "Stops daily/weekly duplicate emails" overstates if hot retries can still duplicate.
> Cleaner: "Reduces duplicate-action rate from scheduled re-runs; retry and concurrent paths are out of scope."

Sell what you built, not what you wish you'd built. The audience for honest framing is six-months-from-now-you, who will inherit this code and need to know what's actually solved.

### 8. Did the verification's inputs come from reality, or from me?

The other seven questions can all be answered **honestly** by the author and still pass a broken artifact. They test whether the work was *executed* correctly. They do not test whether the author's *premises* were correct — and a verifier built on a wrong premise will confirm itself.

If you wrote the artifact **and** its test fixtures, you have proven self-consistency, not correctness. Name the origin of every fixture:

- **Captured** — taken from real output, a real file, a real request, a real message. Paste the capture command.
- **Invented** — hand-authored from your mental model of what the input looks like.

**An invented fixture for anything whose job is checking other work — a hook, a linter, a test harness, a validator, a parser, an audit script — is an automatic block.** That artifact's entire value is agreeing with reality; a fixture you imagined agrees only with you. Capture at least one real input before claiming the checker works.

When the artifact is self-authored *and* self-verified, escalate: hand the artifact and its claim to an **independent reviewer that did not write it** (a subagent, a second session, a colleague), briefed to *break* it rather than confirm it — "here is a checker and its tests; find the input it gets wrong." Confirmation-shaped review reproduces the author's blind spot.

> **Origin incident.** A stop-hook was written to catch fabricated character counts in drafted messages. Its test fixtures were hand-typed in the format the author *assumed* was used. Suite passed 5/5. On the first real message the hook miscounted by 4 characters and false-blocked a correct draft — the real rendering separated quoted paragraphs with an *unquoted* blank line, which the parser never saw. The bug and the fixtures shared one wrong premise, so the suite could not fail. One captured real message would have caught it instantly.

## The closing test

Ask aloud, before declaring done:

> **"If a staff engineer read this doc and ran one verification command of their choice, would my claim survive?"**

- If the answer is *"depends on which command they pick"* → not done.
- If the answer is *"yes, any command"* → done.
- If the answer is *"I'd want to pick the command they run"* → not done, and you know what to fix.

## Anti-patterns this skill prevents

These are the specific failure modes that motivated this skill. Each one is an agent-natural failure mode — they recur because they are the path of least resistance for a model trying to reach a plausible-looking endpoint.

| Anti-pattern | What it looks like | Why it fails |
|---|---|---|
| **Count drift** | "11 agents changed" with an 18-row table | Counts are the cheapest verification; getting them wrong signals nothing else was verified either |
| **Asserted correctness** | "The index handles the race" with no test | The race is the most likely production failure; asserting it doesn't fail isn't the same as proving it |
| **Naked magic numbers** | `24h`, `90d`, `0.95` with no rationale | Future-you cannot safely tune values without the original reasoning |
| **Ad-hoc string composition** | `` `${accountId}:${threshold}d` `` repeated in 18 places | The day the format changes in one place, every consumer silently breaks |
| **Verification floor mismatch** | "Ships duplicate prevention" with only `tsc` + `eslint` | The claim is about runtime behavior; the floor only proves syntax |
| **Hidden scope** | "What landed" without "What this does NOT solve" | Reader assumes the problem is solved; production says otherwise |
| **Self-reported success** | "Agent says done" without independent verification | The agent reaches a plausible-looking endpoint and reports done; the actual work may not have landed |
| **Self-consistent fixtures** | A parser and its test inputs both hand-written by the same author, suite green | The fixtures encode the same wrong premise as the code, so the suite cannot fail; it tests the implementation's assumptions, not reality |

## How to apply

Hold the eight questions in mind throughout the work — not just at the end. They are cheap to answer while writing the code and expensive to answer while rewriting the doc.

When a question cannot be answered yet (e.g., concurrent-pod test needs CI infrastructure that doesn't exist), the answer is *"deferred to Phase X, tracked in [link]"* — never silence. Silence reads as *{{PROJECT_STANDARD}}*. Deferred-with-link reads as honest.

When the user asks "is this done?" — run the gate. Out loud, in writing, in the response. If you can answer affirmatively at the depth the work warrants, the answer is yes. If you can't, the answer is "not yet, here's what's missing."

**There is no outside.** Every done-claim passes the gate; only depth varies. On routine work that is a single honest sentence — what was claimed, how it was checked — not a verification narrative. On hard-mandatory surfaces it is the full eight in writing. The discipline is preserved by being universal in attendance and proportional in depth. A gate with an exemption clause is a gate the agent will exempt itself from at precisely the wrong moment.

## The standard, restated

The bar is not *working*. The bar is *{{PROJECT_STANDARD}}*.

An agent reaches the bar the human holds, not the bar it states. The human's bar in this codebase is: **the work is provably correct, the counts are real, the magic numbers are justified, the failure modes are tested not asserted, and the doc tells the truth about what it does and doesn't do.**

This skill exists so the engineer agent holds the same bar, every time, without the human having to catch it.

## Cross-references

- {{METHODOLOGY_DOC}} — the engineering methodology this skill operationally implements at the agent layer. The methodology doc explains *why* this works (with the activation-function, backpropagation, and Wright-brothers framings worked out in detail); this skill enforces *how* the agent participates in it.
- Your project's contributor docs (e.g., `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`) — the codebase-audit methodology this skill is load-bearing for.
- initiate-change — the companion skill at the upstream *let's change X* gate: https://github.com/ronrey/initiate-change
