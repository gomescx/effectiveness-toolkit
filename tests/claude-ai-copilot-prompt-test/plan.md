# Implementation Plan: TMM Tags, Size, and Report

**Branch**: `002-tmm-tags-size-report` | **Date**: 2026-04-09
**Spec**: [specs/002-tmm-tags-size-report/spec.md](./spec.md)
**Baseline**: [specs/001-tmm/plan.md](../001-tmm/plan.md)

## Summary

This plan extends the standalone TMM HTML tool (`public/tools/tmm/index.html`) with three capabilities: a structured multi-tag system replacing the `category` field (US-006), a t-shirt size field with five proportional CSS dot sizes (US-007), and a tabular report view with bulk tag assignment and markdown export (US-008). All changes are surgical modifications to the single `index.html` file — no new files, no build tooling, no external dependencies.

## Technical Context

**Language/Version**: HTML5 + vanilla JavaScript (ES6+), inline CSS — identical to baseline  
**Primary Dependencies**: None — native browser APIs only (File API for save/export, DOM for report view)  
**Storage**: Same `effectivenessToolkit` JSON file — schema change limited to the `tmm` section  
**Testing**: Manual only (Constitution §VII — Standalone HTML Tools Exception)  
**Target Platform**: Modern evergreen browsers (Chrome, Edge, Firefox, Safari)  
**Project Type**: Standalone HTML tool — single file, all JS/CSS inline  
**Constraints**: Offline-capable, no network requests, no third-party libraries, single HTML file  
**Scale/Scope**: Extends existing baseline; same single-user, single-session model  

## Constitution Check (Pre-Phase 0)

| # | Principle | Status | Rationale |
|---|-----------|--------|-----------|
| I | Offline-First Architecture | **PASS** | Single HTML file, no network requests, no backend — unchanged from baseline |
| II | Single-Session, Single-User | **PASS** | No multi-user or collaboration features |
| III | Privacy by Default | **PASS** | No telemetry, cookies, or tracking |
| IV | Respect Upstream (mind-elixir) | **N/A** | TMM is not a mind-elixir tool |
| V | Simplicity Over Completeness | **PASS** | CSS modifier classes for dot sizes; show/hide sections for report view; no new frameworks |
| VI | Shared Data Format & Tool Independence | **PASS** | Only `tmm` section modified; `category` removal is backward-compatible for other tools (they never read `tmm.category`); envelope structure unchanged |
| VII | Testing Standards | **PASS** | Standalone HTML Tools Exception; manual test plan defined |
| VIII | Commit Standards | **PASS** | Conventional Commits with `tmm` scope |
| IX | Environment Configuration | **N/A** | No build system |
| X | Console Output Standards | **N/A** | No build/batch processing |
| — | Form-Based Tools Guardrail | **PASS** | Single HTML file, zero build tooling |
| — | Suite Infrastructure (Deployment) | **PASS** | Same deployment path `public/tools/tmm/index.html` |

**Gate result: PASS** — no violations, no justifications required.

---

## Project Structure

### Documentation (this feature)

```text
specs/002-tmm-tags-size-report/
├── business-statement.md    # PO-approved scope statement
├── spec.md                  # This feature's spec (US-006, US-007, US-008)
├── plan.md                  # This file
├── tasks.md                 # Implementation task list
├── manual-test-plan.md      # UAT scenarios (Given/When/Then)
└── checklists/
    └── requirements.md      # Specification quality checklist
```

### Source Code (repository root — single file modified)

```text
public/tools/tmm/
└── index.html               # Extend in place — no new files
```

**Structure Decision**: All changes are confined to `public/tools/tmm/index.html`. The baseline tool already uses in-memory `initiatives[]`, `renderCanvas()`, and a File API save/load pattern. This feature adds CSS rules, modifies two form sections (add form + detail panel), adds a canvas filter control, adds a report view section (show/hide with CSS), and updates the serialiser/deserialiser. No new HTML files, no external assets.

