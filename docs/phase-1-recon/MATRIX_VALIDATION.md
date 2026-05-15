# Phase 1 / Deliverable #2 — MATRIX_VALIDATION

> Validates matrix dimensions, pin mapping, layout structure, and GPIO budget untuk polydactyl layout target. Read-only audit. Critical findings di Section F akan affect Phase 2 PCB redesign.

---

## A. Matrix dimensions (confirmed)

| Setting | Value | Source |
|---|---|---|
| MATRIX_ROWS | **8** | [config.h:35](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) |
| MATRIX_COLS | **6** | [config.h:36](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) |
| DIODE_DIRECTION | COL2ROW | [keyboard.json:17](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/keyboard.json) |
| DEBOUNCE | 5 ms | [config.h:37](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) |

**Topology:** Split keyboard pakai QMK row-offset model.
- Each half punya 4 row pins fisik (hardware rows)
- QMK menggabungkan jadi 8 logical rows: **left half = rows 0-3, right half = rows 4-7**
- 6 col pins shared (each half scans col-nya sendiri independently)
- Logical matrix: **8 × 6 = 48 max keys**
- Polydactyl variant uses **44 of 48 slots** (4 unused: row 0 col 0 left, row 3 col 0 left, row 4 col 0 right, row 7 col 0 right)

## B. Matrix pin assignment (fork current)

### Rows (4 pins per half — same array, software offsets right half)

| Logical row L / R | Pin (RP2040 GPxx) | Source |
|---|---|---|
| 0 / 4 | GP5 | `keyboard.json` matrix_pins.rows[0] |
| 1 / 5 | GP6 | matrix_pins.rows[1] |
| 2 / 6 | GP7 | matrix_pins.rows[2] |
| 3 / 7 | GP8 | matrix_pins.rows[3] |

### Cols (6 pins, same array per half)

| Logical col | Pin (RP2040 GPxx) | Source | RP2040-Zero exposed? |
|---|---|---|---|
| 0 | GP27 | matrix_pins.cols[0] | ✅ broken out (ADC2 pad) |
| 1 | GP26 | matrix_pins.cols[1] | ✅ broken out (ADC1 pad) |
| 2 | GP22 | matrix_pins.cols[2] | ❌ **NOT broken out on RP2040-Zero** |
| 3 | GP20 | matrix_pins.cols[3] | ❌ **NOT broken out** |
| 4 | GP23 | matrix_pins.cols[4] | ❌ **NOT broken out** |
| 5 | GP21 | matrix_pins.cols[5] | ❌ **NOT broken out** |

> 🚨 **Critical** — 4 of 6 col pins tidak available on RP2040-Zero footprint. See Section F.1.

### Note: C macros commented out in config.h

[config.h:32-34](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h):
```c
//#define MATRIX_ROW_PINS { GP5, GP6, GP7, GP8 }
//#define MATRIX_COL_PINS { GP27, GP26, GP22, GP20, GP23, GP21 }
//#define DIODE_DIRECTION COL2ROW
```

Modern QMK is **data-driven via keyboard.json** — C macros are vestigial comments untuk legacy reference. Source of truth = keyboard.json.

## C. Other pin assignments (full GPIO budget snapshot)

| GPIO | Function | Source | RP2040-Zero exposed? | Status |
|---|---|---|---|---|
| GP0 | WS2812 RGB DI | `keyboard.json` ws2812.pin | ✅ edge | KEEP |
| GP1 | Serial (currently half-duplex TX) | [config.h:49](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) | ✅ edge | KEEP |
| GP2 | I²C1 SDA (OLED data) | [config.h:26](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) | ✅ edge | KEEP |
| GP3 | I²C1 SCL (OLED clock) | [config.h:25](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) | ✅ edge | KEEP |
| GP4 | (reserved for full-duplex serial RX) | config.h:52 commented | ✅ edge | Phase 2 lock-in |
| GP5–GP8 | Matrix rows × 4 | matrix_pins.rows | ✅ edge | KEEP |
| GP9 | Audio PWM (DROPPED feature) | [config.h:115](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) | ✅ edge | **FREED** |
| GP10–GP15 | Unused di fork | — | ✅ edge | **Available untuk col remap (F.1)** |
| GP16 | Internal RGB LED on RP2040-Zero board (status indicator) | Waveshare wiki | ⚠️ not on edge | Skip — reserved by board |
| GP17–GP19, GP24, GP25 | Not broken out on RP2040-Zero | — | ❌ | Skip permanently |
| GP20–GP23 | **Matrix cols 2-5 (CURRENT fork mapping)** | matrix_pins.cols | ❌ NOT on RP2040-Zero | **MUST REMAP** (F.1) |
| GP26, GP27 | Matrix cols 0-1 | matrix_pins.cols | ✅ ADC pads | KEEP |
| GP28, GP29 | Encoder A, B (left half) | `keyboard.json` encoder | ✅ ADC pads | KEEP |

