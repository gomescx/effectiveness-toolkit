# Data Model: Time Management Matrix (TMM)

**Feature**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md) | **Research**: [research.md](./research.md)
**Date**: 2026-04-06

---

## Entity: Initiative (shared, initiative-level)

The initiative is the top-level shared entity in the `effectivenessToolkit` project file. The TMM tool creates and manages initiatives in the `initiatives[]` array, reading/writing only the `tmm` section while preserving all other tool sections.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string (UUID v4) | Yes | Unique identifier, generated on creation |
| `name` | string | Yes | Initiative name (free text, no uniqueness enforced) |
| `coer` | object \| null | No | COER tool data (preserved by TMM, never modified) |
| `sob` | object \| null | No | Strength of Belief data (preserved by TMM) |
| `memoryMap` | object \| null | No | Memory Map data (preserved by TMM) |
| `tmm` | TMM object \| null | No | TMM-specific data (created/managed by this tool) |
| `impactMap` | object \| null | No | Impact Map data (preserved by TMM) |

**Validation rules**:
- `name` is required; empty string is rejected at the form level (FR-US1.02, FR-US5.02)
- `id` is generated as UUID v4 on creation; never user-editable
- Unknown fields at the initiative level are preserved on round-trip (Constitution Principle VI)

---

## Entity: TMM Section (tool-specific, per initiative)

The `tmm` section within each initiative stores the TMM-specific data: category, details, and canvas position.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `category` | string | No | `""` | User-defined category (free text, no predefined list) — FR-US3.01 |
| `details` | string | No | `""` | Free-text details/description — FR-US1.02 |
| `x` | number (0.0–1.0) | Yes | `0.5` | Normalised Urgency coordinate (0 = low/left, 1 = high/right) — FR-US2.03 |
| `y` | number (0.0–1.0) | Yes | `0.5` | Normalised Impact coordinate (0 = low/bottom, 1 = high/top) — FR-US2.03 |

**Validation rules**:
- `x` and `y` are clamped to [0.0, 1.0] on drag release (FR-US2.04)
- `category` accepts any free-text string; no format or uniqueness validation (FR-US3.01)
- `details` accepts any free-text string; no length restriction
- Default position (0.5, 0.5) = canvas centre (FR-US1.03)

---

## Entity: Project File Envelope

The top-level wrapper for the shared JSON file. The TMM tool reads/writes this structure on save/load.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `effectivenessToolkit.version` | string | Yes | Schema version. Current: `"1.0"` |
| `effectivenessToolkit.lastModified` | string (ISO 8601) | Yes | Updated to current timestamp on each save (FR-US4.07) |
| `effectivenessToolkit.initiatives` | array of Initiative | Yes | Array of all initiatives |

---

## Relationships

```text
Project File (effectivenessToolkit envelope)
└── initiatives[] (shared array)
    └── Initiative
        ├── id, name (shared fields)
        ├── tmm: { category, details, x, y }   ← TMM owns this section
        ├── coer: { ... } | null                ← preserved, not modified
        ├── sob: { ... } | null                 ← preserved, not modified
        ├── memoryMap: { ... } | null           ← preserved, not modified
        └── impactMap: { ... } | null           ← preserved, not modified
```

---

## State Transitions

### Initiative Lifecycle (TMM perspective)

```text
[Created] ──add──→ [On Canvas at centre (0.5, 0.5)]
                         │
                    drag──→ [Repositioned (x, y updated)]
                         │
                    edit──→ [Updated (name/category/details changed)]
                         │
                    save──→ [Persisted to JSON file]
                         │
                    load──→ [Restored from JSON file]
```

### Detail Panel States

```text
[Closed] ──click dot──→ [Viewing] ──edit action──→ [Editing]
   ↑                        │                          │
   └────close──────────────←─┘                          │
   ↑                                                    │
   └──────────confirm/cancel────────────────────────────┘
```

---

## Canvas Coordinate System

```text
Impact (y)
1.0 ┌──────────────────┬──────────────────┐
    │  High Impact      │  High Impact      │
    │  Low Urgency      │  High Urgency     │
    │  (Q2 — Plan)      │  (Q1 — Do)        │
0.5 ├──────────────────┼──────────────────┤
    │  Low Impact       │  Low Impact       │
    │  Low Urgency      │  High Urgency     │
    │  (Q4 — Eliminate) │  (Q3 — Delegate)  │
0.0 └──────────────────┴──────────────────┘
   0.0                 0.5                 1.0
                                    Urgency (x)
```

**Mapping to DOM**:
- `left = x * 100%` (normalised Urgency → horizontal position)
- `top = (1 - y) * 100%` (normalised Impact → vertical position, inverted because CSS top increases downward)

---

## In-Memory State (JavaScript)

The TMM tool maintains the following state in memory during a session:

| Variable | Type | Purpose |
|----------|------|---------|
| `initiatives` | array of objects | Current canvas state — all initiatives with their fields |
| `loadedProjectFile` | object \| null | Full parsed JSON for round-trip preservation |
| `selectedInitiativeId` | string \| null | ID of the initiative whose detail panel is open |
| `isEditing` | boolean | Whether the detail panel is in edit mode |
