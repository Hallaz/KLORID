# KLOR Custom Build — Project Spec

> **Last updated 2026-05-14** (Phase 2 in-progress — schematic MOD #1 + MOD #2 done, ERC clean). Spec dibagi 2 tier: REQUIREMENTS (locked, user-owned) + COMPONENT DECISIONS (Claude-decided, justified). Tier 1 berubah hanya dengan explicit user discussion. Tier 2 bisa berubah dengan justifikasi teknis/sourcing.

---

## TIER 1: REQUIREMENTS (locked)

User's functional spec — apa keyboard ini harus DO.

| # | Requirement | Notes |
|---|---|---|
| R1 | 42-key split keyboard, polydactyl layout | **21 keys per side + 1 encoder/knob position per side** (total 42 keys + 2 encoders); 19.05 × 19.05 mm key spacing |
| R2 | MX switch ecosystem | Universal MX hotswap socket; switch + keycap MX-compatible |
| R3 | USB-C native MCU primary | No Pro-Micro daughterboard awkward USB |
| R4 | USB-C inter-half connector | Replace TRRS jack |
| R5 | Per-key RGB matrix | 42 LED total (21/side × 2), south-facing orientation |
| R6 | OLED display | Status / mode / animation rendering |
| R7 | Rotary encoder optional via jumper | EC11-style with push button (1 per side default) |
| R8 | JLCPCB-manufacturable | 2-layer FR4 1.6mm, standard DRC compliant |
| R9 | Vial-QMK firmware | Real-time keymap edit without re-flash |
| R10 | Socketable MCU | Untuk prototyping iteration |

---

## TIER 2: COMPONENT DECISIONS

### MCU

| | Decision | Rationale |
|---|---|---|
| **Part** | Original Waveshare RP2040-Zero (not clones) | 20 GPIO sufficient (17 used + 3 headroom), USB-C native, compact, $3-5/unit |
| **Sourcing** | Tokopedia / Shopee (verified seller) | Indonesia sourcing-locked: Elite-Pi unavailable |
| **Mounting** | 1.27mm pitch female pin header (2×12-pin per side) di PCB + 1.27mm male pin header solder ke castellated edge RP2040-Zero | Mill-Max preferred low profile tapi Aliexpress 3-4wk wait. 1.27mm header Tokopedia ~10rb/pasang. Stack ~6mm. |

### Inter-half connector

| | Decision | Rationale |
|---|---|---|
| **Part** | USB-C 16-pin midmount SMD (GCT USB4085 or HRO TYPE-C-31-M-12, final TBD Phase 2) | 16-pin = right pin count; midmount = cleaner PCB profile |
| **Wire allocation (firmware initial)** | **Half-duplex single-wire serial on GP1** (3 wires: VBUS + GND + GP1) | Per fork [config.h:49](hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) currently active |
| **Wire allocation (PCB routing)** | **Route both GP1 + GP4 to USB-C pads** (4 traces) | Future-proof full-duplex upgrade path per fork [CHANGELOG.md:69-73](hardware/CHANGELOG.md). Firmware switch later, no PCB re-spin needed. |
| **CC1, CC2** | Tied to 5.1kΩ pulldown to GND each | Dumb-sink USB-C; implemented in MOD #2 schematic |
| **VBUS / 5V net** | USB-C VBUS pin joined to global `+5V` net (shared with MCU 5V pad + LED VCC). `PWR_FLAG` marker placed on `+5V` net (MCU-side, `#FLG02`) to satisfy ERC | USB-C VBUS pin = passive type — ERC needs PWR_FLAG declared somewhere on net. PWR_FLAG is net-level marker; one instance covers entire `+5V` net regardless of placement. No PCB/physical effect. |
| **Cable** | USB-C to USB-C, ~30cm | Tokopedia sourceable; not part of PCB BOM |

### RGB LED

| | Decision | Rationale |
|---|---|---|
| **Part** | SK6812MINI-E (6028 size), south-facing reverse-mount | Better current + uniformity vs WS2812 3535. Fork klor1_4 sudah south-facing per [README:37](hardware/README.md) |
| **Count** | 42 (21 per side × 2) | Per R5 |
| **Footprint variant** | `SK6812MINI_and_cherry_1` (already di klor1_4 KLOR.pretty) — dual-layer pad placement + Edge.Cuts cutout zone | **No new footprint needed** ([FOOTPRINT_INVENTORY C.3](docs/phase-1-recon/FOOTPRINT_INVENTORY.md)) |
| **Assembly** | Hand-solder reverse-mount dengan hot air / reflow plate | User confirmed SMD capability |

### OLED

| | Decision | Rationale |
|---|---|---|
| **Part** | SSD1306 **0.96" 128×64** I²C 4-pin | **Revised from 128×32** — fork rendering code, animation library (`klorimation.h`), logo bitmap semua tuned untuk 128×64. Switching = significant firmware rewrite untuk zero benefit |
| **Footprint** | `SSD1306_128x64OLED` (existing klor1_4 KLOR.pretty) | Outline ~27×27mm = 0.96" form factor. Pin pitch 2.54mm universal |
| **Sourcing** | Tokopedia / Aliexpress (~25-35rb/unit) | Abundant |

### Encoder

| | Decision | Rationale |
|---|---|---|
| **Part** | EC11 vertical mount with push switch | Standard, fork sudah implementasi, sourceable Tokopedia |
| **Config** | Optional via jumper bridge | Fork's design preserved |
| **Encoder push position di matrix** | L35 [3,5] (left), R30 [7,5] (right) — encoder push-button shares matrix position untuk keycode assignment via Vial | Per fork `keyboard.json` LAYOUT_polydactyl |

### Switch socket & matrix

| | Decision | Rationale |
|---|---|---|
| **Hotswap** | Kailh MX hotswap PG1511 | Fork sudah pakai. Universal MX. Tokopedia |
| **Diode** | 1N4148W SOD-123 SMD | Fork's `Diode_SOD123` footprint dual-side reversible |
| **Direction** | COL2ROW | Per fork `keyboard.json:17` |
| **Matrix size** | 8 rows × 6 cols logical (4 rows × 6 cols per half) | Fork-preserved |

### Matrix pin allocation (RP2040-Zero) — REMAPPED

```json
// keyboard.json matrix_pins (Phase 6 firmware update):
"cols": ["GP27", "GP26", "GP9", "GP10", "GP11", "GP12"],
"rows": ["GP5", "GP6", "GP7", "GP8"]
```

| Logical pin | Pin (was, elite_pi) | Pin (new, RP2040-Zero) | Reason |
|---|---|---|---|
| col 0 | GP27 | GP27 | Keep — RP2040-Zero ADC pad |
| col 1 | GP26 | GP26 | Keep — RP2040-Zero ADC pad |
| col 2 | GP22 | **GP9** | Remap — GP22 not exposed on RP2040-Zero edge; ascending order chosen untuk natural label sequence |
| col 3 | GP20 | **GP10** | Remap |
| col 4 | GP23 | **GP11** | Remap |
| col 5 | GP21 | **GP12** | Remap |
| row 0-3 | GP5-GP8 | GP5-GP8 | Keep — exposed on RP2040-Zero edge |

**Additional GPIO assignments** (RP2040-Zero, 20 GPIO total):

| Function | Pin | Notes |
|---|---|---|
| RESET button | **GP15** | Soft reset via firmware (Vial reset keycode). Hardware reset to bootloader requires fine-wire jumper ke RP2040-Zero RUN test pad — defer ke Phase 5 assembly kalau dibutuhkan untuk "Windows startup recovery" use case |
| Spares | GP13, GP14 | 2 free GPIO untuk future expansion (e.g., second encoder, status LED, additional I²C device) |

Full GPIO allocation: [docs/phase-1-recon/PIN_MAPPING_PROPOSAL.md](docs/phase-1-recon/PIN_MAPPING_PROPOSAL.md).

### Base PCB variant

| | Decision | Rationale |
|---|---|---|
| **Variant** | Fork's `PCB/klor1_4/` (Rev 1.4) | Latest with committed files. Rev 1.5 announced but PCB files absent. LP_KS33 = irrelevant (low-profile beta) |

### Case

| | Decision | Rationale |
|---|---|---|
| **Files** | Fork's `case/3DP/polydactyl/regular/*.stl` + `case/3DP/polydactyl/switchplate/*.stl` | Drop-in from fork; no modification needed |
| **Material** | 3D printed (PETG / PLA / ASA — user choice) | Standard FFF print |

### Manufacturing & assembly model

| | Decision | Rationale |
|---|---|---|
| **PCB fab** | JLCPCB, 2-layer FR4 1.6mm, HASL/LeadFree HASL, black, "Remove Order Number: Yes" | Per fork [PCB/README.md:14](hardware/PCB/README.md) |
| **PCBA service** | ❌ NOT used | User has SMD soldering capability — hand-solder all parts |
| **BOM sourcing** | Mix Tokopedia + Aliexpress + LCSC | Indonesia sourcing-flexible |

### Firmware configuration (Phase 6 spec)

Per [Justin Lam handwired Skeletyl blog](https://www.justinmklam.com/posts/2025/08/handwired-skeletyl/) — bare RP2040-Zero requires these specific settings (NOT same as fork's `elite_pi` config):

**`keyboard.json` changes:**
```diff
- "development_board": "elite_pi"
+ "processor": "RP2040",
+ "bootloader": "rp2040"
```
Reason: Bare RP2040-Zero needs explicit processor + bootloader macros, bukan dev_board alias. Otherwise flashing may fail.

**`config.h` additions:**
```c
#define RP2040_BOOTLOADER_DOUBLE_TAP_RESET           // double-tap reset → bootloader entry
#define RP2040_BOOTLOADER_DOUBLE_TAP_RESET_TIMEOUT 500U
#define PICO_XOSC_STARTUP_DELAY_MULTIPLIER 64        // crystal startup delay = reliable boot
#define SERIAL_PIO_USE_PIO1                          // 🔑 PIO1 untuk serial; prevents conflict dengan WS2812 (PIO0)
```

**`rules.mk` addition:**
```make
SERIAL_DRIVER = vendor
```
`vendor` = QMK's RP2040-specific PIO-based serial driver.

---

## DROPPED (out of scope; do not re-add tanpa discussion)

- Haptic feedback (DRV2605L driver + ERM/LRA motor + Pimoroni footprint) — ✅ dropped from schematic commit `17c1fb3`
- Audio (PWM speaker on GP9 + `lib/klounds.h` sound clips) — ✅ Mallory Buzzer BZ1 dropped commit `17c1fb3`
- **Trackball module** (mouse/pointing device section in original fork schematic) — ✅ dropped commit `17c1fb3`. Discovered during Phase 2 housekeeping; was NOT in initial DROPPED list. User explicitly decided to skip pointing-device feature scope.
- ZMK firmware variant (QMK only)
- Bluetooth / Nice Nano support (wired RP2040-Zero only)
- Battery & power switch chain (wired-only build): drop MSK12C02 slide switch + JST-PH + `custombatteryconnector_small` placements — ✅ dropped commit `ee98b40` (MOD #1)
- `lvc27_onlyInput` 74LVC2G17 buffer — ✅ dropped commit `17c1fb3` (confirmed haptic-support related)

---

## PCB Modification Scope (revised after Phase 1)

1. **MOD #1 — MCU footprint:** Elite-Pi (RP2040 Pro-Micro form-factor) → custom Waveshare RP2040-Zero footprint with 1.27mm pitch socket pads. **Plus** re-route 4 matrix col pins per remap (GP22/20/23/21 → GP9/10/11/12, ascending order). RESET button connected ke GP15 (soft reset).

2. **MOD #2 — Inter-half connector:** TRRS jack (MJ-4PP-9) → USB-C 16-pin midmount SMD. Route both GP1 + GP4 to USB-C pins (future-proof).

3. **MOD #3 — RGB orientation (VERIFY ONLY):** Confirm klor1_4 schematic instantiate `SK6812MINI_and_cherry_1` variant (south-facing already) bukan older `SK6812MINI_and_cherry`. If yes, no PCB modification for RGB. Verify at Phase 2 schematic open.

**Plus housekeeping (Phase 2 schematic):**
- Remove placements: Pimoroni Haptic, Mallory buzzer, MSK12C02 slide switch, JST-PH battery, custombatteryconnector, lvc27_onlyInput (if haptic-related)
- Phase 6: update `keyboard.json` per pin remap + firmware config

### KEEP unchanged

- Switch footprint area (MX hotswap PG1511)
- Key spacing (19.05 mm × 19.05 mm)
- Encoder placement (optional via jumper, matrix [3,5]/[7,5] coords for push-button)
- OLED area (same physical slot for 128×64)
- Diode positions
- Matrix routing logic (8 rows × 6 cols, COL2ROW)
- Reset switch placement
- LED chain order (entry at GP0 → LED 00 → daisy-chain to LED 20)

---

## Phase 1 Recon outputs (reference)

All deliverables di `docs/phase-1-recon/`:
- [INVENTORY.md](docs/phase-1-recon/INVENTORY.md) — File census + LICENSE (GPL-3.0) + fork status (stale ~20 bulan)
- [MATRIX_VALIDATION.md](docs/phase-1-recon/MATRIX_VALIDATION.md) — Matrix dimensions, pin audit, GPIO budget
- [FOOTPRINT_INVENTORY.md](docs/phase-1-recon/FOOTPRINT_INVENTORY.md) — Library audit + mod-critical footprints
- [PIN_MAPPING_PROPOSAL.md](docs/phase-1-recon/PIN_MAPPING_PROPOSAL.md) — Final 17/20 GPIO allocation
- [REFERENCE_ANALYSIS.md](docs/phase-1-recon/REFERENCE_ANALYSIS.md) — zerosprey42 + Justin Lam + fork delta + Phase 2 risk register (15 items)
