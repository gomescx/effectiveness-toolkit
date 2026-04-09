# Business Statement — TMM Tags, Size, and Report

## What this feature does
Extends the Time Management Matrix tool with two new classification dimensions
per initiative — a multi-tag system (replacing the current single category
field) and a t-shirt size — and adds a tabular report view with bulk tag
assignment and markdown export.

## Why it matters
The current single-text category field cannot represent the multi-dimensional
nature of real initiative portfolios. A coaching client managing initiatives
across health, finance, career, and multiple projects needs to slice the canvas
by any combination of those dimensions at once. T-shirt sizing gives an
immediate sense of scale — distinguishing a programme from a task — without
requiring a full project management tool. The report view bridges brainstorming
on the canvas to operational planning, giving the user a structured summary
they can act on and share.

## User Stories
- US-006: As a PEP coach or executive, I want to tag each initiative with one
  or more structured hashtag labels (e.g. `#health`, `#project-alpha`), so
  that I can classify initiatives across multiple dimensions and filter the
  canvas to show only the initiatives relevant to a given context.
- US-007: As a PEP coach or executive, I want to assign a t-shirt size to each
  initiative (XS, S, M, L, XL) and see it visually represented as a
  proportional circle on the canvas, so that I can compare initiative scale at
  a glance without opening each detail panel.
- US-008: As a PEP coach or executive, I want to generate a tabular report of
  all canvas initiatives — with bulk tag assignment and markdown export — so
  that I can transition from brainstorming to operational planning and capture
  the output in a portable file.

## Scope

### In scope
- Replace `category` field with `tags` array (hashtag-formatted tokens, no
  spaces within a token, e.g. `#health #project-alpha`; multiple tags
  space-separated in the input)
- Tag filter on canvas: selecting a tag hides all non-matching dots entirely;
  a clear active-filter indicator is shown on the canvas at all times when a
  filter is active
- T-shirt size field per initiative: XS, S, M, L, XL (single value or null)
- Five visually distinct dot sizes on canvas mapped to XS → XL; null/unset
  renders at the current default size (equivalent to M) so existing
  initiatives are unaffected
- Tabular report view: all initiatives listed with columns for quadrant, name,
  tags, size, and details
- Bulk tag assignment in the report view: checkboxes per row, tag input,
  "Apply to selected" button
- Markdown export of the report (browser download as `.md` file)
- State synchronisation: bulk tag changes in the report view are immediately
  reflected on the canvas when returning to it — no reload required

### Out of scope (this feature)
- Predefined tag taxonomy, autocomplete, or tag management screen — deferred
- Visual grouping or colour-coding on the canvas by tag — deferred
- Canvas multi-select bulk tagging (shift-click dots + sidebar) — deferred to
  wishlist
- T-shirt size mapping to Agile levels (epic/feature/story/task) — deferred
- Report feeding into action plan or Memory Map integration — deferred
- Browser print or PDF export of the report — deferred (markdown export covers
  the capture need for MVP)
- Data migration from `category` to `tags` — not required; `category` was
  a proof-of-concept field with no production data

## Success Criteria
- SC-006.1: A user types `#health #project-alpha` into the tags field, saves,
  reloads, and sees exactly those two tags restored — no spaces accepted within
  a token, no silent reformatting.
- SC-006.2: A user activates a tag filter on the canvas — only matching
  initiatives remain visible, all others disappear, and a clear filter
  indicator is shown. Removing the filter restores all dots.
- SC-007.1: A user assigns size `L` to an initiative — the size is visible in
  the detail panel and persists after save/reload.
- SC-007.2: Five initiatives are assigned XS through XL — each renders as a
  visually distinct dot size on the canvas, with XS clearly the smallest and
  XL clearly the largest. A user with no prior knowledge can identify relative
  scale without opening any detail panel.
- SC-008.1: A user opens the report view and sees a table listing all canvas
  initiatives with columns for quadrant, name, tags, size, and details.
- SC-008.2: A user exports the report and receives a `.md` file that renders
  the same table correctly in any markdown viewer.
- SC-008.3: A user bulk-assigns a tag to three initiatives in the report view,
  returns to the canvas, and sees the filter system immediately reflect those
  tags — no reload, no stale state, no data loss.

## Tech constraints and dependencies
- `category` field removed entirely from the data model — clean replacement
  with `tags: string[]`; no migration logic required
- `size` field added as enum: `"XS" | "S" | "M" | "L" | "XL" | null`
- Tag format rule: each token must match `#[^\s]+`; multiple tokens
  space-separated in the input field
- Dot size rendered via CSS modifier classes (`.dot--xs` through `.dot--xl`)
  on the existing `.dot` element; no structural change to `renderCanvas()`
  required beyond adding the class
- Canvas filter state is session-only — not persisted to the project file
- **State synchronisation (critical)**: the canvas view and report view MUST
  share the same in-memory `initiatives[]` array — not separate copies. The
  report MUST mutate state in place and trigger `renderCanvas()` on return to
  the canvas. Do NOT reconstruct state from DOM or report table rows — always
  read from `initiatives[]`.
- Report view is read-only except for bulk tag assignment — no other editing
  from the report view
- Null/unset size renders at current default dot size (32px) — existing
  initiatives unaffected without a size value

## Test strategy
Approach: Manual only (standalone HTML tool — constitution §VII exception)
Phases:
- Phase 1 — Tags (US-006): tag entry, format validation, filter on/off,
  filter indicator visibility
- Phase 2 — T-shirt size and visual dots (US-007): size assignment in detail
  panel, five distinct dot sizes on canvas, null default renders at current
  size
- Phase 3 — Report and bulk tagging (US-008): report renders all initiatives
  with correct columns, bulk tag apply, state synchronisation on return to
  canvas, markdown export
Known exclusions:
- Canvas multi-select bulk tagging (deferred to wishlist)
- T-shirt → Agile level mapping (deferred)
- Report-to-action-plan integration (deferred)

## PO sign-off
Status: READY FOR SPECIFICATION
Approved by: Claudio
Date: 2026-04-09