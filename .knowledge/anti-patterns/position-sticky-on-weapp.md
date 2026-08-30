# Anti-pattern: position-sticky-on-weapp

> **One-sentence summary.** `position: sticky; top: 0` for the page header works in H5 and in the WeChat Developer Tools simulator but scrolls with the content on a real weapp device, exposing the white/gray seam.

**Status:** active · since 2026-08
**Last observed:** 2026-08 (initial `PageShell` design — replaced by flex-column layout)
**Cost observed:** 1 redesign + every page's `<page>` element refactored

## What it looks like

```scss
.page-shell {
  min-height: 100vh;
}
.page-shell__header {
  position: sticky;
  top: 0;
  background: $ds-surface;
}
.page-shell__content {
  overflow-y: auto;
}
```

In the simulator and in H5 the header pins correctly. On a real weapp device, the header scrolls with the content. When the user scrolls past the top of the content, the white top block ends and the gray content block begins — but because the header is *also* moving down, the area behind the header exposes the window background. With `window.backgroundColor: '#F7F8F7'` (the initial value), the seam is visibly gray.

## Why it is wrong

`position: sticky` works in the browser and in the weapp devtools simulator, but the runtime behavior on real weapp devices does not honor it for elements whose parent does not have `overflow: hidden`. The `<page>` element created by Taro has its own scrolling behavior, which interferes with sticky.

The bug is invisible in the simulator. It is only visible on a real device. The verification loop on this project explicitly excludes "simulator says OK" as proof.

## What to do instead

Use the **flex-column-page-shell** pattern: pin the entire app via `position: fixed; top/bottom/left/right: 0`, lay it out as a `flex-direction: column`, set the header as `flex-shrink: 0`, and make the content the single `overflow-y: auto` flex sibling with `min-height: 0`. The header cannot leave its row because it is a sibling, not a sticky descendant.

```scss
.app-shell {
  position: fixed;
  top: 0; bottom: 0; left: 0; right: 0;
  display: flex;
  flex-direction: column;
}
.page-shell__header { flex-shrink: 0; }
.page-shell__content {
  flex: 1 1 auto;
  min-height: 0;
  overflow-y: auto;
}
```

And set `app.config.ts` `window.backgroundColor: '#FFFFFF'` so that any rubber-band pull that reveals the window background is still white.

## How it slipped in

The original page layout was based on a Web intuition (`position: sticky` is the standard CSS way to pin a header). The H5 build was correct. The simulator was correct. The first real-device preview was the first time anyone saw the bug.

## See also

- `../patterns/flex-column-page-shell.md` — the replacement.
- `../platforms/weapp-sticky-fails-on-real-device.md` — the platform fact.
- `../../docs/DESIGN-SYSTEM.md` §12 — page layout standard.
