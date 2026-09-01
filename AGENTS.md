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
the corresponding repository. Keep family navigation, cross-board context,
and historical discussion here. Do not copy board facts back into this repo
when a link to the authoritative board README or AGENTS file is sufficient.

The original combined source remains available through Git history and the
`rev2` and `rev2.1` tags. Do not recreate a second live copy on this branch.
The OpenDrone release standard is
[RELEASES.md](https://github.com/OpenDrone-hw/.github/blob/main/RELEASES.md).
