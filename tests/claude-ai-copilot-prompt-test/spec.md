# Feature Specification: TMM Tags, Size, and Report

**Feature Branch**: `002-tmm-tags-size-report`
**Created**: 2026-04-09
**Status**: Draft
**Governance**: [Constitution v1.3.0](../../.specify/memory/constitution.md) | [Solo Developer Principles](../../.specify/memory/solo-developer-principles.md)
**Baseline**: [specs/001-tmm/spec.md](../001-tmm/spec.md) | [specs/001-tmm/data-model.md](../001-tmm/data-model.md)
**Input**: [specs/002-tmm-tags-size-report/business-statement.md](./business-statement.md)

## Context

This feature extends the Time Management Matrix tool (US-001 through US-005, delivered in `001-tmm`) with three new capabilities: a structured multi-tag system replacing the single `category` field (US-006), a t-shirt size field with proportional canvas dots (US-007), and a tabular report view with bulk tag assignment and markdown export (US-008).

The baseline TMM tool stores each initiative's data in a `tmm` section within the shared `effectivenessToolkit` project file. This feature modifies only the TMM section schema and the standalone HTML tool at `public/tools/tmm/index.html`. No other tools, the shared envelope structure, or any other section in the project file are affected.

**US numbering continues from baseline**: US-001 through US-005 are defined in `specs/001-tmm/spec.md`. This spec defines US-006, US-007, and US-008.

> Constitutional governance (Principles I–VI) applies. Principles are not repeated here — they are referenced where relevant.

---

## User Scenarios & Testing *(mandatory)*

### User Story 6 — Tag Initiatives with Hashtag Labels (Priority: P1)

A PEP coach or executive opens the detail panel for an initiative and enters one or more hashtag-formatted labels (e.g., `#health #project-alpha`). The tags are validated, stored, and restored on reload. A tag filter on the canvas hides non-matching dots entirely; a filter indicator is always shown when a filter is active.

**Why this priority**: Tags replace the `category` field entirely — they are the primary classification mechanism going forward. Without tags, US-007 (size) and US-008 (report) have degraded value. P1 within this feature set.

**Independent Test**: Open the tool, load or create two initiatives. Add tags `#health #finance` to one and `#career` to the other. Save and reload — each initiative restores its exact tags. Activate the `#health` filter — only the first dot is visible and a filter indicator appears. Clear the filter — both dots return.

**Acceptance Scenarios**:

1. **Given** the "Add Initiative" form is open, **When** the user enters `#health #project-alpha` in the Tags field and submits, **Then** the initiative is created with `tags: ["#health", "#project-alpha"]`; no reformatting occurs and the tags are stored exactly as entered (lowercased or as-typed — no normalisation).
2. **Given** the Tags field in the add form, **When** the user types a token without a leading `#` (e.g., `health`), **Then** form submission is blocked and an inline validation message is shown: "Each tag must start with #".
3. **Given** an initiative with `tags: ["#health", "#project-alpha"]`, **When** the user saves and reloads the project file, **Then** both tags are restored in the detail panel exactly as stored — no ordering change, no silent reformatting, no data loss.
4. **Given** the canvas has three initiatives — two tagged `#health`, one tagged `#career` — **When** the user activates the tag filter for `#health`, **Then** only the two `#health` initiatives remain visible on the canvas; the `#career` initiative disappears (dot not rendered); a filter indicator is displayed on the canvas.
5. **Given** a tag filter is active and the filter indicator is visible, **When** the user clears the filter, **Then** all initiative dots are restored to the canvas and the filter indicator disappears.
6. **Given** an initiative with no tags (empty array), **When** a tag filter is active, **Then** the untagged initiative is hidden (it does not match any tag filter).
7. **Given** the detail panel is open in edit mode, **When** the user clears all tags and saves, **Then** the initiative's `tags` array is stored as `[]`; no error is thrown; the initiative renders at its normal position without any tag label.

---

### User Story 7 — T-Shirt Size with Proportional Canvas Dots (Priority: P2)

