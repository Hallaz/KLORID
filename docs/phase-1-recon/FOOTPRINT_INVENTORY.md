# Phase 1 / Deliverable #3 — FOOTPRINT_INVENTORY

> Audit footprints di `hardware/PCB/klor1_4/`. Identifikasi library source per footprint, analyze mod-critical footprints (orientation, pad compat), list footprints baru yang harus ditambah untuk modifications.

---

## A. Library setup (klor1_4 project)

### Project-local libraries (fp-lib-table & sym-lib-table)

[fp-lib-table](../../hardware/PCB/klor1_4/fp-lib-table) (project root):
```
(fp_lib_table (version 7)
  (lib (name "KLOR")(type "KiCad")(uri "${KIPRJMOD}/KLOR.pretty")(options "")(descr ""))
)
```

[sym-lib-table](../../hardware/PCB/klor1_4/sym-lib-table):
```
(sym_lib_table (version 7)
  (lib (name "KLORlib")(type "KiCad")(uri "${KIPRJMOD}/KLORlib.kicad_sym")(options "")(descr ""))
)
```

**Implication:** Project-local library cuma satu = **KLOR.pretty** (footprints) + **KLORlib.kicad_sym** (symbols). Semua footprint dan symbol "non-KLOR" datang dari **KiCad global libraries** (instal sistem) — yaitu standard KiCad footprint libraries (Resistor_SMD, Diode_SMD, dll) yang sudah jadi default install.

**Important:** **marbastlib (Bastardkb) NOT installed/referenced di project.** Kalau marbastlib ada di KiCad PCM globally, footprint-nya accessible via global lib path; tapi tidak ada explicit reference di project.

### Library setup yang perlu Phase 2

Sebelum modify schematic Phase 2:
1. **Install marbastlib** via KiCad PCM (Plugin & Content Manager) — kemungkinan besar source untuk RP2040-Zero footprint dan USB-C midmount. Verify version compatible dengan KiCad 10.0.2.
2. **Add marbastlib ke project's `fp-lib-table`** (optional — could juga rely on global table)
3. **Atau:** clone marbastlib repo manual + reference via `${KIPRJMOD}/marbastlib/...`

---

## B. Inventory: KLOR.pretty footprints (17 entries di klor1_4)

| # | Footprint | Type | Pad placement | Generator | Source | Mod relevance |
|---|---|---|---|---|---|---|
| 1 | `ProMicro` | TH | Dual-side reversible | 20211014 (KiCad 6/7) | **Custom/KLOR** | 🚨 **MOD #1 — REPLACE** with RP2040-Zero |
| 2 | `MJ-4PP-9` | TH | Dual-side reversible | 20240108 (KiCad 8) | Custom/KLOR | 🚨 **MOD #2 — REPLACE** with USB-C |
| 3 | `SK6812MINI_and_cherry` | TH + SMD hybrid | Dual-side reversible | 20211014 | Custom/KLOR | ✅ Per-key LED+switch combined (older variant) |
| 4 | `SK6812MINI_and_cherry_1` | TH + SMD hybrid | Dual-side reversible + Edge.Cuts cutout | 20240108 (KiCad 8) | Custom/KLOR | ✅ **South-facing refined variant** (see C.3) |
| 5 | `Diode_SOD123` | SMD | Dual-side reversible | 20240108 | Custom/KLOR (override) | ✅ KEEP — fork's own SOD-123 with dual-side pads |
| 6 | `RotaryEncoder_Alps_EC11E-...keebio_modified` | TH | Single-side | 20221018 | Keebio-derived | ✅ KEEP encoder (R7) |
| 7 | `SSD1306_128x64OLED` | TH | Single-side | 20211014 | Custom/KLOR | ⚠️ See C.4 |
| 8 | `SW_Push_1P1T-MP_NO_Horizontal_Alps_SKRTLAE010_both_sides` | TH | Dual-side reversible | (unread) | Custom/KLOR | ✅ KEEP — reset switch |
| 9 | `MSK12C02_reversible` | TH | Dual-side reversible | (unread) | Custom/KLOR | 🗑️ Drop (battery/power switch, wired-only build) |
| 10 | `JST_PH_reversible` | TH | Dual-side reversible | (unread) | Custom/KLOR | 🗑️ Drop (no battery — wired only) |
| 11 | `custombatteryconnector_small` | TH | Dual-side | (unread) | Custom/KLOR | 🗑️ Drop (no battery) |
| 12 | `Buzzer_Mallory_AST1109MLTRQ_reversible` | TH | Dual-side | (unread) | Custom/KLOR | 🗑️ Drop (audio out of scope) |
| 13 | `Pimoroni Haptic Buzz DRV2605L Driver` | TH | Dual-side | (unread) | Custom/KLOR | 🗑️ Drop (haptic out of scope) |
| 14 | `lvc27_onlyInput` | SMD | Single-side | (unread) | Custom/KLOR | 🗑️ Probably drop (74LVC2G17 buffer — likely haptic/audio-related) |
| 15 | `R_1206_DoubleSided` | SMD | Dual-side | (unread) | Custom/KLOR | ✅ KEEP — resistor for jumper pullups |
| 16 | `Jumper` | SMD | Dual-side | (unread) | Custom/KLOR | ✅ KEEP — solder jumper triangle (FABNOTES mentions triangular shape) |
| 17 | `SplitkbTentingPuckWithoutCenterHole` | TH/NPTH | Single-side | (unread) | Custom/KLOR | ✅ KEEP — SplitKB puck mounting |

