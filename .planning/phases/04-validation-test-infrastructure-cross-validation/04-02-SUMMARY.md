---
phase: 04-validation-test-infrastructure-cross-validation
plan: "02"
subsystem: testing
tags: [vspaero, openvsp, surrogate, cross-validation, aerodynamics, vlm]

# Dependency graph
requires:
  - phase: 03-openvsp-native-integration
    provides: VSPIntegration._run_native_sweep() + real VSPAERO VLM solver working
  - phase: 04-01
    provides: Phase 4 test infrastructure + vspaero_native_polars.json initial structure
provides:
  - Real VSPAERO VLM polar data (vspaero_native_polars.json, no mock)
  - Surrogate cross-validation discrepancy table (surrogate_cross_validation.json)
  - Schema test suite for both JSON files
  - Quantitative CL/CD/CM baseline: surrogate vs native agreement documented
affects: [05-calibration, phase-5-surrogate-tuning]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "generate_cross_validation.py: standalone regeneration script with CadQuery mock preamble"
    - "Schema tests load JSON fixtures directly — no OpenVSP import, no computations"
    - "Tests use polar_output=tmp_path arg for isolation; never write to real data dir"

key-files:
  created:
    - scripts/generate_cross_validation.py
    - data/validation/surrogate_cross_validation.json
    - tests/test_cross_validation.py
  modified:
    - data/validation/vspaero_native_polars.json
    - tests/test_vsp_native.py

key-decisions:
  - "Cross-validation is measure-only: no pass/fail thresholds on CL/CD/CM discrepancy values; Phase 5 decides calibration"
  - "generate_cross_validation.py uses same 19-point alpha array (-4 to 14 deg) for both native and surrogate to ensure valid comparison"
  - "CadQuery mock (sys.modules.setdefault) placed before all core imports in generate_cross_validation.py for OpenVSP-only environments"
  - "test_vsp_native.py tests 2+3 fixed to pass polar_output=tmp_path to avoid overwriting real data file with mock vsp_version"

patterns-established:
  - "Standalone data-generation scripts: mock CadQuery at top, REPO_ROOT path setup, explicit output confirmation prints"
  - "Cross-validation schema tests: load JSON fixtures by absolute path, no physics imports, verify structure not accuracy"
  - "Test isolation: _run_native_sweep() callers must always pass polar_output=tmp_path when using mock vsp"

requirements-completed: [VSP-03]

# Metrics
duration: 9min
completed: 2026-03-13
---

# Phase 4 Plan 02: Surrogate Cross-Validation Summary

**Real VSPAERO VLM polars regenerated (OpenVSP 3.48.2), surrogate cross-validation discrepancy table produced: CL RMS delta 0.680, CD RMS delta 0.027, CM RMS delta 3.956 — quantitative baseline for Phase 5 calibration priorities**

## Performance

- **Duration:** 9 min
- **Started:** 2026-03-13T11:13:27Z
- **Completed:** 2026-03-13T11:22:30Z
- **Tasks:** 2
- **Files modified:** 5

## Accomplishments
- Replaced mock VSPAERO data with real OpenVSP 3.48.2 VLM output: 19 alpha points (-4 to 14 deg), CL range [-0.774, 2.390]
- Produced `surrogate_cross_validation.json` discrepancy table: per-point CL/CD/CM native-vs-surrogate deltas with mean/RMS/max stats
- Added 4 schema tests + 1 documentation marker confirming measure-only intent; full suite 207 passed
- Fixed pre-existing test isolation bug: mock tests in `test_vsp_native.py` were overwriting the real data file with mock version strings

## Task Commits

Each task was committed atomically:

1. **Task 1: Create cross-validation script and run it** - `81cf888` (feat)
2. **Task 2: Schema tests + test isolation bug fix** - `449319b` (feat)

**Plan metadata:** (created next)

## Files Created/Modified
- `scripts/generate_cross_validation.py` - Regenerates native polars via VSPAERO; produces surrogate discrepancy JSON; CadQuery-mock-safe; runnable standalone
- `data/validation/vspaero_native_polars.json` - Updated: vsp_version="OpenVSP 3.48.2", 19 real VLM polar points
- `data/validation/surrogate_cross_validation.json` - New: 19-point CL/CD/CM discrepancy table with summary stats
- `tests/test_cross_validation.py` - New: 4 schema validation tests (not-mock guard, 19-point check, JSON structure, alpha array match) + documentation skip
- `tests/test_vsp_native.py` - Bug fix: tests 2+3 now pass `polar_output=tmp_path` to prevent mock data from overwriting real data file

## Decisions Made
- Cross-validation is measure-only: discrepancy data documents gaps, no thresholds enforced. Phase 5 will use CL RMS delta 0.680 and CM RMS delta 3.956 to prioritize calibration.
- Both native and surrogate evaluated at the same 19-point alpha array for a valid comparison (plan requirement honored).
- CadQuery mock placed at top of script before any core imports — required for environments with OpenVSP but not CadQuery installed.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Fixed mock-data pollution from test_vsp_native.py tests 2 and 3**
- **Found during:** Task 2 (schema test creation and verification)
- **Issue:** `test_native_return_schema_with_mock_vsp` and `test_native_alpha_sweep_range` called `bridge._run_native_sweep((-4, 14, 19))` without `polar_output`. Since OpenVSP IS installed on this machine, these tests ran (not skipped) and wrote mock vsp_version `3.48.2-mock` to the real `data/validation/vspaero_native_polars.json`, causing `test_native_polars_not_mock` to fail.
- **Fix:** Added `polar_output=tmp_path / "native_polars.json"` to both test calls for proper isolation.
- **Files modified:** `tests/test_vsp_native.py`
- **Verification:** All 207 tests pass; mock tests no longer pollute the data file; cross-validation tests pass reliably.
- **Committed in:** `449319b` (Task 2 commit)

---

**Total deviations:** 1 auto-fixed (Rule 1 - bug)
**Impact on plan:** Fix was necessary for test_cross_validation.py reliability. No scope creep.

## Issues Encountered
- After the first Task 1 commit, `python3 -m pytest tests/test_cross_validation.py` showed `test_native_polars_not_mock` failing because another pytest run had executed the mock-injected `_run_native_sweep()` and overwritten the file. Root cause identified as the test isolation bug documented above.

## User Setup Required
None - no external service configuration required. Real VSPAERO requires OpenVSP 3.48.2 Python bindings (already installed).

## Next Phase Readiness
- Phase 5 calibration has quantitative discrepancy data to guide priorities:
  - CL: mean_delta=0.427, rms_delta=0.680, max_abs_delta=1.293 (surrogate underestimates at high alpha)
  - CD: mean_delta=0.006, rms_delta=0.027, max_abs_delta=0.067 (reasonable agreement)
  - CM: mean_delta=-2.546, rms_delta=3.956, max_abs_delta=7.454 (large discrepancy — primary calibration target)
- `vspaero_native_polars.json` now contains real VLM data that can be used as ground truth for calibration
- Schema tests protect against mock-data regression

---
*Phase: 04-validation-test-infrastructure-cross-validation*
*Completed: 2026-03-13*
