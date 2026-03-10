---
gsd_state_version: 1.0
milestone: v1.1
milestone_name: Physical Validation & Calibration
status: planning
stopped_at: Completed 01-01-PLAN.md
last_updated: "2026-03-10T04:58:51.587Z"
last_activity: 2026-03-09 — Roadmap created for v1.1
progress:
  total_phases: 6
  completed_phases: 0
  total_plans: 2
  completed_plans: 1
  percent: 0
---

# State: Open-EZ PDE

## Project Reference

See: .planning/PROJECT.md (updated 2026-03-09)

**Core value:** All physics outputs validated against independent reference data
**Current focus:** v1.1 Physical Validation & Calibration — Phase 1 ready to plan

## Current Position

Phase: 0 of 6 (not started)
Plan: —
Status: Ready to plan Phase 1
Last activity: 2026-03-09 — Roadmap created for v1.1

Progress: [░░░░░░░░░░] 0%

## Performance Metrics

**Velocity:**
- Total plans completed: 0
- Average duration: —
- Total execution time: —

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| - | - | - | - |

*Updated after each plan completion*
| Phase 01 P01 | 3m | 2 tasks | 2 files |

## Accumulated Context

### Decisions

Decisions logged in PROJECT.md Key Decisions table.
Key pending decisions for v1.1:

- [Pending]: Datum offset as additive translation constant `datum_offset_in` — not a geometry change
- [Pending]: D-box composite model replaces cap-only I-beam in `fea_adapter.py`
- [Pending]: Surrogate fallback for CI when OpenVSP not installed
- [Phase 01-01]: datum_offset_in = 45.5 in (exact, from NP comparison: internal 153.5 - published 108.0); the previously estimated 51" was imprecise
- [Phase 01-01]: reference_data.json uses published Long-EZ FS datum exclusively; code uses to_published_datum() for conversion
- [Phase 01-01]: AR discrepancy (7.3 published vs 6.34 computed) is reference area convention difference, documented in reference_data.json notes

### Blockers/Concerns

- Phase 3: OpenVSP Python bindings install on MacBook not yet verified — may need Homebrew formula or source build
- Phase 4: Validation tolerances (2" / 3% for NP) are estimates — may need adjustment after seeing real VSPAERO vs surrogate discrepancy
- Phase 6: `physics_baseline.json` is self-referential — do not extend it; Phase 6 replaces it

### Carry-Forward from v1.0

- CadQuery/OCP must be mocked in tests: `sys.modules.setdefault("cadquery", MagicMock())`
- Lazy imports in `core/__init__.py` via `__getattr__` + `_LAZY_IMPORTS` dict
- `apply_washout(0.0)` reconstructs Airfoil from split coords — use relaxed tolerances
- `analyze_distributed` API takes `total_load_lbf` (not `load_per_inch`)
- AR discrepancy (published 7.3 vs computed 6.34) is reference area convention, not bug

## Session Continuity

Last session: 2026-03-10T04:58:51.576Z
Stopped at: Completed 01-01-PLAN.md
Resume file: None
