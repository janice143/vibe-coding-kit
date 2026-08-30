# AI coding note: local-optimum-component-creation

> **One-sentence summary.** Coding agents default to creating page-local components (`_components/`) instead of searching the project for prior art, leading to parallel implementations of the same shape.

**Observed:** 2026-08 (every prototype-to-taro conversion; every medium refactor)
**Agents involved:** Claude Code, OpenCode, Codex
**Task type:** prototype conversion, refactor, new page

## What the agent did

Given a task to "implement this page", the agent:

1. Read the immediate file the user pointed at.
2. Generated the JSX inline in that file.
3. When the JSX exceeded ~80 lines or repeated a structure, created a sibling file under `_components/` for the page.
4. **Did not search `apps/client/src/components/`** for an existing Card / Dialog / EmptyState / SectionHeader that already covered the use case.

Across the project this produced:

- Three different Card implementations across pages, each with a different padding and radius.
- Two Dialog components with different title/body/footer structures.
- Three "empty state" blocks, none of which used the canonical `EmptyState`.
- Two pill components (`MoodPill` and `TagPill`) with the same visual but different markup.

## Why it is the default

The agent's local-optimum strategy solves the page at hand. It treats "the file I'm editing" as the universe. The training distribution favors speed of completion over project-wide search; the agent's loop does not penalize parallel implementations.

The cost is invisible during the implementation of any single page. It becomes visible only when a reviewer looks across the project and sees the same shape in three places.

## What it cost

- ~30% of total commit count was refactoring work that could have been prevented at first write.
- Several rounds of "extract component X" follow-up commits after the agent shipped parallel versions.
- The user eventually added a rule: "first reuse signal → stop and extract", codified in the `reusable-component-detect` skill.

## What to do instead

1. Before writing any non-trivial component, search `apps/client/src/components/` for prior art. The user typically lists the canonical components in the relevant skill (`prototype-to-taro` already enumerates them).
2. At the moment of detecting a second similar block (within the same page or across pages), halt and apply the `reusable-component-detect` skill.
3. Pick the right layer (component / hook / helper / token) per the skill's table; do not collapse all reuse into "make a component".
4. Update the contract: a new component goes into `AGENTS.md` §Shared components; a new token goes into `docs/DESIGN-SYSTEM.md` *before* the SCSS.

## See also

- `../.claude/skills/reusable-component-detect/SKILL.md` — the procedure.
- `../patterns/design-system-token-first.md` — the upstream discipline.
- `../../AGENTS.md` §Frontend and page boundaries — the rule that limits what an agent may do.
