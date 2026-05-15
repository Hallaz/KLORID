# Phase 1 / Deliverable #5 — REFERENCE_ANALYSIS

> Final Phase 1 deliverable. Synthesize: zerosprey42 schematic insights, Justin Lam handwired Skeletyl blog (RP2040-Zero firmware config), Lefuneste83 fork delta vs upstream, KiCad 10 migration risks, dan Phase 2 risk register.

---

## A. zerosprey42 cross-validation

[zerosprey42](../../references/zerosprey42/) = beekeeb's 42-key unibody RP2040-Zero keyboard. Direct exemplar untuk our MCU choice tapi topologi berbeda (monoblock vs split).

### A.1 RP2040-Zero pin → physical pad mapping (from schematic symbol)

Extracted dari [zerosprey42.kicad_sch](../../references/zerosprey42/pcb/zerosprey42.kicad_sch) Lines 396-484 — RP2040-Zero symbol pin definitions:

```
Side A (castellated edge row 1):                Side B (castellated edge row 2):
  pin 23 → GP0    (top)                            pin 1  → 5V/VBUS
  pin 22 → GP1                                     pin 2  → GND
  pin 21 → GP2                                     pin 3  → 3V3
  pin 20 → GP3                                     pin 4  → GP29
  pin 19 → GP4                                     pin 5  → GP28
  pin 18 → GP5                                     pin 6  → GP27
  pin 17 → GP6                                     pin 7  → GP26
  pin 16 → GP7                                     pin 8  → GP15
  pin 15 → GP8                                     pin 9  → GP14
  pin 14 → GP9    (bottom)
  pin 13 → GP10
  pin 12 → GP11
  pin 11 → GP12
  pin 10 → GP13
```

**Implications untuk Phase 3 PCB footprint:**
- RP2040-Zero punya 2 castellated edge rows + bottom ADC pad row
- Side A (14 pads, GP0-GP13) + Side B (top 3 = USB; 4-9 = GP14, GP15, GP26-29 = 6 pads)
- **Total accessible: 16 GPIO castellated + 4 ADC = 20 GPIO** ✓ matches earlier audit
- Pin number ↔ GPxx mapping = reference saat draw custom footprint dengan pin label assignment

### A.2 zerosprey42 vs our proposal — pin allocation comparison

zerosprey42 detailed pin allocation TIDAK extracted di Grep (schematic pakai wire connections tanpa explicit GPxx net labels — would need open di KiCad 10 untuk trace). 

Tapi dari README + screenshot:
- 42 keys monoblock unibody (single PCB, single MCU)
- Per-key diode + hotswap (matrix-scanned, COL2ROW)
- No inter-half serial (unibody → no split transport)
- Choc-spaced (not 19.05mm)

**Difference from our build:**
- Our build = **split** (2 halves, 2 MCUs, inter-half USB-C serial)
- Our pin priorities differ (need serial GP1+GP4, plus I²C OLED GP2/3)
- zerosprey42's wider pin freedom (no inter-half overhead) makes pin allocation more flexible

**Validation outcome:** zerosprey42 confirms RP2040-Zero works for 42-key keyboards dengan QMK+Vial firmware. Specific pin allocation TIDAK directly cross-reference-able, tapi feasibility validated.

> **Phase 2 follow-up:** Open `zerosprey42.kicad_sch` di KiCad 10 untuk extract exact pin nets (matrix col/row to GPxx wire connections). Optional — saat ini proposal kita already sound based on RP2040-Zero edge availability.

---

## B. Justin Lam handwired Skeletyl blog — RP2040-Zero firmware lessons

Source: https://www.justinmklam.com/posts/2025/08/handwired-skeletyl/

### B.1 Critical firmware config untuk bare RP2040-Zero

**`keyboard.json` essentials:**
```json
"bootloader": "rp2040",
"processor": "RP2040",
```

