---
name: wechat-tap-gesture
description: Diagnose and restructure flows that depend on WeChat user-gesture APIs (`shareFileMessage`, `setClipboardData`, `saveImageToPhotosAlbum`, `openDocument`, etc.). Use this skill whenever a flow "works on H5 but fails silently on a real weapp device", whenever an export/share/copy button shows up correctly in the UI but the API call never lands, or whenever a flow contains `await … await … await Taro.shareFileMessage(…)` chained behind the user tap. Do NOT use this skill for browser-only clipboard, native app share sheets, or features that do not depend on gesture-trusted WeChat APIs.
---

# WeChat TAP Gesture Trust Window

## When this skill applies

A flow works in the WeChat Developer Tools simulator but does nothing on a real device, and the suspect line is one of these (or a similar gesture-trusted) APIs:

- `Taro.shareFileMessage`
- `Taro.setClipboardData`
- `Taro.saveImageToPhotosAlbum`
- `Taro.saveFileToDisk` / `Taro.openDocument`
- Any platform call whose docs say "must be triggered by user gesture"

It also applies proactively whenever you are about to write code that does any of:

- `onClick={async () => { await prepare(); await Taro.shareFileMessage(...) }}`
- `await generateFile(); await Taro.setClipboardData(...)`
- Building a Markdown/JSON/PDF blob and then immediately calling a WeChat API.

If you see `await … await … await Taro.shareXxx` chained behind a tap, stop. The flow is broken on real devices even though the simulator accepts it.

## When to skip (do not run this skill)

- Browser-only clipboard / share (Web Share API, `navigator.clipboard.writeText`).
- Native iOS/Android share sheets outside weapp.
- Pure UI work with no platform API involved.

## Why this happens (the mechanism)

WeChat binds a "trust window" to the synchronous tap event. The browser model where any `await` then `await` then API call is fine does not apply on weapp. The runtime measures whether the platform call was triggered **synchronously** inside the original tap handler frame. By the time microtasks/Promises resolve, the trust window has closed.

The fix is not "make it faster." The fix is **inversion**: prepare everything **before** the button becomes enabled, and let the tap handler call the API as its only work.

## Workflow

### 1. Identify the gesture-trusted API

Find every call to `Taro.share*`, `Taro.setClipboard*`, `Taro.save*ToPhotos*`, `Taro.openDocument`. Each one of these is a candidate that must execute synchronously inside a tap handler.

### 2. Draw the two phases

Split the flow into:

- **Prepare phase** — file generation, data serialization, file writing, image encoding, can run async, can show a spinner.
- **Consume phase** — the gesture-trusted API call itself, must be a synchronous tail call of the tap handler.

```
[user opens dialog]  →  prepare (async, spinner)  →  [button enabled]
                                                            ↓
[user taps button]   →  Taro.shareFileMessage(path)  ← synchronous tail
```

### 3. Move preparation out of the tap handler

Rewrite the handler so the only work after `await`s is the platform call. If the path/content is already prepared, the handler should look like:

```tsx
const handleShare = useCallback(() => {
  if (!preparedPath) return                 // button should be disabled anyway
  void Taro.shareFileMessage({ filePath: preparedPath })
}, [preparedPath])
```

Notice: **no `await` before `Taro.shareFileMessage`**. `void` is intentional — it discards the returned Promise so it does not wrap the call in another microtask.

### 4. Make the disabled state a first-class UI state

The button must visibly and behaviorally reflect whether the prepared asset exists:

- `disabled={!preparedPath}` and a label that changes ("正在准备…" → "分享" / "复制" / "保存").
- Never let the user tap a button that cannot yet execute its API call.

### 5. Surface partial failure

`shareFileMessage` and friends can fail on a real device (storage, permission, file size). Do not swallow the rejection:

```tsx
try {
  await Taro.shareFileMessage({ filePath: preparedPath })
} catch (err) {
  // Show a Dialog that says "哪一步没成功 / 数据现在在哪"
  // Keep the raw errMsg in the dev log, do not paste it into user copy.
}
```

The user-facing message must answer two questions:

1. 哪一步没成功 (which step failed).
2. 数据现在在哪 (where the data lives now, so they can retry or use the in-app fallback).

### 6. Add a "trust window" comment

Leave a one-line comment above the gesture-trusted call so the next person does not refactor it back into the broken shape:

```tsx
// Trust window: must be the synchronous tail of a tap handler — no awaits above this.
void Taro.shareFileMessage({ filePath: preparedPath })
```

## Patterns and anti-patterns

### Anti-pattern — Web intuition on weapp

```tsx
// Broken on real devices, silently does nothing.
const onExport = async () => {
  setLoading(true)
  const md = buildMarkdown(records)
  const path = `${Taro.env.USER_DATA_PATH}/export.md`
  await Taro.writeFile({ filePath: path, data: md, encoding: 'utf8' })
  await Taro.shareFileMessage({ filePath: path })
  setLoading(false)
}
```

The two `await`s push `shareFileMessage` outside the trust window. The call returns a resolved Promise but WeChat does not actually open the chooser.

### Pattern — production-before-consumption

```tsx
const [preparedPath, setPreparedPath] = useState<string | null>(null)
const [preparing, setPreparing] = useState(false)

useEffect(() => {
  if (!dialogOpen) return
  let cancelled = false
  setPreparing(true)
  ;(async () => {
    const md = buildMarkdown(records)
    const path = `${Taro.env.USER_DATA_PATH}/export.md`
    await Taro.writeFile({ filePath: path, data: md, encoding: 'utf8' })
    if (!cancelled) setPreparedPath(path)
  })().finally(() => !cancelled && setPreparing(false))
  return () => { cancelled = true }
}, [dialogOpen, records])

const onShareTap = () => {
  if (!preparedPath) return                       // trust window: no awaits above
  void Taro.shareFileMessage({ filePath: preparedPath })
}

;<Button disabled={!preparedPath} onClick={onShareTap}>
  {preparing ? '正在准备…' : '分享'}
</Button>
```

### Pattern — same shape, different API (clipboard, save image)

The pattern is identical for `setClipboardData`, `saveImageToPhotosAlbum`, `openDocument`. The only thing that varies is what "prepared" means (path, base64, blob) and what the gesture call's argument shape is.

## Verification

You cannot verify this in the weapp simulator alone. The simulator is too forgiving. Verify on a real device:

1. `npm run build:weapp` → upload `dist/weapp` to WeChat DevTools → generate a preview QR → scan on a real phone.
3. Open the dialog → wait for the button to enable → tap it once. The chooser (or clipboard toast, or save dialog) must open on the first tap, not on the second.
4. Reject the chooser / deny the permission once. Verify the Dialog surfaces the failure clearly, not the raw `errMsg`.

If the simulator says it works but a real device does not, the trust window is the suspect — re-read §2 of this skill before believing the simulator.

## Output report

End the handoff with:

1. **Gesture API** — which call was at risk.
2. **Prepare phase** — what runs before the button enables, and the state machine that exposes it.
3. **Tap handler** — show the final handler shape; confirm zero `await`s above the gesture call.
4. **Disabled states** — what the button shows and how it transitions.
5. **Failure UX** — how partial failure is surfaced to the user.
6. **Verification** — which steps passed on a real device vs. simulator-only.