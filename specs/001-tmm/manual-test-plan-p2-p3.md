# Manual Test Plan — TMM Phases 5–8 (P2 / P3)

**Tool**: `public/tools/tmm/index.html`
**Branch**: `001-tmm`
**Tester**: Claudio Gomes (PO / UAT owner)
**Scope**: T016–T030 — Drag-and-Drop, Edit, Category, Polish
**Prerequisite**: Phases 1–4 UAT complete ✓ | **Bug 1 (quadrant layout) fixed and deployed** before starting

> Constitution §VII — Standalone HTML Tools Exception: all verification is manual.
> Copilot has no automated test tasks in this phase.

---

## Canvas Layout Reference (after Bug 1 fix)

```
More Impact ▲
            │  Q1 Firefighting  │  Q2 Growth
            │  (top-left)       │  (top-right)
            ├───────────────────┼───────────────────
            │  Q3 Distraction   │  Q4 Waste
            │  (bottom-left)    │  (bottom-right)
            └──────────────────────────────────────▶
         More Urgent                         Less Urgent
```

- **Top-left** = High Impact + High Urgency (Q1 Firefighting)
- **Top-right** = High Impact + Low Urgency (Q2 Growth)
- **Bottom-left** = Low Impact + High Urgency (Q3 Distraction)
- **Bottom-right** = Low Impact + Low Urgency (Q4 Waste)

---

## Pre-Flight Checklist

- [X] Open `public/tools/tmm/index.html` directly in the browser (no server needed)
- [X] Confirm quadrant layout matches the reference above (Bug 1 fix verified)
- [X] Confirm four quadrant labels are visible on an empty canvas

---

## Phase 5 — User Story 2: Drag-and-Drop Repositioning

> **Goal**: A dot can be dragged to any canvas position; it stays there after release; coordinates persist after save/reload.

### Scenario DnD-1: Basic drag follows the cursor

- [X] Add an initiative named "Drag Test"
- [X] Hover over the dot — confirm cursor changes to `grab` (hand icon)
- [X] Press and hold the dot — confirm cursor changes to `grabbing`
- [X] Slowly drag the dot toward the top-left area (Q1 Firefighting)
- [X] **Expected**: dot follows the cursor smoothly in real time; no lag or jump

### Scenario DnD-2: Dot stays at release position

- [X] Drag the "Drag Test" dot to the top-right area (Q2 Growth)
- [X] Release the mouse
- [X] **Expected**: dot remains at the released position; no snap-back to centre

### Scenario DnD-3: Quadrant change reflects Impact / Urgency values

- [X] Drag a dot from the top-left quadrant (Q1) to the bottom-right quadrant (Q4)
- [X] Release
- [X] Click the dot to open the detail panel
- [X] **Expected**: panel opens showing the same name / category / details as before; position change does not corrupt initiative data

### Scenario DnD-4: Boundary clamping — left edge

- [X] Drag a dot to the far left edge of the canvas and continue dragging beyond it
- [X] Release
- [X] **Expected**: dot stops at the left boundary; it does not disappear or overflow; no JS error in console

### Scenario DnD-5: Boundary clamping — right edge

- [X] Drag a dot to the far right edge and release outside the canvas
- [X] **Expected**: dot stops at right boundary; stays visible

### Scenario DnD-6: Boundary clamping — top and bottom edges

- [X] Drag a dot to the top edge and release outside
- [X] **Expected**: dot stops at top boundary
- [X] Drag the same dot to the bottom edge and release outside
- [X] **Expected**: dot stops at bottom boundary

### Scenario DnD-7: Position persists after save / reload

- [X] Add two initiatives: "Alpha" and "Beta"
- [X] Drag "Alpha" to the top-left quadrant (Q1 Firefighting)
- [X] Drag "Beta" to the bottom-right quadrant (Q4 Waste)
- [X] Click **Save** — confirm browser file-save dialog appears; save the file
- [X] Reload the page (F5)
- [X] Click **Load** — pick the saved file
- [X] **Expected**: "Alpha" appears in the top-left area; "Beta" appears in the bottom-right area — positions within 2% of where you placed them

