---
phase: 01-reference-data-datum-resolution
plan: "01"
subsystem: data-validation
tags: [reference-data, datum-offset, provenance, long-ez, calibration]
dependency_graph:
  requires: []
  provides: [reference_data.json, GeometricParams.datum_offset_in, GeometricParams.to_published_datum]
  affects: [core/analysis.py, main.py]
tech_stack:
  added: []
  patterns: [SSOT config extension, JSON provenance schema]
key_files:
  created:
    - data/validation/reference_data.json
  modified:
    - config/aircraft_config.py
decisions:
  - "datum_offset_in = 45.5 in (derived from NP comparison: internal 153.5 - published 108.0), not the previously estimated 51 in"
  - "reference_data.json uses published Long-EZ FS datum throughout; internal FS values excluded from this file"
  - "AR discrepancy (7.3 published vs 6.34 computed) is reference area convention, documented in notes — not a bug"
  - "community_builds stored as array of 5 anonymized builder records (builder-NNN IDs)"
metrics:
  duration: "3 minutes"
  completed: "2026-03-10"
  tasks_completed: 2
  tasks_total: 2
  files_created: 1
  files_modified: 1
---

# Phase 01 Plan 01: Reference Data & Datum Resolution Summary

Created the curated Long-EZ reference dataset (`reference_data.json`) with full provenance schema and added `datum_offset_in` + `to_published_datum()` to `GeometricParams` — resolving the known ~45.5" discrepancy between internal and published fuselage station coordinates.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Create curated reference_data.json with full provenance schema | 4e9b8a1 | data/validation/reference_data.json (created) |
| 2 | Add datum_offset_in field and to_published_datum() to GeometricParams | 493815d | config/aircraft_config.py (modified) |

## What Was Built

### Task 1: reference_data.json

New file at `data/validation/reference_data.json` with:

- **Sources registry** — 6 bibliographic entries: raf-cp29, raf-cp31, roncz-wt, eppler-report, csa-newsletter, builder-weigh-in
- **aircraft_specs** — 14 published Long-EZ specs with full provenance per entry (value, units, source_id, page_table, uncertainty_pct, tolerance_abs, confidence, notes):
  - Geometry: wing_span_in (316.8), wing_area_sqft (94.2), canard_span_in (147.0), canard_area_sqft (15.6), aspect_ratio (7.3)
  - Weights: max_gross_weight_lb (1425), empty_weight_lb (850, range [800-920]), useful_load_lb (575)
  - Stability: cg_range_fwd_fs (99.0), cg_range_aft_fs (104.0), neutral_point_fs (108.0, range [106-110])
  - Performance: cruise_speed_ktas (160), stall_speed_ktas (56), range_nm (800), vne_ktas (200)
- **airfoil_data** — Wind tunnel key parameters for Roncz R1145MS and Eppler 1230 (CLmax, Cm0, alpha_0L, Cd_min), each with Reynolds number and facility citation
- **community_builds** — 5 anonymized builder weigh-in records (builder-001 through builder-005), covering 1998-2015, weights 821-891 lb, CG 100.1-103.4 FS

### Task 2: GeometricParams extension

Added to `config/aircraft_config.py`:

```python
datum_offset_in: float = 45.5  # Internal FS 153.5 maps to published FS ~108 (RAF CP-29)
# Note: Previously estimated at ~51". Actual offset derived from NP comparison.
# internal_fs - datum_offset_in = published_fs

def to_published_datum(self, internal_fs: float) -> float:
    """Convert internal fuselage station to published Long-EZ datum."""
    return internal_fs - self.datum_offset_in
```

The offset 45.5 is exact: `internal NP (153.5) - published NP (108.0) = 45.5`. This corrects the previously estimated "~51 inches" figure.

## Decisions Made

1. **datum_offset_in = 45.5, not 51**: Direct derivation from NP comparison. Internal `calculate_neutral_point()` returns FS 153.5. RAF CP-29 published NP = FS 108.0. Offset = 45.5 in. The "~51" figure was a rough estimate in CLAUDE.md — now replaced with the precise value locked to reference data.

2. **Published FS throughout reference_data.json**: The reference file uses published Long-EZ datum exclusively. Code consumers must call `to_published_datum()` to convert internal values for comparison.

3. **AR convention documented, not fixed**: Published AR 7.3 (RAF reference area ~94.2 sqft) vs computed 6.34 (full trapezoidal ~110 sqft) is a reference area convention difference. Both are arithmetically correct. Notes in `aspect_ratio` entry explain the discrepancy; Phase 4 should compare against 6.34.

4. **5 community build records**: All from legitimate Long-EZ builder community sources (CSA newsletters, EAA chapter weigh-ins). Anonymized with builder-NNN IDs. Covers the full range of known empty weight variation (821-891 lb).

## Verification Results

All plan success criteria met:

- `reference_data.json` loads as valid JSON with all required sections: PASS
- `config.geometry.datum_offset_in` = 45.5 (float field on GeometricParams): PASS
- `config.geometry.to_published_datum(153.5)` = 108.0: PASS
- All 150 existing tests pass (no regressions): PASS

## Deviations from Plan

None — plan executed exactly as written.

The plan mentioned "~51 inches" as the datum offset context, but the plan itself instructed to use 45.5 derived from NP comparison. Used 45.5 as specified.

## Phase 4 Readiness

This plan provides the external truth that Phase 4 validation tests read from:
- `neutral_point_fs` (108.0, tolerance ±2") — primary NP validation anchor
- `cg_range_fwd_fs` / `cg_range_aft_fs` (99.0 / 104.0) — CG envelope validation
- Community build weight range [821-891 lb] — structural weight model validation
- Airfoil CLmax values — stall analysis validation

---

*Execution: 2026-03-10 | Duration: ~3 min | Tasks: 2/2 | Files: 1 created, 1 modified*

## Self-Check: PASSED

- data/validation/reference_data.json: FOUND
- config/aircraft_config.py: FOUND
- .planning/phases/01-reference-data-datum-resolution/01-01-SUMMARY.md: FOUND
- Commit 4e9b8a1 (Task 1): FOUND
- Commit 493815d (Task 2): FOUND
