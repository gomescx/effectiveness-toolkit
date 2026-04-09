# Feature Specification: TMM Tags, Size, and Report

**Feature Branch**: `002-tmm-tags-size-report`  
**Created**: 2026-04-09  
**Status**: Draft  
**Governance**: [Constitution v1.3.0](../../.specify/memory/constitution.md) | [Solo Developer Principles](../../.specify/memory/solo-developer-principles.md)  
**Input**: [specs/002-tmm-tags-size-report/business-statement.md](./business-statement.md)

## Context

This feature extends the TMM tool ([specs/001-tmm/spec.md](../001-tmm/spec.md)) with three enhancements: a structured multi-tag system that replaces the single `category` field, t-shirt sizing per initiative with proportional canvas representation, and a tabular report view with bulk tag assignment and markdown export. User Stories 1–5 and their functional requirements (FR-US1 through FR-US5) are defined in `specs/001-tmm/spec.md` and are not repeated here. Numbering continues from US-006.

> Constitutional governance (Principles I–VI) applies. Principles are not repeated here — they are referenced where relevant.

---

## User Scenarios & Testing *(mandatory)*

### User Story 6 — Tag Initiatives Across Multiple Dimensions (Priority: P1)

A PEP coach or executive assigns one or more hashtag labels to each initiative through the detail panel, using a space-separated input validated to the `#token` format (e.g., `#health #project-alpha`). They can filter the canvas to show only initiatives matching a chosen tag; non-matching dots are hidden entirely. A persistent filter indicator is always visible on the canvas while a filter is active so the user never forgets a filter is applied.

**Why this priority**: Tags replace the existing `category` field — they are a structural change to the data model and the foundation on which the filter (this story) and the report view (US-008) both depend. Without tags in place, no other story in this feature delivers its full value. This is the core deliverable of the feature.

**Independent Test**: Open the tool. Add initiative A tagged `#health #career` and initiative B tagged `#finance`. Activate the tag filter for `#health` — confirm only A is visible and a filter indicator for `#health` is shown. Clear the filter — confirm both dots return. Save the file, reload, click dot A — confirm tags read back as `#health #career` with no changes.

**Acceptance Scenarios**:

1. **Given** the detail panel is open for an initiative, **When** the user views the Tags field, **Then** a text input is shown accepting a space-separated, hashtag-prefixed string (e.g., `#health #career`); the former Category field is absent.
2. **Given** a tags input of `#health #project-alpha`, **When** the user saves the form, **Then** two distinct tag tokens, `#health` and `#project-alpha`, are stored on the initiative.
3. **Given** a tags input containing a token without the `#` prefix (e.g., `health`) or a token with embedded spaces (e.g., `#my tag`), **When** the user submits, **Then** the form is rejected with an inline validation message; no partial save of any tag tokens occurs.
4. **Given** the canvas contains initiatives with mixed tags, **When** the user selects a tag in the filter control, **Then** only initiatives whose tags array includes that exact token remain visible on the canvas; all other dots are hidden entirely.
5. **Given** a tag filter is active, **When** the canvas is visible, **Then** a filter indicator showing the active tag is permanently displayed on or near the canvas; it persists until the filter is explicitly cleared.
6. **Given** a tag filter is active, **When** the user clears the filter, **Then** all previously hidden dots are restored to the canvas immediately.
7. **Given** an initiative with tags `#health #career`, **When** the user saves and reloads the project file, **Then** both tag tokens are restored exactly — same tokens, same order — with no duplication, reordering, or loss.
8. **Given** an initiative with no tags entered, **When** the detail panel is opened, **Then** the Tags field is empty; no default value or placeholder text is stored on the initiative.

---

### User Story 7 — Visualise Initiative Scale with T-Shirt Sizing (Priority: P2)

A PEP coach or executive assigns a t-shirt size (XS, S, M, L, XL) to each initiative through the detail panel. The size is reflected on the canvas as a proportionally larger or smaller dot, making initiative scale immediately comparable at a glance without opening any panel. Initiatives with no size set render at the same default dot diameter as before this feature, ensuring no visual regression for existing data.

**Why this priority**: Builds on the detail panel established in US-001 and US-005. Size adds a second visual dimension to the canvas, enriching the Impact × Urgency view with scale. P2 because it is additive — tags are the structural replacement; size can be omitted from an early iteration without breaking the feature.

**Independent Test**: Add two initiatives — assign XS to one and XL to the other. View the canvas — confirm the XL dot is visually larger. Add a third initiative with no size — confirm its dot matches the pre-feature default diameter. Save; reload — confirm all three sizes are correctly restored and the dots render at the expected diameters.

