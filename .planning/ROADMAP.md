# Roadmap: Open-EZ PDE

## Milestones

- ✅ **v1.0 Physics & Structural Foundations** - Phases A-F (shipped 2026-02-28)
- 🚧 **v1.1 Physical Validation & Calibration** - Phases 1-7 (in progress)

## Phases

<details>
<summary>✅ v1.0 Physics & Structural Foundations (Phases A-F) - SHIPPED 2026-02-28</summary>

Phases A-F completed under legacy naming convention. See MILESTONES.md for full summary.

Key deliverables: 22 validation tests, 7 physics fixes, structural analysis suite, ISA atmosphere, Viterna post-stall, ComplianceTaskTracker, Roncz safety mandate enforced at 4 layers.

</details>

### 🚧 v1.1 Physical Validation & Calibration (In Progress)

**Milestone Goal:** Close the self-referential baseline gap — install real OpenVSP, validate and calibrate all outputs against published data and real VSPAERO, then lock validated values with regression tests.

- [x] **Phase 1: Reference Data & Datum Resolution** - Curate external reference dataset and resolve the 51" FS datum offset (completed 2026-03-10)
- [ ] **Phase 2: D-Box Structural Model** - Replace cap-only beam model with D-box composite section
- [x] **Phase 3: OpenVSP Native Integration** - Install real OpenVSP, implement native VSPAERO sweep, wire into pipeline (gap closure in progress) (completed 2026-03-12)
- [x] **Phase 4: Validation Test Infrastructure** - Precision validation tests comparing outputs to curated reference data and real VSPAERO (completed 2026-03-13)
- [x] **Phase 5: Calibration & Accuracy Report** - Tune parameters, generate machine-readable accuracy report (completed 2026-03-13)
- [x] **Phase 6: Regression Lock-In** - Replace self-referential baselines with externally-validated regression tests (completed 2026-03-13)
- [ ] **Phase 7: Milestone Verification & Gap Closure** - Verify Phase 2, wire community data integration, resolve tech debt (gap closure)

## Phase Details

### Phase 1: Reference Data & Datum Resolution
**Goal**: Published Long-EZ reference data is curated with source citations and the 51" FS datum offset is resolved so computed NP/CG values translate correctly to published values
**Depends on**: Nothing (first phase)
**Requirements**: BUG-01, REF-01, REF-02, REF-03
**Plans:** 2/2 plans complete

Plans:
- [x] 01-01-PLAN.md — Create reference_data.json with provenance schema and add datum offset to GeometricParams
- [x] 01-02-PLAN.md — Wire dual FS display into StabilityMetrics and lock with validation tests

**Success Criteria** (what must be TRUE):
  1. `reference_data.json` exists with published Long-EZ specs from RAF CP-29/CP-31 and wind tunnel data for both airfoils, each entry citing its source
  2. `config` exposes a `datum_offset_in` field and a `to_published_datum()` method that converts computed FS values to the published coordinate system
  3. Computed NP at FS 153.5 (internal) translates to ~FS 102-114 (published) via `to_published_datum()`, matching the ~51" known offset
  4. Community build data (CSA newsletters, builder forums, type club weigh-ins) is captured in reference data with provenance noted

### Phase 2: D-Box Structural Model
**Goal**: Spar model upgraded from cap-only I-beam to D-box composite section, producing realistic tip deflection values for the Long-EZ wing under design loads
**Depends on**: Nothing (parallel to Phase 1)
**Requirements**: BUG-03, VAL-03
**Plans:** 2 plans

Plans:
- [ ] 02-01-PLAN.md — D-box section model, config parameterization, spanwise EI, numerical deflection integration
- [ ] 02-02-PLAN.md — Wire D-box into nominal_spar_check, failure checks, FlutterEstimator integration, validation tests

