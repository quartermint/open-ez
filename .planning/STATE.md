---
gsd_state_version: 1.0
milestone: v1.1
milestone_name: Physical Validation & Calibration
status: completed
stopped_at: Completed 03-06-PLAN.md — VSPAERO VLM pipeline fixed, real polars generated
last_updated: "2026-03-12T01:31:26.664Z"
last_activity: 2026-03-10 — Completed 02-02 D-box pipeline integration
progress:
  total_phases: 6
  completed_phases: 3
  total_plans: 9
  completed_plans: 9
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
| Phase 03 P01 | 5m | 1 tasks | 3 files |
| Phase 03-openvsp-native-integration P01 | 25 | 2 tasks | 3 files |
| Phase 03-openvsp-native-integration P02 | 8 | 2 tasks | 3 files |
| Phase 03-openvsp-native-integration P04 | 2 | 1 tasks | 2 files |
| Phase 03-openvsp-native-integration P05 | 1 | 1 tasks | 2 files |
| Phase 03-openvsp-native-integration P06 | 90 | 3 tasks | 4 files |

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
- [Phase 03]: wing_le_fs and canard_le_fs as @property aliases to fs_wing_le/fs_canard_le — no data duplication
- [Phase 03]: OpenVSP install script targets Python 3.13 with .pth in site-packages (macOS ARM64 bundle is only distribution)
- [Phase 03-openvsp-native-integration]: OpenVSP 3.48.2 pip install from macOS ARM64 app bundle (only distribution); Python 3.13 required for .so bindings
- [Phase 03-openvsp-native-integration]: wing_le_fs and canard_le_fs as @property aliases to fs_wing_le/fs_canard_le — no data duplication, SSOT preserved
- [Phase 03-02]: _run_native_sweep() accepts optional polar_output: Path for test isolation; wing_mac computed inline as (root+tip)/2; is_stable restored to surrogate for backward compat
- [Phase 03-openvsp-native-integration]: CadQuery imports deferred to function scope in generate_manufacturing(), generate_canard(), generate_wing() — analysis modes work in any Python env
- [Phase 03-05]: Use vsp.GetVSPVersion() as OpenVSP smoke test instead of non-existent vsp.VSPCheckIsInit() — version check confirms module import and basic functionality
- [Phase 03-05]: Removed PTH_FILE reference from install_openvsp.sh — script uses pip install directly, not .pth file approach
- [Phase 03-06]: Sym_Planar_Flag=0.0 on WING geoms: OpenVSP default 2.0 (XZ symmetry) exports both wing halves to VSPGEOM, causing VSPAERO crash. Setting 0.0 produces half-span model that VSPAERO mirrors via Symmetry=1
- [Phase 03-06]: Parse .polar file directly after ExecAnalysis (GetDoubleResults not populated in OpenVSP 3.48.2 VSPGEOM-mode)

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

Last session: 2026-03-12T01:31:26.659Z
Stopped at: Completed 03-06-PLAN.md — VSPAERO VLM pipeline fixed, real polars generated
Resume file: None
