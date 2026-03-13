---
phase: 06-regression-lock-in
verified: 2026-03-13T18:00:00Z
status: passed
score: 9/9 must-haves verified
re_verification: false
---

# Phase 6: Regression Lock-In Verification Report

**Phase Goal:** Self-referential baseline loop is broken — regression tests lock to externally-validated values and `RegressionRunner` validates against external truth, not values it generated itself
**Verified:** 2026-03-13
**Status:** passed
**Re-verification:** No — initial verification

---

## Goal Achievement

### Observable Truths

| #  | Truth | Status | Evidence |
|----|-------|--------|----------|
| 1  | Precision regression tests exist for each PASS metric from Phase 5 accuracy report | VERIFIED | `tests/test_regression_lock.py` has 11 tests covering all 9 PASS metrics; 11/11 pass |
| 2  | Tests are locked to Phase 5 calibrated values from accuracy_report.json, not physics_baseline.json | VERIFIED | `LOCKED_*` constants at module level with source citations; traceability test `test_regression_values_match_accuracy_report` confirms sync; `test_regression_baselines_not_self_referential` confirms no data-loading from physics_baseline.json |
| 3  | Tolerances come from reference_data.json physical measurement uncertainty | VERIFIED | Each test loads `reference_data.json` directly and reads `tolerance_abs` for the external truth check; drift tolerances (0.01"/0.5 KTAS) are tighter than reference tolerances |
| 4  | Running pytest catches future physics drift outside calibrated bounds | VERIFIED | All 11 tests pass currently; two-tier assertions enforce both gross-error detection and sub-0.01" drift detection |
| 5  | RegressionRunner reads baselines from accuracy_report.json, not physics_baseline.json | VERIFIED | `compare_to_accuracy_report()` added to `core/simulation/regression.py`; `DEFAULT_ACCURACY_REPORT` constant points to `data/validation/accuracy_report.json` |
| 6  | main.py validate_physics() uses accuracy_report.json as its baseline source | VERIFIED | `main.py` lines 47-55: imports `RegressionRunner, DEFAULT_ACCURACY_REPORT` and calls `compare_to_accuracy_report()` |
| 7  | physics_baseline.json is clearly marked as deprecated and non-authoritative | VERIFIED | `tests/snapshots/physics_baseline.json` has top-level `_DEPRECATED`, `_replacement`, and `_deprecated_date` keys |
| 8  | test_physics_regression.py validates against accuracy_report.json instead of physics_baseline.json | VERIFIED | All 3 tests use `DEFAULT_ACCURACY_REPORT` and `compare_to_accuracy_report()`; 3/3 pass |
| 9  | Full test suite passes with no regressions | VERIFIED | 241 passed, 2 skipped (both skips are pre-existing: documentation marker + OpenVSP not installed) |

**Score:** 9/9 truths verified

---

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `tests/test_regression_lock.py` | Per-metric regression tests locked to Phase 5 calibrated values | VERIFIED | 585 lines (> 120 min), 11 test functions, loads `accuracy_report.json` — all checks pass |
| `core/simulation/regression.py` | RegressionRunner reading from accuracy_report.json | VERIFIED | 293 lines; contains `compare_to_accuracy_report()`, `DEFAULT_ACCURACY_REPORT`, `accuracy_report` string; old methods deprecated |
| `tests/test_physics_regression.py` | Regression test using accuracy_report.json baselines | VERIFIED | 71 lines; 3 tests using `DEFAULT_ACCURACY_REPORT`; all pass |
| `tests/snapshots/physics_baseline.json` | Deprecated baseline file with deprecation notice | VERIFIED | Contains `_DEPRECATED`, `_replacement`, `_deprecated_date` keys at top level |

---

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| `tests/test_regression_lock.py` | `data/validation/accuracy_report.json` | Loads PASS metric computed values as regression baselines | WIRED | Module-level `ACCURACY_REPORT_PATH` constant; `_load_accuracy_report()` function; `test_regression_values_match_accuracy_report` test reads all 9 PASS metrics |
| `tests/test_regression_lock.py` | `data/validation/reference_data.json` | Loads tolerance_abs values for each metric | WIRED | Module-level `REF_DATA_PATH` constant; `_load_ref_data()` function; used in all stability/performance tests for external truth check |
| `tests/test_regression_lock.py` | `core/analysis.py` | Runs PhysicsEngine to get current computed values | WIRED | `from core.analysis import PhysicsEngine` inside each stability test function; `PhysicsEngine().calculate_cg_envelope()` called |
| `core/simulation/regression.py` | `data/validation/accuracy_report.json` | RegressionRunner.load_baseline reads accuracy report metrics | WIRED | `DEFAULT_ACCURACY_REPORT` constant at module level; `compare_to_accuracy_report()` opens and reads the file |
| `main.py` | `data/validation/accuracy_report.json` | validate_physics() baseline path | WIRED | `from core.simulation.regression import RegressionRunner, DEFAULT_ACCURACY_REPORT`; `runner.compare_to_accuracy_report(accuracy_report_path=DEFAULT_ACCURACY_REPORT, ...)` |
| `tests/test_physics_regression.py` | `data/validation/accuracy_report.json` | Test baseline path | WIRED | `from core.simulation.regression import RegressionRunner, DEFAULT_ACCURACY_REPORT`; used in all 3 test functions |

---

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|------------|-------------|--------|----------|
| VAL-07 | 06-01-PLAN.md | Precision regression tests created locked to validated values (not self-referential) | SATISFIED | `tests/test_regression_lock.py`: 11 tests with two-tier assertions, all 9 PASS metrics locked with `LOCKED_*` constants citing accuracy_report.json; traceability test confirms no physics_baseline.json data loading |
| VAL-08 | 06-02-PLAN.md | Self-referential baseline loop broken — RegressionRunner validates against external truth | SATISFIED | `compare_to_accuracy_report()` in `core/simulation/regression.py`; `main.py` uses it exclusively; `test_physics_regression.py` uses it; `physics_baseline.json` deprecated with clear notice |

Both requirements mapped in REQUIREMENTS.md to Phase 6 are SATISFIED. No orphaned requirements detected.

---

### Anti-Patterns Found

No TODO, FIXME, PLACEHOLDER, or empty implementation patterns found in any Phase 6 files. No `return null` / stub patterns. No console.log-only handlers.

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| — | — | None | — | — |

---

### Human Verification Required

None. All success criteria are programmatically verifiable and confirmed:

- File existence: confirmed
- Line counts: confirmed (585/293/71 lines in the three test/regression files)
- Test pass/fail: confirmed (11/11 + 3/3 + full suite 241 passed)
- Wiring to accuracy_report.json: confirmed via grep and code inspection
- physics_baseline.json deprecation: confirmed via JSON key inspection
- No self-referential data sources: confirmed via `test_regression_baselines_not_self_referential`

---

### Gaps Summary

No gaps. All must-haves from both plans are verified. Phase goal achieved.

The core innovation — two-tier assertion pattern combining external truth checks (reference_data.json tolerances) with tight drift detection (0.01" / 0.5 KTAS) — is fully implemented and operational. The self-referential loop that existed in `physics_baseline.json` is definitively broken. Any future code change that shifts NP/CG positions by more than 0.01", changes stall speed by more than 0.5 KTAS, or modifies any of the 9 locked config/physics values will trigger a CI failure traceable to external reference data.

---

_Verified: 2026-03-13T18:00:00Z_
_Verifier: Claude (gsd-verifier)_