A PEP coach or executive assigns a t-shirt size (XS, S, M, L, XL) to an initiative in the detail panel. The initiative's dot on the canvas scales proportionally. Five initiatives with different sizes are visually distinguishable without opening any detail panel. Initiatives with no size assigned render at the default 32 px size.

**Why this priority**: Size adds a scale dimension to the canvas, complementing tag-based classification. Depends on the detail panel edit flow (US-005, baseline) but not on the report view (US-008). P2 within this feature.

**Independent Test**: Add five initiatives, assign XS through XL to each. View the canvas — five visually distinct dot sizes are visible. Assign no size to a sixth initiative — it renders at the default dot size. Save and reload — all size assignments are restored.

**Acceptance Scenarios**:

1. **Given** the detail panel's edit mode is open, **When** the user selects size `L` from the size selector, **Then** the size value is stored as `"L"` in `initiative.tmm.size` and displayed in the detail panel view mode.
2. **Given** an initiative with `size: "L"`, **When** the user saves and reloads the project file, **Then** the size is restored as `"L"` in both the in-memory state and the detail panel display.
3. **Given** five initiatives with sizes `XS`, `S`, `M`, `L`, `XL` respectively, **When** the canvas is rendered, **Then** five visually distinct dot sizes are shown — XS is the smallest, XL is the largest — without the user needing to open any detail panel.
4. **Given** an initiative with `size: null` or with no `size` field in the loaded file, **When** the canvas is rendered, **Then** the dot is rendered at the default 32 px size with no visual error or missing-dot behaviour.
5. **Given** an existing project file saved before this feature (no `size` field in any tmm section), **When** loaded by the updated tool, **Then** all initiatives render at the default size; no console errors are thrown; re-saving the file does not corrupt any other field.

---

### User Story 8 — Tabular Report View with Bulk Tag Assignment and Markdown Export (Priority: P3)

A PEP coach or executive opens a report view showing all canvas initiatives in a table — quadrant, name, tags, size, and details. They can bulk-assign a tag to multiple initiatives using checkboxes. Changes made in the report are immediately visible on the canvas when returning to it. The report can be exported as a `.md` file.

**Why this priority**: The report bridges brainstorming (canvas) to operational planning (shareable summary). Depends on tags (US-006) and size (US-007) being in place so all columns are meaningful. P3 within this feature.

**Independent Test**: Load a canvas with four initiatives across different quadrants. Open the report view — a table appears with all four rows and correct columns. Check three rows, enter `#review` in the bulk tag field, click "Apply to selected" — those three rows show the new tag. Return to the canvas, activate the `#review` tag filter — three dots appear. Click "Export Markdown" — a `.md` file downloads and renders the correct table in a markdown viewer.

**Acceptance Scenarios**:

1. **Given** the canvas has at least one initiative, **When** the user opens the report view, **Then** a table is rendered listing every initiative with the following columns: Quadrant (Q1/Q2/Q3/Q4 label), Name, Tags (space-separated token display), Size (XS/S/M/L/XL or blank), and Details.
2. **Given** the report is open and the canvas has initiatives in different quadrants, **When** the table is viewed, **Then** each row's Quadrant column correctly reflects the initiative's canvas position quadrant (Q1 = High Impact/High Urgency, Q2 = High Impact/Low Urgency, Q3 = Low Impact/High Urgency, Q4 = Low Impact/Low Urgency).
3. **Given** the report view is open with multiple rows, **When** the user checks the checkboxes for three rows, enters `#review` in the bulk tag input, and clicks "Apply to selected", **Then** the tag `#review` is appended to each of those three initiatives' `tags` arrays (existing tags are preserved); the table rows update to show the new tag.
4. **Given** the user bulk-assigns a tag in the report view, **When** they return to the canvas view, **Then** the canvas immediately reflects the new tags on those initiatives — the filter system recognises `#review` without any page reload or manual refresh.
5. **Given** the canvas has at least one initiative, **When** the user clicks "Export Markdown", **Then** the browser downloads a `.md` file named using the pattern `tmm-report-<ISO-date>.md`.
6. **Given** the downloaded `.md` file, **When** opened in any standards-compliant markdown viewer (e.g., GitHub preview, VS Code preview), **Then** a table is rendered with the same rows and columns as the report view (Quadrant, Name, Tags, Size, Details); all initiative data is present and correctly formatted.
7. **Given** the report view is open, **When** the user performs no bulk action and returns to the canvas, **Then** no canvas state is altered — no tags added, no positions changed, no data lost.

