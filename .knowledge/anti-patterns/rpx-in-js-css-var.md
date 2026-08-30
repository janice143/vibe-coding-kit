# Anti-pattern: rpx-in-js-css-var

> **One-sentence summary.** Writing a bare `rpx` (or `rem`) literal into a JS-set CSS variable silently breaks the platform that does not understand the unit.

**Status:** active · since 2026-08
**Last observed:** 2026-08 (`PageShell` initial draft — header overlapped content on H5)
**Cost observed:** ~1 hour + a content-overlap bug caught only on H5 preview

## What it looks like

```tsx
<View
  style={{
    '--page-header-padding-top': `80rpx`,
    '--page-header-clearance':  `60rpx`,
  }}
/>
```

The build emits these values into the weapp `.wxss` unchanged. Weapp renders correctly because it understands `rpx`. The browser, however, does not — it silently drops the entire `padding-top: var(--page-header-padding-top)` declaration. The header collapses to `0` and overlaps the content.

## Why it is wrong

Taro's `pxtransform` plugin rewrites SCSS source at build time (`px` → `rpx × 2` on weapp, `px` → `rem ÷ 20` on H5). It does **not** touch JavaScript inline styles or CSS variables. The value you write in JS reaches the runtime verbatim. The runtime is platform-aware (weapp vs. browser) but it does not transform unknown ones; it drops them.

## What to do instead

Use the **`js-css-var-unit-formula`**: `${value * weappScale}${cssUnit}` where `weappScale = 2` on weapp and `1` elsewhere, and `cssUnit = 'rpx'` on weapp and `'px'` on H5.

```tsx
const isWeapp = !!process.env.TARO_ENV === 'weapp'
const weappScale = isWeapp ? 2 : 1
const cssUnit   = isWeapp ? 'rpx' : 'px'

<View
  style={{
    '--page-header-padding-top': `${80 * weappScale}${cssUnit}`,
  }}
/>
```

There is no third option. JS-set CSS variables must always be unit-aware.

## How it slipped in

The author was thinking about weapp (the primary shipping target) and used `rpx` because "that's what Taro uses". The first platform check passed (weapp OK), the second did not (H5 broken). The bug was caught only because both platforms are part of the verification matrix in `AGENTS.md`.

## See also

- `../patterns/js-css-var-unit-formula.md` — the replacement.
- `../platforms/taro-pxtransform-scss-only.md` — the build-pipeline fact.
- `../../docs/DESIGN-SYSTEM.md` §13 — unit pipeline specification.
- `scripts/smoke-platform-css.mjs` — local assertion that catches `rpx` in `dist/h5` output.
