# Contracts: TMM — Project File Schema

**Feature**: [../spec.md](../spec.md) | **Data Model**: [../data-model.md](../data-model.md)

## Overview

The TMM tool is a standalone HTML application with no backend. All "contracts" are client-side data format agreements governing the shared JSON project file. These contracts ensure inter-tool compatibility per Constitution Principle VI.

---

## Contract 1: TMM Section Schema

**Purpose**: Define the structure of the `tmm` section within each initiative in the shared project file.

### Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TMM Section (per initiative)",
  "description": "Time Management Matrix data stored within each initiative's tmm field",
  "type": "object",
  "required": ["x", "y"],
  "additionalProperties": true,
  "properties": {
    "category": {
      "type": "string",
      "default": "",
      "description": "User-defined category label (free text, no predefined list)"
    },
    "details": {
      "type": "string",
      "default": "",
      "description": "Free-text description or notes for the initiative"
    },
    "x": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "default": 0.5,
      "description": "Normalised Urgency coordinate (0.0 = low/left, 1.0 = high/right)"
    },
    "y": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "default": 0.5,
      "description": "Normalised Impact coordinate (0.0 = low/bottom, 1.0 = high/top)"
    }
  }
}
```

### Notes

- `category` and `details` default to empty string (`""`) when not provided by the user (FR-US3.04).
- `x` and `y` default to `0.5` (canvas centre) when an initiative is first added (FR-US1.03).
- `additionalProperties: true` ensures unknown fields are preserved on round-trip.
- `id` and `name` are NOT part of the `tmm` section — they are initiative-level fields per the shared project file schema (see Research Task 1).

---

## Contract 2: Initiative-Level Fields (TMM Creates)

**Purpose**: Define the initiative-level fields that the TMM tool creates when adding a new initiative.

### Schema

```json
{
  "type": "object",
  "required": ["id", "name"],
  "additionalProperties": true,
  "properties": {
    "id": {
      "type": "string",
      "description": "UUID v4, generated on creation"
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "description": "Initiative name (required, user-provided)"
    },
    "tmm": {
      "$ref": "#contract-1",
      "description": "TMM section with canvas position and metadata"
    },
    "coer": {
      "description": "COER data — null when created by TMM",
      "default": null
    },
    "sob": {
      "description": "Strength of Belief data — null when created by TMM",
      "default": null
    },
    "memoryMap": {
      "description": "Memory Map data — null when created by TMM",
      "default": null
    },
    "impactMap": {
      "description": "Impact Map data — null when created by TMM",
      "default": null
    }
  }
}
```

### Example: New initiative created by TMM

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "Improve team communication",
  "tmm": {
    "category": "Career",
    "details": "Weekly 1:1s with each direct report",
    "x": 0.75,
    "y": 0.85
  },
  "coer": null,
  "sob": null,
  "memoryMap": null,
  "impactMap": null
}
```

---

## Contract 3: Full Project File Example (TMM with multiple initiatives)

**Purpose**: Illustrate the complete project file structure when the TMM tool has created and positioned multiple initiatives.

```json
{
  "effectivenessToolkit": {
    "version": "1.0",
    "lastModified": "2026-04-06T14:30:00.000Z",
    "initiatives": [
      {
        "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "name": "Improve team communication",
        "tmm": {
          "category": "Career",
          "details": "Weekly 1:1s with each direct report",
          "x": 0.75,
          "y": 0.85
        },
        "coer": {
          "lastModified": "2026-04-05T10:00:00.000Z",
          "specificResults": ["Monthly feedback score above 8"],
          "reason": "Build trust with the team"
        },
        "sob": null,
        "memoryMap": null,
        "impactMap": null
      },
      {
        "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "name": "Launch fitness routine",
        "tmm": {
          "category": "Health",
          "details": "",
          "x": 0.3,
          "y": 0.6
        },
        "coer": null,
        "sob": null,
        "memoryMap": null,
        "impactMap": null
      },
      {
        "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
        "name": "Family holiday planning",
        "tmm": {
          "category": "",
          "details": "Research destinations for August",
          "x": 0.2,
          "y": 0.25
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

---

## Contract 4: Round-Trip Preservation

**Purpose**: Guarantee that loading and re-saving a project file does not lose data from other tools.

### Rules

1. On load, the TMM tool reads the entire JSON file into memory (`loadedProjectFile`).
2. The TMM tool modifies **only**:
   - `effectivenessToolkit.lastModified` (updated to current ISO 8601 timestamp)
   - `effectivenessToolkit.initiatives[*].name` (if the user edits an initiative name)
   - `effectivenessToolkit.initiatives[*].tmm` (canvas position and TMM-specific fields)
   - New entries appended to `effectivenessToolkit.initiatives[]` (when user adds an initiative)
3. All other fields within each initiative (`coer`, `sob`, `memoryMap`, `impactMap`, unknown fields) are written back **exactly as loaded**.
4. Fields that were `null` remain `null`. Fields that were absent (key not present) remain absent.
5. Unknown fields at any nesting level are preserved — no stripping, no defaulting.
6. Initiatives that exist in the loaded file but have no `tmm` section are displayed at canvas centre (0.5, 0.5) and only receive a `tmm` section if the user interacts with them (drags or edits).

### Test Scenarios

| Scenario | Input | Expected Output |
|----------|-------|-----------------|
| Load file with `coer` data on initiative | `"coer": { "reason": "trust" }` | `"coer": { "reason": "trust" }` preserved after TMM save |
| Load file with `sob: null` | `"sob": null` | `"sob": null` in saved file |
| Load file with unknown initiative field | `"customField": true` | `"customField": true` preserved |
| Load file with no `tmm` section on initiative | No `tmm` key | Initiative shown at centre; no `tmm` key written unless user interacts |
| Add new initiative in TMM, save | — | New initiative has `id`, `name`, `tmm`; other tool sections set to `null` |
| Load file with unknown fields at envelope level | `"newField": 42` | `"newField": 42` preserved |

---

## Contract 5: File Naming Convention

**Purpose**: Define the filename pattern for TMM save operations.

### Pattern

```
<sanitised-initiative-name>-tmm-<YYYY-MM-DD>.json
```

- If no initiatives exist at save time, use `tmm-<YYYY-MM-DD>.json`
- Sanitisation: strip filesystem-unsafe characters (`/\?%*:|"<>`), replace whitespace with `_`, truncate to 80 characters
- Example: `Career_Goals-tmm-2026-04-06.json`

**Note**: Since TMM manages multiple initiatives, the filename uses a generic label or the first initiative's name as a convenience. The file content always contains all initiatives.
