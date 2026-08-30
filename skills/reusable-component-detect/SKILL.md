---
name: reusable-component-detect
description: Stop mid-implementation when an AI coding agent has produced an obvious reuse signal, extract the right layer (component / hook / helper / token), update the project's reuse contract (`AGENTS.md` + design system), and resume. Use this skill whenever an AI writes a second similar-looking card / dialog / input / handler / style block, whenever a page-local file crosses ~150 lines and the structure starts to feel copy-pasted from a sibling page, or whenever a third-party reviewer would say "you should have factored this out two files ago". Do NOT use this skill for a single isolated component, for design-system token creation (that is a rule update, see `AGENTS.md`), or for speculative abstractions where no second use exists yet.
---

# Reusable Component Detect

## When this skill applies

You are about to write (or have just written) code that:

- Mirrors the structure of an existing component with only visual props changed.
- Repeats a dialog / drawer / modal that is structurally identical to one already in the codebase.
- Re-implements a hook that another page already exports.
- Inlines a style block that re-declares values that exist in `docs/DESIGN-SYSTEM.md` §2 / §5 / §8.
- Defines a new color, font-size step, radius, or shadow that isn't in the design-system.

Apply it **at the moment of detection**, not after the third copy. The earlier you stop, the less you have to undo.

## When to skip (do not run this skill)

- The repeated code is one occurrence. Wait for the second concrete use before extracting.
- The change is one isolated visual tweak in one file.
- You are writing the *first* page in a brand-new area; nothing exists to reuse yet.

## Why this matters

AI coding agents default to local-optimum completion: solve the page at hand, copy what's in front of them, and never look across the project for prior art. The output is functionally correct but the codebase drifts — three slightly-different card paddings, two competing input styles, a hook that exists in three pages with three names. This skill is the interrupt: stop, look sideways, and let the project grow as one system instead of N parallel sandpiles.

## The four layers

Pick the right layer for what is actually repeating. Do not collapse all reuse into "make a component":

| What is repeating | Extract to | Examples |
|---|---|---|
| Visual structure (markup + style) | A component in your project's shared `components/` directory (e.g. `src/components/`) | `Card`, `Dialog`, `MoodPill`, `EmptyState`, `SectionHeader` |
| Stateful behavior (data + effects) | A hook in your project's shared `hooks/` directory (or page-local first) | `useDisclosure`, `useAsyncTask`, `useConfirmDanger` |
| Pure logic (parsing, formatting, computation) | A helper in your project's shared `utils/` directory (or a contracts / types module) | `formatDateKey`, `groupBy`, `diffRecords` |
| A specific visual value (color / radius / shadow / spacing / type) | A token update in `docs/DESIGN-SYSTEM.md` (or your design-system spec) + the corresponding `_tokens.scss` / token file | `--ds-radius-md`, `--ds-shadow-card`, `$ds-font-size-caption` |

> **Adjust the paths to match your project.** The names `components/`, `hooks/`, `utils/` and `docs/DESIGN-SYSTEM.md` are conventions — substitute the actual paths your `AGENTS.md` §Repository map points to.

Decision rule:

- If the duplication shows up in JSX shape → component.
- If the duplication is `useState` + `useEffect` logic → hook.
- If the duplication is a function with no React → helper.
- If the duplication is one specific number → token.

## Workflow

### 1. Halt before the second copy is committed

When you notice the second similar block being written, stop the implementation. Do not finish the page.

If you are the one writing the code:

> Pause the file. Output a one-paragraph "reuse signal" note before continuing.

If you are reviewing another agent's output:

> Reject the change. Ask for the extraction first.

### 2. Inspect the existing project for prior art

Before extracting, search:

- your shared `components/` directory — is there already a Card / Dialog / EmptyState / MoodPill?
- your shared `hooks/` directory — is there already `useXxx`?
- your shared `utils/` directory — is there already `formatXxx`?
- `docs/DESIGN-SYSTEM.md` (or your design-system spec) — is the value already a token?

