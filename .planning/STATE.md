---
gsd_state_version: 1.0
milestone: null
milestone_name: null
status: between_milestones
stopped_at: v1.1 milestone completed and archived
last_updated: "2026-03-13"
last_activity: 2026-03-13 — Completed v1.1 milestone, archived to milestones/
progress:
  total_phases: 0
  completed_phases: 0
  total_plans: 0
  completed_plans: 0
  percent: 0
---

# State: Open-EZ PDE

## Project Reference

See: .planning/PROJECT.md (updated 2026-03-13)

**Core value:** All physics outputs validated against independent reference data
**Current focus:** Planning next milestone

## Current Position

Status: Between milestones — v1.1 shipped, next milestone not yet started
Last activity: 2026-03-13 — v1.1 Physical Validation & Calibration shipped

## Accumulated Context

### Carry-Forward

- CadQuery/OCP must be mocked in tests: `sys.modules.setdefault("cadquery", MagicMock())`
- Lazy imports in `core/__init__.py` via `__getattr__` + `_LAZY_IMPORTS` dict
- `apply_washout(0.0)` reconstructs Airfoil from split coords — use relaxed tolerances
- `analyze_distributed` API takes `total_load_lbf` (not `load_per_inch`)
- AR discrepancy (published 7.3 vs computed 6.34) is reference area convention, not bug
- OpenVSP 3.48.2 requires Python 3.13 for .so bindings (macOS ARM64 only)
- Sym_Planar_Flag=0.0 required on WING geoms for VSPAERO
- Parse .polar file directly (GetDoubleResults not populated in VSPGEOM-mode)
- fs_wing_le calibration (125.61") pending CP-31 physical confirmation

## Session Continuity

Last session: 2026-03-13
Stopped at: v1.1 milestone completed and archived
Resume file: None
