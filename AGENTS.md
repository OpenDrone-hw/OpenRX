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
`$KPY hardware/kicad/panel_merge.py panel/panel.json` in `scripts` rebuilds it
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
