# Business Statement — Time Management Matrix (TMM)

## What this feature does
The Time Management Matrix is a visual prioritisation canvas based on
Stephen Covey's Impact × Urgency model. Users place their "big rocks"
(initiatives) as draggable dots on a four-quadrant canvas, assign each
a user-defined category and a free-text details field, and save the
result to a JSON file. The PoC is a standalone tool; the JSON format is
structured for clean integration with the shared effectivenessToolkit
envelope in a future feature.

## Why it matters
PEP coaches and executives need a fast, visual way to see where their
initiatives sit across urgency and importance — not just a ranked list.
The drag-and-drop interaction makes reprioritisation effortless as
circumstances change, and the category field lets each user organise
their big rocks by life area or role in a way that is meaningful to them
personally.

## User Stories
- US-001: As a PEP coach or executive, I want to place my big rocks on
  a visual Impact × Urgency canvas, so that I can see at a glance where
  to focus my attention and energy.
- US-002: As a PEP coach or executive, I want to add, edit, and
  reposition initiatives on the canvas by dragging them with the mouse,
  so that my priorities stay current as circumstances change.
- US-003: As a PEP coach or executive, I want to assign a user-defined
  category to each initiative, so that I can group my big rocks by life
  area or role when reviewing them.

## Scope

### In scope
- Four-quadrant canvas (Impact vertical, Urgency horizontal)
- Add a new initiative: name, category (free text), details (free text)
- Dot representation per initiative, positioned on the canvas
- Drag-and-drop repositioning; position maps to Impact/Urgency coordinates
- Click a dot to view initiative details (name, category, details text)
- JSON save/load (standalone PoC file, structured for future envelope
  compatibility using the effectivenessToolkit envelope with a tmm section)
- User-defined categories stored per initiative

### Out of scope (this feature)
- Markdown rendering of the details field — post-PoC enhancement
- Category management table — deferred to JSON integration feature
- Integration with COER / Memory Map JSON files — next feature
- Filtering or grouping by category on the canvas — post-PoC enhancement
- Predefined or mandated category lists — user-defined only by design
- LLM-generated descriptions — future capability, not PoC scope

## Success Criteria
- SC-001: A user can open the tool, add a new initiative with a name,
  category, and details, and see it appear as a dot on the canvas in
  the correct quadrant.
- SC-002: A user can drag a dot to a new position on the canvas and the
  change persists when the file is saved and reloaded.
- SC-003: A user can save the canvas to a JSON file, reload it, and see
  all initiatives restored with their positions, names, categories, and
  details intact.
- SC-004: A user can click on a dot and view the initiative's details
  (name, category, details text) without leaving the canvas.

## Tech constraints and dependencies
- Standalone HTML tool — no build tooling (per constitution Form-Based
  Tools guardrail)
- Drag-and-drop via native browser APIs (mouse events or HTML5 drag) —
  no heavy libraries
- JSON file must use the effectivenessToolkit envelope structure with a
  tmm section per initiative, even as a standalone PoC
- Deploy to public/tools/tmm/ following suite infrastructure pattern
- Branch name follows existing 001-xxx convention (e.g. 001-tmm)

## PO sign-off
Status: READY FOR SPECIFICATION
Approved by: Claudio
Date: 2026-04-06