**Acceptance Scenarios**:

1. **Given** the detail panel is open for an initiative, **When** the user views the Size field, **Then** five options are available — XS, S, M, L, XL — and a blank "none" option is also available representing no size set.
2. **Given** the user selects XL and saves, **When** the canvas renders, **Then** that initiative's dot is visually larger than a dot assigned XS on the same canvas.
3. **Given** five initiatives with sizes XS, S, M, L, and XL respectively, **When** the canvas renders, **Then** all five dot diameters are visually distinguishable — no two size values produce the same dot diameter.
4. **Given** an initiative with size set to null (none), **When** the canvas renders, **Then** the dot renders at the same default diameter as existing initiatives that pre-date this feature — no visual regression.
5. **Given** an initiative with size L, **When** the user saves and reloads the project file, **Then** "L" is restored as the size and the dot renders at the L diameter on the canvas.
6. **Given** the detail panel is open for an initiative previously saved with size XS, **When** the panel renders, **Then** XS is shown as the currently selected size option.

---

### User Story 8 — Tabular Report with Bulk Tagging and Markdown Export (Priority: P3)

A PEP coach or executive switches from the canvas to a Report view that lists all initiatives in a structured table with columns for quadrant, name, tags, size, and details. They select multiple initiatives by checkbox and apply a tag to all selected rows in one bulk action. They export the full table as a `.md` file that downloads to their device and renders correctly in any standard markdown viewer.

**Why this priority**: The report view depends on tags (US-006) and size (US-007) being defined, as those are table columns. It is the endpoint for transitioning from visual brainstorming to a portable, shareable output. P3 as the last story to deliver.

**Independent Test**: With five initiatives on the canvas, switch to Report — confirm all five appear with correct Quadrant, Name, Tags, Size, and Details. Select three rows, apply tag `#reviewed`, switch back to canvas — confirm the three dots show updated tag state immediately with no reload. Export as markdown — open the downloaded `.md` file, confirm the table has the correct columns and all five initiative rows.

**Acceptance Scenarios**:

1. **Given** the TMM tool is open with at least one initiative, **When** the user activates the Report view, **Then** the view displays a table with columns: Quadrant, Name, Tags, Size, and Details, with one row per initiative.
2. **Given** the report view is open, **When** the user reads any table row, **Then** the Quadrant value is derived from the initiative's canvas position (e.g., "Q1 — Firefighting"), the Tags column lists all tag tokens space-separated, and Size shows the assigned size or is blank if unset.
3. **Given** the report view is open, **When** the user attempts to directly edit a row's Name, Size, Details, or Quadrant cell, **Then** those cells are read-only; no editing is possible from report rows.
4. **Given** the report view is open, **When** the user checks one or more row checkboxes, enters a valid tag token (e.g., `#reviewed`) in the bulk-tag input, and clicks "Apply to selected", **Then** the entered tag is appended to the tags array of all checked initiatives.
5. **Given** a bulk tag that an initiative already carries is applied again to that initiative, **When** the apply action executes, **Then** the tag is not duplicated — the initiative's tags array remains deduplicated.
6. **Given** a bulk tag operation has been applied in the report view, **When** the user returns to the canvas view, **Then** the updated tags are immediately visible in the tag filter list and on any affected dot — no page reload is required and no stale dot data is shown.
7. **Given** the user attempts to apply a bulk tag with no rows selected, **When** the "Apply to selected" action is triggered, **Then** no changes are made and the user is informed that no rows are selected.
8. **Given** the report view is open, **When** the user triggers "Export Markdown", **Then** the browser initiates a download of a `.md` file.
9. **Given** the downloaded `.md` file, **When** opened in a standard markdown viewer (e.g., GitHub, VS Code markdown preview), **Then** a correctly formatted table is rendered with heading row (Quadrant | Name | Tags | Size | Details), separator row, and one data row per initiative with no raw formatting characters visible.
10. **Given** an initiative with null size is included in an export, **When** the markdown file is rendered, **Then** the Size cell for that row is blank — the strings "null", "undefined", or any placeholder are absent.

---

### Edge Cases