**Success Criteria** (what must be TRUE):
  1. `BeamFEAAdapter` (or equivalent D-box model) computes tip deflection of 5-15" under 450 lbf load, replacing the physically absurd 89,169" cap-only result
  2. D-box section properties (skin contribution, shear web) are parameterized from `aircraft_config.py` (ply counts, chord fraction, laminate schedule)
  3. Validation test `test_beam_deflection.py` passes with D-box model and deflection within expected composite behavior range
  4. Shear stress and Tsai-Wu failure checks continue to pass with updated section geometry

### Phase 3: OpenVSP Native Integration
**Goal**: Real OpenVSP Python bindings are installed and working, `_run_native_sweep()` builds Long-EZ geometry and runs VSPAERO VLM to produce real CL/CD/CM polars, and the pipeline uses real VSP with CI-safe surrogate fallback
**Depends on**: Nothing (parallel to Phases 1 and 2)
**Requirements**: VSP-01, VSP-02, VSP-04
**Plans:** 5/5 plans complete

Plans:
- [x] 03-01-PLAN.md — Add VSP geometry params to config, create install script, user installs OpenVSP
- [x] 03-02-PLAN.md — Implement native VSPAERO sweep, wire into pipeline, CI surrogate fallback tests
- [x] 03-04-PLAN.md — Defer CadQuery imports in main.py (gap closure)
- [x] 03-05-PLAN.md — Fix VSPCheckIsInit bug, verify native VSPAERO end-to-end (gap closure)
- [x] 03-06-PLAN.md — Fix VSPAERO VLM solver pipeline — thin surface export and geometry validation (gap closure)

**Success Criteria** (what must be TRUE):
  1. `import openvsp` succeeds on MacBook without error and the installed version is confirmed in a requirements file or install script
  2. `_run_native_sweep()` in `vsp_integration.py` returns a polar dict with CL, CD, CM values across the configured alpha sweep (not "FIXME" or placeholder data)
  3. `python main.py --generate-all` invokes real VSPAERO when `openvsp` is importable, and automatically falls back to `OpenVSPAdapter` surrogate when it is not
  4. The surrogate fallback path is tested in CI without OpenVSP installed and all existing tests continue to pass

### Phase 4: Validation Test Infrastructure & Cross-Validation
**Goal**: Precision validation tests compare all major physics outputs against curated reference data (Phase 1) and real VSPAERO polars (Phase 3), and the surrogate is cross-validated against real VSP with discrepancies documented
**Depends on**: Phase 1, Phase 3
**Requirements**: VAL-01, VAL-02, VAL-04, VSP-03
**Plans:** 2/2 plans complete

Plans:
- [x] 04-01-PLAN.md — Precision validation tests for stability (VAL-01), airfoil (VAL-02), and performance (VAL-04)
- [x] 04-02-PLAN.md — Surrogate cross-validation: regenerate native polars, produce discrepancy table (VSP-03)

**Success Criteria** (what must be TRUE):
  1. Stability outputs (NP, CG range, static margin) pass validation tests against published Long-EZ specs within 2" / 3% tolerance after datum translation
  2. Airfoil processing outputs (CLmax, Cm0, alpha_0L) pass validation tests against NACA/NASA wind tunnel data for Roncz R1145MS and Eppler 1230
  3. Performance outputs (stall speed, max gross weight) pass validation tests against published specs within 5% tolerance
  4. `OpenVSPAdapter` surrogate is cross-validated against real VSPAERO polars and a discrepancy table (per metric) is committed to `data/validation/`

### Phase 5: Calibration & Accuracy Report
**Goal**: Config values and surrogate coefficients are tuned to minimize error vs reference data and real VSPAERO, and a machine-readable accuracy report is generated with per-metric grades
**Depends on**: Phase 4
**Requirements**: VAL-05, VAL-06
**Plans:** 2/2 plans complete

Plans:
- [x] 05-01-PLAN.md — Calibrate fs_wing_le from NP delta analysis, remove xfail decorators, write calibration log (VAL-05)
- [x] 05-02-PLAN.md — Create accuracy report generator script, wire into main.py, add schema/traceability tests (VAL-06)

