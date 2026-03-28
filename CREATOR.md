---
name: datahall-map-creator
description: Interactive wizard to create and configure data hall rack layouts. Outputs valid JSON for ~/.datahall/layouts.json compatible with the datahall-map skill. Trigger phrases - "create a hall", "add a new data hall", "set up my site layout", "configure racks", "define a new hall layout".
---

# Datahall Map Creator

Interactive wizard that walks users through defining a new data hall layout. Produces a validated layout entry for `~/.datahall/layouts.json`, compatible with the [datahall-map](https://github.com/rpatino-cw/datahall-map) skill.

---

## Storage

Layouts persist at `~/.datahall/layouts.json`. Create the directory on first write.

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

## Layout JSON Schema

Each layout is keyed as `SITE-CODE.HALL` (e.g., `US-EAST-07A.DH2`).

```json
{
  "US-EAST-07A.DH2": {
    "racks_per_row": 10,
    "columns": [
      { "label": "Left", "start": 1, "num_rows": 12 },
      { "label": "Right", "start": 121, "num_rows": 14, "racks_per_row": 10 }
    ],
    "serpentine": true,
    "entrance": "bottom-right",
    "total_racks": 260
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `racks_per_row` | int | Yes | Default racks per row (overridable per column) |
| `columns` | array | Yes | Column groups, rendered side-by-side |
| `columns[].label` | string | Yes | Display name: "Left", "Right", "A", "B", etc. |
| `columns[].start` | int | Yes | First rack number in this column |
| `columns[].num_rows` | int | Yes | Row count for this column |
| `columns[].racks_per_row` | int | No | Override the default for this column only |
| `serpentine` | bool | Yes | Odd rows reverse direction if true |
| `entrance` | string | Yes | Door location: `bottom-right`, `bottom-left`, `top-right`, `top-left`, `none` |
| `total_racks` | int | Yes | Sum of all column racks (validated before save) |

---

## Wizard Flow

Walk the user through steps **one at a time**. Ask, wait for the answer, then advance. Show a running summary after every step.

**UX rules:**
- Present **numbered options** wherever possible. Only use free-text input for site codes, hall names, column labels, and starting rack numbers.
- Keep prompts short. One question per step.
- Accept bare numbers (e.g., `1`) or the option text (e.g., `serpentine`).

### Visual Feedback

**After every step**, render a progress bar and running summary:

```
Step 3/10  ██████░░░░░░░░░░░░  30%  Rack Numbering
┌─────────────────────────────────┐
│  Site: US-EAST-07A              │
│  Hall: DH2                      │
│  Numbering: Serpentine          │
└─────────────────────────────────┘
```

Step labels: `Site Code` > `Hall Name` > `Numbering` > `Racks/Row` > `Columns` > `Column Defs` > `Total Racks` > `Entrance` > `Review` > `Save`

Progress bar: 18 chars. `█` = completed, `░` = remaining. Show percentage.

**Live ASCII preview** starts at Step 6. Render a growing mini-map as columns are defined. Use `[ Column N: ? ]` as a placeholder for columns not yet entered. After Step 7 (entrance), add the entrance marker.

#### Alignment Rules (mandatory for all ASCII previews)

1. **Right-justify rack numbers** to a consistent width within each column (pad with spaces: `  1`, ` 20`, `140`)
2. **Fixed dash width** per column. Every row in a column has the same character width: left number + ` - ` repeated + right number
3. **Fixed column gap** of 20 spaces between the last character of one column and the first of the next
4. **Unequal heights** are bottom-aligned. Shorter columns get blank padding at the top so taller columns extend below
5. **Row pairs** (forward + reverse in serpentine) are grouped together, separated from the next pair by one blank line
6. **Number width consistency** within a column. If any rack number has 3 digits, all numbers in that column use 3-char width

#### Entrance Marker Rules

The entrance marker must reflect **which wall** the entrance is on, with clear spacing from the rack data:

- **Right wall** (`bottom-right`, `top-right`): marker goes to the RIGHT of the rightmost column, with 6 spaces of clearance. Use `◀ ENTRANCE` (pointing left toward the wall) or `ENTRANCE ▶` (pointing right toward the door). Arrow points **toward the wall/door**.

  ```
  301 - - - - - - - - - 310
  320 - - - - - - - - - 311      ◀ ENTRANCE
  ```

- **Left wall** (`bottom-left`, `top-left`): marker goes to the LEFT of the leftmost column, with 6 spaces of clearance. Use `ENTRANCE ▶` (pointing right toward the racks).

  ```
                ENTRANCE ▶        1 - - - - - - - - -  10
                                 20 - - - - - - - - -  11
  ```

- **Top entrances** (`top-right`, `top-left`): marker appears ABOVE the first row of the relevant column.
- **Bottom entrances** (`bottom-right`, `bottom-left`): marker appears BELOW the last row of the relevant column.
- **`none`**: no marker rendered.

The marker must **never overlap** with rack data. Always maintain at least 6 spaces between the last rack number and the entrance label.

Example (one column defined, one pending):

```
  US-EAST-07A DH2 — Building...

  Left (R1-R120)                       [ Column 2: ? ]
    1 - - - - - - - - - - 10                ...
   20 - - - - - - - - - - 11                ...
                  :
  111 - - - - - - - - - - 120               ...
```

---

### Step 1: Site Code

> **What's your site code?**
>
> This identifies the physical location. Any name works.
> Examples: `US-EAST-07A`, `US-VO201`, `EU-WEST-03`, `LAB-01`

### Step 2: Hall Name

> **Which hall or section?**
>
> Examples: `DH1`, `DH2`, `SEC1`, `HALL-A`

The layout key becomes `{site_code}.{hall_name}` (e.g., `US-EAST-07A.DH2`).

### Step 3: Rack Numbering Style

> **How are racks numbered?**
>
> ```
> 1) Serpentine (most common) — alternating rows reverse direction
>    Row 0:  1  2  3  4  5  6  7  8  9  10  ->
>    Row 1: 20 19 18 17 16 15 14 13 12  11  <-
>    Row 2: 21 22 23 24 25 26 27 28 29  30  ->
>
> 2) Sequential — all rows go the same direction
>    Row 0:  1  2  3  4  5  6  7  8  9  10  ->
>    Row 1: 11 12 13 14 15 16 17 18 19  20  ->
>    Row 2: 21 22 23 24 25 26 27 28 29  30  ->
> ```
>
> Default: **1** (serpentine)

### Step 4: Default Racks Per Row

> **How many racks per row?**
>
> ```
> 1) 10 racks per row (most common)
> 2) 5 racks per row (smaller sections)
> 3) Other (type a number)
> ```

### Step 5: Column Groups

> **How many column groups does the hall have?**
>
> ```
> 1) 2 columns — Left and Right sides separated by a central aisle
> 2) 3 columns — A, B, C groups with two aisles
> 3) More (type a number)
> ```

### Step 6: Define Each Column

For each column, collect:

1. **Label** — "What do you call this group?" (Left, Right, A, B, Pod1, etc.)
2. **Starting rack number** — "What's the first (lowest) rack number?"
3. **Number of rows** — "How many rows of racks?"
4. **Racks per row** — "Same as default ({default_rpr}), or different?"

After each column, show a summary and the live ASCII preview:

```
Column 1 "Left":
  Starts at R1, 12 rows x 10 racks = R1-R120
