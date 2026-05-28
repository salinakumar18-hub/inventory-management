---
name: redesign-sidebar-nav
description: Redesign this Vue 3 app's UI from a horizontal top nav bar to a modern SaaS-style vertical sidebar. Trigger phrases: "redesign to use a sidebar", "add sidebar navigation", "convert top nav to sidebar", "SaaS sidebar layout", "vertical navigation sidebar".
---

# Sidebar Navigation Redesign Skill

Convert the app from a horizontal top-navigation layout to a modern SaaS-style layout with a dark vertical sidebar. Follow every step in order. Do not skip steps.

---

## Architecture Overview

**Before:**
```
.app (flex-direction: column)
  └── .top-nav (70px sticky bar, horizontal)
  └── <FilterBar> (sticky below top-nav, top: 70px)
  └── .main-content (flex: 1)
        └── <router-view>
```

**After:**
```
.app (flex-direction: row, height: 100vh, overflow: hidden)
  └── .sidebar (240px, full height, sticky)
        ├── .sidebar-brand (logo + subtitle)
        ├── .sidebar-nav (vertical router-links with icons)
        └── .sidebar-footer (LanguageSwitcher + ProfileMenu)
  └── .main-area (flex: 1, overflow: hidden)
        ├── .main-header (compact sticky bar with FilterBar)
        └── .main-content (router-view content, overflow-y: auto)
```

---

## Step 1 — Rewrite App.vue Template

Replace the entire `<template>` block in `client/src/App.vue` with:

```html
<template>
  <div class="app">
    <aside class="sidebar">
      <div class="sidebar-brand">
        <div class="brand-logo">
          <svg width="28" height="28" viewBox="0 0 28 28" fill="none">
            <rect width="28" height="28" rx="7" fill="#2563eb"/>
            <path d="M7 10h14M7 14h10M7 18h12" stroke="white" stroke-width="2" stroke-linecap="round"/>
          </svg>
        </div>
        <div class="brand-text">
          <span class="brand-name">{{ t('nav.companyName') }}</span>
          <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
      </div>

      <nav class="sidebar-nav">
        <router-link to="/" class="nav-item" :class="{ active: $route.path === '/' }">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 18 18" fill="none">
            <rect x="2" y="2" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.5"/>
            <rect x="10" y="2" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.5"/>
            <rect x="2" y="10" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.5"/>
            <rect x="10" y="10" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.5"/>
          </svg>
          <span>{{ t('nav.overview') }}</span>
        </router-link>

        <router-link to="/inventory" class="nav-item" :class="{ active: $route.path === '/inventory' }">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 18 18" fill="none">
            <path d="M2 5h14M2 9h14M2 13h14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            <circle cx="5" cy="5" r="1" fill="currentColor"/>
            <circle cx="5" cy="9" r="1" fill="currentColor"/>
            <circle cx="5" cy="13" r="1" fill="currentColor"/>
          </svg>
          <span>{{ t('nav.inventory') }}</span>
        </router-link>

        <router-link to="/orders" class="nav-item" :class="{ active: $route.path === '/orders' }">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 18 18" fill="none">
            <path d="M3 3h12l-1.5 9H4.5L3 3z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
            <circle cx="7" cy="15.5" r="1" stroke="currentColor" stroke-width="1.5"/>
            <circle cx="12" cy="15.5" r="1" stroke="currentColor" stroke-width="1.5"/>
          </svg>
          <span>{{ t('nav.orders') }}</span>
        </router-link>

        <router-link to="/spending" class="nav-item" :class="{ active: $route.path === '/spending' }">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 18 18" fill="none">
            <circle cx="9" cy="9" r="7" stroke="currentColor" stroke-width="1.5"/>
            <path d="M9 5v1m0 6v1m-2.5-4h4a1 1 0 110 2H7a1 1 0 100 2h5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          <span>{{ t('nav.finance') }}</span>
        </router-link>

        <router-link to="/demand" class="nav-item" :class="{ active: $route.path === '/demand' }">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 18 18" fill="none">
            <polyline points="2,14 6,8 10,11 15,4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            <circle cx="15" cy="4" r="1.5" fill="currentColor"/>
          </svg>
          <span>{{ t('nav.demandForecast') }}</span>
        </router-link>

        <router-link to="/reports" class="nav-item" :class="{ active: $route.path === '/reports' }">
          <svg class="nav-icon" width="18" height="18" viewBox="0 0 18 18" fill="none">
            <rect x="3" y="2" width="12" height="14" rx="2" stroke="currentColor" stroke-width="1.5"/>
            <path d="M6 6h6M6 9h6M6 12h4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          <span>Reports</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <LanguageSwitcher sidebar />
        <ProfileMenu
          sidebar
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="main-area">
      <header class="main-header">
        <FilterBar />
      </header>
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />
    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>
```

