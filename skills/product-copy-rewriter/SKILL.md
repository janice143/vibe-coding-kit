---
name: product-copy-rewriter
description: Strip "implementation history" out of any user-facing string before it ships — dialog titles, dialog bodies, empty states, onboarding steps, button labels, tooltips, release notes shown to users, in-app banners, success / error messages. Use this skill whenever an AI proposes UI copy that references "we used to…", "now we…", "previously…", "we changed this because…", "as you may have noticed…", "improved / redesigned / refactored", or any equivalent phrasing that betrays internal decision history. Do NOT use this skill for internal handoff reports, developer-facing commit messages, CHANGELOG entries, or any string the user does not read.
---

# Product Copy Rewriter

## When this skill applies

Any string that lands in front of a user is in scope:

- Dialog title / body / confirm button / cancel button.
- Empty state headline / subhead / CTA.
- Onboarding step text.
- In-app banner / notice / toast.
- Tooltip / hint.
- Release notes shown inside the product.
- AI-generated analysis output that the user reads.

The signal is one of these patterns appearing in copy:

- "现在 / 新版 / 已升级 / 已重构 / 已优化" — anything implying there was an old version.
- "我们 / 我们已经 / 我们决定" — first-person plural that exposes a decision-maker.
- "为了 / 因此 / 所以" — explaining a rationale the user did not ask for.
- "以前是 / 之前 / 之前是 / 原来的 / 旧的" — referencing prior state.
- "你可能已经注意到 / 不难发现" — admitting that the product has changed.
- "更轻松 / 更智能 / 更好 / 更强大" — superlatives that only make sense compared to something.
- An English equivalent of any of the above.

If the user reads the string, run this skill. If only the developer reads it, do not.

## When to skip (do not run this skill)

- Internal handoff reports and developer-facing commit messages.
- `CHANGELOG.md`, `AGENTS.md`, `docs/`, code comments.
- AI explanations in chat or in PR descriptions.
- Error toasts that genuinely need to explain a system-level failure in operational terms (e.g. "Network unreachable — retry").

## The principle

> Treat every user-facing screen as if it were the first and only version of the product.

This rule is already in `AGENTS.md` §User-facing product voice. This skill operationalizes it: it is the *how*, not just the *what*.

A new user opening the product has no idea:

- What the previous version said.
- What was changed.
- Why it was changed.
- Who decided.
- What trade-offs were made.

They only know:

- What the product is right now.
- What they can do.
- What is expected of them.

So every user-facing string should describe **only the current state and the current action**.

## The two contexts

Keep these contexts separate:

- **Developer context** is allowed history: previous implementation → problem → decision → new implementation. Lives in commit messages, PRs, `CHANGELOG.md`, code comments, internal handoffs.
- **User context** is forever-present: current product state, current capability, current ask. Lives in dialogs, empty states, onboarding, banners, copy.

Do not let developer context leak into user context.

## Workflow

### 1. Read the string the AI proposed

Take the exact string. Read it as a fresh user who has never seen a previous version of the product.

### 2. Identify the history leaks

Mark each phrase that only makes sense if you know the previous state:

- "新版" → no previous version exists for the user.
- "我们决定" → the user did not ask about decisions.
- "更轻松" → easier than what?
- "为了更专注于…" → why are they being told a rationale?

If a phrase is marked, the rewrite must drop the rationale, not preserve it.

### 3. Rewrite to describe only the current state

Apply three transformations:

1. **Strip the rationale.** The user did not ask "why this?"; they need to know "what now?".
2. **State the present tense.** Describe the current product, not its history.
3. **Keep the action clear.** If the string is a button, the verb is the action. If it is a body, the noun is the thing the user does.

### 4. Keep length under control

Rewrites are usually **shorter**, not longer. A history-leaking string often needs justification, and dropping the justification removes words. If the rewrite is longer than the original, suspect that you are preserving rationale.

### 5. Verify against the principle

Read the rewrite as a fresh user. If anything in the string requires knowing what came before, rewrite again.

## Worked rewrites

These come from real bugs in this project. Each row is the AI's first draft, the leak, and the rewrite.

| Original (leaky) | Leak | Rewrite |
|---|---|---|
| "为了让复盘更加轻松，我们现在将每日总结调整为今日回顾。" | full leak: rationale + history + decision-maker | "回顾今天。" |
| "新的记录方式更加简洁，让你专注于真正重要的内容。" | "新的" implies an old, "让你专注于" is rationale | "想到什么，就先记一条。" |
| "现在你不需要再完成完整复盘了。" | "现在" + "再" reveal a prior constraint | "想到什么，就先记一条。" |
| "每日总结（已升级为 AI 智能回顾）" | parenthetical version history | "AI 回顾" |
| "我们已优化加载速度" | first-person plural + superlative | (drop entirely — speed is not a user-facing claim) |
| "新增 Mood 维度" | "新增" implies prior absence | "今天感觉如何？" |
| "由于后端重构，部分功能暂时不可用" | rationale + technical detail | "暂时无法连接，请稍后重试。" |
| "Failed to load: ECONNREFUSED" | raw errMsg | "暂时无法连接，请稍后重试。" |

The pattern in the right column is consistent: describe what the user can do (or what is happening) without explaining why or how it got there.

## Anti-patterns

### The "we made this better" sentence

Any sentence whose grammatical subject is "we" or whose temporal anchor is "new / now / improved / upgraded" is leaky. Rewrite to remove the subject or the anchor.

### The parenthetical version marker

"(v2 / 新版 / 升级后 / 改进版 / beta)" — the user does not see other versions, so the marker has no referent.

### The "as you may have noticed"

Implicit acknowledgment of change. The user *did not* notice because they did not see the prior version. Drop the acknowledgment.

### The release note inside the product

Release notes belong in `CHANGELOG.md` and store update text, not in the product UI. If a user-facing string reads like a release note, it is in the wrong file.

### The error message that explains the system

"Failed to write file: ENOSPC" — operational detail is not user copy. Translate to "存储空间不足，请清理后重试。"

## Verification

Before shipping copy changes:

1. Read every changed string as a fresh user. If it requires knowing prior versions, rewrite.
2. Read every changed string aloud. Sentences that need breath or context to parse are usually leaky.
3. Compare lengths. A rewrite that is longer than the original is suspect.
4. Check that no string contains: 我们 / 现在 / 新版 / 已升级 / 已优化 / 更 / 以前 / 之前 / 旧的 — unless the word is part of the actual noun ("今天的回顾" is fine; "我们今天回顾" is not).

## Output report

For each rewrite, end the handoff with:

1. **Original string** — verbatim, including file and location.
2. **Leak identified** — which of the seven patterns it matched.
3. **Rewrite** — verbatim, in the same file / location.
4. **Principle check** — one sentence confirming the rewrite reads as if the product has always been this way.