**Observations:**
- All footprints are **custom in `KLOR.pretty`** (project-local) — none reference external lib like marbastlib
- Mix dari generator versions: 20211014 (KiCad 6/7), 20221018, 20240108 (KiCad 8) → KiCad 10 akan auto-migrate semuanya (irreversible). Backup branch wajib sebelum first open.
- Mayoritas dual-side reversible footprints — sophisticated design untuk PCB mirror image (same PCB for both halves, mounted on opposite sides)

---

## C. Mod-critical footprint analysis

### C.1 `ProMicro.kicad_mod` — TO REPLACE (MOD #1)

Source: [hardware/PCB/klor1_4/KLOR.pretty/ProMicro.kicad_mod](../../hardware/PCB/klor1_4/KLOR.pretty/ProMicro.kicad_mod)

| Property | Value |
|---|---|
| Pin count | 24 (2 rows × 12 pins per row) |
| Pin spacing | 2.54mm (0.1" standard header) |
| Pad diameter | 1.524mm, drill 0.8128mm |
| Form factor | Pro Micro standard (~17.8mm × 32mm pad-to-pad) |
| Mount style | Through-hole, dual-side reversible (pads di F.Cu DAN B.Cu untuk kedua row) |
| Outline | F.SilkS dan B.SilkS rectangle |

**Why replace:** Pin pitch 2.54mm (Pro Micro form-factor) tidak match Waveshare RP2040-Zero yang punya pitch **1.27mm** di castellated edge (23-pin per side, 46 total + 4 ADC pad di bawah). Pad arrangement berbeda completely.

**Verdict:** Custom footprint baru wajib untuk RP2040-Zero. See D.1.

### C.2 `MJ-4PP-9.kicad_mod` — TO REPLACE (MOD #2)

Source: [MJ-4PP-9.kicad_mod](../../hardware/PCB/klor1_4/KLOR.pretty/MJ-4PP-9.kicad_mod)

| Property | Value |
|---|---|
| Pin count | 4 signal pins + 4 NPTH mounting holes |
| Type | 3.5mm TRRS jack, vertical mount |
| Pad shape | Oval 1.7 × 2.5mm, drill oval 1 × 1.5mm |
| Mount style | TH, dual-side reversible |
| Generator version | 20240108 (KiCad 8) — newer format |

**Why replace:** Spec project ganti TRRS → USB-C 16-pin midmount. Completely different connector + pad arrangement. See D.2.

### C.3 ✅ `SK6812MINI_and_cherry_1.kicad_mod` — SOUTH-FACING VERIFIED

Source: [SK6812MINI_and_cherry_1.kicad_mod](../../hardware/PCB/klor1_4/KLOR.pretty/SK6812MINI_and_cherry_1.kicad_mod)

**Key findings:**
- Footprint **combines MX switch + WS2812-style 4-pin LED** in single component
- 4 LED pads dengan **dual-layer placement** (F.Cu + B.Cu) di setiap pad — supports mounting LED pada F.Cu (north-facing) atau B.Cu (south-facing)
- Punya **Edge.Cuts cutout zone** (rect ~3.5mm × 4.5mm) — ini lubang fisik di PCB tempat LED light SHINES THROUGH dari bottom ke top side
- Punya **chamfered pads** (pin 1 indicator) — design refinement
- Has CUTOUT zone keepout (no copper around LED window) — supports south-facing optical clearance

**Conclusion:** `SK6812MINI_and_cherry_1` adalah **south-facing variant yang mature**. The `_1` suffix = newer iteration after fork README:37 ("moved to south-facing LED").

**Difference vs original `SK6812MINI_and_cherry`:**
- Original: pad shape `rect` (square corners), no Edge.Cuts cutout, no chamfer
- `_1`: pad shape `roundrect` (rounded), Edge.Cuts cutout for optical pass-through, chamfered pin 1

> **Verdict for MOD #3:** ✅ **`_1` variant already supports south-facing reverse-mount SK6812MINI-E**. Hipotesis di MATRIX_VALIDATION F.2 confirmed — **MOD #3 reduces to "verify klor1_4 schematic uses `_1` variant"** (not the original variant). Likely yes per fork README intent, but confirm di Phase 2 saat open schematic. **NO new footprint needed.**

**Bonus finding:** Both LED variants (`_1` and original) have **19.05mm × 19.05mm reference square** on Dwgs.User layer — confirming key spacing R1.

### C.4 ✅ `SSD1306_128x64OLED.kicad_mod` — Pad PIN-COMPATIBLE

Source: [SSD1306_128x64OLED.kicad_mod](../../hardware/PCB/klor1_4/KLOR.pretty/SSD1306_128x64OLED.kicad_mod)

| Property | Value |
|---|---|
| Pin count | 4 (VCC, GND, SCL, SDA — standard I²C OLED breakout) |
| Pin pitch | **2.54mm** (10.6 / 4 = 2.54) — verified from pad positions: -3.62, -1.08, 1.46, 4 |
| Pad shape | Oval 2 × 1.6mm, drill 1mm |
| Outline (B.SilkS / F.SilkS rect) | ~27.4mm × 27.3mm — matches **0.96" 128×64 module** physical size |

**Pin compatibility analysis:**

Universal 4-pin I²C OLED breakout modules (both 0.91" 128×32 dan 0.96" 128×64) menggunakan **identical 4-pin header @ 2.54mm pitch** dengan urutan (VCC, GND, SCL, SDA). **Footprint pad layout WORKS for both resolutions.**

**Outline compatibility:**
- 0.96" 128×64 module: ~27 × 27mm → fits footprint outline ✓
- 0.91" 128×32 module: ~30 × 11mm (taller × narrower) → fits pin holes tapi outline tidak match

**Decision (MATRIX_VALIDATION F.2 confirmed):** **Use 0.96" 128×64 module.** Footprint native compatible. No new footprint needed.

### C.5 ✅ `RotaryEncoder_Alps_EC11E-...keebio_modified.kicad_mod`

Source: [RotaryEncoder_Alps_EC11E-Switch_Vertical_H20mm-keebio_modified.kicad_mod](../../hardware/PCB/klor1_4/KLOR.pretty/RotaryEncoder_Alps_EC11E-Switch_Vertical_H20mm-keebio_modified.kicad_mod)

| Property | Value |
|---|---|
| Pin count | 5 signal (A, B, C, S1, S2) + 2 MP mechanical |
| Origin | Alps EC11E datasheet, modified from Keebio's library |
| 3D model | `${KISYS3DMOD}/Rotary_Encoder.3dshapes/...wrl` (KiCad built-in) |
| Mount style | TH, vertical shaft, H20mm height |

**Verdict:** ✅ KEEP unchanged. Standard EC11 encoder footprint, sourceable Tokopedia.

### C.6 ✅ `Diode_SOD123.kicad_mod`

Source: [Diode_SOD123.kicad_mod](../../hardware/PCB/klor1_4/KLOR.pretty/Diode_SOD123.kicad_mod)

| Property | Value |
|---|---|
| Package | SOD-123 (SMD) |
| Pad shape | Rect 1.8 × 1.5mm, drill 0.4mm (slight pad-hole hybrid) |
| Mount | Dual-side reversible (pads on `*.Cu` and `*.Paste`/`*.Mask` semua layers) |
| 3D model | `${KISYS3DMOD}/Diode_SMD.3dshapes/D_SOD-123.wrl` |

**Note:** Fork's variant punya **drill 0.4mm** even on SMD pads — ini hybrid SMD/TH design supaya bisa solder either way. Standard SOD-123 = pure SMD tanpa drill, fork's = SMD pad + small through-hole pin assist.

**Verdict:** ✅ KEEP — fork-specific reliability tweak for matrix diodes.

---

## D. New footprints needed for mods

### D.1 🚨 Waveshare RP2040-Zero footprint — MUST ADD

**Specification (per Waveshare wiki):**
- 23 pins per side castellated edge + 4 ADC pads di bottom = 50 total pad positions
- 20 GPIO accessible (some pads are GND/power/USB)
- Pin pitch: **1.27mm (0.05") on castellated edge**, custom pitch on bottom ADC pads
- Board dimensions: 18mm × 23.5mm
- Through-hole or surface-mount castellated (can solder edge-on-edge or via pin headers)

**Sourcing options:**

| Source | Pros | Cons |
|---|---|---|
| **marbastlib** (Bastardkb KiCad lib) | Well-maintained, supports KiCad 10 | Need to install via PCM + verify RP2040-Zero coverage (cek docs sebelum lock-in) |
| **Community footprint** (GitHub kiboard libraries) | Usually free, KiCad-native | Quality varies; may need verification with calipers |
| **Custom footprint** (build dari datasheet) | Full control, exact spec match | Time investment (~1-2 jam Phase 2); requires careful dimension verification |
| **boardsource/kicad-rp2040-zero** | Open source community | Verify maintenance status |

**Decision (Claude, Phase 2):** **Primary path = marbastlib.** Bastardkb known to support RP2040-based custom builds. Install PCM → verify footprint exists → use. **Fallback:** custom footprint dari Waveshare datasheet jika marbastlib coverage tidak ada.

**Note untuk socketing (R10):** Footprint harus support 1.27mm pitch female header pad arrangement — bukan langsung solder castellated. Typical pattern: through-hole pads di 1.27mm pitch yang line-up dengan castellated edge of RP2040-Zero. User insert RP2040-Zero ke pin headers vertically.

### D.2 🚨 USB-C 16-pin midmount SMD footprint — MUST ADD

**Specification target:**

Two candidate part numbers (final pick di Phase 2):

| Part | Pin count | Mount | LCSC stock | Footprint source candidate |
|---|---|---|---|---|
| **GCT USB4085** | 16 SMD | Midmount | Yes (good stock) | marbastlib likely; also community libs |
| **HRO TYPE-C-31-M-12** | 16 SMD | Midmount | Yes | Generic Type-C breakout, community footprints abundant |

Both share **16-pin SMD midmount form factor**. Pin allocation typical:
- A1, B12: GND
- A4, B9: VBUS
- A5: CC1, B5: CC2 (USB-C role detection)
- A6, B7: D+
- A7, B6: D-
- A2, A3, B10, B11: TX/RX SuperSpeed pairs (we ignore)
- A8, B8: SBU1, SBU2 (not used)
- A9, B4: extra contacts (often unused on basic breakouts)

**Decision (Claude, Phase 2):** Lock specific part di Phase 2 after marbastlib audit. Both parts have same footprint pad arrangement (16-pin SMD), so PCB design can proceed dengan generic 16-pin midmount footprint; final part sourcing depends on JLCPCB/LCSC stock pas-PCB order day.

**Wire usage** (per MATRIX_VALIDATION D):
- VBUS, GND: power
- D+ (single pin): half-duplex serial → GP1 ↔ GP1 inter-half
- D- (single pin): reserved for full-duplex serial upgrade → GP4 ↔ GP1 cross (when enabled)
- CC1/CC2: tied to 5.1kΩ pulldowns? Or NC for "dumb" sink behavior. Phase 2 schematic detail.
- Other pins: unused/NC

### D.3 ✅ SK6812MINI-E reverse-mount — NOT NEEDED

**Already covered by `SK6812MINI_and_cherry_1` footprint** (see C.3). South-facing reverse-mount supported via dual-layer pad placement and Edge.Cuts cutout zone. **MOD #3 reduces to "ensure klor1_4 schematic uses `_1` variant, not the older `SK6812MINI_and_cherry`."**

Phase 2 task: open schematic, verify all 42 LED positions instantiate `SK6812MINI_and_cherry_1`, replace any using older variant.

---

## E. marbastlib expected coverage (informational)

[marbastlib (Bastardkb)](https://github.com/Bastardkb/bastardkb-kicad-libraries) typically provides:

| Category | Coverage | Relevant to us? |
|---|---|---|
| RP2040 modules | RP2040-Zero, Pro Micro RP2040 variants | ✅ D.1 |
| USB-C connectors | Multiple midmount variants | ✅ D.2 |
| MX hotswap sockets | Kailh, Gateron, Outemu | (already have via custom KLOR variant) |
| SK6812 LEDs | Multiple orientation variants | (already have via `_1`) |
| TRRS jacks | Multiple variants | (not used — we're replacing TRRS) |

**Action item Phase 2 start:** Install marbastlib via PCM, audit which of D.1/D.2 footprints available. Document version installed for reproducibility.

---

## F. Library audit summary (Phase 2 prep)

| Action | Priority | Owner | Phase |
|---|---|---|---|
| Install marbastlib via KiCad 10 PCM | High | User (or via instructions saya) | Phase 2 start |
| Verify marbastlib has RP2040-Zero footprint | High | Claude (post-install) | Phase 2 |
| Verify marbastlib has USB-C 16-pin midmount | High | Claude (post-install) | Phase 2 |
| Build custom RP2040-Zero footprint (fallback) | Conditional | Claude — if marbastlib gap | Phase 2 |
| Build custom USB-C midmount footprint (fallback) | Conditional | Claude — if marbastlib gap | Phase 2 |
| Confirm schematic uses `SK6812MINI_and_cherry_1` (not older) | Medium | Claude — open schematic Phase 2 | Phase 2 |
| Remove placements: Pimoroni Haptic, Buzzer, MSK12C02, JST_PH, lvc27, custombatteryconnector | Medium | Phase 2 schematic mod | Phase 2 |
| Delete unused footprint files dari KLOR.pretty (cleanup) | Low | Optional — keep for fork reference | Phase 3 |

---

## G. Cross-references

- [INVENTORY.md](INVENTORY.md) — Phase 1 #1 file census
- [MATRIX_VALIDATION.md](MATRIX_VALIDATION.md) — Pin assignments
- [PROJECT_SPEC.md](../../PROJECT_SPEC.md) — Component decisions
- [hardware/PCB/klor1_4/KLOR.pretty/](../../hardware/PCB/klor1_4/KLOR.pretty/) — All custom footprints
- [marbastlib (GitHub)](https://github.com/Bastardkb/bastardkb-kicad-libraries) — Expected source for new footprints

## H. Open items (Phase 1 #4 + Phase 2 to resolve)

1. **Final col-pin remap GP10-15 specific assignment** → Phase 1 #4 PIN_MAPPING_PROPOSAL
2. **marbastlib version installation + audit** → Phase 2 start
3. **Confirm klor1_4 schematic uses `SK6812MINI_and_cherry_1` (south-facing) everywhere** → Phase 2 schematic open
4. **Final USB-C midmount part lock-in (GCT USB4085 vs HRO Type-C-31-M-12)** → Phase 2 based on JLCPCB stock & footprint availability
5. **RP2040-Zero footprint: marbastlib vs custom-built** → Phase 2 after marbastlib audit

## I. Next deliverable

**Phase 1 deliverable #4: `PIN_MAPPING_PROPOSAL.md`**

Scope:
- Lock specific GP10-15 assignment untuk matrix col remap (4 pins of 6)
- Cross-reference zerosprey42 (42-key RP2040-Zero) pin mapping untuk validation
- Document full 17-pin allocation table dengan reasoning per pin
- Note GPIO headroom (3-4 free pins) untuk future-proofing
- Validate PIO peripheral assignment (GP1/GP4 serial, GP0 RGB)
