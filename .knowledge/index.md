# Knowledge Base Index

One line per entry. Entries are grouped by topic; the order within a group is "most-referenced first", not chronological.

Keep this file under ~80 lines. It is the table of contents — not the content.

> The entries below are **worked examples** from the project that produced this kit (DayRetro / 随心迹). They show the format and the kind of fact that belongs here. Replace or delete them as you start using the kit in your own project — see `README.md` §About the example entries shipped with this kit.

## Patterns

- [production-before-consumption](patterns/production-before-consumption.md) — split gesture-API flows into prepare (async) + consume (sync tap tail).
- [flex-column-page-shell](patterns/flex-column-page-shell.md) — `position: fixed` + flex-column + `min-height: 0` for pinned header on weapp.
- [hairline-pseudo-element](patterns/hairline-pseudo-element.md) — 200% × 200% pseudo + `scale(.5)` for true device-pixel hairlines.
- [js-css-var-unit-formula](patterns/js-css-var-unit-formula.md) — `${value * weappScale}${cssUnit}` for any JS-set CSS variable.
- [design-system-token-first](patterns/design-system-token-first.md) — read `docs/DESIGN-SYSTEM.md` first; extend tokens before code.

## Anti-patterns

- [await-chain-sharefile](anti-patterns/await-chain-sharefile.md) — `await … await Taro.shareFileMessage(…)` silently no-ops on real weapp.
- [rpx-in-js-css-var](anti-patterns/rpx-in-js-css-var.md) — bare `rpx` in a JS-set CSS variable is silently dropped on H5.
- [position-sticky-on-weapp](anti-patterns/position-sticky-on-weapp.md) — `position: sticky` works in simulator, scrolls with content on real weapp.
- [empty-text-as-visual-element](anti-patterns/empty-text-as-visual-element.md) — empty `<Text>` collapses to `0 × 0` on weapp; use `<View>`.
- [data-color-aliased-to-brand](anti-patterns/data-color-aliased-to-brand.md) — `--ds-data-did` aliased to `--ds-brand` loses identity in non-sage themes.

## Platforms (weapp / H5 / Taro)

- [weapp-tap-trust-window](platforms/weapp-tap-trust-window.md) — gesture APIs need synchronous tap handler tail; awaits close the window.
- [weapp-empty-text-zero-size](platforms/weapp-empty-text-zero-size.md) — `<Text>` with no children → `0 × 0` on weapp.
- [weapp-sticky-fails-on-real-device](platforms/weapp-sticky-fails-on-real-device.md) — `position: sticky` only the simulator accepts; real device disagrees.
- [taro-pxtransform-scss-only](platforms/taro-pxtransform-scss-only.md) — `pxtransform` rewrites SCSS source only; JS-set CSS variables and inline styles are not touched.

## Decisions

- [local-first-storage](decisions/local-first-storage.md) — `wx.storage` / `localStorage` only in v1; no API / DB / auth / cloud sync.
- [data-did-fixed-teal](decisions/data-did-fixed-teal.md) — `--ds-data-did` is `#52A3A8` (Mood 4), independent of brand theme.
- [minimum-content-bg-f4f6f7](decisions/minimum-content-bg-f4f6f7.md) — content area background locked to `#F4F6F7` across all five pages.
- [four-radius-three-shadow-four-type](decisions/four-radius-three-shadow-four-type.md) — visual system constrained to 4 / 3 / 4 token tiers.

## AI coding

- [implementation-history-in-ui](ai-coding/implementation-history-in-ui.md) — agents leak dev rationale into user-facing copy ("新版", "我们已").
- [local-optimum-component-creation](ai-coding/local-optimum-component-creation.md) — agents create page-local components instead of searching for prior art.
- [superpowers-process-inflation](ai-coding/superpowers-process-inflation.md) — heavyweight workflows inflate process for small tasks.

---

## How to add an entry

1. Read `.knowledge/README.md` to confirm the entry belongs here (vs. `AGENTS.md` or a `SKILL.md`).
2. Copy the matching template from `_templates/` to the correct subdirectory.
3. Fill in the template fields concretely — no placeholders.
4. Add a one-line entry under the right heading above.
5. Reference the entry from `AGENTS.md` or a skill if it changes default behavior.
