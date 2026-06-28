# The Loop Engineering Framework (reference)

Condensed from the June 2026 working note *Loop Engineering: The Anthropic Playbook for
Designing Systems That Prompt Your Agents* (see `../SOURCES.md`). This is the rubric the
auditor scores against. Read the section you need; you don't have to load all of it.

## Table of contents
1. The four-layer stack
2. The five moves of one turn
3. The six parts
4. Parts ↔ moves mapping
5. Generator / evaluator (the "say no")
6. The five anti-patterns
7. The four silent costs
8. The first-loop checklist (Table VI)
9. Scheduling: cloud vs. local
10. Same capability across two toolchains
11. The amplifier principle

---

## 1. The four-layer stack

Each layer minds something one size larger than the layer below. The layers stack; they do
not replace each other.

| Layer | What it minds | Core question |
|-------|---------------|---------------|
| Prompt eng. | Writing one good prompt | What should I tell the model? |
| Context eng. | What goes in the window now | What to retrieve, summarize, clear out |
| Harness eng. | Arming a single run | Which tools, which actions, what counts as done |
| **Loop eng.** | **Scheduling on the harness** | **How to make it run itself, over and over** |

Loop engineering sits one floor above the harness: the harness arms a single run; the loop
makes it run itself. Three verbs separate harness from loop — **runs on a timer**, **spawns
helpers**, **feeds itself** (its output becomes next round's input). The change is one of
identity: from the person who operates the agent to the person who schedules it.

**The single most important intuition:** each layer's failure has a different blast radius. A
bad prompt is caught on the spot. At the loop layer, a misreading gets written into the state
file, read back the next morning as established fact, and built upon for many turns — by the
time anyone looks, the wrong assumption is load-bearing. The cost of a mistake scales with the
number of turns it survives before someone catches it, and a loop is by construction a machine
for maximizing turns.

## 2. The five moves of one turn

Drop any one and the loop will not turn, or will turn in place.

1. **Discovery** — find this turn's work on its own (read CI / issues / commits). Sets the ceiling on the whole loop's quality.
2. **Handoff** — move the task into the agent's hands; each finding gets its own isolated git worktree.
3. **Verification** — swap in another agent to say "no". The hardest move and the easiest to skip.
4. **Persistence** — write state outside the conversation (PR + inbox + state file).
5. **Scheduling** — make it turn round after round; this is what makes one run into a loop.

## 3. The six parts

| Part | What it is | Note |
|------|-----------|------|
| Automations | Runs off a schedule/trigger | Should trigger a *named skill*, not a wall of cron-pasted instructions. Local (machine on) or cloud (machine off). |
| Worktrees | Isolated git dirs for parallel agents | Value scales with parallelism; turns "runs but messy" into "runs and clean". |
| Skills | Permanent project knowledge in `SKILL.md` | Pays off *intent debt* — the cost of re-explaining the project every turn. |
| Connectors | MCP hookup to external systems | Decide the loop's radius of vision. Filesystem-only = tiny loop. |
| Sub-agents | Generator separated from judge | When one agent is player and referee, the referee plays favorites. |
| Memory | Persistent state on disk | Context is flushed on refresh; memory persists across rounds/days. The agent forgets, the repo does not. |

## 4. Parts ↔ moves mapping

- Discovery runs on **Skills**
- Handoff runs on **Worktrees**
- Verification runs on **Sub-agents**
- Persistence runs on **Memory** (and **Connectors**)
- Scheduling runs on **Automations**

## 5. Generator / evaluator — the "say no"

The hardest part of a loop is not getting the agent to run, but putting something inside that
can say "no" — and the agent that wrote the code is the one least likely to say it.

- **It always praises itself.** An agent grading its own output praises it confidently (Rajasekaran). Not a smarts problem — it's grading its own homework. Its context is full of the reasons it wrote the code that way, so it sees the chain of self-persuasion, not the result.
- **Tune a skeptic, don't fix a modest author.** Tuning a standalone evaluator to be skeptical is far more tractable than making a generator critical of its own work. Structural, not wording: swap in another agent with different instructions that sees the code from scratch. (Idea borrowed from GANs.)
- **The evaluator should act, not just read.** Hook it to tools (e.g. Playwright MCP) so it opens the page, clicks, screenshots, inspects the DOM — judging "I clicked the button, the page navigated" rather than "this JSX looks fine". Swapping the underlying model helps too; same model + new instructions keeps its blind spots. Default stance: assume broken until proven otherwise.
- **Fresh model on the stop condition.** Maker–checker principle (from banking): the one who decides "done" must differ from the one doing the work. Claude Code's `/goal` runs until a condition holds, judged after each turn by a fresh small model. Don't confuse `/goal` (run-until-condition) with `/loop` (rerun on interval).

A loop's floor is its evaluator: the generator's level decides what a loop *can* produce; the
evaluator's level decides what it *will not* produce.

Example evaluator agent (`.claude/agents/reviewer.md`):
```
ROLE: Adversarial code reviewer.
ASSUME: this code is BROKEN until proven otherwise.
DO NOT praise. Find what fails.
CHECK, in order:
1. Does it run? (execute, don't read)
2. Tests: run them, paste real output.
3. Edge cases the author skipped.
4. Does behavior match the ticket?
USE Playwright MCP: open the page, click, screenshot, inspect the DOM. Judge behavior, not intent.
VERDICT: PASS only if every check holds. Otherwise REJECT + list each reason.
```
Stop condition: `/goal all tests in test/auth pass and the lint step is clean`

## 6. The five anti-patterns (one per skipped move)

| Anti-pattern | Move skipped | Symptom | Fix |
|--------------|--------------|---------|-----|
| **Nodding loop** | Verification | Never once said "no" across hundreds of turns (statistical impossibility = no real check) | Generator/evaluator split |
| **Amnesiac loop** | Persistence | No cumulative progress; each morning starts from the same place | State file on disk |
| **Tangled loop** | Handoff | Parallel agents collide in one working dir; merge is a mess. Shows up only under parallelism | One isolated worktree per task |
| **Blind loop** | Discovery | Human still spends the morning deciding what the loop should do | Teach discovery into a skill |
| **Manual loop** | Scheduling | Works the day it's demoed, silently stops after; last run = demo day | A real trigger (timer/event) |

These cluster: a hasty loop installs only discovery + handoff (the two that produce visible
output) and skips the three that produce safety.

## 7. The four silent costs

All four share one trait: silence while the loop runs. They reinforce one another (more
unverified output → less understanding → more surrender → longer unwatched runs → bigger bill →
more unverified output).

| Cost | What it is | Guard |
|------|-----------|-------|
| Verification debt | Unverified output piling up in the gap between "runs" and "right" | An independent evaluator |
| Comprehension rot | Humans no longer understand the code the loop wrote | Read a representative sample daily; force yourself to explain each change |
| Cognitive surrender | Humans stop forming an opinion and accept whatever it hands back | At least one permanent human checkpoint — the loop can execute but not decide |
| Token blowout | Idle bugs/retries burn the budget overnight (the only cost that hits the bill directly) | Hard caps set *before* shipping: per-run, daily, max-retries |

## 8. The first-loop checklist (Table VI)

| Element | Ask yourself |
|---------|--------------|
| Discovery | What does it read on a timer? (CI / issues / commits / inbox) |
| State file | Which disk file holds the cross-round memory? |
| Evaluator | Is there an independent check that can say "no"? |
| Isolation | Does each parallel agent get its own worktree? |
| Token cap | Did you set a spending ceiling? Who stops it if it runs off? |
| Human review | Which step pauses for you to look, rather than auto-ing all the way through? |

The first two decide whether the loop *can run*; the last four decide whether it gets into
trouble once it does. Beginners most often ship with only the first two built.

## 9. Scheduling: cloud vs. local

| | Cloud | Desktop | `/loop` |
|--|-------|---------|---------|
| Where it runs | cloud | machine | machine |
| Machine on? | no | yes | yes |
| Session open? | no | no | yes |
| Min interval | 1h | 1min | 1min |
| See local files? | no | yes | yes |

The choice follows mechanically from one question: is the loop's work glued to the local
machine, or can it leave? Local buys frequency + local-file access at the cost of keeping the
machine on; cloud buys true autonomy at the cost of a coarser interval and a fresh clone each
run. A mature loop often uses both. Don't confuse "run a few extra rounds while I'm here"
(local rerun) with "run even when I'm not" (cloud).

## 10. Same capability across two toolchains

Loop engineering is a set of capabilities, not a product. Audit for the capability, not the brand.

| Capability | Claude Code | Codex |
|-----------|-------------|-------|
| Scheduling | `/loop` worker | Automations tab |
| Run until met | `/goal` | automation rerun + judge |
| Parallel isolation | `--worktree` | background worktree |
| Sub-agents | `.claude/agents/` | `.codex/agents/` |
| External connection | MCP + plugins | MCP connector |
| Explicit skill | `SKILL.md` | `$skill-name` |
| Machine-off run | Cloud Routines | cloud (planned) |

## 11. The amplifier principle

The same loop, built by two people, can end in opposite places — and the difference is not in
the loop. A loop is a faithful multiplication sign, and what it multiplies is the person: bring
understanding and it amplifies understanding; bring laziness and it amplifies laziness. Two
loops built with a "free myself fast" mindset vs. an "I still mean to be the engineer" mindset
may be 90% identical in code; the difference is one or two checkpoints, and those decide who is
in control six months later.

Loops make generation nearly free; **judgment is the scarce resource**. Closing posture to
audit toward: *build the loop, but build it like someone who intends to stay the engineer, not
just the one who presses go.*
