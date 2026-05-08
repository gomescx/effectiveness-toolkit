# Tasks: TMM Tags, Size & Report

**Feature branch**: `002-tmm-tags-size-report`
**Input**: `specs/002-tmm-tags-size-report/` — plan.md, spec.md, ux-spec.md, data-model.md, contracts/project-file-schema.md, research.md, quickstart.md
**Only file modified**: `public/tools/tmm/index.html`
**Tests**: Manual only (Constitution VII standalone HTML exception — no automated tests required)

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (independent edits within the single file, no cross-task blocking)
- **[Story]**: Plan phase this task belongs to (A1–A4, B1–B4)
- All tasks modify `public/tools/tmm/index.html` unless noted otherwise

---

## Phase 1: Setup (Read Existing File + Scaffold State)

**Purpose**: Understand the current implementation and introduce the new in-memory state variables before any structural changes.

- [X] T001 Read `public/tools/tmm/index.html` in full; identify the locations of: toolbar, canvas-wrapper, detail panel, modal, `submitModal()`, `renderCanvas()`, `saveToFile()`, `loadFromFile()`, and all existing `var` declarations in the `<script>` block
- [X] T002 Add new state variables to the top of the `<script>` block in `public/tools/tmm/index.html`: `var isDirty = false;`, `var isDirtyPanel = false;`, `var editSnapshot = null;`, `var inboxItems = [];`, `var activeTag = null;`, `var currentView = 'canvas';`

**Checkpoint**: State scaffold in place — implementation phases can begin

---

## Phase 2: Foundational (Blocking Helpers)

**Purpose**: Shared utility functions and the `setDirty()` helper that all subsequent phases depend on. Must be complete before Phase A3, Phase B, and any save/load work.

- [X] T003 Add `deriveQuadrant(x, y)` helper function to `<script>` block in `public/tools/tmm/index.html`: returns `'Q1 — Firefighting'` / `'Q2 — Growth'` / `'Q3 — Distraction'` / `'Q4 — Waste'` per data-model.md quadrant table
- [X] T004 Add `validateTags(input)` helper function to `<script>` block in `public/tools/tmm/index.html`: trims input, splits on whitespace, filters empty tokens, tests each against `^#\S+$`; returns `{ tokens, invalid, valid }` per quickstart.md reference
- [X] T005 Add `applyDotSize(dotEl, size)` helper function to `<script>` block in `public/tools/tmm/index.html`: applies `width`/`height` from `SIZE_DIAMETER` map `{ XS:16, S:24, M:32, L:44, XL:58 }`; for `null` size sets transparent background + `var(--color-primary)` border; for sized sets `var(--color-primary)` background + white border; per quickstart.md reference
- [X] T006 Add `setDirty(val)` helper function to `<script>` block in `public/tools/tmm/index.html`: sets `isDirty = val`; shows `#unsaved-banner` when `val === true`; hides it when `val === false`

**Checkpoint**: All shared helpers available — all plan phases can now proceed

---

## Phase 3: A1 — Toolbar & Layout Restructure

**Story**: A1 — Toolbar & layout restructure (UX fix — breaking change to 001-tmm)

**Goal**: Replace the old toolbar with the new layout; scaffold the flex wrapper for Inbox + canvas + detail panel; add unsaved-changes banner placeholder.

**Independent Test**: Open `public/tools/tmm/index.html` in a browser. Confirm toolbar shows `[+ Quick Add]`, `[📊 Report]`, `[💾 Save]`, `[📂 Load]`, and a tag filter `<select>`. Confirm a red banner element exists between toolbar and canvas (hidden by default). Confirm `#canvas-wrapper` is a flex row with `#inbox-panel`, `#canvas-area`, and `#detail-panel` as direct children. No functionality needed — structural check only.

