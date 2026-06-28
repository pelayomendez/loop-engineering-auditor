# Sources

This skill's rubric is derived entirely from one primary document and the works it synthesizes.
It encodes no proprietary content — only the publicly-described framework.

## Primary source

- **Title:** *Loop Engineering: The Anthropic Playbook for Designing Systems That Prompt Your Agents* — "A Field Study of Designing Loops That Run Themselves"
- **Subtitle/format:** 2026 Working Note on Agentic Software Engineering Practice (conference-style reformatting)
- **Author of the reformatting:** Hua Shu / Hua Sheng (© 2026)
- **Origin:** An independent reformatting of the open "Orange Book" guide *Loop Engineering: Stop Asking Me What It Is* (v260615), originally released June 2026; freely available at `huasheng.ai/orange-books`.
- **Provided to this skill as:** `Loop-Engineering-IEEE.pdf` (11 pages), supplied by the user.

This document is the single source for the four-layer stack, the five moves, the six parts, the
generator/evaluator analysis, the five anti-patterns, the four silent costs, the first-loop
checklist (Table VI), the scheduling comparison, and the two-toolchain mapping.

## People and works the document credits

These are the original sources the working note synthesizes (as listed in its References and
Acknowledgment sections):

1. **Addy Osmani** — "Loop Engineering," personal blog and Substack, June 2026. Named the term; source of the framework and quoted formulations, the four-layer "one floor above the harness" framing, and the morning-triage loop example.
2. **Peter Steinberger** — post on designing loops that prompt coding agents, social media, June 2026 (author of OpenClaw). The post that "lit the fuse."
3. **Boris Cherny** — public remarks on writing loops that prompt Claude, Anthropic (lead on Claude Code), June 2026.
4. **Prithvi Rajasekaran (Anthropic)** — "Building long-running agentic applications: the generator/evaluator pattern," Anthropic engineering blog, 2026. Source of the self-praise finding and "tune a skeptic, don't fix a modest author."
5. **Steve Kaliski (Stripe)** — "Stripe's Minions: 1,300 PRs a week," *How I AI* podcast, 2026. The enterprise-scale case; reliability from constraints, not model size; built on a fork of Goose.
6. **Model Context Protocol (MCP) specification** — open standard, 2025–2026.
7. **Goose** — open-source agent framework, project documentation, 2025–2026.
8. **Claude Code documentation** — `/loop`, `/goal`, worktrees, skills, automations, Anthropic, 2026.
9. **Hua Shu**, *Loop Engineering: Stop Asking Me What It Is*, Orange Books, v260615, June 2026.

## Notes on fidelity

- The working note itself cautions that widely-circulated figures (e.g. "~90% of Claude Code is written by itself") are mostly secondhand and should be treated as rough reference. This skill therefore avoids asserting such numbers.
- Product/command details (Claude Code version numbers, Codex feature names) may change; the skill audits for *capabilities*, not specific commands. Refer to each tool's official documentation for current syntax.
- This skill is an analytical tool that applies the framework's concepts; it does not reproduce the source document.
