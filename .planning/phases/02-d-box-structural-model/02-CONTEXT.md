# Phase 2: D-Box Structural Model - Context

**Gathered:** 2026-03-10
**Status:** Ready for planning

<domain>
## Phase Boundary

Replace the cap-only I-beam bending model in `BeamFEAAdapter` with a D-box composite section (spar caps + shear web + D-box skins from LE to spar at 25% chord). Produces realistic tip deflection (5-15" under 450 lbf elliptic load) instead of the physically absurd 89,169" from the cap-only model. FlutterEstimator transitions to D-box EI for bending frequency. Existing cap-only tests preserved as regression.

</domain>

<decisions>
## Implementation Decisions

### D-Box Section Geometry
- D-box extends from leading edge to main spar (25% chord) — aft skins NOT included (conservative)
- Structural contributors to bending stiffness: spar caps (UNI), upper/lower D-box skins (BID), shear web (foam-core sandwich)
- Shear web modeled as foam-core sandwich: BID glass face sheets (1 ply each, 0.013") on Clark foam core (~0.25")
- Reuse geometry definition pattern from `build_wing_torsion_section()` which already models the same 25%-chord D-box for torsion

### Spanwise Variation
- D-box section properties vary station-by-station along the span (tapered wing: root 62" to tip 27" chord)
- Spar cap ply count tapers from root to tip via configurable schedule (e.g., [17, 17, 14, 11, 8] at 5 stations)
- Deflection computed by numerical integration of M(x)²/EI(x) over the span, not closed-form
- Follows pattern of `CompositeFEAAdapter.analyze_spar_cap()` which already does station-by-station analysis

### Adapter Architecture
- Route bending through CLT/ABD path using `CompositeFEAAdapter` infrastructure (not simple `BeamSection`)
- `BeamFEAAdapter` either upgraded to accept D-box sections or a new `DBoxBeamAdapter` wraps `CompositeSection` for bending
- `nominal_spar_check()` updated to use D-box model as primary, cap-only as legacy reference

### Validation Load Case
- Primary validation: elliptic distributed load (most realistic for wing lift) — `analyze_elliptic(span=79.2, total_load=450)`
- Expected result: 5-15" tip deflection
- Secondary (conservative): point load at tip — reported alongside elliptic for reference
- Both reported in `nominal_spar_check()` output

### Failure Checks
- Tsai-Wu on spar cap plies (existing, upgraded for D-box geometry)
- Tsai-Wu on D-box skin plies (new)
- Shear check on web BID skins (new)
- Foam core compression failure check (new) — uses existing foam allowables dict (styrofoam_blue: 25 psi, urethane_2lb: 45 psi, divinycell_h45: 85 psi)

### FlutterEstimator Integration
- `FlutterEstimator.bending_frequency_hz()` switches from cap-only `BeamSection` EI to D-box EI
- Torsion already uses D-box (`build_wing_torsion_section`) — now bending matches
- Expected improvement: higher bending frequency → better frequency ratio → more realistic flutter margin

### Config Parameterization
- New config fields: D-box chord fraction (default 0.25), skin layup schedule, shear web foam thickness, web BID ply count
- Spar cap ply schedule as list (replaces single `spar_cap_plies` integer for the tapered model)
- Derive D-box depth from chord fraction × local chord × airfoil thickness ratio
- All existing config fields preserved for backward compatibility

### Test Strategy
- Existing 8 cap-only beam tests KEPT as regression (they validate Euler-Bernoulli math, still correct)
- New D-box validation tests: deflection in 5-15" range, failure margins positive, weight estimate reasonable
- Shear stress and Tsai-Wu checks must continue to pass with updated section geometry

### Claude's Discretion
- Exact numerical integration scheme for tapered EI (Simpson's rule, trapezoidal, etc.)
- Number of spanwise stations for D-box analysis (minimum 5, more if convergence needs it)
- Whether to create a new `DBoxBeamAdapter` class or upgrade `BeamFEAAdapter` directly
- BucklingAnalyzer integration approach (use D-box stresses or keep existing heuristic)
- Weight estimation method for D-box structure

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `build_wing_torsion_section()` in `fea_adapter.py:663-698` — models same 25%-chord D-box for torsion (Bredt-Batho), defines enclosed area and wall segments
- `CompositeFEAAdapter` in `fea_adapter.py:435-618` — CLT/ABD analysis, station-by-station spar cap checks, ply-by-ply Tsai-Wu
- `CompositeSection.abd_matrices()` — computes [A], [B], [D] for arbitrary ply stackups
- `CompositeSection.equivalent_bending_stiffness()` — returns D11 × width (beam EI equivalent)
- `CompositePly.stiffness_matrix_global()` — ply-level Q-bar transformation
- Foam allowables dict in `calculate_shear_stress()` at line 148 — ready for foam compression checks
- `UNI_GLASS_PROPERTIES` and `BID_GLASS_PROPERTIES` material dicts — complete with Tsai-Wu allowables

### Established Patterns
- SSOT config: all dimensions from `config/aircraft_config.py`, accessed via `from config import config`
- Dataclass sections: `BeamSection`, `CompositePly`, `CompositeSection`, `TorsionSection` — D-box section follows this pattern
- Station-by-station analysis: `analyze_spar_cap()` iterates over N stations with variable moments
- Result dataclasses: `BeamResult`, `SparCapResult` — D-box result follows this pattern
- CadQuery mocking in tests: `sys.modules.setdefault("cadquery", MagicMock())`

### Integration Points
- `BeamFEAAdapter.__init__()` at line 39 — constructs section from config materials (upgrade point)
- `FlutterEstimator.bending_frequency_hz()` at line 718 — uses `BeamFEAAdapter` for EI (switch to D-box)
- `nominal_spar_check()` at line 106 — primary structural output (add D-box results)
- `config.flutter.skin_thickness_in = 0.048` — already parameterized for D-box skin
- `config.materials.spar_cap_plies = 17`, `spar_cap_width = 3.0`, `spar_modulus_psi = 2.8e6` — existing spar cap config

</code_context>

<specifics>
## Specific Ideas

- The 89,169" deflection is mathematically correct for cap-only I-beam (I = 0.000895 in⁴) — it's not a bug, it's an incomplete model. The D-box adds orders of magnitude more stiffness.
- `build_wing_torsion_section()` already computes D-box geometry (enclosed area, wall segments) — the bending model should share this geometry definition to avoid inconsistency between torsion and bending D-box assumptions.
- The ply drop-off schedule [17, 17, 14, 11, 8] is a starting point — exact values should come from Rutan plans or standard practice for glass/foam composite wings. Research should identify real Long-EZ ply schedules.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 02-d-box-structural-model*
*Context gathered: 2026-03-10*