### Scenario DnD-8: Drag does not disrupt other dots

- [X] Add three initiatives and drag each to a different quadrant
- [X] **Expected**: all three dots remain visible and in their last positions; dragging one dot does not move or hide the others

### Scenario DnD-9: Performance with 15 dots (SC-005)

- [X] Add 15 initiatives (names can be short: "I1" through "I15")
- [X] Drag several dots across the canvas
- [X] **Expected**: drag response remains immediate with no visible lag; canvas renders without delay

---

## Phase 6 — User Story 5: Edit an Existing Initiative

> **Goal**: An initiative's name, category, and details can be edited in the detail panel; changes show immediately on the canvas label and persist through save/reload.

### Scenario Edit-1: Edit button enters edit mode

- [X] Add an initiative named "Health"
- [X] Click the dot to open the detail panel (read-only view)
- [X] Click the **Edit** button
- [X] **Expected**: Name, Category, and Details fields become editable (`<input>` / `<textarea>`); current values are pre-filled

### Scenario Edit-2: Name change updates canvas label immediately

- [X] While in edit mode, clear the Name field and type "Fitness"
- [X] Confirm the edit (click Save / Confirm)
- [X] **Expected**: panel closes; the dot on the canvas now shows the label "Fitness" without a page reload

### Scenario Edit-3: Cancel reverts all fields

- [X] Click the "Fitness" dot to open the detail panel
- [X] Click **Edit**
- [X] Change the Name to "Running" and the Category to "Sport"
- [X] Click **Cancel**
- [X] Click the dot again
- [X] **Expected**: Name is still "Fitness"; Category is unchanged from before the edit; no data was written

### Scenario Edit-4: Name is required during editing

- [X] Open a dot's detail panel, click **Edit**
- [X] Clear the Name field completely
- [X] Attempt to confirm the edit
- [X] **Expected**: edit is rejected; an inline validation message appears; panel stays open in edit mode

### Scenario Edit-5: Edit persists after save / reload

- [X] Edit an initiative — change name to "Fitness", category to "Health", add details text
- [X] Confirm the edit
- [X] Click **Save**
- [X] Reload the page and **Load** the file
- [X] Click the dot
- [X] **Expected**: Name = "Fitness", Category = "Health", details text intact — all edited values restored

### Scenario Edit-6: Edit does not reset canvas position

- [X] Drag an initiative to the top-right quadrant (Q2 Growth)
- [X] Open the detail panel, click **Edit**, change the name
- [X] Confirm
- [X] **Expected**: dot remains in the top-right area; edit does not move the dot back to centre

---

## Phase 7 — User Story 3: Category Assignment

> **Goal**: Category is free text, stored per initiative, displayed correctly, and survives save/reload.

**Aborted**: I've decided to change categories and replace them with tags. Therefore, testing this user story is no longer needed. 

### Scenario Cat-1: Free text accepted, no validation

- [ ] Click **Add Initiative**
- [ ] In the Category field, type a string with spaces and mixed case: `Personal Development`
- [ ] Enter a name and submit
- [ ] **Expected**: form accepts the category without error or format warning

### Scenario Cat-2: Category displayed in detail panel

- [ ] Click the dot for the initiative with category "Personal Development"
- [ ] **Expected**: detail panel shows "Personal Development" in the category field

### Scenario Cat-3: No category → blank field (no default)

- [ ] Add an initiative with a name only — leave Category empty
- [ ] Click the dot
- [ ] **Expected**: category field in the detail panel is blank; no placeholder text or default value is shown

### Scenario Cat-4: No category cross-contamination

- [ ] Add "Initiative A" with category "Health"
- [ ] Add "Initiative B" with category "Career"
- [ ] Click "Initiative A" dot → confirm category shows "Health"
- [ ] Close panel; click "Initiative B" dot → confirm category shows "Career"
- [ ] **Expected**: each dot shows only its own category; no bleed between initiatives

