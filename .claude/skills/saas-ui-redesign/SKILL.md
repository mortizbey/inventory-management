---
name: saas-ui-redesign
description: Redesign the inventory-management Vue 3 client into a modern SaaS-style interface - left vertical sidebar navigation instead of the top nav bar, a design-token spacing scale, and a polished, consistent look across all views. Use when asked to redesign the UI, add a sidebar, modernize the layout, or give the app a SaaS / professional look.
---

# SaaS UI Redesign (inventory-management)

Turns the current top-nav layout into a SaaS shell: fixed left sidebar, sticky top bar with page title and filters, tokenized spacing, and a consistent card/table/badge style. Tailored to this repo's files and coupling points.

Read before starting:
- [reference/design-tokens.md](reference/design-tokens.md) - token values and sidebar spec. Use these exact values.
- [reference/checklist.md](reference/checklist.md) - acceptance checklist for the final step.

## Guardrails

- **Delegate `.vue` edits.** Project rule: any create or significant modification of a `.vue` file goes to the `vue-expert` agent. If `vue-expert` is not available as an agent type, use a `general-purpose` agent and tell it to read and follow `.claude/agents/vue-expert.md`. Pass it the two reference files and the relevant step(s) below.
- Client only. No changes to `server/`, `server/data/`, `api.js` contracts, or `vite.config.js`.
- No new npm dependencies. Icons are inline SVG. No emojis anywhere in the UI.
- Keep existing global class names (`.page-header`, `.stats-grid`, `.stat-card`, `.card`, `.card-header`, `.card-title`, `.table-container`, `.badge.*`, `.loading`, `.error`) - all views depend on them. Restyle, don't rename.
- Keep all task and modal logic in `App.vue` (`tasks`, `addTask`, `deleteTask`, `toggleTask`, `showProfileDetails`, `showTasks`) behaviorally unchanged.
- Comment non-obvious changes (e.g. why a sticky offset uses a token, why a dropdown opens upward).
- Do not add a route for `views/Backlog.vue` (it is currently unrouted). Mention it in the final report instead.
- Do not commit. Report the diff summary.

## Known coupling points (fix all of them)

| Location | Coupling | Fix |
|---|---|---|
| `client/src/App.vue` `.top-nav`, `.nav-container`, `.nav-tabs` | 70px top header, 1600px max-width | Replace with sidebar + top bar shell |
| `client/src/App.vue` `.main-content` | `max-width: 1600px; margin: 0 auto` | Max-width applies to inner content only; gutter via tokens |
| `client/src/components/FilterBar.vue` `.filters-bar` | `position: sticky; top: 70px` | Becomes part of the sticky top bar (or `top: var(--topbar-h)`) |
| `client/src/components/FilterBar.vue` `.filters-container` | own `max-width: 1600px; padding: 0 2rem` | Remove; align to content gutter `var(--space-8)` |
| `client/src/components/ProfileMenu.vue` `.dropdown-menu` | opens downward, `right: 0` | Open upward (`bottom: calc(100% + 0.5rem); left: 0`) when in sidebar footer |
| `client/src/components/LanguageSwitcher.vue` dropdown | positioned for top-right header | Re-check placement in sidebar footer; open upward |
| `client/src/views/Spending.vue` sticky element (~line 783) | sticky offset may assume old header | Verify; use `var(--topbar-h)` based offset if needed |
| `client/src/App.vue` Reports link | hard-coded `Reports` text | Add `nav.reports` to `locales/en.js` and `locales/ja.js` |

## Workflow

### 1. Preflight
1. Read `client/src/App.vue`, `client/src/main.js`, `client/src/components/FilterBar.vue`, `ProfileMenu.vue`, `LanguageSwitcher.vue`, `client/src/locales/en.js`, `ja.js`.
2. Confirm dev servers: frontend `http://localhost:3000`, backend `http://localhost:8001`. If not running, use `./scripts/start.sh` (servers bind to loopback only; never add `--host`).
3. If the Playwright MCP is available, capture baseline screenshots of `/`, `/inventory`, `/orders`, `/spending`, `/demand`, `/reports` at 1440px and 768px widths.

