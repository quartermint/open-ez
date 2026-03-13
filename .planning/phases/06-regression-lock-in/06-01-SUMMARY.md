---
phase: 06-regression-lock-in
plan: 01
subsystem: testing
tags: [pytest, regression, physics-validation, accuracy-report, traceability]

# Dependency graph
requires:
  - phase: 05-calibration-accuracy-report
    provides: accuracy_report.json with 9 PASS metrics calibrated against reference_data.json

provides:
  - Precision regression tests locked to Phase 5 calibrated values (11 tests)
  - Two-tier assertion pattern: external truth check + drift detection check
  - Traceability chain: locked constants -> accuracy_report.json -> reference_data.json -> RAF CP-29/wind tunnel

affects:
  - Any future phase that changes physics parameters (geometry, config, analysis algorithms)
  - CI pipeline — these tests will catch any physics drift

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Two-tier regression lock: external truth tolerance (coarse) + drift detection tolerance (tight)"
    - "Module-level LOCKED_* constants with explicit source citations in comments"
    - "Traceability self-test: test file reads its own source to verify no self-referential data"

key-files:
  created:
    - tests/test_regression_lock.py
  modified: []

key-decisions:
  - "Two-tier assertion pattern: reference tolerance catches gross errors, drift tolerance (0.01\"/0.5 KTAS) catches code regressions"
  - "LOCKED_* constants defined at module level with explicit accuracy_report.json source citations — must not be updated without re-calibration"
  - "physics_baseline.json explicitly excluded: traceability test verifies no data loading from self-referential source"
  - "Config value tests (CLmax, alpha_0L, gross weight) use exact equality for drift detection — these are config reads, not computed values"

patterns-established:
  - "Regression lock pattern: lock computed values from an externally-graded report, not from runtime computation"
  - "Traceability test pattern: test reads its own source to verify excluded files are not loaded as data sources"

requirements-completed: [VAL-07]

# Metrics
duration: 2min
completed: 2026-03-13
---

# Phase 6 Plan 01: Regression Lock Summary

**11 precision regression tests locking 9 PASS metrics from Phase 5 accuracy report, using two-tier assertions (RAF CP-29 tolerance + 0.01"/0.5 KTAS drift detection) to break the self-referential physics baseline loop**

## Performance

- **Duration:** 2 min
- **Started:** 2026-03-13T15:03:50Z
- **Completed:** 2026-03-13T15:06:21Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments

- Created `tests/test_regression_lock.py` with 11 test functions covering all 9 PASS metrics from the Phase 5 accuracy report
- Implemented two-tier assertion pattern: external truth check (reference_data.json tolerance) + tight drift detection (0.01" for FS, 0.5 KTAS for speed, exact for config values)
- Added traceability tests that verify locked constants match accuracy_report.json and that physics_baseline.json is never loaded as a data source
- Full suite: 241 passed, 2 skipped (previously 230 + 11 new)

## Task Commits

Each task was committed atomically:

1. **Task 1: Create precision regression tests locked to Phase 5 calibrated values** - `e43cd92` (feat)

**Plan metadata:** (docs commit follows)

## Files Created/Modified

- `/Users/ryanstern/open-ez/tests/test_regression_lock.py` - 11 regression lock tests with two-tier assertions for all 9 PASS metrics from Phase 5 accuracy report

## Decisions Made

- **Two-tier assertion pattern:** Each test makes two assertions — (1) external truth check within reference_data.json tolerance, (2) drift detection within tight tolerance of Phase 5 calibrated value. The tight tolerance (0.01" for FS positions, 0.5 KTAS for speed) catches code regressions that would still pass the broader external tolerance.
- **LOCKED_* constants:** Defined at module level with explicit source citations citing accuracy_report.json generation date (2026-03-13). Module docstring prominently warns not to update without re-calibrating against external data.
- **Config value drift detection:** CLmax, alpha_0L, and gross weight tests use exact equality (`==`) for drift detection since these are direct config reads. Any accidental config change will be caught immediately.
- **physics_baseline.json excluded:** The traceability test `test_regression_baselines_not_self_referential` reads its own source and asserts no data-loading operations reference physics_baseline.json, enforcing the anti-self-referential constraint going forward.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None - all 11 tests passed on first run without needing any fixes.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Regression lock in place: any future physics drift outside 0.01" (stability) or 0.5 KTAS (speed) will fail CI
- Ready for Phase 6 Plan 02 (next plan in regression lock-in phase)
- If future calibration changes calibrated values, the LOCKED_* constants must be updated alongside re-running Phase 5 calibration

---
*Phase: 06-regression-lock-in*
*Completed: 2026-03-13*

## Self-Check: PASSED

- tests/test_regression_lock.py: FOUND
- .planning/phases/06-regression-lock-in/06-01-SUMMARY.md: FOUND
- Commit e43cd92: FOUND
- Commit b41dcf1: FOUND
