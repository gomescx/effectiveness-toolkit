# Tasks: TMM Tags, Size, and Report

**Feature**: `002-tmm-tags-size-report` | **Branch**: `002-tmm-tags-size-report` | **Date**: 2026-04-09
**Input**: `specs/002-tmm-tags-size-report/` — spec.md, plan.md, business-statement.md
**Baseline**: `specs/001-tmm/` — the existing tool at `public/tools/tmm/index.html` is the only file modified
**Tech Stack**: Vanilla JavaScript (ES6+), HTML5, CSS3 — zero build tooling, zero dependencies
**Delivery**: `public/tools/tmm/index.html` — all changes inline in the single standalone HTML file
**Tests**: Manual test scenarios only (per Constitution §VII — Standalone HTML Tools Exception). No automated test tasks included.

## Format: `[ID] [P?] [US?] Description — file path`

- **[P]**: Task can execute in parallel with other [P] tasks in the same phase (independent sections of the same file)
- **[US6–US8]**: User story this task belongs to
- Task IDs continue from the 001-tmm baseline (T001–T030); this feature starts at **T031**

---

## Phase 1: Data Model & Serialisation (Blocking Prerequisite)

**Purpose**: Update the in-memory state, save serialiser, and load deserialiser for the new `tags` and `size` fields. All subsequent phases depend on these changes — they MUST be complete before any UI work begins.

**⚠️ CRITICAL**: Phases 2–5 cannot start until T031–T034 are complete.

- [ ] T031 [US6] Update the initiative object template in the "Add Initiative" form handler: replace `tmm.category: ""` with `tmm.tags: []`; add `tmm.size: null`. The new object shape is `{ id, name, coer: null, sob: null, memoryMap: null, impactMap: null, tmm: { tags: [], size: null, details: "", x: 0.5, y: 0.5 } }` — in `public/tools/tmm/index.html`

  **Given** the "Add Initiative" form is submitted, **When** T031 is applied, **Then** the created initiative object contains `tmm.tags: []` and `tmm.size: null` instead of `tmm.category`.

- [ ] T032 [US4] [US6] Update the save serialiser: write `tmm.tags` (array) and `tmm.size` (string or null) to the saved JSON; remove any reference to `tmm.category` from the serialiser. The saved tmm section MUST contain: `{ tags, size, details, x, y }` — in `public/tools/tmm/index.html`

  **Given** the user triggers "Save" after T032, **When** the JSON file is opened in a text editor, **Then** each initiative's `tmm` section contains `tags` (array) and `size` (string or null); no `category` key is present.

- [ ] T033 [US4] [US6] Update the load deserialiser: when parsing a loaded initiative's `tmm` section, read `tmm.tags` (default `[]` if absent) and `tmm.size` (default `null` if absent); ignore `tmm.category` if present. Do NOT throw an error if `tags` or `size` are missing — graceful defaults — in `public/tools/tmm/index.html`

  **Given** a project file with no `tmm.tags` or `tmm.size` fields is loaded after T033, **When** the canvas renders, **Then** all initiatives render at the default dot size with empty tag lists; no console errors are thrown.

- [ ] T034 [US6] Update the `openDetailPanel()` and `openEditMode()` functions: replace all references to `initiative.tmm.category` and `#panel-category` / `#edit-category` with the new tags and size fields. This is a scan-and-replace across the read paths — in `public/tools/tmm/index.html`

  **Given** a dot is clicked after T034, **When** the detail panel opens, **Then** the Tags and Size fields are shown (not Category); no "undefined" or missing text appears.

**Checkpoint**: Data model fully updated — UI phases can now begin in parallel.

---

## Phase 2: Add Form & Detail Panel UI (US-006, US-007)

**Purpose**: Replace the Category field in the add form and in the detail panel with Tags (free-text, hashtag-validated) and Size (select). This phase covers the full input/display/edit/cancel/save flow.

