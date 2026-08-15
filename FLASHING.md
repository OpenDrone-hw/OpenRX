# OpenRX Flashing & Debug Guide

Single reference for flashing and debugging every board on the bench: the four OpenRX
receivers, the OpenFC-Lite flight controller, and the GX12 transmitter. Procedures and
facts only. This file absorbs the former BRINGUP.md and BRINGUP-DEBUG.md.

---

## 1. Shared facts (all OpenRX receivers)

- **Firmware**: local ExpressLRS 4.0 fork at `~/Documents/GitHub/ExpressLRS`, branch
  `opendrone-lr1121-dual-c3-fixes`. The fork is **required** on Mono and Gemini: stock
  ELRS never runs `SetTcxoMode`, the TCXO stays unpowered, and the LR1121 passes SPI
  detection (internal RC oscillator) but does no RF. Fix list in §10. All commands run
  from the fork's `src/` directory.
- **Bind phrase**: `your-bind-phrase` (UID derived from it), baked at configure time.
  The RX boots already bound; there is no bind step.
- **CRSF baud**: `--rx-baud 420000`. Betaflight's CRSF driver is fixed at 420000.
- **Regulatory domain, critical**: `user_defines.txt` sets a build-time default
  (`-DRegulatory_Domain_EU_868` → domain 2), but `binary_configurator.py` **replaces
  the entire options block** and only writes `domain` when `--domain` is passed on the
  command line. Omit it and the deployed firmware falls back to
  `doc["domain"] | 0` = **AU915**: 2.4 GHz still works, sub-GHz never syncs against an
  EU868 TX. **Always pass `--domain eu_868` when baking LR1121 firmware.** Verify any
  baked binary before flashing:

  ```sh
  python3 -c "import re,sys;d=open(sys.argv[1],'rb').read();print([m.group() for m in re.finditer(rb'\{[^{}\x00]{20,800}\}',d) if b'uid' in m.group()])" firmware.bin
  ```

  The printed JSON must contain `"domain": 2` (EU868).

  | index | domain | `--domain` value |
  |---|---|---|
  | 0 | AU915 | `au_915` |
  | 1 | FCC915 | `fcc_915` |
  | 2 | EU868 | `eu_868` |
  | 3 | IN866 | `in_866` |
  | 4 | AU433 | `au_433` |
  | 5 | EU433 | `eu_433` |
  | 6 | US433 | `us_433` |
  | 7 | US433 wide | `us_433_wide` |

- **Antennas on every U.FL before power.** The PA keys telemetry within seconds of
  linking; an open U.FL is a bad VSWR into the PA.
- **TX and RX domains must match.** The GX12's sub-GHz domain is baked into its TX
  firmware the same way.

## 2. Build + bake

```sh
cd ~/Documents/GitHub/ExpressLRS/src
pio run -e <ENV>
cp .pio/build/<ENV>/firmware.bin /tmp/firmware.bin
python3 python/binary_configurator.py --target <TARGET> \
  --phrase <bind-phrase> --domain eu_868 --auto-wifi 60 \
  --ssid <ap-ssid> --password <ap-password> --rx-baud 420000 /tmp/firmware.bin
```

| board | ENV | TARGET | `--domain` |
|---|---|---|---|
| Mono | `Unified_ESP32C3_LR1121_RX_via_UART` | `opendrone.rx_dual.mono` | `eu_868` |
| Gemini | `Unified_ESP32C3_LR1121_RX_via_UART` | `opendrone.rx_dual.gemini` | `eu_868` |
| Lite | `Unified_ESP32C3_2400_RX_via_UART` | `opendrone.rx_2400.lite` | omit (2.4 only) |
| Lite-UFL | `Unified_ESP32C3_2400_RX_via_UART` | `opendrone.rx_2400.lite_ufl` | omit (2.4 only) |

`bootloader.bin`, `partitions.bin`, `boot_app0.bin` for a first flash come from the same
`.pio/build/<ENV>/` directory. Adding `--flash bf --port <port>` to the configurator
command bakes **and** flashes via Betaflight passthrough in one step (§4).

