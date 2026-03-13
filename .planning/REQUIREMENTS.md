# Requirements: Open-EZ PDE

**Defined:** 2026-03-09
**Core Value:** All physics outputs validated against independent reference data — not self-referential baselines

## v1.1 Requirements

Requirements for Physical Validation & Calibration milestone. Each maps to roadmap phases.

### Bug Fixes

- [x] **BUG-01**: FS datum offset (~51") resolved so computed NP/CG translate correctly to published Long-EZ values via `datum_offset_in` config field and `to_published_datum()` method
- [x] **BUG-03**: Spar model upgraded from cap-only I-beam to D-box composite section producing realistic deflection values (5-15" under 450 lbf, not 89,169")

### Reference Data

- [x] **REF-01**: Published Rutan Long-EZ specifications curated from RAF CP-29/CP-31 and plans chapters with source citations in `reference_data.json`
- [x] **REF-02**: NACA/NASA wind tunnel data collected for Roncz R1145MS and Eppler 1230 airfoils
- [x] **REF-03**: Community build data researched from CSA newsletters, builder forums, and type club weigh-in records

### OpenVSP Integration

- [x] **VSP-01**: OpenVSP Python bindings installed on MacBook and `import openvsp` verified working
- [x] **VSP-02**: `_run_native_sweep()` in `vsp_integration.py` implemented — builds Long-EZ geometry, runs VSPAERO VLM, extracts CL/CD/CM polars (not "FIXME")
- [x] **VSP-03**: Surrogate (`OpenVSPAdapter`) cross-validated against real VSPAERO with discrepancies documented per metric
- [x] **VSP-04**: Real VSP wired into `main.py --generate-all` pipeline with surrogate fallback for CI

### Validation

- [x] **VAL-01**: Stability outputs (NP, CG, static margin) validated against published specs within 2" / 3% tolerance
- [x] **VAL-02**: Airfoil processing outputs (CLmax, Cm0, alpha_0L) validated against wind tunnel data
- [x] **VAL-03**: Structural model (D-box deflection, stress) validated against expected composite behavior
- [x] **VAL-04**: Performance outputs (stall speed, weights) validated against published specs
- [ ] **VAL-05**: Config values and surrogate coefficients calibrated to minimize error vs reference data AND real VSPAERO
- [ ] **VAL-06**: Machine-readable accuracy report generated with per-metric error margins and pass/fail grades
- [ ] **VAL-07**: Precision regression tests created locked to validated values (not self-referential)
- [ ] **VAL-08**: Self-referential baseline loop broken — `RegressionRunner` validates against external truth

## v2 Requirements

Deferred to future milestones. Tracked but not in current roadmap.

### Extended Validation
- **EVAL-01**: Altitude-dependent Reynolds number (currently hardcoded 8000 ft)
- **EVAL-02**: Canard AC uses MAC instead of root chord (~0.3" improvement)
- **EVAL-03**: BucklingAnalyzer stress estimate upgraded from heuristic to beam theory

### Extended Integration
- **EINT-01**: CG envelope validation across pilot weight × fuel load matrix
- **EINT-02**: Manufacturing G-code validated on physical CNC hardware

## Out of Scope

| Feature | Reason |
|---------|--------|
| BUG-02 (canard AC sweep correction) | Already fixed in Phase B3 — not needed |
| GU25-5(11)8 canard bypass | Safety mandate — no bypass possible |
| Full CFD (RANS/LES) | VLM sufficient for preliminary design |
| GUI for OpenVSP model viewer | CLI pipeline, headless VSPAERO |
| Physical CNC validation | Requires hardware access, deferred |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| BUG-01 | Phase 1 | Complete |
| REF-01 | Phase 1 | Complete |
| REF-02 | Phase 1 | Complete |
| REF-03 | Phase 1 | Complete |
| BUG-03 | Phase 2 | Complete |
| VAL-03 | Phase 2 | Complete |
| VSP-01 | Phase 3 | Complete |
| VSP-02 | Phase 3 | Complete |
| VSP-04 | Phase 3 | Complete |
| VAL-01 | Phase 4 | Complete |
| VAL-02 | Phase 4 | Complete |
| VAL-04 | Phase 4 | Complete |
| VSP-03 | Phase 4 | Complete |
| VAL-05 | Phase 5 | Pending |
| VAL-06 | Phase 5 | Pending |
| VAL-07 | Phase 6 | Pending |
| VAL-08 | Phase 6 | Pending |

**Coverage:**
- v1.1 requirements: 17 total
- Mapped to phases: 17
- Unmapped: 0 ✓

---
*Requirements defined: 2026-03-09*
*Last updated: 2026-03-09 after initial definition*
