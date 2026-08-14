# OpenRX

Open source ExpressLRS receiver family. Four board variants share an ESP32-C3
core, a TLV75533 3.3 V LDO and the ExpressLRS unified firmware; they differ in
radio IC, frequency band, RF front end and antenna interface. All four are
6-layer boards.

## Repo

| | |
|---|---|
| Maintainer | @Just4Stan. Taken over from @bastian2001 on 2026-08-11 |
| Status | See the `status-*` topic on the repo. Never written here. |
| Designed in | KiCad 10 |
| Layout | Multi-variant. One directory per variant at repo root, each with its own KiCad project |
| Variants | `OpenRX-Lite/`, `OpenRX-Lite-UFL/`, `OpenRX-Mono/`, `OpenRX-Gemini/` |
| Shared | `shared/sheets/` hierarchical sheets, `shared/libs/`, `shared/elrs-targets/` firmware target JSON |
| Shared library | [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), catalogue only; every library this board uses is local to the repo |
| Archive | Five superseded designs live in git history (removed from the tip 2026-08-14, last at `archive/legacy-projects/`) |
| License | CERN-OHL-S-2.0 |

Each variant directory sits exactly one level below the repo root, which is what
makes the shared 3D model paths resolve.

## Rules

Identical in every OpenDrone repo. Do not edit here; edit the template.

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
- **Metadata yes, connections no.** An agent may write BOM and documentation
  fields. An agent may not change nets, wiring, routing, placement, footprint
  assignment, or any value that changes the circuit.
- **Close KiCad before any write to a KiCad file.**
- **Reuse before you draw.** Check
  [KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library) and its
  `PARTS-USED.md` first, and copy what fits into this repo's own library.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **Sheets are shared across four variants.** A change in `shared/sheets/`
  changes every board that instantiates it. Say which variants you intend to
  affect.
- **ERC and DRC clean before every pull request**, on every variant you touched.

```sh
kicad-cli sch erc --exit-code-violations OpenRX-<variant>/OpenRX-<variant>.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones --exit-code-violations OpenRX-<variant>/OpenRX-<variant>.kicad_pcb
```


