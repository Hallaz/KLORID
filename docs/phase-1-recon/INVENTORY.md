# Phase 1 / Deliverable #1 — INVENTORY

> **Source:** `hardware/` — Lefuneste83 fork  
> **HEAD:** `2dbde45` (2024-09-10)  
> **Method:** read-only audit. No modifications to fork files.  
> **Date generated:** 2026-05-13

---

## A. File tree overview

```
hardware/
├── README.md                    (11.2 KB)  — Fork overview; Rev 1.4 = main build target
├── CHANGELOG.md                 (11.0 KB)  — Dates 2024-07 → 2024-09; mentions Rev 1.5
├── FABNOTES.md                  (5.4 KB)   — First-time builder recommendations
├── FIRMWARE.md                  (3.5 KB)   — Firmware variants overview (QMK + ZMK)
├── LICENSE.md                   (35 KB)    — GNU GPL v3 (full text)
├── .github/FUNDING.yml
├── PCB/                         ← 3 KiCad project variants
│   ├── klor1_3/                 — Rev 1.3 (original-derived, WS2812 3535)
│   ├── klor1_4/                 — Rev 1.4 ← MAIN TARGET (SK6812MINI-E south)
│   ├── klor1_4_LP_KS33/         — Rev 1.4 LP (Gateron KS33 low-profile, beta)
│   ├── Alternate DIY Haptic Module/
│   └── README.md                — Manufacturing notes (JLCPCB params)
├── FIRMWARE/
│   ├── qmk_rp2040/electronlab/klor/  — Main QMK source
│   ├── zmk/config/boards/shields/klor/ — ZMK shield (Nice Nano wireless)
│   └── Ready-to-flash/          — 8 pre-built UF2 binaries
├── case/
│   ├── 3DP/{konrad, polydactyl, saegewerk, yubitsume}/
│   ├── acrylic/{konrad, polydactyl, saegewerk, yubitsume}/
│   └── Alternate Case Design/   — Community v2/v2.1 gasket designs
├── docs/
│   ├── buildguide_{3DP, 3DP_ble, acrylic, acrylic_ble}.md
│   ├── images/                  — ~50 images including KLOR_1.5_picture1-5
│   └── klor_rev1-3_ibom.html    — Interactive BOM (Rev 1.3 only)
└── knob/                        — EC11 knob design
```

**File counts (Glob):**

| Type | Count | Globbed via |
|---|---|---|
| `.kicad_pro` | 3 | `hardware/**/*.kicad_pro` |
| `.kicad_sch` | 3 | `hardware/**/*.kicad_sch` |
| `.kicad_pcb` | 3 | `hardware/**/*.kicad_pcb` |
| `.kicad_sym` | 3 | `hardware/**/*.kicad_sym` |
| `.kicad_mod` (footprints) | 50 | `hardware/**/*.kicad_mod` |
| `.stl` (3DP) | 73 | `hardware/case/**/*.stl` |
| `.step` | 9 | `hardware/case/**/*.step` |
| `.dxf` | 16 | `hardware/case/**/*.dxf` |
| `.svg` | 90 | `hardware/case/**/*.svg` |

---

## B. KiCad files

### B.1 PCB project variants (three)

| Project folder | Variant | Status | Files |
|---|---|---|---|
| `PCB/klor1_3/` | Rev 1.3 | Stable, JLCPCB-passing, WS2812 3535 (older LED) | `klor1_3.kicad_pro/sch/pcb` + `KLORlib.kicad_sym` + `KLOR.pretty/` (16 footprints) |
| `PCB/klor1_4/` | **Rev 1.4 ← MAIN** | Refactored layout, SK6812MINI-E **south-facing** | `klor1_4.kicad_pro/sch/pcb` + `KLORlib.kicad_sym` + `KLOR.pretty/` (17 footprints) |
| `PCB/klor1_4_LP_KS33/` | Rev 1.4 LP | Beta — Gateron KS33 low-profile sockets | `klor1_4_LP_KS33.kicad_pro/sch/pcb` + `KLORlib.kicad_sym` + `KLOR.pretty/` (17 footprints) |

