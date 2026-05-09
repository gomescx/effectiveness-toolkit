# Contracts: TMM Tags, Size & Report — Project File Schema (002-tmm-tags-size-report)

**Feature**: [../spec.md](../spec.md) | **Data Model**: [../data-model.md](../data-model.md)
**Date**: 2026-05-08
**Supersedes**: [specs/001-tmm/contracts/project-file-schema.md](../../001-tmm/contracts/project-file-schema.md) for the `tmm` section only.

---

## Contract 1: TMM Section Schema (v2 — updated)

**Change from v1**: `category` removed; `tags` (array) and `size` (enum | null) added.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TMM Section (per initiative) — v2",
  "description": "Time Management Matrix data. category field removed; tags and size added.",
  "type": "object",
  "required": ["x", "y"],
  "additionalProperties": true,
  "properties": {
    "tags": {
      "type": "array",
      "items": {
        "type": "string",
        "pattern": "^#\\S+$"
      },
      "default": [],
      "description": "Ordered array of hashtag-prefixed tag tokens. Empty array = no tags."
    },
    "size": {
      "oneOf": [
        {
          "type": "string",
          "enum": ["XS", "S", "M", "L", "XL"]
        },
        { "type": "null" }
      ],
      "default": null,
      "description": "T-shirt size. null = not yet sized."
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
      "description": "Normalised Urgency coordinate (0.0 = high urgency/left, 1.0 = low urgency/right)"
    },
    "y": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "default": 0.5,
      "description": "Normalised Impact coordinate (0.0 = low impact/bottom, 1.0 = high impact/top)"
    }
  }
}
```

### Migration notes

- Files produced by v1 (001-tmm) may contain `category` in the `tmm` section. This field is **silently ignored on load** (FR-US6.08) and **not written on save**.
- Files produced by v1 will have no `tags` field — default to `[]` on load (FR-US6.03).
- Files produced by v1 will have no `size` field — default to `null` on load (FR-US7.01).
- `additionalProperties: true` ensures unknown fields are preserved on round-trip.

---

## Contract 2: Inbox Item (session-only, not persisted)

Inbox items are held in memory only. They are never written to the project file. This contract documents the in-memory shape for implementation clarity.

```json
{
  "type": "object",
  "required": ["id", "name"],
  "properties": {
    "id": {
      "type": "string",
      "description": "Same UUID as the corresponding initiative in initiatives[]"
    },
    "name": { "type": "string" },
    "tags": { "type": "array", "items": { "type": "string" }, "default": [] },
    "size": { "oneOf": [{ "type": "string", "enum": ["XS","S","M","L","XL"] }, { "type": "null" }], "default": null }
  }
}
```

---

## Contract 3: Markdown Export Format

The exported `.md` file contains a standard GFM table. This contract defines the exact format.

### Table structure

```markdown
| Quadrant | Name | Tags | Size | Details |
|---|---|---|---|---|
| Q1 — Firefighting | Initiative A | #health #career | L | Notes |
| Q2 — Growth | Initiative B | #finance |  |  |
```

### Rules

- **Quadrant column**: Short label `Q1 — Firefighting`, `Q2 — Growth`, `Q3 — Distraction`, `Q4 — Waste`. Derived from `x`/`y` per data-model quadrant derivation table.
- **Tags column**: All tag tokens joined with a single space. Empty string if no tags.
- **Size column**: Enum value string (`XS`, `S`, `M`, `L`, `XL`). **Empty string** (not `null`, not `"null"`) if size is `null` (FR-US8.11).
- **Details column**: Raw text; pipe characters (`|`) in details MUST be escaped as `\|` to avoid breaking the table.
- **Sort order**: Initiatives are listed in the order they appear in `initiatives[]` (insertion order).
- **Filename**: `effectiveness-toolkit-tmm-report.md`

---

## Contract 4: Filter State (session-only, not persisted)

| Field | Type | Default | Persistence |
|-------|------|---------|-------------|
| `activeTag` | `string \| null` | `null` | Never persisted. Reset to `null` on every load. (FR-000.06) |

The tag filter source list is derived from the union of all `tags` values present in `initiatives[]` at render time. No separate tag registry is maintained.

---

## Example: Updated Project File

```json
{
  "effectivenessToolkit": {
    "version": "1.0",
    "lastModified": "2026-05-08T10:00:00.000Z",
    "initiatives": [
      {
        "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "name": "Improve team communication",
        "coer": null,
        "sob": null,
        "memoryMap": null,
        "impactMap": null,
        "tmm": {
          "tags": ["#career", "#health"],
          "size": "L",
          "details": "Weekly 1:1s with each direct report",
          "x": 0.25,
          "y": 0.75
        }
      },
      {
        "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "name": "Fix broken CI pipeline",
        "coer": null,
        "sob": null,
        "memoryMap": null,
        "impactMap": null,
        "tmm": {
          "tags": [],
          "size": null,
          "details": "",
          "x": 0.1,
          "y": 0.9
        }
      }
    ]
  }
}
```
