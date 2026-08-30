---
name: task-size-router
description: Decide up front, in 30 seconds, whether a task is small / medium / large, and route it to the right amount of process. Use this skill at the very start of every task that touches code or docs — before reading files, before forming a plan, before invoking any other skill. Do NOT use this skill after the task is already underway (route mid-flight is allowed only if scope materially changes), and do NOT use it to justify creating heavyweight artifacts for a small change.
---

# Task Size Router

## When this skill applies

At the start of **every** task, before any other action. This is a 30-second routing decision, not an analysis. If you find yourself spending more than two minutes on this skill, you have already miscategorized the task.

## When to skip

Never. Even a one-line copy edit benefits from a one-second "small" categorization. The cost of running this skill is so low that skipping it is itself a process violation.

## The decision tree

Answer three questions in order. The first "no" stops the routing.

### Q1 — Does the task touch more than three files, or cross a domain boundary?

- **Yes** → continue to Q2.
- **No** → **Small**. Stop here.

"Domain boundary" means: client / contracts / docs, frontend / backend, design-system / page, schema / runtime. If the change stays inside one of those, it is not crossing a boundary even if it touches two files.

### Q2 — Is the requirement clear, the affected code already understood, and the blast radius local?

- **Yes** → **Medium**. Stop here.
- **No** (any of: ambiguous requirement, or unknown blast radius, or a choice the user should make first) → continue to Q3.

### Q3 — Is this a new subsystem, a data-model change, a cross-cutting refactor, a security-sensitive change, a migration, or genuinely unclear?

- **Yes** → **Large**. Plan, ask, possibly worktree or subagent — see `AGENTS.md` §Large or high-risk changes.
- **No** (the work is real but it is not any of the above) → **Medium**. A short inline approach statement is enough.

## The three sizes

### Small

Examples: copy edit, one CSS value, an isolated visual bug, an obvious local fix in one to three files.

Process:

```text
read relevant files → make focused change → narrowest verification
```

Do not enter plan mode. Do not write a spec. Do not decompose into tasks. Do not launch subagents. Do not restart planning after a clarification unless scope materially changes.

Common false signals that push small tasks upward: the agent wants to be thorough, the user asked "is this OK?", the change has a long history in the conversation. None of these justify process. Only scope justifies process.

### Medium

Examples: a few related files, a contained refactor, a meaningful but localized implementation choice.

Process:

```text
inspect affected code → state a short inline approach if helpful → implement → verify at affected boundary
```

Ask only questions that block a safe decision; otherwise implement directly. Do not create persistent specs or plans by default.

A medium task might produce an inline approach paragraph ("I will extract `useDisclosure` and rewire `Dialog` and `ConfirmDialog` to use it") but not a markdown file.

### Large

Examples: new subsystem, architecture or data-model change, cross-cutting refactor, security-sensitive change, migration, genuinely unclear requirements.

Process:

```text
clarify the important constraints → present approach for approval → plan / subagent / independent review only when their benefit outweighs overhead
```

Independent review, worktree, and subagent are tools, not defaults. They are valuable when:

- The blast radius spans code you have not read yet.
- Multiple agents could plausibly make progress in parallel.
- A second pair of eyes would catch a regression cheaply.
- The user explicitly asks for them.

If none of these are true, a large task still gets a plan and an approach-confirmation, but not necessarily a worktree or a subagent.

## Anti-patterns

### Process inflation

The default mode of any capable agent is to over-process. Adding a plan, a spec, a task list, a subagent, a worktree, a review, and a CHANGELOG entry to a 10-line copy change is process inflation. The cost of the process exceeds the cost of the change.

Rule of thumb: if the change is shorter than the process document you are about to write, the process is wrong.

### Workflow framework reflex

Don't reach for `brainstorm → spec → plan → subagent → implement → test → review` because a framework exists. The default is `inspect → act → verify → report`. Reach for additional steps only when the size of the task forces it.

### Subagent for serial work

Subagents are for parallelism or context isolation. A serial five-step refactor is faster with one agent reading files in order. Do not split a serial task into subagents for theater.

### Plan mode for a clear change

Plan mode is for alignment when the path is unclear. If you can describe the change in two sentences and you have read the affected code, plan mode is overhead. Implement directly.

### Spec file for a small or medium task

Specs are durable records for large or high-risk work. Specs for a CSS adjustment create documentation debt with no reader. The agent that reads it next will not need it; the user will not look at it.

## Re-routing mid-flight

If the scope changes materially while you are working:

- A small task reveals a hidden cross-module dependency → re-route to Medium, possibly Medium-with-approach-statement.
- A medium task reveals a data-model consequence → re-route to Large, stop, confirm with the user.
- A large task simplifies because a hidden invariant already exists → re-route down, drop the plan.

Do not re-route up because the user asked a clarifying question. Questions are not scope expansion.

## Routing examples

Concrete examples from this project:

| Task | Files touched | Blast radius | Size |
|---|---|---|---|
| Change a button label | 1 file, 1 string | local | Small |
| Adjust a card padding to match design system | 1 SCSS file | local | Small |
| Rename a CSS variable across the codebase | SCSS files that reference it | design-system level | Medium |
| Add a new page following `prototype-to-taro` | new page folder, possibly one shared component | local | Medium |
| Add a new section to `docs/DESIGN-SYSTEM.md` | design system + SCSS tokens | cross-cutting, but contained | Medium |
| Add a new top-level page to the app | new folder + bottom nav entry + design system | cross-cutting | Large |
| Migrate all storage to a new schema | storage layer + every page that reads + backup format | full surface | Large |
| Build a new subsystem (e.g., AI analysis) | new files + new contract + new state + new UI | cross-cutting | Large |
| Add a new design token | design system + SCSS + (if visible) components | design-system level | Medium (token update is the work, then propagation) |
| Introduce a new color palette | design system + theme files + every component using brand | cross-cutting, every page | Large |

Notice that "Medium" can still be cross-cutting if the change is well-understood. The deciding factor is **clarity and blast-radius**, not file count alone.

## Output report

The router itself does not produce a long report. It produces a one-line routing decision that goes at the top of the task report:

```text
Route: small (1 file, copy edit, no boundary).
```

If Medium, also include the inline approach in one or two sentences.

If Large, the report is the approach-confirmation exchange with the user, not a markdown file the router writes.