The `<script>` block requires no changes.

---

## Step 2 — Replace Layout CSS in App.vue

In the `<style>` block, replace the layout-level classes (`.app`, `.top-nav`, `.nav-container`, `.nav-tabs`, `.main-content`) with:

```css
/* App shell */
.app {
  display: flex;
  flex-direction: row;
  height: 100vh;
  overflow: hidden;
}

/* Sidebar */
.sidebar {
  width: 240px;
  flex-shrink: 0;
  background: #0f172a;
  border-right: 1px solid rgba(255, 255, 255, 0.06);
  display: flex;
  flex-direction: column;
  height: 100vh;
  position: sticky;
  top: 0;
  overflow-y: auto;
  z-index: 100;
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1.25rem 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
}

.brand-logo { flex-shrink: 0; }

.brand-text {
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
  overflow: hidden;
}

.brand-name {
  font-size: 0.875rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.01em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-subtitle {
  font-size: 0.688rem;
  color: #64748b;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 0.75rem;
  gap: 0.125rem;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.625rem 0.75rem;
  color: #94a3b8;
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  border-radius: 7px;
  /* transition needed here because active/hover states use background color and the left border accent — without it the state changes feel abrupt */
  transition: background 0.15s ease, color 0.15s ease;
  border-left: 2px solid transparent;
}

.nav-item:hover {
  background: rgba(255, 255, 255, 0.06);
  color: #e2e8f0;
}

.nav-item.active {
  background: rgba(37, 99, 235, 0.15);
  color: #93c5fd;
  border-left-color: #2563eb;
}

.nav-icon {
  flex-shrink: 0;
  opacity: 0.8;
}

.nav-item.active .nav-icon { opacity: 1; }

.sidebar-footer {
  padding: 0.875rem 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.06);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

/* Main area */
.main-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  min-width: 0;
}

/* overflow: hidden here (not auto) is intentional — .main-content handles scrolling,
   keeping the header pinned without a second scrollbar */
.main-header {
  background: #f8fafc;
  border-bottom: 1px solid #e2e8f0;
  flex-shrink: 0;
  position: sticky;
  top: 0;
  z-index: 90;
}

.main-content {
  flex: 1;
  overflow-y: auto;
  padding: 1.5rem 2rem;
}
```

Keep all other global classes untouched: `.stats-grid`, `.stat-card`, `.card`, `.badge.*`, `.table-container`, `table`, `.loading`, `.error`, `.page-header`, `.modal-*`.

---

## Step 3 — Update FilterBar.vue

Open `client/src/components/FilterBar.vue`. In `<style scoped>`:

Change `.filters-bar` to:
```css
.filters-bar {
  background: transparent;
  padding: 0.625rem 0;
}
```

Change `.filters-container` to:
```css
.filters-container {
  padding: 0 2rem;
  display: flex;
  align-items: center;
  gap: 1rem;
}
```

Remove: `position: sticky`, `top: 70px`, `z-index: 90`, `max-width: 1600px`, `margin: 0 auto` from these rules. The parent `.main-header` in App.vue handles sticky behavior and the bottom border.

---

## Step 4 — Update ProfileMenu.vue

