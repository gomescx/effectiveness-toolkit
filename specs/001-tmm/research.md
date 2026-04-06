# Research: Time Management Matrix (TMM)

**Feature**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)
**Date**: 2026-04-06

---

## Research Task 1: TMM Data Placement in the Shared Project File

### Question

FR-US4.03 lists `id`, `name`, `category`, `details`, `x`, `y` as fields in the `tmm` section. The existing project-file-schema (COER contract) places `id` and `name` at the initiative level. Should `id`/`name` be duplicated inside `tmm`, or follow the COER pattern?

### Findings

The existing project file schema (`specs/001-coer-form/contracts/project-file-schema.md`) defines:

```json
"Initiative": {
  "required": ["id", "name"],
  "properties": {
    "id": { "type": "string" },
    "name": { "type": "string" },
    "coer": { ... },
    "tmm": { ... },
    ...
  }
}
```

The COER tool (`public/tools/coer/index.html`) stores `id` and `name` at the initiative level and tool-specific data within the `coer` section. Constitution Principle VI requires a shared data format with namespaced tool sections.

### Decision

**Follow the COER pattern**: `id` and `name` are initiative-level fields. The `tmm` section contains only TMM-specific data: `category`, `details`, `x`, `y`. FR-US4.03's listing of `id` and `name` is interpreted as "each initiative has these fields" — they are not duplicated inside the `tmm` sub-object.

### Rationale

- Consistent with the existing COER implementation and project-file-schema contract
- Avoids data duplication and potential sync issues between initiative-level and tmm-level name fields
- Upholds Constitution Principle VI: shared data format with namespaced tool sections

### Alternatives Considered

- **Duplicate `id`/`name` inside `tmm`**: Rejected — creates two sources of truth for name, invites divergence, inconsistent with COER pattern.
- **Store everything flat at initiative level (no `tmm` section)**: Rejected — violates namespaced tool-section pattern required by Constitution Principle VI.

---

## Research Task 2: Drag-and-Drop Implementation Approach

### Question

FR-US2.01 requires native browser drag-and-drop APIs. Which native API is best for smooth, real-time repositioning of DOM elements on a canvas?

### Findings

Three native approaches were evaluated:

| Criterion | HTML5 DnD API | Pointer Events | Mouse Events |
|---|---|---|---|
| Smooth real-time tracking | No (ghost image, throttled) | Yes (frame-rate events) | Yes (desktop only) |
| Touch device support | No | Yes (unified input) | Requires duplication |
| Pointer capture | N/A | Built-in (`setPointerCapture`) | Manual (document listeners) |
| Boundary clamping | Difficult | Trivial | Trivial |
| Coordinate access during drag | Unreliable (Firefox `drag` events) | Reliable | Reliable |
| Modern evergreen support | Partial | Full (Chrome 55+, Edge 12+, Firefox 59+, Safari 13+) | Full |

### Decision

**Use the Pointer Events API** (`pointerdown`, `pointermove`, `pointerup`).

### Rationale

- Pointer Events provide smooth, frame-rate coordinate updates — essential for a dot that must "follow the cursor in real time" (FR-US2.02)
- `setPointerCapture()` ensures drag continues even if pointer moves faster than the dot
- Unified input model handles mouse, touch, and pen with a single code path
- Boundary clamping is straightforward: `Math.max(0, Math.min(containerSize, pos))`
- The HTML5 DnD API was designed for inter-zone data transfer (e.g., drag files between panels), not intra-container repositioning

### Alternatives Considered

- **HTML5 Drag and Drop API**: Rejected — provides no real-time visual feedback for repositioning (ghost image lags, cannot style per-frame), unreliable coordinate access during `drag` events on Firefox, no touch support.
- **Mouse Events**: Viable but inferior — requires duplicating logic for `touchstart`/`touchmove`/`touchend` on touch devices, lacks pointer capture, no benefit over Pointer Events on any modern browser.
- **Mouse + Touch hybrid**: Strictly worse than Pointer Events — more code, more surface for bugs, identical result.

### Implementation Pattern

```
pointerdown  → setPointerCapture, record offset, add dragging class
pointermove  → clamp to container bounds, update transform/left/top
pointerup    → releasePointerCapture, compute normalised coords, store
```

Key CSS on draggable dot elements:
```css
touch-action: none;      /* prevent scroll/zoom hijack */
will-change: transform;  /* GPU compositing hint */
cursor: grab;
```

Normalised coordinate calculation:
```
normX = (dotCenterX - canvasLeft) / canvasWidth    // 0.0–1.0 (Urgency)
normY = 1 - (dotCenterY - canvasTop) / canvasHeight // 0.0–1.0 (Impact, inverted: high at top)
```

---

## Research Task 3: Canvas Rendering Approach

### Question

Should the four-quadrant canvas be built with CSS-positioned divs, an HTML5 `<canvas>` element, or SVG?

### Findings

| Approach | Pros | Cons |
|---|---|---|
| CSS divs + absolute positioning | Simple DOM structure; native text rendering; easy event binding per dot; accessible (real DOM nodes); no redraw logic | Many absolutely-positioned elements could be complex; needs careful z-index management |
| HTML5 `<canvas>` | Efficient for many elements; single draw surface | Text rendering requires manual measurement; no native DOM events per dot (hit-testing needed); inaccessible to screen readers; complex redraw loop |
| SVG | Scalable; per-element events; accessible | More complex than divs for this use case; namespace management; overkill for ~15 elements |

