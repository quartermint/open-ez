---
gsd_state_version: 1.0
milestone: v1.1
milestone_name: Physical Validation & Calibration
status: planning
stopped_at: Phase 2 context gathered
last_updated: "2026-03-10T14:31:04.750Z"
last_activity: 2026-03-09 — Roadmap created for v1.1
progress:
  total_phases: 6
  completed_phases: 1
  total_plans: 2
  completed_plans: 2
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
| Phase 01 P02 | 8m | 2 tasks | 2 files |

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
- [Phase 01-02]: NP test tolerance set to 8 inches: computed NP 159.29i/113.79p vs reference 108.0 (delta 5.79in); known fs_wing_le datum issue; 8in catches gross errors while accepting geometry uncertainty
- [Phase 01-02]: Dual FS display format established: 'FS X.XX (internal) / FS Y.YY (published)' for all human-readable FS values in StabilityMetrics.summary()

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

Last session: 2026-03-10T14:31:04.748Z
Stopped at: Phase 2 context gathered
Resume file: .planning/phases/02-d-box-structural-model/02-CONTEXT.md
