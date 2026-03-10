---
phase: 03-openvsp-native-integration
plan: "01"
subsystem: config
tags: [openvsp, geometry, config, installation]
dependency_graph:
  requires: []
  provides: [VSP-01]
  affects: [core/openvsp_runner.py]
tech_stack:
  added: []
  patterns: [SSOT config extension, computed property aliases]
key_files:
  created:
    - scripts/install_openvsp.sh
  modified:
    - config/aircraft_config.py
    - requirements.txt
decisions:
  - "wing_le_fs and canard_le_fs implemented as @property aliases to fs_wing_le / fs_canard_le — avoids data duplication while satisfying export_native_vsp3() interface"
  - "OpenVSP install script targets Python 3.13 (macOS ARM64 bundle) with .pth file in site-packages — simplest non-venv approach"
  - "fuselage_length=214.0 matches fs_tail (tail cone terminus) as documented in plan"
  - "wing_le_wl=0.0 (wing at longeron level), canard_le_wl=12.0 (12 inch above longerons)"
metrics:
  duration: "5m"
  completed_date: "2026-03-10"
  tasks_completed: 1
  tasks_total: 2
  files_modified: 3
---

# Phase 3 Plan 01: VSP Geometry Parameters and Install Script Summary

**One-liner:** Added 8 missing OpenVSP geometry attributes to GeometricParams (6 fields + 2 property aliases) and created macOS ARM64 install script for OpenVSP 3.48.2 Python bindings.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Add VSP geometry parameters to GeometricParams and create install script | 843d5c1 | config/aircraft_config.py, scripts/install_openvsp.sh, requirements.txt |

## Tasks Pending (Human Action Required)

| Task | Name | Status |
|------|------|--------|
| 2 | Install OpenVSP Python bindings on MacBook | Awaiting user action |

## What Was Built

### GeometricParams Extensions (config/aircraft_config.py)

New fields added under `# === OPENVSP GEOMETRY (positions for 3D model) ===`:

| Field | Value | Notes |
|-------|-------|-------|
| `wing_le_wl` | `0.0` in | Wing LE waterline — at longeron level |
| `canard_le_wl` | `12.0` in | Canard ~12" above longerons |
| `winglet_height` | `16.0` in | Winglet vertical span (Rutan Ch.19) |
| `winglet_root_chord` | `20.0` in | At wing tip junction |
| `winglet_tip_chord` | `12.0` in | Winglet tip |
| `fuselage_length` | `214.0` in | = fs_tail (nose to terminus) |

New computed properties:

| Property | Returns | Purpose |
|----------|---------|---------|
| `wing_le_fs` | `fs_wing_le` (133.0) | OpenVSP X-position for wing |
| `canard_le_fs` | `fs_canard_le` (36.0) | OpenVSP X-position for canard |

All 8 attributes used by `export_native_vsp3()` are now accessible on `config.geometry`.

### Install Script (scripts/install_openvsp.sh)

Executable shell script that:
1. Verifies Python 3.13 is installed (OpenVSP 3.48.2 bundles 3.13 bindings)
2. Downloads OpenVSP 3.48.2 macOS ARM64 ZIP from GitHub releases
3. Extracts to `~/.local/openvsp/`
4. Auto-detects the `python/` subdirectory with `.so` bindings
5. Installs a `.pth` file in Python 3.13 site-packages
6. Runs smoke test: `import openvsp as vsp; print(vsp.GetVSPVersion())`
7. Provides troubleshooting guidance on failure (Gatekeeper, PYTHONPATH)

**Note:** OpenVSP is NOT available on pip or conda-forge. The macOS application bundle is the only distribution channel for prebuilt bindings.

### requirements.txt

Updated comment from `# openvsp>=3.35.0` to `# openvsp 3.48.2 — installed via scripts/install_openvsp.sh (not available on pip/conda)`.

## Verification

```
python3 -c "
from config import config
g = config.geometry
attrs = ['wing_le_fs','wing_le_wl','canard_le_fs','canard_le_wl',
         'winglet_height','winglet_root_chord','winglet_tip_chord','fuselage_length']
for a in attrs:
    v = getattr(g, a)
    assert isinstance(v, (int, float))
    print(f'  {a} = {v}')
print('All VSP geometry params present and numeric')
"
```

Output:
```
  wing_le_fs = 133.0
  wing_le_wl = 0.0
  canard_le_fs = 36.0
  canard_le_wl = 12.0
  winglet_height = 16.0
  winglet_root_chord = 20.0
  winglet_tip_chord = 12.0
  fuselage_length = 214.0
All VSP geometry params present and numeric
```

All 191 existing tests continue to pass.

## Decisions Made

1. **Property aliases vs. duplicate fields:** `wing_le_fs` and `canard_le_fs` are `@property` aliases to existing `fs_wing_le` / `fs_canard_le` fields. This avoids data duplication while satisfying the `export_native_vsp3()` interface. The properties document their OpenVSP usage clearly.

2. **Install script Python 3.13 requirement:** OpenVSP 3.48.2 ships Python 3.13 `.so` bindings. The MacBook runs Python 3.14 by default, so the script requires `python3.13` to be installed separately (`brew install python@3.13`). This is documented clearly in the script.

3. **`.pth` file approach:** Installing a `.pth` file in site-packages is the simplest approach that survives shell restarts without requiring `PYTHONPATH` exports in `.zshrc`. The script documents the limitation for venv switching.

## Deviations from Plan

None - plan executed exactly as written.

## Self-Check

- [x] `config/aircraft_config.py` modified with 6 new fields + 2 properties
- [x] `scripts/install_openvsp.sh` created and executable
- [x] `requirements.txt` updated
- [x] Commit 843d5c1 exists
- [x] All 191 tests pass

## Self-Check: PASSED
