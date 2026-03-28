---
name: datahall-map
description: Generate ASCII data hall floor maps with rack highlights, walking routes, and connection maps. Use when anyone mentions rack map, floor plan, walking route, "where is rack N", data hall layout, connection trace, or needs to visualize rack positions in a data hall.
---

# Datahall Map

Generate ASCII floor maps for CoreWeave data halls. Maps show rack positions with highlights, walking routes from entrance, and connection traces between racks.

## Data Source

Read layout from a `layouts.json` file. Search in order:
1. `~/.datahall/layouts.json` (recommended default)
2. `./dh_layouts.json` (project-local fallback)

If no file exists, run the **Setup Wizard** (see below).

```python
import json, os

LAYOUT_PATHS = [
    os.path.expanduser("~/.datahall/layouts.json"),
    os.path.join(os.getcwd(), "dh_layouts.json"),
]

layouts = None
for p in LAYOUT_PATHS:
    if os.path.exists(p):
        with open(p) as f:
            layouts = json.load(f)
        break

if layouts is None:
    # No layouts found — run setup wizard
    pass
```

### Layout Schema

```json
{
  "racks_per_row": 10,
  "columns": [
    {"label": "Left", "start": 1, "num_rows": 12},
    {"label": "Right", "start": 121, "num_rows": 14, "racks_per_row": 10}
  ],
  "serpentine": true,
  "entrance": "bottom-right",
  "total_racks": 260
}
```

| Field | Type | Description |
|-------|------|-------------|
| `racks_per_row` | int | Default rack count per row (can be overridden per-column) |
| `columns` | array | Column definitions, rendered side-by-side with corridor gaps |
| `columns[].label` | string | Display name ("Left", "Right", "A", "B", etc.) |
| `columns[].start` | int | First rack number in this column |
| `columns[].num_rows` | int | How many rows this column has |
| `columns[].racks_per_row` | int? | Override default racks_per_row for this column |
| `serpentine` | bool | If true, odd rows reverse rack numbering direction |
| `entrance` | string | Where the door is: "bottom-right", "bottom-left", "top-right", "top-left" |
| `total_racks` | int | Total rack count (informational) |

Keys follow the pattern `SITE-CODE.HALL` (e.g., `US-CENTRAL-07A.DH2`, `US-CDZ01.SEC1`).

## Setup Wizard

When no layouts.json exists or the user asks to add a new hall, walk them through these questions:

1. **Site code** — e.g., `US-CENTRAL-07A`, `US-VO201`, `US-EAST-13A` (or any name the user wants)
2. **Hall name** — e.g., `DH1`, `DH2`, `SEC1`
3. **How many column groups?** — Most halls have 2 (Left/Right), some have 3 (A/B/C)
4. **For each column:**
   - Label (Left, Right, A, B, C, etc.)
   - Starting rack number
   - Number of rows
   - Racks per row (if different from default)
5. **Serpentine numbering?** — "Do odd rows count backwards?" (most DH halls = yes, some sections = no)
6. **Entrance location** — Which corner is the door?
7. **Total racks** — Sum of all columns

Then write to `~/.datahall/layouts.json` (create directory if needed):

```python
import os, json

path = os.path.expanduser("~/.datahall/layouts.json")
os.makedirs(os.path.dirname(path), exist_ok=True)

existing = {}
if os.path.exists(path):
    with open(path) as f:
        existing = json.load(f)

key = f"{site_code}.{hall_name}"
existing[key] = new_layout

with open(path, "w") as f:
    json.dump(existing, f, indent=2)
```

---

## Modes

### Mode 1: Highlight Map

Show one or more racks highlighted on the floor plan.

**Trigger phrases:** "show R145", "where is rack 85", "find R200 in DH2"

Single character per rack. Rack numbers on **both sides** of each row. Blank line between every 2 rows (walking aisle).

```
  US-CENTRAL-07A DH2 — Rack R145

  Left (R1–R120)                              Right (R121–R260)
    1 - - - - - - - - - - 10                    121 - - - - - - - - - - 130
   20 - - - - - - - - - - 11                    140 - - - - - - - - - - 131

   21 - - - - - - - - - - 30                    141 - - - - # - - - - - 150
   40 - - - - - - - - - - 31                    160 - - - - - - - - - - 151

  # = R145 (RIGHT column)
```

### Mode 2: Walking Route

Show the shortest path from the entrance to a target rack.

**Trigger phrases:** "walk to R85", "route to rack 200", "how do I get to R45 in DH1"

Box-drawing characters trace the path through the corridor between columns:

| Char | Use |
|------|-----|
| `\|` | Vertical path in corridor |
| `+` | Turn from corridor into target row |
| `=` | Horizontal path / entrance line |

Route starts at entrance, goes up/down the corridor, then turns into the target column row. The entrance line spans the bottom of the map.

```
  US-CENTRAL-07A DH2 — Rack R25

  Left (R1–R120)                              Right (R121–R260)
    1 - - - - - - - - - - 10        |          121 - - - - - - - - - - 130
   20 - - - - - - - - - - 11        |          140 - - - - - - - - - - 131

   21 - - - - # - - - - - 30   ====+           141 - - - - - - - - - - 150
   40 - - - - - - - - - - 31        |          160 - - - - - - - - - - 151

   41 - - - - - - - - - - 50        |          161 - - - - - - - - - - 170
   60 - - - - - - - - - - 51        |          180 - - - - - - - - - - 171
                                     =====================================

  # = R25 (LEFT column)  === walking route
```

### Mode 3: Connection Map

Show two related racks (source + peer) highlighted in different colors. Used for tracing cable connections, IB links, etc.