- [X] T007 [A1] In `public/tools/tmm/index.html`, replace the existing toolbar HTML with the new toolbar: keep `[💾 Save]` and `[📂 Load]` buttons; replace `[+ Add Initiative]` with `<button id="btn-quick-add">[+ Quick Add]</button>`; add `<button id="btn-report">[📊 Report]</button>`; add tag filter `<select id="tag-filter"><option value="">(all tags)</option></select>` and a hidden `<button id="btn-clear-filter" style="display:none">[× Clear]</button>`
- [X] T008 [A1] In `public/tools/tmm/index.html`, add `<div id="unsaved-banner" style="display:none; background:#e53e3e; color:#fff; padding:8px 16px;">Changes not saved — click 💾 Save to preserve your edits</div>` immediately after the toolbar and before `#canvas-wrapper`
- [X] T009 [A1] In `public/tools/tmm/index.html`, restructure `#canvas-wrapper` as a CSS flex row: add `<aside id="inbox-panel">` as the first child (collapsed by default showing `<button id="btn-inbox-expand">[▶ 0]</button>`); ensure the existing canvas `<div>` is wrapped in `<div id="canvas-area">` as the middle child; add `<div id="detail-panel" style="display:none">` as the right child (replaces any existing modal or fixed-position detail panel)
- [X] T010 [A1] In `public/tools/tmm/index.html`, add `<section id="report-view" style="display:none">` after `#canvas-wrapper`; inside it: a toolbar with `<button id="btn-export-md">[Export Markdown]</button>` and `<button id="btn-back-canvas">[← Back to Canvas]</button>`, the unsaved banner slot, and a `<table id="report-table">` with `<thead>` containing columns: `☐`, Q, Name, Tags, Size, Details

**Checkpoint**: Structural scaffold complete — A2, A4, B4 can now build on the HTML structure

---

## Phase 4: A2 — Inbox Panel (Bulk Add + List + Collapse)

**Story**: A2 — Inbox panel lifecycle (UX fix)

**Goal**: Full Inbox panel — bulk add entry form, list mode, collapse/expand, and `[Done]` button. New initiatives go to `inboxItems[]` + `initiatives[]` with `x=null, y=null`.

**Independent Test**: Open the tool. Click `[+ Quick Add]` — confirm Inbox opens in bulk add mode with a form showing Name, Tags, Size. Type a name, press Enter — confirm the initiative appears in the Inbox list above the form and the count badge increments. Click `[Done]` — confirm bulk add mode ends, form is hidden, list remains visible. Click `[─]` — confirm Inbox collapses to `[▶ N]`. Click the badge to expand. No canvas dot appears for any unpositioned initiative.

- [X] T011 [A2] In `public/tools/tmm/index.html`, build the Inbox panel HTML inside `#inbox-panel`: add a header area with title `INBOX (<span id="inbox-count">0</span>)`, a `<button id="btn-done-add">[Done]</button>` (hidden initially), and a `<button id="btn-inbox-collapse">[─]</button>`; add `<ul id="inbox-list"></ul>` for the initiative rows; add `<div id="inbox-form">` containing `<input id="inbox-name" placeholder="Initiative name">`, `<input id="inbox-tags" placeholder="Tags (e.g. #health)">`, `<select id="inbox-size">` with options blank/XS/S/M/L/XL, and hint text "Tab · Enter to add"
- [X] T012 [A2] In `public/tools/tmm/index.html`, implement `openInbox()`: show `#inbox-panel` in bulk add mode (show `#inbox-form`, show `#btn-done-add`, hide `#btn-inbox-collapse`); focus `#inbox-name`; wire `[+ Quick Add]` button click to `openInbox()`
- [X] T013 [A2] In `public/tools/tmm/index.html`, implement inbox entry form submission: on `keydown` in `#inbox-name`, `#inbox-tags`, or `#inbox-size` — Tab moves focus Name→Tags→Size→Name; Enter on any field calls `submitInboxEntry()`; `submitInboxEntry()` reads Name (required — skip if empty), Tags (optional, validate with `validateTags()` — show inline error below `#inbox-tags` if invalid and abort), Size (optional); creates initiative object with `id = uuid()`, `name`, `tmm: { tags, size, details:'', x:null, y:null }`, pushes to `initiatives[]` and `inboxItems[]`; calls `renderInbox()`; clears form; refocuses `#inbox-name`; calls `setDirty(true)`
- [X] T014 [A2] In `public/tools/tmm/index.html`, implement `renderInbox()`: rebuild `#inbox-list` from `inboxItems[]`; each row contains drag handle `<span class="drag-handle">⠿</span>`, initiative name text, and `<button class="btn-inbox-remove" data-id="[id]">[×]</button>`; update `#inbox-count` and the collapse badge; wire `[×]` buttons to remove the initiative from both `inboxItems[]` and `initiatives[]` immediately (no confirmation) and call `renderInbox()` + `setDirty(true)`
- [X] T015 [A2] In `public/tools/tmm/index.html`, implement `[Done]` button: hides `#inbox-form`, hides `#btn-done-add`, shows `#btn-inbox-collapse`; switches Inbox to list mode; implement `[─]` collapse: hides list and header, shows only the `[▶ N]` badge button; implement `[▶ N]` expand: restores full Inbox panel; update `renderCanvas()` to skip any initiative where `initiative.tmm.x === null` (do not render unpositioned initiatives as dots)

