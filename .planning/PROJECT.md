# Open-EZ PDE

## What This Is

A "Plans-as-Code" Parametric Design Environment for modernizing the Rutan Long-EZ (Model 61) aircraft. Transforms fragmented legacy plans into executable Python code with CadQuery geometry generation, OpenVSP aerodynamic validation, airfoil processing with spline interpolation, 4-axis G-code for CNC foam cutting, and FAA 51% Rule compliance tracking.

## Core Value

All physics outputs must be validated against independent reference data — not self-referential baselines — so that builders can trust the computed dimensions, loads, and stability margins before cutting foam.

## Requirements

### Validated

<!-- Shipped and confirmed valuable — Phases A-F (v1.0) -->

- ✓ Washout sign convention fix (negate theta for CW rotation) — Phase B1
- ✓ Sweep-corrected lift curve slope (Anderson eq 5.69) — Phase B2
- ✓ Geometry-based canard efficiency factor — Phase B3
- ✓ Distributed + elliptic load methods on BeamFEAAdapter — Phase B4
- ✓ Piecewise MAC for strake+wing — Phase B5
- ✓ Proper induced drag formula cd = cd0 + CL²/(πeAR) — Phase B6
- ✓ Smooth airfoil blend replacing binary switch at eta=0.5 — Phase B7
- ✓ Shear stress calculation — Phase C1
- ✓ BucklingAnalyzer — Phase C2
- ✓ CLT-based Tsai-Wu failure criterion — Phase C3
- ✓ Reynolds number calculation with ISA atmosphere — Phase D1
- ✓ Stall modeling (Viterna-style post-stall) — Phase D2
- ✓ Linear reflex ramp — Phase D3
- ✓ ComplianceTaskTracker — Phase E
- ✓ Roncz R1145MS canard mandate enforced at 4 layers — Phase E
- ✓ Integration validation (NP, structural loads, manufacturing pipeline) — Phase F
- ✓ 22 validation tests covering physics, beam theory, lift curve — Phase A

<!-- Shipped and confirmed valuable — Phases 1-7 (v1.1) -->

- ✓ FS datum offset resolved (45.5" translation constant, reference_data.json with provenance) — v1.1
- ✓ D-box composite structural model (deflection 89,169" → 2.34", realistic weight estimates) — v1.1
- ✓ Published Long-EZ reference data curated from RAF CP-29/CP-31 with source citations — v1.1
- ✓ OpenVSP 3.48.2 native VSPAERO VLM integration with CI-safe surrogate fallback — v1.1
- ✓ Precision validation against published specs, wind tunnel data, and real VSPAERO — v1.1
- ✓ Self-referential baselines replaced with externally-validated regression tests — v1.1
- ✓ Calibrated accuracy report (12 metrics, 9 PASS) with per-metric error margins — v1.1

### Active

<!-- Next milestone scope — TBD -->

- [ ] Altitude-dependent Reynolds number (currently hardcoded 8000 ft)
- [ ] Canard AC uses MAC instead of root chord (~0.3" improvement)
- [ ] BucklingAnalyzer stress estimate upgraded from heuristic to beam theory
- [ ] CG envelope validation across pilot weight × fuel load matrix
- [ ] Manufacturing G-code validated on physical CNC hardware

### Out of Scope

- GU25-5(11)8 canard airfoil bypass — Safety mandate, no bypass possible
- GUI/visualization layer — CLI-first, parametric code output
- Full FEA mesh solver — Beam/CLT approximations sufficient for preliminary design
- Electric propulsion validation — IC engine (O-235) is the baseline
- Full CFD (RANS/LES) — VLM sufficient for preliminary design

## Context

- **Tech stack:** Python 3.10+, CadQuery (OpenCASCADE), OpenVSP 3.48.2 (NASA), NumPy/SciPy, ezdxf
- **SSOT config:** `config/aircraft_config.py` — all dimensions derive from here
- **Codebase:** 17,367 LOC Python, 241 tests passing
- **Datum:** Resolved — `datum_offset_in = 45.5` converts internal FS to published Long-EZ coordinates via `to_published_datum()`
- **Structural model:** D-box composite section (EI ~101M lb-in², deflection 2.34" under 450 lbf, weight ~8 lb/wing half)
- **OpenVSP:** Native VSPAERO VLM solver operational; `_run_native_sweep()` builds geometry, runs solver, extracts CL/CD/CM polars; surrogate fallback for CI
- **Validation:** 12-metric accuracy report with 9 PASS grades; 3 known convention gaps (wing area, empty weight, static margin) documented
- **Regression:** Locked to externally-validated values via `accuracy_report.json`; `physics_baseline.json` deprecated
- **Known gaps:** Wing area uses full trapezoidal (110 sqft) vs RAF semi-panel convention (94.2 sqft); empty weight is partial structural model; fs_wing_le calibration (125.61") pending CP-31 physical confirmation

## Constraints

- **Safety:** Roncz R1145MS canard airfoil is mandatory — enforced at config, factory, generator, and validation layers
- **FAA compliance:** System must generate Fabrication Aids, not finished parts (14 CFR Part 21.191(g))
- **CI compatibility:** Tests must pass with and without OpenVSP installed (surrogate fallback)
- **Backward compatibility:** Internal FS coordinate system preserved; datum translation is additive

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Roncz R1145MS mandatory canard | GU25-5(11)8 causes dangerous pitch-down in rain | ✓ Good |
| SSOT config singleton | All dimensions derive from one source | ✓ Good |
| Anderson eq 5.69 for lift curve slope | More accurate than simple lifting-line for swept wings | ✓ Good |
| Datum offset as translation constant (not geometry change) | Internal FS system is self-consistent; issue is only in comparison to published data | ✓ Good — 45.5" offset validated against published NP |
| D-box composite model for spar | Cap-only I-beam gives physically absurd deflections | ✓ Good — deflection 2.34", weight ~8 lb/wing |
| Surrogate fallback for CI | OpenVSP not available on all runners | ✓ Good — native + surrogate paths both tested |
| fs_wing_le calibrated to 125.61" | Analytical NP sensitivity derivation (dNP/dFS=0.7838) | ⚠️ Revisit — pending CP-31 physical confirmation |
| Two-tier regression (reference + drift) | Reference tolerance catches gross errors, drift tolerance catches code regressions | ✓ Good |
| PASS-only regression filtering | 3 FAIL metrics are known convention differences, not regressions | ✓ Good |
| Community weight data as context field | Preserves grade/reference unchanged, adds builder range as validation context | ✓ Good |

## Shipped Milestones

- **v1.0** Physics & Structural Foundations — Phases A-F (shipped 2026-02-28)
- **v1.1** Physical Validation & Calibration — Phases 1-7 (shipped 2026-03-13)

## Current Milestone

Planning next milestone. Use `/gsd:new-milestone` to begin.

---
*Last updated: 2026-03-13 after v1.1 milestone*
