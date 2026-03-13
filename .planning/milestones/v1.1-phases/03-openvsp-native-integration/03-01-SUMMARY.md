---
phase: 03-openvsp-native-integration
plan: "01"
subsystem: infra
tags: [openvsp, geometry, config, installation, python]
dependency_graph:
  requires: []
  provides: [VSP-01]
  affects: [core/openvsp_runner.py, 03-02, 03-03]
tech_stack:
  added: [openvsp 3.48.2, openvsp_config 0.1.0, degen_geom 0.0.1, vsp_airfoils 0.0.0, utilities 0.1.0]
  patterns: [SSOT config extension, computed property aliases, pip install from app bundle packages]
key_files:
  created:
    - scripts/install_openvsp.sh
  modified:
    - config/aircraft_config.py
    - requirements.txt
key-decisions:
  - "wing_le_fs and canard_le_fs implemented as @property aliases to fs_wing_le/fs_canard_le — no data duplication, SSOT preserved"
  - "OpenVSP 3.48.2 Python bindings installed via pip from bundled packages in macOS ARM64 ZIP (app bundle is the only macOS distribution)"
  - "Python 3.13 required for OpenVSP .so bindings (system Python 3.14 is too new for compiled extensions)"
  - "fuselage_length=214.0 matches fs_tail (tail cone terminus)"
  - "wing_le_wl=0.0 (wing at longeron level), canard_le_wl=12.0 (12 inch above longerons)"
metrics:
  duration: "25m"
  completed_date: "2026-03-10"
  tasks_completed: 2
  tasks_total: 2
  files_modified: 3
requirements-completed: [VSP-01]
---

# Phase 3 Plan 01: VSP Geometry Parameters and Install Script Summary

**GeometricParams extended with 8 OpenVSP-required geometry fields (6 new + 2 property aliases); OpenVSP 3.48.2 Python bindings installed and verified via pip from macOS ARM64 app bundle**

## Performance

- **Duration:** ~25 min (including human-action checkpoint for OpenVSP install)
- **Started:** 2026-03-10T15:26:00Z (estimated)
- **Completed:** 2026-03-10T15:42:00Z
- **Tasks:** 2 (1 auto + 1 human-action checkpoint)
- **Files modified:** 3

## Accomplishments

- Extended `GeometricParams` with 6 new fields (`wing_le_wl`, `canard_le_wl`, `winglet_height`, `winglet_root_chord`, `winglet_tip_chord`, `fuselage_length`) using physically-derived Long-EZ values
- Added `wing_le_fs` and `canard_le_fs` as `@property` aliases preserving SSOT while satisfying the `export_native_vsp3()` interface
- Created `scripts/install_openvsp.sh` documenting the macOS ARM64 pip-based install path
- OpenVSP 3.48.2 verified importable: `python3.13 -c "import openvsp as vsp; print(vsp.GetVSPVersion())"` returns `OpenVSP 3.48.2`
- 191 existing tests pass — no regressions

## Task Commits

1. **Task 1: Add VSP geometry parameters to GeometricParams and create install script** - `843d5c1` (feat) + `a28639e` (fix: correct URL and pip install)
2. **Task 2: Install OpenVSP Python bindings on MacBook** — Human-action checkpoint; completed by user. Verified via `python3.13 -c "import openvsp as vsp; print(vsp.GetVSPVersion())"`.

**Plan metadata:** `1d41e38` (docs: complete plan)

## Files Created/Modified

- `config/aircraft_config.py` — 6 new GeometricParams fields + 2 @property FS aliases under new `# === OPENVSP GEOMETRY ===` section
- `scripts/install_openvsp.sh` — macOS ARM64 install automation: downloads OpenVSP 3.48.2 ZIP, extracts, pip-installs bundled packages into Python 3.13 venv, smoke-tests import
- `requirements.txt` — Updated comment noting openvsp installed via scripts/install_openvsp.sh

## GeometricParams Extension Detail

New fields under `# === OPENVSP GEOMETRY (positions for 3D model) ===`:

| Field | Value | Notes |
|-------|-------|-------|
| `wing_le_wl` | `0.0` in | Wing LE waterline — at longeron level |
| `canard_le_wl` | `12.0` in | Canard ~12" above longerons |
| `winglet_height` | `16.0` in | Winglet vertical span (Rutan Ch.19) |
| `winglet_root_chord` | `20.0` in | At wing tip junction |
| `winglet_tip_chord` | `12.0` in | Winglet tip |
| `fuselage_length` | `214.0` in | = fs_tail (nose to terminus) |

New computed properties (no data duplication):

| Property | Returns | Purpose |
|----------|---------|---------|
| `wing_le_fs` | `fs_wing_le` (133.0) | OpenVSP X-position for wing |
| `canard_le_fs` | `fs_canard_le` (36.0) | OpenVSP X-position for canard |

## Decisions Made

1. `@property` aliases for `wing_le_fs`/`canard_le_fs` preserve SSOT — no forking of FS station data
2. OpenVSP 3.48.2 bundle ships pip-installable wheel packages; pip install is cleaner than a .pth file
3. Python 3.13 isolated venv required — system Python 3.14 incompatible with compiled .so extensions
4. OpenVSP installed to `~/.local/openvsp/OpenVSP-3.48.2-MacOS/` — user-local, no sudo required

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Fixed incorrect install URL and approach in install script**
- **Found during:** Task 1 post-commit verification
- **Issue:** Initial script used a guessed direct-download URL (invalid for openvsp.org) and used .pth file strategy; actual 3.48.2 bundle ships pip-installable packages
- **Fix:** Corrected script to pip-install bundled packages from extracted ZIP at `~/.local/openvsp/OpenVSP-3.48.2-MacOS/python/`; smoke test passes
- **Files modified:** `scripts/install_openvsp.sh`
- **Verification:** `python3.13 -c "import openvsp as vsp; print(vsp.GetVSPVersion())"` returns `OpenVSP 3.48.2`
- **Committed in:** `a28639e` (fix commit)

---

**Total deviations:** 1 auto-fixed (Rule 1 — bug)
**Impact on plan:** Fix was necessary for the install script to work. No scope creep.

## Issues Encountered

- OpenVSP 3.48.2 macOS package ships pip-installable wheel packages rather than requiring a .pth file; actual structure was cleaner than planned
- Python 3.14 (system default) incompatible with compiled .so extensions; Python 3.13 required and available via Homebrew

## User Setup Required

**OpenVSP Python bindings require a one-time manual install** (Task 2 was a human-action checkpoint). Now COMPLETE.

Install location: `/Users/ryanstern/.local/openvsp/OpenVSP-3.48.2-MacOS/`

Verification: `python3.13 -c "import openvsp as vsp; print(vsp.GetVSPVersion())"` = `OpenVSP 3.48.2`

For other developers, follow: `bash scripts/install_openvsp.sh`

## Next Phase Readiness

- All 8 `GeometricParams` fields required by `export_native_vsp3()` are populated with physically-derived Long-EZ values
- `import openvsp` succeeds on the MacBook — Plan 02 (native VSP3 geometry builder) can proceed
- No blockers for Phase 03-02

## Self-Check

- `config/aircraft_config.py` — all 8 VSP attrs verified present and numeric
- `scripts/install_openvsp.sh` — exists and executable
- OpenVSP import: `python3.13 -c "import openvsp as vsp; print(vsp.GetVSPVersion())"` = `OpenVSP 3.48.2`
- Commits `843d5c1` and `a28639e` exist in git log
- 191 tests pass

## Self-Check: PASSED

---
*Phase: 03-openvsp-native-integration*
*Completed: 2026-03-10*
