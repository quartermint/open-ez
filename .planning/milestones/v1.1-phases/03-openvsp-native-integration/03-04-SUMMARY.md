---
phase: 03-openvsp-native-integration
plan: "04"
subsystem: infra
tags: [python, cadquery, imports, main-entrypoint]

# Dependency graph
requires:
  - phase: 03-openvsp-native-integration
    provides: native VSPAERO integration wired into main.py pipeline
provides:
  - main.py runnable without CadQuery for --analysis, --validate, --summary, --validate-physics modes
  - CadQuery imports deferred to function scope in geometry modes only
affects:
  - any future main.py modifications
  - CI environments without CadQuery

# Tech tracking
tech-stack:
  added: []
  patterns: [deferred-imports, function-scoped-imports]

key-files:
  created: []
  modified:
    - main.py
    - tests/test_integration.py

key-decisions:
  - "CadQuery imports deferred to function scope in generate_manufacturing(), generate_canard(), generate_wing() — analysis modes work in any Python env"
  - "test_integration.py patches updated from main.CanardGenerator to core.structures.CanardGenerator to match new import location"

patterns-established:
  - "Function-scoped imports pattern: geometry-generating functions import CadQuery modules locally; analysis/validation functions keep no CadQuery dependency"

requirements-completed: []

# Metrics
duration: 2min
completed: 2026-03-10
---

# Phase 03 Plan 04: Defer CadQuery Imports in main.py Summary

**CadQuery imports moved to function scope so --analysis, --validate, --summary run in any Python environment without CadQuery installed**

## Performance

- **Duration:** 2 min
- **Started:** 2026-03-10T19:47:32Z
- **Completed:** 2026-03-10T19:49:31Z
- **Tasks:** 1
- **Files modified:** 2

## Accomplishments
- Removed top-level `from core.structures import ...` and `from core.manufacturing import ...` from main.py module level
- Added local imports at the top of `generate_manufacturing()`, `generate_canard()`, and `generate_wing()` functions
- `python3 main.py --analysis`, `--validate`, `--summary` all work without CadQuery
- All 196 tests pass

## Task Commits

Each task was committed atomically:

1. **Task 1: Move CadQuery-dependent imports into functions** - `570dde4` (feat)

**Plan metadata:** (pending docs commit)

## Files Created/Modified
- `/Users/ryanstern/open-ez/main.py` - Removed module-level CadQuery imports, added function-scoped imports in 3 geometry functions
- `/Users/ryanstern/open-ez/tests/test_integration.py` - Updated mock patches from `main.CanardGenerator` to `core.structures.CanardGenerator`

## Decisions Made
- Patch targets in test_integration.py updated to `core.structures.CanardGenerator` because `unittest.mock.patch` must target where the name is looked up at call time — after the import is deferred into the function, the name lives in `core.structures`, not `main`

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Fixed test_integration.py mock patch targets**
- **Found during:** Task 1 (Move CadQuery-dependent imports into functions)
- **Issue:** Two tests patched `main.CanardGenerator` which no longer exists as a module-level attribute after deferring the import. AttributeError: `<module 'main'> does not have the attribute 'CanardGenerator'`
- **Fix:** Changed both patch targets from `main.CanardGenerator` to `core.structures.CanardGenerator`
- **Files modified:** tests/test_integration.py
- **Verification:** `python3 -m pytest tests/ -x -q` → 196 passed, 1 skipped
- **Committed in:** 570dde4 (Task 1 commit)

---

**Total deviations:** 1 auto-fixed (Rule 1 - bug)
**Impact on plan:** Necessary correctness fix for tests broken by the planned import change. No scope creep.

## Issues Encountered
- `python3.13 main.py --analysis` falls back to surrogate rather than printing "Using native VSPAERO" due to pre-existing `openvsp` module attribute issue (`module 'openvsp' has no attribute 'VSPCheckIsInit'`). This is a pre-existing condition from Phase 03-01/03-02, not caused by this plan. The core objective (no CadQuery crash) is met.

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- All analysis/validation modes work without CadQuery
- Geometry modes (--canard, --wing, --jigs, --generate-all) still import CadQuery locally when called
- 196 tests passing, ready for UAT or next phase

---
*Phase: 03-openvsp-native-integration*
*Completed: 2026-03-10*
