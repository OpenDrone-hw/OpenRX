I have all the data I need in the prompt. Let me synthesize the final report directly.

# OpenRX-Lite (AntAIO) — Final Design Review

**Board:** OpenRX-Lite extended to integrated antweight/beetleweight combat-robot controller (ELRS 2.4 GHz RX + IMU + triple brushed-DC ESC).
**Verdict:** **NOT READY FOR FAB.** Multiple confirmed SEV1 issues, two of them safety-critical (uncommanded drive-motor motion at every boot/reset) and two that make the board effectively un-flashable. One real wiring defect (floating radio ground). A redesign of the ESP32-C3 pin-mux/motor-control scheme is required, not just resistor edits.

---

## 1. Executive Summary

The sensing, radio, IMU, power-conversion, clock, and RF-front-end subcircuits are largely correct and match reference designs. The fatal problems are concentrated in the **ESP32-C3 pin-mux for the three motor drivers** and the **programming/debug path**:

- The 16-GPIO-class ESP32-C3 (QFN-32, FH4) is **out of pins**. To wire 3×(PH+EN) motor control plus SPI(4) + radio(NSS/BUSY/DIO1) + IMU_CS + LED + BOOT + VSENSE, the design consumed **both** the native USB-Serial/JTAG pins (GPIO18/19) **and** UART0 (U0RXD/U0TXD). The result: no usable flashing/console interface, and two motor-enable lines that are forced HIGH at reset by the chip's own internal pull-ups.
- This produces **uncommanded LEFT and RIGHT drive-motor motion on every power-up, reset, and flash** — a safety-critical failure on a combat robot. The WEAPON channel, by contrast, defaults safe.
- One genuine wiring defect: **SX1281 pin 5 (GND) is floating.**
- A 5 V intermediate rail that exists only to feed a 3.3 V LDO (efficiency + thermal SEV2), a 2S-max battery ceiling set by the DRV8212 (SEV2), missing local VM decoupling on the drivers (SEV2), and assorted SEV3 robustness items round out the list.

**Readiness: Block fab. Resolve the SEV1 cluster (pin-mux/programming/boot-safety) with a control-scheme redesign or larger-pin/expander part, fix the floating radio ground, then re-review.**

---

## 2. SAFETY-CRITICAL — Uncommanded Motor/Weapon Motion

The dominant hazard. Confirmed by adversarial verification. PH/EN mode is selected on all three drivers (MODE = +3V3, DRV8212 Table 8-2). In PH/EN mode, **EN=1 drives the H-bridge; EN=0 = brake/Hi-Z** (Table 8-4). The DRV8212 inputs have only an internal ~100 kΩ pulldown (RPD). The ESP32-C3's internal pulls at reset overpower that 100 kΩ:

| Channel | EN net | ESP pin / reset state | EN node voltage at reset | Result |
|---|---|---|---|---|
| **LEFT drive (U6)** | LEFT_EN = U1.26 = **GPIO19 (USB D+)** | USB_PU (~1.5 kΩ FS pull-up) | 3.3·100k/101.5k ≈ **3.25 V** ≫ VIH 1.45 V | **ENABLED.** LEFT_PH=GPIO18 has a 50 µs HIGH power-up glitch (Table 2-2) → defined forward-then-reverse twitch |
| **RIGHT drive (U10)** | RIGHT_EN = U1.28 = **U0TXD** | WPU 45 kΩ + output-enabled (UART idles driven HIGH) | 3.3·100k/145k ≈ **2.28 V** > VIH 1.45 V; or driven push-pull HIGH | **ENABLED.** RIGHT_PH=U0RXD idles HIGH → sustained **Forward drive** |
| **WEAPON (U11)** | WEAPON_EN = U1.16 = **GPIO10** | IE only, no pull (Table 2-1); 5 ns LOW glitch | DRV 100 kΩ pulldown wins → **~0 V** | **DISABLED — SAFE** |

So **both drive motors are commanded ON, uncommanded, during the multi-ms window between reset release and firmware GPIO config**, on every power-up, brown-out recovery, watchdog reset, and flash cycle. RIGHT is the worst: U0TXD is actively driven high and the UART bitstream toggles the bridge (DRV tPD 135 ns ≪ one 115200-baud bit). The WEAPON channel is the only correctly-defaulting one (GPIO10 has no internal pull; PH hard-tied to GND).

