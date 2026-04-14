---
name: redesign-ui
description: Redesign the entire Vue 3 client into a modern SaaS interface with a left vertical navigation sidebar, fixed design tokens, and consistent spacing. Use when the user asks to redesign the UI, modernize the look, switch to a sidebar layout, or polish the styling across the app.
---

# Redesign UI — SaaS Sidebar Layout

Redesign the full Vue 3 client (`client/src/`) in one pass into a modern SaaS-style interface: vertical navigation sidebar on the left, content area on the right, uniform spacing, and consistent component styling across every view.

The design system below is **locked** — do not invent colors, spacing, or typography. Apply the same tokens everywhere.

## Scope

Touch:
- `client/src/App.vue` — replace top nav with sidebar shell
- `client/src/views/*.vue` — align headings, cards, tables, padding, empty states
- `client/src/components/*.vue` — align cards, modals, buttons, form controls, `FilterBar`, `ProfileMenu`, `LanguageSwitcher`

Do NOT touch:
- `client/src/api.js`, `client/src/main.js` (router), composables, locales, server code, JSON data
- Business logic inside `<script>` blocks — only template structure required for the new layout and `<style scoped>` blocks change. If a view needs a template wrapper for the new layout (e.g. `.page-header`, `.page-body`), add it without changing what data is rendered.

## Mandatory delegation

Per `CLAUDE.md`: **any create/significant-modify on a `.vue` file MUST go through the `vue-expert` subagent.** Spawn one `vue-expert` per view/component file (parallel where possible). Pass the full design system below in the prompt so every agent works from the same tokens.

## Design System (locked)

### Color tokens — CSS variables defined once in `App.vue`'s global `<style>` (not scoped)

```css
:root {
  /* Surfaces */
  --bg-app:        #f6f7f9;   /* page background */
  --bg-surface:    #ffffff;   /* cards, modals, sidebar */
  --bg-sidebar:    #0f172a;   /* dark sidebar */
  --bg-subtle:     #f1f5f9;   /* hover, zebra rows */

  /* Text */
  --text-primary:  #0f172a;
  --text-secondary:#475569;
  --text-muted:    #94a3b8;
  --text-on-dark:  #e2e8f0;
  --text-on-dark-muted: #94a3b8;

  /* Borders */
  --border:        #e2e8f0;
  --border-strong: #cbd5e1;

  /* Accent (single brand color — used sparingly) */
  --accent:        #4f46e5;   /* indigo 600 */
  --accent-hover:  #4338ca;
  --accent-soft:   #eef2ff;
  --accent-on:     #ffffff;

  /* Status */
  --success: #16a34a;  --success-soft: #dcfce7;
  --warning: #d97706;  --warning-soft: #fef3c7;
  --danger:  #dc2626;  --danger-soft:  #fee2e2;
  --info:    #2563eb;  --info-soft:    #dbeafe;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(15,23,42,0.04);
  --shadow-md: 0 1px 3px rgba(15,23,42,0.06), 0 1px 2px rgba(15,23,42,0.04);
  --shadow-lg: 0 10px 15px -3px rgba(15,23,42,0.08), 0 4px 6px -4px rgba(15,23,42,0.05);

  /* Radii */
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;
  --radius-pill: 999px;

  /* Spacing scale (4px base) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-10: 40px;
  --space-12: 48px;

  /* Layout */
  --sidebar-w: 240px;
  --sidebar-w-collapsed: 68px;
  --header-h: 64px;

  /* Typography */
  --font-sans: -apple-system, BlinkMacSystemFont, "Inter", "Segoe UI", Roboto, sans-serif;
  --font-mono: "SF Mono", Menlo, Consolas, monospace;
  --text-xs: 12px;
  --text-sm: 13px;
  --text-base: 14px;
  --text-md: 15px;
  --text-lg: 17px;
  --text-xl: 20px;
  --text-2xl: 24px;
  --text-3xl: 30px;
  --weight-regular: 400;
  --weight-medium: 500;
  --weight-semibold: 600;
  --line-tight: 1.25;
  --line-normal: 1.5;
}

html, body, #app { height: 100%; }
body {
  margin: 0;
  background: var(--bg-app);
  color: var(--text-primary);
  font-family: var(--font-sans);
  font-size: var(--text-base);
  line-height: var(--line-normal);
  -webkit-font-smoothing: antialiased;
}
* { box-sizing: border-box; }
```