## D. Inter-half serial transport — current state

**Currently active: half-duplex single-wire on GP1.** Full-duplex documented but commented out.

[config.h:48-55](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h):
```c
//Half Duplex communication        ← ACTIVE
#define SERIAL_USART_TX_PIN GP1

//Full Duplex communication        ← COMMENTED OUT
//#define SERIAL_USART_TX_PIN GP4
//#define SERIAL_USART_RX_PIN GP1
//#define SERIAL_USART_FULL_DUPLEX
//#define SERIAL_USART_PIN_SWAP
```

> 🔧 **Correction to earlier assumption** (Phase 0.5 framework): saya tulis "full-duplex GP1/GP4 active" — that's wrong. Fork's compiled firmware uses **half-duplex GP1 only**. Full-duplex documented as available option, tapi not active. Will update [PROJECT_SPEC.md](../../PROJECT_SPEC.md) Component Decisions after Phase 1 done.

**Implications untuk USB-C inter-half wiring:**

| Transport | Wires needed | GP usage | PCB routing |
|---|---|---|---|
| Half-duplex (current) | 3: VBUS + GND + Data | GP1 only | Simple |
| Full-duplex (upgrade) | 4: VBUS + GND + TX + RX | GP4 + GP1 (cross-wired between halves) | Both pads routed to USB-C |

**Decision (Claude, Phase 2):** Design PCB **future-proof** — route both GP1 and GP4 ke USB-C connector pads. Firmware start dengan half-duplex (proven working), upgrade ke full-duplex post-validation. Cost = 1 extra trace, near-zero downside.

## E. LAYOUT_polydactyl — key count & coordinate mapping

### Key count (from keyboard.json LAYOUT_polydactyl)

**44 keys total** — counted by enumerating `keyboard.json` layouts.LAYOUT_polydactyl.layout array entries:

| Position group | Left side | Right side | Subtotal |
|---|---|---|---|
| Top row (cols 1-5) | L01-L05 (5) | R01-R05 (5) | 10 |
| Home row (cols 0-5) | L10-L15 (6) | R10-R15 (6) | 12 |
| Bottom row (cols 0-5) | L20-L25 (6) | R20-R25 (6) | 12 |
| **Polydactyl horn** (col 5, between row 2 & thumb) | **L35 (1)** | **R30 (1)** | **2** |
| Thumb cluster (cols 1-4) | L31-L34 (4) | R31-R34 (4) | 8 |
| **TOTAL** | **22** | **22** | **44** |

### Matrix coordinate → physical position (LEFT half)

```
              ┌───┬───┬───┬───┬───┐
              │L01│L02│L03│L04│L05│      y=0  [matrix row 0]
              │0,1│0,2│0,3│0,4│0,5│
              └───┴───┴───┴───┴───┘
          ┌───┬───┬───┬───┬───┬───┐
          │L10│L11│L12│L13│L14│L15│      y=1  [matrix row 1]
          │1,0│1,1│1,2│1,3│1,4│1,5│
          └───┴───┴───┴───┴───┴───┘
          ┌───┬───┬───┬───┬───┬───┬───┐
          │L20│L21│L22│L23│L24│L25│L35│  y=2  [row 2 + horn at row 3 col 5]
          │2,0│2,1│2,2│2,3│2,4│2,5│3,5│
          └───┴───┴───┴───┴───┴───┴───┘
                  ┌───┬───┬───┬───┐
                  │L31│L32│L33│L34│      y=3  [matrix row 3 thumb]
                  │3,1│3,2│3,3│3,4│
                  └───┴───┴───┴───┘
```

Per cell: top = QMK label, bottom = matrix [row, col].

### Matrix coordinate → physical position (RIGHT half, mirrored)