**Checkpoint**: Inbox bulk-add, list mode, and collapse are fully functional

---

## Phase 5: A3 — Inbox Drag-to-Canvas

**Story**: A3 — Inbox drag-to-canvas (UX fix)

**Goal**: Drag handle `⠿` on each Inbox row enables Pointer Events drag onto the canvas; successful drop positions the initiative and removes it from the Inbox.

**Independent Test**: With an initiative in the Inbox, grab its `⠿` handle and drag it onto the canvas. Confirm: (1) a semi-transparent ghost follows the pointer during drag; (2) on drop within the canvas, a dot appears at the drop position and the Inbox row disappears; (3) the Inbox count badge decrements; (4) on drop outside the canvas, the ghost disappears and the Inbox row remains unchanged.

- [X] T016 [A3] In `public/tools/tmm/index.html`, implement `pointerdown` listener on `#inbox-list` (delegated to `.drag-handle` elements): record the dragged initiative `id`; create a ghost `<div id="drag-ghost">` styled as a semi-transparent dot (`opacity:0.5; pointer-events:none; position:fixed; width:32px; height:32px; background:var(--color-primary); border-radius:50%`); append ghost to `<body>`; call `setPointerCapture` on the drag handle element
- [X] T017 [A3] In `public/tools/tmm/index.html`, implement `pointermove` listener: move ghost `<div>` to follow pointer coordinates (`ghost.style.left = e.clientX - 16 + 'px'; ghost.style.top = e.clientY - 16 + 'px'`)
- [X] T018 [A3] In `public/tools/tmm/index.html`, implement `pointerup` listener: remove ghost from DOM; get canvas bounding rect via `document.getElementById('canvas-area').getBoundingClientRect()`; if pointer coordinates are within canvas rect, compute normalised `x = (e.clientX - rect.left) / rect.width` and `y = 1 - (e.clientY - rect.top) / rect.height`; clamp both to `[0, 1]`; find the initiative in `initiatives[]` by id; set `initiative.tmm.x = x` and `initiative.tmm.y = y`; remove the initiative from `inboxItems[]`; call `renderInbox()` and `renderCanvas()`; call `setDirty(true)`; if pointer is outside canvas rect, do nothing (cancel drag silently)

**Checkpoint**: Inbox drag-to-canvas is fully functional

---

## Phase 6: A4 — Detail Panel Redesign

**Story**: A4 — Detail panel redesign: always-edit, dirty-flag, Tags + Size placeholders, Delete (UX fix)

**Goal**: Detail panel opens always in edit mode; `[Close]` gated by `isDirtyPanel`; Category replaced by Tags + Size (HTML only — functional validation in B1/B2); Delete with confirmation.

**Independent Test**: Click a canvas dot — confirm the detail panel opens immediately in edit mode with cursor in the Name field. Edit the Name field — confirm `[Close]` becomes visually disabled. Click `[Cancel]` — confirm changes are reverted and the panel closes. Click the dot again, make no changes, click `[Close]` — confirm panel closes. Click the dot, click `[Delete]`, confirm the confirmation dialog appears; click `[Cancel]` — confirm initiative is unchanged; click `[Delete]` again, click `[Yes, delete]` — confirm the dot disappears. Confirm no Category field is visible; Tags and Size fields are present.

