# proof-before-done

> A reusable skill that gates "done" claims with an eight-question verification protocol.
> The operational implementation of incremental engineering at the agent layer.

When an AI engineering agent (Claude Code, Codex, or any agent that consumes skill files)
is about to claim work is "done," this skill fires and forces the agent to answer a
specific set of questions in writing before the claim propagates downstream. Each question
targets a recurring failure mode of agent-generated work.

The gate fires on **every** done-claim — there is no opt-out. What scales is depth: routine
work passes with a short written statement, while risky surfaces (auth, migrations, deploy
config, cross-service contracts, money paths) get the full eight questions. An agent that
has mis-assessed its work will also mis-assess whether it needs a gate, so the decision to
run it is not the agent's to make.

## The eight questions

1. **Does my count match my list?** Recount from ground truth, never from memory or from what the previous step said.
2. **Did I test what I asserted, or did I assert it?** An unverified claim is fine when labeled unverified. Silent assertion is not.
3. **Are my magic numbers justified at the call site?** Why 24h and not 12h, one line away from the constant.
4. **Are my string-encoded keys schemaed?** One encoder, one decoder — not eighteen callers composing the same identifier inline.
5. **What does my work NOT solve?** Written before the "what landed" section, or you don't know the boundary of what you fixed.
6. **Does my verification floor match my claim?** A type-check proves syntax. If the claim is about runtime behavior, downgrade the claim or raise the floor.
7. **Is my framing honest about what I did?** Re-read the opening sentence. Does it claim more than the floor proves?
8. **Did the verification's inputs come from reality, or from me?** ← *the one most teams are missing*

Question 8 is the one that catches what the other seven structurally cannot. The first
seven test whether the work was **executed** correctly; none of them tests whether the
**premise** was right. If you wrote the artifact and also wrote its test fixtures, a green
suite proves the two agree with each other — both can encode the same wrong assumption, so
the suite cannot fail. For anything whose job is checking other work (a hook, a linter, a
validator, a parser, an audit script), an invented fixture is an automatic block: capture
at least one real input first.

## The upstream companion

proof-before-done is the **downstream** half of a two-gate discipline:

