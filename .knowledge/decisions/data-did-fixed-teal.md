# Decision: data-did-fixed-teal

> **One-sentence summary.** `--ds-data-did` is `#52A3A8` (Mood 4 "充实"), a fixed value independent of the active brand theme; it must not be aliased to `--ds-brand`.

**Decided:** 2026-08
**Decided by:** project owner
**Status:** active

## Context

The design system originally aliased `--ds-data-did` to `--ds-brand`. In the default `sage` theme this happened to render as a teal-green that read correctly as "Did". When the user added `sunset` (orange brand), `walnut` (brown brand), and `mulberry` (purple brand), Did lost its identity:

- In `sunset`, Did became orange and visually collided with `--ds-data-bad` (red-orange).
- In `mulberry`, Did became purple and was confused with `--ds-data-thinking` (blue-gray).
- In `walnut`, Did became brown and read as "no data".

Additionally, on charts, the legend dot for "Did" and the sparkline stroke diverged in color because the sparkline was reading from one token and the dot from another.

## Decision

`--ds-data-did` is pinned to `#52A3A8` — Mood 4 ("充实"). It does not change with theme. The same rule applies to `--ds-data-bad` (`#E06C62`, Mood 1 "低谷") and `--ds-data-thinking` (`#7B8E9D`, Mood 3 "平淡"). All three are off-limits for aliasing to brand or to each other.

## Why

Data colors carry **semantic** meaning that must be identifiable in every theme. Brand carries **identity** meaning that varies by theme. Aliasing them collapses two channels into one and lets the louder one (brand) win, leaving data ambiguous. Fixing the data layer as a fixed palette keeps the chart readable across all five themes.

## Consequences

- Adding a new theme no longer requires revisiting any chart or legend component.
- The Mood palette now has a structural role: it provides three of the data colors directly. Adding a new data category requires picking a Mood (or a new fixed token).
- Designers cannot "tune" Did color by tweaking brand; the data layer is locked.

## When to revisit

- A future requirement where data colors must themselves vary by theme (e.g. accessibility mode where contrast against the brand background must change). At that point, introduce *two* data tokens per category: a "default" and a "high-contrast on <brand>" variant, and route by computed contrast.
- A user-data-driven need to recolor Did (e.g. category-specific). Introduce explicit user preferences, not a theme alias.

## See also

- `../anti-patterns/data-color-aliased-to-brand.md` — the broken aliasing shape.
- `../../docs/DESIGN-SYSTEM.md` §2.3.1 — data color specification.
