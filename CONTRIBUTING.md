# CONTRIBUTING.md — Project Guidance

> Instructions for collaborators (human + AI) working in this KLORID workspace. Read first before making changes.

## Project mission

KLORID = custom 42-key split keyboard (polydactyl variant: 21 keys + 1 encoder per side), derivative of [Lefuneste83/KLOR](https://github.com/Lefuneste83/KLOR) (which forks [GEIGEIGEIST/KLOR](https://github.com/GEIGEIGEIST/KLOR)) with Indonesia-sourceable component selections. PCB modifications target: MCU footprint → Waveshare RP2040-Zero, TRRS → USB-C 16-pin midmount, RGB orientation south-facing (already present in fork's `SK6812MINI_and_cherry_1` footprint — verify only).

Full spec: [PROJECT_SPEC.md](PROJECT_SPEC.md). Phase progress: [PHASE_LOG.md](PHASE_LOG.md). Folder overview: [README.md](README.md). Phase 1 deliverables: [docs/phase-1-recon/](docs/phase-1-recon/).

## Collaboration model (established 2026-05-13)

Project owner is a CTO with electronics + PCB-design background but **not a keyboard DIY expert**. Decision authority split 2 tiers (applies to AI assistants e.g. Claude):

- **REQUIREMENTS (Tier 1, user-owned, LOCKED):** Functional spec — what the keyboard must DO. Listed R1-R10 in [PROJECT_SPEC.md](PROJECT_SPEC.md). Don't propose changing these without explicit discussion.
- **COMPONENT DECISIONS (Tier 2, AI-owned, justified):** Specific part picks, design tradeoffs, assembly model. AI makes the call, briefly justifies in PROJECT_SPEC.md / PHASE_LOG.md. User reviews and may override.

**Default mode = AI decides.** Make the call confidently with technical justification. Ask user only when:
1. Tradeoff is genuinely 50/50 (no clear technical winner)
2. Decision touches user preference (aesthetics, cost ceiling, brand bias)
3. Sourcing or local-context reality requires knowledge user has but AI doesn't (Indonesia-specific shop availability, user's existing parts stash, etc.)

## Hard rules

1. **Requirements R1-R10 are LOCKED.** See [PROJECT_SPEC.md](PROJECT_SPEC.md) Tier 1.
2. **PCB modification scope is 2 mods + 1 verify:** MCU footprint (+ 4 col pins remap), USB-C connector, RGB orientation verify. Plus housekeeping (drop haptic/audio/battery/trackball placements). Don't add a 4th mod. "While we're at it" reasoning is forbidden; flag separately for future scope.
3. **KEEP list is sacred:** switch footprint area, 19.05 mm spacing, encoder placement (matrix [3,5]/[7,5] coords), OLED area, diode positions, matrix routing logic, LED chain order.

## Working principles

1. **Step-by-step.** Produce ONE deliverable per turn. Wait for user confirmation before next. Do NOT batch multiple Phase deliverables.
2. **Make decisions confidently.** Component-level: pick + justify in 1-2 sentences. Don't ask user for permission unless one of the 3 conditions above applies.
3. **Cite sources.** Every technical claim → `file/path:line` or full URL.
4. **No bulk file ops without dry-run.** Multi-file moves/destructive actions → show plan, wait for "go".
5. **Markdown for documentation.** Readable structure, tables where they help.
6. **Indonesian + English mix.** User narrates in Bahasa Indonesia; technical terms in English (matrix, footprint, schematic, transport). Reply in same mix.
7. **Verifiable claims.** Every pin/setting/spec assertion must trace to a source file or URL.

## Environment

- OS: Windows 11
- Shell: PowerShell primary
- KiCad: 10.0.2 release build (May 2026)
- Git: available
- QMK CLI: NOT installed; deferred to Phase 6

## Component sourcing context (Indonesia)

Project owner in Indonesia. All component sourcing decisions must work within this:

**Primary channels:** Tokopedia / Shopee (most hobby electronics), AliExpress (specialty, 2-4 wk wait), LCSC/Mouser (rarely direct), JLCPCB (PCB fabrication only — no PCBA used).

**Known sourcing constraints:**
- Elite-Pi (Keeb.io): ❌ unavailable di Indonesia → RP2040-Zero locked
- Mill-Max 0305 sockets: ❌ Aliexpress 2-4 wk wait → use 2.54mm standard female pin header (Dupont-style)
- SMD soldering capability available — south-facing reverse-mount SK6812MINI-E doable by hand

Default to Tokopedia-sourceable parts. Flag wait time risks dalam decision justification.

## Folder map

```
KLORID/ (this repo)
├── README.md, PROJECT_SPEC.md, PHASE_LOG.md, CONTRIBUTING.md, LICENSE  ← root meta
├── hardware/                ← WORKING fork; edit here
│   ├── PCB/klor1_4/         ← MAIN target (Rev 1.4) — KiCad files
│   ├── FIRMWARE/            ← QMK + ZMK firmware variants
│   ├── case/                ← 3DP STL + acrylic DXF/SVG per variant
│   └── docs/                ← Fork's original hardware docs
├── docs/
│   └── phase-1-recon/       ← 5 markdown deliverables (Phase 1 outputs)
└── .gitignore               ← excludes .claude/, references/, working/, *.lck, .history/

LOCAL ONLY (not in repo — see .gitignore):
├── references/              ← READ-ONLY upstream clones (KLOR-upstream, zerosprey42, Skeletyl)
├── working/                 ← Derived artifacts only — never source of truth (gerbers, BOM, firmware build)
└── .claude/                 ← AI assistant harness state
```