Use the project's own vocabulary. Do not introduce a new name if `Card` already exists; do not introduce `CardItem` if `Card.Body` already exists.

### 3. Choose the layer (component / hook / helper / token)

Apply the table in §The four layers. If two layers look plausible, choose the narrower one — a hook inside one page is fine before promoting to `hooks/`.

### 4. Extract with a single owner

When extracting:

- One file owns the canonical implementation. No parallel copies remain.
- The previous two pages get rewired to the extracted owner in the same change, not "later".
- The new file lives in the right place from the start (`components/` vs `hooks/` vs `utils/`). Do not park it in `_components/` of one page and call it "promoted later".

### 5. Update the reuse contract

The project has two pieces of contract that the agent reads before coding:

1. `AGENTS.md` (or `CLAUDE.md`) — short rule list.
2. `docs/DESIGN-SYSTEM.md` — tokens and component shapes.

For a **component** extraction:

- Add a one-line entry in `AGENTS.md` listing the component and its path. If `AGENTS.md` does not have a "shared components" section, add one. Format: `- \`<ComponentName>\` — one-line description — \`<path-to-component>\``.

For a **token** extraction:

- Update `docs/DESIGN-SYSTEM.md` §2 / §5 / §8 **first** (per `AGENTS.md` "Source of truth"). Then add the SCSS variable to `_tokens.scss`. Order matters.

For a **hook** or **helper**:

- Add the path to `AGENTS.md` under a small "shared hooks / utils" section, or update the relevant page reference if it is genuinely page-local.

### 6. Re-run the page that triggered the signal

With the extraction in place, redo the page that triggered the signal using the new owner. Verify:

- Same visual output (within design-system tolerances).
- Same behavior on H5 and weapp.
- The page's own `_components/` and inline styles no longer contain the duplicated structure.

### 7. Resume and verify the rest of the work

After extraction + rewire + contract update, continue with the original task. The extraction is not a side-quest — it is part of finishing the task that triggered it.

## Signals worth stopping for

These are real signals observed in this project. Each is a stop-and-extract moment:

- **Two cards with the same shadow / radius / padding / divider.** Promote to a Card component or a `Card.Section` slot.
- **Two dialogs with the same title / body / confirm-cancel footer.** Promote to a Dialog component with a `danger` prop.
- **Two input groups with the same label / input / error layout.** Promote to a Field component, or to a shared `entry-module` style (see `docs/DESIGN-SYSTEM.md` §7.4.1).
- **Two empty states with the same illustration + headline + CTA.** Promote to an EmptyState component.
- **Two places computing the same date key from a Date / string.** Promote to `formatDateKey` in `utils/`.
- **A new color, radius, shadow, or font-size step that is not in the design system.** Add the token first.
- **A page-local `_components/` file that already has 3+ siblings and clear domain meaning.** Promote to `components/`.

## Anti-patterns

### Speculative abstraction

Don't extract a "shared" component the moment you write the first occurrence. Two concrete uses is the minimum signal. One occurrence is not yet a pattern.

### Re-promotion

If a component already lives in your shared `components/` directory, do not create a "v2" or "extended" copy in a page-local `_components/` folder. Extend the canonical component or wrap it; do not parallelize.

### Parallel naming

If `Card` already exists, do not name the new one `CardItem`, `RecordCard`, `InfoCard`. Use `Card` with a `variant` or composition. New names for the same concept create parallel APIs.

### Token-via-style

Do not "save a token for later" by hardcoding a hex in a component. Either it belongs in `docs/DESIGN-SYSTEM.md` §2 (and then the SCSS layer) or it is a violation. There is no middle ground.

## Output report

End the handoff with:

1. **Signal** — what duplication triggered the stop.
2. **Layer chosen** — component / hook / helper / token, and why.
3. **Owner file** — exact path the canonical implementation was written to.
4. **Contract updates** — which sections of `AGENTS.md` / `docs/DESIGN-SYSTEM.md` were updated.
5. **Rewired call sites** — list of files that now use the owner.
6. **Verification** — same visual / same behavior on H5 + weapp at the affected boundary.