Do not set `--auto-wifi` lower than 60 on a dual-band single-radio board (Mono): the
900+2.4 rate scan needs tens of seconds and a short interval drops the RX into WiFi
before the scan reaches the second band.

## 3. First flash over UART (virgin board)

No USB and no reset line on any variant; flashing is via the ESP32-C3 ROM serial
bootloader on UART0. The C3's native USB pins (GPIO18/19) are consumed by board
functions, so there is never a USB console.

**Flash points**

| Signal | Lite / Lite-UFL / Mono | Gemini |
|---|---|---|
| ESP RX (U0RXD, GPIO20) | TP1 (RX pad) | TP1 |
| ESP TX (U0TXD, GPIO21) | TP2 (TX pad) | TP2 |
| 5 V | TP3 / `5V` pad | TP3 |
| GND | TP4 | TP4 |
| BOOT (GPIO9) | **TP5** (R4 pull-up) | **BOOT button U9** |

**Wiring rule**: 3.3 V USB-UART, adapter TX → TP1, adapter RX → TP2, GND → TP4. Power
the board from its **own clean 5 V**; leave the adapter's VBUS **disconnected**, keep its
USB plugged so it stays enumerated. Sharing the adapter's 5 V browns out the adapter on
every board power cycle and drops its USB enumeration (this caused repeated 0-byte serial
captures). 3.3 V logic only, never 5 V on RX/TX/BOOT.

**Procedure**

1. Full power-down, caps drained (~10 s). A marginal supply ramp gives a bad
   power-on-reset.
2. Pull BOOT (GPIO9) low: short TP5 → GND, or hold the Gemini's U9 button.
3. Apply 5 V with a clean edge, release BOOT.
4. Flash all four regions:

```sh
esptool --chip esp32c3 --port <PORT> --baud 460800 --before no_reset --after no_reset \
  write_flash -z --flash_mode dio --flash_freq 40m --flash_size detect \
  0x0 bootloader.bin  0x8000 partitions.bin  0xe000 boot_app0.bin  0x10000 firmware.bin
```

5. Full power-cycle with BOOT released to run.

**Strapping caveats (GPIO8 must not be low while GPIO9 is low)**

- Lite / Lite-UFL / Mono: GPIO8 = WS2812 DIN, floats. If download mode won't start, add
  ~10 kΩ pull-up to 3V3 on GPIO8.
- Gemini: GPIO8 = radio-2 BUSY (an output). If download mode refuses, hold radio-2 in
  reset during entry: RST_B (GPIO10) low.
- Gemini strapping pins in normal boot: GPIO9 = BOOT (R4 pull-up), GPIO8 = radio-2 BUSY
  (R3 pull-up, LR1121 drives BUSY high through POR), GPIO2 = radio-1 NRESET (R2 10 k
  pull-up). If boot ever misbehaves, scope GPIO2 at power-up.

## 4. Flash through the FC (Betaflight passthrough)

Verified working on OpenFC-Lite with a Betaflight build that includes the
`serialpassthrough` CLI command. One command bakes and flashes:

```sh
cd ~/Documents/GitHub/ExpressLRS/src
cp .pio/build/Unified_ESP32C3_LR1121_RX_via_UART/firmware.bin /tmp/firmware.bin
python3 python/binary_configurator.py --target opendrone.rx_dual.mono \
  --phrase <bind-phrase> --domain eu_868 --auto-wifi 60 --ssid <ap-ssid> --password <ap-password> \
  --rx-baud 420000 --flash bf --port /dev/cu.usbmodem<N> /tmp/firmware.bin
```

The script enters the FC CLI, validates the serial-RX config, starts
`serialpassthrough` on the CRSF UART, triggers the ELRS bootloader over CRSF, and runs
the fork's vendored esptool in `--passthrough` mode (flashes `0x10000 firmware.bin`
only; the other regions are already present).

**Preconditions and pitfalls, all hit in practice**

- FC must have `serialpassthrough` compiled in (`USE_SERIAL_PASSTHROUGH`).
- `set serialrx_halfduplex = OFF` + `save` (the script refuses otherwise; the OpenRX
  wiring is full duplex). The OpenFC target's default is ON.
