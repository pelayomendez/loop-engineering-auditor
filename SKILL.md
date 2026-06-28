---
name: loop-engineering-auditor
description: Audit an existing agentic / coding-agent design against the Loop Engineering framework (Osmani, Steinberger, Cherny, Rajasekaran; Stripe Minions). Use this skill whenever the user wants to review, audit, grade, critique, or improve an autonomous agent loop, a self-running pipeline, a coding-agent setup, a "loop", a scheduled agent workflow, or a generator/evaluator setup — even if they don't say "loop engineering". Triggers include: "review my agent design", "is my loop safe / reliable", "audit this automation", "why does my agent keep approving its own bad output", "grade my .claude setup / GitHub Actions agent / Codex automation", or pointing at a repo and asking what's missing. Inspects real files (workflows, .claude/skills, agent configs, cron/automation, worktree usage, state files) and produces a scored report with concrete fixes.
---

# Loop Engineering Auditor

Audit an existing agentic design — a self-running coding-agent loop, automation pipeline,
or generator/evaluator setup — against the **Loop Engineering** framework, and return a
**scored report with concrete fixes**.

The framework comes from a June 2026 working note synthesizing Addy Osmani, Peter
Steinberger, Boris Cherny (Anthropic), Prithvi Rajasekaran (Anthropic), and Steve Kaliski
(Stripe). The full framework and citations live in `references/framework.md` and `README.md`.
When a builder pushes back on a finding, ground it in the mechanism: `references/further-reading.md`
maps the framework's load-bearing claims (self-preference bias, the limits of self-correction,
independent verification, automation bias) to the peer-reviewed literature behind them.

## Core idea you are auditing against

Loop engineering is the fourth layer of a stack (**prompt → context → harness → loop**). A
loop *removes the human from the inner cycle of doing the work*. The central risk: **the cost
of a mistake scales with the number of turns it survives before someone catches it, and a loop
is a machine for maximizing turns.** Everything you check for exists to shorten the distance
between a mistake and its discovery.

A real loop performs **five moves** every turn, realized by **six parts**. Each move that is
skipped produces a named **anti-pattern**, and loops that run unwatched accrue **four silent
costs**. Your audit grades all of these.

## When to use vs. not

Use this when there is an *existing* design to evaluate (a repo, a config, a described
setup). If the user wants to *build* their first loop from scratch, point them to the
first-loop checklist in `references/framework.md` instead — don't force an audit onto an
empty repo.

## Workflow

Work through these steps in order. Don't skip the evidence-gathering — a score without a
file/line citation is just an opinion, and this skill exists to replace opinion with evidence.

### Step 1 — Establish what you're auditing

Confirm the target: a path to a repo/folder, a config file, or (fallback) a written
description. If given a repo, inspect it directly. If given only prose, score what's described
and clearly flag every item you could not verify from files as "asserted, not verified".

### Step 2 — Gather evidence for the six parts

Read `references/detection-signals.md` for the concrete file patterns, commands, and grep
queries that indicate each part is present. For a repo, run those searches before scoring.
The six parts and what to look for, in brief:

| Part | Realizes move | Look for |
|------|---------------|----------|
| Automations | Scheduling | cron, `schedule:` in CI YAML, `/loop`, Codex Automations, timers/triggers |
| Worktrees | Handoff | `git worktree`, `--worktree`/`-w`, per-task isolated dirs, sandbox-per-agent |
| Skills | Discovery | `SKILL.md`, named skill invocation, `$skill-name` — NOT a wall of inline prompt |
| Connectors | Persistence / Discovery | MCP servers, issue-tracker/Slack/DB hookups, plugins |
| Sub-agents | Verification | separate generator vs. evaluator agents, `.claude/agents/`, reviewer config |
| Memory | Persistence | a state file on disk (markdown/board) committed across runs, not just context |

Record a one-line piece of evidence (file path, line, or quote) for every part — present or absent.

### Step 3 — Score the five moves

