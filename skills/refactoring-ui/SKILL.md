---
name: refactoring-ui
description: Apply Refactoring UI principles when designing, polishing, or refactoring frontend interfaces. Use this skill whenever the user asks to improve a UI, visual hierarchy, spacing, typography, color, responsive layout, cards, buttons, forms, empty states, or interaction polish—even when they do not explicitly mention Refactoring UI. Prefer the smallest shippable improvement, preserve the existing framework, and work from the project's design tokens before introducing new values.
---

# Refactoring UI

Use this skill to turn functional interfaces into intentional, readable, production-grade interfaces. Design the actual feature first, then refine the surrounding shell. Work in short, disposable iterations: make the smallest useful change, inspect it, and improve only what is necessary.

## Operating principles

1. **Inspect before changing.** Read the existing component, styles, design-system documentation, and nearby patterns. Do not propose values or components you have not inspected.
2. **Use the project's system first.** Reuse existing colors, typography, spacing, radius, shadow, icon, and responsive tokens. Add a new token only when the existing system cannot express the requirement.
3. **Design the feature before the shell.** Establish the hierarchy of the actual content and actions before adjusting global navigation, headers, or page chrome.
4. **Start with the smallest useful version.** Defer nice-to-have decoration and configuration until the core interaction is clear.
5. **Prefer de-emphasis over amplification.** If the primary action does not stand out, reduce the contrast, size, weight, or visual noise of competing elements before making the primary action louder.
6. **Verify the golden path.** For frontend changes, run the relevant checks and use the UI when a dev server or browser is available. Report any platform or visual boundary that could not be verified.

## Visual hierarchy

### Emphasis

Use a small, deliberate hierarchy:

- Primary content: darkest ink and strong weight.
- Supporting content: muted ink and normal weight.
- Tertiary content: light ink, captions, or metadata.
- Use font weight and color before changing font size.
- Avoid small text below `400` weight; it becomes difficult to read.
- Do not make every element equally prominent.

### Actions

Match treatment to importance:

- **Primary:** solid, high-contrast button with a clear label.
- **Secondary:** outline or quieter surface treatment.
- **Tertiary:** text or link treatment that remains discoverable.
- Destructive actions should be quiet until the confirmation step, where the destructive choice becomes explicit and unmistakable.
- Labels are supporting content. Prefer a concise value that explains itself, such as `12 left`, over a redundant `Remaining: 12`.

### Alignment

- Left-align body copy and multi-line descriptions.
- Center-align only short headings or descriptions of roughly two or three lines.
- Keep long text within a readable line length, approximately 45–75 characters where the layout allows.
- Use spacing to make groups apparent: space between groups should exceed space within a group.

## Layout and spacing

- Start with generous whitespace, then reduce only where the interface feels disconnected or too sparse.
- Use the project's spacing scale. If none exists, prefer a constrained scale such as `4, 8, 12, 16, 24, 32, 48, 64` rather than inventing one-off values.
- Avoid two near-identical spacing values that communicate no meaningful difference.
- Give each element only the width it needs; do not fill available space without a reason.
- On wide screens, use content-appropriate max-widths instead of stretching every text block across the viewport.
- Design for a narrow mobile canvas first, then verify wider layouts.
- Use fixed dimensions for layout constraints when appropriate. Reserve relative units for intentional scaling, not as a default.
- Do not let a rigid grid override the content's natural hierarchy.

## Typography

- Establish a constrained type scale before styling individual elements. A useful baseline is `12, 14, 16, 18, 20, 24, 30, 36, 48px`, but use the project's tokens when available.
- Keep body text around `1.5–1.7` line-height for comfortable reading.
- Tighten line-height as headings get larger; large headlines often need approximately `1.0–1.2`.
- Set line-height and paragraph spacing independently.
- Tighten letter-spacing for large display headings when it improves cohesion.
- Add letter-spacing to all-caps labels for readability.
- Use normal text weight for most content and reserve heavier weight for meaningful emphasis.

## Color

