---
name: chalkdoor-ui-ux
description: Chalkdoor UI/UX rules. Covers picking a community design system (shadcn/ui by default, also Carbon, MUI, Mantine and others), the rule that the user's own template or design always wins, UX quality and accessibility rules, and how to suggest UX improvements. Use when building or changing UI, choosing a design system, or when the user shares a design.
---

# UI and UX

## Choosing the design

1. **The user's own design comes first.** If they share a template, Figma file, screenshot, brand kit or existing UI, follow it as closely as possible. Do not swap it for a design system. Only adapt it to the stack, for example by rebuilding it with Tailwind.
2. **No design given** → ask once, offering a recommendation:
   > Do you have a design or template? If not, I'll use **shadcn/ui** (clean, modern, works natively with Next.js and Tailwind). Other options: IBM Carbon (enterprise and data-heavy apps), Material UI, Mantine.
3. Pick from `references/design-systems.md` by app type. Use **one design system per app**. Record it as `stack.ui_system`.
4. Migration: keep the existing UI library during the parity pass. A design system switch is its own slice, done only if the user agrees.

## UX rules

- **Clear first screen:** what the app is, and the one main action.
- **Every data view has its states:** loading (a skeleton, not just a spinner), empty (with a next step), error (plain words and a retry), and success feedback (a toast or inline message).
- **Forms:** visible labels; inline validation messages next to the field; the submit button disabled while pending; the user's input kept after an error.
- **Navigation:** at most about 7 top-level items, the current page highlighted, and a working back/forward through URL state for filters and tabs.
- **Mobile-first:** test at 375 px and 1280 px. Touch targets are at least 44 px. No horizontal scroll.
- **Accessibility (WCAG 2.2 AA):** semantic HTML, keyboard reachable with visible focus, 4.5:1 text contrast, `alt` on images, and no meaning carried by color alone.
- **Consistency:** use design tokens (colors, spacing, radius, type scale) from the design system. No one-off hex values or pixel sizes.
- **Speed:** use `next/image` with sizes and `next/font`, and avoid layout shift. Keep client JavaScript small (`skills/nextjs-standard`).

## Suggesting improvements

When you notice a UX problem, suggest the fix in one line, with its community reason, and let the user decide. For example: "Suggestion: add an empty state on /projects; a blank table makes users think the app is broken. Want it?" Do not change agreed UI on your own.

## Frontend-led backend

The user's UI demands drive the backend. When a requested screen needs data or an action that does not exist yet, add the smallest server function or Server Action for it (`skills/nextjs-standard/references/backend.md`). Do not build backend features that no screen uses.
