# loop-engineering-auditor

A Claude skill that **audits an existing agentic design** — a self-running coding-agent loop,
automation pipeline, or generator/evaluator setup — against the **Loop Engineering** framework,
and returns a **scored report with concrete fixes**.

## What it does

Point it at a repo, a config, or a written description of an agent loop. It inspects real files
(CI workflows, `.claude/skills`, agent configs, cron/automation, worktree usage, state files,
MCP connectors) and grades the design against the framework:

- **Five moves** of one turn — discovery, handoff, verification, persistence, scheduling
- **Six parts** that realize them — automations, worktrees, skills, connectors, sub-agents, memory
- **Five anti-patterns** — nodding, amnesiac, tangled, blind, manual (one per skipped move)
- **Four silent costs** — verification debt, comprehension rot, cognitive surrender, token blowout
- The **first-loop checklist** (six load-bearing elements)

Output is a structured scorecard with evidence citations, detected anti-patterns, cost
exposure, and a prioritized list of fixes.

## Layout

```
loop-engineering-auditor/
├── SKILL.md                        # the skill: workflow + report template
├── SOURCES.md                      # full provenance and citations
├── README.md
└── references/
    ├── framework.md                # the framework as a scoring rubric
    ├── detection-signals.md        # file patterns / grep queries per part
    └── further-reading.md          # peer-reviewed papers backing each scoring rule
```

## Usage

Install the skill folder where your Claude tooling discovers skills, then ask something like:

> "Audit my agent loop in `./my-repo` — is it actually safe to run unattended?"

The skill triggers on requests to review, audit, grade, or improve an autonomous agent loop,
self-running pipeline, coding-agent setup, scheduled workflow, or generator/evaluator design.

## Provenance

The rubric is derived from the publicly-described Loop Engineering framework, with each
scoring rule additionally grounded in peer-reviewed research (self-preference bias, the limits
of self-correction, independent verification, automation bias). The primary source, every
credited author, and the supporting literature are documented in [`SOURCES.md`](./SOURCES.md)
and [`references/further-reading.md`](./references/further-reading.md). No proprietary content
is reproduced — the skill encodes the framework's concepts as an analytical tool.

## License

The framework concepts belong to their original authors (see `SOURCES.md`). This skill — the
rubric encoding, detection signals, and report template — is provided as-is for analysis use.
