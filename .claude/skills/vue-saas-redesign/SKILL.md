---
name: vue-saas-redesign
description: Workflow for redesigning a Vue 3 application's UI into a modern SaaS-style interface with a vertical left sidebar, design-token normalization, and consistent spacing. Use this skill when converting a top navigation bar to a sidebar layout, modernizing or polishing a Vue app's look, or normalizing an app's colors and spacing into CSS variables.
---

# Vue SaaS Redesign Workflow

Transform a Vue 3 app from a top-nav layout into a modern SaaS-style interface: vertical left sidebar navigation, a normalized design-token system, consistent spacing, and a polished professional look.

Work through the phases **in order** — each phase depends on the output of the previous one. Do not skip the audit.

## Core Principle: Adapt, Don't Replace

The redesign must preserve the app's existing brand identity. Audit what exists, keep the brand palette, and normalize it into a token system.

```
❌ BAD:  Replace the app's blues with your favorite indigo palette
✅ GOOD: Promote the app's most-used blue to --color-primary and use it consistently
```

If the app already looks 80% consistent, your job is to formalize that consistency, not to redesign from taste.

## Phase 1: Audit Current Layout & Tokens

Produce a written inventory **before touching any code**:

1. **Token census** — grep across all `.vue` files (and any global CSS) for:
   - Hex colors / rgb(a) values: `#[0-9a-fA-F]{3,8}` — group by usage (primary action, text, borders, backgrounds, status colors)
   - `font-size`, `font-family`, `font-weight` values
   - `border-radius`, `box-shadow` values
   - `z-index` values — build a stacking table (which element sits above which)
2. **Layout root** — find the component that owns the page shell (usually `App.vue`): where the nav markup lives, where `<router-view>` renders, what containers wrap it.
3. **Global vs scoped styles** — identify which style blocks are unscoped (global). Unscoped classes are a contract with every view: list the classes views depend on (`.card`, `.page-header`, etc.). These selectors must survive the redesign verbatim.
4. **Sticky elements** — record every `position: sticky/fixed` element and what its `top:` offset depends on (e.g., a filter bar stuck at `top: 70px` because the old header is 70px tall). These break silently when the header is removed.
5. **Navigation inventory** — list all nav links, their routes, how the active state is computed (router-link classes vs manual `:class` bindings), and whether labels are i18n'd (`t('nav.x')`) or hardcoded. Hardcoded labels need new keys in **every** locale file.

## Phase 2: Extract Tokens to `:root` CSS Variables

Normalize the Phase 1 inventory into a `:root` block in the global stylesheet:

```css
:root {
  /* Colors — promoted from the app's existing palette, not invented */
  --color-primary: #2563eb;
  --color-primary-hover: #1d4ed8;
  --color-text: #0f172a;
  --color-text-secondary: #64748b;
  --color-border: #e2e8f0;
  --color-surface: #ffffff;
  --color-bg: #f8fafc;

  /* Spacing — 4/8px-based scale */
  --space-1: 0.25rem;  --space-2: 0.5rem;  --space-3: 0.75rem;
  --space-4: 1rem;     --space-5: 1.25rem; --space-6: 1.5rem;  --space-8: 2rem;

  /* Radii & shadows — from the audit's most common values */
  --radius-sm: 6px;  --radius-md: 8px;  --radius-lg: 10px;
  --shadow-card: 0 4px 12px rgba(0, 0, 0, 0.06);
  --shadow-dropdown: 0 10px 25px rgba(0, 0, 0, 0.1);

  /* Layout */
  --sidebar-width: 240px;
  --header-height: 56px;

  /* Z-index scale — must mirror the audited stacking order.
     The header needs its own layer ABOVE sticky bars: see pitfall 3. */
  --z-sidebar: 50;
  --z-sticky: 90;
  --z-header: 100;
  --z-dropdown: 1000;
  --z-modal: 2000;
}
```

Rules:
- **Value-neutral**: this phase must not change any computed style. Screenshot before/after one view to confirm zero visual drift.
- Collapse near-duplicate values deliberately (e.g., 1.25rem vs 1.375rem paddings → one token), not blindly — check each usage.
- Replace hardcoded values with variables in files you touch during the redesign; do NOT mass-rewrite untouched views in the same pass.

## Phase 3: Build the Sidebar Shell

Read `references/sidebar-template.md` for complete copy-adaptable markup and CSS.

Layout root becomes a two-column grid:

```css
.app {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
}
.sidebar {
  position: sticky; top: 0;
  height: 100vh;
  overflow-y: auto;
}
```

Sidebar anatomy (top to bottom):
1. **Brand area** — logo / app name / subtitle
2. **Nav section** (scrollable) — links, optionally grouped with small uppercase labels
3. **Footer area** — version info, settings, or secondary widgets

Active state: left accent bar + tinted background — **not** the old horizontal-tab underline:

```
❌ BAD:  Inherit the old `.nav-tabs a.active::after` bottom-underline styles
✅ GOOD: border-left / inset box-shadow accent + background tint + stronger text color
```

