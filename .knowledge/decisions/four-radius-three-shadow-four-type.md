# Decision: four-radius-three-shadow-four-type

> **One-sentence summary.** The visual system is constrained to four radius tiers (8 / 12 / 16 / pill), three shadow tiers (card / hover / floating), and four type sizes (13 / 15 / 17 / 24); no additional tiers may be introduced without updating the design-system spec first.

**Decided:** 2026-08
**Decided by:** project owner
**Status:** active

## Context

AI coding agents default to inventing per-page visual values. During early prototyping the codebase accumulated `border-radius: 4`, `10`, `13`, `14`, `17`, `18`, `20`, `36px` and `box-shadow: 0 1px 4px rgba(...)`, `0 3px 10px rgba(...)`, and font-size steps at `12`, `14`, `15`, `16`, `18`, `20`. Each looked fine in isolation; together they read as no system at all.

## Decision

Radius tiers: `--ds-radius-sm` `8px`, `--ds-radius-md` `12px`, `--ds-radius-lg` `16px`, `--ds-radius-pill` `999px`. Shadow tiers: `--ds-shadow-card`, `--ds-shadow-hover`, `--ds-shadow-floating`. Type sizes (text-bearing): `--ds-font-size-caption` `13px`, `--ds-font-size-body` `15px`, `--ds-font-size-title` `17px`, `--ds-font-size-header` `24px`. (Icon glyph sizes follow a separate three-tier system per `docs/DESIGN-SYSTEM.md` §5.3 — that is intentional, not a violation.)

Downstream SCSS and TS code may only reference these tokens. Bare `border-radius: Npx`, `box-shadow: ...`, or `font-size: Npx` outside the tiers is not allowed in `.scss` or `.tsx`. The HTML showcase at `../pages/design-system/index.html` has historical examples using non-token values; those are not load-bearing and are explicitly labeled as historical in `docs/DESIGN-SYSTEM.md` §8.1.

## Why

Constraining the visual system to a small token set is the cheapest way to keep AI-generated code consistent. The cost of discipline (occasionally having to reuse a slightly-less-perfect token) is much lower than the cost of drift (every page reads as a different product).

## Consequences

- Adding a new tier is a design change, not an implementation detail. It requires updating `docs/DESIGN-SYSTEM.md`, then the SCSS / TS variables, then the code.
- Designers cannot request "border-radius 13 on this card" without re-opening the spec.
- Some components will reuse a tier that is one step off from "ideal". This is accepted; consistency wins over local perfection.

## When to revisit

- A genuinely new visual mode is introduced (e.g. a "marketing landing page" inside the app, or an embedded media player with full-bleed controls). At that point the spec gains new tiers scoped to the new mode, not a global widening.
- The four tiers become clearly insufficient — at least three components per tier are reaching for "almost the next size". That is the empirical signal to add a tier.

## See also

- `../../docs/DESIGN-SYSTEM.md` §5.1 / §8.1 / §8.2 — token specifications.
- `../../AGENTS.md` §Source of truth — the rule that codifies this decision.
- `../patterns/design-system-token-first.md` — the procedure.