**Note (correcting the original audit):** WEAPON_EN does **not** float and is **not** the dominant hazard — the original "floating weapon" framing was inverted. The weapon is safe by construction; the **drive** channels are the hazard.

**Why the simple fix is insufficient:** a 10 kΩ external pulldown on LEFT_EN cannot beat the ~1.5 kΩ USB_PU on GPIO19; on RIGHT_EN, 10 kΩ vs 45 kΩ WPU still leaves ~0.6 V (above VIL 0.4 V) and loses entirely when UART0 drives push-pull. **No pulldown wins against these pins.**

**Required fix (hardware):**
1. **Re-map LEFT_EN and RIGHT_EN (and their PH lines) off the USB D+/D- and UART0 console pins** onto plain GPIOs that have no internal reset pull-up (mirror the GPIO10/WEAPON pattern), with external pulldowns sized to dominate the DRV 100 kΩ.
2. **Hard-gate all three H-bridges' VM (or VCC) through a load switch that defaults OFF** until firmware asserts a separate "motors-armed" GPIO that itself powers up low. Treat "all bridges disabled at reset" as a hardware guarantee, not firmware-dependent. (DRV8212 VCC can be GPIO-gated as an arm path per datasheet 8.4.2.)
3. Add an explicit external pulldown on **WEAPON_EN** too (defense-in-depth on a weapon line), and ensure firmware brown-out + ELRS link-loss failsafe forces every EN low within one frame.

---

## 3. SEV1 — Showstoppers

### SEV1-A — No usable programming/debug interface (USB and UART0 both consumed by motors)
**Refs/nets:** LEFT_PH=GPIO18(U1.25), LEFT_EN=GPIO19(U1.26) → U6; RIGHT_PH=U0RXD(U1.27), RIGHT_EN=U0TXD(U1.28) → U10.
**What's wrong:** GPIO18/19 are the ESP32-C3's **only** native USB-Serial/JTAG D-/D+; U0RXD/U0TXD are the **only** UART0 console/download pins. Both pairs drive motor inputs. There is no USB connector and no clean UART header (only test points TP1/TP2 tapping the right-motor nets). Consequences: (1) USB programming and JTAG debug are permanently unavailable; (2) the only remaining flash path is BOOT-button + UART0 over TP1/TP2 — which is shared with the right-drive DRV8212, so flashing twitches the wheel; (3) Wi-Fi OTA cannot do the first flash or recover a bricked bootloader/partition.
**Evidence:** ESP32-C3 datasheet Table 2-4 (GPIO18/19 = USB_D-/D+), Section 2.3.x (default USB-Serial/JTAG); lite.json net members above; no USB/UART bridge in inventory.
**Fix:** Reserve a real programming interface — free GPIO18/19 for USB-Serial/JTAG and add a USB-C connector or D+/D- pads; move LEFT motor elsewhere. Realistically the part is out of GPIO, so the genuine fix is **collapse the motor-control scheme** (single-pin sign-magnitude PWM per driver, or an SPI/I²C motor-driver expander) to free UART0 and/or USB, or move to a higher-pin-count MCU.

### SEV1-B — Boot-time UART0 traffic drives the RIGHT motor
**Refs/nets:** RIGHT_EN=U0TXD(U1.28)→U10.5, RIGHT_PH=U0RXD(U1.27)→U10.6.
**What's wrong:** U0TXD is in the After-Reset state WPU + output-enabled (Table 2-1, fn 8): UART0 TX idles actively-driven HIGH and follows any UART init, holding RIGHT_EN high (and U0RXD/PH high → Forward). Covered quantitatively in §2; called out separately because it removes the serial console as well.
**Correction to original audit:** the default ROM-message channel on ESP32-C3 is the **USB-Serial/JTAG controller, not UART0** (Section 2.6.2 / Table 2-12; UART printing requires EFUSE_DIS_USB_SERIAL_JTAG=1). So "the ROM prints to U0TXD and toggles RIGHT" is the wrong mechanism — but the enable-forced-high conclusion holds via the After-Reset output-enabled-high state and any firmware UART0 init.
**Fix:** Never place a motor enable on the UART0 console TX. Re-map RIGHT off U0RXD/U0TXD; keep UART0 free for CRSF/console.

