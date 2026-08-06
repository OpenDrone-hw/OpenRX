# Agent notes

Facts for AI agents working in this repo.

- KiCad 10 projects, one directory per variant: `OpenRX-Lite/`, `OpenRX-Lite-UFL/`, `OpenRX-Mono/`, `OpenRX-Gemini/`. Top schematic and board are `OpenRX-<variant>.kicad_sch` / `.kicad_pcb`, all 6 copper layers. Gemini also holds the combined production board `OpenRX-all.kicad_pcb` and its `fab/` gerber set.
- Clone with `git clone --recursive`; the `libs/KiCad-Library` submodule is referenced by the project lib tables, alongside the in-repo `shared/libs/OpenRX-Shared.*`. Symbols and footprints are embedded in the design files.
- Never edit `.kicad_*` files as text. Use kicad-skip or the pcbnew API, and only for metadata (text variables, symbol BOM/doc fields). Never change nets, placement, or component values.
- Checks and exports:

```
kicad-cli sch erc --exit-code-violations OpenRX-Lite/OpenRX-Lite.kicad_sch
kicad-cli pcb drc --exit-code-violations OpenRX-Lite/OpenRX-Lite.kicad_pcb
kicad-cli sch export netlist --format kicadsexpr -o /tmp/OpenRX-Lite.net OpenRX-Lite/OpenRX-Lite.kicad_sch
```

- Fabrication Toolkit config: per-variant `fabrication-toolkit-options.json` (tracked). Generated `production/` output is gitignored.
- ELRS hardware targets: `shared/elrs-targets/` is canonical and must stay identical to the firmware-side copies. Pin maps in [DESIGN.md](DESIGN.md), flashing in [FLASHING.md](FLASHING.md).
- `archive/legacy-projects/` and `shared/sheets/` are frozen history; do not extend them.
- Docs are deterministic: current fact only, no TODOs or plans.
- `main` is protected; push feature branches and open PRs.