---

### Edge Cases

- **Empty canvas open report**: Opening the report view with zero initiatives renders an empty table with column headers — no error shown.
- **Initiative with no tags and size**: A report row for an initiative with `tags: []` and `size: null` shows empty Tags and Size cells — no null or "undefined" text.
- **Bulk apply with no rows checked**: Clicking "Apply to selected" with no checkboxes checked has no effect — no tags are added, no error is thrown.
- **Bulk apply with empty tag input**: Clicking "Apply to selected" with the tag input empty is blocked — inline validation message shown: "Enter a tag to apply".
- **Bulk apply with invalid tag format**: Entering a token that does not begin with `#` is blocked at the bulk tag input with the same format validation as the add-initiative form.
- **Report view while detail panel is open**: If the user switches to report view while the detail panel is open, the panel is closed and no edit is in progress when the canvas view is restored.
- **Tag filter + size null**: An initiative with `size: null` that matches the active tag filter is shown at the default dot size — it is not hidden by the size being null.
- **Long tag lists**: An initiative with six or more tags displays them space-separated in the detail panel; the report cell shows all tags (no truncation in the data, visual truncation with full title tooltip is acceptable).
- **Dot size CSS and drag**: Adding size modifier classes to dots MUST NOT break the existing pointer-event drag behaviour defined in US-002/T016–T020.

---

## Requirements *(mandatory)*

### Functional Requirements — US6: Tags

- **FR-US6.01**: The `tmm` section of each initiative MUST store tags as a `tags: string[]` field (an array of hashtag-formatted tokens); the `category` field is removed entirely from the data model — no migration logic is required (FR-US4.03 in `001-tmm/spec.md` is superseded for the `category` field).
- **FR-US6.02**: The "Add Initiative" form MUST replace the Category input with a Tags input field; the existing Name and Details fields are unchanged.
- **FR-US6.03**: The Tags input field MUST accept space-separated tokens; each token MUST conform to the format `#[^\s]+` (starts with `#`, no internal spaces); any token failing this format MUST block form submission with an inline validation message.
- **FR-US6.04**: The detail panel MUST display the initiative's tags (space-separated) in the view and edit modes; in edit mode the tags field MUST be editable with the same format validation as the add form.
- **FR-US6.05**: An initiative MAY have zero tags (`tags: []`); this is a valid stored state and MUST NOT trigger any warning or validation error.
- **FR-US6.06**: The tool MUST provide a tag filter control on the canvas; selecting a tag from the filter MUST hide all initiative dots that do not include that tag in their `tags` array; dots are hidden entirely (not greyed out).
- **FR-US6.07**: When a tag filter is active, a clearly visible filter indicator MUST be displayed on the canvas at all times; the indicator MUST identify the active tag and provide a way to clear the filter.
- **FR-US6.08**: Clearing the tag filter MUST restore all initiative dots to the canvas and remove the filter indicator immediately — no page reload.
- **FR-US6.09**: Canvas filter state (active tag selection) MUST be session-only; it MUST NOT be persisted to the project file.
- **FR-US6.10**: Tags MUST be persisted to the project file in the `tmm.tags` array on save and restored on load exactly as stored.

### Functional Requirements — US7: T-Shirt Size

