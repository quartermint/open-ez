# Milestones: Open-EZ PDE

## v1.1 Physical Validation & Calibration (Shipped: 2026-03-13)

**Phases:** 7 | **Plans:** 16 | **Commits:** 97 | **Files changed:** 78 | **LOC:** 17,367 Python
**Timeline:** 4 days (2026-03-09 → 2026-03-13)
**Tests:** 241 passing, 2 skipped

**Key accomplishments:**
- Curated Long-EZ reference dataset from RAF CP-29/CP-31 with full provenance and resolved 45.5" FS datum offset
- Replaced cap-only I-beam with D-box composite structural model (deflection 89,169" → 2.34", weight ~8 lb/wing)
- Integrated OpenVSP 3.48.2 native VSPAERO VLM solver with CI-safe surrogate fallback
- Built precision validation suite: 241 tests against published specs, wind tunnel data, and real VSPAERO
- Calibrated fs_wing_le (133→125.61") and CG margins to match Rutan CP-29; generated 12-metric accuracy report (9 PASS)
- Broke self-referential baseline loop: regression tests locked to externally-validated accuracy_report.json

**Known gaps carried forward:**
- Wing area: full trapezoidal (110 sqft) vs RAF semi-panel convention (94.2 sqft)
- Empty weight: partial structural model vs community builds (821-891 lb)
- fs_wing_le calibration (125.61") pending CP-31 physical confirmation
- altitude_ft param ignored in Reynolds calc (hardcoded 8000 ft)

---

## v1.0 — Physics & Structural Foundations (Complete)

**Completed:** 2026-02-28
**Phases:** A through F (6 phases)

### Summary

Established the physics, structural, and manufacturing foundations for the Open-EZ PDE. Implemented 7 critical physics fixes (washout, lift curve slope, canard efficiency, distributed loads, piecewise MAC, induced drag, airfoil blending), added structural analysis (shear stress, buckling, CLT Tsai-Wu), and aero improvements (Reynolds number, stall modeling, reflex ramp). Merged 3 PRs, validated integration across NP calculation, structural load cases, and manufacturing pipeline.

### Key Deliverables

- 22 validation tests (airfoil washout, lift curve theory, beam deflection, OpenVSP runner, physics regression)
- Anderson eq 5.69 sweep-corrected lift curve slope
- Piecewise MAC for strake+wing planform
- BeamFEAAdapter with point, distributed, and elliptic load methods
- BucklingAnalyzer and CompositeFEAAdapter with Tsai-Wu
- ISA atmosphere model for Reynolds calculation
- Viterna post-stall model
- ComplianceTaskTracker for FAA 51% Rule
- Roncz R1145MS canard safety mandate enforced at 4 layers

### Known Issues Carried Forward

- FS datum offset (~51" from published) — not a formula error, needs translation constant
- Spar deflection 89,169" — correct for cap-only model, needs D-box upgrade
- Self-referential physics baselines
- OpenVSP `_run_native_sweep()` is placeholder
- `altitude_ft` param ignored in Reynolds calc (hardcoded 8000 ft)

### Last Phase Number: F (legacy naming — next milestone uses numeric phases starting at 1)

---
*Last updated: 2026-03-09*
