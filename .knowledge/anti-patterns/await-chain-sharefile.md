# Anti-pattern: await-chain-sharefile

> **One-sentence summary.** A `Taro.shareFileMessage` (or any gesture-trusted API) call that sits behind one or more `await`s in a tap handler silently does nothing on real weapp devices.

**Status:** active · since 2026-08
**Last observed:** 2026-08 (initial record-document export flow)
**Cost observed:** ~3 hours of "works in simulator, does nothing on phone" debugging before recognizing the trust-window mechanism

## What it looks like

```tsx
const onExport = async () => {
  setLoading(true)
  const md = buildMarkdown(records)
  const path = `${Taro.env.USER_DATA_PATH}/export.md`
  await Taro.writeFile({ filePath: path, data: md, encoding: 'utf8' })
  await Taro.shareFileMessage({ filePath: path })   // ← trust window already closed
  setLoading(false)
}

<Button onClick={onExport}>分享</Button>
```

The handler is `async`. Two `await`s land in microtasks before `shareFileMessage` runs. The function resolves with no error, the dev tools log shows "ok", but no chooser opens on a real device.

## Why it is wrong

WeChat binds a "trust window" to the synchronous tap event frame. By the time microtasks resolve, the runtime has decided this is no longer a user gesture. The platform call's *Promise* still resolves successfully (which is what makes the bug silent) — the platform just does not act on it.

The fix is not to make it faster. The fix is to invert the order: prepare before the tap, then let the tap call the API as its synchronous tail.

## What to do instead

Use the **production-before-consumption** pattern:

1. Prepare the file in a `useEffect` that runs when the dialog opens.
2. Store the path in state (`preparedPath`).
3. Disable the button until `preparedPath` is set.
4. The tap handler is synchronous and has zero `await`s above `shareFileMessage`.

```tsx
const onShareTap = () => {
  if (!preparedPath) return
  void Taro.shareFileMessage({ filePath: preparedPath })
}
```

## How it slipped in

The shape is the natural Web intuition: "user clicks → do work → call API". It also passes every visible check — typecheck, build, simulator click — because the simulator does not enforce the trust window. The bug only surfaces on a real device preview, which is the last step of the verification loop.

## See also

- `../patterns/production-before-consumption.md` — the replacement.
- `../platforms/weapp-tap-trust-window.md` — the platform fact.
- `../.claude/skills/wechat-tap-gesture/SKILL.md` — the full procedure.
