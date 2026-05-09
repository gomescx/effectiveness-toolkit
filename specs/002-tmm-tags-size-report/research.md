# Research: TMM Tags, Size & Report (002-tmm-tags-size-report)

**Feature**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)
**Date**: 2026-05-08

---

## Research Task 1: Inbox panel — drag-to-canvas from a side panel

### Question

The UX spec defines a two-zone drag interaction: items originate in the Inbox side panel and are dropped onto a canvas that is a separate positioned container. Can the existing Pointer Events API approach (used for dot repositioning) cover this cross-container drag? What data-transfer mechanism is needed?

### Findings

The existing TMM tool uses `setPointerCapture` on the *dot itself*. For Inbox-to-canvas drag, the dragged element originates outside the canvas. Two viable paths exist:

| Approach | How it works | Pros | Cons |
|---|---|---|---|
| **Pointer Events + manual hit-test** | `pointerdown` on inbox row; `pointermove` updates a floating ghost element; `pointerup` hit-tests the canvas rect to check for valid drop | No new API; consistent with existing drag code | Ghost element must be rendered manually; hit-test on `pointerup` |
| **HTML5 DnD API** | `draggable="true"` on inbox rows; `dragover`/`drop` on canvas | Declarative; built-in ghost image | No real-time visual tracking; poor on touch; rejected in research.md for dot drag |

### Decision

**Use Pointer Events + manual canvas hit-test** for Inbox-to-canvas drag. This is consistent with the existing drag pattern (research.md, Task 2). A semi-transparent ghost `<div>` is created on `pointerdown` and removed on `pointerup`. On `pointerup`, check whether the pointer coordinates fall within the canvas bounding rect; if yes, compute normalised (x, y) and place the initiative.

### Rationale

- Consistent with the existing tool's drag implementation — no second drag API introduced
- Works on touch and mouse with the same code path
- `pointer-events: none` on the ghost prevents it from interfering with the canvas `pointerup` event
- Constitution Principle V (simplicity): one drag API, not two

### Alternatives Considered

- **HTML5 DnD**: Rejected — ghost image lags, no touch support, same reasons as research.md Task 2
- **Click-to-select + click-to-place (two-click)**: Would satisfy FR-UX-020 without drag, but degrades the UX spec's explicit wireframe showing drag handles (⠿). Deferred as fallback if drag proves unreliable

---

## Research Task 2: Detail panel — change-tracking for conditional close

### Question

FR-UX-009 and FR-UX-011 require the [Close] button to be disabled once any change is made, with [Cancel] as the only revert path. How should change-tracking be implemented in a standalone HTML file (no framework reactivity)?

### Findings

Two strategies were evaluated:

| Strategy | How it works | Pros | Cons |
|---|---|---|---|
| **Snapshot comparison** | Save field values on panel open; on each `input` event compare current values to snapshot | Simple; no extra state per field | Requires reading all fields on every keystroke |
| **Dirty flag** | Boolean `isDirty` set on first `input`/`change` event | O(1) check; simpler logic | Does not detect "return to original value" as clean (minor UX edge case, acceptable per FR-UX-009) |

### Decision

**Dirty flag** (`isDirty` boolean). Set to `true` on the first `input` or `change` event on any panel field. Reset to `false` on panel open (via snapshot reset). Once `isDirty = true`, the [Close] button gains a `disabled` visual style (`opacity: 0.4; cursor: not-allowed; pointer-events: none`); "click same dot" is blocked.

### Rationale

- Simpler logic — one boolean governs all close-path gating
- The edge case (user types then deletes back to original — still `isDirty`) is acceptable: Constitution Principle V says avoid complexity not justified by concrete user value
- FR-UX-009 says "visually disabled once any change is made" — dirty flag is sufficient

---

## Research Task 3: T-shirt size → dot diameter mapping

### Question

FR-UX-018 requires five visually distinct dot diameters for XS–XL that rescale with the canvas. What proportional scale produces clear visual distinction and remains consistent at any window size?

### Findings

Fixed-pixel values (original decision: XS=16px…XL=58px) were rejected during testing because dots did not rescale when the browser window was resized, and the size differences were not visually distinct enough at larger canvas sizes.

Proportional sizing anchored to quadrant width resolves both issues:

| Size | Formula | At 600px canvas (300px quadrant) |
|---|---|---|
| XS | quadrantWidth / 16 | ≈ 19px |
| S  | canvasWidth × 11/128 | ≈ 52px |
| M  | canvasWidth × 9/64  | ≈ 84px |
| L  | canvasWidth × 25/128 | ≈ 117px |
| XL | canvasWidth / 4     | ≈ 150px |

Null (unset) renders as a hollow outlined circle at the M-equivalent diameter (same proportional formula, `background: transparent; border: 2px solid var(--color-primary)`).

