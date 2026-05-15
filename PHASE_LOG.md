# Phase Log

Append-only chronological log of decisions, milestones, blockers. Latest entries at bottom.

---

## Phase 0 — Project Setup · 2026-05-13 (morning)

**Done:**
- Cloned 4 repos: KLOR (Lefuneste83), KLOR-upstream (GEIGEIGEIST), zerosprey42 (beekeeb), Skeletyl (Bastardkb)
- Workspace reorganized: `hardware/`, `references/{KLOR-upstream, zerosprey42, Skeletyl}/`
- Scaffolded `docs/phase-1-recon/`, `working/{gerbers, bom, firmware, screenshots}/`
- Captured initial spec → [PROJECT_SPEC.md](PROJECT_SPEC.md) (refactored Phase 0.5)
- Captured project overview → [README.md](README.md)
- Captured Claude guidance → [CONTRIBUTING.md](CONTRIBUTING.md)

**Decisions confirmed:**
- Haptic + audio features → DROPPED
- MCU framing: Elite-Pi (RP2040 Pro-Micro form-factor) → Waveshare RP2040-Zero
- KiCad files edited in-place

**Environment:**
- Windows 11 (build 26200); KiCad 10.0.2; Git
- QMK CLI deferred Phase 6

---

## Phase 0.5 — Collaboration framework + spec refactor · 2026-05-13 (afternoon)

**Trigger:** User clarified — electronics + PCB-design background but NOT keyboard DIY expert.

**Framework changes:**
- [PROJECT_SPEC.md](PROJECT_SPEC.md) refactored: 2 tiers (REQUIREMENTS locked + COMPONENT DECISIONS Claude-decided)
- [CONTRIBUTING.md](CONTRIBUTING.md) updated collaboration model + Indonesia sourcing context

**Sourcing-driven decisions:**
- RP2040-Zero LOCKED (Elite-Pi unavailable Indonesia)
- 1.27mm female pin header (vs Mill-Max — Aliexpress wait avoided)
- User confirmed SMD capability → JLCPCB bare PCB only, no PCBA

**Initial Tier 2 component decisions** (all revisable per Phase 1 findings).

---

## Phase 1 — Recon · COMPLETED 2026-05-13

All 5 deliverables in `docs/phase-1-recon/`:

