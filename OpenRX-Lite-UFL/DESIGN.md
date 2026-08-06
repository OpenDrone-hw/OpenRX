# OpenRX-Lite-UFL

ESP32-C3 + SX1281, 2.4 GHz only, U.FL antenna connector. Same circuit as the Lite, different antenna interface.

Common circuit, antennas, I/O pads, pin map and firmware targets: [../DESIGN.md](../DESIGN.md). This file carries only what is specific to the Lite-UFL.

## Board preview

| Front | Back |
|-------|------|
| ![Front](../images/openrx-lite-ufl-front.png) | ![Back](../images/openrx-lite-ufl-back.png) |

## Schematic

- Main sheet: `esp32c3_sx1281_lite.kicad_sch`
- RF chain: `SX1281 (U3) RFIO -> 2450FM07D0034T (FL1) -> U.FL-R-SMT-1(80) (J1)`
- No RF front-end (PA/LNA), no RF switch, no sub-GHz
- 2450FM07D0034T output is 50 ohm, U.FL is 50 ohm: clean match

### No boot button

Same as the Lite: GPIO 9 pull-up only, no physical switch.

## Firmware

Same target as the Lite. ELRS target, platform, upload methods and pin map: the [Firmware targets](../DESIGN.md#firmware-targets) and [Pin map](../DESIGN.md#pin-map) sections of ../DESIGN.md, sourced from `shared/elrs-targets/OpenRX Lite-UFL 2400.json`.

## Flash interface

Pads and BOOT behaviour are the family default: [I/O pads and button](../DESIGN.md#io-pads-and-button).

## Sourcing

- All parts LCSC basic/preferred where possible
- `C2651081` 2450FM07D0034T: 2.4 GHz band-pass filter
- `C2151551` SX1281IMLTRT: watch stock for volume runs
- `C88374` U.FL-R-SMT-1(80): antenna connector