### Typography rules
- Page title (`h1` in view): `var(--text-2xl)` / `var(--weight-semibold)` / `var(--line-tight)`
- Section heading (`h2`): `var(--text-lg)` / `var(--weight-semibold)`
- Card/tile title: `var(--text-md)` / `var(--weight-semibold)`
- Body: `var(--text-base)` / `var(--weight-regular)` / `var(--text-primary)`
- Secondary/helper text: `var(--text-sm)` / `var(--text-secondary)`
- Kicker/label (UPPERCASE): `var(--text-xs)` / `var(--weight-medium)` / letter-spacing 0.06em / `var(--text-muted)`

### Layout (locked)

```
┌────────────────────────────────────────────────────────┐
│  SIDEBAR (240px)   │          TOPBAR (64px)            │
│                    ├───────────────────────────────────┤
│  logo              │          FILTER BAR               │
│                    ├───────────────────────────────────┤
│  nav item          │                                   │
│  nav item          │          ROUTER-VIEW              │
│  nav item  active  │          (padding 32px)           │
│  nav item          │                                   │
│                    │                                   │
│  footer (profile)  │                                   │
└────────────────────────────────────────────────────────┘
```

Sidebar rules:
- Fixed-position, full-height, `width: var(--sidebar-w)`, `background: var(--bg-sidebar)`, `color: var(--text-on-dark)`
- Top: product name + subtitle, padding `var(--space-6)`, bottom border `1px solid rgba(255,255,255,0.08)`
- Nav items: flex row, `padding: var(--space-3) var(--space-4)`, `gap: var(--space-3)`, `border-radius: var(--radius-md)`, `margin: 2px var(--space-3)`, hover `background: rgba(255,255,255,0.06)`, active `background: var(--accent)` + `color: var(--accent-on)`
- Every nav item: 20×20 inline SVG icon + label (use simple stroked SVGs, `stroke="currentColor"`, `stroke-width="1.75"`)
- Footer: profile row with avatar + name (truncated), padding `var(--space-4) var(--space-6)`, top border `1px solid rgba(255,255,255,0.08)`
- Content area: `margin-left: var(--sidebar-w)`, background `var(--bg-app)`
- Topbar: sticky top, height `var(--header-h)`, background `var(--bg-surface)`, bottom border `1px solid var(--border)`, contains page title on the left and `LanguageSwitcher` + quick actions on the right
- `FilterBar` sits below the topbar, background `var(--bg-surface)`, bottom border
- Router-view wrapper: `padding: var(--space-8)`, `max-width: 1400px`, `margin: 0 auto`

Responsive: below 1024px, sidebar collapses to `var(--sidebar-w-collapsed)` showing icons only; labels hidden.

### Component rules (apply everywhere)

**Cards / panels** — `background: var(--bg-surface)`, `border: 1px solid var(--border)`, `border-radius: var(--radius-md)`, `padding: var(--space-6)`, `box-shadow: var(--shadow-sm)`. No double-bordered cards nested in other cards.

**Tables** — `border-collapse: separate; border-spacing: 0;`, rounded container with `border: 1px solid var(--border)` and `border-radius: var(--radius-md)` on the wrapper. `th`: UPPERCASE kicker style, `background: var(--bg-subtle)`, `border-bottom: 1px solid var(--border)`, padding `var(--space-3) var(--space-4)`. `td`: padding `var(--space-4)`, border-bottom `1px solid var(--border)` (remove on last row). Hover: `background: var(--bg-subtle)`.

**Buttons**
- Primary: `background: var(--accent)`, `color: var(--accent-on)`, `border: 1px solid var(--accent)`, hover `background: var(--accent-hover)`
- Secondary: `background: var(--bg-surface)`, `color: var(--text-primary)`, `border: 1px solid var(--border-strong)`
- Ghost: transparent, `color: var(--text-secondary)`, hover `background: var(--bg-subtle)`
- Danger: `background: var(--danger)`, `color: #fff`, hover darker
- All: `padding: var(--space-2) var(--space-4)`, `border-radius: var(--radius-sm)`, `font-weight: var(--weight-medium)`, `font-size: var(--text-sm)`, focus ring `box-shadow: 0 0 0 3px var(--accent-soft)`

**Form controls (input, select, textarea)** — `padding: var(--space-2) var(--space-3)`, `border: 1px solid var(--border-strong)`, `border-radius: var(--radius-sm)`, `background: var(--bg-surface)`, focus: `border-color: var(--accent)`, `box-shadow: 0 0 0 3px var(--accent-soft)`.