### Scenario Cat-5: Category survives save / reload

- [ ] With "Health" and "Career" initiatives on the canvas, click **Save**
- [ ] Reload and **Load** the file
- [ ] Click each dot in turn
- [ ] **Expected**: "Initiative A" → "Health"; "Initiative B" → "Career" — exactly as entered, no alteration

---

## Phase 8 — Polish & Edge Cases

> **Goal**: Edge cases are handled gracefully; the tool is stable end-to-end.

### Scenario EC-1: Empty canvas (T027)

- [X] Open `index.html` with no file loaded
- [X] **Expected**: four quadrant labels are visible; axes are labelled; no JS errors in console; **Add Initiative** button is available and functional

### Scenario EC-2: Long name truncation on dot (T028)

- [X] Add an initiative with a name longer than 30 characters, e.g. `"This is a very long initiative name for testing"`
- [X] **Expected**: dot label is visually truncated with `…` (ellipsis); dot remains usable
- [X] Click the dot
- [X] **Expected**: full untruncated name is displayed in the detail panel

### Scenario EC-3: Two dots at same position (T029)

- [X] Add two initiatives without dragging either (both default to centre)
- [X] **Expected**: both dots are visible on the canvas — neither is completely hidden behind the other (a small offset or stacking is acceptable as long as both are clickable)
- [X] Click each dot in turn
- [X] **Expected**: each opens its own detail panel with the correct name

### Scenario EC-4: Round-trip preservation of non-TMM data (T013 regression)
**Deferred** Due to the change of categories to tags, this has been postponed. 
- [ ] Load [docs/json-samples/BSC_-_Supervision_Reporting-coer-2026-04-06.json](../../docs/json-samples/BSC_-_Supervision_Reporting-coer-2026-04-06.json) (or any file containing `coer` data)
- [ ] Confirm initiatives appear on canvas (TMM data defaults to centre if no `tmm` section)
- [ ] Drag one dot to a new position
- [ ] Click **Save** — save as a new file
- [ ] Open the saved file in a text editor
- [ ] **Expected**: `coer` section is intact and unchanged; no data stripped or defaulted by the TMM tool

### Scenario EC-5: Load file with unknown extra fields

- [X] Create or edit a JSON file to add an unknown field to a `tmm` section, e.g. `"customTag": "test"`
- [X] Load the file in TMM
- [X] Save back to a new file
- [X] Inspect the saved JSON
- [X] **Expected**: `"customTag": "test"` is still present in the saved output

### Scenario EC-6: Load file with no `tmm` section on any initiative

- [X] Load a file where no initiative has a `tmm` key (e.g. a COER-only file)
- [X] **Expected**: canvas loads without error; empty canvas or dots at centre (no crash, no user-facing error)

---

## Final Sign-Off Checklist (T030 — SC-001 to SC-005)

- [X] **SC-001**: Open fresh, add one initiative, see labelled dot on canvas in under 1 minute
- [X] **SC-002**: Drag dot to new quadrant → save → reload → dot appears in same quadrant; no data loss
- [X] **SC-003**: Save file with 5 initiatives → reload → all 5 restored with name, category, details, and position
- [X] **SC-004**: Click any dot → name, category, and details visible immediately without leaving canvas
- [X] **SC-005**: 15 initiatives on canvas → drag several → no lag, canvas renders without delay

---

## Bug Log

| # | Scenario | Description | Priority | Status |
|---|----------|-------------|----------|--------|
| B1 | Canvas layout | Quadrant colours/orientation wrong (logged as GitHub Issue) | P1 | Fix in progress (Copilot remote) |
| B2 | EC-X | Close (✕) button hidden behind toolbar | P3 | Open — Escape key works as workaround |

---

*Generated: 2026-04-07 | Spec: [spec.md](./spec.md) | Tasks: [tasks.md](./tasks.md)*
