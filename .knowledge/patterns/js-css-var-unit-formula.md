# Pattern: js-css-var-unit-formula

> **One-sentence summary.** Any CSS variable set from JavaScript must compute its unit as `${value * weappScale}${cssUnit}`; never write a bare `rpx` or `rem` literal in a JS-set CSS variable.

**Status:** active · since 2026-08
**Last validated:** 2026-08-30
**Used in:** `apps/client/src/components/PageShell/PageShell.tsx` (`--page-header-padding-top`)

## What it is

Two constants are defined once per file:

```ts
const isWeapp = !!process.env.TARO_ENV === 'weapp'
const weappScale = isWeapp ? 2 : 1
const cssUnit   = isWeapp ? 'rpx' : 'px'
```

Every JS-set CSS variable uses the formula:

```ts
style={{
  '--foo': `${value * weappScale}${cssUnit}`,
}}
```

SCSS source code (`px` literals) does **not** use this formula — Taro's `pxtransform` plugin rewrites those at build time (`px` → `rpx × 2` on weapp, `px` → `rem ÷ 20` on H5). The formula is only needed where the value comes from JavaScript.

## Why it works

`pxtransform` only inspects SCSS source files at build time. It never sees the strings you write into a React inline style or into `style={{ '--foo': '...' }}`. The browser then evaluates the CSS variable as-is:

- `'80rpx'` is a valid weapp unit and renders correctly there.
- The same `'80rpx'` is invalid in a browser. The browser silently drops the declaration → `padding-top: 0` → the header overlaps the content.

The formula works because the JS already knows which platform it is on via `process.env.TARO_ENV`, and emits a unit the runtime can actually consume.

## When to use

- Any inline style or `style={{ ... }}` that sets a CSS custom property.
- Any `Taro.createSelectorQuery` callback that writes a measured pixel value back into a CSS variable.
- Any dynamically computed size (`header height`, `safe-area inset`, status bar height) that needs to land in a CSS variable.

## When **not** to use

- Plain SCSS files — `pxtransform` handles them.
- Static values that never need platform-aware units.
- `width` / `height` set as numbers in JSX (Taro converts numbers to `rpx` / `px` correctly per platform).

## Example

```ts
const isWeapp = !!process.env.TARO_ENV === 'weapp'
const weappScale = isWeapp ? 2 : 1
const cssUnit   = isWeapp ? 'rpx' : 'px'

const statusBarHeight = Taro.getWindowInfo().statusBarHeight ?? 0
const headerPaddingTop = 20 + statusBarHeight

<View
  style={{
    '--page-header-padding-top': `${headerPaddingTop * weappScale}${cssUnit}`,
  }}
/>
```

The corresponding SCSS consumer reads the variable without re-multiplying:

```scss
.page-header {
  padding-top: var(--page-header-padding-top);
}
```

## Anti-shape (what to avoid)

```ts
// Looks fine on weapp, breaks on H5.
style={{ '--page-header-padding-top': '80rpx' }}

// Looks fine on H5, breaks on weapp scale.
style={{ '--page-header-padding-top': '40px' }}
```

## See also

- `../platforms/taro-pxtransform-scss-only.md` — the build-pipeline fact.
- `../anti-patterns/rpx-in-js-css-var.md` — the broken shape.
- `../../docs/DESIGN-SYSTEM.md` §13 — the unit pipeline specification.
- `../../AGENTS.md` §Multi-platform verification — verification matrix.
