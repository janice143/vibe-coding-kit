# Claude Code adapter

Read `AGENTS.md` first. It is the shared collaboration contract for this project and applies to all coding agents.

## Claude-specific behavior

- Follow the minimum-sufficient process in `AGENTS.md`: `inspect → act → verify → report` by default.
- Do not invoke workflow frameworks automatically.
- Use plan mode, written plans, specs, worktrees, subagents, or task decomposition only for genuinely large, ambiguous, high-risk, or cross-cutting work, or when the user explicitly asks.
- Do not create commits, push changes, or modify remote services unless the user explicitly asks.
- Preserve unrelated working-tree changes and do not delete build outputs or caches without explicit permission.

## Project-specific pointers

- Before any frontend change, read your design-system spec (typically `docs/DESIGN-SYSTEM.md`).
- The design-system spec is the **唯一真源** for design intent and visual specification (tokens, typography, component shape, layout). It is **not** the place for feature inventories, data / backup formats, API flows, or exact copy — see `AGENTS.md` and §0.1 of the document itself. If a routine feature change seems to require a design-system edit, fix the document's scope instead of documenting the feature.
- For UI, navigation, storage, theme, or native chrome changes, follow the multi-platform verification matrix in `AGENTS.md`.
- Keep final reports concise: changed paths, focused verification, platform limitations, and remaining unverified boundaries.