- The FC must be in **normal run mode**, not sitting in CLI. A leftover CLI session
  makes the script print "No CLI available. Already in passthrough mode?" and the
  bootloader trigger lands in the CLI as garbage. Recover: send `exit` (reboots FC),
  retry.
- **The RX must be linked when the trigger is sent.** An unlinked RX (rate-scanning or
  in WiFi mode) ignores the CRSF bootloader trigger and esptool hangs at
  "Connecting...". Put the TX on any rate the RX's current firmware binds (2.4 GHz
  always works), confirm link (`status` shows RX rate > 0), then flash.
- After `set serialrx_halfduplex = OFF`, confirm `save` actually printed
  "Rebooting" / dropped the port. A save that did not reboot did not persist, and the
  next passthrough attempt fails the duplex check again.
- After a successful flash, esptool's "hard reset via RTS" only reaches the FC's USB,
  not the RX: **the RX stays in its bootloader until power-cycled**. The FC is also
  still in passthrough mode. Unplug and replug the FC USB; that power-cycles both.

The same passthrough is the wired debug channel to an installed RX: in the FC CLI,
`serialpassthrough UART0 420000` streams the RX UART out the FC USB port (CRSF binary
frames, plus `DBGLN` text on DEBUG_LOG builds). Exit requires an FC power cycle.

## 5. WiFi: options and OTA

WiFi serves **config and OTA only**. There is no log endpoint, no WebSocket, no telnet:
`DBGLN` output cannot be read over WiFi with stock ELRS.

**Entering WiFi mode**: power the RX (FC USB is enough), leave the TX off, wait
`wifi-on-interval` = 60 s. LED pulses yellow/green. The RX first tries to join network
the configured AP SSID and password: then it is at `http://elrs_rx.local`. If
that network does not exist it starts AP **`ExpressLRS RX`** (password `expresslrs`):
connect and it is at `http://10.0.0.1`.

**Endpoints**: `/` (web UI), `/options.json` (GET/POST), `/config`, `/hardware.json`,
`/update` (firmware POST), `/networks.json`, `/reboot`.

**OTA update** (board already runs ELRS, only `firmware.bin` needed):

```sh
curl -H "X-FileSize: $(stat -f%z /tmp/firmware.bin)" \
  -F "file=@/tmp/firmware.bin" http://<RX-IP>/update
```

**Change regulatory domain without reflashing** (runtime options saved on LittleFS
override baked defaults at every boot):

```sh
curl -s http://<RX-IP>/options.json -o /tmp/options.json
python3 -c "import json;o=json.load(open('/tmp/options.json'));o['domain']=2;json.dump(o,open('/tmp/options.json','w'))"
curl -H "Content-Type: application/json" -d @/tmp/options.json http://<RX-IP>/options.json
curl http://<RX-IP>/reboot
```

## 6. Debug channels

- **UART0 (RX/TX pads) is the only debug channel.** The C3's native USB Serial/JTAG
  (GPIO18/19) is consumed on every variant; the fork routes `DBGLN` to UART0 instead of
  the dead `USBSerial`.
- On the bench: external USB-UART per the §3 wiring rule, port at the CRSF baud
  (420000, or whatever `--rx-baud` was baked). UART0 also carries CRSF, so on a
  DEBUG_LOG build without an FC attached the log reads clean.
- Installed on a quad: FC CLI `serialpassthrough UART0 420000` (§4).
- Debug build flags: `-DDEBUG_LOG` (enables `DBGLN`), `-DDEBUG_CRSF_NO_OUTPUT`
  (silences CRSF frames so logs are clean), `-DDEBUG_LOG_VERBOSE`. Production builds
  carry none of these: debug shares UART0 with CRSF.

## 7. Bench verify

LED (WS2812) shows a rainbow at boot, then:

| LED | Meaning |
|---|---|
| Solid (color = rate) | Linked |
| Slow orange blink | Powered, no TX link: check TX on, same phrase, same domain |
| Rainbow / color cycle | Rate-scanning, not locked |
| Red fast blink | Radio not detected (layout/hardware) |
| Yellow↔green pulse | WiFi mode (after `auto-wifi` idle) |

