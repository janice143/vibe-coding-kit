# Platform note: weapp-sticky-fails-on-real-device

> **One-sentence summary.** `position: sticky; top: 0` for the page header pins correctly in H5 and in the WeChat Developer Tools simulator but scrolls with the content on a real weapp device.

**Platforms affected:** weapp (real device only; simulator is unreliable)
**Verified on:** 2026-08
**Verified with:** iPhone 12 (iOS 17), WeChat 8.0.45, page-header on `home` and `trend` pages

## The fact

`<View style={{ position: 'sticky', top: 0 }}>` does not pin on a real weapp device. The header scrolls with the content. In the WeChat Developer Tools simulator the same code pins correctly, and on H5 it pins correctly. Real-device behavior diverges from both.

The most visible symptom is a seam appearing between the white top block (status bar + PageHeader) and the gray content block. If `app.config.ts` `window.backgroundColor` is also `#F7F8F7` (the initial guess), the seam shows that color too.

## Why intuition gets it wrong

Sticky positioning is a well-supported CSS feature in every modern browser. The H5 build is correct. The simulator is correct. The fact that real weapp devices diverge from both is not in the official Taro docs and is not in any common Stack Overflow answer. It is only visible by running on a phone.

## How to verify

1. Implement a sticky page header.
2. Open the page in H5 — verify it pins.
3. Open in the WeChat Developer Tools simulator — verify it pins.
4. Build the weapp, generate a preview QR, scan on a real device — observe that the header scrolls.

Step 4 is the only step that surfaces this bug. The simulator must be considered unreliable for this behavior.

## What depends on this

- The layout of every page in the app (home / trend / records / mine / record-document).
- The design-system page layout standard (§12) — the flex-column pattern exists specifically to bypass this bug.

## Source

Taro GitHub issue tracker: multiple reports of `position: sticky` not pinning inside `<page>`. WeChat Mini Program runtime limitation around `<page>` element's own scrolling container. Internal investigation August 2026.

## See also

- `../patterns/flex-column-page-shell.md` — the layout that replaces sticky.
- `../anti-patterns/position-sticky-on-weapp.md` — the broken shape.
- `../../docs/DESIGN-SYSTEM.md` §12.1 — page-shell specification.
