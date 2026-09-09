# OpenRX family hub

This repository is the durable family home for OpenRX. It preserves the
combined repository history, stars, issues, tags, and releases. It does not
own live KiCad sources or ExpressLRS target definitions after the split.

## Scope routing

| Board | Authoritative repository |
|---|---|
| OpenRX Lite | [OpenDrone-hw/OpenRX-Lite](https://github.com/OpenDrone-hw/OpenRX-Lite) |
| OpenRX Lite-UFL | [OpenDrone-hw/OpenRX-Lite-UFL](https://github.com/OpenDrone-hw/OpenRX-Lite-UFL) |
| OpenRX Mono | [OpenDrone-hw/OpenRX-Mono](https://github.com/OpenDrone-hw/OpenRX-Mono) |
| OpenRX Gemini | [OpenDrone-hw/OpenRX-Gemini](https://github.com/OpenDrone-hw/OpenRX-Gemini) |

Route board design, firmware-target, validation, rendering, and release work to
the corresponding repository. Keep family navigation and cross-board context
here; historical discussion stays in Git history and GitHub threads. Do not
copy board facts back into this repo when a link to the authoritative board
README or AGENTS file is sufficient.

The original combined source remains available through Git history and the
`rev2` and `rev2.1` tags. Do not recreate a second live copy on this branch.

## The production panel

`panel/` holds `OpenRX-panel-rev2`, the four boards merged onto one strip for
combined fabrication and assembly. It is manufacturing geometry, not a design
source, and it is the one board file this repository owns: it belongs to no
single board repository because it contains all four.

Every part on it comes from a board repository and nothing is designed here. A
change to nets, footprints or component values goes to the owning board
repository and the panel is rebuilt from the result. References are unique
across the strip in blocks of 100 per board, so `U217` is OpenRX-Lite-UFL's
`U17`; `OpenRX-panel-rev2_refmap.csv` is the generated map and
`hardware/kicad/panel_renumber.py` in `scripts` regenerates it.

The panel has no schematic, so the export gate checks its BOM against the four
released board BOMs instead of a netlist. Release it with the production
handoff adapter and `--panel-of` naming all four.
The OpenDrone release standard is
[RELEASES.md](https://github.com/OpenDrone-hw/.github/blob/main/RELEASES.md).