- [X] T019 [A4] In `public/tools/tmm/index.html`, rewrite the detail panel HTML inside `#detail-panel`: header with title "INITIATIVE" and `<button id="btn-panel-close">[Close]</button>`; fields: Name `<input id="edit-name">`, Tags `<input id="edit-tags" placeholder="e.g. #health #career">` with `<span id="edit-tags-error" style="display:none"></span>`, Size `<select id="edit-size">` with options blank/XS/S/M/L/XL, Details `<textarea id="edit-details"></textarea>`; action row: `<button id="btn-panel-cancel">[Cancel]</button>` and `<button id="btn-panel-confirm">[Confirm]</button>`; delete trigger `<button id="btn-panel-delete">[Delete]</button>` and inline confirmation `<div id="delete-confirm" style="display:none">Delete this initiative? This cannot be undone. <button id="btn-delete-cancel">[Cancel]</button> <button id="btn-delete-yes">[Yes, delete]</button></div>` (remove any old modal HTML)
- [X] T020 [A4] In `public/tools/tmm/index.html`, implement `openDetailPanel(id)`: show `#detail-panel`; set `selectedId = id`; populate fields from `initiatives[i]` (name, tags as space-separated string, size, details); set `editSnapshot = { name, tags, size, details }`; set `isDirtyPanel = false`; apply `opacity:1; cursor:pointer; pointer-events:auto` to `#btn-panel-close`; focus `#edit-name`; remove any previous `onclick` wired to canvas dots and wire each dot's click to call `openDetailPanel(id)` or to close the panel if `!isDirtyPanel && selectedId === id`
- [X] T021 [A4] In `public/tools/tmm/index.html`, implement dirty-flag tracking: add `input` and `change` event listeners to `#edit-name`, `#edit-tags`, `#edit-size`, `#edit-details`; on first event set `isDirtyPanel = true` and apply disabled style to `#btn-panel-close` (`opacity:0.4; cursor:not-allowed; pointer-events:none`); wire `#btn-panel-close` click: if `!isDirtyPanel` call `closeDetailPanel()`; if `isDirtyPanel` do nothing (button is `pointer-events:none`)
- [X] T022 [A4] In `public/tools/tmm/index.html`, implement `closeDetailPanel()`: hide `#detail-panel`; clear `selectedId = null`; reset `isDirtyPanel = false`; implement `#btn-panel-cancel` click: restore fields from `editSnapshot`, close panel; implement canvas background click handling — remove any existing "click blank canvas closes panel" behaviour (clicking blank canvas space must NOT close the panel per FR-UX-010); tab order in panel: Name → Tags → Size → Details → Confirm → Name
- [X] T023 [A4] In `public/tools/tmm/index.html`, implement `confirmEdit()`: read Name (required — show inline error and abort if empty), Tags (placeholder — store raw string as array split on whitespace for now; validation wired in B1), Size (store selected value or null), Details; update `initiatives[i]` fields; set `isDirtyPanel = false`; call `renderCanvas()`; call `setDirty(true)`; close panel; wire `#btn-panel-confirm` to `confirmEdit()`
- [X] T024 [A4] In `public/tools/tmm/index.html`, implement Delete flow: `#btn-panel-delete` click shows `#delete-confirm`; `#btn-delete-cancel` hides `#delete-confirm`; `#btn-delete-yes` removes the initiative from `initiatives[]` by `selectedId`, calls `renderCanvas()`, calls `setDirty(true)`, calls `closeDetailPanel()`

**Checkpoint**: Layer A complete — detail panel always-edit with dirty-flag gating and delete confirmed working

---

## Phase 7: B1 — Tags Field Validation and Canvas Filter

**Story**: B1 — Tags: `#token` validation + canvas filter (US6, P1)

**Goal**: `validateTags()` wired into `confirmEdit()` and Inbox form; tag filter `<select>` populated from live initiatives; filter indicator with `[× Clear]`; `saveToFile()`/`loadFromFile()` updated for `tags`; empty-canvas filter message.

**Independent Test**: Open tool. Add initiative A tagged `#health #career` and initiative B tagged `#finance` (via Inbox drag-to-canvas). Click dot A, set Tags to `#health #career`, Confirm — no error. Click dot A again, set Tags to `career` (no `#`), Confirm — confirm inline error shown and save blocked. Select `#health` in filter dropdown — confirm only initiative A dot is visible and a `[× Clear]` button appears. Clear filter — both dots return. Save → reload → click dot A — confirm `#health #career` reads back exactly.

