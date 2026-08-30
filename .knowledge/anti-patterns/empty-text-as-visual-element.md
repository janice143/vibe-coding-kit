# Anti-pattern: empty-text-as-visual-element

> **One-sentence summary.** Using `<Text>` as a wrapper for purely visual decoration (legend dot, inline icon, color swatch) collapses the element to `0×0` on weapp.

**Status:** active · since 2026-08
**Last observed:** 2026-08 (trend chart legend dots invisible in week view; mood icons overflowed parent in year view)
**Cost observed:** 3 visual bugs across charts and pills before the rule was codified

## What it looks like

```tsx
<Text
  className="legend-dot"
  style={{ background: dsDataDid }}
/>
```

```tsx
<Text className="mood-icon" style={{ fontSize: 22 }} />
```

The element renders fine in H5 (where `<span>` always reserves its own size) and looks fine in the weapp devtools simulator. On a real weapp device, an empty `<Text>` (no children) is collapsed by the runtime to `0 × 0`. The element occupies no space and is invisible.

## Why it is wrong

The weapp runtime treats `<Text>` as a *text-bearing* element. An empty `<Text>` is not a layout primitive — it is a missing text node. The runtime optimizes for the text case and emits `0 × 0`. `<View>` (the layout primitive) does not have this behavior; empty `<View>` always reserves its box.

The H5 build (where `<Text>` becomes `<span>`) does not exhibit this because the browser preserves span box dimensions regardless of content.

## What to do instead

- For visual-only elements (dots, swatches, decorative pills, icon boxes): always `<View>`.
- `<Text>` is for text-bearing content only. The `AGENTS.md` and `prototype-to-taro` skill both state this.
- For inline icons that ship as text glyphs (Lucide font fallback), use a `<View>` wrapper around the text, or use a real `<Image>` from `theme/icon-paths.ts`.

```tsx
<View className="legend-dot" style={{ background: dsDataDid }} />
```

## How it slipped in

The shape came from the H5 prototype, where the dot was `<span>` and worked. The Taro converter preserved the element type ("it's text-adjacent, so `<Text>`") without considering that the element had no text content.

## See also

- `../platforms/weapp-empty-text-zero-size.md` — the platform fact.
- `../../.claude/skills/prototype-to-taro/SKILL.md` §6 — platform self-check item: "No empty `<Text>` used as a visual element".
