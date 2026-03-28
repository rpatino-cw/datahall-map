# datahall-map

A Claude Code skill that generates ASCII data hall floor maps for CoreWeave sites. Visualize rack positions, trace walking routes, highlight connections between racks, and display issue heatmaps — all from your terminal.

## Install

Copy this folder into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/datahall-map
cp SKILL.md README.md sample-layouts.json ~/.claude/skills/datahall-map/
```

Or clone directly:

```bash
git clone https://github.com/YOUR_USERNAME/datahall-map.git ~/.claude/skills/datahall-map
```

## First-Time Setup

The skill stores hall layouts in `~/.datahall/layouts.json`. On first use, Claude will walk you through a setup wizard to define your site's halls.

Or copy the sample and edit:

```bash
mkdir -p ~/.datahall
cp ~/.claude/skills/datahall-map/sample-layouts.json ~/.datahall/layouts.json
```

Then edit `~/.datahall/layouts.json` to match your site. Each key is `SITE-CODE.HALL`:

```json
{
  "US-YOUR-SITE.DH1": {
    "racks_per_row": 10,
    "columns": [
      {"label": "Left", "start": 1, "num_rows": 14},
      {"label": "Right", "start": 141, "num_rows": 17}
    ],
    "serpentine": true,
    "entrance": "bottom-right",
    "total_racks": 310
  }
}
```

## Usage

Once installed, use `/datahall-map` in Claude Code or just ask naturally:

| What you say | What you get |
|---|---|
| "show R145 in DH2" | Highlight map with R145 marked |
| "walk to R85 in DH1" | Floor map with walking route from entrance |
| "connection map R45 and R200" | Two racks highlighted in different colors |
| "heatmap DH2" + issue data | Density visualization of problem racks |
| "add a new hall" | Setup wizard for a new layout |
| "show all halls" | List configured layouts |

### Example Output

```
  US-YOUR-SITE DH2 — Rack R145

  Left (R1-R120)                              Right (R121-R260)
    1 - - - - - - - - - - 10                    121 - - - - - - - - - - 130
   20 - - - - - - - - - - 11                    140 - - - - - - - - - - 131

   21 - - - - - - - - - - 30                    141 - - - - # - - - - - 150
   40 - - - - - - - - - - 31                    160 - - - - - - - - - - 151

  # = R145 (RIGHT column)
```

## Layout Schema

| Field | Type | Description |
|---|---|---|
| `racks_per_row` | int | Default rack count per row |
| `columns` | array | Column groups rendered side-by-side |
| `columns[].label` | string | Display name ("Left", "Right", "A", etc.) |
| `columns[].start` | int | First rack number in column |
| `columns[].num_rows` | int | Number of rows in column |
| `columns[].racks_per_row` | int? | Override default for this column |
| `serpentine` | bool | Odd rows count backwards |
| `entrance` | string | Door location: "bottom-right", "bottom-left", etc. |
| `total_racks` | int | Total rack count (informational) |

### Serpentine vs Sequential

Most CW data halls use **serpentine** numbering — odd rows count backwards so rack numbers snake through the floor:

```
Row 0:  1  2  3  4  5  6  7  8  9  10   (left to right)
Row 1: 20 19 18 17 16 15 14 13 12  11   (right to left)
Row 2: 21 22 23 24 25 26 27 28 29  30   (left to right)
```

Some sections (like `SEC1` halls) use **sequential** numbering where every row counts left to right.

## Contributing

To add your site's layout:

1. Fork this repo
2. Add your site to `sample-layouts.json`
3. Open a PR with the site code and hall name

Keep sensitive information out — rack counts and numbering patterns are fine, but don't include hostnames, IPs, or device serial numbers.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or desktop app
- No additional dependencies — the skill instructs Claude to generate maps using Python or plain text