- [ ] T025 [B1] [US6] In `public/tools/tmm/index.html`, wire `validateTags()` into `confirmEdit()`: after reading the Tags input value, call `validateTags(value)`; if `!result.valid` show inline error in `#edit-tags-error` ("Use # prefix — e.g. #health") and abort save; if valid store `result.tokens` as the `tags` array on the initiative; hide `#edit-tags-error` on valid
- [ ] T026 [B1] [US6] In `public/tools/tmm/index.html`, wire `validateTags()` into `submitInboxEntry()` (A2 integration — replace the placeholder validation with the real helper): show inline error below `#inbox-tags`; abort if invalid; store `result.tokens` on valid
- [ ] T027 [B1] [US6] In `public/tools/tmm/index.html`, implement tag filter population in `renderCanvas()`: collect union of all `tags` arrays from `initiatives[]`; rebuild `#tag-filter` `<option>` elements (first option always `(all tags)` with value `""`); preserve currently selected `activeTag` value if still present in the union
- [ ] T028 [B1] [US6] In `public/tools/tmm/index.html`, implement filter state: wire `#tag-filter` `change` event to set `activeTag = selectedValue || null` and call `renderCanvas()`; wire `#btn-clear-filter` click to set `activeTag = null`, reset `#tag-filter` to `""`, hide `#btn-clear-filter`, call `renderCanvas()`; in `renderCanvas()` skip rendering any dot where `activeTag !== null && !initiative.tmm.tags.includes(activeTag)`; show `#btn-clear-filter` when `activeTag !== null`, hide it otherwise
- [ ] T029 [B1] [US6] In `public/tools/tmm/index.html`, add empty-canvas filter message: when `activeTag !== null` and no dots are rendered, show a centred `<div id="filter-empty-msg">` with text "No initiatives match this filter" over the canvas; hide it otherwise
- [ ] T030 [B1] [US6] In `public/tools/tmm/index.html`, update `saveToFile()`: write `tags` array to `initiative.tmm.tags`; do NOT write `category` field
- [ ] T031 [B1] [US6] In `public/tools/tmm/index.html`, update `loadFromFile()`: for each initiative, read `tmm.tags` (default `[]` if absent or not an array); silently ignore `tmm.category`; reset `activeTag = null` on every load (FR-000.06); call `setDirty(false)`
- [ ] T032 [B1] [US6] In `public/tools/tmm/index.html`, update `openDetailPanel()` populate step: read `initiative.tmm.tags` (array), join with `' '` for display in `#edit-tags`

**Checkpoint**: US6 complete — tags, validation, filter, and save/load are independently testable

---

## Phase 8: B2 — T-Shirt Sizing (Dot Visual + Detail Panel)

**Story**: B2 — T-shirt sizing: size-proportional dots, hollow null dot (US7, P2)

**Goal**: Size `<select>` in detail panel wired to `confirmEdit()`; `applyDotSize()` called in `renderCanvas()` for every dot; `saveToFile()`/`loadFromFile()` updated for `size`; Inbox form Size field also wired.

**Independent Test**: Add two initiatives — assign XS to one and XL to the other. View the canvas — confirm XL dot is visually larger. Add a third with no size — confirm its dot is hollow at M diameter (32px). Save; reload — confirm all three sizes restore correctly and dots render at the expected diameters. Open detail panel for the XS initiative — confirm XS is pre-selected in the Size dropdown.

- [ ] T033 [B2] [US7] In `public/tools/tmm/index.html`, wire the Size `<select>` (`#edit-size`) in the detail panel: `confirmEdit()` reads `#edit-size` value; stores `value || null` as `initiative.tmm.size`; calls `renderCanvas()` (already done in A4 — verify the size field is read and stored correctly)
- [ ] T034 [B2] [US7] In `public/tools/tmm/index.html`, call `applyDotSize(dotEl, initiative.tmm.size)` for every dot rendered in `renderCanvas()` (replaces any previous fixed-diameter style on dots)
- [ ] T035 [B2] [US7] In `public/tools/tmm/index.html`, update `openDetailPanel()` populate step: set `#edit-size` value to `initiative.tmm.size || ''` so the correct option is pre-selected on panel open
- [ ] T036 [B2] [US7] In `public/tools/tmm/index.html`, update `saveToFile()`: write `size` field to `initiative.tmm.size` (may be `null`)
- [ ] T037 [B2] [US7] In `public/tools/tmm/index.html`, update `loadFromFile()`: read `tmm.size`; validate against enum `['XS','S','M','L','XL']`; default to `null` if absent or invalid
- [ ] T038 [B2] [US7] In `public/tools/tmm/index.html`, ensure `submitInboxEntry()` in the Inbox form reads `#inbox-size` value and stores `value || null` as `initiative.tmm.size` (this was scaffolded in A2/T013 — verify size is stored and not discarded)

