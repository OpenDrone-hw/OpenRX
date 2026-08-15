# OpenRX

Open source ExpressLRS receiver family. Four board variants share an ESP32-C3
core, a TLV75533 3.3 V LDO and the ExpressLRS unified firmware; they differ in
radio IC, frequency band, RF front end and antenna interface. Consumer specs
(dimensions, telemetry power, per-variant feature table) live in the
[README](README.md); this file is the technical write-up.

## Repo

| | |
|---|---|
| Maintainer | @bastian2001 |
| Status | See the `status-*` topic on the repo. Never written here. |
| Designed in | KiCad 10 |
| Layout | Multi-variant: one KiCad project per variant, each directory exactly one level below repo root, which is what makes the shared library and 3D model paths resolve |
| Variants | `OpenRX-Lite/`, `OpenRX-Lite-UFL/`, `OpenRX-Mono/`, `OpenRX-Gemini/`, each with `OpenRX-<Variant>.kicad_pro`, `.kicad_sch`, `.kicad_pcb`, `.kicad_dru` |
| Local library | `shared/libs/OpenRX-Shared.pretty/` and `.3dshapes/`, nickname `OpenRX-Shared`, referenced via `${KIPRJMOD}/../shared/libs/`. The sym-lib-tables still name `OpenRX-Shared.kicad_sym`, deleted from the repo: symbols exist only embedded in the design files and carry an `LCSC` property for JLCPCB BOM export; passives and some packages use stock KiCad footprints resolved through their embedded copies |
| Shared library | [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), catalogue only; every library this repo uses is local to the repo |
| Firmware targets | `shared/elrs-targets/`: per-variant ExpressLRS hardware JSON plus `targets_entries.json` |
| Fab | `OpenRX-<Variant>/fab/` holds the committed release sets; `production/` working exports are gitignored, regenerable with the Fabrication Toolkit (`fabrication-toolkit-options.json` per variant) |
| Board setup | Four boards, each 6 layers, 1.0 mm. Line standard: 0.09 mm clearance and track, via 0.35 on 0.20 drill |
| Archive | Five superseded designs live in git history, removed from the tip 2026-08-14, last at `archive/legacy-projects/`. `shared/sheets/` is a legacy sheet no variant instantiates |
| License | CERN-OHL-S-2.0 |

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
- **Reuse before you draw.** Check
  [KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library) and its
  `PARTS-USED.md` first. If the part is there we have already sourced,
  footprinted and shipped it: copy the symbol and footprint into this repo's
  `lib` library and use it. Draw a new part only when the library has nothing
  that fits, and import it with `easyeda2kicad` from its LCSC number.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **ERC and DRC clean before every pull request.** Commands below.

## Environment

```sh
# schematic and board checks, per variant touched
kicad-cli sch erc --exit-code-violations OpenRX-<Variant>/OpenRX-<Variant>.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones --exit-code-violations OpenRX-<Variant>/OpenRX-<Variant>.kicad_pcb

# netlist, for scripted analysis
kicad-cli sch export netlist --format kicadsexpr -o /tmp/<variant>.net OpenRX-<Variant>/OpenRX-<Variant>.kicad_sch
```

On macOS `kicad-cli` is at
`/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`, and `pcbnew` imports
only under KiCad's bundled Python. Shared scripts (renders, STEP export,
packaging art) live in `OpenDrone-Scripts`.

The design files are KiCad 10 format (`version 20260306`). Current kicad-skip
reads them fine (symbol fields, nets) but is not write-safe on this format: an
in-place metadata write reformats the entire file and can drop tokens it does
not parse. Until kicad-skip supports it, treat kicad-skip as read-only here and
make scripted edits through the pcbnew API or the KiCad GUI.

## Architecture

Every variant is the same core: an ESP32-C3 (U1, QFN-32 5 x 5 mm) runs the
ExpressLRS unified firmware, talks CRSF over its UART0, and drives a WS2812B
RGB status LED (D1). The 2450AT18A100E ceramic chip antenna (AE1, net `WIFI`)
is the ESP32-C3's own Wi-Fi antenna for OTA flashing and configuration on every
variant; the ELRS link antenna interface is per variant (RF chains below).
Clocks: 40 MHz crystal (X1) for the ESP32-C3 on all
variants; the radio TCXO is 52 MHz on the SX1281 variants and 32 MHz on the
LR1121 variants.

Sheets: Lite and Lite-UFL each carry their own copy of
`esp32c3_sx1281_lite.kicad_sch`, identical apart from the antenna termination;
Mono uses `esp32c3_lr1121_mono.kicad_sch`.
Gemini is hierarchical: `OpenRX-Gemini.kicad_sch` instantiates
`esp32-c3.kicad_sch`, `clock.kicad_sch` and `lr1121.kicad_sch` twice
(`esp32c3_lr1121_gemini.kicad_sch` in that directory is a legacy flat sheet,
not in the hierarchy).

RF chains:

