# Tasks: Time Management Matrix (TMM)

**Feature**: `001-tmm` | **Branch**: `001-tmm` | **Date**: 2026-04-06
**Input**: `specs/001-tmm/` — plan.md, spec.md, data-model.md, contracts/project-file-schema.md, research.md, quickstart.md
**Tech Stack**: Vanilla JavaScript (ES6+), HTML5, CSS3 — zero build tooling, zero dependencies
**Delivery**: `public/tools/tmm/index.html` — single standalone HTML file (all JS/CSS inline)
**Tests**: Manual test scenarios only (per Constitution §VII — Standalone HTML Tools Exception). No automated test tasks included.

## Format: `[ID] [P?] [Story?] Description — file path`

- **[P]**: Task can execute in parallel with other [P] tasks in the same phase (different sections of the same file — coordinate to avoid merge conflicts)
- **[US1–US5]**: User story this task belongs to
- Task IDs: `T001–T002` = setup | `T003–T006` = foundational | `T007–T011` = US1 | `T012–T015` = US4 | `T016–T020` = US2 | `T021–T023` = US5 | `T024–T026` = US3 | `T027–T030` = polish

---

## Phase 1: Setup

**Purpose**: Create the file structure and skeleton. No user-visible behaviour yet.

- [ ] T001 Create directory `public/tools/tmm/` and skeleton file `public/tools/tmm/index.html` with valid HTML5 boilerplate (doctype, `<meta charset>`, viewport, title "TMM — Time Management Matrix", empty `<body>`)
- [ ] T002 Add TMM tool link to the suite launcher at `public/index.html` (consistent with existing COER link style)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core in-memory state, UUID generation, and HTML/CSS skeleton that ALL user story phases depend on. No user story work begins until this phase is complete.

**⚠️ CRITICAL**: Phases 3–7 cannot start until T003–T006 are complete.

- [ ] T003 Implement in-memory state object (`initiatives[]`, `loadedProjectFile`, `selectedInitiativeId`, `isEditing`) and stub `renderCanvas()` dispatcher function that iterates `initiatives[]` and syncs DOM in `public/tools/tmm/index.html`
- [ ] T004 [P] Implement `generateUUID()` function (UUID v4 — matches regex `/^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i`) inline in `public/tools/tmm/index.html`
- [ ] T005 [P] Implement HTML skeleton: toolbar `<header>` (Add, Save, Load buttons), canvas container `<div id="canvas">`, hidden detail panel `<aside id="detail-panel">`, hidden add-initiative `<dialog>` or modal overlay in `public/tools/tmm/index.html`
- [ ] T006 Implement CSS foundation: custom properties (`--color-q1` through `--color-q4`, spacing, typography), four-quadrant grid layout using CSS Grid with Impact (Y) and Urgency (X) axis labels, quadrant headings (Q1 Do / Q2 Plan / Q3 Delegate / Q4 Eliminate), and canvas `position: relative` overflow boundary in `public/tools/tmm/index.html`

**Checkpoint**: Foundation complete — US1, US4, US2, US5, US3 phases can now begin sequentially in priority order

---

## Phase 3: User Story 1 — Place and View Initiatives on the Canvas (Priority: P1) 🎯 MVP

**Goal**: A user can open the tool, add a named initiative with optional category and details, see it appear as a labelled dot at canvas centre, and click the dot to view its details in a panel — without leaving the canvas.

**Independent Test**: Open `public/tools/tmm/index.html`, click "Add Initiative", enter a name, category, and details text, submit the form — a labelled dot appears at canvas centre. Click the dot — a detail panel opens showing name, category, and details. Close the panel — canvas restores with the dot in place.

