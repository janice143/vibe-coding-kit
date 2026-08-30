# Platform note: weapp-tap-trust-window

> **One-sentence summary.** WeChat Mini Program runs gesture-trusted platform APIs only if they are the synchronous tail of a tap handler; any `await` between the tap and the API closes the trust window.

**Platforms affected:** weapp only (H5 / iOS / Android are unaffected)
**Verified on:** 2026-08
**Verified with:** WeChat iOS client ~8.0.45, real iPhone 12 (DPR 3), `Taro.shareFileMessage`, `Taro.setClipboardData`

## The fact

The runtime measures whether a platform API call originated from a synchronous frame attached to the user gesture. The relevant APIs (`Taro.shareFileMessage`, `Taro.setClipboardData`, `Taro.saveImageToPhotosAlbum`, `Taro.openDocument`, `Taro.saveFileToDisk`) all require this. If the call sits behind one or more `await`s, the Promise still resolves successfully — the runtime just does not act on it. There is no error, no warning, no log line that says "trust window closed". The function call appears to succeed but no chooser opens, no clipboard toast shows, no save dialog appears.

## Why intuition gets it wrong

The browser model treats `await` as syntactic: any handler can `await` anything, the runtime will resume the microtask. Web Share API, `navigator.clipboard.writeText`, etc. all follow this rule. The weapp runtime uses a different model — it attaches a "this is a real gesture" flag to the original tap frame and discards it on the first microtask boundary. The two models are not reconcilable; you have to write code that fits the weapp model.

## How to verify

The simulator **lies**. It accepts the broken shape and resolves the Promise without acting. The only reliable check is a real device:

1. `npm run build:weapp` from `fullstack/`.
2. Upload `apps/client/dist/weapp` to WeChat Developer Tools.
3. Generate a preview QR and scan on a real device.
4. Tap the export / share / copy button once. The chooser (or clipboard toast, or save dialog) must open on the first tap, not the second.

If the simulator says it works, you have verified nothing about real-device behavior.

## What depends on this

- Every flow that ends in `Taro.shareFileMessage` — file paths and Markdown content must be prepared before the tap.
- Every flow that ends in `Taro.setClipboardData` — the text must already be in state.
- Every flow that ends in `Taro.saveImageToPhotosAlbum` — the image path must already be written.
- The `prototype-to-taro` skill's self-check matrix.
- The `wechat-tap-gesture` skill.

## Source

WeChat Mini Program docs for `shareFileMessage`, `setClipboardData`, `saveImageToPhotosAlbum`: all explicitly say "must be triggered by user gesture". The trust window is the runtime's interpretation of "triggered". Internal bug observed in `apps/client/src/pages/record-document/` initial export flow, August 2026.

## See also

- `../patterns/production-before-consumption.md` — the code shape that works.
- `../anti-patterns/await-chain-sharefile.md` — the code shape that does not.
- `../.claude/skills/wechat-tap-gesture/SKILL.md` — the full procedure.