- **Lite**: `SX1281 (U3) RFIO -> 2450FM07D0034T (FL1) -> 47948-0001 chip
  antenna (AE2)`. **Lite-UFL**: identical circuit, terminating in a U.FL (J1)
  instead; the filter output and U.FL are both 50 ohm. No PA/LNA, no RF switch,
  no sub-GHz on either.
- **Mono**, single LR1121 dual-band. 2.4 GHz: `LR1121 RFIO_HF -> FL1 ->
  RFX2401C (U4) PA/LNA -> SKY13373 (U5) -> J1 U.FL`. Sub-GHz TX: `LR1121
  RFO_HP_LF -> IPD (T1) TX_HP -> SKY13373 -> J1`. Sub-GHz RX: `J1 -> SKY13373
  -> IPD RX -> LR1121 RFI_P/N_LF`. The IPD's TX_LP port is unconnected: ELRS
  never uses the LR1121 low-power PA.
- **Gemini**, dual LR1121, two copies of the Mono chain: radio 1 (U3, U4, U5,
  T1, FL1) feeds `J1`, radio 2 (U6, U7, U8, T2, FL2) feeds `J2`. In dual-band
  modes the firmware never swaps radios: radio 1 is always sub-GHz (J1 takes
  the 900 MHz antenna), radio 2 always 2.4 GHz (J2). The single 32 MHz TCXO in
  `clock.kicad_sch` feeds both radios and is powered from 3.3 V, both VTCXO
  pins unconnected; the fabricated boards predate that supply change and power
  it from radio 2's (U6) VTCXO pin, so there U6 must initialise first or
  neither radio has a clock.

