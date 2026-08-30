# Knowledge Base

This directory holds **durable, project-specific knowledge** that should outlive any single conversation or agent session. It is the place where repeated mistakes, validated patterns, and platform facts live so the next agent (or the next you) does not pay the same cost twice.

It is **not** the design system. Visual tokens and component shapes belong in your `docs/DESIGN-SYSTEM.md`. It is **not** the collaboration contract. Process and rules belong in your `AGENTS.md`. It is **not** a skill. Procedural workflows belong in `../.claude/skills/` (or your agent's skill directory).

It is the middle layer: **concrete, factual, scoped** — the kind of thing that earns a row in this README only if it would otherwise be re-discovered the hard way.

## What this kit ships

This kit ships **only the templates** (`_templates/`). It deliberately ships **no example entries** — every project's knowledge base is project-specific by design. Copy a template, fill it in with the first fact your project surfaces, and let the directory grow from there.

## When to add an entry

Add an entry when **any** of these is true:

- A bug took longer than 30 minutes to diagnose and the cause is not obvious from reading the code.
- A pattern was validated across two or more concrete uses and should be the default next time.
- A platform fact (WeChat, Taro, browser runtime, iOS, Android, …) contradicted intuition and the contradiction is not in the official docs in plain form.
- A decision was made that the next agent would benefit from understanding (decision log, not design rationale).

Do not add an entry when:

- The information is already in your `docs/DESIGN-SYSTEM.md` or `AGENTS.md` (point there instead).
- The information is a one-off observation with no expected reuse.
- The information is speculative — wait for it to be validated.

## Directory layout

```text
.knowledge/
├── README.md            ← this file
├── index.md             ← optional one-line index, sorted by reference (create when you have ≥10 entries)
├── _templates/          ← entry templates (do not delete)
│   ├── pattern.md          ← template for a validated reusable pattern
│   ├── anti-pattern.md     ← template for a known-bad shape to avoid
│   ├── platform-note.md    ← template for a platform-runtime fact
│   ├── decision.md         ← template for a recorded decision
│   └── ai-coding-note.md   ← template for a coding-agent behavior observation
├── patterns/            ← validated, project-specific patterns (you create)
├── anti-patterns/       ← shapes that cost time and should not recur (you create)
├── platforms/           ← weapp / H5 / Taro runtime facts (you create)
├── ai-coding/           ← facts about working with coding agents (you create)
└── decisions/           ← durable decisions, with date and reason (you create)
```

Each entry is a single Markdown file. Filenames are short kebab-case names (`rpx-in-js-css-vars.md`, not `the-time-i-spent-3-hours-on-rpx.md`). The first line of each file is a one-sentence summary that ends up in `index.md`.

## How this differs from skills and rules

| Layer | Question it answers | Lives in |
|---|---|---|
| **Rule** | What must always be true? | `AGENTS.md`, your `docs/DESIGN-SYSTEM.md` §0 |
| **Skill** | How do I do this procedure? | your `.claude/skills/<name>/SKILL.md` |
| **Knowledge** | What did we learn that's specific to *this* project? | `.knowledge/**` |

Rules are invariants. Skills are procedures. Knowledge is the record.

If something should change behavior on every run → rule. If something should change behavior when a specific procedure runs → skill. If something is a fact about the world (or this codebase) that the next agent will benefit from → knowledge.

## Lifecycle

- **Write** when a fact earns its place (see "When to add an entry" above).
- **Reference** from `AGENTS.md`, skills, or commit messages when relevant — keep references short.
- **Re-validate** when the underlying fact changes (a framework upgrade, a platform change, a redesign). If the entry is no longer true, delete it or update it explicitly.
- **Do not** keep entries "just in case". A small, accurate `.knowledge/` is more useful than a large, dated one.

## Verification before adding

Before writing an entry, check:

1. Is the fact already in your `docs/DESIGN-SYSTEM.md`, `AGENTS.md`, or a `SKILL.md`? If yes, point to it instead of duplicating.
2. Is the fact a generic engineering truth (e.g. "validate inputs")? If yes, it does not belong here.
3. Is the fact one-off or speculative? If yes, wait.
4. Will the next agent actually be better off reading this? If not, do not write it.

If all four pass, write it.
