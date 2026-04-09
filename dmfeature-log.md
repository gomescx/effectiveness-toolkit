# /dmfeature Conversation Log — TMM (Time Management Matrix)

**Project**: Effectiveness Toolkit
**Feature**: `001-tmm` — Time Management Matrix
**Branch**: `001-tmm`
**Dates**: 2026-04-06 to 2026-04-09
**Roles**: Claudio (PO / UAT) · Claude Code (BA / Architect / QA) · Copilot (Developer)

---

## Purpose of This Log

This log captures the full `/dmfeature` workflow for the TMM tool — from initial concept through spec, plan, tasks, implementation, and UAT — including every decision, correction, bug, and scope change made along the way. Intended as a reference for improving the solo-dm `/dmfeature` skill instructions.

---

## 1. Feature Request (Trigger)

**User prompt** (paraphrased):

> Design a standalone HTML tool for the Effectiveness Toolkit. It should display a four-quadrant Impact × Urgency canvas (Covey-style). Users place "big rocks" (initiatives) as draggable dots. Quadrants: Q1 Firefighting, Q2 Growth, Q3 Distraction, Q4 Waste. Initiatives have a name, a user-defined category (free text life area / role), and a details field. Drag-and-drop repositioning. Save/load to JSON using the shared `effectivenessToolkit` envelope. Branch numbered `001-tmm`.

**What Claude Code did**:
- Ran the PO conversation to confirm axes, categories, success criteria, and tech constraints
- Confirmed category is free text (not a predefined list); markdown rendering deferred as non-mandatory POC simplicity wins
- Confirmed no additional tech constraints beyond Constitution v1.3.0 (standalone HTML, no build tooling)
- Saved `business-statement.md` and `copilot-handoff.md` at project root

---

## 2. Spec Review (`speckit.specify` output)

Copilot generated `specs/001-tmm/spec.md`. Claude Code reviewed before implementation.

### Catch 1 — Missing US5 (Edit Initiative)

**Problem**: Spec included an assumption "no editing after creation". Claudio confirmed editing IS required.

**Action**: Claude Code added US5 (Edit an Existing Initiative) with 5 Acceptance Scenarios and FR-US5.01–05. Removed "no editing" assumption. Removed editing from Deferred list.

**Lesson for /dmfeature**: Explicitly ask during PO conversation — "Can users edit an initiative after creation?" before passing to Copilot. Do not let Copilot infer this.

### Catch 2 — FR-US4.03 field placement inconsistency

**Problem**: Research correctly placed `id` and `name` at initiative level (matching COER pattern). But FR-US4.03 still described them as being inside the `tmm` section.

**Before**:
```
tmm section containing: id, name, category, details, x, y
```

**After**:
```
id and name at initiative level; tmm section contains: category, details, x, y
```

**Action**: FR-US4.03 rewritten in spec.md.

**Lesson for /dmfeature**: When the data model references an existing tool's pattern (COER), verify that the spec's FRs match the research conclusion — not just that research and FRs exist independently.

---

## 3. Plan Review (`speckit.plan` output)

Copilot generated `specs/001-tmm/plan.md`. Claude Code reviewed.

**Result**: Clean. Constitution checks pass pre/post Phase 1. Key decisions documented:
- Drag API: Pointer Events (over HTML5 DnD and Mouse Events)
- Canvas rendering: CSS divs + absolute positioning (over `<canvas>` and SVG)
- Data placement: `tmm` section per initiative following COER pattern
- Label truncation: CSS `text-overflow: ellipsis`

**Decision on `/speckit.analyse`**: Skipped. Manual review had already caught all meaningful issues. Running analyse after a thorough manual review would be redundant for a solo-file project this size.

**Lesson for /dmfeature**: For single-file standalone HTML tools, manual spec review is sufficient before plan. Reserve `/speckit.analyse` for multi-file TypeScript projects.

---

## 4. Tasks Review (`speckit.tasks` output)

Copilot generated `specs/001-tmm/tasks.md`. Claude Code reviewed.

### Catch 3 — T008 data structure wrong

**Problem**: T008 (form submission handler) created the initiative object with `category` and `details` at the top level, contradicting the data model. T012 (Save) expected them inside the `tmm` section.

**Before** (T008):
```javascript
{ id, name, category, details, tmm: { x: 0.5, y: 0.5 } }
```

**After** (T008 corrected):
```javascript
{ id, name, coer: null, sob: null, memoryMap: null, impactMap: null,
  tmm: { category: "", details: "", x: 0.5, y: 0.5 } }
```

**Action**: T008 corrected in tasks.md before Copilot began implementation.

**Lesson for /dmfeature**: When reviewing tasks, trace the data object created in the "Add" task all the way through to the "Save" task. Inconsistencies in field placement are the most common task-level error for tools with nested JSON structures.

