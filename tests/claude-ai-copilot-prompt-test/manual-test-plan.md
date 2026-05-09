# Manual Test Plan: TMM Tags, Size, and Report

**Feature**: `002-tmm-tags-size-report` | **Branch**: `002-tmm-tags-size-report` | **Date**: 2026-04-09
**Tool**: `public/tools/tmm/index.html` (open directly in browser — no server required)
**Testing approach**: Manual only (Constitution §VII — Standalone HTML Tools Exception)
**Spec**: [specs/002-tmm-tags-size-report/spec.md](./spec.md)
**Traceability**: Each scenario references the Acceptance Scenario (AS) from spec.md

---

## Setup

Before running any phase:
1. Open `public/tools/tmm/index.html` in a modern browser (Chrome, Edge, Firefox, or Safari)
2. Confirm the baseline tool loads: canvas renders, Add/Save/Load buttons are visible
3. Use a fresh tool state for each phase (reload the page or clear state as needed)

---

## Phase 1 — Tags (US-006)

**Covers**: FR-US6.01 through FR-US6.10 | SC-006.1 | SC-006.2

**Prerequisite**: T031–T035 and T039–T042 are implemented.

---

### MTP-006-01 — Tag entry accepted in Add Initiative form

**Traces to**: AS US6.1

- [ ] **Given** the TMM tool is open and the "Add Initiative" form is accessible,
  **When** the user opens the Add form, enters `Initiative A` in the Name field, enters `#health #project-alpha` in the Tags field, and submits,
  **Then** a dot labelled "Initiative A" appears on the canvas; no error message is shown; the Tags field in the detail panel shows `#health #project-alpha`.

---

### MTP-006-02 — Invalid tag format is rejected (no leading #)

**Traces to**: AS US6.2

