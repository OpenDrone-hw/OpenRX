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

[![Status](https://img.shields.io/badge/status-alpha-e08c00)](https://github.com/OpenDrone-hw/.github/blob/main/CONTRIBUTING.md#the-life-of-a-project)
[![Shop](https://img.shields.io/badge/shop-opendrone.be-ffb700)](https://opendrone.be/products/openrx)
[![Discord](https://img.shields.io/badge/Discord-%23receivers-5865F2?logo=discord&logoColor=white)](https://discord.com/channels/1494019459822653512/1494758332903456969)
[![Video](https://img.shields.io/badge/YouTube-How%20ExpressLRS%20Receivers%20Work-FF0000?logo=youtube&logoColor=white)](https://www.youtube.com/watch?v=ssmQkRkXE84)

## Specifications

| | Lite | Lite-UFL | Mono | Gemini |
|---|---|---|---|---|
| Band | 2.4 GHz | 2.4 GHz | Dual band | Dual band, Xrossband |
| Radio | SX1281 | SX1281 | LR1121 | 2x LR1121 |
| Antenna | On-board ceramic | U.FL | U.FL | 2x U.FL |
| Telemetry power | 13 dBm (20 mW) | 13 dBm (20 mW) | | |
| Protocol | CRSF | CRSF | CRSF | CRSF |
| MCU | ESP32-C3 | ESP32-C3 | ESP32-C3 | ESP32-C3 |
| Input | 5 V pad | 5 V pad | 5 V pad | 5 V pad |
| Firmware | ExpressLRS | ExpressLRS | ExpressLRS | ExpressLRS |
| Flashing | Betaflight passthrough or Wi-Fi | Betaflight passthrough or Wi-Fi | Betaflight passthrough or Wi-Fi | Betaflight passthrough or Wi-Fi |
| Wi-Fi antenna | Separate on-board ceramic | Separate on-board ceramic | Separate on-board ceramic | Separate on-board ceramic |
| Dimensions | 10.0 x 11.5 mm | 10.0 x 11.5 mm | | |
| PCB | 6-layer, 1.6 mm | 6-layer, 1.6 mm | 6-layer | 6-layer |

OSHWA certified, one UID per variant:
[Lite BE000030](https://certification.oshwa.org/be000030.html) ·
[Lite-UFL BE000031](https://certification.oshwa.org/be000031.html) ·
[Mono BE000032](https://certification.oshwa.org/be000032.html) ·
[Gemini BE000033](https://certification.oshwa.org/be000033.html)

Technical write-up, part list and layout constraints: [AGENTS.md](AGENTS.md).

## In the line

What pairs with what, and what is available:
[opendrone.be](https://opendrone.be).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Hardware licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt),
see [LICENSE](LICENSE).
