---
phase: 03-openvsp-native-integration
plan: "06"
title: Fix VSPAERO VLM solver pipeline — thin surface export and geometry validation
subsystem: aerodynamics/vsp-integration
tags: [openvsp, vspaero, vlm, geometry, polar]
requirements: [VSP-02]

dependency_graph:
  requires: [03-05]
  provides: [vspaero-native-polars, working-vlm-pipeline]
  affects: [main.py analysis output, data/validation/vspaero_native_polars.json]

tech_stack:
  added: []
  patterns:
    - "Sym_Planar_Flag=0.0 for half-span VLM half-model (VSPAERO mirrors via Symmetry=1)"
    - "Parse .polar file directly after ExecAnalysis (GetDoubleResults not populated in 3.48.2)"
    - "_parse_vspaero_polar() static method for reusable polar file parsing"

key_files:
  created:
    - scripts/vspaero_diagnostic.py
    - data/validation/vspaero_native_polars.json
  modified:
    - core/vsp_integration.py
    - tests/test_vsp_native.py

decisions:
  - "Sym_Planar_Flag=0.0 on WING geoms: OpenVSP default is 2.0 (XZ symmetry), which exports both wing halves to VSPGEOM (2 surfaces, 11+ kutta nodes). Setting 0.0 produces half-span model (1 surface, 6 kutta nodes) that VSPAERO mirrors via Symmetry=1"
  - "ExportFile thick_set=SET_NONE, thin_set=THIN_SET (user set 3): only lifting surfaces in VSPGEOM, no fuselage displacement bodies"
  - "GetDoubleResults() does not populate from VSPGEOM-mode VLM analysis in OpenVSP 3.48.2; parse .polar file directly"
  - "VSPGEOM filename must share exact base path with VSP3 file (VSPAERO derives input path from SetVSP3FileName)"

metrics:
  duration: "~90m across 2 sessions"
  completed: "2026-03-12"
  tasks_completed: 3
  files_modified: 4
---

# Phase 03 Plan 06: Fix VSPAERO VLM Solver Pipeline Summary

Real VSPAERO VLM analysis pipeline working: `python3.13 main.py --analysis` prints "Using native VSPAERO", iterates 5 wake passes per alpha point across a 19-point sweep, and writes physically plausible CL/CD/CM polars to `vspaero_native_polars.json`.

## Tasks Completed

| Task | Description | Commit |
|------|-------------|--------|
| 1 | Create standalone VSPAERO diagnostic script | 889422b |
| 2 | Fix `_run_native_sweep()` based on diagnostic findings | 824a148 |
| 3 | Validate polars and commit `vspaero_native_polars.json` | bc5133c |

## Root Cause Found and Fixed

**Symptom:** VSPAERO crashed with "Could not find global edge for trailing wake! Looking for edge with nodes: 196, 197" — the solver exited immediately, 0 thin surfaces, 0 kutta nodes, no polar data.

**Root cause:** OpenVSP WING geoms default to `Sym_Planar_Flag=2.0` (XZ plane symmetry). When `ExportFile()` is called with symmetry enabled, it exports **both wing halves** to the VSPGEOM:
- Full span: Y=-span/2 to +span/2
- 2 surface copies (`copy#=1` and `copy#=2` in `.vkey`)
- 187 nodes, 11 kutta nodes

VSPAERO 7.2.2 with `Symmetry=1` in the `.vspaero` file expects a **half-span model**:
- Right half only: Y=0 to +span/2
- 1 surface copy (`copy#=1` only in `.vkey`)
- 102 nodes, 6 kutta nodes

When given both halves, VSPAERO adds wake nodes for both sides simultaneously, creating duplicate trailing-edge topology that fails the global edge lookup. The reference file `test_vlm2.vspgeom` (which worked) was confirmed to have only 1 surface copy by comparing Y ranges (0 to 100 vs -100 to +100).

**Fix:** Explicitly set `Sym_Planar_Flag=0.0` (no OpenVSP internal symmetry) on all WING geoms before `ExportFile()`. Only the right half of each lifting surface is defined; VSPAERO handles the mirroring via `Symmetry=1`.

## Results

```
python3.13 main.py --analysis:
  Aerodynamic sweep: Using native VSPAERO
  Polar data: 19 points written to data/validation/vspaero_native_polars.json

CL/deg slope: 0.176/deg (physically plausible)
CL at alpha=4 deg: 0.634 (within 0.1-1.0 range for Long-EZ)
All CD positive and increasing with alpha^2: True
```

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] GetDoubleResults() returns zeros for VSPGEOM-mode VLM analysis**
- **Found during:** Task 1 diagnostic script
- **Issue:** `vsp.GetDoubleResults(results_id, "CL")` returns empty/zero arrays after `ExecAnalysis("VSPAEROSweep")` when using VSPGEOM-mode in OpenVSP 3.48.2. This is a known limitation of the Python bindings.
- **Fix:** Added `_parse_vspaero_polar()` static method to parse the `.polar` file written to disk by VSPAERO. Rewrote mock tests to write fake `.polar` files instead of mocking `GetDoubleResults`.
- **Files modified:** `core/vsp_integration.py`, `tests/test_vsp_native.py`
- **Commit:** 824a148

**2. [Rule 1 - Bug] VSPGEOM filename must match VSP3 base path exactly**
- **Found during:** Task 1 diagnostic
- **Issue:** VSPAERO derives its input `.vspgeom` path from `SetVSP3FileName()`. Exporting with a different suffix (e.g., `_optA.vspgeom`) causes VSPAERO to look for the wrong file.
- **Fix:** Use `vsp3_path.replace(".vsp3", ".vspgeom")` to ensure the base name matches.
- **Files modified:** `core/vsp_integration.py`
- **Commit:** 824a148

**3. [Rule 1 - Bug] Default Sym_Planar_Flag=2.0 exports full-span geometry, crashing VSPAERO**
- **Found during:** Task 2 debugging — comparing `test_vlm2.vkey` (1 copy, works) vs generated `.vkey` (2 copies, crashes). Y-coordinate analysis confirmed full span (Y=-100 to +100) vs half span (Y=0 to +100).
- **Fix:** Set `Sym_Planar_Flag=0.0` on all WING geoms. Documented in code comment.
- **Files modified:** `core/vsp_integration.py`
- **Commit:** 824a148

## Verification

All four plan success criteria confirmed:

1. `python3.13 main.py --analysis` → "Using native VSPAERO", solver iterates 5 wake passes per alpha, 19 CL/CD/CM values produced
2. `data/validation/vspaero_native_polars.json` → `vsp_version: "OpenVSP 3.48.2"`, 19 data points, CL/deg=0.176, CL=0.634 at alpha=4
3. `python3 main.py --analysis` (no openvsp) → "Using surrogate (OpenVSP not installed)", no crash
4. `python3 -m pytest tests/ -x -q` → 197 passed, 1 skipped, 0 failures

## Self-Check: PASSED

All files exist at expected paths. All per-task commits verified in git log.