**Checkpoint**: US7 complete — size-proportional dots, hollow null dot, save/load all independently testable

---

## Phase 9: B3 — Unsaved Changes Banner

**Story**: B3 — Unsaved changes banner (cross-cutting, FR-UX-013)

**Goal**: `setDirty()` wired to all mutation points; banner appears on any edit and clears on save/load; also visible in Report view.

**Independent Test**: Open the tool — confirm no banner visible. Add an initiative via Quick Add and drag it to canvas — confirm red banner appears. Click `[💾 Save]` — confirm banner disappears. Reload the file — confirm banner is absent. Make any edit via detail panel Confirm — confirm banner reappears. Switch to Report view — confirm banner is still visible there.

- [ ] T039 [B3] In `public/tools/tmm/index.html`, verify `#unsaved-banner` HTML is in place (added in T008) and `setDirty()` helper is implemented (added in T006); audit all mutation call sites and confirm `setDirty(true)` is called for: `submitInboxEntry()`, inbox `[×]` remove, successful `pointerup` drop onto canvas (drag-to-canvas), `confirmEdit()`, `[Yes, delete]` (delete initiative), and any canvas dot drag-reposition; confirm `setDirty(false)` is called in `saveToFile()` on success and in `loadFromFile()` on success

**Checkpoint**: B3 complete — banner wiring verified across all mutation points

---

## Phase 10: B4 — Report View (Spreadsheet Editor + Bulk Tag + Markdown Export)

**Story**: B4 — Report view: spreadsheet editor, bulk tag, markdown export (US8, P3)

**Goal**: Full Report view accessible from toolbar; `renderReportView()` populates table from `initiatives[]`; inline cell editing for Name/Tags/Size/Details (Q read-only); keyboard navigation (arrows, Tab, Enter, Esc); checkbox per row + bulk tag apply with deduplication; Export Markdown download; Back to Canvas restores updated canvas state.

**Independent Test**: With five initiatives on the canvas, click `[📊 Report]` — confirm all five appear with correct Q, Name, Tags, Size, Details. Navigate cells with arrow keys — confirm Q cells are skipped. Press Enter on a Name cell — confirm edit mode activates with cursor. Type a new name, press Tab — confirm change is committed, canvas shows updated name on return. Navigate to a Tags cell, press Enter, type `career` (no `#`), press Tab — confirm inline error shown and change blocked. Select two rows via checkbox, enter `#reviewed` in bulk tag input, click "Apply to selected" — confirm tag is added to both initiatives; switch back to canvas, open each dot — confirm `#reviewed` is present. Click "Export Markdown" — confirm download triggers; open the `.md` file and confirm correct GFM table structure. With no rows selected, click "Apply to selected" — confirm "No initiatives selected" message shown.