---

## Phase 0: Research Findings

*(No external research required — this is an extension of a working tool. Key findings below.)*

- **`category` removal**: The `category` field exists at `initiative.tmm.category` and is referenced in the add form (line ~632), detail panel view (line ~566), detail panel edit (line ~589), the `openDetailPanel()` function (line ~878), `openEditMode()` (line ~925, ~931), `saveEdit()` (line ~957, ~963), and `cancelEdit()` (line ~987). All six references must be updated to `tags`.
- **Dot element structure**: Dots are `<div class="dot">` elements created in `renderCanvas()`. Adding a CSS modifier class (`.dot--xs` etc.) requires one line of change per dot creation — `dot.className = 'dot' + sizeClass`.
- **State synchronisation**: `initiatives[]` is the single source of truth. The report view must never copy data to a parallel array; it must read from and write to `initiatives[]` directly. On return from report view, `renderCanvas()` must be called.
- **Report view approach**: The simplest viable approach is a `<section id="report-view">` sibling to `<div id="canvas">`, toggled with CSS `display: none / block`. No routing, no URL manipulation.
- **Markdown export**: Build a GFM table string from `initiatives[]`, convert to a `Blob`, and download via the existing `<a download>` pattern already used in Save.
- **Quadrant derivation**: Canvas coordinates `x` (Urgency 0=low, 1=high) and `y` (Impact 0=low, 1=high). Q1 = `x >= 0.5 && y >= 0.5`, Q2 = `x < 0.5 && y >= 0.5`, Q3 = `x >= 0.5 && y < 0.5`, Q4 = `x < 0.5 && y < 0.5`.

---

## Phase 1: Design

### Data Model Change

The `tmm` section per initiative is updated as follows:

**Before**:
```json
{
  "tmm": {
    "category": "Health",
    "details": "...",
    "x": 0.7,
    "y": 0.8
  }
}
```

**After**:
```json
{
  "tmm": {
    "tags": ["#health", "#project-alpha"],
    "size": "L",
    "details": "...",
    "x": 0.7,
    "y": 0.8
  }
}
```

**Loading compatibility**: When loading a project file, if `tmm.tags` is absent, default to `[]`. If `tmm.size` is absent, default to `null`. `tmm.category` is ignored on load if present (no migration needed).

### CSS Dot Sizes

| Class | Diameter | Offset (centering) |
|-------|-----------|--------------------|
| `.dot--xs` | 18 px | −9 px |
| `.dot--s` | 24 px | −12 px |
| `.dot` (default / null) | 32 px | −16 px |
| `.dot--m` | 32 px | −16 px (same as default) |
| `.dot--l` | 44 px | −22 px |
| `.dot--xl` | 58 px | −29 px |

The `.dot` class defines the default (M-equivalent) size. `.dot--m` explicitly sets the same values so explicit M assignment is visually identical to no assignment. This ensures backward compatibility for existing initiatives.

### Tag Filter UI

A `<div id="filter-bar">` is placed below the canvas toolbar. It contains a `<select id="tag-filter-select">` populated on `renderCanvas()` with all unique tags across `initiatives[]`, plus a "— No filter —" option. A `<span id="active-filter-badge">` shows the active tag and a ×-button to clear. Filter state is `let activeTagFilter = null` — a module-level variable, not persisted.

### Report View Layout

```text
┌─────────────────────────────────────────────────────┐
│ [Canvas] [Report]   toolbar tabs                     │
├─────────────────────────────────────────────────────┤
│ ☐ | Quadrant | Name | Tags | Size | Details         │ ← header
│ ☐ | Q1       | ...  | ...  | L    | ...             │ ← rows
│ ☐ | Q2       | ...  | ...  |      | ...             │
├─────────────────────────────────────────────────────┤
│ [Tag: __________] [Apply to selected]               │
│                                [Export Markdown]    │
└─────────────────────────────────────────────────────┘
```

