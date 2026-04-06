# Feature Specification: Time Management Matrix (TMM)

**Feature Branch**: `001-tmm`  
**Created**: 2026-04-06  
**Status**: Draft  
**Governance**: [Constitution v1.3.0](../../.specify/memory/constitution.md) | [Solo Developer Principles](../../.specify/memory/solo-developer-principles.md)  
**Input**: [specs/001-tmm/business-statement.md](./business-statement.md)

## Context

The TMM is Tool 1 in the Effectiveness Toolkit suite. It is a four-quadrant visual canvas based on Stephen Covey's Impact × Urgency model, used in PEP coaching to help coaches and executives see at a glance where their initiatives (Big Rocks) sit. This MVP delivers the canvas as a standalone HTML tool with drag-and-drop repositioning, initiative detail viewing, and save/load to a JSON file using the shared `effectivenessToolkit` envelope for future integration compatibility.

> Constitutional governance (Principles I–VI) applies. Principles are not repeated here — they are referenced where relevant.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Place and View Initiatives on the Canvas (Priority: P1)

A PEP coach or executive opens the TMM tool in a browser and sees a four-quadrant Impact × Urgency canvas. They add a new initiative by entering a name, a user-defined category, and an optional details field. The initiative appears as a dot on the canvas in the correct quadrant. They can click the dot to view the initiative's details without leaving the canvas view.

**Why this priority**: This story delivers the entire core value of the tool. Without the canvas, dot placement, and detail view, no other story has context. It is the shippable MVP on its own.

**Independent Test**: Open the tool, add an initiative with a name, category, and details text. Confirm a dot appears on the canvas in the expected quadrant. Click the dot and confirm the name, category, and details are visible.

**Acceptance Scenarios**:

1. **Given** the TMM tool is open, **When** the user opens the "Add Initiative" form, **Then** the form shows three fields: Name (required, single-line), Category (free text, optional), and Details (free text, multi-line, optional).
2. **Given** the "Add Initiative" form is open, **When** the user submits it with a name and category but no details, **Then** the initiative is created and a dot appears on the canvas at a neutral default position (canvas centre).
3. **Given** a dot is on the canvas, **When** the user clicks the dot, **Then** a detail panel or overlay opens showing the initiative's name, category, and details text without navigating away from the canvas.
4. **Given** the detail panel is open, **When** the user closes it, **Then** the canvas returns to its previous state with all dots visible and in their last positions.
5. **Given** the canvas is loaded with at least one initiative, **When** the user views the canvas, **Then** each dot is visually labelled with the initiative name so initiatives are distinguishable at a glance.

---

### User Story 2 — Drag-and-Drop Repositioning (Priority: P2)

A PEP coach or executive repositions an initiative on the canvas by dragging its dot to a new location. The new position maps to updated Impact and Urgency coordinates. The updated position persists after a save/reload cycle.

**Why this priority**: The core differentiator of the TMM over a ranked list is the freely-positionable canvas. Without drag-and-drop, the tool lacks its key interaction. P2 because the canvas must exist first (US1) before repositioning is meaningful.

**Independent Test**: Add an initiative, drag its dot from the top-right quadrant to the bottom-left quadrant, save the file, reload — the dot appears in the bottom-left quadrant with updated coordinates.

**Acceptance Scenarios**:

1. **Given** a dot on the canvas, **When** the user starts dragging the dot, **Then** the dot follows the mouse cursor in real time while constrained within the canvas boundary.
2. **Given** the user is dragging a dot, **When** they release the mouse button, **Then** the dot remains at the released position; the initiative's Impact and Urgency values update to reflect the new canvas coordinates.
3. **Given** a dot positioned in the "High Impact, Low Urgency" quadrant, **When** the user drags it to the "Low Impact, High Urgency" quadrant and releases, **Then** the dot's coordinates reflect the new quadrant.
4. **Given** the user attempts to drag a dot beyond the canvas edge, **When** they release, **Then** the dot stops at the canvas boundary and does not disappear or produce an error.
5. **Given** initiatives are repositioned, **When** the user saves and reloads the file, **Then** all dot positions are restored to their saved coordinates.

---

### User Story 3 — Assign a Category to Each Initiative (Priority: P3)

A PEP coach or executive assigns a user-defined category label (e.g., "Health", "Career", "Family") to each initiative when adding it, and can see the category when viewing the initiative's detail panel. Categories are free text and are stored per initiative with no system-managed list.