| # | Deliverable | Status | Key output |
|---|---|---|---|
| 1 | [INVENTORY.md](docs/phase-1-recon/INVENTORY.md) | ✅ | File census + **LICENSE = GPL-3.0** (not CC-BY-NC-SA) + fork stale ~20 bulan |
| 2 | [MATRIX_VALIDATION.md](docs/phase-1-recon/MATRIX_VALIDATION.md) | ✅ | 8×6 matrix; **GP20-23 not exposed on RP2040-Zero** (4 col pins must remap); OLED is 128×64 (fork) not 128×32 |
| 3 | [FOOTPRINT_INVENTORY.md](docs/phase-1-recon/FOOTPRINT_INVENTORY.md) | ✅ | KLOR.pretty 17 footprints custom (no marbastlib ref); `SK6812MINI_and_cherry_1` = south-facing ready (mod #3 = verify only) |
| 4 | [PIN_MAPPING_PROPOSAL.md](docs/phase-1-recon/PIN_MAPPING_PROPOSAL.md) | ✅ | Final 17/20 GPIO allocation; col remap GP22/20/23/21 → GP13/12/11/10 |
| 5 | [REFERENCE_ANALYSIS.md](docs/phase-1-recon/REFERENCE_ANALYSIS.md) | ✅ | Justin Lam firmware critical config; 15-item Phase 2 risk register |

### Major findings affecting PROJECT_SPEC.md (corrections applied this date)

1. **R1 clarification:** 42-key polydactyl = 21 keys + 1 knob per side (encoder/knob occupies matrix [3,5]/[7,5] in `LAYOUT_polydactyl`). User confirmed via "21 keys per side tidak termasuk knob"
2. **OLED:** firmware uses **128×64** (not 128×32) — `config.h:74` defines `OLED_DISPLAY_128X64` + footprint outline ~27×27mm. Accept fork's 128×64; revised spec.
3. **Inter-half serial:** **half-duplex GP1 active** (not full-duplex as initial assumption from CHANGELOG documentation). Route both GP1+GP4 pada PCB future-proof.
4. **GP20-23 NOT exposed on RP2040-Zero** → 4 col pins remap to GP10-13.
5. **MOD #3 reduces to "verify only"** — fork's `SK6812MINI_and_cherry_1` footprint already south-facing (dual-layer pad + Edge.Cuts cutout).
6. **Bare RP2040-Zero firmware config gotchas** (Justin Lam blog):
   - `keyboard.json`: `processor: RP2040, bootloader: rp2040` instead of `development_board: elite_pi`
   - `config.h`: add `SERIAL_PIO_USE_PIO1` (prevents PIO conflict with WS2812), `RP2040_BOOTLOADER_DOUBLE_TAP_RESET`, `PICO_XOSC_STARTUP_DELAY_MULTIPLIER 64`
   - `rules.mk`: `SERIAL_DRIVER = vendor`
7. **PCB housekeeping** (Phase 2 schematic): drop placements for Pimoroni Haptic, Mallory buzzer, MSK12C02 slide switch, JST-PH, custombatteryconnector, lvc27_onlyInput (74LVC2G17 buffer)
8. **Fork delta vs upstream:** 152 commits 100% Lefuneste83 work (118+34 alt identity). klor1_4 is Lefuneste's primary contribution. Rev 1.5 announced 2024-09-10 tapi PCB files NEVER committed → stay di klor1_4.

### PROJECT_SPEC.md corrections applied (this date)

- Tier 1 R1 wording updated dengan "21 keys + 1 knob per side"
- Tier 2 OLED 128×32 → **128×64**
- Tier 2 serial wire allocation → **half-duplex active GP1; GP4 routed reserve**
- Tier 2 matrix_pins → new col pin mapping `[GP27, GP26, GP13, GP12, GP11, GP10]`
- Tier 2 firmware config block added (Justin Lam settings — `keyboard.json` processor/bootloader, config.h defines, rules.mk SERIAL_DRIVER)
- MOD #3 status → "verify only"
- DROPPED list expanded (lvc27, battery items explicit)
- PCB modification scope clarified per Phase 1 risks

---

## Phase 2 — Schematic Mod · in progress (pending kickoff)

### Phase 2 prep tasks (before opening klor1_4 in KiCad 10)

1. **Install marbastlib via KiCad 10 PCM**
   - Document version installed
   - Audit RP2040-Zero footprint coverage
   - Audit USB-C 16-pin midmount footprint coverage
   - Fallback plan jika gaps: custom-build dari Waveshare datasheet + community footprints

2. **Create git branch** `kicad-10-migration` di `hardware/`
   - First KiCad 10 open will auto-migrate file format (irreversible)
   - Branch retains pre-migration state for rollback

3. **Open `klor1_4.kicad_pro` di KiCad 10 untuk first migration**
   - Verify migration dialog success
   - Run DRC immediately (catch KiCad 10 stricter rule violations)
   - Visual inspect schematic + PCB editor
   - Commit migration result

### Phase 2 deliverables — in progress

**Completed (branch `kicad-10-migration`):**
- ✅ KiCad 10 migration (commit `75b16dd`)
- ✅ Custom RP2040-Zero footprint added to KLOR.pretty (commit `4029b11`)
- ✅ RP2040-Zero symbol added to KLORlib (commit `f98d576`)
- ✅ Schematic MOD #1: ProMicro → RP2040-Zero, drop battery section, ERC clean (commit `ee98b40`)
- ✅ Schematic MOD #2: TRRS → USB-C 16-pin midmount + 5.1kΩ CC pulldowns + PWR_FLAG on 5V, ERC clean (commit `fa041cc`)

**Pending Phase 2:**
- ⏳ MOD #3 verify: confirm schematic instantiates `SK6812MINI_and_cherry_1` (not older variant) for all 42 LED positions
- ⏳ Drop remaining placements if still in schematic: Pimoroni Haptic DRV2605L, Mallory buzzer, `lvc27_onlyInput` 74LVC2G17 buffer
- ⏳ Annotate symbols (Tools → Annotate Schematic) — assign final refdes
- ⏳ Update PCB from schematic (F8 in PCB editor / Tools → Update PCB from Schematic)
- ⏳ Verify PCB DRC clean

**Spec updates APPLIED 2026-05-14** (doc-sync session — committed Phase 2 schematic decisions ke spec docs):
- ✅ Matrix col pins: revised from `[GP27, GP26, GP13, GP12, GP11, GP10]` (descending) to `[GP27, GP26, GP9, GP10, GP11, GP12]` (ascending — user-natural label order, accepted). Applied di [PROJECT_SPEC.md](PROJECT_SPEC.md) Matrix section + MOD #1 description, dan [PIN_MAPPING_PROPOSAL.md](docs/phase-1-recon/PIN_MAPPING_PROPOSAL.md) Sections C/D/E/J.
- ✅ col2 → GP9, col3 → GP10, col4 → GP11, col5 → GP12 (was GP13-GP10 in original spec)
- ✅ RESET button → GP15 (soft reset via firmware; user wants accessible reset, GP15 was spare). Documented di PROJECT_SPEC Matrix section + PIN_MAPPING Section C row.
- ✅ 5V net uses `+5V` power port + `PWR_FLAG` on 5V net (USB-C VBUS is passive type, needs PWR_FLAG to satisfy ERC). Documented sebagai new row "VBUS / 5V net" di PROJECT_SPEC Inter-half connector table.
- ✅ Spares now: GP13, GP14 (GP15 used for RESET, GP9 used for col2). PIN_MAPPING Section C total updated: 18 used / 2 spare (was 17 used / 3 spare).
- ⏳ **keyboard.json deferred ke Phase 6** — akan diupdate bareng Justin Lam config block (`processor`/`bootloader`/SERIAL_PIO_USE_PIO1/etc.)

---

### Phase 2 doc-sync session · 2026-05-14

Sebelum lanjut MOD #3 verify, sync spec docs ke actual schematic state (commits `ee98b40` + `fa041cc`) untuk avoid stale baseline going into Phase 3 PCB layout.

**Done:**
- ✅ ERC verified clean — [ERC.rpt](hardware/PCB/klor1_4/erc/ERC.rpt) timestamp `2026-05-14T18:32:30`: **0 errors, 0 warnings**
- ✅ PROJECT_SPEC.md sync (4 edits): header date + status, Inter-half VBUS row, Matrix section ascending + RESET note + spares table, MOD #1 pin remap text
- ✅ PIN_MAPPING_PROPOSAL.md sync (6 edits): Section C table, total usage, Section D JSON + logic, Section E compatibility table, Section J summary, Section I open items (3 resolved + 1 new)

**PWR_FLAG semantics clarification (user concern resolved):**
- User question: "MCU 5V ada PWR_FLAG, USB-C 5V tidak ada — aman?"
- Answer: **AMAN**. `power:+5V` symbol creates global label → SEMUA `+5V` symbols (di MCU dan di USB-C) auto-connected sebagai net `+5V` tunggal oleh KiCad netlister. PWR_FLAG bersifat net-level (bukan node-level); satu instance pada net `+5V` (`#FLG02` near MCU) satisfies ERC untuk seluruh net. PWR_FLAG tidak ada efek fisik (bukan footprint/pad/copper).
- Bukti: ERC 0 errors. Topology verified via symbol position mapping di `klor1_4.kicad_sch`: MCU U1 `(at 49.53, 36.83)`, USB-C J1 `(at 119.38, 30.48)`, `+5V` #PWR03 dekat MCU, `+5V` #PWR04 dekat USB-C, `PWR_FLAG` #FLG02 dekat MCU `+5V`.

**Pending Phase 2 (unchanged from previous session):**
- ⏳ MOD #3 verify (LED footprint `_1` variant for all 42 positions)
- ⏳ Drop residual placements (Haptic / buzzer / lvc27 — verify if still present)
- ⏳ Annotate schematic (final refdes)
- ⏳ Update PCB from Schematic (F8)
- ⏳ PCB DRC clean

### Phase 2 — MOD #3 + Housekeeping · 2026-05-14 (commit `17c1fb3`)

**MOD #3 LED south-facing verify** — ✅ done (no schematic change needed):
- 42/42 LED placement instances use `KLOR:SK6812MINI_and_cherry_1` (south-facing variant)
- 0 instances use older `Button_Switch_Keyboard:SK6812MINI_and_cherry` (only present as overridden library default)
- Footprint files both exist locally di `KLOR.pretty/`
- Verify pattern: pure file read on `klor1_4.kicad_sch` (no KiCad open required)

**Housekeeping (commit `17c1fb3`):**
- Dropped via KiCad GUI delete (user-side action):
  - BZ1 Mallory Buzzer (`Device:Buzzer` + `KLOR:Buzzer_Mallory_AST1109MLTRQ_reversible` footprint)
  - J2 Pimoroni Haptic Buzz DRV2605L (`Connector_Generic:Conn_01x05` + `KLOR:Pimoroni Haptic Buzz DRV2605L Driver` footprint)
  - J3 lvc27_onlyInput 74LVC2G17 buffer (`Connector_Generic:Conn_01x05` + `KLOR:lvc27_onlyInput` footprint)
  - **Trackball module section** — NEW DROPPED item discovered during cleanup. Was NOT in original DROPPED list. User explicitly decided to skip pointing-device feature scope. Updated PROJECT_SPEC.md + CONTRIBUTING.md DROPPED lists.
  - Cosmetic text labels: "HAPTIC FEEDBACK", "SPEAKER", "TRACKBALL MODULE"
- ERC re-verified clean: `0 errors, 0 warnings` per [ERC.rpt](hardware/PCB/klor1_4/erc/ERC.rpt) timestamp `2026-05-14T19:50:35`
- Critical components preserved (post-cleanup grep verification):
  - 42× `SW_PUSH-MX_W_LED` placement (= 42 keys × switch+LED composite symbol)
  - 1× RP2040-Zero, 1× USB-C 16-pin connector, 1× SSD1306 OLED, 1× Rotary Encoder
  - 22× `D_Small` matrix diode (pre-existing fork count, unchanged)

**Pending Phase 2 (updated):**
- ⏳ Annotate schematic (Tools → Annotate Schematic Symbols) — assign final refdes after housekeeping
- ⏳ Update PCB from Schematic (F8 in PCB Editor)
- ⏳ PCB DRC clean — likely flag orphan footprints from dropped components; Phase 3 PCB layout territory

---

## Phase 3 — PCB layout mod · pending
## Phase 4 — Gerbers + JLCPCB · pending
## Phase 5 — Assembly · pending
## Phase 6 — Firmware (Vial-QMK fork + RP2040-Zero config) · pending
