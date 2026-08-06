# OpenRX-Gemini

Dual-LR1121 Gemini/Xrossband receiver. ESP32-C3 + 2x LR1121 + 2x RFX2401C + 2x SKY13373-460LF + 2x Johanson IPD.

Common circuit, antennas, I/O pads, pin map and firmware targets: [../DESIGN.md](../DESIGN.md). This file carries only what is specific to the Gemini.

## Board preview

| Front | Back |
|-------|------|
| ![Front](../images/openrx-gemini-front.png) | ![Back](../images/openrx-gemini-back.png) |

## Schematic

- Top sheet: `OpenRX-Gemini.kicad_sch` -> `esp32-c3.kicad_sch` + `clock.kicad_sch` + `lr1121.kicad_sch` (instantiated twice). `esp32c3_lr1121_gemini.kicad_sch` is a legacy flat sheet, not in the hierarchy.
- Radio 1 (U3 LR1121, U4 RFX2401C, U5 SKY13373, T1 IPD, FL1 BPF): RF chain -> `J1` U.FL
- Radio 2 (U6 LR1121, U7 RFX2401C, U8 SKY13373, T2 IPD, FL2 BPF): mirrors radio 1 -> `J2` U.FL
- Per-radio 2.4 GHz and sub-GHz paths are identical to the Mono (see [../OpenRX-Mono/DESIGN.md](../OpenRX-Mono/DESIGN.md)), including the SKY13373 truth table
- Shared 32 MHz TCXO in `clock.kicad_sch`, powered from radio 2's (U6) VTCXO pin: U6 must initialize first or neither radio has a clock
- Each radio drives its own switch and front-end: `DIO5 -> RXEN`, `DIO6 -> TXEN`, `DIO7 -> V1`, `DIO8 -> V2`. Wiring is symmetric, so the single `radio_rfsw_ctrl` applies to both radios via `SetDioAsRfSwitch`.
- In DualBand/X modes the firmware never swaps radios: radio 1 (U3) is always sub-GHz, radio 2 (U6) is always 2.4 GHz. Antennas: `J1` = 900 MHz, `J2` = 2.4 GHz.

## Firmware

Same binary as the Mono. ELRS target, platform, upload methods and pin map: the [Firmware targets](../DESIGN.md#firmware-targets) and [Pin map](../DESIGN.md#pin-map) sections of ../DESIGN.md, sourced from `shared/elrs-targets/OpenRX Gemini LR1121.json`. Gemini-specific settings in that JSON:

- `radio_nss_2` enables dual-radio mode; radio 2 pins NSS 7, RST 10, BUSY 8, DIO1 18
- `radio_rfsw_ctrl: [15, 0, 12, 8, 8, 6, 0, 5]`, same as the Mono ([decode table](../OpenRX-Mono/DESIGN.md#rfsw_ctrl-decode))
- Requires the ExpressLRS fork branch: TCXO enable, radio-1 second reset (NRESET on strapping pin GPIO 2), and software chip-select for radio-1 NSS on GPIO 0

## Flash interface

Pads are the family default: [I/O pads and button](../DESIGN.md#io-pads-and-button). Gemini delta: BOOT is a tactile button (U9, TS2306A) on GPIO 9 instead of the TP5 pad, held during power-up for UART download mode.

## Sourcing

- `C150853` SKY13373-460LF (x2)
- `C19842466` 0900PC16J0042001E (x2, consign from DigiKey)
- `C7498014` LR1121IMLTRT (x2)
- `C2651081` 2450FM07D0034T (x2)
- `C783588` RFX2401C (x2)
