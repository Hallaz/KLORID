# KLORID

42-key split keyboard, polydactyl layout, sourced untuk komponen yang mudah didapat di **Indonesia** (Tokopedia / Shopee). Derivative dari [Lefuneste83/KLOR](https://github.com/Lefuneste83/KLOR), yang sendirinya fork dari [GEIGEIGEIST/KLOR](https://github.com/GEIGEIGEIST/KLOR).

> **"ID"** suffix = Indonesia variant. Sourcing decisions menghindari long-wait import (Mill-Max, Elite-Pi, etc.), default ke part Tokopedia-sourceable.

## Hardware modifications dari upstream KLOR

| # | Modification | Source → Target |
|---|---|---|
| MOD #1 | MCU footprint | Elite-Pi (Pro-Micro form factor) → bare **Waveshare RP2040-Zero** (2.54mm pitch through-hole) |
| MOD #2 | Inter-half connector | TRRS jack MJ-4PP-9 → **USB-C 16-pin midmount** (HRO Type-C-31-M-12) |
| MOD #3 | RGB orientation | south-facing **verified** — fork's existing `SK6812MINI_and_cherry_1` footprint sudah benar (no schematic change needed) |

Plus housekeeping (out-of-scope features dropped from original fork):
- Haptic feedback (DRV2605L + Pimoroni breakout)
- Audio speaker (PWM buzzer on GP9 — pin now repurposed sebagai matrix col 2)
- Battery + power switch chain (wired-only build)
- Trackball / pointing device module
- `lvc27_onlyInput` buffer (haptic-support related)

## Status — Phase 2 close

Schematic modifications complete, PCB update applied. Detail per-phase progress di [PHASE_LOG.md](PHASE_LOG.md).

Next: **Phase 3 PCB layout** — reposition new components (RP2040-Zero, USB-C, R1/R2 CC pulldowns), re-route matrix traces, clean up dangling traces dari removed components.

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
