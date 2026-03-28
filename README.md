<div align="center">
  <img src="assets/banner.png" alt="datahall-map" width="600">
  <p>ASCII floor maps for CoreWeave data halls. A <a href="https://docs.anthropic.com/en/docs/claude-code">Claude Code</a> skill.</p>

  [![License](https://img.shields.io/github/license/rpatino-cw/datahall-map?style=flat-square)](LICENSE)
  [![Issues](https://img.shields.io/github/issues/rpatino-cw/datahall-map?style=flat-square)](https://github.com/rpatino-cw/datahall-map/issues)
</div>

---

## What it does

```
  US-CENTRAL-07A DH2 — Rack R145

  Left (R1–R120)                              Right (R121–R260)
    1 - - - - - - - - - - 10                    121 - - - - - - - - - - 130
   20 - - - - - - - - - - 11                    140 - - - - - - - - - - 131

   21 - - - - - - - - - - 30                    141 - - - - # - - - - - 150
   40 - - - - - - - - - - 31                    160 - - - - - - - - - - 151

  # = R145 (RIGHT column)
```

Four modes: **highlight** a rack, **walk** to it, **trace** a connection between two racks, or **heatmap** issue density.

---

## Install

```bash
git clone https://github.com/rpatino-cw/datahall-map.git ~/.claude/skills/datahall-map
```

Then either edit `sample-layouts.json` with your site's halls, or let Claude walk you through setup on first use.

---

## Usage

| Say this | Get this |
|---|---|
| `show R145 in DH2` | Rack highlighted on floor map |
| `walk to R85` | Walking route from entrance |
| `trace R45 to R200` | Both racks highlighted (connection map) |
| `heatmap DH2` | Issue density visualization |
| `add a new hall` | Interactive setup wizard |

---

## Layout config

One JSON file at `~/.datahall/layouts.json`:

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

See [`sample-layouts.json`](sample-layouts.json) for examples.

---

## Files

```
SKILL.md              ← Map rendering skill (Claude reads this)
CREATOR.md            ← Layout creation wizard
sample-layouts.json   ← Example hall configs
```

---

## Contributing

1. Fork → add your site to `sample-layouts.json` → PR.

No hostnames, IPs, or serial numbers. Rack counts and numbering patterns only.

---

## License

[MIT](LICENSE)