The report table is rebuilt from `initiatives[]` each time the report view is shown — it is never the source of truth.

---

## Implementation Sequence

### Step 1 — Data Model & Serialisation (blocking all other steps)
- Remove `category` from in-memory object template and form creation
- Add `tags: []` and `size: null` to the in-memory object template
- Update save serialiser to write `tmm.tags` and `tmm.size`; remove `tmm.category`
- Update load deserialiser to read `tmm.tags` (default `[]`) and `tmm.size` (default `null`)

### Step 2 — Form & Detail Panel (US-006, US-007)
- Replace Category input with Tags input in the add form; add tag format validator
- Add size `<select>` to the add form (optional — can default to null on add, edit to assign size)
- Update detail panel view to display tags and size
- Update detail panel edit mode to allow editing tags (with validation) and size (with selector)
- Update `openDetailPanel()`, `openEditMode()`, `saveEdit()`, `cancelEdit()` for new fields

### Step 3 — Canvas Tag Filter (US-006)
- Add filter bar HTML below toolbar
- Populate filter `<select>` in `renderCanvas()` with unique tags
- Store active filter in `activeTagFilter` variable
- In `renderCanvas()`, skip dot render for non-matching initiatives when filter is active
- Show/hide filter indicator badge; wire clear button to reset `activeTagFilter` and call `renderCanvas()`

### Step 4 — CSS Dot Sizes (US-007)
- Add `.dot--xs` through `.dot--xl` CSS rules
- In `renderCanvas()` dot creation, compute size class from `initiative.tmm.size` and set `dot.className`

### Step 5 — Report View (US-008)
- Add `<section id="report-view" style="display:none">` with table, checkboxes, bulk tag panel, and export button
- Add "Report" tab button next to "Add"/"Save"/"Load" toolbar — wires to `showReportView()`
- `showReportView()`: build table rows from `initiatives[]`, show report section, hide canvas section
- `showCanvasView()`: hide report section, show canvas section, call `renderCanvas()`
- Wire "Apply to selected" button: read checked IDs, validate tag, mutate `initiatives[]`, rebuild table
- Wire "Export Markdown" button: build GFM table string from `initiatives[]`, trigger `.md` download

---

## Invariants & Safety Rules

1. **`initiatives[]` is the single source of truth** — the report table is built FROM it; changes write BACK to it. Never read from table row text content.
2. **`renderCanvas()` is the single render path** — called on: add, edit confirm, load, filter change, and return from report view. Never manipulate dot DOM nodes directly outside `renderCanvas()`.
3. **`category` field is gone** — no fallback, no migration, no read of `tmm.category` on load.
4. **Filter state is never saved** — `activeTagFilter` is in-memory only; the save serialiser must not touch it.
5. **Drag behaviour is unchanged** — size modifier classes are CSS-only; they must not change the pointer event binding logic in T016–T020.

---

## Constitution Check (Post-Phase 1 Design)

| # | Principle | Status | Post-Design Notes |
|---|-----------|--------|-------------------|
| I | Offline-First | **PASS** | No network dependencies in design |
| II | Single-Session | **PASS** | No multi-user features in design |
| III | Privacy | **PASS** | No telemetry, tracking, or external calls |
| IV | Upstream | **N/A** | — |
| V | Simplicity | **PASS** | CSS modifier classes + show/hide sections = simplest viable approach; no third-party libraries |
| VI | Shared Format | **PASS** | Only `tmm` section changed; other tools unaffected; envelope stable |
| VII | Testing | **PASS** | Manual test plan defined in `manual-test-plan.md` |
| VIII | Commits | **PASS** | Conventional Commits with `tmm` scope |
| — | Form-Based Tools | **PASS** | Single file, no build tooling |
| — | Suite Infrastructure | **PASS** | Same deployment path |

**Post-design gate result: PASS** — no violations, no justifications required.

## Complexity Tracking

> No constitution violations — this section is intentionally empty.
