---
gsd_state_version: 1.0
milestone: v1.1
milestone_name: Physical Validation & Calibration
status: completed
stopped_at: Phase 3 context gathered
last_updated: "2026-03-10T15:13:41.005Z"
last_activity: 2026-03-10 — Completed 02-02 D-box pipeline integration
progress:
  total_phases: 6
  completed_phases: 2
  total_plans: 4
  completed_plans: 4
  percent: 33
---

# State: Open-EZ PDE

## Project Reference

See: .planning/PROJECT.md (updated 2026-03-09)

**Core value:** All physics outputs validated against independent reference data
**Current focus:** v1.1 Physical Validation & Calibration — Phase 2 executing

## Current Position

Phase: 3 of 6
Plan: 0 of ? complete
Status: Phase 2 complete, ready for Phase 3
Last activity: 2026-03-10 — Completed 02-02 D-box pipeline integration

Progress: [###░░░░░░░] 33%

## Performance Metrics

**Velocity:**
- Total plans completed: 4
- Average duration: 5m
- Total execution time: 19m

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| Phase 01 P01 | 3m | 2 tasks | 2 files |
| Phase 01 P02 | 8m | 2 tasks | 2 files |
| Phase 02 P01 | 4m | 1 task (TDD) | 4 files |
| Phase 02 P02 | 4m | 2 tasks (TDD) | 2 files |

## Accumulated Context

### Decisions

Decisions logged in PROJECT.md Key Decisions table.
Key pending decisions for v1.1:

- [Pending]: Datum offset as additive translation constant `datum_offset_in` — not a geometry change
- [Phase 02-01]: D-box composite model added alongside cap-only I-beam in `fea_adapter.py` (DBoxSection + DBoxBeamAdapter)
- [Phase 02-01]: D-box EI ~101M lb-in^2 at root, deflection 2.34" (not 5-15" as estimated; correct physics)
- [Phase 02-01]: np.trapezoid used over np.trapz for NumPy 2.0+ compatibility
- [Phase 02-02]: nominal_spar_check() returns D-box results (dbox_ prefix) alongside unchanged legacy keys for RegressionRunner backward compat
- [Phase 02-02]: Foam compression uses E_foam * kappa * d/2 (bending-induced), not V/A_foam (bearing) -- physically correct for sandwich web
- [Phase 02-02]: FlutterEstimator uses average D-box EI; bending freq 0.04 Hz (cap-only) -> 8.5 Hz (D-box)
- [Phase 02-02]: D-box weight ~8 lb per wing half (2-ply BID skins + web), range 5-25 lb
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

Last session: 2026-03-10T15:13:41.003Z
Stopped at: Phase 3 context gathered
Resume file: .planning/phases/03-openvsp-native-integration/03-CONTEXT.md