```
              ┌───┬───┬───┬───┬───┐
              │R01│R02│R03│R04│R05│      y=0  [matrix row 4]
              │4,5│4,4│4,3│4,2│4,1│
              └───┴───┴───┴───┴───┘
          ┌───┬───┬───┬───┬───┬───┐
          │R10│R11│R12│R13│R14│R15│      y=1  [matrix row 5]
          │5,5│5,4│5,3│5,2│5,1│5,0│
          └───┴───┴───┴───┴───┴───┘
        ┌───┬───┬───┬───┬───┬───┬───┐
        │R30│R20│R21│R22│R23│R24│R25│    y=2  [row 6 + horn at row 7 col 5]
        │7,5│6,5│6,4│6,3│6,2│6,1│6,0│
        └───┴───┴───┴───┴───┴───┴───┘
              ┌───┬───┬───┬───┐
              │R31│R32│R33│R34│          y=3  [matrix row 7 thumb]
              │7,4│7,3│7,2│7,1│
              └───┴───┴───┴───┘
```

> **Note:** Right side matrix col indexing reversed (e.g., R01 = [4,5], not [4,1]) — mirror image semantics typical untuk split keyboards. Col 0 secara fisik = outer pinky, di kanan terletak di sisi kanan, di kiri di sisi kiri. Pin GP27 on kedua half = col 0.

## F. Critical findings untuk Phase 2 / 3 / 6

### F.1 🚨 GP20–GP23 not exposed on RP2040-Zero → 4 col pins MUST be remapped

**Issue:** Fork's `matrix_pins.cols` array references GP20, GP22, GP21, GP23 — pin GPxx labels yang elite_pi (RP2040 ProMicro form-factor) expose via Pro Micro pinout. **Waveshare RP2040-Zero only breaks out GP0-15 + GP26-29** = 20 GPIO. GP16-25 (except GP26, GP27) are NOT accessible on the board's castellated edge.

**Affected pins:** GP22, GP20, GP23, GP21 — semua 4 col pins yang bukan GP26/GP27.

**Decision (Claude, Phase 2 prep):** Remap 4 col pins ke GP10-GP15 range (6 contiguous unused pins di RP2040-Zero). Phase 1 #4 PIN_MAPPING_PROPOSAL will lock specific assignment.