Family-level design description of the four OpenRX variants. Values are extracted from the KiCad design files and the ExpressLRS target JSON in `shared/elrs-targets/`. Per-variant RF-chain detail is in [Variants](#variants) below. RF-switch state decode: [rfsw_ctrl decode](#rfsw_ctrl-decode).

## Common circuit

- **MCU**: Espressif **ESP32-C3** (QFN-32, 5 x 5 mm), U1. GPIO 9 is the BOOT/strapping pin.
- **LDO**: **TLV75533PDQNR** (U2, X2SON-4), 5 V on the `5V` pad to the 3.3 V rail. On-canvas note: 5 V to 3.3 V, 238 mV @ 500 mA maximum dropout.
- **Clock**:
  - ESP32-C3 crystal **CJ17-400001010B20** (X1, 40 MHz), all variants.
  - Radio **TCXO**: **OW7EL89CENUNFAYLC-52M** (52 MHz) on the SX1281 variants (Lite, Lite-UFL); **OW7EL89CENUYO3YLC-32M** (32 MHz) on the LR1121 variants (Mono, Gemini). On Gemini the single TCXO (`clock.kicad_sch`) is shared by both radios and powered from radio 2's VTCXO pin.
- **Status LED**: WS2812B RGB (**XL-1010RGBC-WS2812B**, D1), powered from +3.3 V. LED signal GPIO differs by variant (see [Pin map](#pin-map)).
- **Telemetry / control**: CRSF over the ESP32-C3 UART0 (`U0RXD`/`U0TXD`).
- **Power input**: 5 V on the `5V` solder pad.
- **Flashing**: UART, Wi-Fi OTA, or Betaflight passthrough, as declared per target in `shared/elrs-targets/targets_entries.json`. Procedures are the standard ExpressLRS ones, see the [ExpressLRS receiver flashing docs](https://www.expresslrs.org/quick-start/receivers/).

## Antennas

- Every variant carries a **2450AT18A100E ceramic chip antenna (AE1)** on the net `WIFI`: this is the **ESP32-C3 Wi-Fi antenna** used for OTA flashing/config, not the ELRS link antenna.
- The **ELRS link antenna** is fed from the radio's RF output through the band-pass filter (`FL1-OUT`): on **Lite** it terminates at the Molex **47948-0001** chip antenna (AE2); on **Lite-UFL / Mono / Gemini** it terminates at the **U.FL** connector(s).
- **Mono** and **Gemini** share one firmware binary; `radio_nss_2`/`radio_rst_2` in the Gemini target JSON enables dual-radio (Xrossband) mode. In dual-band modes the firmware assigns radio 1 (U3, J1) to sub-GHz and radio 2 (U6, J2) to 2.4 GHz: J1 takes the 900 MHz antenna, J2 the 2.4 GHz antenna.

## I/O pads and button

Solder pads carry the external interface. Pad to net mapping is derived from the schematic netlists:

| Pad | Net | ESP32-C3 | Function |
|---|---|---|---|
| `RX` (TP1) | `U0RXD` | GPIO 20 | CRSF / serial in to RX |
| `TX` (TP2) | `U0TXD` | GPIO 21 | CRSF / serial out / telemetry |
| `5V` (TP3) | `+5V` | - | 5 V supply in to TLV75533 LDO |
| `GND` (TP4) | `GND` | - | Ground |
| `BOOT` (TP5) | `BOOT` | GPIO 9 | Pull low at power-up for UART download mode |

- **Lite, Lite-UFL, Mono**: `BOOT` is a solder pad (TP5).
- **Gemini**: the BOOT/GPIO 9 function is a populated **tactile button** (U9, TS2306A) instead of a pad; the `RX`/`TX`/`5V`/`GND` pads remain.

## Pin map

Radio interface and per-variant GPIO assignments, from the target JSON in `shared/elrs-targets/`:

| Function | Lite / Lite-UFL | Mono | Gemini |
|---|---|---|---|
| Serial RX / TX | 20 / 21 | 20 / 21 | 20 / 21 |
| Radio SCK / MOSI / MISO | 6 / 4 / 5 | 6 / 4 / 5 | 6 / 4 / 5 |
| Radio NSS / RST | 7 / 2 | 7 / 2 | 0 / 2 |
| Radio BUSY / DIO1 | 3 / 1 | 3 / 1 | 3 / 1 |
| Radio 2 NSS / RST / BUSY / DIO1 | - | - | 7 / 10 / 8 / 18 |
| RF switch control | - | LR1121 DIO5-DIO8, `radio_rfsw_ctrl` `[15,0,12,8,8,6,0,5]` | same, per radio |
| Status LED | 8 (GRB) | 8 (GRB) | 19 (GRB) |
| BOOT / button | 9 | 9 | 9 (button) |

On the LR1121 variants the RF switch and front-end are driven by the radio's own DIO pins, not ESP32-C3 GPIOs: DIO5 = RFX2401C RXEN, DIO6 = RFX2401C TXEN, DIO7 = SKY13373 V1, DIO8 = SKY13373 V2. Each `radio_rfsw_ctrl` byte is a DIO5-DIO8 bitmask passed to `SetDioAsRfSwitch`; the decode table is under [rfsw_ctrl decode](#rfsw_ctrl-decode).

Transmit power per variant is in the variant table in the [README](README.md); the authoritative values are `power_values` in the per-variant target JSON.

## Firmware targets

The ExpressLRS hardware-target definitions live in this repo (`shared/elrs-targets/`, with `targets_entries.json` prepared for upstream submission; they are not merged into the upstream [ExpressLRS/targets](https://github.com/ExpressLRS/targets) repo). The referenced unified firmware images exist upstream:

| Variant | Product name | ELRS firmware target | Platform | Upload |
|---|---|---|---|---|
| Lite | OpenRX Lite 2.4GHz RX | `Unified_ESP32C3_2400_RX` | esp32-c3 | UART, Wi-Fi, Betaflight |
| Lite-UFL | OpenRX Lite-UFL 2.4GHz RX | `Unified_ESP32C3_2400_RX` | esp32-c3 | UART, Wi-Fi, Betaflight |
| Mono | OpenRX Mono Dual Band RX | `Unified_ESP32C3_LR1121_RX` | esp32-c3 | UART, Wi-Fi, Betaflight |
| Gemini | OpenRX Gemini XrossBand RX | `Unified_ESP32C3_LR1121_RX` | esp32-c3 | UART, Wi-Fi, Betaflight |

Minimum ExpressLRS version **3.5.0**, as declared by `min_version` in `targets_entries.json`. Hardware pin maps live in the per-variant target JSON. Mono and Gemini currently require an ExpressLRS fork branch for TCXO enable (`SetTcxoMode`) and, on Gemini, radio-1 reset and chip-select handling; stock unified firmware runs the SX1281 variants unmodified.

## Libraries

Symbols and footprints are embedded in the design files. The project lib tables reference the in-repo `shared/libs/OpenRX-Shared.*` library (`${KIPRJMOD}/../shared/libs/`). Passives and some packages (coax connectors, QFNs) use stock KiCad library footprints resolved through their embedded copies. Symbols carry an `LCSC` property for JLCPCB BOM export.

## Revisions

- **2026-08-05**: OSHWA certification (BE000030 to BE000033), Lite/Lite-UFL ELRS target pin remap, Mono/Gemini target updates, shared Incutec KiCad-Library submodule wired (2026-08-04). Layout rework (clock 3.3 V supply, enlarged pads, Lite/Lite-UFL and Mono outlines +1.0 mm) landed after the validated build and has not been fabricated.
- **2026-06-10**: combined `OpenRX-all` fabrication set ordered at JLCPCB (gerbers, BOM, CPL in `OpenRX-Gemini/`).
- **2026-06-07**: single-source-of-truth docs pass, standardized board renders.
- **2026-03-23**: initial repo, 6-receiver lineup; later reduced to the four current variants (retired designs in git history, formerly `archive/legacy-projects/`).


## Variants


### OpenRX-Lite

ESP32-C3 + SX1281, 2.4 GHz only, on-board chip antenna. Same circuit as Lite-UFL, different antenna interface.

Common circuit, antennas, I/O pads, pin map and firmware targets are shared, see above. This section carries only what is specific to the Lite.

#### Board preview

| Front | Back |
|-------|------|
| ![Front](OpenRX-Lite/../images/openrx-lite-front.png) | ![Back](OpenRX-Lite/../images/openrx-lite-back.png) |

#### Schematic

- Main sheet: `esp32c3_sx1281_lite.kicad_sch`
- RF chain: `SX1281 (U3) RFIO -> 2450FM07D0034T (FL1) -> 47948-0001 chip antenna (AE2)`
- No RF front-end (PA/LNA), no RF switch, no sub-GHz

#### No boot button

GPIO 9 pull-up only, no physical switch.

#### Firmware

ELRS target, platform, upload methods and pin map: the [Firmware targets](#firmware-targets) and [Pin map](#pin-map) sections above, sourced from `shared/elrs-targets/OpenRX Lite 2400.json`.

#### Flash interface

Pads and BOOT behaviour are the family default: [I/O pads and button](#io-pads-and-button).

#### Sourcing

- All parts LCSC basic/preferred where possible
- `C2651081` 2450FM07D0034T: 2.4 GHz band-pass filter
- `C2151551` SX1281IMLTRT: watch stock for volume runs
- `C152351` 47948-0001: Molex chip antenna


### OpenRX-Lite-UFL

ESP32-C3 + SX1281, 2.4 GHz only, U.FL antenna connector. Same circuit as the Lite, different antenna interface.

Common circuit, antennas, I/O pads, pin map and firmware targets are shared, see above. This section carries only what is specific to the Lite-UFL.

#### Board preview

| Front | Back |
|-------|------|
| ![Front](OpenRX-Lite-UFL/../images/openrx-lite-ufl-front.png) | ![Back](OpenRX-Lite-UFL/../images/openrx-lite-ufl-back.png) |

#### Schematic

- Main sheet: `esp32c3_sx1281_lite.kicad_sch`
- RF chain: `SX1281 (U3) RFIO -> 2450FM07D0034T (FL1) -> U.FL-R-SMT-1(80) (J1)`
- No RF front-end (PA/LNA), no RF switch, no sub-GHz
- 2450FM07D0034T output is 50 ohm, U.FL is 50 ohm: clean match

#### No boot button

Same as the Lite: GPIO 9 pull-up only, no physical switch.

#### Firmware

Same target as the Lite. ELRS target, platform, upload methods and pin map: the [Firmware targets](#firmware-targets) and [Pin map](#pin-map) sections above, sourced from `shared/elrs-targets/OpenRX Lite-UFL 2400.json`.

#### Flash interface

Pads and BOOT behaviour are the family default: [I/O pads and button](#io-pads-and-button).

#### Sourcing

- All parts LCSC basic/preferred where possible
- `C2651081` 2450FM07D0034T: 2.4 GHz band-pass filter
- `C2151551` SX1281IMLTRT: watch stock for volume runs
- `C88374` U.FL-R-SMT-1(80): antenna connector


### OpenRX-Mono

Single-LR1121 dual-band receiver. ESP32-C3 + LR1121 (U3) + RFX2401C PA/LNA (U4) + SKY13373-460LF RF switch (U5) + Johanson 0900PC16J0042001E balun/IPD (T1).

Common circuit, antennas, I/O pads, pin map and firmware targets are shared, see above. This section carries only what is specific to the Mono.

#### Board preview

| Front | Back |
|-------|------|
| ![Front](OpenRX-Mono/../images/openrx-mono-front.png) | ![Back](OpenRX-Mono/../images/openrx-mono-back.png) |

#### Schematic

- Main sheet: `esp32c3_lr1121_mono.kicad_sch`
- 2.4 GHz path: `LR1121 RFIO_HF -> 2450FM07D0034T (FL1) -> RFX2401C TXRX -> PA/LNA -> RFX2401C ANT -> SKY13373 -> ANT -> J1 U.FL`
- Sub-GHz TX: `LR1121 RFO_HP_LF -> IPD (T1) TX_HP -> SKY13373 -> ANT -> J1 U.FL`
- Sub-GHz RX: `J1 U.FL -> ANT -> SKY13373 -> IPD (T1) RX -> LR1121 RFI_P/N_LF`
- IPD TX_LP is disconnected: ELRS never uses the LR1121 LP PA
- RF switch and front-end are driven by the LR1121's own DIOs, not ESP32-C3 GPIOs: `DIO5 -> RFX2401C RXEN`, `DIO6 -> RFX2401C TXEN`, `DIO7 -> SKY13373 V1`, `DIO8 -> SKY13373 V2`

#### SKY13373 truth table (V1, V2)

| V1 | V2 | Path |
|---|---|---|
| 1 | 0 | 2.4 GHz TX/RX (RFX2401C ANT) |
| 0 | 1 | Sub-GHz TX HP (IPD TX_HP) |
| 1 | 1 | Sub-GHz RX (IPD RX) |
| 0 | 0 | Shutdown, antenna disconnected |

#### 2450FM07D0034 impedance note

Filter pin 1 (chipset side) is 40 ohm, designed for SX1280/SX1281. LR1121 RFIO_HF is 50 ohm. The mismatch gives ~19 dB return loss (VSWR 1.25), only 0.05 dB mismatch loss: negligible. The filter's own internal return loss (14 dB typ) dominates. Kept as-is.

#### Firmware

ELRS target, platform, upload methods and pin map: the [Firmware targets](#firmware-targets) and [Pin map](#pin-map) sections above, sourced from `shared/elrs-targets/OpenRX Mono LR1121.json`. Mono-specific settings in that JSON:

- `radio_rfsw_ctrl: [15, 0, 12, 8, 8, 6, 0, 5]`; each byte is a DIO5-DIO8 bitmask passed to `SetDioAsRfSwitch` (bit0 = DIO5 RXEN, bit1 = DIO6 TXEN, bit2 = DIO7 V1, bit3 = DIO8 V2)
- `radio_dcdc: true`
- Requires the ExpressLRS fork branch that enables the TCXO via `SetTcxoMode`; stock unified firmware does not bring up the 32 MHz TCXO on this board

#### rfsw_ctrl decode

| Index | Mode | Value | DIO5 (RXEN) | DIO6 (TXEN) | DIO7 (V1) | DIO8 (V2) |
|---|---|---|---|---|---|---|
| 0 | Enable | 15 | on | on | on | on |
| 1 | Standby | 0 | 0 | 0 | 0 | 0 |
| 2 | Sub-GHz RX | 12 | 0 | 0 | 1 | 1 |
| 3 | Sub-GHz TX LP | 8 | 0 | 0 | 0 | 1 |
| 4 | Sub-GHz TX HP | 8 | 0 | 0 | 0 | 1 |
| 5 | 2.4 GHz TX | 6 | 0 | 1 | 1 | 0 |
| 6 | unused | 0 | - | - | - | - |
| 7 | 2.4 GHz RX | 5 | 1 | 0 | 1 | 0 |

#### Flash interface

Pads and BOOT behaviour are the family default: [I/O pads and button](#io-pads-and-button). The Mono has no boot button, only the TP5 pad.

#### Sourcing

- `C150853` SKY13373-460LF
- `C19842466` 0900PC16J0042001E: consign from DigiKey, 0 LCSC stock
- `C7498014` LR1121IMLTRT
- `C2651081` 2450FM07D0034T
- `C783588` RFX2401C


### OpenRX-Gemini

Dual-LR1121 Gemini/Xrossband receiver. ESP32-C3 + 2x LR1121 + 2x RFX2401C + 2x SKY13373-460LF + 2x Johanson IPD.

Common circuit, antennas, I/O pads, pin map and firmware targets are shared, see above. This section carries only what is specific to the Gemini.

#### Board preview

| Front | Back |
|-------|------|
| ![Front](OpenRX-Gemini/../images/openrx-gemini-front.png) | ![Back](OpenRX-Gemini/../images/openrx-gemini-back.png) |

#### Schematic

- Top sheet: `OpenRX-Gemini.kicad_sch` -> `esp32-c3.kicad_sch` + `clock.kicad_sch` + `lr1121.kicad_sch` (instantiated twice). `esp32c3_lr1121_gemini.kicad_sch` is a legacy flat sheet, not in the hierarchy.
- Radio 1 (U3 LR1121, U4 RFX2401C, U5 SKY13373, T1 IPD, FL1 BPF): RF chain -> `J1` U.FL
- Radio 2 (U6 LR1121, U7 RFX2401C, U8 SKY13373, T2 IPD, FL2 BPF): mirrors radio 1 -> `J2` U.FL
- Per-radio 2.4 GHz and sub-GHz paths are identical to the [Mono](#openrx-mono), including the SKY13373 truth table
- Shared 32 MHz TCXO in `clock.kicad_sch`, powered from radio 2's (U6) VTCXO pin: U6 must initialize first or neither radio has a clock
- Each radio drives its own switch and front-end: `DIO5 -> RXEN`, `DIO6 -> TXEN`, `DIO7 -> V1`, `DIO8 -> V2`. Wiring is symmetric, so the single `radio_rfsw_ctrl` applies to both radios via `SetDioAsRfSwitch`.
- In DualBand/X modes the firmware never swaps radios: radio 1 (U3) is always sub-GHz, radio 2 (U6) is always 2.4 GHz. Antennas: `J1` = 900 MHz, `J2` = 2.4 GHz.

#### Firmware

Same binary as the Mono. ELRS target, platform, upload methods and pin map: the [Firmware targets](#firmware-targets) and [Pin map](#pin-map) sections above, sourced from `shared/elrs-targets/OpenRX Gemini LR1121.json`. Gemini-specific settings in that JSON:

- `radio_nss_2` enables dual-radio mode; radio 2 pins NSS 7, RST 10, BUSY 8, DIO1 18
- `radio_rfsw_ctrl: [15, 0, 12, 8, 8, 6, 0, 5]`, same as the Mono ([decode table](#rfsw_ctrl-decode))
- Requires the ExpressLRS fork branch: TCXO enable, radio-1 second reset (NRESET on strapping pin GPIO 2), and software chip-select for radio-1 NSS on GPIO 0

#### Flash interface

Pads are the family default: [I/O pads and button](#io-pads-and-button). Gemini delta: BOOT is a tactile button (U9, TS2306A) on GPIO 9 instead of the TP5 pad, held during power-up for UART download mode.

#### Sourcing

- `C150853` SKY13373-460LF (x2)
- `C19842466` 0900PC16J0042001E (x2, consign from DigiKey)
- `C7498014` LR1121IMLTRT (x2)
- `C2651081` 2450FM07D0034T (x2)
- `C783588` RFX2401C (x2)