EdgeTX → Telemetry → Discover Sensors → expect RQly, RSNR, RSSI/TRSS (2RSS/RSS on
Gemini), TPWR. Close range: RQly 100 %, RSSI ≈ −30…−50 dBm.

## 8. ELRS settings per variant (GX12 pairing)

- **Lite / Lite-UFL**: 2.4 GHz rates only.
- **Mono**: one LR1121, **one band at a time**. Any plain 2.4 GHz rate, or any plain
  sub-GHz rate in the RX's baked domain (EU868). **X (Xrossband) rates never bind on
  the Mono**: they transmit both bands simultaneously and need two receive chains.
- **Gemini**: everything, including X modes. In DualBand/X the firmware never swaps
  radios: **radio 1 (U3, chain A) is always sub-GHz, radio 2 (U6, chain B) is always
  2.4 GHz**. Antennas: **J1 = 900 MHz, J2 = 2.4 GHz**. (Both chains are electrically
  dual-band; the assignment comes from the firmware, `rx_main.cpp HandleFHSS()`.)
- TX settings change only from the handset (EdgeTX → SYS → Tools → ExpressLRS Lua) or
  the TX web UI. The RX cannot change TX settings; its parameter folder in Lua rides
  the RF link.

## 9. RF front-end and target-JSON reference

`radio_rfsw_ctrl = [enable, standby, RxCfg, TxCfg, TxHpCfg, TxHfCfg, RFU, WifiCfg]`.
Each byte is a bitmask of the LR1121's own RF-switch DIOs (bit0=DIO5, bit1=DIO6,
bit2=DIO7, bit3=DIO8) passed to `SetDioAsRfSwitch`; the chip drives them per state.
Per chain (Mono, and each Gemini chain): DIO5 = RFX2401C RXEN, DIO6 = RFX2401C TXEN,
DIO7 = SKY13373 V1, DIO8 = SKY13373 V2.

`[15,0,12,8,8,6,0,5]` decoded: standby = all off; rx(12) = sub-GHz RX; tx/txhp(8) =
sub-GHz TX_HP; txhf(6) = 2.4 GHz TX (PA); wifi(5) = 2.4 GHz RX (LNA). The LR1121
borrows the "wifi" slot as its 2.4 GHz receive state.

Paths: 2.4 GHz RFIO_HF → BPF → RFX2401C → SKY13373 → U.FL; sub-GHz TX RFO_HP_LF →
balun → SKY13373 → U.FL; sub-GHz RX U.FL → SKY13373 → balun → RFI_P/N_LF.
SKY13373 truth (V1,V2): `10` = 2.4 GHz, `01` = sub-GHz TX_HP, `11` = sub-GHz RX,
`00` = shutdown (antenna disconnected: a silent no-range failure if V1/V2 are never
driven). AE1 (2450AT18A100E) is the ESP32-C3 WiFi/OTA antenna on every variant, never
in the link RF path.

Canonical target JSONs live in `shared/elrs-targets/` and must stay identical to the
fork's `src/hardware/RX/` copies. Verified pin facts: Lite/Lite-UFL/Mono radio SPI is
`MISO 5, MOSI 4, SCK 6, NSS 7, DIO1 1, RST 2, BUSY 3`; Gemini radio 1 is
`NSS 0, RST 2, BUSY 3, DIO1 1` and radio 2 `NSS 7, RST 10, BUSY 8, DIO1 18`;
LED is `led_rgb` 8 (19 on Gemini) with `led_rgb_isgrb: true`; `radio_dcdc: true` on
both LR1121 boards.

## 10. Firmware fork: the fixes and the exit plan

Fixes carried by branch `opendrone-lr1121-dual-c3-fixes`, all confirmed necessary:

