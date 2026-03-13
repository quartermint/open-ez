# Phase 5: Calibration & Accuracy Report - Context

**Gathered:** 2026-03-13
**Status:** Ready for planning

<domain>
## Phase Boundary

Tune config values (Oswald efficiency, canard downwash, datum offset / fs_wing_le) to minimize error vs reference data, then generate a machine-readable accuracy report (JSON) with per-metric error margins, pass/marginal/fail grades, and full source traceability. Surrogate recalibration is out of scope — surrogate stays as a rough CI fallback. Phase 6 uses the accuracy report to lock regression tests.

</domain>

<decisions>
## Implementation Decisions

### Calibration Scope
- Config values only: wing_oswald_e, canard_oswald_e, datum_offset_in, and fs_wing_le
- Surrogate (OpenVSPAdapter) NOT recalibrated — it stays as CI fallback with known CL/CD gaps
- All 4 xfail tests (NP, CG fwd, CG aft, stall speed) MUST pass after calibration — no known-limit exceptions

### Calibration Method
- Research-based manual tuning: investigate correct values from literature and VSPAERO results, set them directly
- No scipy.optimize — values must have physical justification, not just minimize error numerically
- Investigate fs_wing_le root cause (find correct value from Rutan plans) rather than just refining the datum offset constant

### Calibration Artifacts
- Before/after calibration log as a separate JSON file (e.g., data/validation/calibration_log.json)
- Shows old value → new value → source/justification for each tuned parameter
- Separate from the accuracy report — clean separation of "what changed" vs "how accurate are we now"

### Accuracy Report Format
- JSON format, written to data/validation/accuracy_report.json
- Consistent with existing data/validation/*.json pattern
- Includes summary section (counts: X PASS, Y MARGINAL, Z FAIL) + detailed per-metric breakdown
- All validated metrics included: NP, CG fwd/aft, static margin, stall speed, CLmax, Cm0, alpha_0L, empty weight, gross weight, wing area, AR

### Accuracy Report Invocation
- Standalone script: scripts/generate_accuracy_report.py
- Also available via main.py --accuracy-report flag (calls same logic)
- Both paths produce identical output

### Grading Scheme
- PASS: computed value within tolerance (reads tolerance_abs/tolerance_pct from reference_data.json)
- MARGINAL: within 2x tolerance
- FAIL: beyond 2x tolerance

### xfail Test Resolution
- All 4 xfail tests must pass after calibration — tolerance is the contract, not a suggestion
- Remove xfail decorators entirely when tests pass (git history shows they were once xfail)
- Replace with brief comment noting Phase 5 resolution (e.g., "# Resolved Phase 5: calibrated datum_offset_in")
- Leave both test layers: sanity checks (generous) + precision tests (calibrated). Two safety nets.
- Stall speed xfail (VAL-04) already XPASSes — remove xfail now as part of Phase 5

### Traceability Enforcement
- Schema enforcement: each metric entry in report JSON has required 'source' field
- Valid sources: reference_data.json key or 'vspaero_native' — report generation fails if source missing or invalid
- Hard block on self-referential sources: report generation raises error if any source resolves to physics_baseline.json
- Full VSPAERO provenance: report metadata includes vsp_version, run_timestamp, solver_settings
- Schema validation test: pytest test loads accuracy report, validates schema, checks all sources exist, rejects physics_baseline.json

### Claude's Discretion
- Exact JSON schema field ordering and nesting for accuracy report
- How to structure the report generation script internally
- Which VSPAERO solver settings to include in provenance metadata
- Exact tolerance values for metrics not yet in reference_data.json (e.g., static margin)

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `data/validation/reference_data.json` — curated external truth with per-entry tolerance_abs/tolerance_pct and source citations
- `data/validation/surrogate_cross_validation.json` — measure-only discrepancy table (Phase 4)
- `data/validation/vspaero_native_polars.json` — real VSPAERO output with version/timestamp
- `tests/test_precision_validation.py` — 4 xfail tests documenting exact deltas to close
- `scripts/generate_cross_validation.py` — pattern for validation data generation scripts
- `config/aircraft_config.py` — GeometricParams with wing_oswald_e (0.80), canard_oswald_e (0.75), datum_offset_in (45.5), fs_wing_le (133)

### Established Patterns
- SSOT config: all values from config/aircraft_config.py
- JSON output in data/validation/ with metadata headers
- CadQuery mocking in tests: sys.modules.setdefault("cadquery", MagicMock())
- _load_ref_data() helper pattern in test_datum_resolution.py

### Integration Points
- `config/aircraft_config.py:104` — wing_oswald_e = 0.80 (tunable)
- `config/aircraft_config.py:112` — canard_oswald_e = 0.75 (tunable)
- `config.geometry.datum_offset_in` — 45.5 (tunable or replaced by fs_wing_le fix)
- `config.geometry.fs_wing_le` — 133 (root cause investigation target)
- `main.py` — needs --accuracy-report flag addition
- `core/analysis.py:PhysicsEngine` — produces NP, CG, static margin
- `core/analysis.py:AirfoilFactory` — produces CLmax, Cm0, alpha_0L

</code_context>

<specifics>
## Specific Ideas

- The 5.8" NP error (computed 113.79 vs reference 108.0 published) is the primary calibration target — fs_wing_le investigation should be done first as it may cascade to other metrics
- Stall speed already passes (1.8% delta) — just remove the xfail decorator
- The surrogate CM values are orders of magnitude different from VSPAERO (e.g., -0.013 vs -1.445 at alpha=3°) — this is likely a reference length/area convention difference, not a calibration issue. Document but don't try to fix.
- The calibration log pattern (before/after with justification) is valuable for Phase 6 regression lock-in — Phase 6 can read the calibration log to understand why values were chosen

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 05-calibration-accuracy-report*
*Context gathered: 2026-03-13*
