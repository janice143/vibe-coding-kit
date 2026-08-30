# Anti-pattern: data-color-aliased-to-brand

> **One-sentence summary.** Aliasing any of the three data colors (`--ds-data-did`, `--ds-data-bad`, `--ds-data-thinking`) to `--ds-brand` causes them to lose their identity in non-sage themes and collide with semantic "brand" surfaces.

**Status:** active · since 2026-08
**Last observed:** 2026-08 (donut Did arc turned orange in sunset theme; sparkline stroke diverged from legend dot color)
**Cost observed:** 4 chart-related regressions; 1 round-trip to fix `--ds-data-did` specifically

## What it looks like

```scss
// Anti-pattern: data color follows brand theme.
--ds-data-did: var(--ds-brand);          // ← green in sage, orange in sunset, brown in walnut
--ds-data-thinking: var(--ds-brand-soft); // ← too low contrast, ambiguous
```

The intent was reasonable: "make the data layer feel cohesive with the brand". The result was that:

- In `sunset`, Did became orange and collided visually with `--ds-data-bad` (red-orange).
- In `mulberry`, Did became purple and read as "thinking + did merged".
- Across all themes, the legend dot for "Did" looked like a brand surface, not a data signal.

## Why it is wrong

Data colors carry **semantic** meaning. The user must be able to identify "this arc is Did" independently of the theme. Brand carries **identity** meaning. Aliasing them collapses two meanings into one signal, and the meaning that survives is whichever one is louder — usually brand, leaving the data ambiguous.

The fix: data colors are fixed values pinned to the Mood palette. `--ds-data-did` is `#52A3A8` (mood-4, "充实"), `--ds-data-bad` is `#E06C62` (mood-1, "低谷"), `--ds-data-thinking` is `#7B8E9D` (mood-3, "平淡"). All three are off-limits for aliasing.

## What to do instead

```scss
// Correct: fixed values, no aliasing.
--ds-data-did: #52A3A8;        // mood-4
--ds-data-bad: #E06C62;         // mood-1
--ds-data-thinking: #7B8E9D;    // mood-3
```

When introducing a new data category, pick a fixed color from the Mood palette (or a new fixed token) — never alias to brand, never alias to a soft variant, never alias to another data color.

## How it slipped in

The first version of the design system treated `--ds-data-did` as a brand-relative accent. The semantic confusion was not apparent in the default `sage` theme (where brand was already `#4A7C59` and the data aliasing happened to look fine). It surfaced when the user added the `sunset` and `mulberry` themes.

## See also

- `../decisions/data-did-fixed-teal.md` — the decision that codifies the fix.
- `../../docs/DESIGN-SYSTEM.md` §2.3.1 — data color specification.
- `../../AGENTS.md` §Source of truth — design-system rule.
