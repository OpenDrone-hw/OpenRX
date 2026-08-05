# AntAIO — Verified Change List

1–2S antweight AIO (ESP32-C3 + SX1281 ELRS/ESPNOW, 4×N20 tank drive via 2 paired
DRV8212 + 1 weapon DRV8212, LSM6DS3 IMU). Forked from OpenRX-Lite.

Status as of 2026-06-18. Items below are **verified against the netlist** — open
items still need doing in KiCad; "already fixed" is for reference.

---

## Open, to do in KiCad

### 🔴 Critical
- [ ] **Swap PH↔EN on both drive drivers (boot-safety — motors run at power-on otherwise).**
  EN currently sits on pins driven HIGH at reset (USB D+ pull-up / UART TX), so the
  bridges enable before firmware. Move EN to the quiet pins, PH to the driven pins:
  - Left (U6):  **IN2/EN → GPIO18 (D-)**, **IN1/PH → GPIO19 (D+)**   *(swap of today's 19/18)*
  - Right (U10): **IN2/EN → GPIO20 (U0RXD)**, **IN1/PH → GPIO21 (U0TXD)** *(swap of today's 21/20)*
  - Weapon (U11) already safe (EN=GPIO10, no pull) — leave it.
  - Update the firmware target pin map to match after the swap.
- [ ] **Tie SX1281 (U3) pin 5 GND → GND** (currently floating; ERC missed it — pin type unspecified).

### 🟠 Important
- [ ] **Add a local 0.1 µF on each driver's VM and VCC** — U6, U10, U11 (6 caps total),
  placed at the pins. Today only distant bulk on +BATT.
- [ ] **Add a TVS on +BATT** (~9 V standoff, <11 V clamp) for inductive spikes / mis-plug.
  DRV8212 VM is 2S-max — **document the board as 2S-max** (3S destroys all drivers).
- [ ] **Add 2 UART flashing pads: GPIO20 (U0RXD) + GPIO21 (U0TXD).** With the existing
  BOOT button + GND (TP7) + a 3V3 tap, that's a full flash path (hold BOOT, power-cycle).
  Flash with motor battery OFF — these pins are also the right-drive lines.

### 🟡 Verify / minor
- [ ] **Add 100 nF on the VSENSE node → GND** (GPIO0). Today unfiltered + ~8 kΩ high-Z into the ADC.
- [ ] **Reconcile the ELRS antenna (AE2)** — netlist has `2450AT18A100E`, docs say Molex
  `47948-0001`, and there's **no match network**. Pick one and add the matching network.
- [ ] (cosmetic) ESP32-C3 symbol labels pins 4/5 as `XTAL_32K` — they are GPIO0/GPIO1. Relabel for clarity.

---

## Already fixed (verified in schematic)
- ✅ P-FET Q2 orientation correct (battery→Source, loads→Drain, Zener clamp on battery side).
- ✅ 3× DRV8212 placed + wired (2 drive paired, 1 weapon); MODE→3V3 (PH/EN); weapon PH→GND.
- ✅ SX1281 NRESET → CHIP_EN (radio resets with MCU; `radio_rst` undefined in firmware — confirmed safe in ELRS source).
- ✅ IMU SDO/SA0 → MISO (was on MOSI); shared SPI bus, CS=GPIO2, NSS=GPIO7.
- ✅ Power tree: AON7407 reverse P-FET → TPS63070 buck-boost → 5 V → TLV75533 LDO → 3V3.
  FB 52.3k/10k ≈ 5.0 V; EN pull-up; PS/SYNC→GND (forced PWM); VAUX cap; FB2→GND (datasheet-OK).
- ✅ VSENSE divider 52.3k/10k → GPIO0 (ADC). NSS pull-up (10k→3V3) on GPIO2 strapping.

## Decisions (do NOT change)
- **Keep the 5 V rail at 5 V.** It powers the servo/BLDC weapon output — do not retarget the
  buck-boost to ~4 V (the ~34 % LDO loss is small in absolute terms at ~0.4 A logic).
- **LDO thermal:** fine for ELRS (WiFi off). For **ESPNOW mode** (continuous WiFi) the 1×1 mm
  TLV75533 runs warm dropping 5→3.3 V — give it copper pour; no part change needed.
- **VSENSE stays on GPIO0, IMU_CS on GPIO2** — GPIO0 is the only non-strapping ADC pin; a divider
  on the GPIO2 strapping pin would break boot.

## Reference — final 15-pin map (ESP32-C3)
`0`=VSENSE · `1`=DIO1 · `2`=IMU_CS(+pull-up) · `3`=BUSY · `4`=MOSI · `5`=MISO · `6`=SCK ·
`7`=NSS · `8`=WS2812 · `9`=BOOT · `10`=WEAPON_EN · `18/19`=Left EN/PH* · `20/21`=Right EN/PH* ·
SX1281 RST→CHIP_EN. *(EN/PH order flips after the boot-safety swap above.)*
