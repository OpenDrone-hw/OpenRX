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
assembly. The fab arrays this panel and adds rails and fiducials; it does not
panelise mixed designs or edit customer Gerbers. It is manufacturing geometry,
not a design source.

It is generated, never edited: `panel/panel.json` names each board's
repository, tag, offset and reference block, and
`$KPY <hardware-tooling>/hardware/kicad/panel_merge.py panel/panel.json` rebuilds it
from the tagged board files, refills every zone and writes the refmap.
`--check` must pass before a release. A hand-edited panel shipped an empty
Mono In3 +3V3 plane in 2026 and every Mono built from it was dead.
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
`panel/single/<board>/` holds one board per variant, built the same way from
a one-board `panel.json` with no frame: the board at tag `rev2.2` with its
own references and outline, titleblock `rev2.2` and the same `mpn_renumber`
(Mono and Gemini). Lite and Lite-UFL are `rev2.3`, with the 1 Nov 2026 fix:
released directly from their board repositories, whose titleblock says
`rev2.3`, so no single board is kept here for them.
It is for fabricating each variant on its own, where the fab arrays each
design itself. Release each with `--panel-of` naming that
board's released BOM.
The OpenDrone release standard is
[RELEASES.md](https://github.com/OpenDrone-hw/.github/blob/main/RELEASES.md).

## Rules

The [Board rules](#board-rules) at the end of this file apply, a CI-checked copy of the [hardware-template](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules) section. Only what differs for this board is listed here.

## Board rules

Shared by every OpenDrone KiCad board repository, byte for byte, and checked
in CI against [hardware-template](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules).
Edit them there, never in a board.

### Files

Every board has this layout; its Repo table lists only what differs.

| | |
|---|---|
| Designed in | KiCad 10 |
| Status | The `status-*` topic on the repository, never written in a file |
| Local library | `hardware/lib.kicad_sym`, `hardware/lib.pretty/`, `hardware/lib.3dshapes/`, nickname `lib` |
| Shared library | `hardware/KiCad-Library/`, submodule of [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), nickname `OpenDrone`; 3D models and exact component datasheets resolve through the project text variable `OPENDRONE_LIB` |
| Fab config | `hardware/fabrication-toolkit-options.json` |
| Board setup | 6 layers, 0.09 mm clearance and track, via 0.35 on 0.20 drill |
| License | CERN-OHL-S-2.0 |

A repository holds design files only. Sourcing (suppliers, prices, quotes, RFQs, contacts) is handled by Incutec and never lives in OpenDrone repositories.

### Parts and datasheets

- **Per-repository part index:** the root schematic and its listed sub-sheets
  are authoritative for what this board fits. Export the netlist with the
  command below when a script-readable board index is needed; do not maintain
  a second hand-written BOM.
- **Proven shared parts:**
  `hardware/KiCad-Library/PARTS-USED.md` is the catalogue index. Its `Boards`
  column identifies every repository using each LCSC part; filter that column
  for the repository name to get its proven shared-part view.
- **Exact datasheets:**
  `hardware/KiCad-Library/datasheet/manifest.json` maps shared symbols to the
  committed PDFs and their SHA-256 hashes. The PDFs live in
  `hardware/KiCad-Library/datasheet/`, and symbol links resolve there through
  `OPENDRONE_LIB`.
- **Local-only parts:** inspect `hardware/lib.kicad_sym`,
  `hardware/lib.pretty/`, and `hardware/lib.3dshapes/`, then verify supplier
  fields in the board schematic. Do not duplicate a shared part or datasheet.

### Commands

`<name>` is the KiCad project named in the board's Repo table.

```sh
# schematic and board checks
kicad-cli sch erc hardware/<name>.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones hardware/<name>.kicad_pcb

# netlist, for scripted analysis
kicad-cli sch export netlist --format kicadsexpr -o /tmp/<name>.net hardware/<name>.kicad_sch
```

On macOS `kicad-cli` is at
`/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`, and `pcbnew` imports
only under KiCad's bundled Python, `KPY`:
`/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3`.
Reusable scripts for renders, STEP export and packaging art come from Incutec
hardware tooling. The OpenDrone release standard is
[RELEASES.md](https://github.com/OpenDrone-hw/.github/blob/main/RELEASES.md).
Board-specific scripts, where a board has any, live in `hardware/tools/`.

### Rules

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
  kicad-skip cannot round-trip KiCad 10 schematics: use a full S-expression
  parser, `kicad-cli sch upgrade --force`, then an ERC and netlist diff.
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
  The commands are under Commands.

### By task

- Check the design: run the ERC and DRC commands in Commands before every pull request.
- Add a part: place it from the `OpenDrone` library if `hardware/KiCad-Library/PARTS-USED.md` lists it; otherwise import it into `lib` with `$KPY <hardware-tooling>/hardware/kicad/import_part.py` (read `--help` first), KiCad closed.
- Render the board for the README: `$KPY <hardware-tooling>/hardware/kicad/render_board.py hardware/<name>.kicad_pcb --outdir images`, KiCad closed.
- Analyse the netlist: export it with the netlist command in Commands, then read it with a script; never hand-write a second BOM.
- Update the shared library: `git submodule update --remote hardware/KiCad-Library`, run DRC, commit as its own reviewed change.