- [ ] T007 [US1] Implement "Add Initiative" toolbar button that opens the modal form with three fields: Name (`<input type="text">`, required), Category (`<input type="text">`, optional free text), and Details (`<textarea>`, optional multi-line) in `public/tools/tmm/index.html`
- [ ] T008 [US1] Implement form submission handler: validate Name non-empty (block submit + show inline error if empty), generate UUID v4 id, create initiative object `{ id, name, coer: null, sob: null, memoryMap: null, impactMap: null, tmm: { category: "", details: "", x: 0.5, y: 0.5 } }` where `category` and `details` are populated from the form fields inside the `tmm` section (not at initiative level), push to `initiatives[]`, close modal, call `renderCanvas()` in `public/tools/tmm/index.html`
- [ ] T009 [US1] Implement `renderCanvas()`: for each initiative, create an absolutely-positioned dot `<div class="dot">` at `left = x * 100%`, `top = (1 - y) * 100%`, with a visible name label using CSS `text-overflow: ellipsis` (max visual width ~30 chars); bind click handler per dot in `public/tools/tmm/index.html`
- [ ] T010 [US1] Implement dot click handler: set `selectedInitiativeId`, populate detail panel `<aside>` with initiative name, category, and details (read-only view mode), make panel visible — canvas and all other dots remain visible behind the panel in `public/tools/tmm/index.html`
- [ ] T011 [US1] Implement detail panel close button: clear `selectedInitiativeId`, set `isEditing = false`, hide panel, return canvas to full view — all dots remain in their last positions in `public/tools/tmm/index.html`

**Checkpoint**: User Story 1 fully functional — shippable MVP (canvas, add, dot render, detail view, close panel)

---

## Phase 4: User Story 4 — Save and Load the Canvas State (Priority: P1)

**Goal**: A user can save all initiatives with their names, categories, details, and positions to a JSON file, and later reload the file to restore the full canvas. Other tool sections in the project file are preserved unchanged.

**Independent Test**: Add two initiatives, position one by editing x/y manually, click "Save" — browser prompts for file save. Reload the tool, click "Load", pick the saved file — both dots appear at their saved positions with all fields intact. Open the JSON — `effectivenessToolkit` envelope is present with correct structure.

- [ ] T012 [US4] Implement "Save" toolbar button and save action: build `effectivenessToolkit` JSON envelope (`version: "1.0"`, `lastModified: new Date().toISOString()`, `initiatives: [...]`); for each in-memory initiative merge updated `tmm` section (`category`, `details`, `x`, `y`) with any existing sections from `loadedProjectFile`; trigger download via `Blob` + hidden `<a download>` in `public/tools/tmm/index.html`
- [ ] T013 [US4] Implement round-trip preservation in the serialiser: when building the save payload, copy each initiative's `coer`, `sob`, `memoryMap`, and `impactMap` from `loadedProjectFile` unchanged (or `null` for new initiatives); the TMM tool MUST NOT alter or default any non-`tmm` section (Constitution Principle VI) in `public/tools/tmm/index.html`
- [ ] T014 [US4] Implement "Load" toolbar button and load action: open file picker via `<input type="file" accept=".json">`, read file with FileReader, parse JSON, store full parsed object in `loadedProjectFile`, extract `initiatives[]` and their `tmm` sections into in-memory state, call `renderCanvas()` in `public/tools/tmm/index.html`
- [ ] T015 [US4] Handle load edge cases: no `tmm` key on an initiative → skip that initiative's tmm data (no crash); missing or empty `initiatives` array → render empty canvas silently; unknown extra fields within any `tmm` section → preserved in `loadedProjectFile` and written back on next save; invalid JSON → show brief console.error, no user-facing crash in `public/tools/tmm/index.html`

**Checkpoint**: User Stories 1 + 4 complete — canvas work survives tab close (P1 shippable pair)

---

## Phase 5: User Story 2 — Drag-and-Drop Repositioning (Priority: P2)

**Goal**: A user drags an initiative dot to any point on the canvas and releases — the dot stays at the new position, constrained to the canvas boundary. Updated coordinates are included in the next save.

**Independent Test**: Add an initiative, drag its dot from canvas centre to the top-right area, release — dot stays at new position. Save and reload — dot appears in the same top-right position with coordinates matching the drop location.

