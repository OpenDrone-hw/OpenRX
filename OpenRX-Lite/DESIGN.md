# OpenRX-Lite

ESP32-C3 + SX1281, 2.4 GHz only, on-board chip antenna. Same circuit as Lite-UFL, different antenna interface.

Common circuit, antennas, I/O pads, pin map and firmware targets: [../DESIGN.md](../DESIGN.md). This file carries only what is specific to the Lite.

## Board preview

| Front | Back |
|-------|------|
| ![Front](../images/openrx-lite-front.png) | ![Back](../images/openrx-lite-back.png) |

## Schematic

- Main sheet: `esp32c3_sx1281_lite.kicad_sch`
- RF chain: `SX1281 (U3) RFIO -> 2450FM07D0034T (FL1) -> 47948-0001 chip antenna (AE2)`
- No RF front-end (PA/LNA), no RF switch, no sub-GHz

### No boot button

GPIO 9 pull-up only, no physical switch.

## Firmware

ELRS target, platform, upload methods and pin map: the [Firmware targets](../DESIGN.md#firmware-targets) and [Pin map](../DESIGN.md#pin-map) sections of ../DESIGN.md, sourced from `shared/elrs-targets/OpenRX Lite 2400.json`.

## Flash interface

Pads and BOOT behaviour are the family default: [I/O pads and button](../DESIGN.md#io-pads-and-button).

## Sourcing

- All parts LCSC basic/preferred where possible
- `C2651081` 2450FM07D0034T: 2.4 GHz band-pass filter
- `C2151551` SX1281IMLTRT: watch stock for volume runs
- `C152351` 47948-0001: Molex chip antenna