**Why this priority**: Enhances the organisation value of the canvas for users who manage Big Rocks across multiple life areas or roles. The category field is part of the add-initiative form (US1) so it is captured at the same time, but grouping or filtering by category is deferred. P3 because the canvas delivers value without visual category grouping.

**Independent Test**: Add two initiatives with different category values. Click each dot and confirm the assigned category is visible in the detail panel.

**Acceptance Scenarios**:

1. **Given** the "Add Initiative" form, **When** the user types a category in the Category field, **Then** it is accepted as free text with no predefined list, validation, or format restriction.
2. **Given** an initiative with a category of "Career", **When** the user clicks its dot and opens the detail panel, **Then** "Career" is displayed as the category.
3. **Given** an initiative with no category entered, **When** the user opens its detail panel, **Then** the category field is displayed as blank (no placeholder or default value substituted).
4. **Given** two initiatives with categories "Health" and "Career", **When** both dots are on the canvas, **Then** both categories are correctly associated with their respective dots (no category cross-contamination between initiatives).
5. **Given** an initiative with a category, **When** the user saves and reloads the project file, **Then** the category value is restored exactly as entered.

---

### User Story 4 — Save and Load the Canvas State (Priority: P1)

A PEP coach or executive saves the current canvas state — all initiatives with their names, categories, details, and positions — to a JSON file. They can later reload the file to restore the full canvas.

**Why this priority**: Without persistence, all canvas work is lost on tab close. Save/load is the core data-survival mechanism and belongs alongside US1 as a P1 deliverable (a canvas that cannot be saved has no practical value).

**Independent Test**: Add two initiatives, position them, save to file. Close and reopen the tool, load the saved file — both dots appear at their saved positions with all fields intact.

**Acceptance Scenarios**:

1. **Given** at least one initiative on the canvas, **When** the user triggers "Save", **Then** the browser prompts for a save location and filename; the file is written as valid JSON using the `effectivenessToolkit` envelope with a `tmm` section.
2. **Given** a saved project file, **When** the user opens the tool and loads the file, **Then** all initiatives are restored with their name, category, details, and canvas position (Impact/Urgency coordinates).
3. **Given** a project file where `coer`, `sob`, `memoryMap`, and `impactMap` sections are `null` or absent, **When** loaded in the TMM tool, **Then** the tool loads and operates correctly without errors; on re-save those sections are written back exactly as received — no data is added, altered, or defaulted by the TMM tool (Constitution Principle VI — Tool Independence).
4. **Given** the user saves, then adds another initiative and saves again, **When** the file is reloaded, **Then** all initiatives from both saves are present and no data is lost.
5. **Given** a save from this tool, **When** the JSON file is inspected, **Then** the `effectivenessToolkit` envelope is present with `version`, `lastModified`, and a `tmm` section per initiative containing `id`, `name`, `category`, `details`, `x`, and `y`  fields.

---

### User Story 5 — Edit an Existing Initiative (Priority: P2)

A PEP coach or executive opens the detail panel for an existing initiative and edits its name, category, or details. The changes are reflected immediately on the canvas dot label and persist after a save/reload cycle.

**Why this priority**: Without editing, any data entry mistake requires deleting and re-creating the initiative, which also resets its canvas position. Editing is essential for practical day-to-day use. P2 because the initiative must exist first (US1) before it can be edited.

**Independent Test**: Add an initiative named "Health", open its detail panel, edit the name to "Fitness", close the panel — the dot label updates to "Fitness". Save and reload — the name "Fitness" is present.

**Acceptance Scenarios**:

1. **Given** the detail panel is open for an initiative, **When** the user activates the Edit action (button or inline field click), **Then** the Name, Category, and Details fields become editable.
2. **Given** the user has edited one or more fields, **When** they confirm the changes, **Then** the panel closes, the dot label on the canvas updates immediately to reflect the new name, and the edited values are stored in the canvas state.
3. **Given** the user has edited one or more fields, **When** they cancel the edit, **Then** all fields revert to their original values and no changes are applied to the canvas state.
4. **Given** the user clears the Name field and attempts to confirm, **Then** the edit is rejected with an inline validation message — Name is required.
5. **Given** an initiative whose name, category, and details have been edited, **When** the user saves and reloads the project file, **Then** the edited values are restored exactly.

---

### Edge Cases

