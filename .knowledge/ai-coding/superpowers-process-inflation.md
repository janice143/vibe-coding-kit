# AI coding note: superpowers-process-inflation

> **One-sentence summary.** Comprehensive AI workflow frameworks (brainstorm → spec → plan → subagent → test → review) inflate process for small tasks, turning 5-minute changes into 30-minute procedure.

**Observed:** 2026-08 (first half of project)
**Agents involved:** Claude Code with full plugin suite, OpenCode
**Task type:** copy edits, CSS adjustments, isolated bug fixes

## What the agent did

For a copy edit (change "每日总结" to "今日回顾"):

1. Entered brainstorm mode.
2. Asked clarifying questions about the rename's motivation.
3. Generated a spec.md describing the change.
4. Generated a plan.md with steps.
5. Decomposed into a task list.
6. Considered spawning a subagent for "independent review".
7. Wrote the change.
8. Wrote tests for the change.
9. Wrote a handoff report.

The actual code change was one string in one file. The procedure that surrounded it was 20× the change.

## Why it is the default

Coding agents are trained on professional software engineering workflows where the cost of bugs is high and the cost of process is amortized over many commits. The agent does not know that a personal project has different trade-offs. The skill or framework that codifies "good practice" treats every task as if it deserves the full ceremony.

The bug is not the framework itself; it is the framework's failure to scale to small tasks.

## What it cost

- A copy edit that should take 30 seconds took 20 minutes.
- The user (the project's owner) eventually disabled the framework entirely.
- The user replaced it with: scale process to task size, with `inspect → act → verify → report` as the default, expanded only for large or high-risk work.

## What to do instead

1. Apply the `task-size-router` skill at the start of every task — 30-second routing decision, not 5-minute analysis.
2. Default: `inspect → act → verify → report`. No plan mode for an obvious change. No subagent for serial work.
3. Promote to medium / large only when scope genuinely warrants it (cross-module boundary, data-model change, unclear requirements).
4. Recognize that "the framework exists" is not a reason to use it. Use a framework when its overhead is smaller than its benefit.

## See also

- `../.claude/skills/task-size-router/SKILL.md` — the routing procedure.
- `../../AGENTS.md` §Minimum sufficient process — the rule form.
- `../.claude/skills/` — the project's own skill set, which intentionally avoids framework-style ceremony.