- [ ] T035 [P] [US6] Replace the Category `<input>` in the "Add Initiative" form with a Tags `<input type="text" id="field-tags" placeholder="#tag1 #tag2">` and add an `<span id="field-tags-error">` for inline validation. Update the form submit handler to read, split on spaces, validate each token against `#[^\s]+`, block submission on invalid tokens with the message "Each tag must start with #", and set `tmm.tags` — in `public/tools/tmm/index.html`

  **Given** the Add form is open, **When** the user enters `health` (no #) and submits, **Then** the form is blocked and "Each tag must start with #" is shown inline. **When** the user enters `#health #finance` and submits, **Then** the initiative is created with `tags: ["#health", "#finance"]`.

- [ ] T036 [P] [US7] Add a Size `<select id="field-size">` to the "Add Initiative" form (after the Tags field) with options: `<option value="">— None —</option>`, `XS`, `S`, `M`, `L`, `XL`. Update the form submit handler to read the selected value and store it as `tmm.size` (empty string stored as `null`) — in `public/tools/tmm/index.html`

  **Given** the Add form is open, **When** the user selects "L" and submits, **Then** the initiative is created with `tmm.size: "L"`. **When** "— None —" is selected, **Then** `tmm.size` is `null`.

- [ ] T037 [P] [US6] [US7] Update the detail panel view mode HTML: replace `<span id="panel-category">` with `<span id="panel-tags">` (shows space-joined tags) and add `<span id="panel-size">` (shows size or "—"). Update `openDetailPanel()` to populate both spans from `initiative.tmm.tags` and `initiative.tmm.size` — in `public/tools/tmm/index.html`

  **Given** a dot is clicked for an initiative with `tags: ["#health"]` and `size: "M"`, **When** the detail panel opens, **Then** "#health" appears in the tags field and "M" appears in the size field.

- [ ] T038 [P] [US6] [US7] Update the detail panel edit mode HTML: replace `<input id="edit-category">` with `<input id="edit-tags" placeholder="#tag1 #tag2">` and add `<select id="edit-size">` with the same options as T036. Update `openEditMode()` to populate both fields from the current initiative values. Update `saveEdit()` to read, validate, and save both fields. Update `cancelEdit()` to restore the snapshot for both fields — in `public/tools/tmm/index.html`

  **Given** edit mode is open for an initiative with `tags: ["#health"]`, **When** the user clears tags and enters `#career #finance` and selects "XL", then confirms, **Then** `tmm.tags` is `["#career", "#finance"]` and `tmm.size` is `"XL"`. **When** the user cancels, **Then** tags revert to `["#health"]`.

**Checkpoint**: Tags and size are fully editable and displayed in the add form and detail panel.

---

## Phase 3: Canvas Tag Filter (US-006)

**Purpose**: Add a filter bar below the canvas toolbar. Selecting a tag hides non-matching dots. A filter badge shows the active filter and provides a clear button.

- [ ] T039 [US6] Add filter bar HTML: `<div id="filter-bar">` containing `<label>Filter by tag:</label>`, `<select id="tag-filter-select"><option value="">— All —</option></select>`, and `<span id="active-filter-badge" style="display:none">Filtering: <strong id="active-filter-label"></strong> <button id="clear-filter-btn">×</button></span>` — placed below the toolbar in `public/tools/tmm/index.html`

  **Given** the page loads after T039, **When** the filter bar is visible, **Then** a "Filter by tag" select and a hidden badge are present; no existing functionality is broken.

- [ ] T040 [US6] Update `renderCanvas()` to: (1) rebuild the `#tag-filter-select` options with all unique tags across `initiatives[]` (preserve the "— All —" option); (2) skip rendering a dot if `activeTagFilter !== null` and `initiative.tmm.tags` does not include `activeTagFilter`; (3) show/hide `#active-filter-badge` and set `#active-filter-label` based on `activeTagFilter` — in `public/tools/tmm/index.html`

  **Given** three initiatives tagged `#health`, `#finance`, `#health` respectively, **When** `activeTagFilter = "#health"` and `renderCanvas()` runs, **Then** two dots are rendered; the `#finance` dot is absent; the filter badge shows "Filtering: #health".

- [ ] T041 [US6] Wire the `#tag-filter-select` change event: on selection, set `activeTagFilter` to the selected value (or `null` for "— All —"), call `renderCanvas()` — in `public/tools/tmm/index.html`

  **Given** the filter select shows "#health", **When** the user changes it to "— All —", **Then** `activeTagFilter` is set to `null`, `renderCanvas()` runs, and all dots appear.

- [ ] T042 [US6] Wire the `#clear-filter-btn` click event: set `activeTagFilter = null`, reset the select to "— All —", call `renderCanvas()` — in `public/tools/tmm/index.html`

  **Given** the filter badge is visible with "Filtering: #health", **When** the user clicks ×, **Then** `activeTagFilter` is `null`, all dots reappear, and the badge disappears.

**Checkpoint**: Tag filter fully functional — filtering and clearing work; filter state is in-memory only.

---

## Phase 4: CSS Dot Sizes (US-007)

**Purpose**: Add five CSS modifier classes for dot sizes. Update `renderCanvas()` to apply the correct class per initiative.

- [ ] T043 [P] [US7] Add CSS rules for dot size modifier classes to the `<style>` block. Rules must set `width`, `height`, and the centering offsets (`margin-left`, `margin-top` — or `transform: translate`) for each class:
  - `.dot--xs` — 18 px diameter
  - `.dot--s` — 24 px diameter
  - `.dot--m` — 32 px (same as `.dot` default; explicit class for clarity)
  - `.dot--l` — 44 px diameter
  - `.dot--xl` — 58 px diameter
  
  The existing `.dot` class retains its 32 px default (no change) — in `public/tools/tmm/index.html`

  **Given** five dots with classes `.dot--xs` through `.dot--xl` are rendered, **When** viewed on the canvas, **Then** five visually distinct sizes are shown with XS smallest and XL largest; no dot is missing or out of place.

- [ ] T044 [US7] Update `renderCanvas()` dot creation: compute `sizeClass` from `initiative.tmm.size` using a lookup (`{ XS: 'dot--xs', S: 'dot--s', M: 'dot--m', L: 'dot--l', XL: 'dot--xl' }`); set `dot.className = 'dot' + (sizeClass ? ' ' + sizeClass : '')`. Pointer event handlers (T016–T020) MUST remain bound after this change — verify `setPointerCapture`, `pointerdown`, `pointermove`, `pointerup` still work — in `public/tools/tmm/index.html`

  **Given** an initiative with `size: "L"`, **When** `renderCanvas()` runs, **Then** its dot element has class `dot dot--l`. **Given** an initiative with `size: null`, **When** rendered, **Then** its dot element has class `dot` only; drag behaviour is unaffected.

**Checkpoint**: Five dot sizes visually distinguishable on canvas; drag unaffected.

---

## Phase 5: Report View (US-008)

**Purpose**: Add report view section (hidden by default), view toggle, table population from `initiatives[]`, bulk tag assignment, state sync back to canvas, and markdown export.

- [ ] T045 [US8] Add report view HTML: `<section id="report-view" style="display:none">` containing:
  - Report table `<table id="report-table">` with `<thead>` (columns: ☐, Quadrant, Name, Tags, Size, Details)
  - `<tbody id="report-body">` (populated dynamically)
  - Bulk action row below table: `<input id="bulk-tag-input" placeholder="#newtag">`, `<span id="bulk-tag-error">`, `<button id="bulk-apply-btn">Apply to selected</button>`
  - Export button: `<button id="export-md-btn">Export Markdown</button>`
  
  Add "Report" button to the toolbar that calls `showReportView()`. Add a "Canvas" button in the report view header that calls `showCanvasView()` — in `public/tools/tmm/index.html`

  **Given** the page loads after T045, **When** "Report" is clicked, **Then** the report section is visible; the canvas section is hidden; no JavaScript errors occur.

- [ ] T046 [US8] Implement `buildReportTable()`: iterate `initiatives[]`, derive Quadrant from `x`/`y` (`Q1 = x>=0.5 && y>=0.5, Q2 = x<0.5 && y>=0.5, Q3 = x>=0.5 && y<0.5, Q4 = x<0.5 && y<0.5`), build rows with a checkbox `<input type="checkbox" data-id="…">` and display cells for Quadrant, Name, Tags (join with space), Size (or ""), Details. Replace `#report-body` innerHTML on each call — never store external state — in `public/tools/tmm/index.html`

  **Given** three initiatives across three different quadrants, **When** `buildReportTable()` runs, **Then** three rows are rendered with correct Quadrant values; the initiative IDs are on the checkbox `data-id` attributes.

- [ ] T047 [US8] Implement `showReportView()`: set `#canvas-section` display to none, set `#report-view` display to block, call `buildReportTable()` — in `public/tools/tmm/index.html`

  **Given** the canvas has initiatives, **When** "Report" is clicked, **Then** the report table shows all initiatives; switching to canvas and back rebuilds the table from current state.

- [ ] T048 [US8] Implement `showCanvasView()`: set `#report-view` display to none, set `#canvas-section` display to block, call `renderCanvas()` — in `public/tools/tmm/index.html`

  **Given** bulk tags were applied in the report view, **When** "Canvas" is clicked, **Then** `renderCanvas()` runs and the tag filter select reflects the new tags; filter immediately works on the new tags.

- [ ] T049 [US8] Wire `#bulk-apply-btn` click handler: collect all checked checkbox `data-id` values, validate `#bulk-tag-input` value (non-empty, matches `#[^\s]+` — same validation as T035, show error in `#bulk-tag-error` on failure), for each matched initiative in `initiatives[]` push the tag to `tmm.tags` if not already present, call `buildReportTable()` to refresh the view — in `public/tools/tmm/index.html`

  **Given** two checkboxes are checked and `#bulk-tag-input` contains `#review`, **When** "Apply to selected" is clicked, **Then** `#review` is appended to those two initiatives' `tags` arrays; the table rows update to show `#review`; existing tags are preserved. **Given** no checkboxes are checked, **When** clicked, **Then** no changes occur. **Given** input contains `review` (no #), **When** clicked, **Then** the button is blocked and the error span shows "Each tag must start with #".

- [ ] T050 [P] [US8] Wire `#export-md-btn` click handler: build a GFM markdown table string from `initiatives[]` using the same columns as the report table (Quadrant, Name, Tags, Size, Details); download as `tmm-report-YYYY-MM-DD.md` via Blob + `<a download>` pattern (same as T012 save mechanism). Use UTC date for the filename — in `public/tools/tmm/index.html`

  **Given** the report view is open with two initiatives, **When** "Export Markdown" is clicked, **Then** a `.md` file is downloaded; opening the file in a markdown viewer renders a GFM table with two data rows and correct column headers; no HTML or control characters are present.

**Checkpoint**: Report view fully functional — table renders, bulk assign mutates state, canvas syncs on return, markdown export downloads.

---

## Phase 6: Polish & Validation

**Purpose**: Final wiring checks, edge case handling, and manual smoke test pass.

- [ ] T051 Handle report edge cases: (a) zero initiatives → render empty table with headers only, no error; (b) initiative with `tags: []` → empty Tags cell (no "undefined"); (c) initiative with `size: null` → empty Size cell; (d) "Apply to selected" with no rows checked → no-op, no error — in `public/tools/tmm/index.html`

  **Given** the canvas is empty, **When** the report view is opened, **Then** a table with headers and no rows is shown; no JavaScript errors are thrown.

- [ ] T052 [P] Verify that `activeTagFilter` is reset to `null` when a new project file is loaded (the `loadFile()` handler must reset filter state and call `renderCanvas()`). Add `activeTagFilter = null` and reset `#tag-filter-select` value to the load handler — in `public/tools/tmm/index.html`

  **Given** a tag filter is active, **When** the user loads a different project file, **Then** the filter is cleared, the badge disappears, and all the newly loaded initiatives are rendered.

- [ ] T053 Manual smoke test (all phases): Follow the three-phase manual test plan at `specs/002-tmm-tags-size-report/manual-test-plan.md`. Record pass/fail for each scenario. Do not mark tasks complete until all scenarios in Phase 1, Phase 2, and Phase 3 of the manual test plan pass.

  **Given** the manual test plan is executed, **When** all scenarios in Phases 1–3 pass, **Then** the feature is ready for review.
