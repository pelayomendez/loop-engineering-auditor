# loop-engineering-auditor

**Point it at your agent loop. Get back a scored report on whether it's actually safe to run unattended — and the exact fixes, in priority order.**

A Claude skill that audits a self-running coding-agent loop, automation pipeline, or
generator/evaluator setup against the **Loop Engineering** framework. It reads your real files
— CI workflows, `.claude/skills`, agent configs, cron jobs, worktrees, state files, MCP
connectors — and grades the design, with a file-and-line citation behind every score.

## Why it exists

A loop is a machine for maximizing turns — and **the cost of a mistake scales with how many
turns it survives before someone catches it.** Most "loops" ship the parts that produce visible
output (lots of PRs) and skip the parts that produce safety. This skill finds that gap before
your overnight run does.

## What you get

A structured scorecard with evidence, detected anti-patterns, cost exposure, and a
leverage-ordered fix list. It grades against:

- **Five moves** every turn — discovery, handoff, verification, persistence, scheduling
- **Six parts** that realize them — automations, worktrees, skills, connectors, sub-agents, memory
- **Five anti-patterns** — nodding, amnesiac, tangled, blind, manual (one per skipped move)
- **Four silent costs** — verification debt, comprehension rot, cognitive surrender, token blowout
- The **first-loop checklist** — six load-bearing elements, pass/fail

## Use it

Install the folder where your Claude tooling discovers skills, then ask:

> "Audit my agent loop in `./my-repo` — is it actually safe to run unattended?"

It triggers on any request to review, audit, grade, or improve an autonomous loop, self-running
pipeline, coding-agent setup, scheduled workflow, or generator/evaluator design. Pointing it at
an *empty* repo to build a first loop? It'll hand you the first-loop checklist instead.

## Layout

```
loop-engineering-auditor/
├── SKILL.md                    # the skill: workflow + report template
├── README.md                   # you are here
└── references/
    ├── framework.md            # the framework as a scoring rubric
    ├── detection-signals.md    # file patterns / grep queries per part
    └── further-reading.md      # peer-reviewed papers backing each scoring rule
```

## Provenance & credibility

The rubric encodes the publicly-described **Loop Engineering** framework — no proprietary content
is reproduced. It synthesizes work credited to **Addy Osmani** (named the term), **Peter
Steinberger**, **Boris Cherny** (Anthropic, Claude Code), **Prithvi Rajasekaran** (Anthropic,
the generator/evaluator pattern), and **Steve Kaliski** (Stripe's Minions — 1,300 PRs/week),
alongside the MCP spec, Goose, and Claude Code docs.

Every scoring rule is additionally grounded in peer-reviewed research — self-preference bias in
LLM judges, the limits of self-correction, independent/adversarial evaluation, episodic memory,
and automation bias — so the auditor cites *mechanisms*, not assertions. Full citations live in
[`references/further-reading.md`](./references/further-reading.md).

A note on fidelity: widely-circulated figures (e.g. "~90% of Claude Code writes itself") are
secondhand and treated as rough reference, not asserted. The skill audits for *capabilities*,
not specific product commands — refer to each tool's docs for current syntax.

## License

Framework concepts belong to their original authors (credited above). This skill — the rubric
encoding, detection signals, and report template — is provided as-is for analysis use.
