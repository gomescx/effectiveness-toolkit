# Session Log — TMM Tags, Size & Report — 2026-05-09

**Project**: effectiveness-toolkit
**Branch**: `002-tmm-tags-size-report` → merged to `main`
**Skills invoked (in order)**: /dmresume, /dmux, /dmwishlist
*(Copilot invoked: /speckit.plan, /speckit.tasks, /speckit.implement × 11 phases)*

---

## Invocations

### /dmresume — 1
**Trigger**: `/dmresume`
**Produced**: Re-orientation briefing on branch state, pending work (ux-spec missing, plan not yet run), and handoff context
**User corrections**: None

---

### /dmux — 2
**Trigger**: `/dmux retroactive. produce the ux specs that speckit.specify didn't define`
**Produced**:
- Retroactive UX audit identifying 4 problems in the already-implemented 001-tmm tool (UC1–UC4)
- Full Mermaid user journey flowchart
- Screen wireframes: Canvas view, Inbox panel, Detail panel, Report view
- FR-UX-001 to FR-UX-021 as a requirements table
- `specs/002-tmm-tags-size-report/ux-spec.md` written to disk
- `/speckit.plan` handoff prompt

**User corrections**:
- UC3 misdiagnosis corrected twice: Edit→Confirm close behaviour stated wrong in both directions before the correct version was settled ("no exit from panel except invisible ✕")
- "Load a map" → "Load a matrix" (TMM = Time Management Matrix)
- Report view redesigned mid-session: user rejected read-only table ("this use case is weak"), then revised to spreadsheet editor with inline cell editing for Name/Tags/Size/Details
- `[Cancel]` behaviour: initially designed as "reverts, stays open" — corrected to "reverts and closes"
- FR-UX-020 missing: no FR specified what happens when dragging from Inbox to canvas — caught by user, added
- Details field removed from Inbox entry form — Inbox is for fast brain dump, not full editing

---

### /dmwishlist — 3
**Trigger**: "lets document the coer in dmwishlist — not sure how you're dealing with that but I'm keeping a backlog in git issues"
**Produced**: GitHub issue draft for COER link idea (initiative in TMM linking to its COER record); label `backlog`
**User corrections**: None

---

## Transitions

### After /dmux → speckit.plan (Copilot)
**What bridged them**: Handoff prompt produced at end of /dmux; user compacted context and handed prompt to Copilot
**What was skipped**: spec.md update — /dmux wrote ux-spec.md with FR-UX-014 (spreadsheet editor) which directly contradicts spec.md FR-US8.04 (read-only table), but spec.md was not updated in the same session
**Effect**: Copilot correctly treated spec.md as authoritative and built plan.md B4 as a read-only table. Caught in DM plan review. Required manual spec.md update (FR-US8.04 → FR-US8.04a–e) and a second Copilot pass on B4 before tasks could be generated.

### After speckit.plan → speckit.tasks
**What bridged them**: DM plan review, spec.md fix, Copilot update of plan.md B4
**What was skipped**: Nothing — correct sequence
**Effect**: None

### After speckit.tasks → speckit.implement
**What bridged them**: Phase-by-phase implementation with DM review and independent test between phases
**What was skipped**: Integration test scenarios — no cross-feature scenarios were documented before implementation began
**Effect**: `loadFromFile()` + Report view interaction bug found by chance during Phase 10 user testing (not caught by any independent test). Required retrospective spec.md FR-000.08, tasks.md T051 update, and two additional edge cases.

---

## Observed Failures

### 1 — dmux does not update spec.md when UX decisions override existing FRs
**Type**: D governance gap
**Skill file**: `dmux.md`
**Expected**: When /dmux produces a FR-UX that explicitly overrides an existing spec.md FR, the skill should flag the conflict and require spec.md to be updated (or flag it for DM action) before producing the handoff prompt
**Actual**: ux-spec.md was written with FR-UX-014 overriding FR-US8.04 with no flag and no spec.md update. Copilot followed spec.md correctly and produced the wrong B4 plan. The conflict was only caught at plan review.
**Fix direction**: /dmux handoff step should include a check: "Do any FR-UX items contradict or supersede an existing spec.md FR? If yes, update spec.md before producing the handoff prompt."

---

### 2 — DM touched implementation code (boundary violation)
**Type**: A boundary violation
**Skill file**: `solo-dm-speckit.md` (Role Boundaries)
**Expected**: Code changes go through Copilot only
**Actual**: DM directly edited `index.html` to add the missing `applyDotSize()` call in `renderCanvas()`. User let it pass but explicitly flagged: "you're not supposed to touch code — that's Copilot's job."
**Fix direction**: Before editing any file in `public/`, pause and route to Copilot with a targeted fix description instead.

---

### 3 — Copilot handoff prompt included commit message guidance
**Type**: B output quality
**Skill file**: N/A (ad hoc prompt authoring)
**Expected**: Commit messages are the DM's role. Copilot prompts should not reference them.
**Actual**: Phase 9 handoff prompt included "Reference issue #20 in the commit message." User corrected: "Copilot doesn't create commit messages, this is your role."
**Fix direction**: Never include commit message instructions in Copilot prompts.

---

### 4 — Wrong task range stated in Phase 9 prompt
**Type**: B output quality
**Skill file**: N/A (ad hoc prompt authoring)
**Expected**: Phase 9 is T039 only
**Actual**: Prompt said "tasks T039–T042." User corrected: "Phase 9 (B3) is only task T039."
**Fix direction**: Always verify task range against tasks.md before writing Copilot prompts.

---

### 5 — No integration scenarios documented before implementation
**Type**: D governance gap
**Skill file**: `dmux.md` / spec.md template
**Expected**: When two user stories share mutable state (e.g. `loadFromFile` + Report view, save + dirty flag), a cross-feature integration scenario should be documented alongside the per-feature independent tests
**Actual**: No integration scenarios existed. The `loadFromFile()` + Report view bug was discovered by accident. Required retrospective FR-000.08, two new edge cases in spec.md, and a T051 rewrite.
**Fix direction**: /dmux or /speckit.specify should prompt: "For each pair of user stories that share state, document at least one cross-feature scenario." These go in spec.md Edge Cases, not per-story acceptance criteria.

---

## Notes

- Session spanned two context windows (compaction mid-way). Pre-compaction summary was complete; no context loss observed on resume.
- /dmux back-and-forth validation pattern worked well — iterating wireframes in conversation caught 6 corrections before anything was written to disk.
- Phase-by-phase implementation review (stop after each phase, run independent test, commit) caught 3 bugs before they accumulated: missing `applyDotSize()` call, blur-handler scoped only to size field, and load-not-refreshing-table.
- Proportional dot sizing was not in the original spec — discovered during Phase 8 testing. Required retrospective updates to spec.md (FR-US7.03/04), ux-spec.md (FR-UX-018/021), research.md, and quickstart.md. Pattern to avoid: spec artifacts should define proportions, not pixel values, for any measurement that depends on viewport size.
- GitHub issues created for all bugs and enhancement ideas during implementation (#19–#25). Cross-referencing issues in commit messages worked cleanly.
- "No PRs" workflow (direct branch merge) worked without friction.
- 17 commits, 32 files, 7830 insertions merged to main. All independent tests passed.
