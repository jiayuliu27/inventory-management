# Sidebar Layout Template

Copy-adaptable markup and CSS for the SaaS sidebar shell (Phase 3–5 of the vue-saas-redesign workflow). Adapt class names, tokens, and i18n keys to the target app — do not paste blindly.

## Shell Markup (layout root, e.g. App.vue)

```vue
<template>
  <div class="app">
    <aside class="sidebar">
      <div class="sidebar-brand">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
      </div>

      <nav class="sidebar-nav" aria-label="Main navigation">
        <router-link
          to="/"
          :class="{ active: $route.path === '/' }"
          :aria-current="$route.path === '/' ? 'page' : undefined"
        >
          {{ t('nav.overview') }}
        </router-link>
        <!-- one router-link per route; same active/aria-current pattern.
             Root link uses exact path match so it isn't active on every route. -->
      </nav>

      <div class="sidebar-footer">
        <!-- version, help link, or secondary info.
             Do NOT place drop-DOWN widgets here: the sidebar's overflow-y clips them. -->
      </div>
    </aside>

    <div class="content">
      <header class="content-header">
        <!-- slim bar for widgets that lived in the old top nav -->
        <div class="content-header-spacer"></div>
        <LanguageSwitcher />
        <ProfileMenu />
      </header>

      <FilterBar />  <!-- if the app has one; re-anchor its sticky offset -->

      <main class="main-content">
        <router-view />
      </main>
    </div>
  </div>
</template>
```

## Shell CSS

```css
/* Two-column app shell. Grid (not flex) so the sidebar column is rigid
   and the content column can't push it around. */
.app {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
}

.sidebar {
  /* sticky + 100vh keeps the sidebar pinned while content scrolls */
  position: sticky;
  top: 0;
  height: 100vh;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  background: var(--color-surface);
  border-right: 1px solid var(--color-border);
  z-index: var(--z-sidebar);
}

.sidebar-brand {
  padding: var(--space-5) var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--color-border);
}
.sidebar-brand h1 {
  font-size: 1.125rem;
  font-weight: 700;
  letter-spacing: -0.025em;
  color: var(--color-text);
}
.sidebar-subtitle {
  font-size: 0.75rem;
  color: var(--color-text-secondary);
}

.sidebar-nav {
  flex: 1;                      /* pushes the footer to the bottom */
  padding: var(--space-4) var(--space-3);
  display: flex;
  flex-direction: column;
  gap: 2px;
}
.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-sm);
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--color-text-secondary);
  text-decoration: none;
  /* transparent accent reserves the space so the active state doesn't shift text */
  border-left: 3px solid transparent;
  transition: background 0.15s, color 0.15s;
}
.sidebar-nav a:hover {
  background: var(--color-bg);
  color: var(--color-text);
}
.sidebar-nav a.active {
  background: color-mix(in srgb, var(--color-primary) 8%, transparent);
  border-left-color: var(--color-primary);
  color: var(--color-primary);
  font-weight: 600;
}
.sidebar-nav a:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: -2px;
}

.sidebar-footer {
  padding: var(--space-4) var(--space-5);
  border-top: 1px solid var(--color-border);
  font-size: 0.75rem;
  color: var(--color-text-secondary);
}
```

## Content Column + Slim Header

```css
.content {
  min-width: 0;   /* CRITICAL: lets grid children shrink; without it, wide
                     tables/charts force horizontal page overflow */
  display: flex;
  flex-direction: column;
}

.content-header {
  height: var(--header-height);
  display: flex;
  align-items: center;
  gap: var(--space-4);
  padding: 0 var(--space-8);
  background: var(--color-surface);
  border-bottom: 1px solid var(--color-border);
  position: sticky;
  top: 0;
  /* MUST be higher than any sticky bar below it (--z-sticky). position:sticky +
     z-index makes this header a stacking context, so dropdowns inside it can never
     paint above the header's own layer — if a later sticky sibling (filter bar)
     shares that z-index, it wins by DOM order and covers/intercepts the dropdowns. */
  z-index: var(--z-header); /* e.g. 100: sidebar < sticky bars < header < modals */
}
.content-header-spacer { flex: 1; }  /* pushes widgets to the right edge */

.main-content {
  flex: 1;
  padding: var(--space-6) var(--space-8);
  width: 100%;
  max-width: 1400px;   /* readable line lengths on ultrawide monitors */
}
```

If a sticky filter/toolbar sits between the header and main content:

```css
.filters-bar {
  position: sticky;
  /* anchored below the content header, NOT the removed old top-nav height */
  top: var(--header-height);
  z-index: var(--z-sticky);
}
```

## Mobile Variant (off-canvas, ~900px breakpoint)

```css
@media (max-width: 900px) {
  .app { grid-template-columns: 1fr; }

  .sidebar {
    position: fixed;
    left: 0; top: 0;
    width: var(--sidebar-width);
    transform: translateX(-100%);
    transition: transform 0.2s ease;
    z-index: var(--z-dropdown); /* above sticky bars, below modals */
  }
  .sidebar.open { transform: translateX(0); }

  .sidebar-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(15, 23, 42, 0.4);
    z-index: calc(var(--z-dropdown) - 1);
  }
}
```

Markup additions for mobile:

```vue
<button
  class="menu-toggle"
  @click="sidebarOpen = !sidebarOpen"
  aria-label="Toggle navigation"
  :aria-expanded="sidebarOpen"
>
  <!-- inline SVG hamburger icon; no emoji -->
</button>
<div v-if="sidebarOpen" class="sidebar-backdrop" @click="sidebarOpen = false"></div>
```

Place the toggle button in `.content-header`; hide it above the breakpoint. Close the sidebar on route change (`watch($route, () => sidebarOpen = false)`).

## Checklist After Applying

- [ ] Old layout selectors (`.top-nav`, `.nav-tabs`, …) removed AND grep confirms nothing still references them
- [ ] Sticky offsets re-anchored (`top: var(--header-height)` or `0`)
- [ ] `min-width: 0` present on the content column
- [ ] Dropdown widgets in the content header, not inside the scrolling sidebar
- [ ] Root nav link uses exact path matching
- [ ] `aria-current="page"`, `aria-label` on nav, `:focus-visible` styles present
- [ ] All locale files have every nav key