- Build from a small, semantic palette: surface, subtle surface, border, primary ink, muted ink, light ink, brand, and semantic states.
- Never introduce a random gray when a semantic muted or light token exists.
- Avoid pure gray where a subtly cool or warm neutral better fits the interface.
- Text on a colored surface should be a hue-related shade, not an unrelated gray that looks disabled.
- Adjust saturation and hue as well as lightness when creating a darker or lighter brand variant.
- Check normal-text contrast against its actual background; target at least `4.5:1`.
- De-emphasize inactive controls with softer contrast rather than making active controls excessively loud.

## Depth, surfaces, and borders

- Use a small elevation system. A restrained baseline is:
  - Small: `0 1px 3px rgba(0, 0, 0, .12)`
  - Medium: `0 3px 6px rgba(0, 0, 0, .15)`
  - Large: `0 10px 20px rgba(0, 0, 0, .15)`
  - Modal: `0 15px 25px rgba(0, 0, 0, .15)`
  - Tooltip: `0 20px 40px rgba(0, 0, 0, .2)`
  Use the repository's shadow tokens instead of these examples when available.
- Use one subtle border or shadow to establish a surface; do not stack border, radius, and shadow on every nested item.
- Raised elements can gain a little shadow on hover or focus and should visually settle when pressed.
- A light top edge and a softer lower shadow can make a raised element feel grounded, but keep the effect restrained.
- Prefer spacing, surface contrast, or a hairline separator over adding another full card.
- Keep accent borders subtle, generally no more than `3–5px`, and use them to clarify state or hierarchy.

## Components

### Buttons and controls

- Make the primary action obvious without making every action primary.
- Give controls enough horizontal padding for both Chinese and English labels.
- Customize focus, active, pressed, hover, disabled, and loading states instead of relying on browser defaults.
- Keep the selected state visually distinct from focus; they communicate different things.
- Do not use an oversized icon as a substitute for a missing action label unless the icon is unambiguous in context.

### Cards and forms

- Use cards for independent content, independently actionable blocks, or meaningful surface separation.
- Keep internal metrics, list rows, legends, and helper text flat inside their parent card unless they are independently actionable.
- Align parallel form groups with the same surface, border, radius, shadow, padding, and spacing.
- Use a focused border/shadow state rather than inventing a different background for one form section.

### Empty states

Treat empty states as a first-class flow:

- Explain what is missing in one concise sentence.
- Show the next useful action prominently.
- Use an illustration or icon only when it helps the user understand the state.
- Hide filters, tabs, or controls that have no useful function before data exists.
- Verify the first-use experience, not just the populated state.

### Icons and images

- Use the existing icon system and icon sizes.
- Do not scale a small icon far beyond its intended size or force a large icon into a tiny control.
- When a large visual is needed, place a correctly sized icon inside a shaped surface.
- For user images, constrain the container, use an intentional crop, and prevent background bleed.
- Do not design around low-quality placeholders and assume they will be replaced later.

## Responsive and platform behavior

- Test the narrowest supported viewport first.
- Check Chinese and English label lengths, wrapping, truncation, and button width.
- Avoid relying on a browser-only behavior when the project also targets a native or mini-program runtime.
- Keep platform-specific units and runtime style variables consistent with the project's build pipeline.
- If a layout depends on safe-area insets, scrolling containers, sticky/fixed positioning, or native chrome, inspect and verify each target platform separately.

## Refinement checklist

Before reporting a UI change as complete, check:

1. Can the primary action and primary content be identified immediately?
2. Are competing elements intentionally quieter?
3. Are spacing values drawn from the existing scale?
4. Are typography size, weight, line-height, and line length coherent?
5. Does text meet the contrast requirement on its real background?
6. Are borders, shadows, radius, and nested cards used sparingly and consistently?
7. Do hover, focus, pressed, disabled, and loading states communicate state clearly?
8. Does the empty state provide a useful next action?
9. Do Chinese and English labels fit without awkward wrapping?
10. Does the interface work on the project's supported platforms?
11. Did the change stay within the requested scope?

## Output expectations

When implementing a UI change:

- Briefly state the hierarchy or interaction decision before editing if the choice is non-obvious.
- Reuse existing project patterns instead of introducing a parallel design language.
- Keep the implementation focused; do not add speculative abstractions.
- Report changed paths, focused verification, visual/platform limitations, and any remaining unverified boundary.