1. **Target-JSON pinouts** corrected for all four variants (now the §9 reference).
2. **`SetTcxoMode`** (Mono + Gemini): the OW7EL89 32 MHz TCXO is powered from the
   LR1121 `VTCXO` pin; stock ELRS assumes a crystal and never enables it, so the radio
   detects over SPI but does no RF. On the Gemini the TCXO is powered **only from U6's
   VTCXO**, so U6 must init first or neither radio has a clock.
3. **Gemini radio-1 second reset**: U3's NRESET is on strapping pin GPIO2; it needs a
   standalone second NRESET pulse after the supply settles.
4. **Software SPI chip-select** (opt-in): hardware CS does not reliably drive GPIO0
   (Gemini radio-1 NSS, an RTC pad); NSS is bit-banged.
5. **LR1121 upgrade-loop timeout**: the transceiver-FW update path had infinite
   `WaitOnBusy` loops that could brick boot; now times out to WiFi fallback.
6. **C3 debug → UART0** (board-specific, §6).

Hardware respin plan to return to stock ELRS: power the TCXO from +3V3 instead of
VTCXO (drops fix 2; on the Mono that alone suffices); Gemini: move radio-1 NSS off
GPIO0 and NRESET off GPIO2 (drops fixes 3 and 4), or respin on ESP32-S3 with clean
pins; larger or split +3V3 (the single 500 mA TLV75533 is marginal under dual-TX and a
dip below the LR1121 POR threshold can latch a radio off); add the 2.4 GHz matching
network on RFIO_HF.

## 11. Diagnosing a dead LR1121 (lessons already paid for)

- BUSY driven high = chip in reset. BUSY **floating** = digital core not executing.
  BUSY low = ready.
- `GetVersion` works without the TCXO (internal RC oscillator): SPI detection passing
  proves nothing about RF.
- VREG present ≠ core running: the analog LDO holds VREG up while the core is latched
  off after a brown-out. Recovery: full cold power-down, caps drained ~10 s.
- A radio answering with the bootloader version type has corrupt internal firmware:
  recover via forced bootloader, `EraseFlash`, re-upload transceiver FW (Semtech
  SWTL001 flow).
- False leads already ruled out on these boards, do not re-chase: open joints or dead
  chips when BUSY floats (it was reset ordering), 2.4 GHz `rfsw` band theories (it was
  TCXO + short auto-wifi), µs-delays in interrupt-path SPI (crashes the RX with an
  interrupt WDT right after connect), multi-probe diagnostics with confounded ordering
  (only isolated first-operation probes are trustworthy).

## 12. OpenFC-Lite (RP2354B, Betaflight)

Target `OPENFC_LITE_RP2350B`. RP2354B has the RP2350 USB UF2 bootloader in ROM.

1. Back up config: CLI `diff all`, save the output.
2. Enter BOOTSEL mode (hold BOOTSEL while plugging USB): UF2 drive mounts.
3. Copy the Betaflight `.uf2` onto the drive, or
   `picotool load -f <fw>.uf2 && picotool reboot`.
4. Restore config: paste the saved diff in the CLI, `save`.

Required serial settings for the OpenRX link: `serialrx_provider = CRSF`,
`serialrx_halfduplex = OFF` (target default is ON: fix after every config reset),
CRSF on UART0. Pads: SCL = GPIO17 = UART0 RX ← receiver TX; SDA = GPIO16 = UART0 TX →
receiver RX. Motors: RX1 = GPIO21 = M1, M2 = GPIO30, M3 = GPIO29, TX1 = GPIO22 = M4.
DisplayPort to O4 on PIOUART1: TP1 = GPIO26 TX, RP1 = GPIO27 RX. PINIO1 = GPIO11 =
10 V rail enable (USER1 box / AUX4).

## 13. GX12 transmitter

- Settings (rate, band, power): ExpressLRS Lua or TX web UI, never from the RX side.
- TX WiFi: Lua → WiFi Connectivity → Enable WiFi, then `http://elrs_tx.local` (or AP
  `ExpressLRS TX`, password `expresslrs`). Same OTA and options endpoints as §5.
- The TX's sub-GHz regulatory domain is baked into its firmware exactly like the RX
  (§1). TX and RX must both be EU868 for 868 rates to bind.