| Stage | Gate | Fires at | What it gates |
|---|---|---|---|
| **Decide** | [initiate-change](https://github.com/ronrey/initiate-change) | *let's change X* | Whether the proposal is well-formed enough to be built against |
| Do | (the work itself) | — | — |
| **Declare done** | proof-before-done (this repo) | *this is done* | Whether the claim of done is well-formed enough to propagate |

Together they bracket every deliberate change: one gate at the moment you decide to
change something, one at the moment you claim it's finished. Each stands alone —
installing this skill does not require the other.

## What's in this repo

Two files, both useful on their own, more useful together:

- **`SKILL.md`** — the portable skill. Copy it into your project's `.claude/skills/`
  directory (or your agent's equivalent), fill in four placeholders, and the gate is
  installed.

- **`INCREMENTAL-ENGINEERING.md`** — the longer methodology document that explains
  *why* the skill produces the results it does. Read this if you want to understand the
  discipline; skip it if you just want the tool.

## Why this exists

Agents reach the bar the human holds, not the bar they state. Without an explicit gate
at every step of the work, agents propagate plausible-looking "done" claims that don't
survive contact with production. The cost compounds: each unverified step becomes a
foundation that subsequent steps build on, and the bugs you find later are exponentially
more expensive than the bugs you would have caught at the gate.

This skill is the gate. It is small, deliberate, and load-bearing. It was developed
inside a multi-repository production codebase (ComOS — an AI-native commerce platform)
and validated when a single afternoon's worth of work, run through the discipline,
compressed roughly three weeks of architectural design into four hours across four
repositories.

## What the skill is, structurally

The skill is not a checklist. It is a small neural network for high-quality coding,
operating at the agent layer:

- **Activation function** — the eight questions act as a per-step gate that decides
  whether "done" signals propagate forward to downstream work. Below the threshold (any
  question fails or is silent), work is reworked. At the threshold (all pass at the depth
  the work warrants), downstream work can safely depend on this step.

- **Backpropagation** — when the gate catches a failure, the correction signal flows
  backward through the layers of work that produced it. The specific assertion, the
  reasoning chain, the missing convention, the discipline that didn't fire — each gets
  a credit-assigned update. Your team's documentation is the weight matrix; failures
  caught by the gate update the weights.

- **Bounded iteration** — question 5 (*what does my work NOT solve?*) forces explicit
  scope declarations. This is what makes the methodology converge rather than thrash.
  Each iteration's contribution is scoped; future iterations land against a clear seam.

That mapping isn't decorative. The skill behaves like a network: forward pass produces
the work, the activation function gates the "done" signal, the loss function (your
judgment) fires when work falls short, backprop credit-assigns the correction, the
weights (your docs) update permanently. Over many iterations, the network learns to
catch its own failures before the human has to.

## What you actually get when you install

The skill is the *architecture* of a quality-coding network. The weights — your team's
accumulated documentation, conventions, decision logs, contributor docs — are what
make the network produce *your* team's quality, not generic quality. Installing the
template gives you the architecture. The first dozen iterations train the weights.

Teams that adopt this as a checklist will get marginal improvement. Teams that adopt
it as architecture for a network that needs training — and put in the iterations to
let it train — will get the compounding results.

## How to adopt

Three steps:

1. **Copy `SKILL.md`** into your project's agent skill directory:
   ```
   <your-project>/.claude/skills/proof-before-done/SKILL.md
   ```
   For Codex or other agents, follow your agent's skill-installation convention.

2. **Fill in four placeholders** documented in the HTML comment at the top of
   `SKILL.md`:
   - `{{PROJECT_NAME}}` — your codebase or organization
   - `{{PROJECT_STANDARD}}` — your team's bar word ("amazing", "production-ready",
     "ship-ready" — a word that genuinely means more than "compiled")
   - `{{ORIGIN_INCIDENT}}` — the audit or incident that motivated installation
   - `{{METHODOLOGY_DOC}}` — your team's engineering standards doc, if any

3. **Delete the HTML comment block**. The skill should never ship with placeholder
   strings.

The full adoption guide, including what NOT to change and how to verify the skill
loaded correctly, is in the comment block in `SKILL.md` itself.

## What this is not

- Not a process replacement for thinking. The methodology works because the human is
  genuinely engaged in the judgment loop. A team running this on autopilot will get
  autopilot results.

- Not a guarantee of correctness. The skill raises the floor of what reliably ships;
  it does not eliminate the ceiling of failure. New failure modes will surface; what
  changes is whether they surface in planning (cheap) or production (expensive).

- Not specific to AI agents. Human engineers benefit from the same gate. The framing
  emphasizes agent-natural failure modes because that's where the gate is most often
  *missing*, but the discipline is older than AI by decades. See
  `INCREMENTAL-ENGINEERING.md` for the mechanical-engineering lineage.

## Origin and provenance

Developed inside the ComOS engineering ecosystem (an AI-native commerce platform).
Originally codified on 2026-05-19 after an audit surfaced seven distinct failure modes in
one design document — those seven became the first seven questions. Validated at scale on
2026-05-21 during a session that compressed roughly three weeks of architectural work
across four production repositories into a single afternoon. The methodology framing
(activation function, backpropagation, bounded iteration) was named explicitly on
2026-05-22 after the neural-network structure was recognized in retrospect.

**The eighth question came later, and it came from the gate failing.** A checker was
written, its fixtures were hand-authored by the same author, and the suite passed 5/5
while the checker was wrong — because the code and the fixtures encoded one shared wrong
premise. The other seven questions could all be answered honestly and still let it
through: they test whether the work was *executed* correctly, not whether its premises
were. Question 8 asks where the verification's inputs came from, and it is the reason a
self-authored checker with self-authored tests no longer counts as verified.

Two related corrections landed at the same time. The gate had briefly been made opt-in,
and it silently stopped firing — an agent that has mis-assessed its work also mis-assesses
whether it needs a gate. It is now always-on with depth scaling to risk. And the list of
hard-mandatory surfaces exists because "how small is this diff" turned out to be a poor
predictor of how expensive the failure would be.

The skill runs in production in the ComOS repositories today, and has been through several
generations of the codebase it governs — including deprecating two of the four repos it
was originally validated in.

## License and attribution

Shared in the spirit that disciplines should be portable across teams. If you adopt it,
you don't owe attribution. If you find it useful and adapt it further, sharing your
adaptations back (or upstream via a pull request) helps the next team.

## Contributing

This repository is the canonical public home of the skill. If you find bugs in the
template (capitalization issues, broken cross-references, unclear placeholder
documentation), please open an issue or send a pull request.

If you adopt the skill and want to share what you learned — which failure modes the gate
caught for your team, which conventions your weights ended up encoding, how long it took
to see compounding results — those notes are welcome. The methodology gets stronger as
more teams run it.

## Cross-references

- `SKILL.md` — the template (the part you adopt)
- `INCREMENTAL-ENGINEERING.md` — the methodology (the part you understand)
- [initiate-change](https://github.com/ronrey/initiate-change) — the companion skill at
  the upstream *let's change X* gate
