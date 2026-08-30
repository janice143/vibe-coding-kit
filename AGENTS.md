# Agent Collaboration Guide

This file is the shared collaboration contract for every coding agent used in your project (Claude Code, OpenCode, Python-based agents, and others). Tool-specific instruction files (e.g. `CLAUDE.md`) may add tool behavior but must not contradict this file.

## About this contract

This file is a **starter**, not a finished standard. The rules listed below are a default — a way of thinking about how an AI agent should collaborate on *your* codebase. They are expected to grow and adapt with your project:

- **Add** rules as you discover new invariants your project actually has.
- **Remove** rules that turn out not to apply, or that you find yourself constantly overriding.
- **Rewrite** rules so the wording matches how your team actually talks about the work.

The value of this kit is not the contents of these files — it is the *practice* of writing down invariants, separating rules from procedures from project facts, and keeping the three layers honest. A short, accurate `AGENTS.md` your agents actually follow is more useful than a long one copied verbatim from a template. If a rule here does not fit your project, delete it; if your project demands a rule that is not here, write it.

## Source of truth

- **Design system.** If your project has a design-system spec (typically `docs/DESIGN-SYSTEM.md`), that document is the **唯一真源** for design intent and visual specification — tokens, typography, icons, component shape, layout blocks, and how interaction states are presented. Any HTML / Figma / image showcase is implementation reference, not source of truth.
- **Do not put non-design content in the design system.** Feature inventories (which entries a page currently has), data contracts (fields, schema versions, backup / export formats), API and business flows, and exact copy all belong elsewhere: `docs/PRODUCT-BRIEF.md`, `docs/INTERACTION-CONTRACT.md`, `docs/LOCAL-STORAGE.md`, or the code itself.
- **The test:** *removing or adding a feature entry must not require editing the design system.* If a routine feature change forces a design-system edit, the document is over-specified — fix the document instead of documenting the feature. Conversely, changing a radius, adding a font-size step, or altering card layering *must* update the design system first.
- New design tokens require updating the design-system document before code.

## Repository map and common commands

Edit this section to match your project. The map below is illustrative.

- `apps/client/` (or `src/`) — the application surface.
- `apps/api/`, `packages/database/`, `docker-compose.yml` — adjust or remove if you are local-first.
- `packages/contracts/` — shared types consumed across surfaces.
- `docs/` — project documentation and specifications.
- `skills/` — agent procedural skills.
- `.knowledge/` — project-specific knowledge base.

Run commands from this directory:

```sh
npm start            # dev watch (both platforms, if applicable)
npm stop
npm run dev:h5
npm run dev:weapp
npm run build
npm run test
npm run typecheck
```

Replace with the actual scripts your project exposes.

## Minimum sufficient process

Default workflow:

```text
inspect → act → verify → report
```

Use only the process the task needs. Do not automatically create specs, plan documents, task lists, worktrees, subagents, issues, commits, pull requests, or remote actions.

### Small changes

Examples: copy edits, value adjustments, an isolated visual bug, a straightforward local bug fix, or an obvious change in one to three files.

- Read the relevant files, make the focused change, and run the narrowest relevant verification.
- Do not enter plan mode, write a spec, decompose tasks, or launch subagents.
- Do not restart planning after a clarification unless scope materially changes.

### Medium changes

Examples: several related files, a contained refactor, or a change with meaningful but localized implementation choices.

- Inspect the affected code and state a short inline approach if it helps alignment.
- Ask only questions that block a safe decision; otherwise implement directly.
- Do not create persistent specs or plans by default.
- Verify at the affected boundary.

### Large or high-risk changes

Examples: a new subsystem, architecture or data-model change, cross-cutting refactor, security-sensitive change, migration, or genuinely unclear requirements.

- Clarify the important constraints before implementation.
- Present an implementation approach for approval.
- Use a written plan, independent review, worktree, or subagents only when their benefit outweighs their overhead or the user requests them.

## Change safety

- Preserve unrelated working-tree changes. Never overwrite or revert them without explicit permission.
- Do not delete source files, build outputs, caches, generated assets, branches, or database data unless explicitly requested. Build tooling or developer tools may depend on them.
- Do not commit, push, create or comment on issues / PRs, change external services, or alter permissions unless explicitly requested.
- Validate untrusted input at system boundaries. Do not introduce command injection, XSS, SQL injection, or other OWASP risks.

## Agent use

- Use direct reads / searches when the relevant path or symbol is known.
- Use an exploration agent only for genuinely broad repository investigation.
- Use subagents only for independent work that benefits from parallelism or context isolation.
- Do not use workflow frameworks that automatically impose brainstorming, specs, plans, worktrees, or approval chains on every development task.

## Frontend and page boundaries

- Follow your design-system spec exactly; do not invent tokens, arbitrary radius values, or arbitrary shadows.
- Pages split by domain object, not UI state: one record stays on one route and switches among `readonly`, `edit`, and `create` modes internally. Shared chrome belongs in components.
- When asked to "split" or "decouple" a page, confirm whether the boundary is data objects or interaction states before changing routes.

## User-facing product voice

Treat every user-facing screen as if it were the first and only version of the product. Never mention previous versions, changes, improvements, redesigns, decisions, or why something was changed. Describe only the product's current state and what the user can do now.

This rule governs UI copy, empty states, onboarding flows, prompts, tooltips, dialogs, release notes shown to users, and any other content the user reads. Internal handoff reports and developer-facing commit messages are not user-facing and may discuss history freely.

## Multi-platform verification

If your project targets more than one runtime (e.g. H5 + WeChat Mini Program, iOS + Android, web + desktop), Taro / React Native / similar tools differ across platforms in routing, storage, navigation chrome, CSS variables, and DOM support.

For a feature that touches UI, navigation, storage, themes, or native chrome:

1. Run the project's dev / build script to watch all targets, or a single target if appropriate.
2. Exercise the change in each environment that is locally available.
3. Build a release artifact and test on a real device when applicable.
4. Include a verification matrix in the handoff. Explicitly list unavailable verification rather than implying it passed.

A successful build or typecheck on one platform does not prove the other platform works. Surface platform differences explicitly.

For a small SCSS or component adjustment, run only proportional verification; do not run unrelated full-platform builds.

## Handoff

Keep updates concise. Report:

1. Changed paths and intent.
2. Focused verification and result.
3. Relevant platform verification matrix, including unverified rows.
4. Any remaining limitation or environment requirement.

Do not claim UI completion when visual interaction could not be tested.