- **Canvas with no initiatives**: An empty canvas renders correctly with the four quadrant labels visible and no error state.
- **Single dot drag to boundary**: Dragging a dot to the exact canvas edge positions it at the boundary, not outside it.
- **Long initiative name**: An initiative name longer than 30 characters is truncated visually on the dot label but stored and displayed in full in the detail panel.
- **Name-only initiative**: An initiative with only a name (no category, no details) is valid — no field is mandatory except name.
- **Multiple initiatives same position**: Two dots placed at identical coordinates are both rendered (stacked or slightly offset); neither is hidden.
- **Load file with no tmm section**: If a project file has no `tmm` key, the tool starts with an empty canvas — no crash, no error message shown to user.
- **Load file with unknown extra fields**: Unknown fields within an initiative's `tmm` section are preserved on re-save without being stripped.

---

## Requirements *(mandatory)*

### Functional Requirements — US1: Place and View Initiatives

- **FR-US1.01**: The tool MUST display a four-quadrant canvas with Impact on the vertical axis (high at top, low at bottom) and Urgency on the horizontal axis (high at right, low at left); each quadrant MUST be labelled.
- **FR-US1.02**: The tool MUST provide an "Add Initiative" action that opens a form with three fields: Name (required, single-line text), Category (optional, free text), and Details (optional, multi-line free text).
- **FR-US1.03**: On form submission, the tool MUST create a dot on the canvas representing the new initiative; the dot MUST be placed at the canvas centre (50% Impact, 50% Urgency) as the default position.
- **FR-US1.04**: Each dot MUST display the initiative's name as a visible label directly on or adjacent to the dot.
- **FR-US1.05**: Clicking a dot MUST open a detail panel or overlay showing the initiative's name, category, and details text.
- **FR-US1.06**: The detail panel MUST be closeable without navigating away from the canvas; all other dots and the canvas layout MUST remain visible after closing the panel.
- **FR-US1.07**: The canvas MUST remain usable (add, click) when 10 or more initiatives are present.

### Functional Requirements — US2: Drag-and-Drop Repositioning

- **FR-US2.01**: Each dot MUST be draggable using native browser drag-and-drop APIs (no third-party drag libraries).
- **FR-US2.02**: During a drag operation, the dot MUST follow the cursor in real time, constrained to the canvas boundaries.
- **FR-US2.03**: On drag release, the dot's canvas position MUST be stored as normalised Impact (0.0–1.0, bottom-to-top) and Urgency (0.0–1.0, left-to-right) coordinate values.
- **FR-US2.04**: A dot MUST NOT be draggable beyond the canvas boundary; if dropped outside, it MUST snap to the nearest boundary edge.
- **FR-US2.05**: Position changes MUST be reflected in the saved project file on the next save action.

### Functional Requirements — US3: Category Assignment

- **FR-US3.01**: The Category field in the "Add Initiative" form MUST accept any free-text string with no predefined options, no format validation, and no uniqueness constraint.
- **FR-US3.02**: The category value MUST be stored per initiative and retrieved independently of other initiatives' category values.
- **FR-US3.03**: The category MUST be displayed in the initiative's detail panel every time it is opened.
- **FR-US3.04**: An initiative with no category entered MUST store an empty string and display an empty category field in the detail panel (no default substituted).

### Functional Requirements — US4: Save and Load

- **FR-US4.01**: The tool MUST provide a "Save" action that writes the full canvas state to a JSON project file via the browser's file-save mechanism.
- **FR-US4.02**: The saved JSON MUST use the `effectivenessToolkit` envelope: `{ "effectivenessToolkit": { "version": "1.0", "lastModified": "<ISO 8601>", "initiatives": [...] } }`.
- **FR-US4.03**: Each initiative in the `initiatives` array MUST have `id` (unique string) and `name` (string) at the initiative level (not inside the `tmm` section). The `tmm` section MUST contain only TMM-specific fields: `category` (string), `details` (string), `x` (number, normalised 0.0–1.0), `y` (number, normalised 0.0–1.0).
- **FR-US4.04**: The tool MUST provide a "Load" action that reads a JSON project file from the user's file system and restores all initiatives and their positions onto the canvas.
- **FR-US4.05**: On load, the tool MUST NOT modify or discard sections of the project file belonging to other tools (`coer`, `sob`, `memoryMap`, `impactMap`); those sections MUST be written back unchanged on the next save (Constitution Principle VI).
- **FR-US4.06**: If a loaded project file contains no `tmm` section or an empty `initiatives` array, the tool MUST display an empty canvas with no error shown to the user.
- **FR-US4.07**: The tool MUST update the envelope's top-level `lastModified` field to the current ISO 8601 date-time on each save.