## KiCad workflow conventions

- **Edit in-place** at `hardware/PCB/klor1_4/`. Do NOT duplicate.
- **Use git branches before destructive ops.** Multi-file changes / experimental work → feature branch first.
- **marbastlib** must be installed via KiCad's PCM (`com_github_ebastler_marbastlib`) at version matching KiCad 10.x. Provides USB-C 16-pin midmount footprints (HRO Type-C-31-M-12).
- RP2040-Zero footprint must match **Waveshare original** pinout — **2.54mm pitch through-hole pads** (1×9 left edge + 1×5 bottom + 1×9 right edge = 23 pads total). Socket via standard 2.54mm Dupont-style female pin header.

## Reference points for design decisions

### Pin mapping (current + post-remap)
- **Fork's original (elite_pi):** [hardware/FIRMWARE/qmk_rp2040/electronlab/klor/keyboard.json](hardware/FIRMWARE/qmk_rp2040/electronlab/klor/keyboard.json) — `matrix_pins`, `ws2812`, `encoder` blocks
- **Our remap (RP2040-Zero):** [docs/phase-1-recon/PIN_MAPPING_PROPOSAL.md](docs/phase-1-recon/PIN_MAPPING_PROPOSAL.md) — full 18/20 GPIO allocation, col remap GP9/10/11/12 (ascending)

### Firmware config — bare RP2040-Zero (Justin Lam findings)
Per [Justin Lam handwired Skeletyl blog](https://www.justinmklam.com/posts/2025/08/handwired-skeletyl/):

```c
// config.h additions WAJIB untuk RP2040-Zero:
#define RP2040_BOOTLOADER_DOUBLE_TAP_RESET
#define RP2040_BOOTLOADER_DOUBLE_TAP_RESET_TIMEOUT 500U
#define PICO_XOSC_STARTUP_DELAY_MULTIPLIER 64
#define SERIAL_PIO_USE_PIO1                      // 🔑 prevent PIO conflict with WS2812
```

```make
# rules.mk:
SERIAL_DRIVER = vendor
```

```diff
// keyboard.json:
- "development_board": "elite_pi"
+ "processor": "RP2040", "bootloader": "rp2040"
```

### Inter-half serial (current state, fork)
- [hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h](hardware/FIRMWARE/qmk_rp2040/electronlab/klor/config.h) — half-duplex GP1 currently active; full-duplex GP1/GP4 commented documented
- PCB design route both GP1 + GP4 for future-proof upgrade

### QMK root patch (PWM4)
- [hardware/CHANGELOG.md](hardware/CHANGELOG.md) — edit `./platforms/chibios/boards/QMK_PM2040/configs/mcuconf.h` set `RP_PWM_USE_PWM4 TRUE`. (Audio dropped from our scope so this MAY no longer be needed; verify Phase 6.)

### Other references (kept local, not in repo)
- **42-key RP2040-Zero exemplar:** `references/zerosprey42/` — unibody (not split), but validates MCU choice
- **Waveshare RP2040-Zero pinout:** https://www.waveshare.com/wiki/RP2040-Zero
- **Fork README PCB notes:** [hardware/README.md](hardware/README.md) — Rev 1.4 = main target
- **KLORpinout.png:** `references/KLOR-upstream/docs/images/KLORpinout.png` — Pro Micro pinout visual (validated 100% consistent dengan keyboard.json + config.h)

## Things to never silently do

- Run `git push` (publishes work) — confirm first
- Run `git reset --hard`, `git rebase -i`, `git checkout --` on user changes — destructive
- Auto-migrate KiCad files without warning user about reversibility + branching first
- Bulk-edit `keyboard.json` features without listing each toggle
- Install QMK CLI / change Python env without asking
- Move/rename anything in `references/` (cloned read-only snapshots)
- Touch `.claude/` folder at root (harness state, gitignored)
- Use `--no-verify` on git commits unless user explicitly asks

## DROPPED features (confirmed)

Out of scope; do not propose re-adding without explicit user discussion:
- Haptic feedback (DRV2605L + ERM/LRA motor + Pimoroni footprint)
- Audio (PWM speaker on GP9 + `lib/klounds.h` sound clips)
- **Trackball / pointing device module** (section di fork schematic, dropped Phase 2 housekeeping)
- ZMK firmware variant (QMK only)
- Bluetooth / Nice Nano support (wired RP2040-Zero only)
- Battery + power switch chain (wired-only build): MSK12C02 + JST-PH + custombatteryconnector
- `lvc27_onlyInput` 74LVC2G17 buffer (haptic-support related)

## Memory & state

AI assistants (e.g. Claude Code) maintain personal memory in user's home `.claude/projects/` directory (machine-local, not in repo). **Project facts live in CONTRIBUTING.md (this file), PROJECT_SPEC.md, and PHASE_LOG.md** — those are sources of truth. Personal memory captures only user profile + collaboration style.
