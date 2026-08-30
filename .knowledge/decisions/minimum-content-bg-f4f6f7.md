# Decision: minimum-content-bg-f4f6f7

> **One-sentence summary.** The content area background is locked to `#F4F6F7` (`$ds-content-bg`) across all five pages; no page may set its own background color.

**Decided:** 2026-08
**Decided by:** project owner
**Status:** active

## Context

The visual system has three surfaces: the body canvas (`$ds-bg` `#F5F6F8`, used in early HTML showcase only), the white card surface (`$ds-surface` `#FFFFFF`), and the content area background between cards (`$ds-content-bg`). During initial development the content area drifted between `#f4f6f7`, `#f5f6f7`, and `#f3f5f6` across pages. Each page looked fine in isolation; together they read as three different products.

## Decision

`$ds-content-bg = #F4F6F7`. All five pages (`home`, `trend`, `records`, `mine`, `record-document`) use this exact value. `$ds-bg` is retained for the HTML showcase only and is not used in the shipped app. `$ds-surface` (`#FFFFFF`) is reserved for cards and the top block.

## Why

Three roles require three surfaces; one of them was drifting. Pinning `$ds-content-bg` makes the visual hierarchy deterministic and stops AI agents (which otherwise default to inventing per-page hex values) from breaking it.

## Consequences

- The three-surface system (`$ds-bg` showcase canvas · `$ds-content-bg` shipped content area · `$ds-surface` cards and top block) is now load-bearing. Adding a new surface requires updating the design-system spec, not just adding a SCSS variable.
- The content area on every page looks the same, which is a feature: the visual signal "this is app content" is consistent.
- A future dark theme will need to redefine all three values; the structure does not change.

## When to revisit

- A future dark mode is added. `$ds-content-bg` gets a dark counterpart.
- A page genuinely needs a different content background (e.g. a dedicated "onboarding" or "settings" screen). In that case, introduce a new token (`$ds-content-bg-emphasis`) rather than overriding the existing one per page.

## See also

- `../../docs/DESIGN-SYSTEM.md` §2.1 / §12.2 — token definition and page layout standard.
- `../../AGENTS.md` §Multi-platform verification — verification matrix includes "no new page-level background colors".
