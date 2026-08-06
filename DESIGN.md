# OpenRX Design Notes

Family-level design description of the four OpenRX variants. Values are extracted from the KiCad design files and the ExpressLRS target JSON in `shared/elrs-targets/`. Per-variant RF-chain detail: `OpenRX-<variant>/DESIGN.md`. RF-switch state decode: the [rfsw_ctrl decode](OpenRX-Mono/DESIGN.md#rfsw_ctrl-decode) table in OpenRX-Mono/DESIGN.md.

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

On the LR1121 variants the RF switch and front-end are driven by the radio's own DIO pins, not ESP32-C3 GPIOs: DIO5 = RFX2401C RXEN, DIO6 = RFX2401C TXEN, DIO7 = SKY13373 V1, DIO8 = SKY13373 V2. Each `radio_rfsw_ctrl` byte is a DIO5-DIO8 bitmask passed to `SetDioAsRfSwitch`; the decode table is in [OpenRX-Mono/DESIGN.md](OpenRX-Mono/DESIGN.md#rfsw_ctrl-decode).

Transmit power per variant is in the [Specifications](README.md#specifications) table; the authoritative values are `power_values` in the per-variant target JSON.

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

Symbols and footprints are embedded in the design files. The project lib tables reference the in-repo `shared/libs/OpenRX-Shared.*` library (`${KIPRJMOD}/../shared/libs/`) and the shared `Incutec` library from the `libs/KiCad-Library` git submodule (`${KIPRJMOD}/../libs/KiCad-Library/`). Passives and some packages (coax connectors, QFNs) use stock KiCad library footprints resolved through their embedded copies. Symbols carry an `LCSC` property for JLCPCB BOM export.

## Revisions

- **2026-08-05**: hardware validated, all four variants. OSHWA certification (BE000030 to BE000033), Lite/Lite-UFL ELRS target pin remap, Mono/Gemini target updates, shared Incutec KiCad-Library submodule wired (2026-08-04). Layout rework (clock 3.3 V supply, enlarged pads, Lite/Lite-UFL and Mono outlines +1.0 mm) landed after the validated build and has not been fabricated.
- **2026-06-10**: combined `OpenRX-all` fabrication set ordered at JLCPCB (gerbers, BOM, CPL in `OpenRX-Gemini/`).
- **2026-06-07**: single-source-of-truth docs pass, standardized board renders.
- **2026-03-23**: initial repo, 6-receiver lineup; later reduced to the four current variants (retired designs in `archive/legacy-projects/`).