- **Tag filter matches no initiatives**: Selecting a tag that no initiative carries results in an empty canvas with a clearly visible "No initiatives match this filter" message; the filter indicator remains active.
- **Bulk tag already present**: Applying a tag that an initiative already has in its array does not duplicate it — the array remains deduplicated.
- **Empty report view**: Switching to the report view with no initiatives on the canvas shows an empty table state with column headings — no crash, no error message.
- **Tags input with leading/trailing spaces**: Spaces surrounding the input string are trimmed before tokenisation; ` #health  #career ` stores `["#health", "#career"]`.
- **Long tags list display**: An initiative with ten or more tags displays in the detail panel without breaking the layout or overflowing its container.
- **Null size in canvas and export**: Initiatives with no size set render at the default dot diameter on the canvas and produce a blank Size cell in exported markdown — no placeholder leaks.
- **State sync — single source of truth**: Returning to the canvas after any report action never reconstructs initiative state from DOM elements or table rows; the shared in-memory initiatives array is always the authoritative source.

---

## Requirements *(mandatory)*

### Functional Requirements — US6: Tag Initiatives Across Multiple Dimensions

- **FR-US6.01**: The detail panel MUST replace the Category field with a Tags field; the Tags field MUST accept a space-separated string of hashtag tokens where each token matches `#` followed by one or more non-whitespace characters.
- **FR-US6.02**: The tool MUST validate tag input at form submission; any token that does not conform to the `#token` format MUST cause the entire submission to be rejected with an inline validation message — no partial save of valid tokens while invalid tokens remain.
- **FR-US6.03**: Tags MUST be stored as an ordered array of string tokens on the initiative's `tmm` section; an initiative with no tags entered MUST store an empty array — not `null` and not an empty string.
- **FR-US6.04**: The tool MUST provide a tag filter control accessible from the canvas view; selecting a tag MUST hide all initiatives whose `tags` array does not contain that exact token; no dot-fade animation is required — disappearance MAY be immediate.
- **FR-US6.05**: A filter indicator MUST be persistently visible on the canvas whenever a tag filter is active; it MUST clearly identify the active tag and disappear only when the filter is cleared.
- **FR-US6.06**: Clearing the tag filter MUST restore all previously hidden dots immediately.
- **FR-US6.07**: Tags MUST survive a save/reload cycle; the stored array MUST be restored exactly on load — same tokens, same order.
- **FR-US6.08**: The `category` field from US-003 MUST be removed from the detail panel; any `category` value present in a loaded project file MUST be silently ignored — no error displayed, no migration attempted.

### Functional Requirements — US7: Visualise Initiative Scale with T-Shirt Sizing

- **FR-US7.01**: The detail panel MUST include a Size field presenting five discrete options: XS, S, M, L, XL; a blank "none" (null) option MUST be available and selected by default for new initiatives.
- **FR-US7.02**: Saving a size selection MUST cause the initiative's canvas dot to render at the diameter corresponding to that size on the next canvas render.
- **FR-US7.03**: The tool MUST support five visually distinct dot diameters, one per size value (XS through XL); no two size values may produce the same rendered dot diameter.
- **FR-US7.04**: An initiative with null size MUST render at the same default diameter as initiatives that pre-date this feature (equivalent to the current 32px default) — no visual regression.
- **FR-US7.05**: The size value MUST survive a save/reload cycle; a saved value of "L" MUST be restored as "L" and rendered at the L dot diameter.
- **FR-US7.06**: Size is a single value per initiative (not an array); the Size field MUST accept exactly one selection or null.

### Functional Requirements — US8: Tabular Report with Bulk Tagging and Markdown Export

- **FR-US8.01**: The tool MUST provide a Report view accessible via a toolbar action or navigation element from the canvas view, and a corresponding action to return to the canvas.
- **FR-US8.02**: The report MUST list all current initiatives in a table with the following columns: Quadrant (derived from initiative `x`/`y` position), Name, Tags (all tokens), Size, and Details.
- **FR-US8.03**: Quadrant derivation MUST be deterministic: `x < 0.5` = High Urgency; `x ≥ 0.5` = Low Urgency; `y ≥ 0.5` = High Impact; `y < 0.5` = Low Impact; combined labels MUST match the canvas quadrant labels (e.g., "Q1 — Firefighting").
- **FR-US8.04**: The report table MUST be read-only for all columns except the bulk tag assignment control; Name, Size, Details, and Quadrant MUST NOT be editable from report rows.
- **FR-US8.05**: The report MUST provide a checkbox per row for initiative selection.
- **FR-US8.06**: The report MUST provide a tag input and an "Apply to selected" action; activating the action with at least one checkbox checked MUST append the entered tag token to the `tags` array of all checked initiatives, with no duplication if the tag is already present.
- **FR-US8.07**: The bulk tag input MUST validate the entered value as a single `#token` format before applying; any entry that does not conform MUST be rejected with an inline message — no partial application.
- **FR-US8.08**: If "Apply to selected" is triggered with no rows checked, the action MUST produce no changes and MUST notify the user that no initiatives are selected.
- **FR-US8.09**: Bulk tag changes MUST be immediately reflected on the canvas upon returning to the canvas view — no page reload, no reconstruction of state from DOM or table rows (Constitution Principle VI — shared data integrity).
- **FR-US8.10**: The report MUST provide an "Export Markdown" action that triggers a browser download of a `.md` file.
- **FR-US8.11**: The exported `.md` file MUST contain a standard markdown table with heading row (Quadrant | Name | Tags | Size | Details), separator row, and one data row per initiative; null size MUST render as an empty cell — not as the string "null" or "undefined".

