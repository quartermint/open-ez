---
phase: 05-calibration-accuracy-report
plan: "02"
subsystem: validation
tags: [accuracy-report, traceability, grading, pytest, tdd]
dependency_graph:
  requires: [05-01]
  provides: [accuracy_report_json, accuracy_report_tests]
  affects: [06-regression-lock]
tech_stack:
  added: []
  patterns: [tdd-red-green, grade-banding, source-traceability]
key_files:
  created:
    - scripts/generate_accuracy_report.py
    - data/validation/accuracy_report.json
    - tests/test_accuracy_report.py
    - tests/test_accuracy_report_unit.py
  modified:
    - main.py
decisions:
  - "static_margin already stored in percent in StabilityMetrics (no *100 needed — field is margin*100 at line 367)"
  - "Empty weight FAIL is expected — config structural weights are partial (excludes avionics, fairings, paint); documented in convention_note"
  - "Wing area FAIL is expected — full trapezoidal (110 sqft) vs RAF semi-panel convention (94.2 sqft); documented in convention_note"
  - "Static margin FAIL is expected — calibrated NP+CG aft gives 1.9% actual vs 12% derived reference; reference_data.json note documents this"
  - "Unit tests placed in test_accuracy_report_unit.py (core logic) and test_accuracy_report.py (schema/integration)"
metrics:
  duration: "6m"
  completed_date: "2026-03-13"
  tasks_completed: 2
  files_created: 4
  files_modified: 1
---

# Phase 5 Plan 02: Accuracy Report Generator Summary

## One-liner

Per-metric PASS/MARGINAL/FAIL grading against external reference_data.json with physics_baseline.json traceability enforcement and schema validation tests.

## What Was Built

### scripts/generate_accuracy_report.py

Standalone accuracy report generator (330 lines) following the generate_cross_validation.py pattern:

- `grade_metric(computed, reference, tolerance_abs, tolerance_pct)` — returns `(grade, error_abs, error_pct)` with PASS/MARGINAL/FAIL/UNGRADED banding (PASS: error <= tol, MARGINAL: tol < error <= 2*tol, FAIL: error > 2*tol)
- `validate_sources(report, ref_data)` — raises ValueError on any source containing "physics_baseline.json" or not in the valid reference_data.json key set
- `load_vspaero_provenance(data_dir)` — loads vsp_version, run_timestamp, solver_settings from vspaero_native_polars.json
- `collect_metrics(ref_data, config_module, engine)` — grades 12 validated metrics across 4 categories
- `build_report(metrics, vspaero_provenance)` — assembles final report dict with metadata, summary, metrics
- `generate_accuracy_report(output_path)` — main entry callable from main.py

### data/validation/accuracy_report.json

Machine-readable accuracy report with 12 graded metrics:

| Category | Metric | Grade | Error |
|----------|--------|-------|-------|
| Stability | neutral_point_fs | PASS | 0.0007" |
| Stability | cg_range_fwd_fs | PASS | 0.001" |
| Stability | cg_range_aft_fs | PASS | 0.0006" |
| Stability | static_margin_pct | FAIL | 10.09% (known — see note) |
| Performance | stall_speed_ktas | PASS | 2.71 ktas |
| Performance | max_gross_weight_lb | PASS | 0.0 lb |
| Performance | empty_weight_lb | FAIL | 210 lb (partial weight model) |
| Airfoil | canard_clmax | PASS | 0.000 |
| Airfoil | wing_clmax | PASS | 0.000 |
| Airfoil | canard_alpha_0l_deg | PASS | 0.000 |
| Airfoil | wing_alpha_0l_deg | PASS | 0.000 |
| Geometry | wing_area_sqft | FAIL | 15.8 sqft (convention difference) |

Summary: 9 PASS, 0 MARGINAL, 3 FAIL, 0 UNGRADED. All 3 FAILs are documented expected differences, not physics errors.

### tests/test_accuracy_report.py

6-test schema and traceability validation suite:
- `test_report_schema` — required keys, types, metric field completeness
- `test_source_traceability` — hard block on physics_baseline.json; all sources valid
- `test_summary_counts` — PASS/MARGINAL/FAIL/UNGRADED counts match metric grades
- `test_calibration_log_schema` — calibration_log.json structure (3 calibration entries)
- `test_all_grades_valid` — every grade in {PASS, MARGINAL, FAIL, UNGRADED}
- `test_report_not_empty` — at least 10 metrics present

### main.py

Added `--accuracy-report` flag (before validate_config() guard so it runs without geometry validation). Delegates to `generate_accuracy_report()`.

## Verification Results

1. `python3 scripts/generate_accuracy_report.py` — runs without error, prints 12-metric table
2. `python3 main.py --accuracy-report` — identical output path
3. `python3 -m pytest tests/test_accuracy_report.py -v` — 6/6 PASS
4. `python3 -m pytest tests/ -x -q` — 228 passed, 2 skipped, 0 failures
5. No physics_baseline.json in any source field (traceability verified)
6. Summary counts match detailed metrics (verified by test and manual check)

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] static_margin already stored as percent**
- **Found during:** Task 1, running the report generator
- **Issue:** Plan specified `stability.static_margin * 100` to convert fraction to percent, but `StabilityMetrics.static_margin` is already stored as `margin * 100.0` (line 367 of core/analysis.py). Multiplying again yielded 191% vs 1.9%
- **Fix:** Removed the `* 100.0` multiplier; use `stability.static_margin` directly
- **Files modified:** `scripts/generate_accuracy_report.py`
- **Commit:** c28c780 (part of Task 1 commit)

## Self-Check: PASSED

All created files exist and both task commits verified:
- FOUND: scripts/generate_accuracy_report.py
- FOUND: data/validation/accuracy_report.json
- FOUND: tests/test_accuracy_report.py
- FOUND: tests/test_accuracy_report_unit.py
- FOUND: c28c780 feat(05-02): create accuracy report generator...
- FOUND: 89ebb5e test(05-02): add schema and traceability tests...