Proposed remap (subject Phase 1 #4 refinement):
| Logical col | Old pin (elite_pi) | New pin (RP2040-Zero) |
|---|---|---|
| 0 | GP27 | GP27 ✅ keep |
| 1 | GP26 | GP26 ✅ keep |
| 2 | GP22 | **GP10** (or other in 10-15) |
| 3 | GP20 | **GP11** |
| 4 | GP23 | **GP14** |
| 5 | GP21 | **GP15** |

**Affects:**
- ✅ **PCB routing (Phase 3):** physical traces from MCU pad → matrix col diodes/switches
- ✅ **Firmware (Phase 6):** update `keyboard.json` `matrix_pins.cols` array  
- ✅ **PROJECT_SPEC.md update needed** at end of Phase 1

> **This invalidates the Phase 0 assumption** ("PCB route same GPxx physical traces, firmware delta = zero"). Firmware DOES need update — at least `matrix_pins.cols` array. Other pins (rows GP5-8, RGB GP0, encoder GP28/29, I²C GP2/3, serial GP1) tetap valid pada RP2040-Zero.

### F.2 OLED is 128×64 in fork (not 128×32 as user spec)

[config.h:74](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h):
```c
#define OLED_DISPLAY_128X64
```

Plus footprint named `SSD1306_128x64OLED.kicad_mod`. Both consistent → fork built around 128×64.

**Decision (Claude):** **Accept 128×64** (revise PROJECT_SPEC.md from 128×32).

**Justification:**
- Fork OLED rendering code (klor.c, klorimation.h, glcdfont.c logo bitmap) all tuned untuk 128×64
- Changing to 128×32 = firmware rewrite + animation re-tune + logo re-render
- 128×64 0.96" module = ~28×28mm physical, vs 128×32 0.91" ~30×11mm — both fit klor1_4 PCB OLED slot
- Tokopedia stock both, similar price (~25-35rb)
- More screen real estate untuk status/animations
- ✅ Reduces project scope (one less firmware change)

**Update PROJECT_SPEC.md** after Phase 1 complete: COMPONENT_DECISIONS.OLED → "SSD1306 0.96" 128×64 I²C 4-pin".

### F.3 Serial transport = half-duplex GP1 (not full-duplex as initially assumed)

Already detailed in Section D. PROJECT_SPEC.md Component Decision needs update:
- Was: "full-duplex GP1/GP4"
- Correct: "half-duplex GP1 active, GP4 reserved for future upgrade"
- PCB design (Phase 3): route BOTH GP1 + GP4 to USB-C connector future-proof

### F.4 Polydactyl is 44-key (R1 spec says 42-key)

User REQUIREMENT R1 ("42-key split keyboard, polydactyl layout") doesn't match fork's actual `LAYOUT_polydactyl` = **44 keys**.

Variants di fork dengan key count:
- **saegewerk**: 38 keys
- **yubitsume**: 40 keys
- **konrad**: 42 keys (matches "42-key" label)
- **polydactyl**: 44 keys (matches "polydactyl" label)

**Decision (Claude):** **Accept polydactyl = 44 keys.** User specified "polydactyl" name explicitly; "42-key" likely colloquial label (split42 category) bukan precise count. Polydactyl's defining feature = L35/R30 horn keys → without them it's not polydactyl, it's konrad.

**Update R1 setelah Phase 1 complete:** "44-key split keyboard, polydactyl layout, 22 keys per half".

> 🔔 Flagging this untuk Anda explicit — kalau Anda actually MAU 42-key (konrad shape, no horns), ini moment untuk redirect ke konrad. Default = polydactyl-44 sesuai requirement R1 spirit.

## G. GPIO budget recap (RP2040-Zero, 20 GPIO available)

After audio/haptic dropped + col remap planned:

| Function | GPIO count | Pins |
|---|---|---|
| Matrix rows | 4 | GP5, GP6, GP7, GP8 |
| Matrix cols (post-remap) | 6 | GP26, GP27 + 4 from GP10-15 |
| WS2812 RGB DI | 1 | GP0 |
| Serial (current half-duplex) | 1 | GP1 |
| Serial RX (if full-duplex upgrade) | +1 | GP4 |
| I²C (OLED + future trackball) | 2 | GP2, GP3 |
| Encoder A, B | 2 | GP28, GP29 |
| **TOTAL (current half-duplex)** | **16** | |
| **TOTAL (full-duplex upgrade)** | **17** | |

**RP2040-Zero exposes:** 20 GPIO (GP0-15 + GP26-29).  
**Headroom:** **3-4 GPIO free** (GP9 freed from audio + 2 of GP10-15 after col remap).

✅ **Budget passes comfortably** — proceed dengan RP2040-Zero, no GPIO compromise needed.

## H. Cross-references

- [INVENTORY.md](INVENTORY.md) — Phase 1 #1 baseline
- [PROJECT_SPEC.md](../../PROJECT_SPEC.md) — Component decisions (akan ada updates ke OLED, serial, key count post-Phase 1)
- [keyboard.json](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/keyboard.json) — Pin & matrix source of truth
- [config.h](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) — Firmware-level config
- [Waveshare RP2040-Zero pinout](https://www.waveshare.com/wiki/RP2040-Zero) — GPIO availability reference

## I. Open items (will defer atau lock di subsequent deliverables)

| Item | Status | Resolve at |
|---|---|---|
| Col pin remap specific GP10-15 assignment | Proposed (GP10,11,14,15) | Phase 1 #4 PIN_MAPPING_PROPOSAL |
| OLED footprint pad layout (verify 128×64 fits) | Decided 128×64 | Phase 1 #3 FOOTPRINT_INVENTORY (verify only) |
| Half-duplex vs full-duplex final lock-in | Decided: both routed, half-duplex firmware initial | Phase 2 schematic |
| Polydactyl 44-key acceptance | Recommended accept; flag for user override | THIS deliverable — see Section F.4 |
| PROJECT_SPEC.md updates: OLED 128×64, serial half-duplex, R1 → 44-key | Pending | End of Phase 1 (after #3-#5) |

## J. Next deliverable

**Phase 1 deliverable #3: `FOOTPRINT_INVENTORY.md`**

Scope:
- Identify existing footprints per category (MCU, TRRS, RGB LED, switch, encoder, OLED, diode, reset button)
- Library source per footprint (marbastlib? built-in? custom dalam KLORlib?)
- Identify footprints needed untuk modifications:
  - RP2040-Zero footprint (cari source — marbastlib, community, atau custom)
  - SK6812MINI-E **verify orientation** (apakah really south-facing di fork?)
  - USB-C 16-pin midmount (marbastlib coverage check)
- **Verify SSD1306 128×64 footprint pad layout** consistent dengan keputusan F.2

Estimated read targets: open ~5 `.kicad_mod` files dari `KLOR.pretty/`, marbastlib documentation, OLED module datasheets if needed. Output ~150-200 lines.