```

Then render the growing map with defined columns filled in and placeholders for the rest.

### Step 7: Total Racks

> **How many total racks are in this hall?**
>
> This is used as a cross-check against the column definitions. The wizard calculates the expected total from the columns you defined and compares it.

Show the calculated total and ask the user to confirm or correct:

```
Based on your columns:
  Left:  14 rows x 10 = 140 racks (R1–R140)
  Right: 18 rows x 10 = 180 racks (R141–R320)
  Calculated total: 320

Is 320 the correct total rack count?
1) Yes, 320 is correct
2) No, the total is different (type the correct number)
```

If the user provides a different number, identify the discrepancy and offer to adjust column row counts to match. This catches errors early before the review step.

### Step 8: Entrance Location

> **Where's the main entrance?**
>
> ```
> 1) Bottom-right (most common)
> 2) Bottom-left
> 3) Top-right
> 4) Top-left
> 5) Multiple entrances / Skip
> ```

Option 5 sets `"entrance": "none"` and omits the entrance marker from previews.

### Step 9: Review & Confirm

Render the full preview map with entrance marker per Entrance Marker Rules, then ask:

```
  US-EAST-07A DH2 — Preview

  Left (R1-R120)                              Right (R121-R260)
    1 - - - - - - - - - - 10                    121 - - - - - - - - - - 130
   20 - - - - - - - - - - 11                    140 - - - - - - - - - - 131

   21 - - - - - - - - - - 30                    141 - - - - - - - - - - 150
   ...
  111 - - - - - - - - - - 120                    251 - - - - - - - - - - 260      ◀ ENTRANCE
