# Quickstart: TMM — Time Management Matrix

**Feature**: [data-model.md](data-model.md)

## 1. Open the Tool

```text
Open public/tools/tmm/index.html in any modern browser (Chrome, Firefox, Safari, Edge).
Double-click the file, or drag it into a browser window.
No build step. No installation. No server.
```

## 2. Place an Initiative

1. Type an initiative name in the **Initiative** field.
2. Click **Place on canvas** (or press Enter).
3. Click anywhere inside the matrix to drop the dot.
4. The dot appears at the clicked position; the initiative is listed below the canvas.

## 3. Coordinate Reference

The canvas uses normalised `[0.0, 1.0]` coordinates:

| Axis | 0.0 | 0.5 | 1.0 |
|------|-----|-----|-----|
| **X** (horizontal) | Left edge — most urgent | Centre | Right edge — least urgent |
| **Y** (vertical) | Top edge — highest impact | Centre | Bottom edge — lowest impact |

A dot placed at `(x: 0.5, y: 0.5)` appears exactly at the canvas centre,
equidistant from all four quadrant borders.

### Quadrant boundaries

| Quadrant | X range | Y range | Label |
|----------|---------|---------|-------|
| Q1 Firefighting | `x < 0.5` | `y < 0.5` | Top-left — Urgent &amp; Important |
| Q2 Growth | `x ≥ 0.5` | `y < 0.5` | Top-right — Not Urgent but Important |
| Q3 Distraction | `x < 0.5` | `y ≥ 0.5` | Bottom-left — Urgent but Not Important |
| Q4 Waste | `x ≥ 0.5` | `y ≥ 0.5` | Bottom-right — Not Urgent &amp; Not Important |

## 4. Remove an Initiative

- Click any dot on the canvas to remove it, **or**
- Click the **×** button in the Placed Initiatives list below the canvas.

## 5. Save to Project File

- Click **Save** in the toolbar.
- The browser downloads a `.json` project file.
- The file uses the shared `effectivenessToolkit` envelope defined in
  [data-model.md](data-model.md).

## 6. Load a Project File

- Click **Load** in the toolbar.
- Select a previously saved `.json` file.
- All TMM placements are restored. Data from other tools (COER, Memory Map, etc.)
  in the same file is preserved unchanged.

## 7. Clear the Canvas

- Click **Clear All** to remove all placed initiatives.
- A confirmation prompt prevents accidental loss.

## 8. Validation Checks

- [x] Empty canvas loads with correct quadrant layout (Q1 top-left, Q2 top-right, Q3 bottom-left, Q4 Waste bottom-right).
- [x] "More Urgent" label appears under the left column; "Less Urgent" under the right column.
- [x] "More Impact" labels the upper half of the vertical axis; "Less Impact" labels the lower half.
- [x] Clicking at canvas centre places dot at `(x: 0.5, y: 0.5)` — equally spaced from all quadrant borders.
- [ ] Save → Load round-trips all x/y coordinates without drift.
- [ ] Multiple initiatives placed across all four quadrants are correctly listed with their quadrant name.
