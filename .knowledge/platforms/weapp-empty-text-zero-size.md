# Platform note: weapp-empty-text-zero-size

> **One-sentence summary.** A `<Text>` element with no text content is collapsed by the weapp runtime to `0 × 0`; use `<View>` for visual-only elements.

**Platforms affected:** weapp only (H5 preserves `<span>` box size)
**Verified on:** 2026-08
**Verified with:** trend chart legend dots, mood pill icons, donut legend rows — all three classes of element

## The fact

A Taro `<Text>` element whose children are empty (no characters, no glyphs) renders with `0 × 0` dimensions on weapp. The element exists in the DOM, accepts styles, but takes no layout space. This affects:

- Legend dots drawn with `<Text style={{ width: 8, height: 8 }} />`.
- Mood icons rendered as font-glyph fallbacks inside `<Text style={{ fontSize: 22 }} />`.
- Color swatches rendered as empty `<Text>`.

## Why intuition gets it wrong

In the browser, an empty `<span>` reserves its inline box even without content. H5 build of the same code looks correct because Taro compiles `<Text>` → `<span>`. The bug only appears on weapp.

## How to verify

1. Render the element on both H5 and weapp.
2. Open devtools on both, inspect the element's box.
3. H5 will report the styled dimensions; weapp will report `0 × 0`.

The simulator is reliable for this one — the runtime behavior matches the device.

## What depends on this

- All chart legend dots (trend, donut, overview).
- All inline decorative icons that ship as text glyphs.
- All color swatches that are not real `<Image>`.

## Source

WeChat Mini Program `<Text>` component doc: "Text is a text-bearing component. Use View for layout." Internal bugs observed in `apps/client/src/components/LineChart/`, `apps/client/src/components/Donut/`, and `apps/client/src/pages/record-document/`.

## See also

- `../anti-patterns/empty-text-as-visual-element.md` — the broken shape.
- `../../.claude/skills/prototype-to-taro/SKILL.md` §6 — self-check item.