🚨 **Key gotcha:** Untuk **bare Waveshare RP2040-Zero**, jangan pakai `"development_board": "elite_pi"` (yang dipakai fork saat ini). Pakai explicit `processor` + `bootloader` instead. Author Justin Lam emphasize: "the bootloader and processor needed to just be RP2040 instead of the dev board."

> **Implication for fork:** Current fork's `keyboard.json:16` `"development_board": "elite_pi"` akan **break** untuk our RP2040-Zero target. Need switch ke `"bootloader": "rp2040"` + `"processor": "RP2040"`. Phase 6 firmware update task.

**`config.h` RP2040-Zero specific:**
```c
#define RP2040_BOOTLOADER_DOUBLE_TAP_RESET
#define RP2040_BOOTLOADER_DOUBLE_TAP_RESET_TIMEOUT 500U
#define PICO_XOSC_STARTUP_DELAY_MULTIPLIER 64
#define SERIAL_PIO_USE_PIO1
```

Explained per setting:
- **`RP2040_BOOTLOADER_DOUBLE_TAP_RESET`** → enable double-tap-reset-button to enter bootloader. Better UX vs holding BOOT pin saat power on (RP2040-Zero punya BOOT button onboard).
- **`RP2040_BOOTLOADER_DOUBLE_TAP_RESET_TIMEOUT 500U`** → 500ms window between two reset taps. Industry-standard timing.
- **`PICO_XOSC_STARTUP_DELAY_MULTIPLIER 64`** → extends crystal oscillator stabilization delay. Important untuk reliable boot — bare RP2040-Zero kadang ada XOSC startup issue, multiplier 64 = safer.
- **`SERIAL_PIO_USE_PIO1`** → explicitly use **PIO1** state machine block untuk inter-half serial. **Critical:** WS2812 RGB di GP0 already pakai PIO0 by default. Without PIO1 explicit, serial mungkin conflict dengan RGB → garbled comms.

**`rules.mk` RP2040-Zero specific:**
```make
SERIAL_DRIVER = vendor
```

`vendor` driver = QMK's RP2040-specific serial driver (PIO-based). Pairs with `SERIAL_PIO_USE_PIO1` define.

### B.2 Justin's pin allocation (reference, not adopted directly)

Justin's example untuk handwired Skeletyl:
- Columns: GP4-GP8 (5 cols)
- Rows: GP9-GP12 (4 rows)
- Soft serial: GP1
- Diode: COL2ROW