**Success Criteria** (what must be TRUE):
  1. Calibrated config values (e.g., Oswald efficiency, canard downwash factor) produce lower mean absolute error vs reference data than the pre-calibration baseline
  2. A machine-readable accuracy report (JSON or CSV) is generated by a script or `main.py` flag and includes per-metric error margins and pass/fail grades against defined tolerances
  3. Every metric in the accuracy report traces to a source in `reference_data.json` or a real VSPAERO run — no self-referential entries

### Phase 6: Regression Lock-In
**Goal**: Self-referential baseline loop is broken — regression tests lock to externally-validated values and `RegressionRunner` validates against external truth, not values it generated itself
**Depends on**: Phase 5
**Requirements**: VAL-07, VAL-08
**Plans:** 2/2 plans complete

Plans:
- [x] 06-01-PLAN.md — Create precision regression tests locked to Phase 5 calibrated values (VAL-07)
- [x] 06-02-PLAN.md — Rewire RegressionRunner to accuracy_report.json baselines, deprecate physics_baseline.json (VAL-08)

**Success Criteria** (what must be TRUE):
  1. Precision regression tests exist for each validated metric, locked to values from Phase 5 calibration, with tolerances derived from physical measurement uncertainty (not statistical spread of self-generated data)
  2. `RegressionRunner` reads baselines from `reference_data.json` or the Phase 5 accuracy report — not from `physics_baseline.json` generated by its own prior runs
  3. Running `pytest` with the new regression tests fails if physics outputs drift outside calibrated bounds, catching future regressions against external truth
  4. `physics_baseline.json` is either removed or clearly marked as deprecated/non-authoritative in code comments and documentation

### Phase 7: Milestone Verification & Gap Closure
**Goal**: All audit gaps closed — Phase 2 formally verified, community build data wired into accuracy metrics, tech debt resolved, milestone ready for completion
**Depends on**: Phase 6
**Requirements**: BUG-03 (verification), VAL-03 (verification), REF-03 (integration)
**Gap Closure:** Closes gaps from v1.1 milestone audit
**Plans:** 1 plan

Plans:
- [ ] 07-01-PLAN.md — Phase 2 verification, community data wiring, ROADMAP/SUMMARY tech debt cleanup

**Success Criteria** (what must be TRUE):
  1. Phase 2 VERIFICATION.md exists confirming BUG-03 and VAL-03 satisfied with evidence (27 D-box tests pass, SUMMARYs document work)
  2. Community build empty_weight data from `reference_data.json` consumed by `accuracy_report.json` metrics (closing REF-03 integration gap)
  3. ROADMAP.md entries reflect actual completion status (Phase 2 checkbox checked, progress counts accurate)
  4. SUMMARY frontmatter gaps fixed (REF-02, REF-03 in 01-02-SUMMARY, VAL-06 in 05-02-SUMMARY)
  5. All uncommitted data file changes committed

## Progress

**Execution Order:**
Phases 1, 2, 3 can execute in parallel. Phase 4 depends on all three. Phase 5 depends on Phase 4. Phase 6 depends on Phase 5.

| Phase | Milestone | Plans Complete | Status | Completed |
|-------|-----------|----------------|--------|-----------|
| 1. Reference Data & Datum Resolution | 2/2 | Complete   | 2026-03-10 | - |
| 2. D-Box Structural Model | v1.1 | 0/2 | Planning complete | - |
| 3. OpenVSP Native Integration | 5/5 | Complete   | 2026-03-12 | - |
| 4. Validation Test Infrastructure | 2/2 | Complete   | 2026-03-13 | - |
| 5. Calibration & Accuracy Report | 2/2 | Complete   | 2026-03-13 | - |
| 6. Regression Lock-In | 2/2 | Complete   | 2026-03-13 | - |
| 7. Milestone Verification & Gap Closure | v1.1 | 0/1 | Planned | - |
