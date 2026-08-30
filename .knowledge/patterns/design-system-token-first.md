# Pattern: design-system-token-first

> **One-sentence summary.** Read `docs/DESIGN-SYSTEM.md` before any frontend work; if a needed token does not exist, extend the spec first, then the SCSS, then the code — never inline a new value.

**Status:** active · since 2026-08
**Last validated:** 2026-08-30
**Used in:** every frontend task in this project

## What it is

Three steps, in this order:

1. Read `docs/DESIGN-SYSTEM.md` (currently v2.5). Locate the relevant section: §2 for colors, §5 for type, §8 for radius/shadow/spacing, §7 for component shape, §12 for page layout, §13 for units.
2. If the value you need already exists as a token, use the SCSS variable from `_tokens.scss` or the TS variable from `theme/tokens.ts`. Both names must match the spec.
3. If the value does **not** exist, write the spec first (new token + section), then the SCSS variable, then the code. The order is not negotiable — see `AGENTS.md` §Source of truth.

The HTML showcase at `../pages/design-system/index.html` has drifted out of sync and is no longer authoritative; it is only useful for occasional visual cross-checking.

## Why it works

AI coding agents default to inventing per-page values. Each generated page acquires its own gray, its own radius, its own border-radius. The drift is invisible within a single page and obvious across five pages. A single source of truth that the agent reads before coding is the cheapest way to keep the visual language coherent.

## When to use

- Any new component or page.
- Any visual change that requires a value not yet in the spec.
- Any cross-page consistency check.

## When **not** to use

- Pure behavior / logic work that does not change visuals.
- Backend / contract / storage changes.
- Documentation-only changes.

## Example — token map before writing JSX

| What | Token (from `docs/DESIGN-SYSTEM.md` §2 / §5 / §8) |
|---|---|
| Page background | `$ds-content-bg` (§12.2) |
| Card surface | `$ds-surface` / `$ds-surface-subtle` |
| Card border | none (§7.4) — use `$ds-shadow-card` instead |
| Body text | `$ds-ink` / `$ds-ink-muted` / `$ds-ink-light` |
| Brand | `$ds-brand` / `$ds-brand-soft` / `$ds-brand-hover` |
| Data colors | `$ds-data-did` / `$ds-data-bad` / `$ds-data-thinking` (never `$ds-brand`) |
| Type | `$ds-font-size-caption` / `-body` / `-title` / `-header` |
| Radius | `$ds-radius-sm` / `-md` / `-lg` / `-pill` |
| Shadow | `$ds-shadow-card` / `-hover` / `-floating` |

If a value in the prototype is not in this table, stop and tell the user before introducing it.

## Anti-shape (what to avoid)

- Inline `border-radius: 13px` outside the four token tiers.
- A new hex code (`#5a8a6f`, "just a slightly different green").
- A new shadow box drawn from scratch.
- A new font-size step between the four.
- Using `$ds-brand` for data-viz (collides in sunset / walnut / mulberry themes).

## See also

- `../../docs/DESIGN-SYSTEM.md` — the spec.
- `../../AGENTS.md` §Source of truth — the rule.
- `../.claude/skills/prototype-to-taro/SKILL.md` §3 — token map before any JSX.
- `../.claude/skills/refactoring-ui/SKILL.md` — visual hierarchy and depth.
