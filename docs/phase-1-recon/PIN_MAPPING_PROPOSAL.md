# Phase 1 / Deliverable #4 — PIN_MAPPING_PROPOSAL

> Lock specific pin assignments untuk Waveshare RP2040-Zero target. Cross-validated dari `keyboard.json`, `config.h`, [KLORpinout.png](../../references/KLOR-upstream/docs/images/KLORpinout.png) visual reference, dan zerosprey42 (42-key RP2040-Zero) sebagai exemplar.

---

## A. Reference baseline — fork's pin mapping (Pro Micro physical layout)

[KLORpinout.png](../../references/KLOR-upstream/docs/images/KLORpinout.png) menunjukkan **fork PCB uses Pro Micro physical pinout** (24-pin dual-row format). Pin label di PNG = AVR convention (D0-D7, B1-B7, C6, E6, F4-F7). Fork's elite_pi config (`development_board: "elite_pi"`) maps these physical positions ke **RP2040 GPxx labels** via standard elite_pi-to-Pro-Micro pin mapping.

Cross-reference table (Pro Micro pin → AVR label → RP2040 GPxx → function):

| PM pin # | Side | AVR label | RP2040 GPxx | Fork function | Source |
|---|---|---|---|---|---|
| 1 | TL | TX0/D3 | **GP0** | WS2812 RGB DI | KLORpinout, keyboard.json `ws2812.pin` |
| 2 | TL | RX1/D2 | **GP1** | Serial TX (half-duplex) | KLORpinout, [config.h:49](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) |
| 3 | TL | GND | GND | Ground | — |
| 4 | TL | GND | GND | Ground | — |
| 5 | TL | 2/D1/SDA | **GP2** | I²C1 SDA (OLED) | KLORpinout, [config.h:26](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) |
| 6 | TL | 3/D0/SCL | **GP3** | I²C1 SCL (OLED) | KLORpinout, config.h:25 |
| 7 | TL | 4/D4 | **GP4** | (Reserved: serial RX full-duplex) | config.h:52 commented |
| 8 | TL | 5/C6 | **GP5** | Matrix row 0 | KLORpinout, keyboard.json rows[0] |
| 9 | TL | 6/D7 | **GP6** | Matrix row 1 | keyboard.json rows[1] |
| 10 | TL | 7/E6 | **GP7** | Matrix row 2 | keyboard.json rows[2] |
| 11 | TL | 8/B4 | **GP8** | Matrix row 3 | keyboard.json rows[3] |
| 12 | TL | 9/B5 | **GP9** | Audio PWM (DROPPED) | KLORpinout, config.h:115 |
| 13 | TR | B6/10 | **GP21** | Matrix col 5 | keyboard.json cols[5] |
| 14 | TR | B2/16 | **GP23** | Matrix col 4 | keyboard.json cols[4] |
| 15 | TR | B3/14 | **GP20** | Matrix col 3 | keyboard.json cols[3] |
| 16 | TR | B1/15 | **GP22** | Matrix col 2 | keyboard.json cols[2] |
| 17 | TR | F7/A0 | **GP26** | Matrix col 1 | keyboard.json cols[1] |
| 18 | TR | F6/A1 | **GP27** | Matrix col 0 | keyboard.json cols[0] |
| 19 | TR | F5/A2 | **GP28** | Encoder A (left), Encoder B (right per split.encoder) | keyboard.json encoder |
| 20 | TR | F4/A3 | **GP29** | Encoder B (left), Encoder A (right) | keyboard.json encoder |
| 21 | TR | VCC | 3.3V | Logic power | — |
| 22 | TR | RST | RST | Reset (to button) | — |
| 23 | TR | GND | GND | Ground | — |
| 24 | TR | RAW | VBUS | 5V from USB | — |

✅ **Verified cross-consistency:** KLORpinout PNG, `keyboard.json`, `config.h` semua agree — pin assignment fork tersinkron.

---

## B. RP2040-Zero pin availability audit

