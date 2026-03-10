---
phase: 03-openvsp-native-integration
plan: "02"
subsystem: aerodynamics
tags: [openvsp, vspaero, vlm, surrogate, ci]
dependency_graph:
  requires: [03-01]
  provides: [native-vspaero-sweep, surrogate-fallback-ci]
  affects: [main.py, core/vsp_integration.py, tests/test_vsp_native.py]
tech_stack:
  added: []
  patterns: [TDD red-green, surrogate-fallback, OpenVSP VLM sweep]
key_files:
  created:
    - tests/test_vsp_native.py
    - data/validation/vspaero_native_polars.json (runtime artifact)
  modified:
    - core/vsp_integration.py
    - main.py
decisions:
  - "_run_native_sweep() accepts optional polar_output: Path param for test isolation"
  - "wing_mac = (root + tip) / 2 used for cref — no wing_mac property on GeometricParams"
  - "is_stable restored to surrogate return dict for backward compat with test_module_coverage.py"
  - "Runtime exception in native sweep falls back silently to surrogate (never crashes pipeline)"
metrics:
  duration: "8m"
  completed: "2026-03-10"
  tasks: 2
  files: 3
requirements: [VSP-02, VSP-04]
---

# Phase 03 Plan 02: Native VSPAERO Integration Summary

One-liner: VSPAERO VLM sweep (_run_native_sweep) builds Long-EZ geometry via OpenVSP API and runs 19-point alpha sweep with CI-safe surrogate fallback through OpenVSPAdapter.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Implement _run_native_sweep() with VSPAERO geometry builder | 5f13cf4 | core/vsp_integration.py, tests/test_vsp_native.py |
| 2 | Wire native VSPAERO into main.py pipeline with mode logging | 85e6937 | main.py, core/vsp_integration.py |

## What Was Built

**core/vsp_integration.py** — Complete rewrite of `_run_native_sweep()`:
- Clears VSP model and builds Long-EZ geometry (wing, canard, winglets, fuselage) using the same VSP API pattern as `export_native_vsp3()`
- Configures VSPAERO: VLM method, alpha -4 to +14 deg (19 points), Mach 0.0, Y-symmetry
- Sets Sref/bref/cref from config geometry (wing_area, wing_span, wing_mac)
- Executes `ExecAnalysis("VSPAEROSweep")`, extracts CL/CD/CMy arrays
- Writes `data/validation/vspaero_native_polars.json` with source/version/timestamp/solver_settings
- Returns `{mode, source, points, solver_settings, vsp_version}` dict

**_run_surrogate_sweep()** replaced inline mockup with `OpenVSPAdapter.run_vspaero()`, retaining `is_stable` for backward compatibility.

**main.py** — `run_analysis()` calls `vsp_bridge.run_aerodynamic_sweep()` after native VSP3 export. Prints "Using native VSPAERO" or "Using surrogate (OpenVSP not installed)" accordingly.

**tests/test_vsp_native.py** — 6 tests (5 CI-safe, 1 skipped without OpenVSP):
- Surrogate fallback when openvsp unavailable
- Native return schema with mock vsp
- Alpha sweep range (-4 to 14, 19 points)
- Polar JSON file written after native sweep
- Runtime exception falls back to surrogate
- Real native sweep (skip if no OpenVSP)

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Missing wing_mac property on GeometricParams**
- **Found during:** Task 1 implementation
- **Issue:** Plan specified `geom.wing_mac` for VSPAERO cref, but `GeometricParams` has no `wing_mac` attribute
- **Fix:** Computed inline as `(geom.wing_root_chord + geom.wing_tip_chord) / 2.0` — correct for trapezoidal wing
- **Files modified:** core/vsp_integration.py
- **Commit:** 5f13cf4

**2. [Rule 1 - Bug] Surrogate sweep broke is_stable contract**
- **Found during:** Task 2 full test run
- **Issue:** Old `_run_surrogate_sweep()` returned `is_stable`; new OpenVSPAdapter-backed version didn't; `test_module_coverage.py` asserted it
- **Fix:** Added `is_stable` back via `physics.calculate_cg_envelope().is_stable` with backward-compat comment
- **Files modified:** core/vsp_integration.py
- **Commit:** 85e6937

## Verification

- `python3 -m pytest tests/ -x -q` — 196 passed, 1 skipped (real openvsp)
- With OpenVSP (python3.13): `run_aerodynamic_sweep()` returns `mode="native"`, writes polar JSON
- Without OpenVSP (python3): returns `mode="surrogate"`, never crashes
- Plan must_haves all satisfied: _run_native_sweep implemented, main.py logs mode, surrogate fallback tested in CI

## Self-Check: PASSED