```

> **Does this look right?**
>
> ```
> 1) Save
> 2) Edit a field
> 3) Show raw JSON
> 4) Fix alignment
> 5) Start over
> ```

#### Option 4: Fix Alignment

Self-correction workflow:

1. **Audit** the rendered map character-by-character against the Alignment Rules:
   - Rack number widths consistent?
   - Dash sequences uniform per column?
   - Column gaps fixed at 20 spaces on every line?
   - Row pairs vertically aligned?
   - Headers aligned with data?

2. **Report** findings:
   ```
   Alignment check:
   ok  Dash widths consistent
   fix Left column numbers: rows 1-9 use 1-char, should be 3-char
   fix Column gap varies 18-22 spaces, should be fixed at 20
   ```

3. **Re-render** the entire map with all rules enforced.

4. **Confirm**: "Better? (1) Save, (2) Still off — describe what's wrong"

   If (2), the user describes the issue and Claude makes a targeted fix, then re-shows.

### Step 10: Save

Merge into `~/.datahall/layouts.json` and confirm:

> **Saved `{site_code}.{hall_name}` to `~/.datahall/layouts.json`.**
> Use `/datahall-map` to generate maps for this hall.

---

## Validation

Run all checks before saving. Block save on any error.

1. **No overlapping rack ranges** between columns
2. **Ranges are contiguous** within each column: `start + (num_rows * racks_per_row) - 1` = last rack
3. **Total matches** the sum across all columns
4. **No duplicate keys** — warn and offer overwrite if `{site_code}.{hall_name}` exists
5. **Positive integers** for all rack numbers, row counts, and racks-per-row

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

    for i, (label_a, start_a, end_a) in enumerate(ranges):
        for j, (label_b, start_b, end_b) in enumerate(ranges):
            if i < j and start_a <= end_b and start_b <= end_a:
                errors.append(
                    f"Overlap: {label_a} (R{start_a}-R{end_a}) "
                    f"and {label_b} (R{start_b}-R{end_b})"
                )

    if total != layout.get("total_racks", total):
        errors.append(
            f"Total mismatch: columns sum to {total}, "
            f"but total_racks is {layout['total_racks']}"
        )

    return errors
```

### Validation Failure Recovery

On error, show the specific problem and offer targeted recovery — not a full restart:

```
Validation error: Overlap — Left (R1-R120) and Right (R100-R220)

1) Re-enter column "Right"
2) Re-enter all columns
3) Cancel
```

---

## Edit Mode

When the user asks to modify an existing layout:

1. Load from `~/.datahall/layouts.json`
2. Render the current layout as an ASCII map
3. Offer numbered fields:

```
What would you like to change?
1) Site code
2) Hall name
3) Add a column
4) Edit a column
5) Remove a column
6) Entrance location
7) Delete this hall
```

4. Re-validate, preview, save.

Trigger phrases: "edit DH2 layout", "change my hall config", "update rack count", "modify the left column"

---

## Quick Create Mode

When the user provides all details in one message, skip the wizard:

> "Create a hall: US-WEST-05B.DH1, serpentine, 10/row, Left R1-140 (14 rows), Right R141-280 (14 rows), entrance bottom-right"

Parse, validate, render preview, save.

---

## Common Hall Patterns

Use these as smart defaults when the user isn't sure. These represent typical configurations across data center environments:

| Pattern | Columns | Rows | RPR | Serpentine | Example |
|---------|---------|------|-----|------------|---------|
| Standard | 2 (Left/Right) | 12-28 | 10 | Yes | `SITE.DH2` |
| Small | 2 (Left/Right) | 6-10 | 10 or 5 | Yes | `SITE.DH3` |
| Sectioned | 3+ (A/B/C) | 4-8 | 10 | No | `SITE.SEC1` |
| Large | 2 (Left/Right) | 24-32 | 10 | Yes | `SITE.DH1` |
| Mixed RPR | 2 (Left/Right) | varies | 10+5 | Yes | `SITE.DH2` |

---

## List Mode

When the user asks "show my halls", "list layouts", or "what halls do I have":

1. Load `~/.datahall/layouts.json`
2. Show a summary table:

```
Configured halls:

  Site                    Columns   Racks   Serpentine
  US-EAST-07A.DH1        2         280     Yes
  US-EAST-07A.DH2        2         260     Yes
  US-WEST-05B.SEC1       3         150     No
```

3. Offer: "Pick a hall to view its map, or create a new one."
