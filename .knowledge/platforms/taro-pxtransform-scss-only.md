# Platform note: taro-pxtransform-scss-only

> **One-sentence summary.** Taro's `pxtransform` plugin rewrites SCSS source at build time (`px` → `rpx × 2` on weapp, `px` → `rem ÷ 20` on H5) but never touches JavaScript-set CSS variables, JSX inline styles, or runtime-computed values.

**Platforms affected:** weapp · h5 · taro build
**Verified on:** 2026-08
**Verified with:** Taro 4.x, `pxtransform` plugin, comparison of `dist/h5/*.css` vs source SCSS, comparison of `dist/weapp/*.wxss` vs source SCSS

## The fact

Taro's `pxtransform` is a SCSS source transformer. It rewrites every `Npx` in your `.scss` / `.module.scss` files to `Nrpx` on weapp (×2) or `Nrem` on H5 (÷20). It does not see:

- JavaScript inline styles: `style={{ width: 44 }}` — emitted as-is.
- JS-set CSS variables: `style={{ '--foo': '80rpx' }}` — emitted as-is.
- Runtime-computed values: anything written in `useEffect`, `useState`, callbacks, or derived from props.
- `Taro.createSelectorQuery` measurements that get fed back into CSS variables.
- The `:root` of an HTML showcase file if it is not compiled by Taro.

This is the source of the most common H5/weapp split in this codebase. A value that "looks right" on weapp because the SCSS file uses `px` (which becomes `rpx`) is broken on H5 if the same value is computed in JS as `rpx` directly — H5 does not understand `rpx`.

## Why intuition gets it wrong

The mental model "Taro handles units for me" is correct for SCSS and incorrect everywhere else. Most of the time this distinction does not matter because the codebase uses SCSS for almost everything. The moment a value has to be runtime-computed (status bar height, measured element size, theme-aware token), the developer must take responsibility for the unit.

## How to verify

1. Write the same value in three places: SCSS source, JS inline style, JS-set CSS variable.
2. Build weapp, build H5.
3. `grep -r '80rpx' apps/client/dist/h5/` — should return zero results.
4. `grep -r '80rpx' apps/client/dist/weapp/` — should return results from the SCSS source only.
5. Manually inspect any JS-set values that landed in the CSS output. If they contain `rpx` on H5, the unit is wrong.

A local script exists for this: `fullstack/scripts/smoke-platform-css.mjs` (run with `npm run smoke:platform` from `fullstack/`). It asserts `rpx` is absent from the H5 output.

## What depends on this

- Status bar height injection in `PageShell`.
- Safe-area-inset-bottom on `RecordActionDock`.
- Any dynamically measured size fed into a CSS variable.
- Any value derived from a prop at runtime.

## Source

Taro docs on `pxtransform`: scope is explicitly "SCSS / Less / Stylus source files". Internal observation August 2026 — initial `PageShell` had `--page-header-padding-top: '80rpx'` set via JS, which broke H5.

## See also

- `../patterns/js-css-var-unit-formula.md` — the formula.
- `../anti-patterns/rpx-in-js-css-var.md` — the broken shape.
- `../../docs/DESIGN-SYSTEM.md` §13 — unit pipeline specification.
- `../../AGENTS.md` §Multi-platform verification.