### Functional Requirements — US5: Edit an Existing Initiative

- **FR-US5.01**: The detail panel MUST provide an Edit action that switches Name, Category, and Details fields into an editable state.
- **FR-US5.02**: Name MUST remain a required field during editing; submitting with an empty Name MUST be rejected with an inline validation message.
- **FR-US5.03**: On confirming an edit, the initiative's stored values MUST be updated immediately; the dot label on the canvas MUST reflect the new name without requiring a page reload.
- **FR-US5.04**: On cancelling an edit, all field values MUST revert to their state before the edit was activated; no changes MUST be written to the canvas state.
- **FR-US5.05**: Edited values MUST be included in the next save operation and restored correctly on reload.

### Cross-Cutting Requirements

- **FR-000.01**: The tool MUST be delivered as a standalone HTML file deployable to `public/tools/tmm/` with no build tooling required (Constitution Principle V — Simplicity; Technical Guardrails — Form-Based Tools).
- **FR-000.02**: All drag-and-drop interaction MUST use native browser APIs only; no third-party UI or drag libraries are permitted.
- **FR-000.03**: The tool MUST function fully offline — no network requests after the file is opened.
- **FR-000.04**: The tool MUST NOT require any tool section other than `tmm` to be populated in the project file in order to function (Constitution Principle VI — Tool Independence).

### Key Entities

- **Initiative**: A named Big Rock the user is managing. Fields: `id` (unique string), `name` (string), `category` (string), `details` (string), canvas position stored as normalised `x` and `y` coordinates (0.0–1.0).
- **Canvas Position**: Normalised coordinates mapping the dot's placement to Impact (`y`, 0 = low, 1 = high) and Urgency (`x`, 0 = low, 1 = high).
- **Project File**: The shared JSON file owned by the user; uses the `effectivenessToolkit` envelope with `version`, `lastModified`, and `initiatives[]`. The TMM tool reads and writes only the `tmm` section of each initiative (Constitution Principle VI).

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001** *(traces to AS US1.2, US1.5)*: A user can open the tool in a browser, add an initiative, and see it as a labelled dot on the canvas in under 1 minute from a standing start.
- **SC-002** *(traces to AS US2.2, US2.5)*: A user can drag a dot to a new quadrant and, after a save/reload cycle, confirm the dot appears in the same quadrant — with no data loss and no positioning error greater than 1% of canvas size.
- **SC-003** *(traces to AS US4.2)*: A user can save a project file containing five initiatives and reload it; all initiatives are restored with their names, categories, details, and positions intact.
- **SC-004** *(traces to AS US1.5, US3.2)*: A user can click any dot and immediately view the initiative's name, category, and details without leaving the canvas view.
- **SC-005** *(traces to Edge Cases)*: The tool handles 15 initiatives on the canvas without visible performance degradation (drag response remains immediate, canvas renders without delay).

---

## Assumptions

- Initiative name is the primary identifier; uniqueness is not enforced — the user is responsible for distinguishing initiatives by name.
- Canvas coordinates are stored as normalised floats (0.0–1.0) so the layout is resolution-independent when the file is shared across devices with different screen sizes.
- Save is explicitly user-triggered (no auto-save); data in the browser is lost if the tab is closed without saving.
- The default dot placement at canvas centre (50%/50%) is sufficient for the initial add; the user is expected to drag dots to their preferred positions after adding.
- No undo/redo is required for the MVP.
- The standalone HTML file embeds all JavaScript inline; no external scripts or CSS files are loaded at runtime.
- No cross-browser polyfills beyond the standard HTML5 drag-and-drop API are needed; modern evergreen browsers (Chrome, Edge, Firefox, Safari) are the target.

---

## Deferred (Post-MVP)

- Markdown rendering of the details field.
- Visual grouping or filtering by category on the canvas.
- Category management table or reuse suggestions.
- Integration with COER, Memory Map, or SoB JSON project files (shared multi-initiative file selection).
- Undo/redo of drag operations.
- Predefined or mandated category lists.
- LLM-generated initiative descriptions.
- Touch/mobile drag support.
