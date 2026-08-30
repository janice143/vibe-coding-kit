# Knowledge Base

This directory holds **durable, project-specific knowledge** that should outlive any single conversation or agent session. It is the place where repeated mistakes, validated patterns, and platform facts live so the next agent (or the next you) does not pay the same cost twice.

It is **not** the design system. Visual tokens and component shapes belong in your `docs/DESIGN-SYSTEM.md`. It is **not** the collaboration contract. Process and rules belong in your `AGENTS.md`. It is **not** a skill. Procedural workflows belong in `../.claude/skills/` (or your agent's skill directory).

It is the middle layer: **concrete, factual, scoped** — the kind of thing that earns a row in this README only if it would otherwise be re-discovered the hard way.

## About the example entries shipped with this kit

The 21 entries under `patterns/`, `anti-patterns/`, `platforms/`, `decisions/`, and `ai-coding/` are real entries from the project that produced this kit (DayRetro / 随心迹, a Taro + React WeChat Mini Program). They are included as **worked examples** so you can see what a good entry looks like.

Before using this kit in a new project:

1. **Keep the templates** in `_templates/`.
2. **Delete the example entries** that do not apply to your stack or product.
3. **Replace the surviving examples** with your own facts as your project hits the same bugs.
4. **Add new entries** as your project surfaces new platform facts or validated patterns.

The `platforms/` entries lean weapp / H5 / Taro because that is what the source project shipped. If you target different platforms, replace them.

The `decisions/` entries are project-specific by design (decision logs are local). Even the source project does not expect you to inherit its storage strategy.

The `patterns/`, `anti-patterns/`, and `ai-coding/` entries are the most generally applicable — those often survive a project change with light editing.

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
├── index.md             ← searchable index, one line per entry
├── _templates/
│   ├── pattern.md          ← template for a validated reusable pattern
│   ├── anti-pattern.md     ← template for a known-bad shape to avoid
│   ├── platform-note.md    ← template for a platform-runtime fact
│   ├── decision.md         ← template for a recorded decision
│   └── ai-coding-note.md   ← template for a coding-agent behavior observation
├── patterns/            ← validated, project-specific patterns
├── anti-patterns/       ← shapes that cost time and should not recur
├── platforms/           ← weapp / H5 / Taro runtime facts (or your equivalents)
├── ai-coding/           ← facts about working with coding agents
└── decisions/           ← durable decisions, with date and reason
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