---
status: diagnosed
phase: 03-openvsp-native-integration
source: 03-01-SUMMARY.md, 03-02-SUMMARY.md
started: 2026-03-10T20:00:00Z
updated: 2026-03-10T20:15:00Z
---

## Current Test

[testing complete]

## Tests

### 1. VSP Geometry Config Fields
expected: Run `python3 -c "from config import config; g=config.geometry; print(g.wing_le_wl, g.canard_le_wl, g.winglet_height, g.winglet_root_chord, g.winglet_tip_chord, g.fuselage_length, g.wing_le_fs, g.canard_le_fs)"` — prints `0.0 12.0 16.0 20.0 12.0 214.0 133.0 36.0` (8 values, all numeric, no errors)
result: pass

### 2. OpenVSP Python Bindings
expected: Run `python3.13 -c "import openvsp as vsp; print(vsp.GetVSPVersion())"` — prints `OpenVSP 3.48.2` with no import errors
result: pass

### 3. Native VSPAERO Sweep Runs
expected: Run `python3.13 main.py --generate-all` — output includes "Using native VSPAERO" (not "Using surrogate"), completes without errors, and the VSPAERO VLM sweep produces CL/CD data
result: issue
reported: "Traceback: ModuleNotFoundError: No module named 'cadquery' — main.py crashes on import of core.structures which does `import cadquery as cq` at module level. Python 3.13 (required for OpenVSP) does not have cadquery installed."
severity: blocker

### 4. Surrogate Fallback
expected: Run `python3 main.py --generate-all` (system Python without OpenVSP) — output includes "Using surrogate (OpenVSP not installed)", completes without crash, returns aerodynamic results via surrogate model
result: issue
reported: "Same ModuleNotFoundError: No module named 'cadquery' — main.py crashes on import with system Python 3.14 as well. Neither Python environment can run main.py."
severity: blocker

### 5. Polar JSON Output
expected: After a native sweep, `data/validation/vspaero_native_polars.json` exists and contains: `source`, `vsp_version`, `timestamp`, `solver_settings`, and `alpha`/`CL`/`CD` arrays with 19 data points each
result: pass

### 6. Test Suite Green
expected: Run `python3 -m pytest tests/ -x -q` — all tests pass (196+), 1 skipped (real openvsp test in CI), zero failures
result: pass

## Summary

total: 6
passed: 4
issues: 2
pending: 0
skipped: 0

## Gaps

- truth: "main.py runs native VSPAERO sweep end-to-end with python3.13"
  status: failed
  reason: "User reported: ModuleNotFoundError: No module named 'cadquery' — main.py crashes on import of core.structures which does `import cadquery as cq` at module level. Python 3.13 does not have cadquery installed."
  severity: blocker
  test: 3
  root_cause: "main.py line 21 has top-level `from core.structures import CanardGenerator, WingGenerator` which eagerly triggers `import cadquery as cq` in core/structures.py, core/base.py, core/aerodynamics.py, and core/jig_factory.py. This crashes before argparse runs. The analysis/validation modes don't need cadquery at all. Tests pass because they mock cadquery via sys.modules."
  artifacts:
    - path: "main.py"
      issue: "Top-level import of cadquery-dependent modules on lines 21 and 23"
    - path: "core/structures.py"
      issue: "Eager `import cadquery as cq` at line 19"
  missing:
    - "Move `from core.structures import ...` and `from core.manufacturing import ...` from top-level into the functions that need them (generate_canard, generate_wing, generate_manufacturing)"
  debug_session: ""

- truth: "main.py runs surrogate fallback with system Python when OpenVSP unavailable"
  status: failed
  reason: "User reported: Same ModuleNotFoundError with system Python 3.14. Neither Python environment has cadquery, so main.py cannot be imported at all."
  severity: blocker
  test: 4
  root_cause: "Same root cause as test 3 — top-level cadquery imports in main.py crash before argparse runs. The surrogate fallback path (analysis only) never needs cadquery, but the eager import kills it."
  artifacts:
    - path: "main.py"
      issue: "Lines 21 and 23 import cadquery-dependent modules at top level"
  missing:
    - "Defer cadquery-dependent imports so analysis/validation/surrogate paths work without cadquery"
  debug_session: ""
