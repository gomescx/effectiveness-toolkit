# Data Model: TMM Tags, Size & Report (002-tmm-tags-size-report)

**Feature**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md) | **Research**: [research.md](./research.md)
**Date**: 2026-05-08
**Extends**: [specs/001-tmm/data-model.md](../001-tmm/data-model.md)

---

## Entity: TMM Section (updated — replaces 001-tmm definition)

The `tmm` section is **updated** in this feature. `category` is removed; `tags` and `size` are added.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `tags` | array of string | No | `[]` | Ordered array of `#token` strings. Each token matches `^#\S+$`. Empty array for no tags. |
| `size` | `"XS"\|"S"\|"M"\|"L"\|"XL"\|null` | No | `null` | T-shirt size. `null` = unset. |
| `details` | string | No | `""` | Free-text description (unchanged from 001-tmm) |
| `x` | number (0.0–1.0) | Yes | `0.5` | Normalised Urgency coordinate (unchanged) |
| `y` | number (0.0–1.0) | Yes | `0.5` | Normalised Impact coordinate (unchanged) |
| ~~`category`~~ | ~~string~~ | — | — | **Removed**. Ignored on load; not written on save. |

**Validation rules**:
- Each element in `tags` must match `^#\S+$` — validated at input time; invalid tokens block submission
- Leading/trailing whitespace around the tags string is trimmed before tokenisation (edge case: `" #health  #career "` → `["#health", "#career"]`)
- `tags` must be an array, never `null` or empty string; new initiatives default to `[]`
- `size` accepts exactly one of `XS | S | M | L | XL | null`; any other value defaults to `null` on load
- `category` present in a loaded file is silently ignored (FR-US6.08); not written back on save

---

## Entity: Inbox Item (session-only, not persisted)

The Inbox holds initiatives that have been created but not yet positioned on the canvas. Inbox state is **not persisted** — it is cleared on load.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string (UUID v4) | Same ID as the corresponding Initiative |
| `name` | string | Copied from initiative at creation time |
| `tags` | array of string | Copied from initiative at creation time |
| `size` | enum \| null | Copied from initiative at creation time |

**State transitions**:
```
[Quick Add form submitted] → [Inbox item created] → [Dragged to canvas]
                                                          → [Initiative.tmm.x/y set]
                                                          → [Inbox item removed]
                          → [× remove] → [Inbox item AND initiative deleted]
```

---

## Entity: Filter State (session-only, not persisted)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `activeTag` | string \| null | `null` | The single active tag filter token. `null` = no filter active. Reset to `null` on every load. |

---

## Dot Rendering — Size → Diameter Mapping

| Size | Diameter | Fill |
|------|----------|------|
| `null` (unset) | 32px | **Hollow** — transparent fill, `var(--color-primary)` border |
| `"XS"` | 16px | Filled — `var(--color-primary)` |
| `"S"` | 24px | Filled — `var(--color-primary)` |
| `"M"` | 32px | Filled — `var(--color-primary)` |
| `"L"` | 44px | Filled — `var(--color-primary)` |
| `"XL"` | 58px | Filled — `var(--color-primary)` |

**Key distinction** (FR-UX-021): `null` and `"M"` both render at 32px diameter, but `null` is hollow (no fill) and `"M"` is filled. The hollow state signals "sizing not yet done".

---

## Quadrant Derivation (Report View)

Derived from initiative `x`/`y` coordinates. Used in the Report view Q column (FR-US8.03).

| Condition | Label |
|-----------|-------|
| `x < 0.5 && y >= 0.5` | Q1 — Firefighting |
| `x >= 0.5 && y >= 0.5` | Q2 — Growth |
| `x < 0.5 && y < 0.5` | Q3 — Distraction |
| `x >= 0.5 && y < 0.5` | Q4 — Waste |

Note: `x < 0.5` = High Urgency (left); `x >= 0.5` = Low Urgency (right); `y >= 0.5` = High Impact (top); `y < 0.5` = Low Impact (bottom).

---

## In-Memory State (full updated model)

```javascript
// Application-level state
var initiatives      = [];   // Array<Initiative> — single source of truth
var loadedProjectFile = null; // raw parsed JSON from last Load
var isDirty          = false; // unsaved changes banner gate (FR-UX-013)
var selectedId       = null;  // detail panel open for this initiative id
var isDirtyPanel     = false; // detail panel change-tracking (FR-UX-009)
var editSnapshot     = null;  // { name, tags, size, details } on panel open
var activeTag        = null;  // tag filter (FR-US6.04); null = no filter
var inboxItems       = [];   // Array<InboxItem> — unpositioned initiatives
var currentView      = 'canvas'; // 'canvas' | 'report'

// Initiative structure
{
  id:        string,          // UUID v4, stable cross-tool identifier
  name:      string,          // required
  coer:      object | null,
  sob:       object | null,
  memoryMap: object | null,
  impactMap: object | null,
  tmm: {
    tags:    string[],        // default []
    size:    'XS'|'S'|'M'|'L'|'XL'|null, // default null
    details: string,          // default ""
    x:       number,          // [0.0, 1.0]
    y:       number           // [0.0, 1.0]
  }
}
```

---

## State Transitions

### Initiative Lifecycle (updated)

```
[Quick Add form] → [Inbox item (unpositioned)] → [Drag to canvas] → [On Canvas (x,y set)]
                       → [× Remove] → [Deleted]                         │
                                                                    [Drag to reposition]
                                                                         │
                                                                    [Detail panel edit]
                                                                         │
                                                                    [💾 Save → persisted]
```

### Detail Panel States (updated)

```
[Closed] ──click dot──→ [Open: edit mode, cursor in Name, isDirtyPanel=false]
              │                 │
              │           [field change → isDirtyPanel=true]
              │                 │                │
              │           [Confirm] ──→ [saved, panel closes]
              │           [Cancel]  ──→ [reverted, panel closes]
              │
         [same dot click, no changes] ──→ [panel closes]
         [same dot click, isDirtyPanel] ──→ [blocked — no close]
         [blank canvas click] ──→ [no action]
```

### View States

```
[Canvas view] ──[📊 Report]──→ [Report view]
[Report view] ──[← Back to Canvas]──→ [Canvas view]
```
