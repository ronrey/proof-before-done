# proof-before-done

> A reusable skill that gates "done" claims with a seven-question verification protocol.
> The operational implementation of incremental engineering at the agent layer.

When an AI engineering agent (Claude Code, Codex, or any agent that consumes skill files)
is about to claim work is "done," this skill fires and forces the agent to answer seven
specific questions in writing before the claim propagates downstream. Each question
targets a recurring failure mode of agent-generated work.

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

- **Activation function** — the seven questions act as a per-step gate that decides
  whether "done" signals propagate forward to downstream work. Below the threshold (any
  question fails or is silent), work is reworked. At the threshold (all seven pass),
  downstream work can safely depend on this step.

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

Developed inside the ComOS engineering ecosystem (a multi-repository AI-native commerce
platform). Originally codified on 2026-05-19 after an audit surfaced seven distinct
failure modes in one design document. Validated at scale on 2026-05-21 during a session
that compressed roughly three weeks of architectural work across four production
repositories into a single afternoon. The methodology framing (activation function,
backpropagation, bounded iteration) was named explicitly on 2026-05-22 after the
neural-network structure was recognized in retrospect.

The skill is currently in production across four ComOS engineering repositories
(federation, retail, portal, services).

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
