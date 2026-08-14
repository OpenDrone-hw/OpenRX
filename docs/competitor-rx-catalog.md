# ELRS ESP32-C3 Receiver Competitor Catalog

Source: `https://artifactory.expresslrs.org/ExpressLRS/hardware.zip` (the artifact the Configurator pulls). Snapshot taken 2026-04-30.

**Total ESP32-C3 RX SKUs shipped to end users: 55**

Categorized by RF architecture so each OpenRX SKU has direct apples-to-apples comparison.

---

## Gemini-class — C3 + dual LR1121 (Xrossband)

**OpenRX equivalent:** **OpenRX-Gemini** — same architecture (2× LR1121 + 2× RFX2401C + 2× SKY13414)

**Count: 10**

| Vendor | Product | Layout | Pin overlay | Min FW |
|---|---|---|---|---|
| BAYCKRC | BAYCKRC C3 900/2400 Dual Band 100mW Gemini RX | LR1121 True Diversity | — | 3.5.0 |
| BAYCKRC | BAYCKRC RRD2 2.4GHz RX | LR1121 True Diversity | — | 3.5.0 |
| BETAFPV | BETAFPV SuperX Nano RX | LR1121 True Diversity | — | 3.5.0 |
| BrotherHobby | BrotherHobby C3 900/2400 Dual Band 100mW Gemini RX | LR1121 True Diversity | — | 3.5.0 |
| DAKEFPV | DAKEFPV 2G4 SuperD Pro MAX 500mW RX | LR1121 True Diversity | — | 3.5.0 |
| DAKEFPV | DAKEFPV C3 900/2400 Dual Band 100mW Gemini RX | LR1121 True Diversity | — | 3.6.0 |
| DAKEFPV | DAKEFPV SuperD 2.4GHz RX | LR1121 True Diversity | — | 3.5.0 |
| DAKEFPV | DAKEFPV SuperD 900MHz RX | LR1121 True Diversity | — | 3.5.0 |
| GEPRC | GEPRC C3 900/2400 Gemini Xrossband RX | LR1121 True Diversity | — | 3.5.2 |
| HGLRC | HGLRC C3 900/2400 Dual Band PRO RX | LR1121 True Diversity | — | 3.6.0 |

All 10 use Generic C3 LR1121 True Diversity pinout with **zero pin deviations**. Strongest reference: **BETAFPV SuperX Nano** (matches OpenRX-Gemini chip-for-chip).

---

## Mono-class — C3 + single LR1121 (dual-band)

**OpenRX equivalent:** **OpenRX-Mono** — LR1121 + RFX2401C + SKY13414, 2.4GHz + 900MHz

**Count: 24**

| Vendor | Product | Layout | Pin overlay | Min FW |
|---|---|---|---|---|
| BAYCKRC | BAYCKRC C3 900/2400 Dual Band 100mw 6PWM RX | LR1121 PWM | — | 3.5.0 |
| BAYCKRC | BAYCKRC C3 900/2400 Dual Band Nano RX | LR1121 | — | 3.5.0 |
| BAYCKRC | BAYCKRC RR2 2.4GHz RX | LR1121 | `serial1_rx=18`, `serial1_tx=19` | 3.5.0 |
| BAYCKRC | BAYCKRC UR100 Dual Band RX | LR1121 | `serial1_rx=18`, `serial1_tx=19` | 3.5.0 |
| BAYCKRC | BAYCKRC UR1000 Dual Band RX | LR1121 | `serial1_rx=18`, `serial1_tx=19` | 3.5.0 |
| BAYCKRC | BAYCKRC UR500 Dual Band RX | LR1121 | `serial1_rx=18`, `serial1_tx=19` | 3.5.0 |
| BETAFPV | BETAFPV SuperX Mono RX | LR1121 | `radio_nss=0`, `led_rgb=19` | 3.5.0 |
| DAKEFPV | DAKEFPV 1W 2.4GHz RX | LR1121 | — | 3.5.0 |
| DAKEFPV | DAKEFPV 500mW 2.4GHz RX | LR1121 | — | 3.5.0 |
| DAKEFPV | DAKEFPV 500mW 900MHz RX | LR1121 | `ant_ctrl=10` | 3.5.0 |
| DAKEFPV | DAKEFPV 900MHz MAX 1000mW Diversity RX | LR1121 | `ant_ctrl=10` | 3.6.0 |
| DAKEFPV | DAKEFPV 900MHz NANOPRO MAX 500mW RX | LR1121 | — | 3.6.0 |
| DAKEFPV | DAKEFPV Nano 2.4GHz RX | LR1121 | — | 3.5.0 |
| DAKEFPV | DAKEFPV Nano 900MHz RX | LR1121 | — | 3.5.0 |
| GEPRC | GEPRC 900/2400 Single Dual-Band RX | LR1121 | — | 3.5.2 |
| HGLRC | HGLRC C3 900/2400 Dual Band Nano RX | LR1121 | — | 3.6.0 |
| RadioMaster | RadioMaster XR1 Dual Band RX | LR1121 | `serial1_rx=18`, `serial1_tx=19` | 3.5.0 |
| RadioMaster | RadioMaster XR2 2.4GHz RX | LR1121 | `serial1_rx=18`, `serial1_tx=19`, `led=8`, `led_rgb=-1` | 3.5.0 |
| RadioMaster | RadioMaster XR3 Dual Band Diversity RX | LR1121 | `serial1_rx=18`, `serial1_tx=19`, `ant_ctrl=10` | 3.5.0 |
| Spedix | Spedix 900MHz RX | LR1121 | `radio_nss=0`, `led=19`, `led_rgb=-1` | 3.5.0 |
| Spedix | Spedix Dual Band RX | LR1121 | `radio_nss=0`, `led_rgb=19` | 3.5.0 |
| Sub250 | Sub250 900/2400 Single Dual-Band RX | LR1121 | — | 3.5.2 |
| THOBBY | THOBBY 900MHz RX | LR1121 | `radio_nss=0`, `led=19`, `led_rgb=-1` | 3.5.0 |
| THOBBY | THOBBY Dual Band RX | LR1121 | `radio_nss=0`, `led_rgb=19` | 3.5.0 |