**Difference vs our proposal:** Different MCU pin layout (Justin's handwired Skeletyl = different topology). Pin choice tidak directly reusable, tapi confirms:
- GP1 = standard pick untuk serial (✓ matches our PIN_MAPPING_PROPOSAL)
- GP4-15 range = safe for matrix scanning (✓ matches our col remap to GP10-13)
- COL2ROW direction = standard (✓ matches fork's keyboard.json:17)

---

## C. Lefuneste83 fork delta vs upstream GEIGEIGEIST

### C.1 Quantitative summary

| Metric | Value |
|---|---|
| Fork commits ahead of upstream | **152** |
| Files changed | **248** |
| Author breakdown (fork-only) | Lefuneste83 (118) + Hilarion Lefuneste (34) = **152 commits 100% by fork maintainer** |

Top changed directories (fork delta):

| Files changed | Directory | Significance |
|---|---|---|
| 51 | `FIRMWARE/zmk` | ZMK config for Nice Nano BLE variants (Lefuneste maintains separate `zmk-config-klor` repo too) |
| 42 | `case/Alternate Case Design` | Community-contributed v2/v2.1 gasket designs (committed via merge into fork) |
| 41 | `PCB/klor1_4` | **klor1_4 = Lefuneste's primary contribution** — refactored PCB design, south-facing SK6812MINI-E |
| 41 | `PCB/klor1_4_LP_KS33` | Low-profile Gateron KS33 beta — Lefuneste's experimental variant |
| 21 | `PCB/klor1_3` | Legacy fixes (DRC cleanup for JLCPCB validation) |
| 19 | `docs/images` | Build guide images, Rev 1.5 renders (PCB files belum committed) |
| 19 | `FIRMWARE/qmk_rp2040` | QMK RP2040 modernization (elite_pi config, Vial keymap, full-duplex serial doc) |
| 8 | `FIRMWARE/Ready-to-flash` | Pre-built UF2 binaries (will become stale post-Phase 6) |

### C.2 Qualitative fork contributions (per CHANGELOG + commit messages)

**Lefuneste83's core contributions:**

1. **PCB Rev 1.4** — completely refactored layout dari Rev 1.3:
   - South-facing SK6812MINI-E (vs upstream's WS2812 3535 north-facing)
   - Re-routed 90% of traces, cleaned DRC errors
   - Triangular solder jumpers (vs upstream rectangular)
   - Dual ground planes
   - Logo change to Lefuneste's branding
   - 0 JLCPCB DRC errors

2. **PCB Rev 1.4 LP KS33** — low-profile beta dengan Gateron KS33 hotswap.

3. **QMK RP2040 modernization** — elite_pi development_board config, full Vial-QMK support, EE_HANDS firmware-based handedness, half-duplex serial transport documented.

4. **Rev 1.5 announcement (Sept 2024)** — major improvements (direct USB 5V, full duplex, hardware handedness, OLED voltage select) **TAPI PCB files tidak committed di repo**. Hanya renders + CHANGELOG description. Inactive since Sept 2024.

5. **DIY Haptic Module alternative** — DRV2605L DIY breakout untuk cost-saving vs Pimoroni's $15 module. **Dropped from our scope** per haptic decision.

### C.3 Anything to merge back (upstream PR potential)?

Tidak relevant untuk our project (we fork from Lefuneste83 directly), tapi noting:
- Lefuneste's klor1_4 mature enough untuk dianggap upstream-mergeable, tapi GEIGEIGEIST upstream stagnant (last commit 2024 buildguide_acrylic.md update only)
- Lefuneste83 maintains independent fork ecosystem (zmk-config-klor + klor-config + community contributions)

### C.4 Unique-to-fork vs upstream-shared

| Feature | Upstream | Lefuneste's fork |
|---|---|---|
| PCB Rev 1.3 | Original | Inherited + DRC fixes |
| PCB Rev 1.4 | ❌ Not present | ✅ Lefuneste's main work |
| PCB Rev 1.4 LP KS33 | ❌ Not present | ✅ Beta low-profile |
| PCB Rev 1.5 | ❌ Not present (renders only) | ⚠️ Announced, PCB files NOT committed |
| QMK RP2040 elite_pi config | ❌ AVR Pro Micro era | ✅ Lefuneste modernized |
| Vial firmware support | ❌ QMK only | ✅ Lefuneste added |
| ZMK NiceNano support | ⚠️ Initial | ✅ Lefuneste enhanced (separate repo) |
| Alternate community cases | ❌ | ✅ Merged community PRs |
| SK6812MINI-E reverse-mount (`_1` variant) | ❌ WS2812 3535 era | ✅ Lefuneste klor1_4 |

> **For us:** Starting from Lefuneste83's fork = right choice. Inherits everything modernized, RP2040-ready, south-facing LED, Vial support. We just need to swap MCU footprint + USB-C connector.

---

## D. KiCad 10 migration risk register

### D.1 Fork's KiCad file format versions

| File | Generator version | Likely KiCad era |
|---|---|---|
| `klor1_4.kicad_pro` | version 1 (`meta.version`) | KiCad 7/8 |
| `ProMicro.kicad_mod` | 20211014 | KiCad 6/7 |
| `MJ-4PP-9.kicad_mod` | 20240108 (generator 8.0) | KiCad 8 |
| `Diode_SOD123.kicad_mod` | 20240108 | KiCad 8 |
| `RotaryEncoder_..keebio_modified.kicad_mod` | 20221018 | KiCad 7 |
| `SSD1306_128x64OLED.kicad_mod` | 20211014 | KiCad 6/7 |
| `SK6812MINI_and_cherry.kicad_mod` | 20211014 | KiCad 6/7 |
| `SK6812MINI_and_cherry_1.kicad_mod` | 20240108 | KiCad 8 |

Fork = **mixed KiCad 6/7/8 format**. KiCad 10 will auto-migrate semua to v10 format.

### D.2 Migration risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| Auto-migration breaks footprint references | Low | Run KiCad 10 migration dialog di branch terpisah; verify open visually before commit |
| Schematic symbol-to-footprint mappings break | Low-medium | KiCad 10 typically handles backward compat well; tapi some KiCad 6 footprints with deprecated syntax mungkin warning |
| 3D model `${KISYS3DMOD}` path resolution | Medium | KiCad 10 deprecated `KISYS3DMOD` in favor of `KICAD8_3DMODEL_DIR` / `KICAD9_3DMODEL_DIR`. May cause "3D model not found" warnings (cosmetic, doesn't block design) |
| Net class definitions migration | Low | Should auto-migrate |
| DRC rule changes (KiCad 10 stricter) | Medium | Re-run DRC after open; existing 0-error status mungkin reveal new warnings. Fix if blocker |
| Library table format change | Low | `fp-lib-table` already version 7 = KiCad 7+ format, KiCad 10 compatible |
| Custom layer setups | Low | Standard 2-layer FR4, no exotic layers |

### D.3 Migration workflow (Phase 2 start)

**Mandatory steps before first KiCad 10 open:**

1. ✅ `cd hardware && git checkout -b kicad-10-migration` — create dedicated branch *(historical: ran from `base-fork/KLOR/` before Phase 2 close flatten)*
2. ✅ Commit current state if uncommitted (currently 0 dirty per Phase 0 verification)
3. ✅ Verify KiCad 10.0.2 installed (Anda verified earlier)
4. ✅ **Optional but recommended:** Install marbastlib via PCM **before** opening klor1_4 (so KiCad 10 already knows about new libraries during migration scan)
5. ⚠️ Open `klor1_4.kicad_pro` di KiCad 10 → dialog "Migrate files? (Irreversible)" → click Yes
6. ⚠️ KiCad 10 akan generate `_backup` files dari originals
7. ✅ Run DRC immediately to check no new errors (especially after KiCad 10's stricter rules)
8. ✅ Commit migrated files dengan message: "KiCad 10 auto-migration from KiCad 6/7/8 format"
9. ✅ Visual inspect: open schematic + PCB editor, verify nothing broken

### D.4 Rollback plan

Kalau migration breaks something irrecoverable:
- `git checkout main` — back to pre-migration state (KiCad 6/7/8 format intact)
- Either install older KiCad version (e.g., KiCad 8 LTS) OR fix issues manually post-migration
- Migration branch retains progress

---

## E. Phase 2 risk register (consolidated from all Phase 1 findings)

| # | Risk | Severity | Probability | Mitigation |
|---|---|---|---|---|
| 1 | RP2040-Zero footprint marbastlib coverage | High | Medium | Verify marbastlib has RP2040-Zero footprint at Phase 2 start. Fallback: custom-build from Waveshare datasheet (~2-3 jam) |
| 2 | USB-C 16-pin midmount footprint marbastlib coverage | High | Medium | Same fallback plan. GCT USB4085 + HRO TYPE-C-31-M-12 are common community footprints; should be obtainable |
| 3 | KiCad 10 auto-migration of fork's mixed-version files | Medium | High | Branch-based approach (D.3). Test before commit |
| 4 | Schematic uses OLDER `SK6812MINI_and_cherry` variant (without south-facing cutout) | Low-medium | Low | FOOTPRINT_INVENTORY C.3 identified `_1` variant is the correct south-facing. Verify at Phase 2 schematic open. Replace older variant if found |
| 5 | RP2040-Zero **GP20-23 not exposed** = 4 col pins must remap | High | Confirmed | PIN_MAPPING_PROPOSAL section C locks new col pins. Apply at Phase 6 firmware update |
| 6 | `keyboard.json` uses `"development_board": "elite_pi"` but bare RP2040-Zero requires `"processor": "RP2040", "bootloader": "rp2040"` | High | Confirmed (B.1 Justin Lam finding) | Update keyboard.json at Phase 6 firmware step |
| 7 | Inter-half PIO serial conflict with WS2812 (both use PIO by default) | Medium | Possible | Set `SERIAL_PIO_USE_PIO1` di config.h → separates serial (PIO1) from WS2812 (PIO0). Justin Lam confirmed pattern |
| 8 | RP2040-Zero XOSC startup unreliability | Low-medium | Low | Add `PICO_XOSC_STARTUP_DELAY_MULTIPLIER 64` to config.h per Justin Lam recommendation |
| 9 | Bootloader access: BOOT button on RP2040-Zero hard to access in case | Medium | Likely | Enable `RP2040_BOOTLOADER_DOUBLE_TAP_RESET` for double-tap-reset entry (no need to touch BOOT button after first flash) |
| 10 | GP0 RGB chain entry trace routing from new MCU position | Medium | Likely | Phase 3 PCB layout concern — trace from RP2040-Zero GP0 pad to LED 00 entry. Plan early |
| 11 | Mill-Max sockets sourcing fail-over | Low (already mitigated) | Low | PIN_MAPPING + FOOTPRINT decisions already specify 1.27mm pin header instead of Mill-Max |
| 12 | 1.27mm female pin header height adds ~3mm vs Mill-Max — case clearance | Low-medium | Possible | Verify klor1_4 case STL has sufficient MCU clearance (~6mm stack height). May need case mod (not in scope; user accepts trade-off per PROJECT_SPEC) |
| 13 | USB-C 16-pin midmount cutout pada PCB | Low-medium | Likely (per part chosen) | Most midmount USB-C requires PCB edge cutout (slot ~3-4mm deep × 9mm wide). Plan Phase 3 layout |
| 14 | Fork's CC pins handling on USB-C unclear | Low | Possible | Phase 2 schematic: tie CC1/CC2 to 5.1kΩ pulldown to GND for "dumb sink" behavior (standard practice for power-only USB-C connectors) |
| 15 | Drop placements correctness (haptic/audio/battery footprints removal) | Low | Likely | Phase 2 schematic mod: delete instantiated components, verify net cleanup, re-run ERC |

---

## F. PROJECT_SPEC.md update queue (batch at end of Phase 1)

Compiled corrections dari Phase 1 #1-#5, to be applied di single batch update:

### F.1 Tier 1 REQUIREMENTS clarifications
- **R1** clarify: "42-key split keyboard polydactyl layout = 21 keys per side + 1 encoder/knob position per side (total 42 keys + 2 encoders)" — explicit per user confirmation

### F.2 Tier 2 COMPONENT DECISIONS updates
- **OLED:** 128×32 → **128×64** (fork-native; firmware tuned for 128×64; tukar tidak worth it)
- **Inter-half wire allocation:** "full-duplex" → "**half-duplex active GP1; GP4 reserved for full-duplex upgrade (both pads routed pada PCB)**"
- **Matrix col pins remap** added: `GP27, GP26, GP13, GP12, GP11, GP10` (4 of 6 remapped untuk RP2040-Zero edge availability)
- **MOD #3 status:** scope reduces to "verify only" (klor1_4's `SK6812MINI_and_cherry_1` footprint already south-facing per FOOTPRINT_INVENTORY C.3)

### F.3 New firmware config additions (for Phase 6)
Per Justin Lam blog findings:
```c
// config.h additions (RP2040-Zero specific):
#define RP2040_BOOTLOADER_DOUBLE_TAP_RESET
#define RP2040_BOOTLOADER_DOUBLE_TAP_RESET_TIMEOUT 500U
#define PICO_XOSC_STARTUP_DELAY_MULTIPLIER 64
#define SERIAL_PIO_USE_PIO1
```

```make
# rules.mk:
SERIAL_DRIVER = vendor
```

```json
// keyboard.json changes:
// Remove: "development_board": "elite_pi"
// Add:    "processor": "RP2040", "bootloader": "rp2040"
// Update: "matrix_pins.cols": ["GP27","GP26","GP13","GP12","GP11","GP10"]
```

### F.4 PCB scope (housekeeping additions)
Beyond 3 main mods, drop placements (Phase 2 schematic):
- Pimoroni Haptic DRV2605L (haptic out)
- Mallory buzzer (audio out)
- MSK12C02 slide switch (battery out)
- JST-PH connector (battery out)
- `custombatteryconnector_small` (battery out)
- `lvc27_onlyInput` (74LVC2G17 buffer — likely haptic-related, verify and remove if so)

---

## G. Cross-references

- [INVENTORY.md](INVENTORY.md) — Phase 1 #1 file census
- [MATRIX_VALIDATION.md](MATRIX_VALIDATION.md) — Pin assignment audit + GPIO budget
- [FOOTPRINT_INVENTORY.md](FOOTPRINT_INVENTORY.md) — Library audit
- [PIN_MAPPING_PROPOSAL.md](PIN_MAPPING_PROPOSAL.md) — Final pin allocation + col remap
- [zerosprey42 README](../../references/zerosprey42/README.md) — RP2040-Zero exemplar
- [zerosprey42 schematic](../../references/zerosprey42/pcb/zerosprey42.kicad_sch) — RP2040-Zero symbol pin definitions (Phase 2 detailed extraction TBD)
- [Justin Lam blog](https://www.justinmklam.com/posts/2025/08/handwired-skeletyl/) — RP2040-Zero firmware config gotchas
- [PROJECT_SPEC.md](../../PROJECT_SPEC.md) — to be batch-updated dengan F.1-F.4
- [PHASE_LOG.md](../../PHASE_LOG.md) — to be appended dengan Phase 1 completion

---

## H. Phase 1 deliverable status

| # | Deliverable | Status | File |
|---|---|---|---|
| 1 | INVENTORY | ✅ Done | [docs/phase-1-recon/INVENTORY.md](INVENTORY.md) |
| 2 | MATRIX_VALIDATION | ✅ Done | [docs/phase-1-recon/MATRIX_VALIDATION.md](MATRIX_VALIDATION.md) |
| 3 | FOOTPRINT_INVENTORY | ✅ Done | [docs/phase-1-recon/FOOTPRINT_INVENTORY.md](FOOTPRINT_INVENTORY.md) |
| 4 | PIN_MAPPING_PROPOSAL | ✅ Done | [docs/phase-1-recon/PIN_MAPPING_PROPOSAL.md](PIN_MAPPING_PROPOSAL.md) |
| 5 | REFERENCE_ANALYSIS | ✅ **Done (this doc)** | [docs/phase-1-recon/REFERENCE_ANALYSIS.md](REFERENCE_ANALYSIS.md) |

**Phase 1 Recon COMPLETE.** Ready untuk transition ke Phase 2 Schematic Mod.

---

## I. Recommended next step

**Pending batch update:** [PROJECT_SPEC.md](../../PROJECT_SPEC.md), [CONTRIBUTING.md](../../CONTRIBUTING.md), [PHASE_LOG.md](../../PHASE_LOG.md) dengan F.1-F.4 corrections + Phase 1 completion status.

After batch update, Phase 2 prep:
1. Install marbastlib via KiCad PCM
2. Create `kicad-10-migration` branch di `hardware/`
3. Open `klor1_4.kicad_pro` for first KiCad 10 migration
4. Verify migration success + run DRC
5. Begin schematic mod (start dengan MCU footprint replacement)

Confirm "go" untuk batch update file root, atau ada catatan untuk REFERENCE_ANALYSIS.md dulu?