### 2. Design tokens (App.vue global style)
Add the `:root` block from `reference/design-tokens.md` at the top of the unscoped `<style>` in `App.vue`. Replace hard-coded spacing, radii, shadows and colors in the global classes with tokens. Class names stay the same.

### 3. Sidebar component
Create `client/src/components/AppSidebar.vue` per the sidebar spec in `reference/design-tokens.md`:
- Brand block: `t('nav.companyName')`, `t('nav.subtitle')` beneath (hidden when collapsed).
- Nav items, in this order: Overview `/`, Inventory `/inventory`, Orders `/orders`, Finance `/spending`, Demand Forecast `/demand`, Reports `/reports`. Labels from `nav.*` i18n keys. Each with an inline SVG icon (24x24 viewBox, `stroke="currentColor"`, `aria-hidden="true"`).
- Use `router-link` with `exact-active-class` for `/` and `active-class` for others; rely on router-link's built-in `aria-current="page"`.
- Footer: `LanguageSwitcher` and `ProfileMenu`; forward ProfileMenu's `show-profile-details` and `show-tasks` events up to `App.vue`.
- Collapse toggle; persist state in `localStorage` key `sidebar-collapsed`. Collapsed shows icons only with `title` tooltips.
- Responsive modes (breakpoints in a `BREAKPOINTS` constant, read via `matchMedia` listeners cleaned up on unmount):
  - >= 1280px: full sidebar; user collapse toggle persists to `localStorage`.
  - 768-1279px: always an icons-only rail (`--sidebar-w-collapsed`); toggle expands it as an overlay (no layout push) that closes on route change, Escape, or outside click, and never writes `localStorage`.
  - < 768px: off-canvas drawer, closed by default, opened by a hamburger button in the top bar, closed by backdrop click, Escape, or route change.
- Visible `:focus-visible` outlines on all interactive elements.
- Component style uses Composition API, matching the project pattern (`setup()` returning bindings).

### 4. App shell
Rewrite the `App.vue` template:
```
.app (CSS grid: sidebar | workspace)
├── AppSidebar
└── .workspace
    ├── header.topbar (sticky, top: 0)  -> hamburger (mobile), page title, FilterBar
    └── main.main-content -> .content-inner (max-width: var(--content-max)) -> router-view
```
- Page title in the top bar derives from the current route (map route path to the same `nav.*` key).
- Delete `.top-nav`, `.nav-container`, `.nav-tabs`, `.logo`, `.subtitle` styles.
- Keep `ProfileDetailsModal` and `TasksModal` at the end of the template, unchanged.

### 5. FilterBar alignment
Restyle `FilterBar.vue` so it sits inside the top bar: remove its own sticky/top/max-width/horizontal padding, use token spacing, keep all filter logic and `v-model` bindings untouched.

### 6. Polish pass across views
For each of `client/src/views/*.vue` (style blocks only, plus wrapper markup if a view bypasses the shared classes):
- Ensure the page starts with `.page-header` and content sits in `.card` / `.stats-grid`.
- Replace hard-coded spacing with tokens; align card radius/shadow with tokens.
- Do not change data loading, computed logic, or chart math.

Check every modal still overlays the sidebar (`z-index` above `--z-sidebar`).

### 7. i18n
Add `reports: 'Reports'` to `nav` in `en.js` and the Japanese equivalent (`'レポート'`) in `ja.js`. Use it in the sidebar and top bar title.

### 8. Verify
Run every item in [reference/checklist.md](reference/checklist.md). Then run the `code-reviewer` agent (or a general-purpose agent following `.claude/agents/code-reviewer.md`) on the diff and fix confirmed issues.

## Final report
- Files changed / created.
- Checklist results (pass/fail per item; say which items could not be run, e.g. no Playwright MCP).
- Before/after screenshots if captured.
- Follow-ups: unrouted `Backlog.vue`, anything deferred.
