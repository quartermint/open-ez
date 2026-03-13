---
phase: 04-validation-test-infrastructure-cross-validation
plan: "01"
subsystem: testing
tags: [pytest, xfail, validation, aerodynamics, stall-speed, neutral-point, cg-envelope, airfoil, wind-tunnel]

# Dependency graph
requires:
  - phase: 01-reference-data-datum-resolution
    provides: reference_data.json with published Long-EZ specs and airfoil wind tunnel data
  - phase: 01-reference-data-datum-resolution
    provides: config.geometry.to_published_datum() for FS coordinate translation
  - phase: 02-structural-fea-dbox
    provides: core/analysis.py PhysicsEngine.calculate_cg_envelope()
provides:
  - Precision validation test file tests/test_precision_validation.py (10 test functions)
  - VAL-01: NP, CG fwd, CG aft tests at 2"/1" tolerance, xfail documenting exact deltas
  - VAL-02: Airfoil CLmax and alpha_0L verified against wind tunnel reference data
  - VAL-04: Stall speed first-principles computation, gross weight exact match
  - Cross-phase TDD anchor — Phase 5 calibration tests must pass these
affects:
  - 04-02 (next plan in phase): surrogate cross-validation
  - 05-geometry-calibration: calibration targets are encoded in these xfail tests

# Tech tracking
tech-stack:
  added: []
  patterns:
    - xfail(strict=False) for known failures documenting exact deltas — cross-phase TDD
    - _load_ref_data() helper pattern for reading reference_data.json in tests
    - Published-area convention in stall speed formula (94.2 + 15.6 sqft, NOT config areas)

key-files:
  created:
    - tests/test_precision_validation.py
  modified: []

key-decisions:
  - "VAL-01 NP delta confirmed at ~5.79\" (computed 113.79 vs reference 108.0 published); xfail encodes this for Phase 5"
  - "VAL-04 stall speed XPASS: first-principles with published areas gives ~57 KTAS vs 56 KTAS (1.8% — within 5% tolerance)"
  - "strict=False on all xfail decorators: test suite must never hard-fail on known calibration issues"
  - "VAL-02 tests have no xfail: config airfoil params were set from wind tunnel data, exact match expected"

patterns-established:
  - "Cross-phase TDD: xfail(strict=False) with detailed reason string documents exact delta for next-phase calibration"
  - "Published reference areas (RAF CP-31) used in stall speed, NOT config computed areas — area convention difference is documented"

requirements-completed: [VAL-01, VAL-02, VAL-04]

# Metrics
duration: 4min
completed: 2026-03-13
---

# Phase 4 Plan 01: Precision Validation Tests Summary

**10 precision validation tests encoding Phase 4 tolerance targets: xfail for known 5.79" NP datum delta (VAL-01), PASS for airfoil wind tunnel match (VAL-02), XPASS for first-principles stall speed within 5% (VAL-04)**

## Performance

- **Duration:** ~4 min
- **Started:** 2026-03-13T11:12:00Z
- **Completed:** 2026-03-13T11:16:40Z
- **Tasks:** 1
- **Files modified:** 1

## Accomplishments

- Created tests/test_precision_validation.py with 10 test functions (377 lines) covering VAL-01, VAL-02, and VAL-04
- VAL-01 stability tests (NP, CG fwd, CG aft) run as XFAIL documenting exact deltas — computed NP is 113.79" published vs reference 108.0" (5.79" delta), serving as calibration target for Phase 5
- VAL-02 airfoil tests all PASS — Roncz R1145MS (CLmax 1.35, alpha_0L -3.0) and Eppler 1230 (CLmax 1.45, alpha_0L -2.0) match wind tunnel data; Cm0 schema check passes
- VAL-04 gross weight PASSES exactly (1425 lb); stall speed XPASS at ~57 KTAS (first-principles with published areas, 1.8% delta within 5% tolerance)
- Full suite: 203 passed, 1 skipped, 3 xfailed, 1 xpassed, 0 failures (was 197 + 6 new passing)

## Task Commits

Each task was committed atomically:

1. **Task 1: Create precision validation test file for VAL-01, VAL-02, VAL-04** - `975d466` (test)

**Plan metadata:** (docs commit follows this summary)

## Files Created/Modified

- `tests/test_precision_validation.py` - 10 precision validation tests for VAL-01 (stability), VAL-02 (airfoil wind tunnel), VAL-04 (performance); 377 lines

## Decisions Made

- xfail(strict=False) used on all three VAL-01 tests and stall speed — `strict=False` means XPASS is also acceptable, which is what happened for stall speed
- VAL-04 stall speed XPASS confirmed: first-principles formula with published areas (94.2 + 15.6 = 109.8 sqft), 1425 lb, canard CLmax 1.35, sea-level density gives ~57 KTAS vs published 56 KTAS (1.8% delta, within 5% tolerance)
- No xfail on VAL-02 tests — config airfoil parameters were set directly from wind tunnel reference data so exact match is guaranteed

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None. The stall speed test unexpectedly passed (XPASS rather than XFAIL) — this is acceptable behavior with strict=False and confirms the first-principles computation is within 5% of published data.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Precision validation test file is in place as the Phase 4/5 TDD anchor
- VAL-01 xfail tests document the exact calibration target for Phase 5 (5.79" NP delta, CG envelope shifts)
- Phase 4 Plan 02 can now proceed: surrogate cross-validation (VAL-03)
- Phase 5 geometry calibration goal: make test_np_precision_2inch(), test_cg_fwd_limit_precision(), test_cg_aft_limit_precision() PASS by correcting fs_wing_le datum

---
*Phase: 04-validation-test-infrastructure-cross-validation*
*Completed: 2026-03-13*