**Trigger phrases:** "connection map R45 and R200", "show link between rack 10 and rack 155", "trace R80 to R220"

```
  Connection Map  US-CENTRAL-07A DH2
  @ = Source R45 (Left)
  # = Peer R200 (Right)

  Left (R1–R120)                              Right (R121–R260)
    1 - - - - - - - - - - 10                    121 - - - - - - - - - - 130
   20 - - - - - - - - - - 11                    140 - - - - - - - - - - 131

   21 - - - - - - - - - - 30                    141 - - - - - - - - - - 150
   40 - - - - @ - - - - - 31                    160 - - - - - - - - - - 151

   41 - - - - - - - - - - 50                    161 - - - - - - - - - - 170
   60 - - - - - - - - - - 51                    180 - - - - - - - - - - 171

   61 - - - - - - - - - - 70                    191 - - - - - - - - # - 200
```

Multiple peers supported — pass a set of rack numbers to highlight.

---

## Rack Symbols (All Modes)

| Symbol | Meaning |
|--------|---------|
| `@` | Cyan bold — primary / source rack |
| `#` | Yellow bold — secondary / target / peer rack |
| `!` | Red bold — error / wrong connection |
| `-` | Dim — normal rack |

---

## Core Algorithms

### Serpentine Rack Numbering

```python
def rack_at(col_start, row, pos, racks_per_row, serpentine):
    """Return the rack number at a given row and position within a column."""
    if serpentine and row % 2 == 1:
        return col_start + (row + 1) * racks_per_row - 1 - pos
    return col_start + row * racks_per_row + pos
```

Row 0: left-to-right (1, 2, 3... 10)
Row 1: right-to-left (20, 19, 18... 11)
Row 2: left-to-right (21, 22, 23... 30)

### Reverse Lookup (Rack -> Position)

```python
def find_rack_position(rack_num, columns, default_racks_per_row, serpentine):
    """Find which column, row, and position a rack number falls in."""
    for col_idx, col in enumerate(columns):
        rpr = col.get("racks_per_row", default_racks_per_row)
        col_end = col["start"] + col["num_rows"] * rpr - 1
        if col["start"] <= rack_num <= col_end:
            offset = rack_num - col["start"]
            row = offset // rpr
            pos = offset % rpr
            if serpentine and row % 2 == 1:
                pos = rpr - 1 - pos
            return {"col_idx": col_idx, "col_label": col["label"], "row": row, "pos": pos}
    return None
```

### N-Column Side-by-Side Rendering

Supports any number of columns (2, 3, or more). Each column rendered at its own width, joined by `COL_GAP`.

```python
COL_GAP = "       "  # 7-space corridor between columns

def col_width(racks_per_row):
    """Visual width of one column: left_label + cells + right_label."""
    return 4 + (racks_per_row * 2 - 1) + 4

def build_row(col, row, default_rpr, serpentine, highlights={}):
    """Build one row string for a single column."""
    rpr = col.get("racks_per_row", default_rpr)
    chars = []
    for pos in range(rpr):
        rn = rack_at(col["start"], row, pos, rpr, serpentine)
        symbol = highlights.get(rn, "-")
        chars.append(symbol)
    first = rack_at(col["start"], row, 0, rpr, serpentine)
    last = rack_at(col["start"], row, rpr - 1, rpr, serpentine)
    return f"{first:>3} " + " ".join(chars) + f" {last:<3}"

# Render all columns side by side
max_rows = max(c["num_rows"] for c in columns)
for row in range(max_rows):
    parts = []
    for col in columns:
        if row < col["num_rows"]:
            parts.append(build_row(col, row, default_rpr, serpentine, highlights))
        else:
            parts.append(" " * col_width(col.get("racks_per_row", default_rpr)))
    print("  " + COL_GAP.join(parts))

    # Aisle gap every 2 rows
    if row % 2 == 1 and row < max_rows - 1:
        print()
```

### Walking Route Logic

The route runs through the corridor gap between columns. For N columns, the route uses the gap nearest to the target column.

```python
# 1. Find target column index and row
target_info = find_rack_position(target_rack, columns, default_rpr, serpentine)
target_col_idx = target_info["col_idx"]
target_row = target_info["row"]

# 2. Determine which corridor gap to use
# Gap i is between column i and column i+1
if target_col_idx < len(columns) - 1:
    route_gap = target_col_idx      # gap to the right of target column
else:
    route_gap = len(columns) - 2    # rightmost gap

# 3. Draw vertical path from entrance to target row
# 4. Draw turn marker at target row
# 5. Draw entrance line at bottom
```

---

## Usage Examples

Users will say things like:

| Prompt | Mode |
|--------|------|
| "show R145 in DH2" | Highlight |
| "where is rack 85" | Highlight |
| "walk to R200 in DH1" | Walking route |
| "route to rack 45" | Walking route |
| "connection map R45 and R200 in DH2" | Connection |
| "show link between rack 10 and 155" | Connection |
| "add a new hall" / "set up my site" | Setup wizard |
| "show all halls" | List available layouts |

When a user asks for a map without specifying a hall, and multiple halls exist, ask which one. If only one hall is configured, use it automatically.

---

## Notes for Contributors

- Layouts are per-site, per-hall. The JSON key format is `SITE-CODE.HALL` (e.g., `US-VO201.DH1`)
- Serpentine numbering is the most common pattern in CW data halls but not universal — some sections use sequential
- Some halls have columns with different racks-per-row (e.g., left side has 10, right side has 5) — always respect per-column overrides
- The corridor gap width (7 spaces) is fixed — it represents the physical hot/cold aisle between column groups
- See `sample-layouts.json` in this directory for example configurations