- [ ] T040 [B4] [US8] In `public/tools/tmm/index.html`, implement `currentView` toggle: wire `#btn-report` click to set `currentView = 'report'`, hide `#canvas-wrapper`, show `#report-view`, call `renderReportView()`; wire `#btn-back-canvas` click to set `currentView = 'canvas'`, hide `#report-view`, show `#canvas-wrapper`, call `renderCanvas()` (canvas immediately reflects all report edits)
- [ ] T041 [B4] [US8] In `public/tools/tmm/index.html`, implement `renderReportView()`: clear and rebuild `#report-table` `<tbody>` from `initiatives[]`; for each initiative create a `<tr>` with: `<td>` containing `<input type="checkbox" class="row-checkbox" data-id="[id]">`; Q cell `<td class="q-cell">` containing `deriveQuadrant(x,y)` short label (Q1/Q2/Q3/Q4 prefix only — e.g. "Q1"); Name, Tags, Size, Details cells as `<td class="editable-cell" tabindex="0" data-field="[field]" data-id="[id]">` containing the current value; handle empty-initiatives case — show single colspan row with "No initiatives yet — go back to the canvas to add your first initiative."
- [ ] T042 [B4] [US8] In `public/tools/tmm/index.html`, implement keyboard navigation on `#report-table`: single `keydown` event listener on the table; track `focusedCell` (row index, col index — Q col excluded from navigation); Arrow Up/Down moves row; Arrow Left/Right and Tab/Shift+Tab moves column (skip Q column); update `tabindex` and call `.focus()` on the target cell; Enter on a non-editing cell calls `activateCellEdit(cell)`
- [ ] T043 [B4] [US8] In `public/tools/tmm/index.html`, implement `activateCellEdit(cell)`: determine field from `cell.dataset.field`; save `cellSnapshot = cell.textContent`; for Name, Tags, Details: replace cell content with `<input type="text">` (or `<textarea>` for Details) pre-populated with current value; for Size: replace with `<select>` pre-populated with current value; bind Esc keydown to restore `cellSnapshot` and exit edit; bind Tab and Enter keydown (and blur for Size select) to call `commitCellEdit(cell)`
- [ ] T044 [B4] [US8] In `public/tools/tmm/index.html`, implement `commitCellEdit(cell)`: read edited value; for Name: trim + reject empty (show inline error below cell "Name is required"; abort); for Tags: call `validateTags()`; reject on invalid (show inline error below cell "Use # prefix — e.g. #health"; abort); for Size: `value || null`; for Details: raw string; update `initiatives[i]` field by `cell.dataset.id`; hide error if shown; call `setDirty(true)`; restore cell to text display with new value; advance focus to next editable cell in same row (or first editable cell of next row from last column); initiative ID unchanged (FR-US8.04d)
- [ ] T045 [B4] [US8] In `public/tools/tmm/index.html`, implement click-to-edit on editable cells: add `click` event listener on `#report-table` delegated to `.editable-cell`; call `activateCellEdit(cell)` on click (same as Enter key activation)
- [ ] T046 [B4] [US8] In `public/tools/tmm/index.html`, implement select-all checkbox in `#report-table` `<thead>`: toggling the header checkbox selects/deselects all `.row-checkbox` inputs in the current `<tbody>`
- [ ] T047 [B4] [US8] In `public/tools/tmm/index.html`, implement bulk tag apply: wire `#btn-bulk-apply` click to: collect all checked `.row-checkbox` data-ids; if none checked, show inline message `#bulk-tag-msg` "No initiatives selected" and abort; validate `#bulk-tag-input` value as a single `#token` using `validateTags()`; if invalid show error in `#bulk-tag-msg` "Use # prefix — e.g. #reviewed" and abort; for each checked id, append the tag to `initiative.tmm.tags` only if not already present (deduplication); call `setDirty(true)`; call `renderReportView()` to refresh the table
- [ ] T048 [B4] [US8] In `public/tools/tmm/index.html`, implement `exportMarkdown()`: build GFM table string with header `| Quadrant | Name | Tags | Size | Details |`, separator `|---|---|---|---|---|`, and one row per initiative in `initiatives[]` order; Quadrant = `deriveQuadrant(x,y)`; Tags = `tags.join(' ')`; Size = `size || ''` (never string "null"); Details = raw details with `|` escaped as `\|`; create `Blob` with `type: 'text/markdown'`; create object URL; trigger download as `effectiveness-toolkit-tmm-report.md` via `<a>` click; revoke object URL; wire `#btn-export-md` click to `exportMarkdown()`

**Checkpoint**: US8 complete — Report view, bulk tag, and markdown export are independently testable

---

## Phase 11: Polish & Cross-Cutting Concerns

**Purpose**: Integration verification, empty states, and quickstart smoke test

