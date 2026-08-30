# Pattern: flex-column-page-shell

> **One-sentence summary.** Use `position: fixed` + flex-column + `min-height: 0` to make the page header truly pinned and the content area the single scroll surface; never use `position: sticky` for the top header on weapp.

**Status:** active · since 2026-08
**Last validated:** 2026-08-30
**Used in:** every page (home, trend, records, mine, record-document) — see `apps/client/src/components/PageShell/`

## What it is

Three layers form the page:

- `.app-shell`: `position: fixed; top: 0; bottom: 0; left: 0; right: 0; display: flex; flex-direction: column;` — pins the entire app to the viewport; `margin: 0 auto` centers content on wider viewports.
- `.page-shell__header`: `flex-shrink: 0` — always renders at the top, never scrolls.
- `.page-shell__content`: `flex: 1 1 auto; min-height: 0; overflow-y: auto; -webkit-overflow-scrolling: touch;` — the only scroll surface. `min-height: 0` is the critical line; without it the flex item's default `min-height: auto` prevents `overflow-y: auto` from ever engaging.

`<page>` (Taro's outer element) carries `height: 100%; overflow: hidden; background: $ds-surface;` so the page itself does not scroll.

## Why it works

`position: sticky; top: 0` works in browsers and in the WeChat Developer Tools simulator. On a real weapp device the header scrolls with the content, exposing the seam between the white top block and the gray content area. The flex-column layout makes that bug impossible — the header is a flex sibling of the content, not a sticky descendant, so it cannot leave its allocated row.

## When to use

- Every new page in `apps/client/src/pages/<name>/`.
- Any refactor that touches `PageShell`, `PageHeader`, or the page root.
- Any page that previously used `position: sticky` for the top bar.

## When **not** to use

- Components that intentionally scroll within the page (e.g. the trend chart legend list) — those get their own `overflow-y: auto` block, not a sticky page header.
- The weapp simulator-only build, where sticky still works — never trust that.

## Example

```scss
.app-shell {
  position: fixed;
  top: 0; bottom: 0; left: 0; right: 0;
  display: flex;
  flex-direction: column;
  margin: 0 auto;
  background: $ds-surface;
}

.page-shell__header { flex-shrink: 0; }

.page-shell__content {
  flex: 1 1 auto;
  min-height: 0;                              // required for overflow-y to work
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  background: $ds-content-bg;
}
```

```tsx
<View className="app-shell">
  <View className="page-shell__header">
    <PageHeader title="…" subtitle="简单记录 · 持续成长" right={…} />
  </View>
  <View className="page-shell__content">
    {/* the only scroll surface */}
  </View>
</View>
```

## Anti-shape (what to avoid)

```scss
.page-shell__header { position: sticky; top: 0; }   // works in simulator, breaks on real weapp
.page-shell__content { overflow-y: auto; }            // missing min-height: 0 → overflow never engages
.app-shell { min-height: 100vh; }                     // height: 100vh, not min-height
```

And: setting `app.config.ts` `window.backgroundColor` to anything other than `#FFFFFF`. Rubber-band pull on real weapp briefly reveals the window background — if it is gray, the seam shows.

## See also

- `../../docs/DESIGN-SYSTEM.md` §12 — page layout standard (Top block + Content block + bottom nav).
- `../platforms/weapp-sticky-fails-on-real-device.md` — the platform fact that motivates this pattern.
- `../anti-patterns/position-sticky-on-weapp.md` — the broken shape.
- `../../AGENTS.md` §Multi-platform verification — verification matrix.
