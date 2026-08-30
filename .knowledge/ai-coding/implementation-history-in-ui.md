# AI coding note: implementation-history-in-ui

> **One-sentence summary.** Coding agents rewrite user-facing strings to explain *why* a change was made ("新版", "我们已", "为了更…"), leaking implementation history into the product.

**Observed:** 2026-08 (multiple copy edits across the project)
**Agents involved:** Claude Code, OpenCode, Gemini-based prototyping flows
**Task type:** copy edits, dialog text, empty states, onboarding steps

## What the agent did

When asked to change a label like "每日总结" to "今日回顾", the agent routinely produced strings like:

> "为了让复盘更加轻松，我们现在将每日总结调整为今日回顾。"

Or, when adding a Mood dimension:

> "新增 Mood 维度，让你更准确地表达今天的感受。"

Or, when removing a feature:

> "现在你不需要再完成完整复盘了。"

Each string is *factually correct* from the developer's perspective. Each string *fails* from a new user's perspective: the new user has no prior version to compare to, no decision-maker to attribute the change to, and no rationale to accept.

## Why it is the default

The agent's context includes the full implementation history: the previous version of the string, the reason for the change, the trade-offs considered. The agent's natural move is to share useful context. The bug is that this context is **internal**. Sharing it in user-facing copy is a category error.

The pattern generalizes: any change that has a story behind it tends to leak that story into the user-visible surface unless explicitly filtered.

## What it cost

- Several rounds of "rewrite this UI string" → review → "still feels like a release note" → rewrite again.
- User-facing copy that read as a changelog embedded in the product.
- The user eventually added a hard rule (`AGENTS.md` §User-facing product voice: "Treat every user-facing screen as if it were the first and only version of the product").

## What to do instead

1. Apply the `product-copy-rewriter` skill on every UI string before merging.
2. Self-check: read the string as a fresh user. If it requires knowing what came before, rewrite.
3. Strip first-person plural ("我们"), temporal anchors ("新版 / 现在"), and rationale clauses ("为了 / 因此").
4. Compare lengths: a rewrite that is longer than the original is suspect.

## See also

- `../.claude/skills/product-copy-rewriter/SKILL.md` — the procedure and rewrite table.
- `../../AGENTS.md` §User-facing product voice — the rule form.
- `../templates/_templates/` (this knowledge base) — `pattern.md` / `anti-pattern.md` for related future entries.