> **Decision input:** target variant untuk mod = **klor1_4** (MX standard hotswap + south-facing LED per fork README:11-12). Verifikasi di Phase 2 setelah migrate KiCad-8 → KiCad-10 di branch terpisah.

Plus folder `PCB/Alternate DIY Haptic Module/` — separate small PCB module project (DIY haptic alternative). Skipped from main mod scope per "haptic dropped" decision.

Source: Glob `hardware/PCB/**/*.kicad_*`.

### B.2 Footprint library `KLOR.pretty/` (klor1_4 — 17 footprints)

| Footprint file | Function | Relevance to our 3 mods |
|---|---|---|
| `ProMicro.kicad_mod` | MCU footprint (Pro Micro / Elite-Pi pinout) | **MOD #1 target** — replace with RP2040-Zero |
| `MJ-4PP-9.kicad_mod` | TRRS jack (MJ-4PP-9 standard) | **MOD #2 target** — replace with USB-C 16-pin midmount |
| `SK6812MINI_and_cherry.kicad_mod` | Combined LED + MX switch footprint | **MOD #3 verify** — README claims fork sudah south-facing |
| `SK6812MINI_and_cherry_1.kicad_mod` | Variant ke-2 dari combined footprint | TBD inspect; orientation difference? |
| `Pimoroni Haptic Buzz DRV2605L Driver.kicad_mod` | Pimoroni haptic module | **DROP per decision** — remove placements in PCB |
| `Buzzer_Mallory_AST1109MLTRQ_reversible.kicad_mod` | PWM speaker | **DROP per decision** |
| `MSK12C02_reversible.kicad_mod` | Slide power switch | KEEP — useful for wired build? FABNOTES says optional |
| `Diode_SOD123.kicad_mod` | 1N4148W matrix diode | KEEP unchanged |
| `RotaryEncoder_Alps_EC11E-Switch_Vertical_H20mm-keebio_modified.kicad_mod` | EC11 encoder | KEEP unchanged |
| `SSD1306_128x64OLED.kicad_mod` | OLED footprint | **VERIFY** — name says 128×64 tapi spec kita 128×32 (lihat Section G #3) |
| `SplitkbTentingPuckWithoutCenterHole.kicad_mod` | SplitKB tenting puck mount | KEEP |
| `JST_PH_reversible.kicad_mod` | Battery connector | KEEP (battery option ada di KLOR) |
| `custombatteryconnector_small.kicad_mod` | Alt battery connector | KEEP |
| `Jumper.kicad_mod` | Solder jumper (triangular) | KEEP |
| `R_1206_DoubleSided.kicad_mod` | Resistor 1206 SMD | KEEP |
| `lvc27_onlyInput.kicad_mod` | 74LVC2G17 buffer | TBD — relevance unclear; mungkin terkait haptic/audio gate (drop if so) |

`klor1_4_LP_KS33` adds: `KS33_Symetrical.kicad_mod` (low-profile Gateron socket — tidak relevan untuk MX hotswap target).

Source: Glob `hardware/PCB/**/*.kicad_mod`.

### B.3 Footprints needed but ABSENT (must add at Phase 2)

| Needed for | Footprint | Source candidate |
|---|---|---|
| MOD #1 | Waveshare RP2040-Zero (NOT clone) | marbastlib has variant; verify di Phase 1 deliverable #3 FOOTPRINT_INVENTORY |
| MOD #2 | USB-C 16-pin midmount (specific part TBD Phase 2) | marbastlib `USB_C_Midmount` family or KiCad community lib |
| MOD #3 verify | SK6812MINI-E reverse-mount (kalau existing tidak sesuai) | Bastardkb marbastlib likely has |

---

## C. Firmware files

### C.1 QMK source (`FIRMWARE/qmk_rp2040/electronlab/klor/`)

| File | Lines (approx) | Purpose |
|---|---|---|
| `keyboard.json` | 394 | **Already verified** — `development_board: elite_pi`, matrix pins GP5-8/20-27, RGB GP0, encoder GP28/29, split_count [21,21], 4 LAYOUT variants |
| `config.h` | TBD (Phase 1 deliverable #2) | Hardware-level settings; **OLED resolution di sini** |
| `halconf.h` | small | ChibiOS HAL config |
| `mcuconf.h` | small | RP2040 MCU clock/peripheral config |
| `klor.c` / `klor.h` | larger | Custom matrix scan, OLED rendering, haptic, audio, animations |
| `rules.mk` | small | Build flags |
| `lib/glcdfont.c` | large | OLED bitmap font |
| `lib/klorimation.h` | medium | OLED animation frames |
| `lib/klounds.h` | medium | Audio note/sound clips (DROP per audio decision) |
| `keymaps/default/` | 3 files | QMK keymap (config.h, keymap.c, rules.mk) |
| `keymaps/vial/` | 5 files | Vial-QMK keymap (+ klor.vil + vial.json) |

**QMK build path:** `qmk compile -kb electronlab/klor -km {default,vial}` (per FIRMWARE.md:21-25)

**Inter-half transport (per CHANGELOG.md & FIRMWARE.md):**
```c
// Full duplex serial (recommended for RP2040 ProMicro):
#define SERIAL_USART_TX_PIN GP4
#define SERIAL_USART_RX_PIN GP1
#define SERIAL_USART_FULL_DUPLEX
#define SERIAL_USART_PIN_SWAP
```
> Reference: CHANGELOG.md:69-73 + FIRMWARE.md:43-47

**Required QMK-root patch (per CHANGELOG.md:46-50):**  
File `./platforms/chibios/boards/QMK_PM2040/configs/mcuconf.h`:
```c
#define RP_PWM_USE_PWM4   TRUE   // was FALSE
```
> Gotcha untuk Phase 6 build setup — bukan edit di keyboard folder.

### C.2 ZMK source (`FIRMWARE/zmk/config/boards/shields/klor/`)

Shield-style ZMK config:
- Board overlays: `nice_nano.overlay`, `nice_nano_v2.overlay`, `nrfmicro_11.overlay`, `nrfmicro_11_flipped.overlay`, `nrfmicro_13.overlay`
- Device tree: `klor.dtsi`, `klor_common.dtsi`, `klor_{left,right}.overlay`, `klor.keymap`
- Side configs: `klor_left.conf`, `klor_right.conf`
- Battery monitoring: `battery_status.c/h` + 12 icon files (`bat_00.c`, `bat_25.c`, etc.)
- Bluetooth icons: 8 files (`bt_out_*.c`, `bt_pro_*.c`)
- Logo: `klor_logo.c`, `usb_out.c`
- Kconfig: `Kconfig.shield`, `Kconfig.defconfig`
- Shield metadata: `klor.zmk.yml`

> **Not relevant untuk build kita** (RP2040 wired, no BLE). Leave as-is di firmware fork nanti, atau strip kalau Anda mau bersihkan repo Phase 6.

### C.3 Pre-built UF2 binaries (`FIRMWARE/Ready-to-flash/`)

| Path | Variant |
|---|---|
| `qmk-RP2040/QMK/klor_default_{left,master_left,right}.uf2` | QMK default keymap |
| `qmk-RP2040/VIAL/klor_vial_{left,master_left,right}.uf2` | Vial-QMK keymap |
| `zmk-NiceNanoV2/klor_{left,right}-nice_nano_v2-zmk.uf2` | ZMK Nice Nano BLE |

> 8 pre-built binaries — akan **stale** setelah Phase 6 firmware rebuild. Pertimbangkan delete pasca-Phase 6 untuk avoid confusion.

---

## D. Case files

### D.1 3DP per layout variant (4 variants × 4 sub-folders)

Per variant (konrad, polydactyl, saegewerk, yubitsume) sub-folders:
- `acrylic/` — DXF + SVG untuk laser-cut acrylic plate (full version + logotype)
- `bluetooth/` — STL untuk BLE-specific case (skip untuk wired build)
- `regular/` — STL untuk wired case (case L, R, puck L, puck R)
- `switchplate/` — DXF + STEP + STL + SVG + ZIP (gerber untuk FR4 plate)

**Polydactyl-specific files (target Anda):**

| File | Size | Purpose |
|---|---|---|
| `case/3DP/polydactyl/regular/KLOR_polydactyl_case_L.stl` | 383 KB | Left half case |
| `case/3DP/polydactyl/regular/KLOR_polydactyl_case_R.stl` | 383 KB | Right half case |
| `case/3DP/polydactyl/regular/KLOR_polydactyl_case_puck_L.stl` | 364 KB | Left with SplitKB puck cutout |
| `case/3DP/polydactyl/regular/KLOR_polydactyl_case_puck_R.stl` | 364 KB | Right with puck cutout |
| `case/3DP/polydactyl/switchplate/KLOR_polydactyl_3DP_switchplate.{stl,step,dxf,svg,zip}` | 5 formats | FR4-cut switchplate |
| `case/3DP/polydactyl/acrylic/KLOR_polydactyl_3DPcase_acrylic.{dxf,svg}` | 4 files (+ logotype variants) | Bottom acrylic plate untuk 3DP case |

### D.2 Acrylic stacked case (polydactyl)

Layer-by-layer SVG untuk laser-cut acrylic stacked case:
- 6 layers: `01_bottom`, `02`, `03`, `04`, `05_top`, `06_ring`
- Plus `01_bottom_puck` variant (untuk SplitKB puck)
- Plus `_typo` (logotype embedded) variant untuk top layer
- Plus `_ble` variants per layer (untuk BLE build — skip)
- Plus `KLOR_polydactyl_acrylic_case_switchplate.{dxf,step,stl,svg,zip}`

Total ~16 SVG + ~1 DXF + ~1 STEP untuk polydactyl acrylic case (regular + BLE combined).

### D.3 Alternate Case Design

`case/Alternate Case Design/` — community contribution:
- `klor-case-designs-v52.step` — 28 MB combined source STEP for all 4 variants (v52)
- `Konrad/` — Base, BaseWithTenting, Lid variants
- `Polydactyl/` — v2/v2.1 Gasket designs (Lid, Base, SwitchPlate)
- `Saegewerk/`, `Yubitsume/` — similar
- `Tenting Legs/files/` — 7 STL variants (26mm, 36mm, 38mm, 40mm, 42mm, ALT31, ALT37)
- Encoder inserts: `EncoderInsert.stl`, `EncoderInsert-flat.stl`, `EncoderInsert-16mm.stl`, `EncoderInsert-flat-16mm.stl`
- Documentation: `LICENSE.txt`, `README.txt`, PDF guide `535004-case-designs...pdf` (245 KB)

> Worth keeping for reference. Not in mod scope; gunakan kalau Anda mau gasket-style enclosure.

---

## E. License & contribution status

### E.1 License: **GNU GPL v3** (confirmed)

`hardware/LICENSE.md:1-2`:
```
                    GNU GENERAL PUBLIC LICENSE
                       Version 3, 29 June 2007
```

**Implications untuk fork kita:**
- ✅ Commercial use diperbolehkan
- ✅ Modifikasi diperbolehkan (sesuai mod scope kita)
- ✅ Distribusi binary diperbolehkan
- ⚠️ **Copyleft (strong):** kalau kita distribute board (PCB + firmware bundled), source code modified harus GPL-3.0 dan publicly available
- ⚠️ Hardware design files: GPL applicability ke schematic/PCB layout debatable, tapi spirit GPL = share-alike
- ✅ Hobbyist personal build (no distribution): no obligation

> Source: `hardware/LICENSE.md` (35 KB = standard GPL-3.0 full text).

### E.2 Git activity stats

| Metric | Value | Source |
|---|---|---|
| Fork HEAD | `2dbde45` | `git rev-parse HEAD` |
| Upstream HEAD | `e2f9c2c` | `git rev-parse HEAD` di `references/KLOR-upstream/` |
| Fork commits ahead of upstream | **152** | `git log --oneline e2f9c2c..HEAD \| wc -l` |
| Files changed vs upstream | **248** | `git diff --name-only e2f9c2c HEAD \| wc -l` |
| Total commits (fork HEAD ancestry) | 469 | `git rev-list --count HEAD` |
| First commit (inherited from upstream) | 2022-06-21 "Initial commit" | `git log --reverse` |
| Last commit on fork | **2024-09-10** | `git log -1` |
| Time stale (as of 2026-05-13) | **~20 bulan** | calculated |

### E.3 Top contributors (fork repo, includes upstream history)

| Commits | Name | Likely identity |
|---|---|---|
| 291 | schwarzgrau | Upstream GEIGEIGEIST maintainer (inherited) |
| 113 | Lefuneste83 | **Fork maintainer** (GitHub username) |
| 31 | Hilarion Lefuneste | Same person, real name attribution |
| 17 | GEIST | Co-maintainer upstream |
| 4 | moritz-john | Community contributor |

> Lefuneste83 + Hilarion = 144 commits (~31% of total). Active fork divergence focused on firmware modernization (per file change pattern below).

### E.4 Files changed in fork (sample, 30 of 248)

Predominantly **firmware**:
- `FIRMWARE/qmk_rp2040/electronlab/klor/` — all source files (config.h, halconf.h, mcuconf.h, klor.c/h, keyboard.json, lib/*)
- `FIRMWARE/qmk_rp2040/electronlab/klor/keymaps/{default,vial}/` — full keymap rewrites
- `FIRMWARE/Ready-to-flash/qmk-RP2040/` + `zmk-NiceNanoV2/` — pre-built binaries
- Top-level docs: CHANGELOG, FABNOTES, FIRMWARE.md, README

Suggests Lefuneste83's contribution focus = **firmware modernization** (QMK RP2040 elite_pi config, Vial support, full-duplex serial, PWM4 workaround), bukan major PCB layout redesign. Detailed PCB-level diff perlu dilakukan di Phase 1 deliverable #5 REFERENCE_ANALYSIS.

---

## F. Documentation files

| File | Notable content |
|---|---|
| `README.md` | Rev 1.4 = main build target. Mentions Rev 1.5 design tapi PCB files tidak ada. References Sofle/Kyria/Yacc46/Elephant42 sebagai inspiration. Listed credits: KarlK90, MangoIV, 0xCB-dev, microfortnight, dreipunkteinsvier. |
| `CHANGELOG.md` | Last entry 2024-09-10. Rev 1.5 announced dengan major improvements (direct USB 5V, full duplex serial, hardware handedness, OLED voltage select), tapi PCB files belum committed. Full-duplex serial snippet di lines 69-73. PWM4 patch di lines 46-50. |
| `FABNOTES.md` | SK6812MINI-E orientation gotcha, Gateron socket asymmetry, Pimoroni haptic 15 EUR each, **Mill-Max 0.5mm gold wire** rekomendasi untuk socketed MCU mount, power switch optional untuk wired build. |
| `FIRMWARE.md` | Build commands per environment. **Note correction:** keyboard path adalah `electronlab/klor` (bukan `klor`) per FIRMWARE.md:21,25 (newer convention). |
| `PCB/README.md` | JLCPCB manufacturing params: 159.8 mm × 118 mm, 1.6mm thickness, 2 layers, **HASL or LeadFree HASL** finish, **Remove Order Number: Yes**. References gerber ZIP — but path points to Rev 1.3 only. |
| `docs/buildguide_*.md` | 4 guides: 3DP + 3DP_ble + acrylic + acrylic_ble (only `3DP_ble.md` is tiny stub 0.1 KB) — relevant Phase 5 assembly. |
| `docs/klor_rev1-3_ibom.html` | Interactive BOM Rev 1.3 only. **Rev 1.4 ibom belum di-generate** — bisa kita generate di Phase 4 untuk our custom rev. |

---

## G. Findings to flag for Phase 2

Hal-hal non-obvious yang akan affect Phase 2+ decisions:

1. **Rev 1.5 announced tapi PCB files absent.** CHANGELOG 2024-09-10 menunjukkan major design improvements (direct USB 5V distribution antar halves, full-duplex serial, hardware handedness detection, OLED voltage select 3.3V/5V). Tapi `PCB/` hanya berisi 1_3, 1_4, 1_4_LP_KS33. Confirm: kita fork dari **1.4** sebagai base. Worth investigating apakah 1.5 ada di branch lain (Phase 1 deliverable #5 REFERENCE_ANALYSIS).

2. **klor1_4 README claims south-facing LED.** `hardware/README.md:37` ("As Hipyo Tech can confirm, we are in 2024 so no North facing LED are allowed anymore. I have moved to South facing LED."). **MOD #3 mungkin sudah largely done** di fork. Verify di Phase 1 deliverable #3 FOOTPRINT_INVENTORY (open `SK6812MINI_and_cherry.kicad_mod` dan inspect pad orientation).

3. **OLED footprint nama `SSD1306_128x64OLED` vs spec 128×32.** Need to inspect footprint file di Phase 1 deliverable #2 MATRIX_VALIDATION untuk verify apakah ini universal SSD1306 pad layout (same untuk 128×32 dan 128×64) atau spesifik 128×64. Also cek `config.h` untuk `OLED_DISPLAY_HEIGHT` setting.

4. **Full-duplex serial config sudah documented** di CHANGELOG.md:69-73 — `TX=GP4, RX=GP1, FULL_DUPLEX, PIN_SWAP`. Ini base untuk PIN_MAPPING_PROPOSAL (Phase 1 deliverable #4). Konsisten dengan Justin Lam blog yang mention GP1 untuk PIO serial.

5. **PWM4 patch needed di QMK root** (CHANGELOG.md:46-50): file `./platforms/chibios/boards/QMK_PM2040/configs/mcuconf.h` set `RP_PWM_USE_PWM4 TRUE`. Gotcha untuk Phase 6 build — diff terhadap upstream QMK tree, bukan keyboard folder.

6. **Footprint `lvc27_onlyInput` (74LVC2G17 buffer).** Function unclear tanpa baca schematic. Mungkin terkait haptic atau audio signal gating — kalau iya, bisa removed dari layout per drop decision. Verify di Phase 2 schematic mod.

7. **Mill-Max 0.5mm gold wire** disebut FABNOTES sebagai best practice untuk socketed MCU. Bukan COTS part — flag for Phase 5 sourcing (Aliexpress / Mouser).

8. **GPL-3.0 license.** Different dari hipotesis awal CC-BY-NC-SA. Allow commercial + modifikasi tapi copyleft. Hobbyist build = no constraint. Distribution = full source must be available with same license.

9. **Fork ~20 bulan stale.** Lefuneste83 last commit 2024-09-10. Tidak ada upstream maintenance — bug discovered = self-fix. Tidak ada PR community yang merged di antara 2024-09 dan hari ini (verify Phase 1 #5).

10. **`development_board: elite_pi`** di `keyboard.json` mengindikasikan firmware sudah fully RP2040-aware. Setiap pin label sudah `GPxx`. **Implikasi:** kalau PCB kita routing RP2040-Zero ke pin label yang SAMA dengan elite_pi target, firmware delta = minimal (perhaps zero outside dropping haptic/audio).

---

## H. Cross-references

- [PROJECT_SPEC.md](../../PROJECT_SPEC.md) — Locked hardware spec (3 mods scope)
- [PHASE_LOG.md](../../PHASE_LOG.md) — Decision log
- [CONTRIBUTING.md](../../CONTRIBUTING.md) — Project guidance untuk Claude sessions
- [README.md](../../README.md) — Project overview
- [keyboard.json](../../hardware/FIRMWARE/qmk_rp2040/electronlab/klor/keyboard.json) — Pin mapping ground truth
- [hardware/CHANGELOG.md](../../hardware/CHANGELOG.md) — Fork change history
- [hardware/README.md](../../hardware/README.md) — Fork overview

---

## I. Next deliverable (proposed)

**Phase 1 deliverable #2: `MATRIX_VALIDATION.md`**

Scope (per kickoff spec):
- Konfirmasi matrix dimensions polydactyl variant (expected 4 row × 6 col, 21 keys per half)
- Extract `MATRIX_ROW_PINS` dan `MATRIX_COL_PINS` dari `config.h` (cross-reference `keyboard.json` matrix_pins)
- Identifikasi pin Pro Micro original (D0-D10 mapping) — note: fork sudah RP2040, jadi this maps to elite_pi GPxx labels
- Map polydactyl layout (21 keys per half) ke posisi matrix coordinates

Estimated read targets: `config.h`, `keymaps/default/keymap.c`, `klor.h`, `keyboard.json` (full re-read). Output: ~150 lines markdown di `docs/phase-1-recon/MATRIX_VALIDATION.md`.

**Confirm "lanjut" atau ada catatan untuk INVENTORY.md dulu?**