Cautions:
- `overflow-y: auto` on the sidebar **clips absolutely-positioned dropdowns** placed inside it. Widgets with drop-*down* menus belong in a content header, not the sidebar footer (see Phase 5).
- Collapse/expand behavior is optional — skip it unless explicitly requested.

## Phase 4: Migrate Navigation

- Move the `router-link`s into `<aside><nav aria-label="Main navigation">`.
- Preserve i18n calls **exactly** (`{{ t('nav.overview') }}`). Add missing locale keys for any hardcoded labels — in every locale file, not just the default language.
- Accessibility: `aria-current="page"` on the active link (bind it, or use router-link's `exact-active-class`).
- Root-link gotcha: `router-link-active` on `to="/"` matches every route. Keep exact matching (manual `$route.path === '/'` check or `exact-active-class`).
- Icons: add them only if the app already has an icon approach (inline SVG is typical). Never use emojis in the UI.

## Phase 5: Relocate Header Widgets & Sticky Bars

Widgets that lived in the old top nav (profile menu, language switcher, search) need a new home. Two options:

| Option | Trade-off |
|---|---|
| **Slim content header** (recommended) | A `--header-height` bar at the top of the content column. Dropdowns keep opening downward — usually zero changes to the widgets themselves. |
| Sidebar footer | More "SaaS look", but drop-down menus must be rewritten to open upward/right, and the sidebar's overflow clips them. Higher risk. |

Sticky bars (filters, toolbars) that were offset by the old header **must be re-anchored in the same pass**:

```
❌ BAD:  Leave `top: 70px` after the 70px top-nav is gone (bar floats mid-page)
✅ GOOD: `top: var(--header-height)` if below a content header, or `top: 0` if not
```

Also remove any `max-width` centering on such bars that assumed full viewport width — they now live inside the content column.

## Phase 6: Normalize Spacing Across Views

- Content padding, page-header margins, card padding, and grid gaps all move to spacing tokens.
- Aim for one consistent rhythm: e.g., `--space-6` between page sections, `--space-5` card padding, `--space-4` grid gaps.
- Verify each route visually after the change rather than mass-editing every view file.

## Phase 7: Polish Pass

- Shadows and radii come from tokens everywhere they appear.
- Hover states on nav items and cards (background tint, subtle shadow lift).
- Visible `:focus-visible` rings on all interactive elements (nav links, buttons, selects).
- Consistent transition durations (0.15–0.2s) on hover/focus changes.
- Typography hierarchy check: one page title size, one section title size, consistent secondary-text color.

## Phase 8: Verify in the Browser

Use browser automation (Playwright) against the running dev server:

1. **Every route**: renders with the sidebar, correct link has active styling and `aria-current="page"`. Screenshot each.
2. **Interactions**: change a filter → data updates; reset works. Open every dropdown — fully visible, not clipped. Open modals — they overlay the sidebar.
3. **Sticky behavior**: scroll a long page; sticky bars anchor at the intended offset with no gap or overlap.
4. **Locales**: switch to each supported language; sidebar labels must not overflow or truncate at the chosen `--sidebar-width`.
5. **No horizontal overflow**: `document.documentElement.scrollWidth <= document.documentElement.clientWidth`.
6. **Responsive**: at narrow widths (~800px), verify the chosen fallback (off-canvas sidebar or compact top bar — see the reference template).

## Common Pitfalls

1. **Orphaned global selectors** — restructuring the layout markup while global (unscoped) styles still reference removed classes. After renaming, grep for the old class names to confirm nothing else uses them.
2. **Sticky offset rot** — any `top:` offset tied to the old header height breaks silently. Fix in the same commit as the layout change.
3. **Z-index collisions and stacking contexts** — sidebar must sit below dropdowns and modals. Encode the audited order in the `--z-*` tokens and comment why each layer has its value. Critically: `position: sticky` + `z-index` creates a *stacking context*, so a dropdown's `z-index: 1000` only competes **inside its parent's layer**. If the content header and a filter bar below it share the same z-index, the later DOM sibling paints on top and its overlap intercepts clicks on the header's dropdowns. Give the header its own layer above other sticky bars (`--z-header`).
4. **Missing locale keys** — a hardcoded nav label "works" in the default language and regresses every other locale.
5. **Sidebar width vs translations** — longer languages (German, Japanese compounds) may wrap; test before finalizing `--sidebar-width`.
6. **`/` link active everywhere** — non-exact matching highlights Home on all routes.
7. **Scoped-style reach** — styles scoped to the layout root do not reach child components moved during the restructure; keep shared classes global.

## Key Reminders

- Audit first; write the token inventory before editing code.
- Token extraction is value-neutral — zero visual drift allowed in Phase 2.
- Preserve unscoped global classes that views depend on, verbatim.
- Re-anchor every sticky element the moment the old header is removed.
- i18n labels preserved exactly; new keys added to all locale files.
- `aria-current="page"`, nav landmark, `:focus-visible` rings — accessibility is part of "polished".
- No emojis in the UI. Comment the *why* on magic numbers (z-index layers, sidebar width).
- If the project mandates delegation for `.vue` changes (check its CLAUDE.md), follow that rule.