- [ ] T016 [US2] Add drag-specific CSS to all dot elements: `touch-action: none` (prevents scroll hijack), `will-change: transform` (GPU compositing), `cursor: grab` (idle) / `cursor: grabbing` (active) in `public/tools/tmm/index.html`
- [ ] T017 [US2] Implement `pointerdown` handler on each dot: record pointer offset from dot's top-left corner, call `dot.setPointerCapture(e.pointerId)`, add `dragging` CSS class — dragging begins on next `pointermove` in `public/tools/tmm/index.html`
- [ ] T018 [US2] Implement `pointermove` handler: compute new `left`/`top` pixel values from pointer coordinates minus recorded offset, clamp to canvas bounds (`Math.max(0, Math.min(canvasWidth - dotWidth, newLeft))` / same for top), apply via `dot.style.left` / `dot.style.top` in real time in `public/tools/tmm/index.html`
- [ ] T019 [US2] Implement `pointerup` handler: call `dot.releasePointerCapture(e.pointerId)`, remove `dragging` class, compute normalised coordinates (`x = pixelLeft / canvasWidth`, `y = 1 - pixelTop / canvasHeight`), clamp both to [0.0, 1.0], update the matching initiative in `initiatives[]`, call `renderCanvas()` in `public/tools/tmm/index.html`
- [ ] T020 [US2] Verify boundary clamping: drag a dot to each canvas edge and beyond — dot stops at boundary, does not disappear, coordinates stay within [0.0, 1.0]; verify canvas remains responsive with 15 simultaneous initiative dots (SC-005) in `public/tools/tmm/index.html`

**Checkpoint**: User Stories 1 + 4 + 2 complete — drag-and-drop repositioning works and persists

---

## Phase 6: User Story 5 — Edit an Existing Initiative (Priority: P2)

**Goal**: A user opens the detail panel for an existing initiative, edits its name, category, or details, and immediately sees the dot label update on the canvas. Changes persist after save/reload.

**Independent Test**: Add an initiative named "Health", open its detail panel, click Edit, change the name to "Fitness", confirm — the dot label updates to "Fitness" without page reload. Save and reload — "Fitness" is present with the correct position.

- [ ] T021 [US5] Add "Edit" button to the detail panel view mode: clicking switches Name, Category, and Details to editable `<input>`/`<textarea>` fields pre-populated with the current initiative values; sets `isEditing = true` in `public/tools/tmm/index.html`
- [ ] T022 [US5] Implement confirm-edit handler: validate Name field non-empty (reject submit + show inline validation message if empty — FR-US5.02), update the matching initiative in `initiatives[]` with new name/category/details, call `renderCanvas()` so the dot label reflects the new name immediately, close the panel in `public/tools/tmm/index.html`
- [ ] T023 [US5] Implement cancel-edit handler: revert all form field values to their state at the moment Edit was activated using a snapshot taken on edit open, set `isEditing = false`, display view mode — no changes written to `initiatives[]` in `public/tools/tmm/index.html`

**Checkpoint**: User Stories 1 + 4 + 2 + 5 complete — full edit lifecycle with immediate canvas feedback

---

## Phase 7: User Story 3 — Category Assignment (Priority: P3)

**Goal**: The Category field is fully functional as free text per initiative, independently stored, correctly displayed in the detail panel, and persistent across save/load cycles.

**Independent Test**: Add two initiatives with categories "Health" and "Career". Click each dot — confirm each detail panel shows the correct category. Save and reload — both categories are restored exactly as entered.

- [ ] T024 [US3] Verify Category input in "Add Initiative" form: accepts any free-text string with no format validation, no predefined list, and no uniqueness constraint (FR-US3.01); when left blank, stores empty string `""` so detail panel displays blank with no placeholder or default value substituted (FR-US3.04) in `public/tools/tmm/index.html`
- [ ] T025 [P] [US3] Verify detail panel displays each initiative's category independently when clicked — click both "Health" and "Career" dots in turn and confirm no category cross-contamination (FR-US3.02, FR-US3.03); category field empty for name-only initiatives in `public/tools/tmm/index.html`
- [ ] T026 [US3] Verify category survives save/load round-trip: `category` field present in the `tmm` section of the saved JSON with the exact string value entered; restored correctly after load without alteration (per US4 save contract — FR-US4.03) in `public/tools/tmm/index.html`

**Checkpoint**: All five user stories independently functional

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Edge case hardening affecting multiple user stories; final acceptance validation.