### Decision

**Use CSS divs with absolute positioning** inside a relatively-positioned container.

### Rationale

- Simplest approach for the expected scale (~15 initiatives per SC-005)
- Each dot is a real DOM element: native event binding, accessible, keyboard-navigable
- Native text rendering for dot labels — no canvas text measurement
- Quadrant labels and grid lines are simple child divs or pseudo-elements
- Consistent with Constitution Principle V (Simplicity Over Completeness)
- The COER tool uses standard DOM elements; no justification for heavier technology

### Alternatives Considered

- **HTML5 `<canvas>`**: Rejected — over-engineering for ~15 elements; requires manual hit-testing, text measurement, and accessibility overlays. Would violate Simplicity principle.
- **SVG**: Rejected — adds namespace complexity for no benefit at this scale. Would be reconsidered if the tool needed to support 100+ elements.

### Implementation Pattern

```html
<div id="canvas" style="position: relative; width: 100%; aspect-ratio: 1;">
  <!-- Quadrant labels -->
  <div class="quadrant-label" style="top: 25%; left: 25%;">High Impact / Low Urgency</div>
  <!-- ... other quadrant labels ... -->
  
  <!-- Initiative dots (dynamically created) -->
  <div class="dot" style="position: absolute; left: 50%; top: 50%; transform: translate(-50%, -50%);">
    <span class="dot-label">Initiative Name</span>
  </div>
</div>
```

Dot positioning uses `left` (% of container width, maps to Urgency) and `top` (% of container height, maps to inverted Impact). Conversion:
- `left = x * 100%` (x = normalised Urgency 0.0–1.0)
- `top = (1 - y) * 100%` (y = normalised Impact 0.0–1.0, high at top)

---

## Research Task 4: Dot Label Truncation and Overlap

### Question

How to handle long initiative names on dot labels, and what happens when dots overlap?

### Findings

The spec requires:
- Edge case: "An initiative name longer than 30 characters is truncated visually on the dot label but stored and displayed in full in the detail panel"
- Edge case: "Two dots placed at identical coordinates are both rendered (stacked or slightly offset); neither is hidden"

### Decision

1. **Label truncation**: Use CSS `max-width` + `text-overflow: ellipsis` on the label span. Store the full name; only the visual label is truncated.
2. **Overlap handling**: Use a small z-index increment per dot (based on creation order or last-dragged-to-top). When dots overlap, the most recently interacted dot appears on top. No automatic offset — the user can drag to separate.

### Rationale

- CSS truncation is zero-effort and visually clean
- Z-index stacking gives the user implicit control (last touched = on top) without complex collision detection
- Both approaches are consistent with Simplicity principle

---

## Research Task 5: File Save/Load Pattern

### Question

How should the TMM handle save/load given it manages multiple initiatives (unlike COER which manages a single initiative)?

### Findings

The COER tool:
- Stores the full loaded JSON in `loadedProjectFile` for round-trip preservation
- On save, deep-clones `loadedProjectFile`, updates `initiatives[0]` and `lastModified`
- Works with a single initiative (`initiatives[0]`)

The TMM tool:
- Manages the entire `initiatives[]` array (each initiative = one dot on the canvas)
- On "Add Initiative", appends a new entry to `initiatives[]` with a generated UUID
- On save, writes back the full `initiatives[]` array

### Decision

Follow the COER round-trip pattern with these adaptations for multi-initiative management:

1. **On load**: Parse the full JSON, store as `loadedProjectFile`. Extract all initiatives; for each, render a dot using `tmm.x`/`tmm.y` (or default to centre if no tmm section).
2. **On save**: Deep-clone `loadedProjectFile`. Replace/merge the `initiatives[]` array — for each initiative, preserve all non-`tmm` sections (coer, sob, etc.) and write the current `tmm` data. Update `lastModified`.
3. **New initiatives** created in TMM get a fresh UUID and `tmm` section; all other tool sections default to `null`.
4. **Initiatives with no tmm section** (created by other tools) are displayed at canvas centre.

### Rationale

- Maintains Constitution Principle VI: round-trip preservation of other tool sections
- Consistent with COER save/load pattern
- TMM can coexist with initiatives created by other tools

### Alternatives Considered

- **Separate TMM-only file**: Rejected — violates shared data format (Constitution Principle VI).
- **Overwrite full initiatives array on save**: Must be done carefully — deep-clone each initiative and only modify the `tmm` section to preserve other tool data.

---

## Summary of Resolved Clarifications

| # | Unknown | Resolution |
|---|---------|-----------|
| 1 | TMM data placement (`id`/`name` in `tmm` section?) | Follow COER pattern: `id`/`name` at initiative level, `tmm` section holds `category`, `details`, `x`, `y` only |
| 2 | Drag-and-drop API choice | Pointer Events API — smooth, unified, supports boundary clamping |
| 3 | Canvas rendering technology | CSS divs with absolute positioning — simplest for ~15 elements |
| 4 | Dot label truncation | CSS `text-overflow: ellipsis` at 30 chars visual; full name in detail panel |
| 5 | Dot overlap handling | Z-index stacking (last-interacted on top); no auto-offset |
| 6 | Multi-initiative save/load | Deep-clone + merge pattern preserving non-TMM sections per initiative |
