# vibe-coding-kit

> A starter kit for AI-assisted coding — `AGENTS.md` + skills + a knowledge base, distilled from real production pain.

Six procedural skills, twenty-one knowledge entries, one collaboration contract. Total weight: ~30 small Markdown files. Designed to be copied wholesale into your project, then grown with your own entries.

## Why this kit exists

Coding agents default to:

- Inventing per-page values (new grays, new radius, new shadows).
- Copy-pasting components locally instead of searching for prior art.
- Inflating process for small tasks (brainstorm → spec → plan → subagent → test → review for a copy edit).
- Leaking implementation history into user-facing copy ("we've redesigned…").
- Trusting simulators that lie about real-device behavior.

These cost hours. They recur. They do not go away with a better model.

This kit is a set of files an agent reads before coding, plus a structure the agent grows over time.

## What's inside

```text
vibe-coding-kit/
├── AGENTS.md                          ← the collaboration contract
├── CLAUDE.md                          ← Claude Code adapter
├── README.md                          ← this file
├── README.zh-CN.md                    ← 中文版本
├── LICENSE                            ← MIT
├── .gitignore
├── skills/
│   ├── prototype-to-taro/             ← HTML prototype → Taro + React page
│   ├── refactoring-ui/                ← visual hierarchy + design polish
│   ├── wechat-tap-gesture/            ← gesture API flow shape
│   ├── reusable-component-detect/     ← when to halt and refactor
│   ├── product-copy-rewriter/         ← strip implementation history from UI
│   └── task-size-router/              ← small / medium / large process
└── .knowledge/
    ├── README.md                      ← when to add an entry
    ├── index.md                       ← one-line index, sorted by reference
    ├── _templates/                    ← 5 templates: pattern, anti-pattern, platform-note, decision, ai-coding-note
    ├── patterns/                      ← 5 validated patterns
    ├── anti-patterns/                 ← 5 known-bad shapes
    ├── platforms/                     ← 4 weapp / H5 / Taro runtime facts
    ├── decisions/                     ← 4 project decisions
    └── ai-coding/                     ← 3 coding-agent behaviors to intercept
```

## The 3-layer model

| Layer | Question it answers | Lives in |
|---|---|---|
| **Rule** | What must always be true? | `AGENTS.md` |
| **Skill** | How do I do this procedure? | `skills/<name>/SKILL.md` |
| **Knowledge** | What did we learn that is project-specific? | `.knowledge/<category>/` |

Rules are invariants. Skills are procedures. Knowledge is the record.

- If something should change behavior on every run → rule.
- If something should change behavior when a specific procedure runs → skill.
- If something is a fact about the world (or this codebase) that the next agent will benefit from → knowledge.

## Install

```sh
npx skills add <owner>/vibe-coding-kit
```

Or copy the kit into your repo manually:

```sh
# from this repo's root
cp AGENTS.md CLAUDE.md /path/to/your-project/
cp -r skills /path/to/your-project/.claude/skills
cp -r .knowledge /path/to/your-project/.knowledge
```

Then:

1. Update `AGENTS.md` §Repository map to point at your project's actual paths and commands.
2. Update `AGENTS.md` §Source of truth if your project has a design-system spec.
3. Replace the example entries in `.knowledge/` with your own (or delete them).

## Quick start

1. Copy `AGENTS.md` and `CLAUDE.md` to your project root.
2. Copy the skills you want into `.claude/skills/<name>/` of your project.
3. Initialize `.knowledge/` with the templates; delete or replace the example entries.
4. Add new entries as bugs get diagnosed or patterns get validated — `.knowledge/README.md` explains the criteria.

## Provenance

These files were distilled from a real 26-day, ~150-commit Taro + React WeChat Mini Program project (DayRetro / 随心迹). The `.knowledge/` entries were written as the project hit each bug, not retroactively. They are honest examples showing the format — replace the DayRetro specifics with your own.

## What this kit is not

- **Not a framework.** The kit works with Taro, React, Vue, Svelte, plain HTML, anything.
- **Not a design system.** Bring your own `docs/DESIGN-SYSTEM.md`. The `prototype-to-taro` skill is the only one with framework-specific guidance; the rest are framework-agnostic.
- **Not an LLM vendor lock-in.** The same files work with Claude Code, OpenCode, Codex, and other coding agents.
- **Not heavy.** There is no `brainstorm → spec → plan → subagent → review` workflow by default. The default is `inspect → act → verify → report`, scaled to task size.

## License

MIT.