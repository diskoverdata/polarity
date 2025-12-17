# Polarity

Build an interactive simulation of a two-faction grid battle.

## The Grid

The simulation operates on a 2D grid. Each cell exists in one of three states:

- **Empty** (no charge)
- **Positive (+)**
- **Negative (−)**

## Rules

All transitions happen simultaneously—calculate the next state for every cell based on the current state, then apply all changes at once.

Use **Moore neighborhood**: each cell considers all 8 surrounding cells (including diagonals).

### Birth

An empty cell may become charged based on its neighbors:

- Becomes **+** if: exactly 2 or 3 positive neighbors AND at most 1 negative neighbor
- Becomes **−** if: exactly 2 or 3 negative neighbors AND at most 1 positive neighbor
- Stays **empty** if both conditions are met (contested) or neither is met

### Survival

A charged cell counts its **allied** neighbors (same charge) and **enemy** neighbors (opposite charge):

- **Dies** if:
  - Allied neighbors < 2 (isolation)
  - Allied neighbors > 3 (overcrowding)
  - Enemy neighbors > allied neighbors (overwhelmed)
- **Survives** if:
  - Allied neighbors is 2 or 3, AND
  - Enemy neighbors ≤ allied neighbors
- A surviving cell where enemy = allied is under **pressure** (contested survival)

### Edge Behavior

Cells at grid edges have fewer neighbors. Handle edges however you prefer (neighbors outside bounds count as empty, or wrap toroidally—your choice, just be consistent).

## Requirements

### Core (complete these)

1. Render a grid (minimum 20×20, size is your choice)
2. Place + and − cells via mouse interaction (click, drag, or however you design it)
3. Step forward one generation manually (button or keyboard)
4. Auto-play with start/stop toggle
5. Clear/reset the grid

### Stretch (time permitting, in priority order)

1. Generation counter
2. Faction census (live count of + vs − cells)
3. Visual indicator for cells under pressure (contested survival)
4. Preset patterns (load a known seed)
5. Adjustable speed
6. Headless mode (see below)

## Test Patterns

Use these to validate your implementation:

### 1. Stable Block
A 2×2 square of the same charge persists indefinitely.
```
+ +
+ +
```
Each cell has 3 allied neighbors, 0 enemies. All survive. No empty cells have enough neighbors to birth.

### 2. Isolated Cell
A single + or − with no neighbors dies immediately (0 allied < 2).

### 3. Line of Three
```
+ + +
```
- Center cell: 2 allied, 0 enemy → survives
- End cells: 1 allied, 0 enemy → die (isolation)
- After generation 1: single cell remains
- After generation 2: that cell dies

Result: collapses completely in 2 generations.

### 4. Overwhelmed Cell
A + cell with 2 allied and 3 enemy neighbors:
```
− − −
− + +
  +
```
The top-right + has 2 allied (below, below-left) and 3 enemy (left, above-left, above). Since enemy (3) > allied (2), it dies.

### 5. Contested Birth
An empty cell with exactly 2 + neighbors and exactly 2 − neighbors:
```
+ . −
. ? .
+ . −
```
The center cell (?) qualifies for both + birth (2 positive, 0 negative on some neighbor counts—adjust positions as needed to create true contest). When both birth conditions are met, the cell stays empty.

## What We're Looking For

- Working software that demonstrates the core rules
- Clear thinking about problem decomposition
- Effective use of AI tools - we're observing your process
- Reasonable decisions under ambiguity

## What We're NOT Testing

- Memorized algorithms
- Pixel-perfect design
- Comprehensive error handling
- Test coverage

---

## Headless Mode (Stretch Goal)

If you have time, add a `--headless` flag that enables automated testing via stdin/stdout JSON.

### Input (stdin)

```json
{
  "gridSize": { "width": 10, "height": 10 },
  "initialState": [
    { "x": 4, "y": 4, "charge": "+" },
    { "x": 5, "y": 4, "charge": "+" }
  ],
  "generations": 5
}
```

### Output (stdout)

```json
{
  "finalState": [
    { "x": 4, "y": 4, "charge": "+" },
    { "x": 5, "y": 4, "charge": "+" }
  ]
}
```

### Protocol Rules

| Field | Type | Description |
|-------|------|-------------|
| `gridSize.width` | integer | Grid width |
| `gridSize.height` | integer | Grid height |
| `initialState` | array | Starting cells (empty cells omitted) |
| `generations` | integer | Number of generations to simulate |
| `finalState` | array | Resulting cells after simulation |

**Cell format:** `{ "x": <int>, "y": <int>, "charge": "+" | "-" }`

- Coordinates are 0-indexed (origin top-left, x increases right, y increases down)
- `charge` must be exactly `"+"` or `"-"` (strings)
- Empty cells are omitted from arrays (don't include nulls)
- No duplicate coordinates allowed
- Exit with code 0 on success