- [ ] T049 [P] In `public/tools/tmm/index.html`, verify the empty canvas state: when `initiatives[]` is empty and no filter is active, confirm the centred hint "Load a matrix or click [+ Quick Add] to get started" is visible over the canvas (FR-UX-001); `#inbox-panel` shows `[▶ 0]` by default
- [ ] T050 [P] In `public/tools/tmm/index.html`, audit all `renderCanvas()` call sites to confirm: (1) `activeTag` filter is applied; (2) `applyDotSize()` is called for every dot; (3) unpositioned initiatives (`x === null`) are skipped; (4) tag filter `<select>` is repopulated; (5) `#btn-clear-filter` visibility is correct
- [ ] T051 [P] In `public/tools/tmm/index.html`, verify state sync: confirm `initiatives[]` is the single source of truth for both canvas and report views — no state is read from DOM elements or table rows; confirm returning to canvas from report view always calls `renderCanvas()` and shows all edits immediately (FR-000.07, FR-US8.09)
- [ ] T052 In `public/tools/tmm/index.html`, run the quickstart.md manual smoke test sequence end-to-end: open file in browser → verify no console errors → Quick Add initiative → verify Inbox → drag to canvas → verify dot → click dot → edit Name + Tags + Size → Confirm → verify canvas → open Report view → verify row → Export Markdown → verify download → Back to Canvas

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies — start immediately
- **Phase 2 (Foundational)**: Depends on Phase 1 — BLOCKS A3, B1, B2, B3, B4
- **Phase 3 (A1)**: Depends on Phase 2 — structural HTML scaffold for all other phases
- **Phase 4 (A2)**: Depends on Phase 3 (A1 HTML structure)
- **Phase 5 (A3)**: Depends on Phase 4 (A2 Inbox list rows must exist)
- **Phase 6 (A4)**: Depends on Phase 3 (A1 detail panel slot must exist); can proceed in parallel with A2/A3
- **Phase 7 (B1)**: Depends on Phase 4 (A2 Inbox submit) and Phase 6 (A4 `confirmEdit()`)
- **Phase 8 (B2)**: Depends on Phase 6 (A4 detail panel Size field HTML); can proceed after A4
- **Phase 9 (B3)**: Depends on Phase 2 (`setDirty()` helper); audit pass after B1/B2/B4 are done
- **Phase 10 (B4)**: Depends on Phase 7 (B1 `validateTags()`) and Phase 2 (`deriveQuadrant()`)
- **Phase 11 (Polish)**: Depends on all previous phases complete

### Layer A → Layer B dependency

All Layer A phases (A1–A4) must be functionally complete before Layer B (B1–B4) builds on the new UI structures.

### Within Each Phase

- Tasks within a phase are sequential unless marked `[P]`
- T003, T004, T005, T006 in Phase 2 are independent and can be written in any order (all helpers, no cross-dependency)
- T007–T010 in Phase 3 are sequential (each step builds on the previous HTML change)

### Story completion order for MVP scope

- **MVP (A phases only)**: Phases 1–6 deliver the UX fixes with stub Tags/Size fields
- **US6 increment (B1)**: Adds full Tags validation + canvas filter
- **US7 increment (B2)**: Adds Size-proportional dot rendering
- **US8 increment (B4 + B3)**: Adds Report view, banner wiring, markdown export

---

## Parallel Execution Notes

Because all tasks edit a single file (`public/tools/tmm/index.html`), true parallelism is not applicable. The `[P]` markers indicate tasks that are **logically independent** within a phase and can be written in any order if a developer is interleaving changes. They do not imply simultaneous file edits.

---

## Task Summary

| Phase | Tasks | Story |
|-------|-------|-------|
| Phase 1: Setup | T001–T002 | — |
| Phase 2: Foundational | T003–T006 | — |
| Phase 3: A1 Toolbar & Layout | T007–T010 | A1 |
| Phase 4: A2 Inbox Panel | T011–T015 | A2 |
| Phase 5: A3 Drag-to-Canvas | T016–T018 | A3 |
| Phase 6: A4 Detail Panel | T019–T024 | A4 |
| Phase 7: B1 Tags + Filter | T025–T032 | US6 |
| Phase 8: B2 T-Shirt Sizing | T033–T038 | US7 |
| Phase 9: B3 Unsaved Banner | T039 | — |
| Phase 10: B4 Report View | T040–T048 | US8 |
| Phase 11: Polish | T049–T052 | — |
| **Total** | **52 tasks** | |

**Parallel opportunities**: T003–T006 (Phase 2 helpers), T049–T051 (Phase 11 audits)
**MVP scope**: Phases 1–6 (T001–T024) — Layer A UX fixes with stub Tags/Size
**Full feature**: T001–T052