- [ ] T027 [P] Add edge case handling: empty canvas (zero initiatives) renders correctly — four quadrant labels visible, axes clear, no JS errors, add button available (spec §Edge Cases — "Canvas with no initiatives") in `public/tools/tmm/index.html`
- [ ] T028 [P] Add edge case handling: initiative name longer than 30 characters is visually truncated on the dot label with CSS `text-overflow: ellipsis`; full name displayed without truncation in the detail panel Name field (spec §Edge Cases — "Long initiative name") in `public/tools/tmm/index.html`
- [ ] T029 [P] Add edge case handling: two initiatives placed at identical canvas coordinates are both rendered — neither hidden; apply a small CSS offset (`transform: translate(...)`) so both dots are distinguishable (spec §Edge Cases — "Multiple initiatives same position") in `public/tools/tmm/index.html`
- [ ] T030 Run all quickstart.md manual validation scenarios against the completed `public/tools/tmm/index.html`; confirm SC-001 through SC-005 pass

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Phase 1 — BLOCKS all user story phases
- **US1 (Phase 3)**: Depends on Phase 2 — primary P1 story; start first
- **US4 (Phase 4)**: Depends on Phase 3 — save/load requires initiatives state from US1
- **US2 (Phase 5)**: Depends on Phase 3 — dragging requires rendered dots from US1
- **US5 (Phase 6)**: Depends on Phase 3 — editing requires detail panel from US1
- **US3 (Phase 7)**: Depends on Phase 3 — category field already exists from US1; this phase verifies FR-US3 compliance
- **Polish (Phase 8)**: Depends on all user story phases complete

### User Story Dependencies

| Story | Depends On | Can Run Alongside |
|-------|-----------|-------------------|
| US1 (P1) | Foundational phase | — |
| US4 (P1) | US1 (needs initiatives state) | US2, US5, US3 with care |
| US2 (P2) | US1 (needs rendered dots) | US4, US5, US3 with care |
| US5 (P2) | US1 (needs detail panel) | US4, US2, US3 with care |
| US3 (P3) | US1 (category field in form) | After US4 for round-trip test |

> Note: Because all work targets a single HTML file, parallel execution requires careful coordination to avoid conflicting edits. Sequential delivery in priority order (US1 → US4 → US2 → US5 → US3) is the safest approach for a solo developer.

### Within Each User Story

- HTML structure before event handlers
- State mutations before DOM rendering calls
- Core feature complete before edge case hardening (Phase 8)

---

## Parallel Opportunities

### Phase 2 (Foundational)

```
T003 (state + renderCanvas stub)
     ↓ parallel with T004 [P] (generateUUID) and T005 [P] (HTML skeleton)
T006 (CSS foundation — depends on T005 HTML skeleton)
```

### Phase 3 (US1)

```
T007 (Add Initiative form)
T008 (form submission — depends on T007)
T009 (renderCanvas implementation — depends on T008)
T010 (dot click handler — depends on T009)
T011 (panel close — depends on T010)
```
Sequential within US1 — each task builds on the previous.

### Phase 5 (US2 Drag)

```
T016 [P] CSS for drag (can run alongside T017 setup)
T017 pointerdown
T018 pointermove (depends on T017)
T019 pointerup (depends on T018)
T020 verification (depends on T019)
```

### Phase 8 (Polish)

```
T027 [P] + T028 [P] + T029 [P] — all target independent edge cases, can run in parallel
T030 (validation — depends on T027–T029)
```

---

## Implementation Strategy

### MVP Scope (P1 only — Phases 1–4)

Deliver T001–T015 for a shippable tool:
- User can add initiatives and see them as labelled dots (US1)
- User can save and reload the canvas — data survives (US4)
- Zero data loss, offline-only, single HTML file

### Increment 2 (P2 — Phases 5–6)

Add T016–T023 for the full interactive experience:
- Drag-and-drop repositioning with Pointer Events (US2)
- In-place editing of initiative name, category, and details (US5)

### Increment 3 (P3 + Polish — Phases 7–8)

Add T024–T030:
- Category assignment compliance verification (US3)
- Edge case hardening and final validation

### Commit Convention

```
feat(tmm): description [T-XXX]
```

Example: `feat(tmm): implement four-quadrant canvas layout [T006]`
