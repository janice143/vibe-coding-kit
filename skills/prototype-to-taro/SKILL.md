---
name: prototype-to-taro
description: Convert a single-page HTML prototype into a Taro + React page that follows your project's design system and ships for both H5 and WeChat Mini Program. Use this skill whenever the user says "把这个 HTML 落到 Taro"、"按原型实现"、"把 main-app 转成主应用代码"、"从原型图生成页面", or any equivalent. Also use it whenever the user points at a prototype HTML and asks for the corresponding Taro page, even if they don't explicitly say "Taro" or "原型". Do NOT use this skill for design-system source-of-truth edits, small isolated style tweaks, or API/contract design work.
---

# Prototype → Taro Page

This is the only skill in `vibe-coding-kit` that is framework-specific. The other five skills (and the entire `.knowledge/`) are framework-agnostic. If your stack is not Taro + React, replace this skill with your own `<your-stack>-from-prototype` skill and keep the rest.

## When this skill applies

The user has a self-contained HTML prototype (a single page that lives somewhere outside your source tree) and wants the corresponding Taro page. The output must work on **both H5 and WeChat Mini Program** — there is no "primary" platform.

## When to skip (do not run this skill)

- The user wants to edit the design-system spec or any HTML showcase.
- The change is a small isolated style fix (one selector, one component, one number).
- The user wants to change data shapes, API endpoints, or contracts — that's contract work, not prototype work.
- The user is asking for a critique of the prototype, not an implementation.

## Workflow

### 1. Read the prototype like a translator, not a copier

Open the prototype HTML and list, block by block:

- Each block's **semantic role** (e.g., "weekly overview with stat cards + donut", "trend chart with mood track", "calendar with record detail").
- Each block's **visual primitives** (headings, numbers, units, sparklines, arcs, dots, icons, pills).
- Each block's **data needs** (which fields, which period, which anchor).

Then ask: "Which existing system components already cover this?" Read your project's shared-components directory first. Anything new must be named by **domain concept**, not visual concept (e.g. `OverviewStatCard`, `OverviewDonut`, **not** `Card1` / `Card2`).

### 2. Auto-discover the data source

Don't ask the user. Scan:

1. Your API / client-fetch layer — REST methods, request/response types.
2. Your state hooks (`useAppData`, `useCurrentDateKey`, …).
3. Your shared-types package (`@your-org/contracts` or equivalent) — DTO types.
4. Other pages that consume the **same shape of data** and copy that fetch pattern. This is the strongest signal: if `pages/trend` already pulls `summary.points` from `api.getTrends(period, anchor)`, do the same.

If the data the prototype needs is **not** available anywhere — write a clearly-marked mock (`__MOCK__` comment, `TODO: replace with real fetch`) so the page renders. **Do not** invent new API methods or contract types in this pass. Flag the gap in the output report so the user can wire it up later.

### 3. Token map before any JSX

Before writing a line of code, walk through every visual value and map it to a design-system token. This is the rule that catches 80% of regressions. The token names below are placeholders; substitute your own design system's equivalents.

| What | Token (read your `docs/DESIGN-SYSTEM.md`) |
|---|---|
| Page background | canvas / page bg token |
| Card surface | surface / surface-subtle |
| Card border | none on content cards; border on inputs |
| Body text | ink / ink-muted / ink-light |
| Brand | brand / brand-soft / brand-hover |
| Data colors | dedicated data tokens (NEVER brand) |
| Font size | your 4-tier type scale |
| Radius | your 4-tier radius scale |
| Shadow | your 3-tier shadow scale |
| Container width | from your design system's layout spec |

If a value in the prototype isn't in this table, **stop and tell the user** before introducing it — your design-system spec is the source of truth for tokens and components, and must be extended first per `AGENTS.md` §Source of truth.

### 4. HTML → Taro JSX conversion

Apply these substitutions. Do **not** copy the prototype's inline styles into JSX — extract every style to `app.scss` under a page-scoped class.