- [ ] **Given** the Add form is open,
  **When** the user enters `health` (no leading #) in the Tags field and submits,
  **Then** the form submission is blocked; an inline message "Each tag must start with #" (or equivalent) appears next to the Tags field; no initiative is created.

- [ ] **Given** the Add form is still open after the validation error,
  **When** the user corrects the input to `#health` and submits,
  **Then** the initiative is created successfully and the error message disappears.

---

### MTP-006-03 — Tags survive save and reload

**Traces to**: AS US6.3 | SC-006.1

- [ ] **Given** an initiative with tags `#health #project-alpha` has been added to the canvas,
  **When** the user clicks "Save", saves the file, reloads the browser page, clicks "Load", and selects the saved file,
  **Then** the initiative is restored; opening its detail panel shows exactly `#health #project-alpha` in the Tags field — no reordering, no missing tokens, no extra whitespace.

---

### MTP-006-04 — Tag with no leading space between tokens

**Traces to**: AS US6.1 (format rule: no spaces within a token)

- [ ] **Given** the Add form is open,
  **When** the user enters `#health#finance` (no space between tokens) in the Tags field and submits,
  **Then** the initiative is created with a single tag `#health#finance` (treated as one token, since there is no space separator); the detail panel shows `#health#finance` as one tag.

---

### MTP-006-05 — Empty tags are allowed

**Traces to**: AS US6.7

- [ ] **Given** the Add form is open,
  **When** the user leaves the Tags field blank and submits with only a Name,
  **Then** the initiative is created with `tags: []`; the detail panel shows an empty Tags field; no error is shown.

---

### MTP-006-06 — Tag filter hides non-matching dots

**Traces to**: AS US6.4 | SC-006.2

- [ ] **Given** three initiatives are on the canvas — "Alpha" tagged `#health`, "Beta" tagged `#health`, "Gamma" tagged `#career`,
  **When** the user selects `#health` in the tag filter control,
  **Then** only "Alpha" and "Beta" dots are visible on the canvas; "Gamma" is not rendered; a filter indicator shows that `#health` is active.

---

### MTP-006-07 — Filter indicator is always visible when filter is active

**Traces to**: AS US6.4 | SC-006.2

- [ ] **Given** the `#health` filter is active and the filter indicator is visible,
  **When** the user scrolls the canvas, resizes the window, or interacts with the canvas in any other way,
  **Then** the filter indicator remains visible at all times.

---

### MTP-006-08 — Clearing the filter restores all dots

**Traces to**: AS US6.5 | SC-006.2

- [ ] **Given** the `#health` filter is active and only matching dots are visible,
  **When** the user clears the filter (via the × button on the badge or by selecting "— All —" in the filter select),
  **Then** all initiative dots — including "Gamma" tagged `#career` — are immediately restored to the canvas; the filter indicator disappears.

---

### MTP-006-09 — Untagged initiative is hidden when filter is active

**Traces to**: AS US6.6

- [ ] **Given** one initiative has `tags: []` (empty) and one has `tags: ["#health"]`,
  **When** the user activates the `#health` filter,
  **Then** the untagged initiative is hidden (it does not match any tag); only the `#health` initiative is visible.

---

### MTP-006-10 — Tags editable in detail panel edit mode

**Traces to**: AS US6 (detail panel edit path, FR-US6.04)

- [ ] **Given** an initiative with `tags: ["#health"]` exists and its detail panel is open in edit mode,
  **When** the user clears the Tags field, enters `#career #finance`, and confirms the edit,
  **Then** the detail panel view mode shows `#career #finance`; the canvas filter select rebuilds to include `#career` and `#finance`.

- [ ] **Given** the user edits the Tags field to an invalid value `badtag` (no #) and attempts to confirm,
  **Then** the edit is blocked with the same inline validation message as the Add form.

---

## Phase 2 — T-Shirt Size and Visual Dots (US-007)

**Covers**: FR-US7.01 through FR-US7.08 | SC-007.1 | SC-007.2

**Prerequisite**: T031–T034, T036, T038, T043–T044 are implemented.

---

### MTP-007-01 — Size assigned and displayed in detail panel

**Traces to**: AS US7.1 | SC-007.1

- [ ] **Given** the "Add Initiative" form is open,
  **When** the user enters a name, selects size `L`, and submits,
  **Then** the initiative is created; opening its detail panel shows "L" as the Size value.

---

### MTP-007-02 — Size persists after save and reload

**Traces to**: AS US7.2 | SC-007.1

- [ ] **Given** an initiative with `size: "L"` exists on the canvas,
  **When** the user saves, reloads the browser, loads the saved file,
  **Then** opening the initiative's detail panel shows `L` as the size; no other fields are affected.

---

### MTP-007-03 — Five visually distinct dot sizes on canvas

**Traces to**: AS US7.3 | SC-007.2

- [ ] **Given** five initiatives are added with sizes XS, S, M, L, XL respectively (one each),
  **When** all five are visible on the canvas simultaneously,
  **Then** five visually distinct dot sizes are rendered; XS is clearly the smallest; XL is clearly the largest; each size step is distinguishable without opening any detail panel.

---

### MTP-007-04 — Null size renders at default dot size

**Traces to**: AS US7.4 | FR-US7.06

- [ ] **Given** an initiative is added with no size selected (size = None / null),
  **When** the canvas is rendered,
  **Then** the dot renders at the default 32 px size; it is the same size as the "M" dot; no visual error or missing dot.

---

### MTP-007-05 — Existing file without size field loads cleanly

**Traces to**: AS US7.5 | FR-US7.07

- [ ] **Given** a project file saved by the *old* tool (no `tmm.size` or `tmm.tags` fields — only `tmm.category`),
  **When** the updated tool loads the file,
  **Then** all initiatives render at the default dot size; no console errors are thrown; no visible error messages are shown to the user.

---

### MTP-007-06 — Drag-and-drop still works with size modifier class

**Traces to**: FR-US7.08 | AS US2.1–US2.5 (baseline)

- [ ] **Given** an initiative with size `XL` is on the canvas,
  **When** the user drags the dot to a new position and releases,
  **Then** the dot stays at the new position with the `.dot--xl` class still applied; the drag behaviour is identical to the default-size dot.

---

### MTP-007-07 — Size editable in detail panel edit mode

**Traces to**: FR-US7.02 | FR-US7.03

- [ ] **Given** an initiative with `size: "S"` is on the canvas and its detail panel is open in edit mode,
  **When** the user changes the size selector to `XL` and confirms,
  **Then** the detail panel view shows "XL"; the canvas dot immediately updates to the `.dot--xl` class on `renderCanvas()`.

- [ ] **Given** the user changes size to "— None —" and confirms,
  **Then** `tmm.size` becomes `null`; the dot renders at the default 32 px size.

---

## Phase 3 — Report and Bulk Tagging (US-008)

**Covers**: FR-US8.01 through FR-US8.12 | SC-008.1 | SC-008.2 | SC-008.3

**Prerequisite**: T045–T052 are implemented; Phases 1 and 2 test scenarios pass.

---

### MTP-008-01 — Report view opens and shows all initiatives

**Traces to**: AS US8.1 | SC-008.1

- [ ] **Given** the canvas has four initiatives across different quadrants,
  **When** the user clicks "Report",
  **Then** the canvas is hidden; a report table is shown with four rows; each row has columns for Quadrant, Name, Tags, Size, and Details.

---

### MTP-008-02 — Quadrant column is correct

**Traces to**: AS US8.2 | FR-US8.03

- [ ] **Given** four initiatives have been dragged to the four distinct quadrant areas (top-right = Q1, top-left = Q2, bottom-right = Q3, bottom-left = Q4),
  **When** the report view is opened,
  **Then** each row's Quadrant cell shows the correct label (Q1, Q2, Q3, Q4) matching the initiative's canvas position.

---

### MTP-008-03 — Empty canvas shows empty report table

**Traces to**: FR-US8.12

- [ ] **Given** the canvas has zero initiatives (fresh load or all deleted),
  **When** the user opens the report view,
  **Then** an empty table with column headers is shown; no error message, no crash, no "undefined" cells.

---

### MTP-008-04 — Bulk tag assignment appends tag to selected initiatives

**Traces to**: AS US8.3 | SC-008.3

- [ ] **Given** the report view shows three rows,
  **When** the user checks the checkboxes for two of the three rows, enters `#review` in the bulk tag input, and clicks "Apply to selected",
  **Then** those two initiatives' Tags cells update to include `#review`; the third initiative (unchecked) is unchanged; no existing tags are lost.

---

### MTP-008-05 — Bulk tag validation enforced

**Traces to**: FR-US8.06

- [ ] **Given** the report view is open with rows checked,
  **When** the user enters `review` (no #) in the bulk tag input and clicks "Apply to selected",
  **Then** the action is blocked; an inline error message is shown; no tags are changed.

- [ ] **Given** the bulk tag input is empty,
  **When** "Apply to selected" is clicked with checked rows,
  **Then** the action is blocked with an inline message; no tags are changed.

---

### MTP-008-06 — No-op when no rows are checked

**Traces to**: Edge Case — bulk apply with no checkboxes checked

- [ ] **Given** no checkboxes are checked in the report view,
  **When** the user enters `#review` and clicks "Apply to selected",
  **Then** no tags are changed; no error is thrown; the table is unchanged.

---

### MTP-008-07 — State synchronisation: bulk tags immediately visible on canvas return

**Traces to**: AS US8.4 | SC-008.3

- [ ] **Given** the user bulk-assigned `#review` to two initiatives in the report view,
  **When** the user clicks "Canvas" (or equivalent) to return to the canvas view,
  **Then** `renderCanvas()` runs; the tag filter select now includes `#review`; selecting `#review` shows only those two initiatives.

- [ ] **Given** the canvas view is shown after returning from report,
  **When** the user saves the project file and reloads,
  **Then** the `#review` tags are persisted and restored on the bulk-assigned initiatives.

---

### MTP-008-08 — No stale state between report and canvas

**Traces to**: AS US8.7 | FR-US8.08

- [ ] **Given** the user opens the report, makes no changes, and returns to the canvas,
  **Then** no canvas state has changed — dot positions, tags, sizes, and details are identical to before entering the report view.

---

### MTP-008-09 — Markdown export downloads a .md file

**Traces to**: AS US8.5, US8.6 | SC-008.2

- [ ] **Given** the report view shows initiatives,
  **When** the user clicks "Export Markdown",
  **Then** the browser downloads a file with a `.md` extension and a name matching `tmm-report-YYYY-MM-DD.md` (current date).

---

### MTP-008-10 — Exported markdown renders correctly

**Traces to**: AS US8.6 | SC-008.2

- [ ] **Given** a `.md` file has been downloaded,
  **When** the file is opened in VS Code markdown preview, GitHub preview, or any standards-compliant viewer,
  **Then** a table is rendered with column headers (Quadrant, Name, Tags, Size, Details) and one data row per initiative; all values are present and correctly escaped; no raw HTML or broken pipe characters.

---

### MTP-008-11 — Report is read-only except for bulk tag assignment

**Traces to**: FR-US8.09

- [ ] **Given** the report view is open,
  **When** the user inspects the Name, Size, Details, and Quadrant cells,
  **Then** none of these cells are editable (no input fields, no contenteditable, no inline edit options); only the tag bulk-assignment controls are interactive.

---

## Test Results Summary

| Phase | Scenario | Status | Notes |
|-------|----------|--------|-------|
| P1 | MTP-006-01 | | |
| P1 | MTP-006-02 | | |
| P1 | MTP-006-03 | | |
| P1 | MTP-006-04 | | |
| P1 | MTP-006-05 | | |
| P1 | MTP-006-06 | | |
| P1 | MTP-006-07 | | |
| P1 | MTP-006-08 | | |
| P1 | MTP-006-09 | | |
| P1 | MTP-006-10 | | |
| P2 | MTP-007-01 | | |
| P2 | MTP-007-02 | | |
| P2 | MTP-007-03 | | |
| P2 | MTP-007-04 | | |
| P2 | MTP-007-05 | | |
| P2 | MTP-007-06 | | |
| P2 | MTP-007-07 | | |
| P3 | MTP-008-01 | | |
| P3 | MTP-008-02 | | |
| P3 | MTP-008-03 | | |
| P3 | MTP-008-04 | | |
| P3 | MTP-008-05 | | |
| P3 | MTP-008-06 | | |
| P3 | MTP-008-07 | | |
| P3 | MTP-008-08 | | |
| P3 | MTP-008-09 | | |
| P3 | MTP-008-10 | | |
| P3 | MTP-008-11 | | |
