# IT PMO Kanban — project context

Single-page Kanban board for an internal IT PMO. Demo/training tool only — no real
company logos, trademarks, or imitation of official systems. Local build uses a plain
"UOB IT PMO" text wordmark and a corporate blue palette.

## Files
- `index.html` — the whole app (markup + `<style>` + `<script>`). Opens by double-click.
- Published artifact copy: https://claude.ai/artifact/13YcusnWwWfi7UmTN7MGgy
  (neutral "IT PMO Kanban" wordmark, `ITPM-####` IDs, adds dark-mode tokens). It is a
  separate file generated from `index.html`; republish it after changing the app.

## Hard constraints (do not break)
- Vanilla HTML/CSS/JS only. No frameworks, libraries, build step, npm, CDNs, web fonts or image files.
  System font stack; icons are Unicode/inline SVG.
- One file: `index.html`. Must run from `file://` with no server.
- **No persistence**: state lives in memory only. No localStorage, sessionStorage,
  IndexedDB or cookies. Refresh resets to seed data (the UI notes this).
- Only backend: FormSubmit AJAX endpoint, called with `fetch`, never a plain form POST.

## Architecture
- One source of truth: `state = { tasks, filters, nextId, ui }`. `ui` holds the
  open move menu / delete confirmation.
- Always mutate state, then call `renderBoard(focusSelector?)`. Card contents are never
  changed directly outside `renderBoard()` (only the drag/drop-target classes are toggled directly).
- Key functions: `renderBoard`, `renderCard`, `renderSummary`, `addTask`, `moveTask`,
  `deleteTask`, `applyFilters` (pure), `validateForm`, `showToast`, `notifyNewTask`, `escapeHtml`.
- All user strings go through `escapeHtml()` before being put in HTML. Toasts use `textContent`.
- Board events are delegated on `#board` (click, keydown, dragstart/over/leave/drop/end).
- Dates are local `YYYY-MM-DD` strings (`todayISO()`, `addDaysISO()`). Don't use `toISOString()` for dates.
  Seed due dates are relative to today, so the overdue demo always works.

## Domain rules
- Columns (fixed order): Backlog, In Progress, Blocked, Done.
- Task ID: `UOB-ITPM-####`, zero-padded incrementing counter.
- Priority border/pill colours: Critical red, High amber, Medium blue, Low grey. Pills always show text too.
- Overdue = due date before today and status ≠ Done.
- Delete uses an inline "Delete? Yes / No" inside the card. Never use `confirm()` or `alert()`.
- Keyboard fallback: the "Move ▸" menu on each card; Escape closes the menus.
- Form: title required (≤80 chars), description ≤500, assignee required, due date required and
  not in the past. Errors show inline under each field.
- Optimistic add: the card appears at once; FormSubmit runs in parallel. On failure, show the warning
  toast "Card added locally — email notification failed". The submit button shows "Sending…" while the request runs.

## FormSubmit
- Config: `FORMSUBMIT_ENDPOINT` at the top of the script (placeholder `YOUR_EMAIL@example.com`).
- One-time activation: the first submission sends a confirmation email, and nothing is delivered until
  the link in it is clicked.
- From `file://`, the origin is `null`, and FormSubmit may reject it. Serve locally
  (`python -m http.server`) if needed. The artifact viewer blocks all outbound fetches.
- A failure must never break the board. Never send the user's email anywhere else.

## Code style
- Semantic HTML, `<label for>` on every input, `aria-label` on icon buttons, visible focus rings,
  `aria-live="polite"` toast region.
- CSS custom properties for palette and spacing. No `!important`.
- Small named functions and a comment header for each major section.
- Stacks to one column below 768px; the sidebar stacks above the board below 1100px.