Add a `sidebar` boolean prop and dark-variant scoped styles so the button blends into the dark sidebar without using `:deep()` overrides.

In `<script setup>`:
```javascript
const props = defineProps({
  sidebar: { type: Boolean, default: false }
})
```

Bind to `.profile-button` in template:
```html
<button class="profile-button" :class="{ 'profile-button--sidebar': sidebar }" ...>
```

Add to `<style scoped>`:
```css
.profile-button--sidebar {
  background: rgba(255, 255, 255, 0.06);
  border-color: rgba(255, 255, 255, 0.1);
  width: 100%;
  justify-content: flex-start;
}
.profile-button--sidebar .profile-name { color: #cbd5e1; }
.profile-button--sidebar .chevron { color: #64748b; margin-left: auto; }
.profile-button--sidebar:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.15);
}
```

---

## Step 5 — Update LanguageSwitcher.vue

Same pattern as Step 4.

In `<script setup>`:
```javascript
const props = defineProps({
  sidebar: { type: Boolean, default: false }
})
```

Bind to `.language-button` in template:
```html
<button class="language-button" :class="{ 'language-button--sidebar': sidebar }" ...>
```

Add to `<style scoped>`:
```css
.language-button--sidebar {
  background: rgba(255, 255, 255, 0.06);
  border-color: rgba(255, 255, 255, 0.1);
  color: #94a3b8;
  width: 100%;
  justify-content: flex-start;
}
.language-button--sidebar:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.15);
  color: #e2e8f0;
}
```

---

## Step 6 — Design Checklist

After all changes, verify:

**Layout**
- [ ] `.app` is `flex-direction: row`
- [ ] Sidebar is 240px wide, full viewport height, does not scroll with content
- [ ] Main area is `flex: 1`, content scrolls independently
- [ ] No horizontal scrollbar at 1280px viewport

**Sidebar**
- [ ] Background `#0f172a`, right border `rgba(255,255,255,0.06)`
- [ ] Logo + brand name + subtitle visible at top
- [ ] All 6 nav links present and functional
- [ ] Active route: blue tinted background + 2px `#2563eb` left accent border
- [ ] Inactive links: `#94a3b8` (not white)
- [ ] Hover: subtle background highlight, 0.15s transition
- [ ] ProfileMenu and LanguageSwitcher visible in footer, dark-themed

**FilterBar**
- [ ] Renders inside `.main-header`, sticky at top of main area
- [ ] No duplicate sticky bars
- [ ] Filters still work (change warehouse → data updates)

**Components**
- [ ] ProfileMenu dropdown not clipped by sidebar (opens over main area)
- [ ] LanguageSwitcher dropdown not clipped
- [ ] Modals (ProfileDetailsModal, TasksModal) open correctly
- [ ] Language toggle EN ↔ JA works

**Polish**
- [ ] Nav transitions at 0.15s ease
- [ ] No emojis in sidebar
- [ ] Inter font throughout

---

## Common Pitfalls

1. **Do not remove `<FilterBar />`** — only move it from after `.top-nav` into `.main-area > .main-header`
2. **Do not touch `<script>`** in App.vue — all composables and modal logic stay identical
3. **Scoped styles cannot be overridden from App.vue** — use the prop approach (Steps 4–5), not `:deep()`
4. **Z-index order**: sidebar=100, main-header=90, modals=200+
5. **`overflow: hidden` on `.main-area`** is required — `.main-content` handles the actual scroll
6. **"Reports" nav link is hardcoded** — no i18n key exists for it yet; keep it hardcoded

---

## Route-to-Icon Reference

| Route | i18n key | Icon |
|-------|----------|------|
| `/` | `t('nav.overview')` | 2×2 grid squares |
| `/inventory` | `t('nav.inventory')` | Horizontal lines with dots |
| `/orders` | `t('nav.orders')` | Shopping cart |
| `/spending` | `t('nav.finance')` | Circle with dollar sign |
| `/demand` | `t('nav.demandForecast')` | Rising line chart |
| `/reports` | hardcoded "Reports" | Document with lines |