Waveshare RP2040-Zero exposes **20 GPIO** (per [Waveshare wiki](https://www.waveshare.com/wiki/RP2040-Zero)):

| GPxx | Edge pad? | Notes |
|---|---|---|
| GP0–GP15 | ✅ Castellated edge | 16 pins, 1.27mm pitch |
| GP16 | ❌ Internal | Onboard WS2812 status LED (Waveshare boot indicator) |
| GP17–GP19 | ❌ NOT accessible | Not broken out |
| GP20–GP25 | ❌ NOT accessible | Not broken out (USB pins GP24/25) |
| GP26–GP29 | ✅ Bottom ADC pads | 4 pins (separate from castellated edge) |

🚨 **Critical mismatch identified:** Fork's cols GP20, GP21, GP22, GP23 are **NOT exposed** di RP2040-Zero. Need remap.

---

## C. Final pin allocation untuk Waveshare RP2040-Zero

| GPxx | Function | Type | Rationale | Status |
|---|---|---|---|---|
| **GP0** | WS2812 RGB DI | PIO | Native RGB data pin, PIO-driven (any GPIO works, GP0 is convention) | ✅ Keep from fork |
| **GP1** | Serial TX (half-duplex) | UART/PIO | Per fork config.h:49 active config | ✅ Keep from fork |
| **GP2** | I²C1 SDA | I²C alt-func | I²C1 hardware peripheral pair (GP2/GP3 native) | ✅ Keep from fork |
| **GP3** | I²C1 SCL | I²C alt-func | Pair with GP2 | ✅ Keep from fork |
| **GP4** | Serial RX (full-duplex upgrade path) | UART/PIO | Per fork CHANGELOG.md:69-73 documented spec | ⚠️ Route PCB pad even if firmware tidak enable initially |
| **GP5** | Matrix row 0 | GPIO | Keep from fork | ✅ |
| **GP6** | Matrix row 1 | GPIO | Keep from fork | ✅ |
| **GP7** | Matrix row 2 | GPIO | Keep from fork | ✅ |
| **GP8** | Matrix row 3 | GPIO | Keep from fork | ✅ |
| **GP9** | **Matrix col 2** | GPIO | **REMAP** dari GP22 (freed from audio drop). Ascending order chosen untuk natural label sequence | 🆕 New |
| **GP10** | **Matrix col 3** | GPIO | **REMAP** dari GP20 | 🆕 New |
| **GP11** | **Matrix col 4** | GPIO | **REMAP** dari GP23 | 🆕 New |
| **GP12** | **Matrix col 5** | GPIO | **REMAP** dari GP21 | 🆕 New |
| **GP13** | (Headroom / spare 1) | GPIO | Available for future feature | ⚪ Free |
| **GP14** | (Headroom / spare 2) | GPIO | Available for future feature | ⚪ Free |
| **GP15** | **RESET button** | GPIO | Soft reset via firmware (Vial reset keycode). User-accessible reset button on PCB | 🆕 New |
| GP16-GP25 | (Not accessible on RP2040-Zero) | — | Not broken out (board design limit) | ❌ N/A |
| **GP26** | Matrix col 1 | GPIO | Keep from fork (ADC-capable, but used as GPIO here) | ✅ |
| **GP27** | Matrix col 0 | GPIO | Keep from fork | ✅ |
| **GP28** | Encoder A (left) / B (right) | GPIO | Keep from fork. ADC-capable but used as GPIO here. | ✅ |
| **GP29** | Encoder B (left) / A (right) | GPIO | Keep from fork | ✅ |

**Total used: 18 GPIO of 20 available. Headroom: 2 GPIO** (GP13, GP14).

Used breakdown: GP0 (RGB DI) + GP1 (serial TX) + GP2-3 (I²C OLED) + GP4 (serial RX reserve, PCB-routed) + GP5-8 (rows) + GP9-12 (cols 2-5) + GP15 (RESET) + GP26-27 (cols 0-1) + GP28-29 (encoder).

---

## D. Updated `keyboard.json` matrix_pins (Phase 6 firmware update preview)

**Current** (fork's klor1_4):
```json
"matrix_pins": {
    "cols": ["GP27", "GP26", "GP22", "GP20", "GP23", "GP21"],
    "rows": ["GP5", "GP6", "GP7", "GP8"]
}
```

**Proposed** (post-remap untuk RP2040-Zero, **LOCKED** per schematic MOD #1 commit `ee98b40`):
```json
"matrix_pins": {
    "cols": ["GP27", "GP26", "GP9", "GP10", "GP11", "GP12"],
    "rows": ["GP5", "GP6", "GP7", "GP8"]
}
```

**Mapping logic untuk new col pins (ascending):**
- Maintain GP27/GP26 di cols 0-1 (fork-preserved, ADC pad)
- Cols 2-5 remap ascending to GP9, GP10, GP11, GP12 — user-accepted untuk natural label sequence (`col 2 < col 3 < col 4 < col 5` matches `GP9 < GP10 < GP11 < GP12`)
- GP9 freed by audio feature drop (was PWM speaker); now repurposed sebagai col 2

**Status:** locked in schematic. Update `keyboard.json` at Phase 6 firmware step (along with `processor`/`bootloader` macros + Justin Lam config — see PROJECT_SPEC.md firmware section).

---

## E. Pin function compatibility check (RP2040 alt-function table)

Verify none of our assignments conflict dengan RP2040's hardware peripheral routing:

| GPxx | Our use | RP2040 alt-functions yang relevant | Conflict? |
|---|---|---|---|
| GP0 | PIO (WS2812) | UART0 TX, I²C0 SDA, SPI0 RX, PIO | None — PIO is software-defined, no conflict dengan unused alts |
| GP1 | Serial (PIO or UART0 RX) | UART0 RX, I²C0 SCL, SPI0 CS, PIO | None |
| GP2 | I²C1 SDA | I²C1 SDA (native), SPI0 SCK, UART0 CTS | ✅ Using native peripheral |
| GP3 | I²C1 SCL | I²C1 SCL (native), SPI0 TX, UART0 RTS | ✅ Native |
| GP4 | Serial (PIO or UART1 TX) | UART1 TX, I²C0 SDA, SPI0 RX, PIO | None — flexible |
| GP5-GP8 | GPIO (matrix rows) | Various; not used | None |
| GP9-12 | GPIO (matrix cols 2-5) | Various; not used | None |
| GP15 | GPIO (RESET button) | Various; not used | None — used as standard digital input |
| GP26-29 | GPIO (cols + encoder) | ADC inputs; not used as ADC | None (configured as digital GPIO via QMK) |

✅ **No conflicts.** All special peripherals (I²C1, PIO untuk RGB, UART untuk serial) sudah pakai native pins atau PIO-flexible.

---

## F. Cross-validation dengan zerosprey42 (42-key RP2040-Zero exemplar)

[zerosprey42 README](../../references/zerosprey42/README.md) confirms:
- Same MCU: WaveShare RP2040-Zero
- Same firmware: QMK + Vial
- Same key count target: 42 keys
- **Different topology:** monoblock unibody (single PCB, single MCU) — bukan split

Zerosprey42 firmware ada di repo terpisah ([vial-qmk-zerosprey42](https://github.com/beekeeb/vial-qmk-zerosprey42)) — tidak cloned dalam workspace kita. Untuk pin assignment cross-check, akan butuh either:
- Clone vial-qmk-zerosprey42 (defer kalau tidak critical)
- Read zerosprey42's `pcb/zerosprey42.kicad_sch` schematic (sudah ada lokal, Phase 2 task)

**Penting:** zerosprey42 uses **42 individual switches direct-connected** (no matrix multiplexing? unlikely — likely uses matrix tapi RP2040 has plenty GPIO). Atau matrix dengan smaller dimensions (e.g., 6×7 or similar). PCB-back image menunjukkan diode per switch → matrix-scanned.

PCB-back visual menunjukkan **RP2040-Zero footprint di top center** dengan dual-row castellated edge pads — pattern reference untuk our PCB footprint design Phase 3.

> **Action item Phase 2:** Open zerosprey42 schematic untuk extract its exact pin allocation. Cross-validate vs proposal kita untuk catch any RP2040-Zero-specific gotcha (e.g., pins yang ternyata problematic di production).

---

## G. RGB LED data chain reference

[KLOR_LEDorder.png](../../references/KLOR-upstream/docs/images/KLOR_LEDorder.png) menunjukkan **WS2812/SK6812 daisy-chain order** per side:

- **Chain entry:** MCU GP0 → LED index 00 (bottom-right thumb area)
- **Chain progresses:** thumb cluster → bottom alpha row → home row → top alpha row
- **Chain exit:** LED index 20 (top-left)
- Per side: 21 LEDs total (matches `RGBLED_SPLIT { 21, 21 }` di config.h:86)

For RP2040-Zero target:
- GP0 still drives LED chain entry (no change)
- Chain order **preserved as-is** (no key layout changes per mod scope)
- `rgb_matrix.layout` di keyboard.json **unchanged** (matrix coordinates dan visual positions tetap)

> **Implication for Phase 3 PCB:** When relocating MCU footprint to RP2040-Zero physical position, ensure GP0 trace can reach the LED chain entry point on PCB tanpa awkward routing. Current fork uses ProMicro pin 1 (top-left of MCU) untuk GP0. RP2040-Zero edge GP0 pad will be at different physical position — Phase 3 plan trace accordingly.

---

## H. Untouched fork conventions (no change)

| Aspect | Fork value | Our value | Reason |
|---|---|---|---|
| MATRIX_ROWS | 8 (logical, split-aware) | 8 | Unchanged |
| MATRIX_COLS | 6 | 6 | Unchanged |
| DIODE_DIRECTION | COL2ROW | COL2ROW | Unchanged |
| DEBOUNCE | 5 ms | 5 ms | Unchanged |
| SPLIT_USB_DETECT | Enabled | Enabled | Unchanged |
| RGBLED_SPLIT | { 21, 21 } | { 21, 21 } | Unchanged (per side count) |
| OLED I²C address | Default 0x3C | 0x3C | Unchanged (SSD1306 standard) |

---

## I. Open items (Phase 2+)

1. ~~**Final col-pin order locked di Phase 3** based on PCB routing optimization.~~ ✅ **RESOLVED 2026-05-14** — Locked di schematic MOD #1 (commit `ee98b40`): ascending order `[GP27, GP26, GP9, GP10, GP11, GP12]`. User-accepted untuk natural label sequence.
2. ~~**GP4 routing decision Phase 2:** route to USB-C connector unconditionally?~~ ✅ **RESOLVED 2026-05-14** — Routed in schematic MOD #2 (commit `fa041cc`). GP1 + GP4 both connect ke USB-C inter-half data pins. Firmware enables half-duplex GP1 initially; GP4 reserve untuk future full-duplex upgrade.
3. ~~**CC1/CC2 USB-C handling Phase 2:** tie to 5.1kΩ pulldown?~~ ✅ **RESOLVED 2026-05-14** — Implemented in MOD #2 (commit `fa041cc`). 5.1kΩ pulldown ke GND each. PWR_FLAG on `+5V` net to satisfy ERC for passive VBUS pin.
4. **Encoder push-button matrix scanning:** keyboard.json LAYOUT_polydactyl has L35/R30 di matrix [3,5]/[7,5] sebagai encoder push position. Verify physical PCB encoder footprint connects switch terminals correctly — per new col mapping: row 3 (GP8) + col 5 (**GP12**, was GP10) for left, row 7 (GP8) + col 5 (**GP12**) for right. **Phase 3 PCB layout task.**
5. **zerosprey42 schematic pin extraction** — deferred non-critical. Our pin allocation already cross-validated via Justin Lam's blog + Waveshare wiki.
6. ~~**PROJECT_SPEC.md updates batch**~~ ✅ **RESOLVED 2026-05-13/14** — Phase 1 batch applied (OLED, serial, R1, col pins). Phase 2 follow-up applied 2026-05-14 (ascending col order, RESET GP15, PWR_FLAG note).
7. **NEW — RESET button hardware fallback:** GP15 soft-reset works via Vial keycode dari host PC. Untuk "Windows startup recovery" scenario (PC won't boot, can't send keycode), user needs hardware bootloader entry → fine-wire jumper between RP2040-Zero **RUN** test pad + GND. **Defer ke Phase 5 assembly:** decide whether to add jumper trace di PCB atau leave as soldering exercise during build.

---

## J. Summary table — bottom line untuk Phase 2 schematic

**RP2040-Zero pin allocation untuk PCB schematic Phase 2:**

```
GP0  → WS2812 RGB DI               (PIO-driven, route to LED chain entry)
GP1  → Serial TX (half-duplex)     (route to USB-C inter-half data pin)
GP2  → I²C1 SDA (OLED)             (route to OLED pad 4 SDA)
GP3  → I²C1 SCL (OLED)             (route to OLED pad 3 SCL)
GP4  → Serial RX reserve           (route to USB-C inter-half pin, NC initially di firmware)
GP5  → Matrix row 0
GP6  → Matrix row 1
GP7  → Matrix row 2
GP8  → Matrix row 3
GP9  → Matrix col 2
GP10 → Matrix col 3
GP11 → Matrix col 4
GP12 → Matrix col 5
GP13 → (free, spare)
GP14 → (free, spare)
GP15 → RESET button (soft reset, user-accessible)
GP26 → Matrix col 1
GP27 → Matrix col 0
GP28 → Encoder A (left half) / B (right half)
GP29 → Encoder B (left half) / A (right half)
VBUS → 5V from USB-C (USB1 primary + USB-C inter-half VBUS sharing)
3V3  → Logic level (RP2040-Zero internal LDO output, available on castellated)
GND  → ground reference
```

---

## K. Cross-references

- [INVENTORY.md](INVENTORY.md) — Phase 1 #1 file census
- [MATRIX_VALIDATION.md](MATRIX_VALIDATION.md) — Matrix dimensions + initial pin findings
- [FOOTPRINT_INVENTORY.md](FOOTPRINT_INVENTORY.md) — Library audit
- [KLORpinout.png](../../references/KLOR-upstream/docs/images/KLORpinout.png) — Fork pinout visual
- [KLOR_LEDorder.png](../../references/KLOR-upstream/docs/images/KLOR_LEDorder.png) — RGB chain order
- [zerosprey42 README](../../references/zerosprey42/README.md) — RP2040-Zero exemplar
- [Waveshare RP2040-Zero wiki](https://www.waveshare.com/wiki/RP2040-Zero) — GPIO availability spec
- [keyboard.json](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/keyboard.json) — Pin source of truth (fork current)
- [config.h](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) — Firmware config (fork current)

---

## L. Next deliverable

**Phase 1 deliverable #5: `REFERENCE_ANALYSIS.md`**

Scope:
- Compare zerosprey42 pin mapping (extract dari schematic file) vs proposal kita — catch any RP2040-Zero pin gotchas
- Lessons dari Justin Lam handwired Skeletyl blog (PIO serial, double-tap bootloader reset)
- Lefuneste83 fork analysis — what diverged dari upstream GEIGEIGEIST, why
- Compatibility check: KiCad 10 vs fork's KiCad 6/7/8 format (migration risks)
- Risk register untuk Phase 2 (things to watch saat schematic mod)

Estimated output: ~200 baris di `docs/phase-1-recon/REFERENCE_ANALYSIS.md`.

Confirm "go", atau ada catatan untuk PIN_MAPPING_PROPOSAL.md dulu?
