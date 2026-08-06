# OpenRX

Open-source **ExpressLRS (ELRS) receiver** line for FPV/RC, part of the incutec OpenDrone line. Four board variants share an **Espressif ESP32-C3** core, a TLV75533 3.3 V LDO, and the ExpressLRS unified firmware; they differ in radio IC, frequency band, RF front-end, and ELRS antenna. All four are 6-layer boards designed in KiCad 10 for JLCPCB assembly. Family design detail: [DESIGN.md](DESIGN.md). Per-variant notes: `OpenRX-<variant>/DESIGN.md`. Flashing and debug: [FLASHING.md](FLASHING.md).

| | | | |
|:---:|:---:|:---:|:---:|
| **Lite** | **Lite-UFL** | **Mono** | **Gemini** |
| <img src="images/openrx-lite-front.png" width="200" alt="OpenRX-Lite front" /> | <img src="images/openrx-lite-ufl-front.png" width="200" alt="OpenRX-Lite-UFL front" /> | <img src="images/openrx-mono-front.png" width="200" alt="OpenRX-Mono front" /> | <img src="images/openrx-gemini-front.png" width="200" alt="OpenRX-Gemini front" /> |
| <img src="images/openrx-lite-back.png" width="200" alt="OpenRX-Lite back" /> | <img src="images/openrx-lite-ufl-back.png" width="200" alt="OpenRX-Lite-UFL back" /> | <img src="images/openrx-mono-back.png" width="200" alt="OpenRX-Mono back" /> | <img src="images/openrx-gemini-back.png" width="200" alt="OpenRX-Gemini back" /> |
| SX1281, 2.4 GHz, chip antenna | SX1281, 2.4 GHz, U.FL | LR1121, dual-band, U.FL | 2x LR1121, dual-radio, 2x U.FL |

## Status

**Hardware validated**, 2026-08-05, all four variants (Lite, Lite-UFL, Mono, Gemini).
Latest fabrication set is the combined `OpenRX-all` board (ordered 2026-06-10), tracked in `OpenRX-Gemini/fab/` and `OpenRX-Gemini/export/`.

## Certification

<a href="https://certification.oshwa.org/list.html">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="images/oshwa-certified-dark.svg" />
    <img src="images/oshwa-certified.svg" width="160" alt="OSHWA Certified Open Source Hardware" />
  </picture>
</a>