For each move, decide present / partial / missing, and note the evidence. A move is only
"present" if the part that realizes it actually does its job (e.g. Verification is *not*
present if the same agent that wrote the code also grades it — that's the Nodding anti-pattern).

1. **Discovery** — does the loop find its own work on a timer (reads CI/issues/commits), or is a human still handing it a task list each run?
2. **Handoff** — does each parallel task get an isolated worktree, or do agents share one working dir?
3. **Verification** — is there an *independent* evaluator that can say "no" (different agent, ideally different model, defaults to doubt, verifies by acting not just reading, final say by a fresh model)?
4. **Persistence** — does state survive outside the conversation window (a committed file/board the agent forgets but the repo doesn't)?
5. **Scheduling** — does a real trigger fire it (timer/event), or does a human run it by hand?

### Step 4 — Detect anti-patterns (one per skipped move)

Map any missing/partial move to its named failure. See `references/framework.md` for full
descriptions and symptoms:

- **Nodding loop** — verification skipped: generator grades its own work; "never once said no."
- **Amnesiac loop** — persistence skipped: no cumulative progress; restarts from scratch each run.
- **Tangled loop** — handoff skipped: parallel agents collide in one working dir.
- **Blind loop** — discovery skipped: human still decides what to work on each morning.
- **Manual loop** — scheduling skipped: works the day it's demoed, silently stops after.

### Step 5 — Assess the four silent costs

These don't throw errors; flag exposure even when the loop "works":

- **Verification debt** — unverified merged output piling up in the gap between "runs" and "right". Guard: independent evaluator.
- **Comprehension rot** — humans no longer understand code the loop wrote. Guard: read a representative sample daily and force yourself to explain it.
- **Cognitive surrender** — humans stop forming an opinion and just accept output. Guard: at least one human checkpoint kept permanently.
- **Token blowout** — idle bugs/retries burn the budget overnight. Guard: hard per-run / daily / max-retry caps set *before* shipping.

### Step 6 — Run the first-loop checklist

Confirm the six load-bearing elements (Table VI). Each is pass/fail with evidence:
Discovery source · State file on disk · Independent evaluator · Worktree isolation ·
Token cap + stop authority · Human review checkpoint (never auto-merge).

### Step 7 — Produce the scored report

ALWAYS use this exact template:

```
# Loop Engineering Audit — <target name>

## Verdict
<one paragraph: is this a real loop, a one-off, or one of the five anti-patterns wearing a disguise?>
Overall: <X>/10 moves+parts installed · Highest-risk gap: <name>

## Scorecard
### Five moves
| Move | Status | Evidence |
| Discovery | ✅ / ⚠️ / ❌ | <file:line or quote> |
| Handoff | ... | ... |
| Verification | ... | ... |
| Persistence | ... | ... |
| Scheduling | ... | ... |

### Six parts
| Part | Present? | Evidence |
| Automations / Worktrees / Skills / Connectors / Sub-agents / Memory | ✅/❌ | ... |

## Anti-patterns detected
<for each: name, the skipped move, the symptom you observed, the file evidence>

## Silent cost exposure
<verification debt / comprehension rot / cognitive surrender / token blowout — rate exposure low/med/high with reasoning>

## First-loop checklist
<6 rows, pass/fail + evidence>

## Fixes (ordered by leverage)
<numbered. The first two — discovery + the "no"-sayer — decide whether the loop gets into trouble; the rest decide whether it can stop once it does. Each fix: what to add, where, and a minimal snippet. Prioritize installing a real independent evaluator and a human review checkpoint over adding parallelism.>

## The amplifier note
<one line: this loop will faithfully multiply whatever its builder brings. State whether, as built, it scales judgment or scales the absence of it.>
```

### Step 8 — Verify your own audit

Before delivering, re-check: every ✅/❌ has a real file citation (not an assumption); you did
not credit Verification to a self-grading agent; and your top fix targets the highest-blast-radius
gap (usually verification or the human checkpoint), not the most visible one. If auditing from
prose only, state that limitation up front.

## Guardrails

- **Don't reward visible output over safety.** Beginners ship discovery + handoff (the parts that produce visible output) and skip verification, persistence, scheduling (the parts that produce safety). A loop that opens lots of PRs is not thereby a good loop.
- **Parallelism is earned, not assumed.** A loop earns the right to run many agents by first proving it can stop one bad one. If verification is weak, recommend fixing it before scaling worktrees.
- **Tool-agnostic.** The framework is a set of capabilities, not a product. Claude Code (`/loop`, `/goal`, `--worktree`, `.claude/agents/`, MCP, `SKILL.md`, Cloud Routines) and Codex (Automations tab, `.codex/agents/`, MCP connectors, `$skill-name`) realize the same six parts. Audit for the capability, not the brand. See the mapping in `references/framework.md`.