| HTML | Taro / React |
|---|---|
| `<div>` | `<View>` |
| `<span>` / `<p>` / `<label>` | `<Text>` (and only when the element is text-bearing — wrap non-text content in `<View>`) |
| `<img src="…">` | `<Image src={importedPng}>` — local PNG import required (weapp cannot resolve remote URLs at runtime) |
| `<svg>` (decorative) | H5: inline `<Icon>` from your icon registry; weapp: PNG via `<Image>` |
| `<svg>` (data viz, e.g., donut, sparkline) | H5: inline `<svg>` via `dangerouslySetInnerHTML`; weapp: render to `<Canvas>` |
| `<button onclick>` | `<View onClick>` — Taro button does work but View matches the rest of the app's chrome |
| `<input>` | `<Input>` from `@tarojs/components` |
| `onclick` | `onClick` |
| inline `style="…"` | Extract to `app.scss` under a block-scoped class |

For inline icons that don't exist in your icon registry, add them following the existing pattern (Lucide-style, `viewBox="0 0 24 24"`, `stroke="currentColor"`, round caps). If you also need a weapp PNG, generate one matching the existing asset size and stroke conventions — never bake a hex color; let weapp PNGs use the same stroke color the SVG uses.

### 5. Page skeleton

Every page must follow your design system's page-layout spec (typically a `§12 Page Layout` section):

- Root is your page-shell component with `<PageHeader title="…" subtitle="…" right={…} />` unless the page has a reason for a different subtitle.
- The page body class must set its root background to your dedicated content-background token — never hand-pick a near-gray. All pages share this value.
- Content block padding follows your layout spec; bottom padding leaves room for the bottom nav.
- Top block (status-bar spacer + PageHeader) is one continuous surface. Don't insert a different-color strip between the status bar and the PageHeader.

### 6. Platform self-check (H5 + weapp)

Before reporting done, run these checks mentally — if any fail, fix and re-check:

- [ ] No JS-set CSS variable contains a bare `rpx` literal. Use `${value * weappScale}${cssUnit}` (your design system's unit-pipeline spec). This is the most common weapp/H5 split.
- [ ] No bare `rpx` written in SCSS either — `pxtransform` rewrites px only.
- [ ] No inline `<svg>` in the weapp render path. `<Icon>` already handles this; if you wrote a custom SVG, route it through your icon registry + PNG.
- [ ] No empty `<Text>` used as a visual element. Wrap non-text decoration in `<View>` (weapp collapses empty `<Text>` to 0×0).
- [ ] Canvas / touch regions use a `<CoverView>` sibling for touch handlers.
- [ ] Data colors use your dedicated data tokens, not brand — this avoids theme collisions when brand changes.
- [ ] Page-local components go in `pages/<name>/_components/` and only get promoted to `components/` once a second page actually needs them. **Do not pre-promote** based on predicted reuse — wait for the concrete second use.

After implementation, if you have a `scripts/smoke-platform-css.mjs` equivalent, run it to verify the `rpx` distribution across platforms.

## Known pitfalls (open to growing)

Each row is a real bug that already cost time. Read this before designing the conversion. Replace the example values with ones from your own project as they accumulate.

| Bug | Platform | Rule to follow |
|---|---|---|
| `<Text>` used for visual decoration (legend dot, color swatch) collapses to 0×0 | weapp | Use `<View>` for visual-only elements |
| `position: sticky` on top header | weapp (real device) | Use `position: fixed` + flex-column layout |
| `rpx` in a JS-set CSS variable | h5 (silently dropped) | Always `${value * weappScale}${cssUnit}`; let `pxtransform` handle SCSS px |
| Await chain before `Taro.shareFileMessage` | weapp (real device) | Use production-before-consumption pattern |
| Data color aliased to brand | all themes | Data colors are fixed values, never aliased to brand |
| Sparkline canvas redrew every render | both | For data passed into `useEffect` deps that are arrays/objects, derive a stable primitive key (e.g. `values.join(',')`) |
| Misread a `display: contents` wrapper | both | Walk through the prototype layout **block by block**; re-derive the effective item count from the rendered tree, not the HTML nesting depth |

## Output report (always end with this)

After finishing, produce a concise report:

1. **Files changed** — paths only, no diffs in the report.
2. **Components** — list new components (and where they live: `_components/` vs `components/`) and reused components.
3. **Data source** — which hook / API method / mock. If mock, say so explicitly.
4. **Design-system deviation** — every prototype value that didn't map cleanly to an existing token. There should be zero of these in a healthy run; if any exist, the user needs to extend the source.
5. **Platform verification** — what was verified locally (typecheck, smoke script) and what needs real-device re-verification (canvas interactions, tabBar colors, animations).