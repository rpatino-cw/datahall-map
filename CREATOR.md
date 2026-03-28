---
name: datahall-map-creator
description: Interactive wizard to create and configure data hall layouts for the datahall-map skill. Use when someone says "create a hall", "add a new data hall", "set up my site layout", "configure racks", or needs to define a new hall layout from scratch.
---

# Datahall Map Creator

Interactive wizard that walks users through defining a new data hall layout. Outputs a valid layout entry for `~/.datahall/layouts.json` compatible with the `datahall-map` skill.

## Storage

Layouts are stored in `~/.datahall/layouts.json`. Create the directory if it doesn't exist.

```python
import os, json

LAYOUT_PATH = os.path.expanduser("~/.datahall/layouts.json")

def load_layouts():
    if os.path.exists(LAYOUT_PATH):
        with open(LAYOUT_PATH) as f:
            return json.load(f)
    return {}

def save_layouts(layouts):
    os.makedirs(os.path.dirname(LAYOUT_PATH), exist_ok=True)
    with open(LAYOUT_PATH, "w") as f:
        json.dump(layouts, f, indent=2)
```

---

## Wizard Flow

Walk the user through these steps **one at a time**. Ask each question, wait for the answer, then move on. Show a running summary after each step.

### Step 1: Site Code

Ask: **"What's your site code?"**

Examples: `US-CENTRAL-07A`, `US-VO201`, `US-EAST-13A`, `US-CDZ01`

Format: typically `US-{REGION}-{ID}` or `US-{SITE_CODE}`

### Step 2: Hall Name

Ask: **"Which hall or section?"**

Examples: `DH1`, `DH2`, `DH3`, `SEC1`, `SEC2`

The key will be `{site_code}.{hall_name}` (e.g., `US-CENTRAL-07A.DH2`).

### Step 3: Rack Numbering Style

Ask: **"Does your hall use serpentine numbering? (odd rows count backwards)"**

Explain the difference:

```
Serpentine (most common):
  Row 0:  1  2  3  4  5  6  7  8  9  10   →
  Row 1: 20 19 18 17 16 15 14 13 12  11   ←
  Row 2: 21 22 23 24 25 26 27 28 29  30   →

Sequential:
  Row 0:  1  2  3  4  5  6  7  8  9  10   →
  Row 1: 11 12 13 14 15 16 17 18 19  20   →
  Row 2: 21 22 23 24 25 26 27 28 29  30   →
```

Default: `true` (serpentine) — most CW halls use this.

### Step 4: Default Racks Per Row

Ask: **"How many racks per row? (default: 10)"**

Most CW halls have 10 racks per row. Some smaller sections have 5.

### Step 5: Column Groups

Ask: **"How many column groups does your hall have?"**

Explain:
- **2 columns** — most common. Left and Right sides separated by a central aisle.
- **3 columns** — some sections have A, B, C groups with two aisles.
- **More** — rare but supported.

### Step 6: Define Each Column

For each column, ask:

1. **Label** — "What do you call this group?" (Left, Right, A, B, C, Pod1, etc.)
2. **Starting rack number** — "What's the first rack number in this group?"
3. **Number of rows** — "How many rows of racks?"
4. **Racks per row** — "Same as default ({default_rpr}), or different?"

Show a preview after each column:

```
Column 1 "Left":
  Starts at R1, 12 rows x 10 racks = R1–R120

Column 2 "Right":
  Starts at R121, 14 rows x 10 racks = R121–R260
```

### Step 7: Entrance Location

Ask: **"Where's the main entrance to this hall?"**

Options:
- `bottom-right` (most common — door at the far end, right side)
- `bottom-left`
- `top-right`
- `top-left`

### Step 8: Review & Confirm

Show the complete layout as JSON and a **preview map** before saving:

```json
{
  "racks_per_row": 10,
  "columns": [
    {"label": "Left", "start": 1, "num_rows": 12},
    {"label": "Right", "start": 121, "num_rows": 14}
  ],
  "serpentine": true,
  "entrance": "bottom-right",
  "total_racks": 260
}
```

Then render a blank map using the datahall-map skill format so the user can visually verify:

```
  US-CENTRAL-07A DH2 — Preview

  Left (R1–R120)                              Right (R121–R260)
    1 - - - - - - - - - - 10                    121 - - - - - - - - - - 130
   20 - - - - - - - - - - 11                    140 - - - - - - - - - - 131

   21 - - - - - - - - - - 30                    141 - - - - - - - - - - 150
   ...
```

Ask: **"Does this look right? Save it?"**

### Step 9: Save

Write to `~/.datahall/layouts.json`, merging with any existing layouts.

Confirm: **"Saved {site_code}.{hall_name} to ~/.datahall/layouts.json. You can now use `/datahall-map` to generate maps for this hall."**

---

## Validation Rules

Check these before saving:

1. **No overlapping rack numbers** — each column's range must not overlap with another's
2. **Rack ranges make sense** — `start + (num_rows * racks_per_row) - 1` gives the last rack
3. **Total racks matches** — sum of all columns should equal `total_racks`
4. **No duplicate keys** — warn if `{site_code}.{hall_name}` already exists, offer to overwrite
5. **Rack numbers are positive integers**

```python
def validate_layout(layout):
    errors = []
    cols = layout["columns"]
    default_rpr = layout["racks_per_row"]
    total = 0
    ranges = []

    for col in cols:
        rpr = col.get("racks_per_row", default_rpr)
        count = col["num_rows"] * rpr
        total += count
        start = col["start"]
        end = start + count - 1
        ranges.append((col["label"], start, end))

        if start < 1:
            errors.append(f"{col['label']}: start must be >= 1")
        if col["num_rows"] < 1:
            errors.append(f"{col['label']}: num_rows must be >= 1")

    # Check overlaps
    for i, (label_a, start_a, end_a) in enumerate(ranges):
        for j, (label_b, start_b, end_b) in enumerate(ranges):
            if i < j and start_a <= end_b and start_b <= end_a:
                errors.append(f"Overlap: {label_a} (R{start_a}-R{end_a}) and {label_b} (R{start_b}-R{end_b})")

    if total != layout.get("total_racks", total):
        errors.append(f"Total racks mismatch: columns sum to {total}, but total_racks is {layout['total_racks']}")

    return errors
```

---

## Edit Mode

If the user asks to edit an existing hall layout:

1. Load the current layout from `~/.datahall/layouts.json`
2. Show the current config
3. Ask what they want to change
4. Re-validate after changes
5. Show preview map
6. Save

Trigger phrases: "edit DH2 layout", "change my hall config", "update rack count in DH1"

---

## Quick Create Mode

If the user provides all info at once, skip the wizard and go straight to validation + preview:

"Create a hall: US-WEST-05B.DH1, serpentine, 10/row, Left R1-140 (14 rows), Right R141-280 (14 rows), entrance bottom-right"

Parse the info, build the layout, validate, show preview, save.

---

## Common CW Hall Patterns

Use these as smart defaults when the user isn't sure:

| Pattern | Columns | Rows | RPR | Serpentine | Example |
|---------|---------|------|-----|------------|---------|
| Standard DH | 2 (Left/Right) | 12-28 | 10 | Yes | US-CENTRAL-07A.DH2 |
| Small DH | 2 (Left/Right) | 8 | 10+5 | Yes | US-VO201.DH2 |
| Section | 3 (A/B/C) | 5 | 10 | No | US-CDZ01.SEC1 |
| Large DH | 2 (Left/Right) | 28 | 10 | Yes | US-VO201.DH1 |