On the LR1121 variants the front end is driven by the radio's own DIOs, not
ESP32-C3 GPIOs: DIO5 = RFX2401C RXEN, DIO6 = RFX2401C TXEN, DIO7 = SKY13373 V1,
DIO8 = SKY13373 V2. Wiring is symmetric on Gemini, so one switch table serves
both radios (decode under [Firmware](#firmware)).

Known mismatch, kept as-is: 2450FM07D0034T pin 1 is 40 ohm, designed for
SX128x, while LR1121 RFIO_HF is 50 ohm. That gives ~19 dB return loss (VSWR
1.25) and 0.05 dB mismatch loss: negligible next to the filter's own 14 dB
typical return loss.

## Key parts

LCSC numbers from the schematic symbols and the verified `fab/` BOM sets. The
SKY13373 carries no LCSC field in either; its number was checked against LCSC
directly.

| Function | Ref | Part | LCSC | Fitted on |
|---|---|---|---|---|
| MCU | U1 | ESP32-C3 | | all |
| 3.3 V LDO | U2 | TLV75533PDQNR | C2861882 | all |
| 2.4 GHz radio | U3 | SX1281IMLTRT | C2151551 | Lite, Lite-UFL |
| Dual-band radio | U3, U6 | LR1121IMLTRT | C7498014 | Mono; Gemini x2 |
| PA/LNA | U4, U7 | RFX2401C | C19213 | Mono; Gemini x2 |
| RF switch | U5, U8 | SKY13373-460LF | C150853 | Mono; Gemini x2 |
| Sub-GHz balun/IPD | T1, T2 | 0900PC16J0042001E | C19842466 | Mono; Gemini x2 |
| 2.4 GHz band-pass filter | FL1, FL2 | 2450FM07D0034T | C2651081 | all; Gemini x2 |
| Wi-Fi antenna | AE1 | 2450AT18A100E | C89334 | all |
| ELRS chip antenna | AE2 | 47948-0001 | C152351 | Lite |
| ELRS U.FL | J1, J2 | U.FL-R-SMT-1(80) | C88374 | Lite-UFL, Mono; Gemini x2 |
| 40 MHz crystal | X1 | CJ17-400001010B20 | C2875272 | all |
| 52 MHz TCXO | OSC1 | OW7EL89CENUNFAYLC-52M | C22434896 | Lite, Lite-UFL |
| 32 MHz TCXO | OSC1 | OW7EL89CENUYO3YLC-32M | C22381772 | Mono, Gemini |
| Status LED | D1 | XL-1010RGBC-WS2812B | C5349953 | all |
| BOOT button | U9 | TS2306A | C2976675 | Gemini |

Sourcing: the 0900PC16J0042001E has no LCSC stock, consign it from DigiKey.
Watch SX1281 stock for volume runs.

## Power

```
5V pad (TP3)
└── TLV75533PDQNR (U2), 3.3 V, 500 mA
    ├── ESP32-C3 (U1), WS2812B (D1)
    ├── radio(s) and TCXO
    └── RF front end: RFX2401C, SKY13373 (Mono, Gemini)
```

## Connectors and I/O

Solder pads carry the external interface. Pad to net mapping from the
schematic netlists:

| Pad | Net | ESP32-C3 | Function |
|---|---|---|---|
| `RX` (TP1) | `U0RXD` | GPIO 20 | CRSF / serial in to RX |
| `TX` (TP2) | `U0TXD` | GPIO 21 | CRSF / serial out / telemetry |
| `5V` (TP3) | `+5V` | - | 5 V supply in to the LDO |
| `GND` (TP4) | `GND` | - | Ground |
| `BOOT` (TP5) | `BOOT` | GPIO 9 | Pull low at power-up for UART download mode |

On Lite, Lite-UFL and Mono, `BOOT` is the TP5 solder pad and there is no
physical switch. On Gemini the tactile button (U9) sits on `BOOT` alongside a
smaller 1.5 mm test pad (TP5); the other four pads remain.

GPIO assignments, from the target JSON in `shared/elrs-targets/`:

| Function | Lite / Lite-UFL | Mono | Gemini |
|---|---|---|---|
| Serial RX / TX | 20 / 21 | 20 / 21 | 20 / 21 |
| Radio SCK / MOSI / MISO | 6 / 4 / 5 | 6 / 4 / 5 | 6 / 4 / 5 |
| Radio NSS / RST | 7 / 2 | 7 / 2 | 0 / 2 |
| Radio BUSY / DIO1 | 3 / 1 | 3 / 1 | 3 / 1 |
| Radio 2 NSS / RST / BUSY / DIO1 | - | - | 7 / 10 / 8 / 18 |
| RF switch control | - | LR1121 DIO5-DIO8 | same, per radio |
| Status LED | 8 (GRB) | 8 (GRB) | 19 (GRB) |
| BOOT / button | 9 | 9 | 9 (button) |

## Firmware

The ExpressLRS hardware-target definitions live in this repo
(`shared/elrs-targets/`, with `targets_entries.json` prepared for upstream
submission; not yet merged into
[ExpressLRS/targets](https://github.com/ExpressLRS/targets)). The referenced
unified firmware images exist upstream:

| Variant | Product name | ELRS firmware target | Platform | Upload |
|---|---|---|---|---|
| Lite | OpenRX Lite 2.4GHz RX | `Unified_ESP32C3_2400_RX` | esp32-c3 | UART, Wi-Fi, Betaflight |
| Lite-UFL | OpenRX Lite-UFL 2.4GHz RX | `Unified_ESP32C3_2400_RX` | esp32-c3 | UART, Wi-Fi, Betaflight |
| Mono | OpenRX Mono Dual Band RX | `Unified_ESP32C3_LR1121_RX` | esp32-c3 | UART, Wi-Fi, Betaflight |
| Gemini | OpenRX Gemini XrossBand RX | `Unified_ESP32C3_LR1121_RX` | esp32-c3 | UART, Wi-Fi, Betaflight |

Minimum ExpressLRS version **3.5.0** (`min_version` in `targets_entries.json`).
Transmit power per variant is in the README table; the authoritative values
are `power_values` in the per-variant target JSON.

Stock unified firmware runs the SX1281 variants unmodified. Mono and Gemini
require an ExpressLRS fork branch: TCXO enable via `SetTcxoMode` on both, and
on Gemini also radio-1 second reset (NRESET on strapping pin GPIO 2) and
software chip-select for radio-1 NSS on GPIO 0. Mono and Gemini share one
binary; `radio_nss_2` in the Gemini JSON enables dual-radio mode. Both set
`radio_dcdc: true` and `radio_rfsw_ctrl: [15, 0, 12, 8, 8, 6, 0, 5]`: each
byte is a bitmask passed to `SetDioAsRfSwitch`, bit0 = DIO5 through bit3 =
DIO8. V1/V2 = 0/0 shuts the SKY13373 down and disconnects the antenna.

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

## Revisions

| Rev | Date | Change |
|---|---|---|
| rev2 | 2026-08-14 | Fab sets re-exported and verified against board and schematic (`Lite_rev2`, `Lite-UFL_rev2`, `Mono_rev2`, `Gemini_rev2`). Lite-UFL `J1` carries LCSC `C88374` on board and schematic, previously missing from the BOM. E6 substitute STEP models rebuilt from the trusted wrl geometry; schematic PDFs in `exports/schematics/` regenerated. |
| | 2026-08-05 | OSHWA certification (BE000030 to BE000033). Lite/Lite-UFL ELRS target pin remap, Mono/Gemini target updates, shared KiCad-Library submodule wired (2026-08-04). Layout rework (clock 3.3 V supply, enlarged pads, Lite/Lite-UFL and Mono outlines +1.0 mm) landed after the validated build and has not been fabricated. |
| rev1 | 2026-06-10 | Combined `OpenRX-all` fabrication set ordered at JLCPCB (gerbers, BOM, CPL in `OpenRX-Gemini/`). |
| | 2026-06-07 | Single-source-of-truth docs pass, standardized board renders. |
| | 2026-03-23 | Initial repo, 6-receiver lineup; later reduced to the four current variants (retired designs in git history). |
