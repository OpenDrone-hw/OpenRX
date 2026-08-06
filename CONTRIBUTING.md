# Contributing

## Before you edit a board

KiCad files cannot be merged. If two people edit the same `.kicad_pcb`, one of
them loses the work.

1. Open an issue describing the change and wait for the maintainer to agree.
2. Claim the file: `git lfs lock <path>.kicad_pcb`
3. `git lfs locks` shows who holds what. If a file is held, talk to that person
   in the issue instead of starting anyway.
4. After the pull request is merged: `git lfs unlock <path>.kicad_pcb`

A pull request touching a file locked by someone else gets closed, not merged.

## Setup

```
git clone --recursive https://github.com/incutec-hw/OpenRX.git
```

KiCad 10. The `--recursive` flag pulls `libs/KiCad-Library`, which the project
library tables reference for shared parts. Edit in KiCad; never text-edit
`.kicad_sch` or `.kicad_pcb`.

## Before opening a pull request

- ERC and DRC clean on every board you touched:

```
kicad-cli sch erc --exit-code-violations OpenRX-<variant>/OpenRX-<variant>.kicad_sch
kicad-cli pcb drc --exit-code-violations OpenRX-<variant>/OpenRX-<variant>.kicad_pcb
```

- Post before and after board renders in the pull request. A text diff of a
  PCB is not reviewable.
- Say what you tested on real hardware, or say that you tested nothing.

## Parts

Add new symbols, footprints and 3D models to this repo's local libraries.
That is the working default.

[incutec-hw/KiCad-Library](https://github.com/incutec-hw/KiCad-Library) is a
mirror of parts we already use or stock. Check it first: if the part is there,
we have used it before, which saves sourcing work. Promoting a part into it is
a separate deliberate step, not a requirement for contributing here.

Prefer LCSC basic parts.

## Licensing

Hardware contributions are licensed CERN-OHL-S-2.0, the same as the project.
Sign off your commits with `git commit -s` to certify you wrote the work.

## Questions

Open a GitHub issue.