- **FR-US7.01**: The `tmm` section MUST add a `size` field with the type `"XS" | "S" | "M" | "L" | "XL" | null`; `null` is the default for initiatives with no size assigned.
- **FR-US7.02**: The detail panel's edit mode MUST provide a size selector (e.g., a `<select>` or button group) offering the values XS, S, M, L, XL and a "None / unset" option that stores `null`.
- **FR-US7.03**: The selected size MUST be displayed in the detail panel view mode.
- **FR-US7.04**: `renderCanvas()` MUST apply a CSS modifier class to each dot corresponding to its size: `.dot--xs`, `.dot--s`, `.dot--m`, `.dot--l`, `.dot--xl`; no modifier class is added for `null` (the default `.dot` styles apply).
- **FR-US7.05**: Five CSS size modifier classes MUST define visually distinct dot sizes; XS MUST be the smallest, XL the largest, with a clear visual step between each level. The default `.dot` size (32 px) is used for `null`/unset.
- **FR-US7.06**: An initiative with `size: null` or with no `size` field in the loaded project file MUST render at the default 32 px dot size; no error or visual glitch is produced.
- **FR-US7.07**: Size MUST be persisted to the `tmm.size` field on save and restored on load; existing project files loaded without a `size` field MUST default to `null` without error.
- **FR-US7.08**: Adding a size modifier class to a dot MUST NOT break the existing drag-and-drop behaviour (pointer events — FR-US2.01 through FR-US2.05 in `001-tmm/spec.md`).

### Functional Requirements — US8: Report View

- **FR-US8.01**: The tool MUST provide a "Report" navigation action (button or tab) that switches the main view from the canvas to a report view without a page reload.
- **FR-US8.02**: The report view MUST display a table listing all current initiatives with columns: Quadrant (derived from canvas `x`/`y` coordinates), Name, Tags (space-separated tokens), Size (XS/S/M/L/XL or empty), and Details.
- **FR-US8.03**: Quadrant derivation MUST use the same logic as the canvas visual quadrant: Q1 = `x >= 0.5 AND y >= 0.5` (High Urgency, High Impact), Q2 = `x < 0.5 AND y >= 0.5` (Low Urgency, High Impact), Q3 = `x >= 0.5 AND y < 0.5` (High Urgency, Low Impact), Q4 = `x < 0.5 AND y < 0.5` (Low Urgency, Low Impact).
- **FR-US8.04**: The report view MUST provide a checkbox per row to select initiatives for bulk operations.
- **FR-US8.05**: The report view MUST provide a bulk tag input field and an "Apply to selected" button; clicking the button appends the entered tag to the `tags` array of all checked initiatives; existing tags are preserved.
- **FR-US8.06**: The bulk tag input MUST enforce the same `#[^\s]+` format validation as FR-US6.03; submission with an empty input or an invalid token MUST be blocked with an inline message.
- **FR-US8.07**: Bulk tag changes MUST mutate the shared in-memory `initiatives[]` array in place — they MUST NOT reconstruct state from the DOM or the report table rows.
- **FR-US8.08**: Returning from the report view to the canvas MUST trigger `renderCanvas()` so that tag and size changes from the report are immediately visible and filterable — no page reload required.
- **FR-US8.09**: The report view MUST be read-only except for bulk tag assignment; no editing of name, position, details, or size from the report view.
- **FR-US8.10**: The tool MUST provide an "Export Markdown" button in the report view that downloads a `.md` file via the browser's file-save mechanism; the file MUST contain a standard GFM (GitHub Flavoured Markdown) table with the same columns and data as the report table.
- **FR-US8.11**: The exported filename MUST follow the pattern `tmm-report-YYYY-MM-DD.md` using the current date.
- **FR-US8.12**: An empty canvas (zero initiatives) MUST render an empty report table with column headers — no error or missing-table rendering.

### Cross-Cutting Requirements

