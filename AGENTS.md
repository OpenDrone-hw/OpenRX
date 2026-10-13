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

`panel/` holds `OpenRX-panel-rev2.2`: the four boards joined into one 4-in-1
panel by breakaway webs with mouse-bite rows, for combined fabrication and
assembly. The fab arrays this panel and adds rails and fiducials; NextPCB does
not panelise mixed designs or edit customer Gerbers (2026-10-13). It is
manufacturing geometry, not a design source.

It is generated, never edited: `panel/panel.json` names each board's
repository, tag, offset and reference block, and
`$KPY <scripts>/hardware/kicad/panel_merge.py panel/panel.json` (the hardware tooling) rebuilds it
from the tagged board files, refills every zone and writes the refmap.
`--check` must pass before a release. A hand-edited panel shipped an empty
Mono In3 +3V3 plane in 2026 and every Mono of that batch was dead.
The webs and mouse bites come from `frame` in `panel.json`: the outline and
`Panelization:` footprints of the earlier panel at commit 0459acc, nothing else
of it. Copper keepouts over each web keep the planes as on the released boards.

A change to nets, footprints or component values goes to the owning board
repository; tag it, update the tag in `panel.json` and rebuild. References are
unique in blocks of 100 per board, so `U217` is OpenRX-Lite-UFL's `U17`; nets
are named `<board>/<net>`. `mpn_renumber` holds only manufacturer renumbers of
the same part, with the source.

Release it with the production handoff adapter and `--panel-of` naming the four
released board BOMs: the adapter checks the panel BOM line by line against them.
The OpenDrone release standard is
[RELEASES.md](https://github.com/OpenDrone-hw/.github/blob/main/RELEASES.md).

## Rules

Identical in every OpenDrone board repo. Do not edit here; edit the template.

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
- **Metadata yes, connections no.** An agent may write BOM and documentation
  fields (MPN, Manufacturer, LCSC, Cost, Datasheet, text variables). An agent
  may not change nets, wiring, routing, placement, footprint assignment, or any
  value that changes the circuit.
- **Close KiCad before any write to a KiCad file.** KiCad caches library tables
  at process start and overwrites files on save.
- **Reuse before you draw.** Check the `OpenDrone` library and its
  `PARTS-USED.md` first. If the part is there we have already sourced,
  footprinted and shipped it, and its symbol links to the exact committed
  datasheet: place it from `OpenDrone`. Draw a new part into `lib` only when
  the catalogue has nothing that fits, imported with
  `easyeda2kicad` from its LCSC number. Pulling a newer catalogue is a
  deliberate, reviewed commit: `git submodule update --remote
  hardware/KiCad-Library`, then DRC.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **Run ERC and DRC before every pull request.** Existing approved findings
  may remain; a new type or increased count must be reviewed before merge.
  Commands are in Environment above.