---

## 5. Implementation — Phases 1–4

Copilot implemented T001–T015 (setup, foundational, US1, US4) in a single session and produced `public/tools/tmm/index.html` (999 lines, all JS/CSS inline).

### Implementation Fix — `loadFromFile` round-trip violation (P0 pre-UAT fix)

**Problem found during code review** (before UAT):

`loadFromFile` filtered initiatives with `item && item.id && item.name && item.tmm`. This silently dropped any initiative without a `tmm` section (e.g. COER-only initiatives). If a user loaded a shared project file created by another tool, those initiatives would be permanently lost on re-save — violating Constitution Principle VI.

**Fix applied** (Claude Code edited the file directly):

```javascript
// BEFORE — drops non-tmm initiatives:
initiatives = rawInitiatives
  .filter(function(item) {
    return item && item.id && item.name && item.tmm; // ← wrong
  })

// AFTER — keeps all valid initiatives, defaults missing tmm to centre:
initiatives = rawInitiatives
  .filter(function(item) {
    return item && item.id && item.name; // ← correct
  })
  .map(function(item) {
    var tmm = item.tmm || {}; // ← safe fallback
    return {
      id: item.id, name: item.name,
      coer: item.coer !== undefined ? item.coer : null,
      sob: item.sob !== undefined ? item.sob : null,
      memoryMap: item.memoryMap !== undefined ? item.memoryMap : null,
      impactMap: item.impactMap !== undefined ? item.impactMap : null,
      tmm: {
        category: tmm.category !== undefined ? tmm.category : '',
        details:  tmm.details  !== undefined ? tmm.details  : '',
        x: typeof tmm.x === 'number' ? Math.max(0, Math.min(1, tmm.x)) : 0.5,
        y: typeof tmm.y === 'number' ? Math.max(0, Math.min(1, tmm.y)) : 0.5
      }
    };
  });
```

**Lesson for /dmfeature**: For any tool that shares a JSON file with other tools, code-review the `loadFromFile` function before UAT specifically for the filter condition. Any `&& item.[tool-section]` in the filter is a data-loss bug.

---

## 6. Automated Tests — Decision

Tasks.md header: "Manual test scenarios only (per Constitution §VII — Standalone HTML Tools Exception). No automated test tasks included."

Tasks T020, T024–T026, T030 are labelled *verify* — these are UAT owner steps, not Copilot test harness tasks.

**Decision**: No automated tests for the TMM tool. Phases 5–8 follow the same rule.

**Lesson for /dmfeature**: Establish the test strategy explicitly in tasks.md before implementation begins. For standalone HTML tools, state "manual only" in the header so Copilot does not add test scaffolding.

---

## 7. UAT — Phases 1–4

Claudio performed UAT on the first implementation.

### Bug B1 — Quadrant layout wrong (P1)

**Scenario**: Phases 1–4 canvas layout check.

**Expected** (from reference image provided by Claudio):
```
More Impact ▲
            │  Q1 Firefighting  │  Q2 Growth
            │  (top-left, red)  │  (top-right, green)
            ├───────────────────┼──────────────────
            │  Q3 Distraction   │  Q4 Waste
            │  (btm-left, yel)  │  (btm-right, grey)
            └─────────────────────────────────────▶
         More Urgent                        Less Urgent
```

**Actual**: Quadrant colours reversed, axis direction wrong (more urgent was on right, should be left).

**Priority**: P1 — blocks remaining UAT; wrong canvas layout invalidates all drag positioning tests.

**Resolution**: Logged for Copilot (remote). Fix deployed before Phase 5 UAT resumed.

**Data model note**: Coordinate system updated — `x=0` now means high urgency (left). The `data-model.md` coordinate diagram needed updating to match.

### Bug B2 — Close button hidden behind toolbar (P3)

**Scenario**: Scenario 4 step 4 — close the detail panel via the ✕ button.

**Expected**: ✕ button visible in detail panel.

**Actual**: Button hidden behind toolbar (`z-index` of panel = 50, toolbar = 100).

**Workaround**: Escape key works.

**Priority**: P3 — does not block UAT; workaround available.

**Resolution**: Logged for Copilot. Escape key workaround documented.

---

## 8. UAT — Phases 5–8

Claudio executed [specs/001-tmm/manual-test-plan-p2-p3.md](specs/001-tmm/manual-test-plan-p2-p3.md).

### Results Summary