**Badges / status pills** — padding `2px var(--space-2)`, `border-radius: var(--radius-pill)`, `font-size: var(--text-xs)`, `font-weight: var(--weight-medium)`. Use `-soft` backgrounds + matching strong color text. One badge style per semantic meaning (success/warning/danger/info/neutral).

**Modals** — centered, `max-width: 640px` (or 720px for detail modals), `background: var(--bg-surface)`, `border-radius: var(--radius-lg)`, `box-shadow: var(--shadow-lg)`. Overlay `rgba(15,23,42,0.45)`. Header padding `var(--space-6)`, bottom border. Body padding `var(--space-6)`. Footer right-aligned buttons, top border, padding `var(--space-4) var(--space-6)`.

**Empty / loading states** — centered, `padding: var(--space-12) var(--space-6)`, secondary text, no raw "Loading..." string — use the same text styling as other helper text.

**Spacing discipline** — between sibling sections: `var(--space-8)`. Between cards in a grid: `var(--space-4)`. Inside a card (heading → body): `var(--space-4)`. Grids use CSS Grid with `gap` (never margin hacks).

### Iconography
Use inline SVG icons from a consistent stroked set. Add a tiny local component or put 6–8 icons inline in `App.vue`. Required sidebar icons:
- Dashboard (grid), Inventory (box), Orders (shopping bag), Finance (dollar sign), Demand (trending-up), Reports (bar chart)

Do NOT add external icon libraries — stay zero-deps. Do NOT use emojis (per project rule).

## Execution plan

Run in this order. Each step is a separate delegation to `vue-expert`.

1. **Shell rewrite** — `client/src/App.vue`
   - Replace top-nav template with sidebar + topbar layout per the diagram above.
   - Inject all CSS variables in an **unscoped** `<style>` block (or `<style>` tag without `scoped`) so every child component can use them.
   - Keep all existing logic (`setup()`, tasks wiring, i18n, modals) intact.
   - Nav items use `<router-link>` with `active-class` or `exact-active-class` for active styling (avoid manual `$route.path ===` checks — cleaner and handles trailing slashes).
   - Include sidebar icons inline as SVG.

2. **Filter bar** — `client/src/components/FilterBar.vue`
   - Restyle to sit below topbar, surface background, align inputs to new form-control rules.

3. **Topbar widgets** — `client/src/components/ProfileMenu.vue`, `LanguageSwitcher.vue`
   - Move these into the topbar/sidebar footer per layout. Restyle with design tokens.

4. **Views** (one vue-expert per file, parallel OK):
   `Dashboard.vue`, `Inventory.vue`, `Orders.vue`, `Demand.vue`, `Spending.vue`, `Reports.vue`, `Backlog.vue`
   - Wrap content in a `.page-header` (title + subtitle) and `.page-body` grid of cards.
   - Replace hand-rolled colors/paddings with tokens.
   - Keep all existing data bindings, computed props, and event handlers unchanged.

5. **Modals** (parallel OK):
   `InventoryDetailModal.vue`, `OrdersDetailModal` (if exists), `ProductDetailModal.vue`, `BacklogDetailModal.vue`, `CostDetailModal.vue`, `TasksModal.vue`, `ProfileDetailsModal.vue`
   - Apply modal design rules. Consistent header/body/footer structure across all modals.

## Non-obvious constraints — document these in code comments when they apply

Per user preference, when a styling choice is non-obvious, add a one-line comment explaining the WHY. Examples: *"unscoped so tokens cascade to child components"*, *"using active-class to avoid re-matching on trailing slash"*.

## Verification

After all edits, a `verification` subagent must confirm:
1. Frontend builds (`cd client && npm run build`) and dev server runs clean
2. All 6 sidebar routes navigate and render without console errors
3. No references to the removed `.top-nav` / `.nav-tabs` classes remain in any `.vue` file
4. CSS variables resolve (grep any view for a raw hex from the old palette like `#3b82f6` — none should remain unless in an SVG icon)
5. Visual check via Playwright MCP: screenshot `/`, `/inventory`, `/orders`, `/demand`, `/spending`, `/reports` at 1280×800 and confirm sidebar present on the left, no top nav, consistent card styling

On FAIL, fix and re-verify. Do not report completion until PASS.

## Reporting

Final report to the user:
- Files changed (count by type: App.vue, views, components, modals)
- Any component that couldn't be migrated cleanly and why
- Screenshot paths (if captured) so the user can diff at a glance