### Decision

Diameters are calculated at render time from live canvas width. `applyDotSize(dotEl, size, canvasWidth)` replaces the static `SIZE_DIAMETER` map. A `ResizeObserver` on `#canvas` triggers `renderCanvas()` on resize.

Proportions: XS = 1/16 of a quadrant, XL = 1/2 of a quadrant, S/M/L linearly interpolated. (Updated 2026-05-09 after testing revealed fixed-pixel approach failed on resize.)

### Rationale

- Proportional to canvas → consistent visual weight at any window size
- XL at 1/2 quadrant is dramatic and clearly the largest; XS at 1/16 quadrant is clearly smallest
- Linear interpolation between XS and XL gives four visually distinct steps
- ResizeObserver is broadly supported in modern evergreen browsers (target platform)

---

## Research Task 4: Report view — spreadsheet-style keyboard navigation without a grid framework

### Question

FR-UX-014 requires arrow-key navigation, Enter to edit, Esc to revert. Can this be done with plain HTML + JavaScript in a `<table>` without a grid library?

### Findings

Standard `<table>` elements support `tabindex` on `<td>` cells. A focused cell receives keyboard events. Navigation logic:

```
ArrowRight / Tab → nextElementSibling (skip Q column)
ArrowLeft / Shift+Tab → previousElementSibling (skip Q column)
ArrowDown → same column index, next row
ArrowUp → same column index, previous row
Enter → activate edit mode on current cell
Esc → cancel edit, restore previous value, exit edit mode
```

Edit mode: replace cell text content with an `<input>` or `<select>` (for Size), pre-populated with current value. On commit (Tab/Enter), update `initiatives[]` and re-render cell content. On Esc, discard and re-render original.

### Decision

**Plain `<table>` with `tabindex="0"` on editable cells**. Navigation and edit mode managed by a single `keydown` event listener on the table. No grid library required.

### Rationale

- Constitution Principle V: simplest approach that satisfies the requirement
- No third-party dependency (no ag-Grid, no Handsontable)
- Accessible: native `<table>` semantics preserved; `aria-selected` set on focused cell
- Enter activates edit mode: cell content replaced by `<input>` (Name, Tags, Details) or `<select>` (Size); Esc reverts; Tab/Enter commits and advances
- Tags validation uses the shared `validateTags()` helper — same rule as detail panel (FR-US8.04c)
- Details cell uses `<textarea>`; Enter = new line; Tab = commit and advance

> **Updated 2026-05-09**: spec.md FR-US8.04 was revised to specify inline cell editing (FR-US8.04a–e). The previous scope clarification noting read-only report is superseded. This research task now documents the confirmed spreadsheet-editor implementation approach.

---

## Research Task 5: Unsaved-changes banner — detecting in-memory vs. saved state

### Question

FR-UX-013 requires a red banner whenever in-memory state differs from the last saved JSON. How should dirty state be tracked without a framework?

### Findings

Two approaches:

| Approach | How it works |
|---|---|
| **Boolean `isDirty` flag** | Set to `true` on any initiative create/edit/delete/drag; reset to `false` on successful save or successful load |
| **Deep-compare serialised state** | Stringify in-memory array on each change and compare to last-saved string |

### Decision

**Boolean `isDirty` flag** at the application level. Set on: add initiative, edit initiative, delete initiative, drag initiative. Reset on: save success, load success (loaded state = saved state).

### Rationale

- O(1) check; no serialisation cost on every change
- Covers all mutation paths explicitly
- Sufficient for FR-UX-013 — "differs from last saved" is accurately tracked by the flag

---

## Research Task 6: Backward compatibility — loading files with `category` field

### Question

FR-US6.08 requires `category` to be silently ignored on load. FR-US6.03 requires `tags` to default to `[]` when absent. How does this interact with the existing `loadFromFile()` code?

### Decision

In `loadFromFile()`, the `tmm` section mapping is updated:
- `tags`: read `tmm.tags` if present and an array; otherwise default to `[]`
- `size`: read `tmm.size` if present and a valid enum value; otherwise default to `null`
- `category`: **not read** — field is silently ignored (not mapped to in-memory model)

Existing initiatives in loaded files that have `category` set will have it preserved in `loadedProjectFile` (unchanged) but NOT in the in-memory `initiatives[]` array. On save, `category` is NOT written back to the `tmm` section (it is removed from the on-disk schema). This is a one-way migration on save, consistent with FR-US6.08 ("no migration attempted, no error displayed").

### Rationale

- Simplest approach: no migration code, no error handling for the old field
- Consistent with "silently ignored" requirement
- `additionalProperties: true` in the schema means the field is not an error, just unused
