# Pattern: production-before-consumption

> **One-sentence summary.** Split a WeChat user-gesture flow into a prepare phase (async, inside the dialog) and a consume phase (synchronous tail of the tap handler); the button only enables after preparation is done.

**Status:** active · since 2026-08
**Last validated:** 2026-08-30
**Used in:** `apps/client/src/pages/record-document/` (export Markdown share flow)

## What it is

A flow whose final step is a WeChat gesture-trusted API (`shareFileMessage`, `setClipboardData`, `saveImageToPhotosAlbum`, `openDocument`) is split:

1. **Prepare phase** — runs as the dialog/effect opens. Generates the file, writes the path, serializes the payload. Updates a `preparedPath` state when done. Spinner / "正在准备…" label while running.
2. **Consume phase** — the user's tap handler. No `await`s above the API call. The call is the synchronous tail of the handler.

The button is `disabled={!preparedPath}` until the prepare phase completes. Failure of either phase is surfaced in user copy that answers "哪一步没成功 / 数据现在在哪".

## Why it works

WeChat binds a "trust window" to the synchronous tap event. Any `await` between the tap and the API call closes the window — the API call resolves successfully but never actually opens the chooser. The fix is not speed; it is inversion. The prepare phase decouples asset generation from the gesture event entirely.

## When to use

- Any feature that ends in `Taro.share*`, `Taro.setClipboard*`, `Taro.save*ToPhotos*`, `Taro.openDocument`.
- Any export / backup / share button that builds a Markdown / JSON / image before invoking the API.
- Any flow that looks like `await prepare(); await Taro.x()` chained behind a tap.

## When **not** to use

- Browser-only clipboard / share (Web Share API, `navigator.clipboard.writeText`).
- Native iOS/Android share sheets outside weapp.
- API calls that are not gesture-trusted (they can stay chained behind `await`).

## Example

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
  if (!preparedPath) return                    // trust window: no awaits above
  void Taro.shareFileMessage({ filePath: preparedPath })
}

<Button disabled={!preparedPath} onClick={onShareTap}>
  {preparing ? '正在准备…' : '分享'}
</Button>
```

Notice: `void` discards the returned Promise so it does not wrap the call in another microtask, and the handler has **zero** `await`s above `shareFileMessage`.

## Anti-shape (what to avoid)

```tsx
// Broken on real devices — silently does nothing.
const onExport = async () => {
  setLoading(true)
  const md = buildMarkdown(records)
  await Taro.writeFile({ filePath, data: md, encoding: 'utf8' })
  await Taro.shareFileMessage({ filePath })     // trust window closed by the awaits above
  setLoading(false)
}
```

## See also

- `../.claude/skills/wechat-tap-gesture/SKILL.md` — full workflow + verification.
- `../anti-patterns/await-chain-sharefile.md` — the broken shape.
- `../platforms/weapp-tap-trust-window.md` — the platform fact that motivates this pattern.
- `../../AGENTS.md` §Source of truth — design-system rule that constrains the dialog/button visuals.
