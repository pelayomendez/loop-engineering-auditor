# Detection signals (reference)

Concrete things to look for when auditing a real repo or config. Run these before scoring so
every ✅/❌ in the report carries a file/line citation. Patterns cover both Claude Code and
Codex toolchains; absence of a pattern is evidence too.

> Tip: run a broad sweep first (`git ls-files`, then targeted greps), then read the few files
> that match. Cite the path/line you found — or note explicitly that you searched and found nothing.

## Scheduling (→ Automations)

**Present if:** a real trigger fires the loop without a human pressing go.

- CI schedule: `grep -rn "schedule:" .github/workflows/` and look for `cron:` lines.
- GitHub Actions: `.github/workflows/*.yml` with `on: schedule` or `workflow_dispatch` repeated.
- Claude Code: `/loop` invocations, a `loop.md`, `.claude/` automation config, Cloud Routines.
- Codex: an Automations tab config / scheduled job.
- OS-level: `crontab`, `launchd` plists, systemd timers, `at` jobs.

**Red flag (Manual loop):** the only way it runs is a human typing a command. A README that
says "run this each morning" with no trigger = scheduling skipped.

## Handoff (→ Worktrees)

**Present if:** each parallel task runs in its own isolated working directory.

- `grep -rn "worktree" .` — `git worktree add`, `--worktree`, `-w fix/...`.
- Per-task sandboxes / containers (one env per agent), "cattle not pets" environment swapping.
- Distinct branch-per-task or dir-per-task creation in the orchestration script.

**Red flag (Tangled loop):** multiple agents spawned but all `cd` into the same repo root /
write the same files. Single-agent loops look fine here — the collision only appears under
parallelism, so flag the *risk* even if you can't observe a collision.

## Discovery (→ Skills)

**Present if:** the loop finds its own work, and the discovery logic lives in a reusable skill.

- `find . -name SKILL.md` / `.claude/skills/*/SKILL.md`; Codex `$skill-name` references.
- A skill/step that reads CI failures, open issues, recent commits, or an inbox.
- Discovery logic in a maintained file — NOT a giant instruction block pasted into a cron line.

**Red flags:**
- *Blind loop:* the human hands it a fixed task list each run (e.g. "fix these 3 bugs"); no auto-surfacing of work.
- *Intent debt:* the same project context is re-explained inline every run instead of captured once in `SKILL.md`.

## Verification (→ Sub-agents)  ← the highest-leverage check

**Present if:** an *independent* evaluator can say "no" — different agent, ideally different
model, defaults to doubt, verifies by acting, final say by a fresh model.

- `.claude/agents/*.md` (or `.codex/agents/`) with a reviewer/adversary role distinct from the generator.
- Look for "assume broken / until proven", "do not praise", "REJECT + reasons".
- `/goal <stop condition>` with a separate judge model; maker–checker on the stop condition.
- Evaluator wired to tools that *act* (Playwright MCP, runs tests, screenshots) — not read-only review.

**Red flags (Nodding loop):**
- The same agent both writes and grades (`grep` the agent config — is there only one role?).
- Stop condition judged by the generating model itself.
- "Reviewer" only reads diffs, never executes.
- A loop with hundreds of runs and no record of a rejection. **Do not score Verification present just because a step is *named* "review" — confirm independence.**

## Persistence (→ Memory + Connectors)

**Present if:** state survives outside the conversation window and the next run reads it back.

- A committed state file: `state/*.md`, `triage.md`, a tracked board export; `git log` shows it updated across runs.
- The loop writes findings/status to disk AND reads the previous file at the start of the next run.
- Connectors that persist: PR creation, issue-tracker updates, an `inbox/` for human handoff.

**Red flag (Amnesiac loop):** results live only in chat/context; no on-disk artifact carried
between runs; each run rediscovers or redoes the same work.

## Connectors (→ radius of vision)

- `grep -rn "mcp" .` , MCP server configs, `.mcp.json`, plugin manifests.
- Hookups to issue tracker / Slack / database / staging API.
- **Red flag:** filesystem-only loop = a tiny loop; it can't see or act on the outside world.

## The four silent costs (assess even when the loop "works")

- **Verification debt:** count merged-without-review artifacts; is there *any* independent gate? High exposure if PRs auto-merge on green tests with no second agent.
- **Comprehension rot:** is there a daily "read a sample" discipline documented? Absence = rising exposure as volume grows.
- **Cognitive surrender:** is there a permanent human checkpoint, or does everything auto-flow to merge?
- **Token blowout:** `grep -rn` for budget/cap/max-retries/timeout settings. No caps = the loop has delegated spending authority to its own bugs.

## First-loop checklist quick-grep

```
# Discovery source
find . -name SKILL.md; grep -rn "issues\|CI\|commits" .claude/ .github/
# State file
git ls-files | grep -i "state\|triage\|inbox"
# Evaluator
ls .claude/agents/ .codex/agents/ 2>/dev/null; grep -rln "reviewer\|adversar\|assume.*broken" .
# Isolation
grep -rn "worktree" .
# Token cap
grep -rni "budget\|max.?retr\|timeout\|cap" .
# Human review
grep -rni "auto.?merge\|inbox\|review\|approval" .
```
