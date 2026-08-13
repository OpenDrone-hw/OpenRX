# OpenRX

Open source **ExpressLRS receiver** family for FPV, part of the incutec
OpenDrone line. Four variants share an ESP32-C3 core and the ExpressLRS unified
firmware, and differ in radio, band, RF front end and antenna.

<p>
<img src="images/openrx-lite-front.png" width="200" alt="OpenRX-Lite" />
<img src="images/openrx-lite-ufl-front.png" width="200" alt="OpenRX-Lite-UFL" />
<img src="images/openrx-mono-front.png" width="200" alt="OpenRX-Mono" />
<img src="images/openrx-gemini-front.png" width="200" alt="OpenRX-Gemini" />
</p>

| Variant | Radio | Band | Antenna |
|---|---|---|---|
| **Lite** | SX1281 | 2.4 GHz | Chip antenna |
| **Lite-UFL** | SX1281 | 2.4 GHz | U.FL |
| **Mono** | LR1121 | Dual band | U.FL |
| **Gemini** | 2x LR1121 | Dual band, Xrossband | 2x U.FL |

All four are 6-layer boards, 5 V input, CRSF over UART.

<a href="https://certification.oshwa.org/list.html">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="images/oshwa-certified-dark.svg" />
    <img src="images/oshwa-certified.svg" width="160" alt="OSHWA Certified Open Source Hardware" />
  </picture>
</a>

Certified open source hardware by the [Open Source Hardware Association](https://www.oshwa.org/),
one UID per variant:
[Lite BE000030](https://certification.oshwa.org/be000030.html) ·
[Lite-UFL BE000031](https://certification.oshwa.org/be000031.html) ·
[Mono BE000032](https://certification.oshwa.org/be000032.html) ·
[Gemini BE000033](https://certification.oshwa.org/be000033.html)

## In the line

Pairs with any OpenDrone flight controller over the 4-pin receiver connector:

- [OpenFC-Lite](https://github.com/OpenDrone-hw/OpenFC-Lite), 30.5 x 30.5 mm
- [OpenFC-Lite-Mini](https://github.com/OpenDrone-hw/OpenFC-Lite-Mini), 20 x 20 mm

Firmware is [ExpressLRS](https://github.com/ExpressLRS/ExpressLRS) upstream, not
a fork. Targets are declared in `shared/elrs-targets/`.

## Get one

[opendrone.be](https://opendrone.be)

Build video: [How LoRa (ExpressLRS) Drone Receivers Work](https://www.youtube.com/watch?v=ssmQkRkXE84)
on [JustFPV](https://www.youtube.com/@justfpv1432)

## Contributing

Issues and pull requests are welcome on any repo. KiCad files cannot be merged,
so say what you intend to change before you do, on
[Discord](https://discord.gg/v3sWmTcx3R).

The design itself, the part list and the layout constraints are in
[AGENTS.md](AGENTS.md). How everything works:
[CONTRIBUTING.md](CONTRIBUTING.md).

## License

Hardware licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt),
see [LICENSE](LICENSE).
