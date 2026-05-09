# Implementation Plan: Time Management Matrix (TMM)

**Branch**: `001-tmm` | **Date**: 2026-04-06 | **Spec**: [specs/001-tmm/spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-tmm/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

The TMM is a standalone HTML tool that renders a four-quadrant Impact × Urgency canvas. Users add initiatives (Big Rocks) as labelled dots, drag them to position, view/edit details via a panel, and save/load the canvas state to the shared `effectivenessToolkit` JSON project file. Built with vanilla HTML/CSS/JavaScript using only native browser APIs, following the same standalone HTML pattern as the COER tool at `public/tools/coer/index.html`.

## Technical Context

**Language/Version**: HTML5 + vanilla JavaScript (ES6), inline CSS  
**Primary Dependencies**: None — native browser APIs only (HTML5 pointer events for drag-and-drop, File API for save/load)  
**Storage**: File-based JSON save/load via browser File API; shared `effectivenessToolkit` envelope  
**Testing**: Manual testing primary (Constitution VII — Standalone HTML Tools Exception); optional Playwright smoke tests  
**Target Platform**: Modern evergreen browsers (Chrome, Edge, Firefox, Safari)  
**Project Type**: Standalone HTML tool (single file, zero build tooling)  
**Performance Goals**: Canvas usable with 15+ initiatives — drag response immediate, render without visible delay (SC-005)  
**Constraints**: Offline-capable, no network requests after load, no third-party libraries, single HTML file with all JS/CSS inline  
**Scale/Scope**: Single-user, single-session, typically 5–15 initiatives per canvas

## Constitution Check (Pre-Phase 0)

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| # | Principle | Status | Rationale |
|---|-----------|--------|-----------|
| I | Offline-First Architecture | **PASS** | Standalone HTML file, no network requests, no backend |
| II | Single-Session, Single-User | **PASS** | No multi-user features, no collaboration |
| III | Privacy by Default | **PASS** | No telemetry, cookies, tracking, or third-party API calls |
| IV | Respect Upstream (mind-elixir) | **N/A** | TMM is not a mind-elixir tool |
| V | Simplicity Over Completeness | **PASS** | Single HTML file, native APIs only, no build tooling |
| VI | Shared Data Format & Tool Independence | **PASS** | Uses `effectivenessToolkit` envelope; TMM reads/writes only `tmm` section; does not require other tool sections |
| VII | Testing Standards | **PASS** | Standalone HTML Tools Exception: manual test scenarios primary; automated optional |
| VIII | Commit Standards | **PASS** | Conventional Commits with `tmm` scope will be used |
| IX | Environment Configuration | **N/A** | No build system, no `.env` needed |
| X | Console Output Standards | **N/A** | No build/batch processing |
| — | Form-Based Tools Guardrail | **PASS** | Standalone HTML, zero-config delivery, no build tools |
| — | Suite Infrastructure (Deployment) | **PASS** | Deployed to `public/tools/tmm/`, copied to `dist/` by Vite without processing |

**Gate result: PASS** — no violations, no justifications required.

## Project Structure

### Documentation (this feature)

```text
specs/001-tmm/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
│   └── project-file-schema.md
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
public/tools/tmm/
└── index.html           # Single standalone HTML file (all JS/CSS inline)
```

**Structure Decision**: Standalone HTML tool following the COER pattern. A single `index.html` file is placed in `public/tools/tmm/` so Vite copies it unprocessed to `dist/tools/tmm/` during build. No `src/` directories, no build pipeline, no external assets. The launcher page at `public/index.html` will need a link added to the TMM tool.

## Complexity Tracking

> No constitution violations — this section is intentionally empty.

No complexity justifications required.

---

## Constitution Check (Post-Phase 1 Design)

*Re-evaluated after Phase 1 design artifacts (data-model.md, contracts/, quickstart.md) were generated.*

| # | Principle | Status | Post-Design Notes |
|---|-----------|--------|-------------------|
| I | Offline-First | **PASS** | No network dependencies introduced in design |
| II | Single-Session | **PASS** | No multi-user/collaboration in design |
| III | Privacy | **PASS** | No telemetry or tracking in design |
| IV | Upstream | **N/A** | — |
| V | Simplicity | **PASS** | Pointer Events + CSS divs = simplest viable approach per research |
| VI | Shared Format | **PASS** | TMM section per initiative follows COER pattern exactly; round-trip preservation contract defined |
| VII | Testing | **PASS** | Standalone HTML Tools Exception applies; acceptance criteria in spec |
| VIII | Commits | **PASS** | — |
| — | Form-Based Tools | **PASS** | Single HTML file, no build tooling, no external dependencies |
| — | Suite Infrastructure | **PASS** | `public/tools/tmm/` deployment path confirmed |

**Post-design gate result: PASS** — no new violations introduced.