### Cross-Cutting Requirements

- **FR-000.05**: The `category` field is removed from this tool's data model; no migration logic is required; the field is silently ignored on load.
- **FR-000.06**: Canvas tag filter state MUST NOT be persisted to the project file; filter state is session-only and MUST reset to "no filter active" on each project load.
- **FR-000.07**: The canvas and report views MUST share a single in-memory initiatives array; neither view MAY reconstruct initiative state from the DOM or from rendered table rows.

### Key Entities

- **Initiative** *(updated from US-001 spec)*: The `category` field is removed. Two new fields are added: `tags` (array of `#token` strings, default `[]`) and `size` (one of `"XS" | "S" | "M" | "L" | "XL" | null`, default `null`). All other fields (`id`, `name`, `details`, `x`, `y`) are unchanged.
- **Tag Token**: A validated string matching `#` followed by one or more non-whitespace characters. Stored in the `tags` array. Displayed as space-separated tokens.
- **T-Shirt Size**: A single enumerated value per initiative — one of XS, S, M, L, XL, or null (unset). Maps to a visually distinct dot diameter on the canvas.
- **Filter State**: A single active tag token, session-only, not persisted. Governs which initiative dots are visible on the canvas.
- **Report Row**: A derived read-only view over an initiative, displaying Quadrant (from `x`/`y`), Name, Tags, Size, and Details. Augmented with a checkbox for bulk tag selection.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-006.1** *(traces to AS US6.2, US6.7)*: Tags survive a save/reload cycle exactly as entered; the `#token` format rule is enforced at input time so no malformed tag ever reaches the stored array.
- **SC-006.2** *(traces to AS US6.4, US6.5, US6.6)*: Activating a tag filter hides all non-matching dots entirely from the canvas; a filter indicator is visible at all times while filtering and disappears only when the filter is cleared.
- **SC-007.1** *(traces to AS US7.5, US7.6)*: The selected t-shirt size is retrievable from the detail panel after a full save and reload cycle with no change to the stored value.
- **SC-007.2** *(traces to AS US7.3)*: Five initiatives each assigned a different size (XS through XL) display five visually distinguishable dot sizes on the canvas — distinguishable without opening any panel.
- **SC-008.1** *(traces to AS US8.1, US8.2)*: The report table renders all canvas initiatives with accurate values in every column — Quadrant, Name, Tags, Size, and Details — matching the current canvas state.
- **SC-008.2** *(traces to AS US8.9)*: The exported `.md` file displays a correctly structured, correctly labelled markdown table when opened in GitHub markdown preview and VS Code markdown preview — no raw formatting characters and no placeholder text visible.
- **SC-008.3** *(traces to AS US8.6)*: Bulk tag changes applied in the report view are immediately presented on the canvas upon returning to the canvas view — no page reload, no stale initiative data.

---

## Assumptions

- The `category` field is a proof-of-concept with no user data worth preserving; clean replacement with `tags` is confirmed as the correct approach.
- Tag filtering supports single-tag filtering only in this feature; multi-tag AND/OR compound filtering is deferred.
- The tag filter source list (which tags are available to filter by) is derived from the union of tags present on current canvas initiatives; there is no separate tag-management screen.
- T-shirt size maps to dot diameter only; no numeric story-point, Agile level, or complexity mapping is in scope.
- The Report view replaces the canvas view (full-screen switch); the exact navigation mechanism (toolbar button, tab, back link) is an implementation decision for the plan phase.
- Quadrant labels in the report are derived deterministically from `x`/`y` coordinates and match the canvas quadrant names.
- Canvas tag filter state resets to "no filter" on every project load; preserving the last-used filter across loads is not a requirement.
- The markdown export is a browser file download trigger; PDF, print, or clipboard export paths are out of scope.
- Existing initiatives in loaded project files that have no `tags` field or a `category` field are valid; the tool MUST handle them without error — `tags` defaults to `[]` if absent, `category` is ignored if present.
