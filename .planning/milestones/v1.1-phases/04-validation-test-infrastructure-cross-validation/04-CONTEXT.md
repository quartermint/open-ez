# Phase 4: Validation Test Infrastructure & Cross-Validation - Context

**Gathered:** 2026-03-11
**Status:** Ready for planning

<domain>
## Phase Boundary

Precision validation tests comparing all major physics outputs against curated reference data (Phase 1's `reference_data.json`) and real VSPAERO polars (Phase 3), plus surrogate cross-validation (VSP-03). This phase BUILDS test infrastructure and MEASURES discrepancies — it does NOT calibrate or tune parameters (that's Phase 5).

Requirements: VAL-01 (stability 2"/3%), VAL-02 (airfoil vs wind tunnel), VAL-04 (performance 5%), VSP-03 (surrogate cross-validation). Note: VAL-03 (structural) is already complete from Phase 2.

</domain>

<decisions>
## Implementation Decisions

### Tolerance Philosophy — TDD Across Phases
- Precision tests encode the TARGET tolerance from success criteria (2"/3% for stability, 5% for performance) — they WILL FAIL with current physics
- Known discrepancy: NP is 5.8" off (fs_wing_le datum issue), CG range may also exceed 2" tolerance
- Failing tests are intentional and expected — Phase 5 calibration makes them pass
- Tests document the specific delta so Phase 5 knows exactly what to fix
- This is TDD across phases: write the failing test now, fix the code later

### Test Layering — Coexistence
- Keep existing `test_physics_external_validation.py` sanity checks (generous tolerances, always green)
- Add NEW precision validation tests alongside them (tight tolerances, may fail)
- Two layers: sanity checks catch gross formula errors; precision tests track calibration targets
- Both layers run in the same `pytest` suite — precision test failures don't block CI (use `pytest.mark.xfail` or separate marker)

### Surrogate Cross-Validation (VSP-03) — Measure Only
- Produce a discrepancy table comparing surrogate (`OpenVSPAdapter`) vs native VSPAERO per metric
- No pass/fail assertions on surrogate accuracy — just document the gap
- Discrepancy table committed to `data/validation/` as a JSON file
- Phase 5 decides what's acceptable and whether to recalibrate the surrogate

### Native Polar Regeneration
- Phase 4 MUST re-run `_run_native_sweep()` to generate fresh `vspaero_native_polars.json` with confirmed real VSPAERO output
- Current file shows `vsp_version: "3.48.2-mock"` — cannot trust as real data
- Regenerated file must show real version string and real solver output before cross-validation proceeds

### Claude's Discretion
- Test file naming and organization (one file vs multiple per requirement)
- How to mark precision tests (xfail, custom marker, separate directory)
- Exact format of the cross-validation discrepancy JSON
- Whether to read tolerances from reference_data.json or hardcode in test assertions
- Airfoil validation approach (VAL-02): which specific metrics to compare against wind tunnel data

</decisions>

<specifics>
## Specific Ideas

- The existing `test_datum_resolution.py:test_published_np_matches_translation` already tests NP with 8" tolerance — Phase 4 adds a precision version at 2" tolerance that's expected to fail
- `reference_data.json` has `tolerance_abs` and `tolerance_pct` fields per entry (Phase 1 decision) — tests could read from there to be data-driven
- VAL-03 is already done (`test_dbox_deflection.py` with 14 tests) — Phase 4 only needs VAL-01, VAL-02, VAL-04, VSP-03
- The surrogate uses lifting-line + Viterna post-stall; native VSPAERO uses VLM — expect meaningful CL/CD discrepancies at higher alpha

</specifics>

<code_context>
## Existing Code Insights

### Reusable Assets
- `data/validation/reference_data.json` — curated external truth with per-entry tolerances and source citations
- `data/validation/vspaero_native_polars.json` — native VSPAERO output (needs regeneration)
- `data/validation/openvsp_validation.json` — surrogate adapter cached polars
- `test_physics_external_validation.py` — existing sanity-check pattern to coexist with
- `test_datum_resolution.py` — datum offset and reference data schema tests (Phase 1)
- `test_dbox_deflection.py` — D-box structural validation (VAL-03 complete)

### Established Patterns
- CadQuery mocking: `sys.modules.setdefault("cadquery", MagicMock())` at top of every test file
- Config-derived assertions: read expected values from `config` not hardcoded
- REPO_ROOT path resolution: `Path(__file__).resolve().parents[1]`
- JSON reference data loading: `_load_ref_data()` helper in test_datum_resolution.py

### Integration Points
- `core/analysis.py:PhysicsEngine.calculate_cg_envelope()` — produces NP, CG range, static margin (VAL-01)
- `core/analysis.py:AirfoilFactory` — produces CLmax, Cm0, alpha_0L (VAL-02)
- `config.geometry.to_published_datum()` — converts internal FS to published FS for comparison
- `vsp_integration.py:VSPIntegration._run_native_sweep()` — generates real VSPAERO polars
- `openvsp_adapter.py:OpenVSPAdapter.run_vspaero()` — surrogate polars for cross-validation

</code_context>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 04-validation-test-infrastructure-cross-validation*
*Context gathered: 2026-03-11*