| Phase | Scenarios | Result |
|-------|-----------|--------|
| Pre-flight | 3 checks | ✅ All passed |
| Phase 5 — Drag-and-Drop (DnD-1 to DnD-9) | 9 scenarios | ✅ All passed |
| Phase 6 — Edit (Edit-1 to Edit-6) | 6 scenarios | ✅ All passed |
| Phase 7 — Category (Cat-1 to Cat-5) | 5 scenarios | ⛔ Aborted — scope change (see below) |
| Phase 8 — Edge Cases (EC-1 to EC-6) | 6 scenarios | ✅ EC-1,2,3,5,6 passed; EC-4 deferred |
| Final Sign-Off (SC-001 to SC-005) | 5 criteria | ✅ All passed |

### Scope Change — Category → Tags (Phase 7 aborted)

During UAT, Claudio decided to replace the free-text Category field with a Tags model. Phase 7 (US3 Category assignment) was aborted. US3 is now superseded by a future Tags feature.

**Impact on current branch**: Category field remains in the code but is considered interim. Tags feature to be designed separately.

**Lesson for /dmfeature**: Scope changes during UAT are normal. The `/dmfeature` workflow should not treat an aborted user story as a failure — capture the decision, mark the test scenarios as superseded, and carry the intent forward into the next feature.

### EC-4 Deferred

Round-trip preservation test (EC-4) was deferred due to the category→tags scope change. The underlying fix (loadFromFile) was already applied before UAT, so the core risk is mitigated.

---

## 9. Delivery State

| Milestone | Status |
|-----------|--------|
| Phase 1 — Setup | ✅ Complete |
| Phase 2 — Foundational | ✅ Complete |
| Phase 3 — US1 (Canvas + Add + View) | ✅ Complete, UAT passed |
| Phase 4 — US4 (Save / Load) | ✅ Complete, UAT passed |
| Phase 5 — US2 (Drag-and-Drop) | ✅ Complete, UAT passed |
| Phase 6 — US5 (Edit) | ✅ Complete, UAT passed |
| Phase 7 — US3 (Category) | ⛔ Aborted — superseded by Tags |
| Phase 8 — Polish (EC-1,2,3,5,6) | ✅ Complete, UAT passed |
| EC-4 Round-trip regression | ⏸ Deferred |
| Bug B1 (Quadrant layout) | ✅ Fixed before Phase 5 UAT |
| Bug B2 (Close button z-index) | 🔲 Open — P3, Escape workaround |

---

## 10. Artefacts Produced

| File | Description |
|------|-------------|
| `business-statement.md` | PO sign-off — US-001/002/003, scope IN/OUT, SC-001–SC-004 |
| `copilot-handoff.md` | Copilot prompt for `/speckit.specify` |
| `specs/001-tmm/spec.md` | Feature specification (5 user stories, 30+ FRs, edge cases) |
| `specs/001-tmm/plan.md` | Implementation plan (phases, tech decisions, constitution checks) |
| `specs/001-tmm/tasks.md` | Task list T001–T030 with phase/parallel markers |
| `specs/001-tmm/research.md` | 6 resolved unknowns (drag API, coordinate system, COER pattern) |
| `specs/001-tmm/data-model.md` | Entity definitions, coordinate diagram, in-memory state |
| `specs/001-tmm/contracts/project-file-schema.md` | 5 JSON contracts, round-trip preservation rules |
| `specs/001-tmm/quickstart.md` | Architecture overview, key decisions, coordinate system |
| `specs/001-tmm/manual-test-plan-p2-p3.md` | P2/P3 UAT plan — 30 checkbox scenarios |
| `public/tools/tmm/index.html` | Deliverable — standalone HTML tool (all JS/CSS inline) |
| `public/index.html` | Updated — TMM link added to suite launcher |

---

## 11. Key Lessons for /dmfeature Skill Improvement

1. **Ask "can they edit?" explicitly** during PO conversation. Do not let Copilot infer this from silence.

2. **Trace data structure across task boundaries** — especially Add → Save. Mismatches in field placement (top-level vs nested) are the most common task error for JSON-backed tools.

3. **Check FR field placement against research** — if research settles `id`/`name` at initiative level, verify every FR that mentions those fields uses the correct location.

4. **Review `loadFromFile` filter before UAT** — any `&& item.[section]` filter is a potential data-loss bug for tools sharing a multi-section JSON file.

5. **State test strategy in tasks.md header** before Copilot begins implementation. For standalone HTML tools: "manual only". Prevents Copilot adding unnecessary test scaffolding.

6. **Skip `/speckit.analyse` for single-file HTML tools** when a thorough manual review has already been completed. Reserve it for multi-file TypeScript projects.

7. **Scope changes during UAT are valid and expected** — capture the decision, mark affected scenarios as superseded, carry intent forward to next feature rather than treating it as failure.

8. **P1 bugs block all downstream UAT** — when a layout or coordinate system bug is P1, do not continue testing until it is fixed. Later scenarios built on correct positioning will give false results against a wrong canvas.

---

*Log exported: 2026-04-09 | Branch: `001-tmm` | Tool: TMM*
