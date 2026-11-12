# Design tokens and sidebar spec

Paste into the top of the unscoped `<style>` in `client/src/App.vue`.

```css
:root {
  /* Spacing scale (4px base) */
  --space-1: 0.25rem;  /* 4px */
  --space-2: 0.5rem;   /* 8px */
  --space-3: 0.75rem;  /* 12px */
  --space-4: 1rem;     /* 16px */
  --space-6: 1.5rem;   /* 24px */
  --space-8: 2rem;     /* 32px */
  --space-12: 3rem;    /* 48px */

  /* Neutrals (slate) */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-border: #e2e8f0;
  --color-border-strong: #cbd5e1;
  --color-text: #0f172a;
  --color-text-body: #334155;
  --color-text-muted: #64748b;
  --color-hover: #f1f5f9;

  /* Brand and status */
  --color-primary: #2563eb;
  --color-primary-soft: #eff6ff;
  --color-success: #059669;
  --color-warning: #ea580c;
  --color-danger: #dc2626;
  --color-info: #2563eb;

  /* Sidebar (dark slate) */
  --sidebar-bg: #0f172a;
  --sidebar-text: #cbd5e1;
  --sidebar-text-muted: #94a3b8;
  --sidebar-hover: #1e293b;
  --sidebar-active-bg: #1e293b;
  --sidebar-active-text: #ffffff;
  --sidebar-active-indicator: #3b82f6;

  /* Shape */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.04);
  --shadow-md: 0 4px 12px rgba(15, 23, 42, 0.06);
  --shadow-lg: 0 12px 32px rgba(15, 23, 42, 0.12);

  /* Layout */
  --sidebar-w: 248px;
  --sidebar-w-collapsed: 72px;
  --topbar-h: 60px;
  --content-max: 1440px;

  /* Layering: modals already use their own higher z-index; keep them above these */
  --z-topbar: 150;
  --z-sidebar: 200;
  --z-backdrop: 190;

  --transition: 0.2s ease;
}
```

## Global class targets

| Class | Target |
|---|---|
| `.main-content` | padding `var(--space-6) var(--space-8)`; inner `.content-inner` max-width `var(--content-max)` |
| `.page-header` | margin-bottom `var(--space-6)`; h2 1.5rem/700, p muted 0.875rem |
| `.stats-grid` | gap `var(--space-4)`; margin-bottom `var(--space-6)`; `minmax(240px, 1fr)` |
| `.stat-card`, `.card` | surface bg, 1px border, radius `--radius-lg`, padding `var(--space-6)`, `--shadow-sm`; hover `--shadow-md` (stat-card only) |
| `.card-header` | margin-bottom `var(--space-4)`; padding-bottom `var(--space-3)` |
| `th` | bg `--color-bg`, muted text, 0.75rem uppercase, padding `var(--space-3) var(--space-4)` |
| `td` | padding `var(--space-3) var(--space-4)` |
| `.badge` | radius 999px (pill), padding `var(--space-1) var(--space-3)`, no uppercase, 0.75rem/600 |

## Sidebar spec

- Width `var(--sidebar-w)`, collapsed `var(--sidebar-w-collapsed)`, animated with `var(--transition)`.
- Full height, `position: sticky; top: 0; height: 100vh`, dark `--sidebar-bg`, `z-index: var(--z-sidebar)`.
- Brand block: height `var(--topbar-h)`, padding `0 var(--space-4)`, bottom border `rgba(255,255,255,0.06)`. Company name 1rem/700 white; subtitle 0.75rem `--sidebar-text-muted`.
- Nav: padding `var(--space-3)`; items 40px tall, gap `var(--space-3)` between icon and label, radius `--radius-md`, color `--sidebar-text`.
  - Hover: bg `--sidebar-hover`, text white.
  - Active: bg `--sidebar-active-bg`, text `--sidebar-active-text`, 3px left indicator `--sidebar-active-indicator`.
- Icons: 20px, stroke 1.75, `currentColor`.
- Footer: pinned to bottom, top border, holds LanguageSwitcher and ProfileMenu; dropdowns open upward and must not be clipped (`overflow: visible` on the sidebar footer).
- Collapse toggle: icon button in the footer or brand row; collapsed hides labels, subtitle, and profile name, keeps icons centered with `title` tooltips.
- Tablet (768-1279px): icons-only rail, grid column `var(--sidebar-w-collapsed)`; expanded state overlays content at full `--sidebar-w` with `--shadow-lg`.
- Mobile (< 768px): sidebar `position: fixed; transform: translateX(-100%)`, open state `translateX(0)`; backdrop `rgba(15,23,42,0.4)` at `--z-backdrop`; grid collapses to a single column.

## Top bar spec

- `position: sticky; top: 0; z-index: var(--z-topbar)`; surface bg, bottom border, `--shadow-sm`.
- Row 1 (height `var(--topbar-h)`): hamburger (mobile only), page title 1.125rem/600.
- Row 2: FilterBar, padding `var(--space-3) var(--space-8)`, no own max-width.
