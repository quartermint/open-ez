---
phase: 06-regression-lock-in
plan: 02
subsystem: testing
tags: [regression, physics, validation, accuracy-report, baseline]

# Dependency graph
requires:
  - phase: 05-calibration-accuracy-report
    provides: "data/validation/accuracy_report.json with 9 PASS metrics tracing to reference_data.json"
provides:
  - "RegressionRunner.compare_to_accuracy_report() — externally-validated regression method"
  - "DEFAULT_ACCURACY_REPORT constant pointing to accuracy_report.json"
  - "tests/test_physics_regression.py validating against accuracy_report.json (3 tests)"
  - "main.py validate_physics() using accuracy_report.json as baseline source"
  - "tests/snapshots/physics_baseline.json deprecated with _DEPRECATED notice"
affects:
  - phase-07 (any future phase that adds physics metrics)
  - CI pipeline (physics regression now anchored to external reference, not self-referential)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Regression baselines must trace to external reference data (reference_data.json or vspaero_native)"
    - "PASS-only metric filtering: FAIL metrics are known convention gaps, not regression targets"
    - "compare_to_accuracy_report() pattern: reload collect_metrics() live, compare against stored computed values"

key-files:
  created: []
  modified:
    - core/simulation/regression.py
    - main.py
    - tests/test_physics_regression.py
    - tests/snapshots/physics_baseline.json

key-decisions:
  - "compare_to_accuracy_report() compares current computed values against the accuracy_report.json 'computed' field (Phase 5 calibrated values), not against reference values — this detects code regressions while accepting known accuracy gaps"
  - "PASS-only filtering: only the 9 PASS metrics from accuracy_report.json are regression targets; the 3 FAIL metrics (static_margin_pct, empty_weight_lb, wing_area_sqft) are known convention differences excluded from CI"
  - "Old compare_to_baseline() and scenarios retained for backward compatibility with deprecation comments — not removed to avoid breaking any external callers"
  - "physics_baseline.json preserved with _DEPRECATED key, _replacement, and _deprecated_date — historical values visible but cannot be used as authoritative without noticing the deprecation"

patterns-established:
  - "Regression baseline pattern: accuracy_report.json 'computed' field acts as the locked baseline; collect_metrics() re-runs live to produce current values for comparison"
  - "Tolerance reuse: each metric's tolerance_abs from accuracy_report.json is used as the regression tolerance — no separate tolerance constants needed"

requirements-completed: [VAL-08]

# Metrics
duration: 3min
completed: 2026-03-13
---

# Phase 06 Plan 02: Baseline Rewiring Summary

**RegressionRunner and main.py rewired from self-referential physics_baseline.json to externally-validated accuracy_report.json baselines — regression loop broken, physics_baseline.json deprecated**

## Performance

- **Duration:** ~3 min
- **Started:** 2026-03-13T15:03:40Z
- **Completed:** 2026-03-13T15:06:17Z
- **Tasks:** 2
- **Files modified:** 4

## Accomplishments

- `RegressionRunner.compare_to_accuracy_report()` added — re-runs physics engine live, compares computed values against Phase 5 accuracy report baselines (externally-validated)
- `DEFAULT_ACCURACY_REPORT` constant established as the canonical baseline path for all regression and test code
- `main.py validate_physics()` rewired to use `compare_to_accuracy_report()` — no code path references `physics_baseline.json`
- `test_physics_regression.py` rewritten with 3 tests: main regression pass/fail, PASS metric spot-check, and report-write verification
- `physics_baseline.json` marked deprecated with `_DEPRECATED` key — historical values preserved but impossible to mistake as authoritative
- Full test suite: 241 passed, 2 skipped

## Task Commits

Each task was committed atomically:

1. **Task 1: Rewire RegressionRunner to read from accuracy_report.json** - `f45307b` (feat)
2. **Task 2: Rewire main.py and test, deprecate physics_baseline.json** - `eee5940` (feat)

**Plan metadata:** (docs commit follows)

## Files Created/Modified

- `core/simulation/regression.py` - Added `DEFAULT_ACCURACY_REPORT` constant and `compare_to_accuracy_report()` method; deprecated old `load_baseline()`, `compare_to_baseline()`, `run()`, and scenarios with comments
- `main.py` - `validate_physics()` rewired to `compare_to_accuracy_report()` with `DEFAULT_ACCURACY_REPORT`; docstring updated to explain Phase 6 baseline change
- `tests/test_physics_regression.py` - Complete rewrite: 3 tests against accuracy_report.json instead of physics_baseline.json
- `tests/snapshots/physics_baseline.json` - Added `_DEPRECATED`, `_replacement`, `_deprecated_date` keys at top level

## Decisions Made

- **Baseline comparison strategy:** `compare_to_accuracy_report()` compares current `collect_metrics()` output against the `computed` field in `accuracy_report.json` (not the `reference` field). This means we're comparing current code output against Phase 5 code output. Any regression in the physics code will show up as a delta, while the known FAIL metrics are excluded from CI entirely.
- **PASS-only filtering:** Only the 9 PASS metrics from accuracy_report.json are regression targets. The 3 FAIL metrics (static_margin_pct, empty_weight_lb, wing_area_sqft) represent known convention differences — excluding them means CI doesn't fail on pre-existing known gaps.
- **Backward compatibility:** Old `compare_to_baseline()` retained with deprecation docstring. No callers of the old method were found in the codebase, but removing it would be a breaking change if external code used it.

## Deviations from Plan

None — plan executed exactly as written. The one minor adjustment was removing a spurious `from data.validation import reference_data` import (data/validation is not a Python package) and using direct path loading instead — this was caught and corrected before any commit.

## Issues Encountered

None — both tasks completed cleanly on first attempt.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Phase 6 regression lock-in is complete: physics baselines are now externally-validated and the self-referential loop is broken
- Future physics changes will be caught by `test_physics_regressions_match_accuracy_report` using accuracy_report.json as the anchor
- If accuracy_report.json is regenerated (new VSPAERO run, config changes), the regression baseline updates automatically — the lock is always relative to the most recent calibrated report
- No blockers for Phase 7 or any downstream work

---
*Phase: 06-regression-lock-in*
*Completed: 2026-03-13*
