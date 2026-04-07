# Data Model: TMM — Time Management Matrix

**Feature**: [spec.md](spec.md) | **Plan**: [plan.md](plan.md)

## Canvas Coordinate System

The TMM canvas uses a normalised two-dimensional coordinate space where both
axes run from `0.0` to `1.0`.

```
  (0.0, 0.0) ─────────────────── (1.0, 0.0)
       │  Q1 Firefighting  │  Q2 Growth        │
       │  Urgent &         │  Not Urgent but   │   ▲  y = 0.0 = top = high impact
       │  Important        │  Important        │   │
       ├───────────────────┼───────────────────┤   │  y increases downward
       │  Q3 Distraction   │  Q4 Waste         │   │
       │  Urgent but       │  Not Urgent &     │   ▼  y = 1.0 = bottom = low impact
       │  Not Important    │  Not Important    │
  (0.0, 1.0) ─────────────────── (1.0, 1.0)

  x = 0.0 (left) = high urgency
  x = 1.0 (right) = low urgency
```

### X-axis — Urgency

| Value | Position | Meaning |
|-------|----------|---------|
| `0.0` | Left edge | Most urgent |
| `0.5` | Centre    | Moderate urgency |
| `1.0` | Right edge | Least urgent |

> The x-axis runs **left (near/urgent) → right (distant/less urgent)**.
> x = 0.0 is the high-urgency side; x = 1.0 is the low-urgency side.

### Y-axis — Impact

| Value | Position | Meaning |
|-------|----------|---------|
| `0.0` | Top edge  | Highest impact |
| `0.5` | Centre    | Moderate impact |
| `1.0` | Bottom edge | Lowest impact |

> The y-axis runs **top (high impact) → bottom (low impact)**.

### Quadrant Assignment

| Quadrant | Label | Condition | Colour |
|----------|-------|-----------|--------|
| Q1 | Firefighting — Urgent &amp; Important | `x < 0.5` and `y < 0.5` | `#FECACA` (red-200) |
| Q2 | Growth — Not Urgent but Important | `x ≥ 0.5` and `y < 0.5` | `#BBF7D0` (green-200) |
| Q3 | Distraction — Urgent but Not Important | `x < 0.5` and `y ≥ 0.5` | `#FEF08A` (yellow-200) |
| Q4 | Waste — Not Urgent &amp; Not Important | `x ≥ 0.5` and `y ≥ 0.5` | `#E2E8F0` (slate-200) |

### Canvas centre

A dot placed at `(x: 0.5, y: 0.5)` appears exactly at the visual centre of the
canvas, on the intersection of the two axis lines. This is equidistant from all
four quadrant boundaries.

---

## Entities

### ProjectFile (Envelope)

Shared with all Effectiveness Toolkit tools per Constitution Principle VI.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `effectivenessToolkit` | object | Yes | Top-level namespace |
| `effectivenessToolkit.version` | string | Yes | Schema version (`"1.0"`) |
| `effectivenessToolkit.lastModified` | string | Yes | ISO 8601 timestamp of last save |
| `effectivenessToolkit.initiatives` | Initiative[] | Yes | Array of initiatives |

### Initiative

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | UUID v4 |
| `name` | string | Yes | Initiative name (free text) |
| `tmm` | TMMPlacement \| null | No | TMM canvas placement. `null` or absent if not placed |
| `coer` | object \| null | No | COER data (other tool — preserved on round-trip) |
| `sob` | object \| null | No | Strength of Belief data (preserved) |
| `memoryMap` | object \| null | No | Memory Map data (preserved) |
| `impactMap` | object \| null | No | Impact Map data (preserved) |

### TMMPlacement

Position of one initiative on the TMM canvas.

| Field | Type | Range | Description |
|-------|------|-------|-------------|
| `lastModified` | string | — | ISO 8601 timestamp set at save |
| `x` | number | `[0.0, 1.0]` | Normalised horizontal position; `0.0` = most urgent (left), `1.0` = least urgent (right) |
| `y` | number | `[0.0, 1.0]` | Normalised vertical position; `0.0` = highest impact (top), `1.0` = lowest impact (bottom) |

---

## Canonical JSON Example

```json
{
  "effectivenessToolkit": {
    "version": "1.0",
    "lastModified": "2026-04-07T10:00:00.000Z",
    "initiatives": [
      {
        "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "name": "Resolve production outage",
        "tmm": {
          "lastModified": "2026-04-07T10:00:00.000Z",
          "x": 0.1,
          "y": 0.15
        },
        "coer": null,
        "sob": null,
        "memoryMap": null,
        "impactMap": null
      },
      {
        "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "name": "Build coaching skills",
        "tmm": {
          "lastModified": "2026-04-07T10:00:00.000Z",
          "x": 0.75,
          "y": 0.2
        },
        "coer": null,
        "sob": null,
        "memoryMap": null,
        "impactMap": null
      }
    ]
  }
}
```

In this example:
- "Resolve production outage" is at `(x: 0.1, y: 0.15)` → Q1 Firefighting (top-left, urgent and important).
- "Build coaching skills" is at `(x: 0.75, y: 0.2)` → Q2 Growth (top-right, not urgent but important).
