# Acceptance checklist

Mark each item pass / fail / not run (with reason).

## Build
- [ ] `cd client && npm run build` succeeds with no errors.
- [ ] No new dependencies in `client/package.json`.

## Layout (Playwright, routes `/`, `/inventory`, `/orders`, `/spending`, `/demand`, `/reports`)
- [ ] 1440px: sidebar visible on the left, top nav gone, active route highlighted.
- [ ] 1024px: icons-only rail always visible; toggle expands it as an overlay; route change, Escape, and outside click collapse it; desktop `localStorage` preference unchanged.
- [ ] No horizontal scroll at any width.
- [ ] 768px: sidebar hidden; hamburger opens drawer; backdrop click, Escape, and route change close it.
- [ ] Collapse toggle shrinks sidebar to icons; state survives a page reload.
- [ ] Top bar and FilterBar stay sticky while scrolling; nothing overlaps them.
- [ ] Page title in top bar matches the current route.

## Components
- [ ] ProfileMenu dropdown fully visible (not clipped) and its items open Profile Details and Tasks modals.
- [ ] LanguageSwitcher dropdown fully visible; switching to Japanese updates sidebar labels and top bar title.
- [ ] All modals (Tasks, Profile Details, Inventory/Product/Cost/Backlog detail) render above the sidebar and backdrop.
- [ ] Spending view sticky element does not hide under the top bar.

## Behavior unchanged
- [ ] Changing each filter updates data on Dashboard and Orders.
- [ ] No errors in the browser console on any route (pre-existing task API errors excepted; note them).
- [ ] `cd server && uv run pytest ../tests/backend -q` passes. (Run from `server/`: the `tests/` dir has no uv environment with pytest.)

## Code quality
- [ ] No emojis in UI.
- [ ] Global class names preserved.
- [ ] Non-obvious changes commented.
- [ ] v-for keys are unique IDs, not indexes.
- [ ] Inline SVG icons have `aria-hidden="true"`; nav uses `aria-current`.
