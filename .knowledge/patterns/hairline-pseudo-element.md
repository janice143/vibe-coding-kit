# Pattern: hairline-pseudo-element

> **One-sentence summary.** Draw 1px borders on a 200% × 200% pseudo-element and `transform: scale(.5)` back to a true device pixel; never use `border: 1px solid` for control outlines on weapp.

**Status:** active · since 2026-08
**Last validated:** 2026-08-30
**Used in:** every input / textarea / secondary button / separator — see `apps/client/src/styles/_hairlines.scss`

## What it is

The hairline mixins in `_hairlines.scss`:

- `@include hairline.box($color, $radius)` — full outline. Host gets `position: relative; border: 0;`. Pseudo-element `::after` covers `200% × 200%`, draws `1px` border, then `transform: scale(.5)` from the top-left. `box-sizing: border-box` and `pointer-events: none` are mandatory on the pseudo.
- `@include hairline.box-color($color)` — recolour the existing `box()` outline for hover / focus / selected. **Do not** also re-declare `border-color` on the host; the border lives on the pseudo.
- `@include hairline.top($color, $inset)` / `@include hairline.bottom($color, $inset)` — single-edge separators. `top()` uses `::before`, `bottom()` uses `::after`, with absolute positioning and `scaleY(.5)`.

`$radius` mirrors the host's own border-radius. The mixin already doubles the pixel value before scaling so the rendered corner matches the host. Percentage radius (`$ds-radius-pill`) is a deliberate exception: it is already relative to the pseudo-element's own box and must be passed through unchanged.

## Why it works

Taro's `pxtransform` plugin rewrites SCSS `1px` to `2rpx` on weapp. `2rpx` is one full physical pixel on a DPR-2 screen — it is a normal stroke, not a hairline. Drawing the line at `1px` logical size on a 2× scaled box, then halving, lands at one device pixel. H5 also benefits because `box-shadow` based hairlines do not require color management on a pseudo-element.

## When to use

- Input / Textarea / secondary button outlines.
- Selected-state outlines (segment control selected thumb border).
- Inner row separators inside cards.
- Chart strokes (data points, legend dots).

## When **not** to use

- Native `<Input>` / `<Textarea>` on weapp — pseudo-elements do not render reliably inside them. The hairline goes on the **wrapper** element, not the form control itself. (`docs/DESIGN-SYSTEM.md` §8.4 exception.)
- Native `<Button>` on weapp — its `::after` is occupied by the platform's own border. Same workaround: wrapper.
- Where the boundary is genuinely meant to be visible at 1 CSS px (e.g. a chart stroke that is part of the figure, not a container outline).
- Where `::before` or `::after` is already used by another rule — restructure before reaching for a workaround.

## Example

```scss
.input-field {
  position: relative;
  border: 0;                                  // clear inherited borders
  border-radius: $ds-radius-md;
  background: $ds-surface;
  @include hairline.box($ds-border, $ds-radius-md);
}
.input-field:focus-within {
  @include hairline.box-color($ds-brand);
}
.record-card__divider {
  position: relative;
  @include hairline.bottom($ds-border, 0);    // inset 0 = flush to the wrapper edges
}
```

## Anti-shape (what to avoid)

```scss
.input-field { border: 1px solid $ds-border; }   // reads thicker on real weapp
.input-field:focus { border-color: $ds-brand; }  // does nothing — border is on a pseudo
```

## See also

- `../platforms/taro-pxtransform-scss-only.md` — the build pipeline that motivates this pattern.
- `../../docs/DESIGN-SYSTEM.md` §8.4 — the mixin specification.
- `../../AGENTS.md` §Hairlines — rule form of this pattern.
