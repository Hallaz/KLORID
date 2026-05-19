# KLORID

42-key split keyboard, polydactyl layout, sourced untuk komponen yang mudah didapat di **Indonesia** (Tokopedia / Shopee). Derivative dari [Lefuneste83/KLOR](https://github.com/Lefuneste83/KLOR), yang sendirinya fork dari [GEIGEIGEIST/KLOR](https://github.com/GEIGEIGEIST/KLOR).

> **"ID"** suffix = Indonesia variant. Sourcing decisions menghindari long-wait import (Mill-Max, Elite-Pi, etc.), default ke part Tokopedia-sourceable.

## Hardware modifications dari upstream KLOR

### Core PCB modifications (Phase 2)

| # | Modification | Source → Target |
|---|---|---|
| MOD #1 | MCU footprint | Elite-Pi (Pro-Micro form factor) → bare **Waveshare RP2040-Zero** (2.54mm pitch through-hole) |
| MOD #2 | Inter-half connector | TRRS jack MJ-4PP-9 → **USB-C 16-pin midmount** (HRO Type-C-31-M-12) |
| MOD #3 | RGB orientation | south-facing **verified** — fork's existing `SK6812MINI_and_cherry_1` footprint sudah benar (no schematic change needed) |

### Phase 3 PCB layout enhancements

| Phase | Enhancement |
|---|---|
| **3e** | **Dual-purpose SW23 at matrix [3,5]** — single position serves either encoder OR MX switch, builder chooses per side at assembly (mix possible — encoder on one half, switch on other) |
| **3f** | Left thumb cluster **breakaway notch removed** (real-world fragility evidence — fork's intentional snap-off point breaks too easily under normal handling) |
| **3g** | **Haptic feedback re-added as DNP** — Pimoroni Haptic Buzz 5-pin connector (`Conn_01x05`), I2C-shared with OLED (GP2/GP3), trigger on GP14. Optional install at assembly time; PCB ships with footprint+routing ready. |
| **3h** | **Silkscreen branding** — KLORID text + Garuda Pancasila image (Indonesia national emblem) on F.SilkS layer |

### Dropped from upstream (out of scope for V1)

- Audio speaker (PWM buzzer on GP9 — pin now repurposed sebagai matrix col 2)
- Battery + power switch chain (wired-only build)
- Trackball / pointing device module
- `lvc27_onlyInput` buffer (haptic-related, not needed dengan Pimoroni breakout approach)

## Status — Phase 4 ready (Gerber export complete)

PCB design lengkap. All Phase 3 enhancements committed. Gerber files regenerated (ZIP siap upload). Pending: actual JLCPCB order + case fabrication.

**Latest milestones:**
- ✅ Phase 3a-3h PCB layout complete (DRC clean, 0 violations)
- ✅ Schematic + PCB sync (ERC clean, 0 errors)
- ✅ Gerber export untuk all layers (`hardware/PCB/klor1_4/gerbers/gerbers.zip`)
- ✅ Switchplate DXF generated (`working/switchplate_klorid.dxf` — for CO2 laser cut)
- ⏳ JLCPCB order pending
- ⏳ Other case acrylic layers (bottom plate, spacers, top frame) pending generation
- ⏳ Firmware Phase 6 deferred (QMK + Vial dengan Justin Lam RP2040-Zero config)

Detail per-phase progress di [PHASE_LOG.md](PHASE_LOG.md).

## Case strategy (Indonesia-friendly)

PCB modifications drive case requirements (USB-C cutout, encoder/MX position, OLED window). Case approach untuk Indonesia sourcing:

- **PCB fab:** JLCPCB (2-layer FR4 1.6mm, black HASL)
- **Switchplate + acrylic layers:** Local CO2 laser cut shop (DXF generated programmatically from PCB)
- **Case body 3DP:** JLCPCB 3D printing service atau lokal print
- **Hardware:** M2 standoffs + screws from Tokopedia

## Spec & decisions

- [PROJECT_SPEC.md](PROJECT_SPEC.md) — Tier 1 (locked Requirements R1-R10) + Tier 2 (Component Decisions dengan justifikasi)
- [PHASE_LOG.md](PHASE_LOG.md) — Build progress log (append-only, latest at bottom)
- [CONTRIBUTING.md](CONTRIBUTING.md) — Collaboration model + KiCad workflow + hard rules
- [docs/phase-1-recon/](docs/phase-1-recon/) — Phase 1 reconnaissance outputs (5 deliverables)

## Repository structure

```
KLORID/
├── README.md             ← (you are here)
├── LICENSE               ← GPL-3.0 (inherited from upstream)
├── PROJECT_SPEC.md       ← Requirements + component decisions
├── PHASE_LOG.md          ← Build progress
├── CONTRIBUTING.md       ← Collaboration + workflow rules
├── docs/
│   └── phase-1-recon/    ← Phase 1 deliverables
└── hardware/             ← KiCad PCB + schematic + firmware + case
    ├── PCB/klor1_4/      ← Active design (KiCad 10)
    ├── FIRMWARE/         ← QMK + Vial firmware
    ├── case/             ← 3DP STL + acrylic DXF
    └── docs/             ← Fork's original hardware docs
```

Excluded (local-only, not in repo): `references/` (upstream snapshots untuk development context), `working/` (derived gerbers/firmware build artifacts), `.claude/` (harness state).

## License

**GPL-3.0** — same as upstream KLOR. Derivative works WAJIB tetap GPL-3.0. See [LICENSE](LICENSE).

## Attribution

Credit chain:
- **GEIGEIGEIST/KLOR** — original KLOR design
- **Lefuneste83/KLOR** — fork dengan klor1_4 (Rev 1.4) PCB
- **Hallaz/KLORID** — Indonesia-sourceable variant (this repo)

Built with Claude Code as collaborator. AI-assisted PCB modification per [CONTRIBUTING.md](CONTRIBUTING.md) collaboration model.
