<a id="readme-top"></a>

<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <h1>datahall-map</h1>
  <p>
    ASCII data hall floor maps for CoreWeave sites — built as a <a href="https://docs.anthropic.com/en/docs/claude-code">Claude Code</a> skill.
  </p>
  <p>
    Visualize rack positions, trace walking routes, highlight connections, and display issue heatmaps — all from your terminal.
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#about">About</a></li>
    <li><a href="#example-output">Example Output</a></li>
    <li><a href="#getting-started">Getting Started</a></li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#layout-schema">Layout Schema</a></li>
    <li><a href="#modes">Modes</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

---

## About

**datahall-map** is a Claude Code skill that generates ASCII floor maps for CoreWeave data halls. Point it at any site's rack layout and it renders:

- **Highlight maps** — find any rack on the floor instantly
- **Walking routes** — entrance-to-rack path through the corridor
- **Connection maps** — visualize both ends of a cable/link
- **Heatmaps** — issue density across the hall

It works for any CW site. Define your hall layout once, and Claude handles the rest.

### Built With

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — Anthropic's CLI for Claude
- Python (generated on-the-fly by Claude — no install needed)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## Example Output

```
  US-YOUR-SITE DH2 — Rack R145

  Left (R1-R120)                              Right (R121-R260)
    1 - - - - - - - - - - 10                    121 - - - - - - - - - - 130
   20 - - - - - - - - - - 11                    140 - - - - - - - - - - 131

   21 - - - - - - - - - - 30                    141 - - - - # - - - - - 150
   40 - - - - - - - - - - 31                    160 - - - - - - - - - - 151

  # = R145 (RIGHT column)
```

```
  Connection Map  US-YOUR-SITE DH2
  @ = Source R45 (Left)     # = Peer R200 (Right)

  Left (R1-R120)                              Right (R121-R260)
    1 - - - - - - - - - - 10                    121 - - - - - - - - - - 130
   20 - - - - - - - - - - 11                    140 - - - - - - - - - - 131

   21 - - - - - - - - - - 30                    141 - - - - - - - - - - 150
   40 - - - - @ - - - - - 31                    160 - - - - - - - - - - 151

   61 - - - - - - - - - - 70                    191 - - - - - - - - # - 200
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## Getting Started

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or desktop app

### Installation

1. **Clone into your skills directory:**

   ```bash
   git clone https://github.com/rpatino-cw/datahall-map.git ~/.claude/skills/datahall-map
   ```

   Or copy manually:

   ```bash
   mkdir -p ~/.claude/skills/datahall-map
   cp SKILL.md README.md sample-layouts.json ~/.claude/skills/datahall-map/
   ```

2. **Set up your hall layouts:**

   ```bash
   mkdir -p ~/.datahall
   cp ~/.claude/skills/datahall-map/sample-layouts.json ~/.datahall/layouts.json
   ```

3. **Edit `~/.datahall/layouts.json`** to match your site (see [Layout Schema](#layout-schema) below).

   Or skip this step — Claude will run a setup wizard on first use and ask you about your halls interactively.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

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

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## Layout Schema

Each hall is a key in `layouts.json` using the format `SITE-CODE.HALL`:

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

| Field | Type | Description |
|---|---|---|
| `racks_per_row` | int | Default rack count per row |
| `columns` | array | Column groups rendered side-by-side |
| `columns[].label` | string | Display name ("Left", "Right", "A", etc.) |
| `columns[].start` | int | First rack number in column |
| `columns[].num_rows` | int | Number of rows in column |
| `columns[].racks_per_row` | int? | Override default for this column |
| `serpentine` | bool | Odd rows count backwards |
| `entrance` | string | Door location: `bottom-right`, `bottom-left`, `top-right`, `top-left` |
| `total_racks` | int | Total rack count (informational) |

### Serpentine vs Sequential

Most CW data halls use **serpentine** numbering — odd rows count backwards so rack numbers snake through the floor:

```
Row 0:  1  2  3  4  5  6  7  8  9  10   (left to right)
Row 1: 20 19 18 17 16 15 14 13 12  11   (right to left)
Row 2: 21 22 23 24 25 26 27 28 29  30   (left to right)
```

Some sections (like `SEC1` halls) use **sequential** numbering where every row counts left to right.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## Modes

### Highlight

Find a rack on the floor. The target rack is marked with `#` (yellow bold in terminal).

### Walking Route

Traces a path from the hall entrance through the corridor to the target rack using box-drawing characters (`|`, `+`, `=`).

### Connection Map

Highlights two related racks in different colors — `@` (cyan) for source, `#` (yellow) for peer. Useful for tracing cables, IB links, or network connections.

### Heatmap

Colors racks by issue density:

| Symbol | Meaning |
|---|---|
| `X` (red) | 10+ issues |
| `x` (yellow) | 5-9 issues |
| `.` (green) | 1-4 issues |
| `-` (dim) | Clean |

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## Roadmap

- [x] Highlight maps
- [x] Walking routes
- [x] Connection maps (dual highlight)
- [x] Heatmaps
- [x] N-column support (2, 3, or more)
- [x] Setup wizard for new sites
- [ ] Image export (PNG)
- [ ] Jira integration (pull issue counts for heatmaps)
- [ ] NetBox integration (auto-detect hall layouts)

See the [open issues](https://github.com/rpatino-cw/datahall-map/issues) for a full list of proposed features and known issues.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## Contributing

Contributions are welcome. To add your site's layout:

1. Fork the repo
2. Add your site to `sample-layouts.json`
3. Open a PR with the site code and hall name

**Keep sensitive information out** — rack counts and numbering patterns are fine, but don't include hostnames, IPs, or device serial numbers.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## License

Distributed under the MIT License. See `LICENSE` for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

## Contact

Romeo Patino — [@rpatino-cw](https://github.com/rpatino-cw)

Project Link: [https://github.com/rpatino-cw/datahall-map](https://github.com/rpatino-cw/datahall-map)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

---

<!-- MARKDOWN LINKS & IMAGES -->
[contributors-shield]: https://img.shields.io/github/contributors/rpatino-cw/datahall-map.svg?style=for-the-badge
[contributors-url]: https://github.com/rpatino-cw/datahall-map/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/rpatino-cw/datahall-map.svg?style=for-the-badge
[forks-url]: https://github.com/rpatino-cw/datahall-map/network/members
[stars-shield]: https://img.shields.io/github/stars/rpatino-cw/datahall-map.svg?style=for-the-badge
[stars-url]: https://github.com/rpatino-cw/datahall-map/stargazers
[issues-shield]: https://img.shields.io/github/issues/rpatino-cw/datahall-map.svg?style=for-the-badge
[issues-url]: https://github.com/rpatino-cw/datahall-map/issues
[license-shield]: https://img.shields.io/github/license/rpatino-cw/datahall-map.svg?style=for-the-badge
[license-url]: https://github.com/rpatino-cw/datahall-map/blob/main/LICENSE