All four variants are **certified open source hardware** by the [Open Source Hardware Association](https://www.oshwa.org/). Each board carries its own OSHWA UID:

| Variant | OSHWA UID |
|---|---|
| **Lite** | [BE000030](https://certification.oshwa.org/be000030.html) |
| **Lite-UFL** | [BE000031](https://certification.oshwa.org/be000031.html) |
| **Mono** | [BE000032](https://certification.oshwa.org/be000032.html) |
| **Gemini** | [BE000033](https://certification.oshwa.org/be000033.html) |

Build video: [How LoRa (ExpressLRS) Drone Receivers Work](https://www.youtube.com/watch?v=ssmQkRkXE84)

## Links

- Product page: [opendrone.be/products/openrx](https://opendrone.be/products/openrx)
- Video channel: [JustFPV on YouTube](https://www.youtube.com/@justfpv1432)

## Specifications

| Parameter | Lite | Lite-UFL | Mono | Gemini |
|---|---|---|---|---|
| Radio IC | Semtech SX1281 | Semtech SX1281 | Semtech LR1121 | 2x Semtech LR1121 |
| Band | 2.4 GHz | 2.4 GHz | Sub-GHz + 2.4 GHz | Sub-GHz + 2.4 GHz, dual-radio (Xrossband) |
| ELRS antenna | Chip antenna (AE2) | U.FL (J1) | U.FL (J1) | 2x U.FL (J1, J2) |
| RF front-end | Band-pass filter | Band-pass filter | PA/LNA + RF switch + balun + BPF | 2x (PA/LNA + RF switch + balun + BPF) |
| Max TX power | 13 dBm | 13 dBm | 12-22 dBm | 12-22 dBm per radio |
| Board size | 10.0 x 10.5 mm | 10.0 x 10.5 mm | 10.0 x 16.3 mm | 17.0 x 15.7 mm |
| PCB | 6-layer, 1.0 mm | 6-layer, 1.0 mm | 6-layer, 1.0 mm | 6-layer, 1.0 mm |

Common to all variants: ESP32-C3 MCU, 5 V pad input, CRSF over UART0, WS2812B status LED, dedicated 2.4 GHz Wi-Fi chip antenna for OTA. Part-level detail (LDO, clocks, RF chains, pads, pin maps, firmware targets) is in [DESIGN.md](DESIGN.md).

## Repository layout

| Path | Contents |
|---|---|
| `OpenRX-Lite/` | KiCad 10 project: SX1281 2.4 GHz, on-board chip antenna |
| `OpenRX-Lite-UFL/` | KiCad 10 project: SX1281 2.4 GHz, U.FL |
| `OpenRX-Mono/` | KiCad 10 project: single LR1121 dual-band, U.FL |
| `OpenRX-Gemini/` | KiCad 10 project: dual LR1121, 2x U.FL; also the combined `OpenRX-all` board with its `fab/` gerber set |
| `shared/libs/` | `OpenRX-Shared` symbol/footprint/3D library (in-repo) |
| `shared/sheets/` | Shared hierarchical sheet (RX core, legacy) |
| `shared/elrs-targets/` | ExpressLRS hardware-target JSON + `targets_entries.json` |
| `libs/KiCad-Library` | Shared Incutec symbol/footprint/3D library (git submodule) |
| `images/` | Board renders and certification marks |
| `exports/` | Schematic PDFs (per variant + ELRS-team drops) |
| `verification/` | BOM and design-verification scripts |
| `archive/legacy-projects/` | Retired designs (Nano, 900, PWM, Dual) |
| `DESIGN.md` | Family design notes |
| `FLASHING.md` | Flashing and debug guide |

Each variant directory holds the KiCad project (`.kicad_pro`/`.kicad_sch`/`.kicad_pcb`), a `DESIGN.md`, render `images/`, and an `export/` with BOM, STEP, or schematic PDF output; `OpenRX-Gemini/fab/` additionally holds the ordered gerber/drill set.

## Design entry points

- **Lite / Lite-UFL**: `OpenRX-Lite/OpenRX-Lite.kicad_sch` and `OpenRX-Lite-UFL/OpenRX-Lite-UFL.kicad_sch` (top) -> `esp32c3_sx1281_lite.kicad_sch` (ESP32-C3 + SX1281 + LDO + RGB LED)
- **Mono**: `OpenRX-Mono/OpenRX-Mono.kicad_sch` (top) -> `esp32c3_lr1121_mono.kicad_sch`
- **Gemini**: `OpenRX-Gemini/OpenRX-Gemini.kicad_sch` (top) -> `esp32-c3.kicad_sch` + `clock.kicad_sch` + `lr1121.kicad_sch` (instantiated twice)
- Board layouts: `OpenRX-<variant>/OpenRX-<variant>.kicad_pcb`, 6 copper layers; combined production board `OpenRX-Gemini/OpenRX-all.kicad_pcb`

Symbols and footprints are embedded in the design files, so the schematics and boards open without any external library. The project lib tables reference the in-repo `shared/libs/OpenRX-Shared.*` library plus the shared `Incutec` library from the `libs/KiCad-Library` submodule, used for new parts. Passives and some packages (coax connectors, QFNs) use stock KiCad library footprints that resolve through their embedded copies. Symbols carry an `LCSC` property for JLCPCB BOM export.

## Build and export

```
git clone --recursive https://github.com/incutec-hw/OpenRX.git
```

Open a variant's `.kicad_pro` in KiCad 10. Production exports (gerbers, BOM, CPL) are generated with the [KiCad Fabrication Toolkit](https://github.com/bennymeg/Fabrication-Toolkit) plugin, using the tracked per-variant `fabrication-toolkit-options.json`. Headless checks and exports use `kicad-cli`:

```
kicad-cli sch erc --exit-code-violations OpenRX-Lite/OpenRX-Lite.kicad_sch
kicad-cli pcb drc --exit-code-violations OpenRX-Lite/OpenRX-Lite.kicad_pcb
kicad-cli pcb export gerbers -o out/ OpenRX-Lite/OpenRX-Lite.kicad_pcb
```

## Manufacturing

Fabricated and assembled at JLCPCB: 6-layer, 1.0 mm boards, LCSC parts. The validated build was ordered as the combined `OpenRX-all` board (all four variants on one PCB): gerbers in `OpenRX-Gemini/fab/`, BOM and placement in `OpenRX-Gemini/export/`. Revision history: [CHANGELOG.md](CHANGELOG.md).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Hardware licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt). See [LICENSE](LICENSE).