All 24 share the Generic C3 LR1121 SPI/UART/RST/BUSY/DIO1 core. Common deviations:
- `serial1_rx=18, serial1_tx=19` — adds second UART on FE control pins (BAYCKRC, RadioMaster)
- `radio_nss=0, led_rgb=19` — diversity-style NSS+LED swap (BETAFPV, Spedix, THOBBY)
- `ant_ctrl=10` — SPDT antenna switch control (DAKEFPV, RadioMaster XR3)

---

## Lite-class with FE — C3 + SX1280/SX1281 + PA (Generic C3 2400 PA)

**OpenRX equivalent:** **OpenRX-Lite (UFL+amp variant)** — only if you add the FE; current Lite has no FE

**Count: 8**

| Vendor | Product | Layout | Pin overlay | Min FW |
|---|---|---|---|---|
| DeepSpace | DEEPSPACE 2.4GHz Nano 100mw RX | 2400 PA | — | 3.5.0 |
| NewBeeDrone | NewBeeDrone Diversity 2.4Ghz RX V2 | 2400 PA | `power_rxen=-1`, `ant_ctrl=18` | 3.5.0 |
| ORBIT | ORBIT 2.4Ghz Nano RX | 2400 PA | — | 3.5.0 |
| Oxbot | Oxbot 2.4GHz RX | 2400 PA | — | 3.5.0 |
| SkyGuy | SkyGuy Max 2.4GHz RX | 2400 PA | — | 3.5.0 |
| SkyGuy | SkyGuy Nano 2.4GHz RX | 2400 PA | — | 3.5.0 |
| SpeedyBee | Speedybee AIO 2.4Ghz RX | 2400 PA | — | 3.5.0 |
| TuneRC | TuneRC 2.4G nano PA RX | 2400 PA | — | 3.5.0 |

---

## Lite-class no FE — C3 + SX1280/SX1281 bare (Generic C3 2400)

**OpenRX equivalent:** **OpenRX-Lite (ceramic + UFL)** — direct match

**Count: 7**

| Vendor | Product | Layout | Pin overlay | Min FW |
|---|---|---|---|---|
| BotLabDynamics | BotLabDynamics BotLink1 2.4GHz RX | 2400 | `led_rgb=-1`, `led=8` | 3.5.0 |
| Flycolor | Flycolor 2.4GHz Nano RX | 2400 | — | 3.5.0 |
| HDZero | HDZero 2.4GHz AIO RX | 2400 | — | 3.5.0 |
| NewBeeDrone | NewBeeDrone 2.4Ghz RaceSpec RX | 2400 | — | 3.5.0 |
| OMPHOBBY | OMPHOBBY OFS3+ 2.4 GHz RX | 2400 | — | 3.5.0 |
| Spedix | Spedix 2.4GHz RX | 2400 | `led_rgb=-1`, `led=19` | 3.5.0 |
| THOBBY | THOBBY 2.4GHz RX | 2400 | `led_rgb=-1`, `led=19` | 3.5.0 |

---

## Lite-class True Diversity — C3 + 2× SX1281 (Generic C3 2400 True Diversity)

**Count: 3**

| Vendor | Product | Layout | Pin overlay | Min FW |
|---|---|---|---|---|
| BAYCKRC | BAYCKRC RRD1 2.4GHz RX | 2400 True Diversity | — | 3.5.0 |
| GEPRC | GEPRC Nano True Diversity 2.4GHz RX | 2400 True Diversity | — | 3.5.0 |
| HDZero | HDZero Halo FC 2.4GHz Gemini RX | 2400 True Diversity | `led_rgb=-1`, `led=19` | 3.5.0 |

---

## Lite-class with PWM — C3 + SX1281 + PWM outputs (Generic C3 2400 PWM)

**Count: 2**

| Vendor | Product | Layout | Pin overlay | Min FW |
|---|---|---|---|---|
| BAYCKRC | BAYCKRC C3 2.4Ghz 6PWM 10mw RX | 2400 PWM | — | 3.5.0 |
| Jumper | Jumper AION P6 2.4GHz RX | 2400 PWM | — | 3.5.0 |

---

## Convention summary

- **No vendor ships a wholly custom C3 hardware.json file.** All 54 products reference one of 8 Generic C3 layouts and customize via the `overlay` mechanism in targets.json.
- Generic vendor entries are **stripped from the shipped artifact** — end users cannot select Generic targets. The Generic JSONs serve as base layouts only.
- For OpenRX: ship our own product names in targets.json, reference the Generic C3 layouts, customize via overlay (power tables, radio_rfsw_ctrl, optional ant_ctrl/serial1).

## Source verification

Pull and re-run anytime:
```bash
curl -sL https://artifactory.expresslrs.org/ExpressLRS/hardware.zip -o /tmp/elrs-hw.zip
unzip -p /tmp/elrs-hw.zip targets.json | python3 -c "import json,sys; d=json.load(sys.stdin); [print(k) for k in sorted(d.keys())]"
```