### SEV1-C — SX1281 pin 5 (GND) floating
**Refs/nets:** U3.5 → single-node net `unconnected-(U3-GND-Pad5)`.
**What's wrong:** Pin 5 is a **dedicated die ground** (SX1281 datasheet Table 2-1, p.17), distinct from the exposed pad (pin 25). It sits between XTA(4) and XTB(6) — the reference-oscillator/PA guard ground. It is the only U3 ground pin left open (13/20/21/23/24 and EP are on GND). Floating it removes the local oscillator/PA ground return and shield, degrading phase noise, spurs, and EMC. ERC missed it because the custom symbol's pin 5 electrical type is `unspecified`.
**Fix:** Tie U3.5 to the GND net/plane and re-pour. Scope the fix to **pin 5 only** — XTB(6) NC is correct TCXO behavior, do not "fix" it. Set the symbol pin to power_in/passive so ERC catches this class of omission in future.

---

## 4. SEV2 — Major

### SEV2-A — 5 V intermediate rail serves nothing but the 3.3 V LDO
**Refs/nets:** `+5V` = [C37.1, C38.1, C39.1, R16.2, TP15.1, U4.3, U4.4, U8.7]. Only U4 (TLV75533 LDO) is a real load. Motors run from +BATT; all logic from +3V3.
**What's wrong:** Chain is +BATT → TPS63070 buck-boost → 5.0 V → linear LDO → 3.3 V. The LDO burns (5.0−3.3)/5.0 = **34% of all logic-rail energy** for no functional reason. TPS63070 output range is 2.5–9 V (it can regulate 3.3–4.0 V directly). Setpoint confirmed 4.98 V via R16=52.3k/R17=10k, VFB=0.8 V.
**Fix:** Retarget the buck-boost to **~3.8–4.0 V** (R16 ≈ 40k with R17=10k) to keep the LDO in regulation while cutting its loss ~4×, or set 3.3 V and delete the LDO (but then the SX1281 loses the LDO's RF supply filtering). **Note:** the original "~3.5 V" suggestion is too low — TLV75533 dropout is 238 mV max @500 mA and it is spec'd at VIN ≥ VOUT+0.5 V, so 3.5 V starves regulation/PSRR.

### SEV2-B — TLV75533 (1×1 mm X2SON-4) thermal margin on the shared 3V3 rail
**Refs:** U4 OUT=+3V3 feeds U1, U3, D1, U2, U6/U10/U11 VCC. RθJA = 168.4 °C/W (DQN).
**What's wrong:** Dropping 1.7 V at sustained current pushes Tj past the 125 °C recommended max in an enclosed combat ambient: ~290 mA @25 °C or ~250 mA @45 °C breaches 125 °C; 335 mA (ESP Wi-Fi TX @100% duty, Table 4-7) → Tj ~121 °C @25 °C / ~141 °C @45 °C. Risk of thermal foldback → MCU/radio brownout.
**Corrections to original audit:** thermal shutdown is ~165 °C (not 150 °C); RθJA is steady-state, so the real driver is **sustained** Wi-Fi TX during OTA/SoftAP config, not motor PWM transients — during live combat the ELRS link is RX-dominated (~84 mA), so the direct in-match link-loss coupling is weaker than originally stated. Still a real margin issue.
**Fix:** Lower LDO Vin to ~3.8–4.0 V (Pd drops ~4–8×) per SEV2-A, or move to the 2×2 mm DRV SON-6 package (RθJA 100.2 °C/W) with a generous thermal-pad pour; cap WS2812 brightness/duty. Re-verify Tj < 125 °C at real ambient.

### SEV2-C — No local 0.1 µF VM ceramic on any of the three drivers
**Refs/nets:** +BATT (=VM) carries only bulk: C31–C34/C42 = 22 µF, C35 = 10 µF. No 100 nF anywhere on +BATT; no per-VM local HF cap.
**What's wrong:** DRV8212 Table 8-1 / VM pin description mandates a 0.1 µF low-ESR ceramic VM→GND **at each device** plus bulk. Bulk MLCC at distance cannot damp VM ringing from the H-bridge edges; risk of VM overshoot toward the 12 V abs-max and false OCP/UVLO trips.
**Fix:** Add one 0.1 µF (X7R, ≥16 V) VM→GND at pin 1/pin 4 of **each** of U6/U10/U11, placed immediately at the pins. Also ensure a dedicated 0.1 µF CVCC at each driver's pin 8 (required for the DSG package) is placed locally. (VM abs-max is exactly 12 V, not 11.)

### SEV2-D — DRV8212 VM caps the pack at 2S; 3S overvolts all three drivers
**Refs:** U6.1/U10.1/U11.1 = +BATT = VM (direct, only Q2 in series).
**What's wrong:** DRV8212 VM = 11 V recommended / **12 V abs-max**. 3S LiPo at 12.6 V exceeds abs-max → driver damage. 2S (8.4 V) is within rating. The DRV8212 is the single binding limit — TPS63070 (16/20 V) and AON7407 (−20 V VDS) are not. **Correction:** an earlier note citing "AON7407 30 V" is wrong; it is a 20 V part. Brushed-motor stall/regen flyback and hot-plug inrush (VM transient ramp abs-max 2 V/µs) can push VM above 12 V even on a compliant 2S pack — the +BATT rail currently has no TVS.
**Fix:** Hard-label/spec **2S max (8.4 V)** in README/DESIGN and silkscreen. Add a +BATT TVS with standoff < 12 V (e.g. ~8.5–9 V working, SMBJ-class) and confirm bulk handles inductive flyback. For 3S support the DRV8212 must be replaced (e.g. DRV8220, 18 V — but 1 Ω RDS(on)).

### SEV2-E — ESP32-C3 symbol mislabels pins 4/5 as XTAL_32K_P/N instead of GPIO0/GPIO1
**Refs:** U1 pin4 (symbol name "XTAL_32K_P") = VSENSE; pin5 ("XTAL_32K_N") = DIO1.
**What's wrong:** Netlist binds by pin number so the connection is electrically correct (pin4 = GPIO0/ADC1_CH0, pin5 = GPIO1/ADC1_CH1; the 32 kHz names are alternate functions, no 32 kHz crystal is used). But the symbol pin names are misleading and a real review/maintenance hazard — a future spin could "fix" a non-bug or wire a real 32 kHz crystal there.
**Fix:** Rename the symbol pins to GPIO0/GPIO1 (or "GPIO0/XTAL_32K_P") in OpenRX-Shared.kicad_sym. Metadata-only; no netlist change.

---

## 5. SEV3 — Minor / Robustness / EMC

- **L1 24 nH series inductor in the XTAL_P leg (X1 40 MHz):** non-standard and asymmetric vs the reference two-cap oscillator. XTAL_P=[L1.2,U1.30] → C1(15 pF)+X1.1; XTAL_N=[C3.1,U1.29,X1.3]. The series L (and placing C1 on the crystal side, not the pin) shifts effective load capacitance and can reduce startup/negative-resistance margin on a safety-critical board. **Fix:** remove L1 (0 Ω), put 15 pF symmetric load caps directly at both XTAL_P/XTAL_N per Espressif reference; if L1 is deliberate (EMC), make it symmetric and verify startup margin. Load caps C1=C3=15 pF are otherwise correct for the CL=10 pF crystal.

- **SX1281 NRESET shares the ESP32-C3 EN reset RC** (CHIP_EN = [C28.1, R5.1, U1.7, U3.3]; R5=10k, C28=1µF, ~10 ms POR). Radio cannot be reset independently of the MCU; a wedged radio needs a full ESP power-cycle. NRESET has an internal 50 kΩ pull-up so the wired POR is valid. **Downgraded from SEV2 → SEV3:** functionally fine for ELRS, watchdog-reset recovers it, no motor path. **Fix:** set SX1281 reset=−1 in the ELRS target JSON and document the shared-reset choice; optionally route NRESET to a spare GPIO.

- **WEAPON_EN relies only on the DRV 100 kΩ internal pulldown** — no external pulldown on a weapon-arming line. **Downgraded from SEV2 → SEV3:** the default state is robustly off (DRV VIH=1.45 V needs ~14.5 µA to false-trip; not credible from ESP Hi-Z leakage; GPIO10 power-up glitch is LOW). Still worth a dedicated 4.7k–10k external pulldown at U11.5 as combat-robot best practice.

- **VSENSE node has no anti-alias/decoupling cap and ~8.4 kΩ source impedance.** VSENSE=[R11.1(52.3k),R12.2(10k),U1.4]; taps raw +BATT (shared with motor VM). High-Z + no filter → jittery ADC, possible nuisance LVC/failsafe trips. **Fix:** add 10–100 nF VSENSE→GND at the ADC pin (forms RC with the divider). Divider ratio 0.1605 is correct (2S→1.35 V, in ADC range).

- **TPS63070 COUT may dip below the 40 µF-effective floor after DC-bias derating.** C37(10µF/0603)+C38(22µF/0805)+C39(22µF/0805) = 54 µF nominal → ~30–40 µF effective at 5 V bias. **Fix:** match the datasheet 3×22 µF reference (swap C37 10µF→22µF) or use better DC-bias-retaining parts. Milder if rail is retargeted to ~3.8 V.

- **+BATT bulk thin / verify cap voltage rating.** ~120 µF nominal (heavily derated to ~40–80 µF effective at 8.4 V) shared by three H-bridges. Confirm C45783/C92487 are ≥16 V (preferably 25 V) rated for inductive transients; consider adding a low-ESR polymer/electrolytic at the +BATT entry for the weapon transient.

- **WS2812B (D1) VDD on +3V3 is below the 3.5 V datasheet minimum** (spec 3.5–5.5 V). 3.3 V is out-of-spec; clones usually light but color accuracy/internal-regulator behavior is unguaranteed. **Fix:** move D1 VDD to +5V — DIN direct from 3.3 V GPIO8 still meets VIH=2.8 V, no level shifter needed. *(Note: this is listed SEV2 in the source audit; rated SEV3 here as it is a status LED, not function-critical — but it is genuine out-of-spec operation, fix it.)*

- **WS2812B DIN has no series damping resistor.** R3 (10k to +3V3) is the GPIO8 boot strap, not a series term. Fine for a single short-stub LED; add ~47–100 Ω series in the DIN trace if EMI/edge integrity matters later (keep the 10k strap on the GPIO8 side).

- **VR_PA decoupled by single 10 nF (C27) vs reference 2×10 nF; DCC_FB second cap 10 nF (C26) vs reference 100 nF.** Add a parallel cap on VR_PA and change C26 → 100 nF to match the Semtech reference. Low priority.

- **TCXO (OSC1) VCC has no ferrite/dedicated local bypass; powered straight off the noisy shared +3V3.** SX1281 TCXO app note (Fig 15-3) shows a ferrite bead + 100 nF to keep digital/switcher noise off the 52 MHz reference (phase noise directly degrades RX). **Fix:** add a 600 Ω@100 MHz bead + local 100 nF at OSC1 VCC.

- **WSON-8 DRV8212 thermal ceiling ~2 A/channel continuous.** I²R = I²·0.28 Ω; at 3 A → ~2.5 W → TSD. OCP trips at 4 A (1.7 ms retry). Weapon/drive stall will OCP-cycle/TSD. **Fix:** dense thermal-via field under each EP, firmware current/PWM limits ~2 A/channel; for the weapon consider paralleling half-bridges (8 A) — requires Hi-Z MODE redesign.

- **L6 (1.5 µH 2016) Isat headroom thin vs 3.6 A switch limit** — acceptable only because the real 5 V load is light. Confirm Isat ≥ ~2.5 A when sourcing.

---

## 6. ELRS RF Front-End — Open Items (review-flagged, schematic-scope)

- **AE2 antenna BOM/doc mismatch + no matching network.** Netlist value for AE2 is `2450AT18A100E` (same as the Wi-Fi AE1), but README/CLAUDE.md state the Lite link antenna is the Molex **47948-0001**. Reconcile BOM vs docs. Net-(FL1-OUT)=[AE2.1, FL1.3] has **no matching components and no retune pads**, while the 2450AT18A100E datasheet explicitly requires a match (RL only 9.5 dB min). **Fix:** decide the real part and make the value field match; add a pi/L match footprint (0201 placeholders) on the FL1-OUT→antenna node; budget the loss into the link budget. *(Worth resolving but not a SEV1 — link still functions.)*

- **RFIO → FL1 direct (no series match/DC-block).** Acceptable: FL1's IN port is a 40 Ω chipset-matched port intended to mate with SX1281 RFIO directly, and the 2450FM07 is internally DC-blocked. Confirm against the Johanson app note / ELRS target, keep RFIO→FL1 trace short and impedance-controlled, and add FL1 IL (0.75–1.0 dB) to the link budget. If FL1 DC-pass is ever unconfirmed for a future grounded-feed antenna, add a series 100 pF C0G.

- **Two 2.4 GHz radios (ELRS link + ESP Wi-Fi), separate antennas, no RF switch.** Standard ELRS-RX approach; coexistence is firmware time-shared (Wi-Fi OTA-only). Maximize AE1/AE2 isolation in layout; document Wi-Fi as OTA-only in DESIGN.md.

---

## 7. Unconnected-Pin Disposition

| Pin(s) | Disposition |
|---|---|
| U1.19–24 (SPIHD/WP/CS0/CLK/D/Q) | **NC — correct.** In-package flash on ESP32-C3FH4; must not route externally |
| U2.4 INT1, U2.9 INT2 (IMU) | **NC — acceptable.** Polling-only; no spare GPIO for DRDY. For melty/heading-hold a hardware DRDY would give deterministic sampling — note as limitation |
| U2.10, U2.11 (IMU NC) | **NC — correct.** Datasheet: leave electrically unconnected, keep pads soldered |
| U3.5 (GND) | **DEFECT — must tie to GND.** See SEV1-C |
| U3.6 (XTB) | **NC — correct.** TCXO drives XTA via C19; second crystal terminal unused |
| U3.9 DIO2, U3.10 DIO3 | **NC — correct.** Optional radio DIO, unused by ELRS |
| U8.2 (PG) | **NC — acceptable.** Open-drain power-good; could optionally gate driver arm |
| D1.1 (DOUT) | **NC — correct.** Single WS2812, end of chain |
| AE1.2, AE2.2 | **NC — correct.** Single-feed chip-antenna ground/keep-out pads |
| GPIO20/GPIO21 (U0RTS/U0CTS) | **Unused — the only free pins.** Repurposing these is the lever to fix the pin-budget/programming-path problem |

---

## 8. Confirmed-Correct / INFO

- **Strapping pins GPIO2/GPIO8/GPIO9** all correctly biased (10k pull-ups to +3V3; BOOT button to GND): GPIO2(IMU_CS)=1, GPIO8(LED)=1, GPIO9(BOOT)=1 idle / 0 pressed → clean SPI boot, download only on button. R2 doubles as IMU CS idle-high and the GPIO2 strap; R3 doubles as the GPIO8 strap. **Ensure R2/R3/R4 are populated.**
- **CHIP_EN RC** = 10k(R5) × 1µF(C28) ≈ 10 ms POR, satisfies tSTBL/tRST(50 µs), not floating (datasheet requirement met).
- **VSENSE on GPIO0/ADC1_CH0** — valid ADC pin (ADC1 works with Wi-Fi on); divider 0.1605 keeps 1S–3S in the ATTEN3 0–2500 mV window; cell count bounded by DRV8212 VM, not the ADC.
- **LNA_IN Wi-Fi pi-match** (C22 1.2pF shunt / L2 2.5nH series / C21 1.2pF shunt) → AE1: canonical ESP32-C3 single-ended topology; tune on VNA.
- **L3 = Murata LQP03TN2N0B02D (2.0 nH RF inductor), correct** — the +3V3→L3→C25(1µF)→VDD3P3(pins 2,3) topology matches the Espressif analog/RF (Wi-Fi PA) reference. The "2N0" MPN decodes to 2.0 nH; **not** a mislabeled ferrite bead.
- **SX1281 SMPS mode** correct (VDD_IN tied to DCC_FB, L4 15µH on DCC_SW, 470nF on DCC_FB). **TCXO front-end** correct (52 MHz clipped-sine → C19 1nF AC-couple → XTA; XTB NC). **SPI shared bus** correct: MISO/MOSI/SCK shared SX1281+IMU+ESP, dedicated CS each, NSS 10k pull-up, IMU CS 10k pull-up — no contention, both slaves tri-state MISO when deselected (Semtech Fig 9-5 endorses this topology).
- **LSM6DS3TR-C** fully correct: 4-wire SPI not swapped (SDI←MOSI, SDO→MISO, SPC←SCK, CS←IMU_CS); aux-I²C SDx/SCx tied to GND (datasheet Mode-1); VDD/VDDIO on +3V3 with 100nF each (C15/C18); all node voltages within 4.8 V abs-max.
- **FL1 2450FM07D0034T** orientation correct (IN 40Ω→RFIO, OUT 50Ω→antenna, both GND pads grounded, internally DC-blocked).
- **Q2 AON7407 reverse-polarity P-FET** topology and gate network correct: source on battery B+, drain on +BATT load; R13 10k gate pulldown turns it on in normal use; **U7 6.2 V Zener correctly clamps VGS within the ±8 V abs-max** (required at 2S where unclamped VGS would be 8.4 V). *(See §9 — the "FET is backwards" claim is one investigator's contested finding.)*
- **5V→3V3 sequencing** correct (LDO EN tied to +5V, so 3V3 cannot precede 5V). TPS63070 EN via R15 100k to +BATT (always-on, within 20 V abs-max), PS/SYNC to GND via R14 (forced PWM, low ripple — good for RF), FB R16/R17 = 4.98 V, Vaux 100 nF.
- **DRV8212 MODE=+3V3 (PH/EN)** consistent on all three; **WEAPON PH=GND** is a sensible fixed-direction single-quadrant weapon drive; **EP→GND** on all three; OUT routing to motor pads correct.

---

## 9. Investigated — Not an Issue (dropped per verification)

- **"LEFT_PH GPIO18 50 µs glitch is a standalone SEV1 uncommanded-motion path."** **INVALID / demoted to INFO.** Per DRV8212 Table 8-4, EN=0 forces brake→Hi-Z with PH as don't-care, and PH has its own internal pulldown — a PH glitch alone causes no motion. The real hazard is the EN line (covered in §2). The 50 µs glitch only sets direction once EN is (separately) asserted.
- **"VDD_SPI (pin18) tied to +3V3 is a SEV2 topology/labeling risk."** **INVALID / demoted to INFO.** For the embedded-flash FH4, VDD_SPI is a backup-power **input** whose recommended input range (3.0/3.3/3.6 V) equals the rail; tying to +3V3 is the standard Espressif approach used in every C3 module. The "internal-LDO-shorted" scenario requires an off-package flash that does not exist (pins 19–24 unbonded). Residual: the symbol's `power_out` type for pin 18 is cosmetically imprecise (may trip a benign ERC warning) — optionally set to power_input/passive.
- **"Analog VDD3P3/VDDA decoupling SEV2 — L3 is a mislabeled ferrite bead."** **INVALID.** L3 is a genuine 2.0 nH RF inductor (Murata LQP03TN2N0B02D, see §8). The topology is the Espressif analog/RF reference. The only residue is a routine layout reminder (place a 100 nF adjacent to pins 2/3, 11, 17, 31/32) — INFO, not SEV2.
- **Q2 "FET is wired backwards / body diode defeats reverse protection" (one subcircuit investigator).** **Contested / not carried as a finding.** The dedicated Q2 pin-level audit and the power-tree audit both conclude the AON7407 orientation (source→battery B+, drain→+BATT load) with the R13 pulldown and 6.2 V Zener clamp is the correct high-side P-FET reverse-polarity/load-switch topology that blocks on reverse-connect. Given the majority pin-level verdict, this is not flagged as a defect, but it is the **one disagreement among reviewers**: confirm Q2 source/drain orientation against the AON7407 body-diode direction on the actual symbol/footprint before the next rev, since a genuinely reversed P-FET would pass reverse current through the body diode. Low effort to verify, high consequence if wrong.

---

### Priority fix order
1. **Re-architect ESP32-C3 motor control** to free USB (GPIO18/19) and UART0 — single-pin sign-magnitude PWM or SPI/I²C motor expander, or larger MCU. This resolves SEV1-A/B and the §2 boot-drive hazard simultaneously.
2. **Hardware motor-disable / arm load switch** defaulting OFF (VM or VCC gated), plus external pulldowns on all three EN lines.
3. **Tie SX1281 pin 5 to GND** (SEV1-C).
4. Retarget buck-boost to ~3.8–4.0 V (SEV2-A/B); add per-driver 0.1 µF VM + VCC caps (SEV2-C); 2S-max label + +BATT TVS (SEV2-D); fix the symbol pin names (SEV2-E).
5. SEV3 cleanup: remove/symmetrize L1, VSENSE filter cap, WS2812 on +5V, TCXO bead, COUT match, reconcile AE2 BOM/docs + add match pads.

**Relevant files:** `hardware/AntAIO.kicad_sch`, `hardware/esp32c3_sx1281_lite.kicad_sch`, symbol library `shared/libs/OpenRX-Shared.kicad_sym` (SX1281 pin-5 type, ESP32-C3 pin4/5 names).