- **FR-000.10**: The `category` field removal is a breaking schema change scoped to the `tmm` section only; the shared envelope structure, `id`, `name`, and all other tool sections are unchanged (Constitution Principle VI — Tool Independence).
- **FR-000.11**: Project files saved by the updated tool MUST remain loadable by any future tool that ignores unknown or absent tmm fields (Constitution Principle VI).
- **FR-000.12**: All new UI elements (tags input, size selector, report view, filter indicator, bulk action) MUST be implemented in the same standalone HTML file (`public/tools/tmm/index.html`) with all JS/CSS inline — no new files, no external dependencies (Constitution Principle V).

### Key Entities (updated)

- **Initiative.tmm** (updated): Fields `id`, `name` unchanged at initiative level. TMM section now: `tags: string[]` (replaces `category: string`), `size: "XS" | "S" | "M" | "L" | "XL" | null`, `details: string`, `x: number`, `y: number`.
- **Tag Token**: A string matching `#[^\s]+`. An initiative holds an ordered array of tag tokens with no uniqueness enforcement.
- **T-Shirt Size**: An enum value from `{ "XS", "S", "M", "L", "XL" }` or `null`. Maps to a CSS modifier class for dot rendering.
- **Report Row**: A derived view of an initiative — Quadrant (computed from x, y), Name, Tags (display), Size (display), Details. Not persisted; always derived from `initiatives[]`.
- **Canvas Filter State**: Session-only boolean + active tag string. Not persisted to file.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-006.1** *(traces to AS US6.3)*: A user types `#health #project-alpha` into the tags field, saves, reloads, and sees exactly those two tags restored — no spaces accepted within a token, no silent reformatting.
- **SC-006.2** *(traces to AS US6.4, US6.5)*: A user activates a tag filter on the canvas — only matching initiatives remain visible, all others disappear entirely, and a clearly labelled filter indicator is shown with a mechanism to clear the filter. Clearing the filter restores all dots immediately.
- **SC-007.1** *(traces to AS US7.1, US7.2)*: A user assigns size `L` to an initiative — the size is visible in the detail panel and persists after save/reload.
- **SC-007.2** *(traces to AS US7.3)*: Five initiatives are assigned XS through XL — each renders as a visually distinct dot size on the canvas, with XS clearly the smallest and XL clearly the largest. A user with no prior knowledge can identify relative scale without opening any detail panel.
- **SC-008.1** *(traces to AS US8.1, US8.2)*: A user opens the report view and sees a table listing all canvas initiatives with columns for Quadrant, Name, Tags, Size, and Details — all values match the canvas state.
- **SC-008.2** *(traces to AS US8.5, US8.6)*: A user exports the report and receives a `.md` file that renders the same table correctly in any standards-compliant markdown viewer.
- **SC-008.3** *(traces to AS US8.3, US8.4)*: A user bulk-assigns a tag to three initiatives in the report view, returns to the canvas, and sees the filter system immediately reflect those tags — no reload, no stale state, no data loss.

---

## Assumptions

- The `category` field in the existing `public/tools/tmm/index.html` was a proof-of-concept field with no production user data; no migration from `category` to `tags` is required.
- Tag format validation (`#[^\s]+`) is enforced at input time; the saved project file is assumed to contain only valid tokens (no re-validation on load).
- The canvas filter is a single-tag filter (select one tag, hide non-matching); multi-tag AND/OR filtering is deferred.
- The "Export Markdown" filename uses the UTC date of export; no user-configurable filename is needed.
- The report view is treated as a second panel/section within the same HTML document — it is shown/hidden with CSS (no navigation or URL change).
- Tag display order in the report table matches the order stored in the `tags` array.
- The t-shirt size selector in the detail panel is a `<select>` element for simplicity; no custom dropdown is needed.

---

## Deferred (Post-Feature)

- Tag autocomplete, taxonomy management, or predefined tag lists.
- Visual colour-coding or grouping of dots by tag on the canvas.
- Canvas multi-select bulk tagging (shift-click dots).
- T-shirt size mapping to Agile levels (epic / feature / story / task).
- Report feeding into action plan or Memory Map integration.
- Browser print or PDF export of the report.
- Multi-tag canvas filter (AND / OR combinations).
- Sort/group capabilities in the report table.
