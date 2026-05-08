# Quickstart: TMM Tags, Size & Report (002-tmm-tags-size-report)

**Branch**: `002-tmm-tags-size-report`
**Date**: 2026-05-08
**File under edit**: `public/tools/tmm/index.html`

---

## What this feature changes

This feature modifies the single-file TMM tool (`public/tools/tmm/index.html`) in-place. No new files are created; no build tools are added. All changes are HTML/CSS/JavaScript edits to one file.

**Layer A — UX fixes** (breaking changes to 001-tmm UI):
- Replace modal-per-initiative with Inbox + Quick Add bulk add workflow
- Move detail panel to always-open edit mode; add dirty-flag close gating
- Replace Category field with Tags field; add Size field to detail panel

**Layer B — New features**:
- Tags with `#token` validation and canvas filter
- T-shirt sizing with size-proportional dots and hollow null-size dot
- Report view (read-only table + bulk tag + markdown export)
- Unsaved-changes red banner

---

## File structure (unchanged)

```
public/tools/tmm/
└── index.html    ← single file; all HTML + CSS + JS in-lined
```

---

## Development workflow

No build step. Open the file directly in a browser:

```
file:///Users/claudiogomes/Desktop/Github/effectiveness-toolkit/public/tools/tmm/index.html
```

Or via the Vite dev server (serves `public/` statically):

```bash
npm run dev
# then navigate to http://localhost:5173/tools/tmm/
```

---

## Testing

### Automated tests
TMM is a standalone HTML tool — Constitution Section VII (Standalone HTML Tools Exception) applies. Manual testing is primary. No Vitest tests are required.

Manual test scenarios will be documented in `specs/002-tmm-tags-size-report/checklists/` after implementation.

### Manual smoke test after each task

1. Open `public/tools/tmm/index.html` in the browser
2. Verify no console errors
3. Add an initiative via Quick Add → verify it appears in Inbox
4. Drag it to canvas → verify dot appears, Inbox count decrements
5. Click dot → detail panel opens in edit mode
6. Edit Name + Tags + Size → Confirm → dot updates on canvas
7. Open Report view → verify initiative row present → Back to Canvas

---

## Key implementation notes

### Single-file constraint
Everything lives in `index.html`. CSS is in `<style>`, JS is in `<script>`. No imports, no modules.

### Shared state variables
All state is `var` declarations at the top of the `<script>` block. New variables for this feature:
- `isDirty` — unsaved changes flag
- `isDirtyPanel` — detail panel dirty flag
- `inboxItems` — unpositioned initiatives array
- `activeTag` — current tag filter (null = none)
- `currentView` — `'canvas'` or `'report'`

### Inbox items vs. canvas initiatives
An initiative in `inboxItems` also exists in `initiatives[]`. The Inbox is a view filter (unpositioned), not a separate data store. When dragged to canvas, the initiative's `tmm.x`/`tmm.y` are set and it is removed from `inboxItems`.

### Quadrant derivation helper
```javascript
function deriveQuadrant(x, y) {
  if (x < 0.5 && y >= 0.5) return 'Q1 — Firefighting';
  if (x >= 0.5 && y >= 0.5) return 'Q2 — Growth';
  if (x < 0.5 && y < 0.5)  return 'Q3 — Distraction';
  return 'Q4 — Waste';
}
```

### Tag validation helper
```javascript
function validateTags(input) {
  var tokens = input.trim().split(/\s+/).filter(Boolean);
  var invalid = tokens.filter(function(t) { return !/^#\S+$/.test(t); });
  return { tokens: tokens, invalid: invalid, valid: invalid.length === 0 };
}
```

### Dot rendering with size
```javascript
var SIZE_DIAMETER = { XS: 16, S: 24, M: 32, L: 44, XL: 58 };

function applyDotSize(dotEl, size) {
  var d = size ? SIZE_DIAMETER[size] : 32;
  dotEl.style.width  = d + 'px';
  dotEl.style.height = d + 'px';
  if (!size) {
    dotEl.style.background = 'transparent';
    dotEl.style.border     = '2px solid var(--color-primary)';
  } else {
    dotEl.style.background = 'var(--color-primary)';
    dotEl.style.border     = '2px solid #fff';
  }
}
```

---

## Commit strategy

One commit per task, using the format:
```
feat(tmm): <description> [T-002.XX]
```

Example:
```
feat(tmm): replace modal with inbox quick-add panel [T-002.01]
```
