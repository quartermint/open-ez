# Phase 4: Validation Test Infrastructure & Cross-Validation - Research

**Researched:** 2026-03-11
**Domain:** Python pytest test authoring, OpenVSP surrogate cross-validation, physics output validation against external reference data
**Confidence:** HIGH — all findings derived from direct codebase inspection; no speculative external research needed for this internal-to-project phase

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Tolerance Philosophy — TDD Across Phases**
- Precision tests encode the TARGET tolerance from success criteria (2"/3% for stability, 5% for performance) — they WILL FAIL with current physics
- Known discrepancy: NP is 5.8" off (fs_wing_le datum issue), CG range may also exceed 2" tolerance
- Failing tests are intentional and expected — Phase 5 calibration makes them pass
- Tests document the specific delta so Phase 5 knows exactly what to fix
- This is TDD across phases: write the failing test now, fix the code later

**Test Layering — Coexistence**
- Keep existing `test_physics_external_validation.py` sanity checks (generous tolerances, always green)
- Add NEW precision validation tests alongside them (tight tolerances, may fail)
- Two layers: sanity checks catch gross formula errors; precision tests track calibration targets
- Both layers run in the same `pytest` suite — precision test failures don't block CI (use `pytest.mark.xfail` or separate marker)

**Surrogate Cross-Validation (VSP-03) — Measure Only**
- Produce a discrepancy table comparing surrogate (`OpenVSPAdapter`) vs native VSPAERO per metric
- No pass/fail assertions on surrogate accuracy — just document the gap
- Discrepancy table committed to `data/validation/` as a JSON file
- Phase 5 decides what's acceptable and whether to recalibrate the surrogate

**Native Polar Regeneration**
- Phase 4 MUST re-run `_run_native_sweep()` to generate fresh `vspaero_native_polars.json` with confirmed real VSPAERO output
- Current file shows `vsp_version: "3.48.2-mock"` — cannot trust as real data
- Regenerated file must show real version string and real solver output before cross-validation proceeds

### Claude's Discretion
- Test file naming and organization (one file vs multiple per requirement)
- How to mark precision tests (xfail, custom marker, separate directory)
- Exact format of the cross-validation discrepancy JSON
- Whether to read tolerances from reference_data.json or hardcode in test assertions
- Airfoil validation approach (VAL-02): which specific metrics to compare against wind tunnel data

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| VAL-01 | Stability outputs (NP, CG, static margin) validated against published specs within 2" / 3% tolerance | `PhysicsEngine.calculate_cg_envelope()` produces NP/CG; `config.geometry.to_published_datum()` converts; `reference_data.json` has `neutral_point_fs.value=108.0` and `tolerance_abs=2.0`; known delta is 5.8" so test will xfail intentionally |
| VAL-02 | Airfoil processing outputs (CLmax, Cm0, alpha_0L) validated against NACA/NASA wind tunnel data | `reference_data.json` has Roncz R1145MS and Eppler 1230 wind tunnel values; `config.aero_limits` holds the code values; validation is a direct config-vs-reference comparison |
| VAL-04 | Performance outputs (stall speed, weights) validated against published specs within 5% tolerance | `reference_data.json` has `stall_speed_ktas.value=56` and `max_gross_weight_lb.value=1425`; stall speed must be computed from CL equation; gross weight from weight balance |
| VSP-03 | Surrogate (`OpenVSPAdapter`) cross-validated against real VSPAERO with discrepancies documented per metric | `OpenVSPAdapter.run_vspaero()` produces surrogate polars; `_run_native_sweep()` must be re-run with real OpenVSP; discrepancy JSON written to `data/validation/` |
</phase_requirements>

---

## Summary

Phase 4 is a test-writing phase, not a code-fixing phase. The work is entirely within the `tests/` directory and `data/validation/` directory. All the physics infrastructure, reference data, and integration points already exist from Phases 1-3. The planner needs to write precision validation tests against known reference data, mark them appropriately so CI stays green, regenerate the native VSPAERO polars file with real data, and produce a surrogate cross-validation JSON document.

The key insight from codebase inspection: VAL-01 (stability) and VAL-04 (performance) precision tests are expected to fail with current code — this is intentional TDD. The xfail marker pattern is the correct tool. VAL-02 (airfoil) is a config-vs-reference comparison that may already pass since `config.aero_limits` values match `reference_data.json` wind tunnel data exactly. VSP-03 requires running the real OpenVSP binary (which may only be available on Python 3.13 where the bindings are installed).

**Primary recommendation:** One new test file `test_precision_validation.py` for VAL-01/02/04, plus a standalone script `scripts/generate_cross_validation.py` for VSP-03 polar regeneration and discrepancy table generation. The script is a better fit than a test for the regeneration work because it requires real OpenVSP and writes data files.

---

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| pytest | 7.x (already installed) | Test runner, xfail marks, fixtures | Project standard, 197 tests already running |
| json | stdlib | Load `reference_data.json`, write discrepancy tables | Already used in all test files |
| pathlib.Path | stdlib | REPO_ROOT resolution | Established pattern in all test files |
| unittest.mock.MagicMock | stdlib | CadQuery mock | Established pattern: `sys.modules.setdefault("cadquery", MagicMock())` |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| pytest.mark.xfail | built-in | Mark precision tests expected to fail until Phase 5 | VAL-01 NP/CG tests; VAL-04 stall speed if computed wrong |
| pytest.mark (custom) | built-in | `@pytest.mark.precision` custom marker | Alternative to xfail — separates precision from sanity suite |
| math | stdlib | Stall speed equation: V_s = sqrt(2W/(rho*S*CLmax)) | VAL-04 stall speed computation |

### Installation
Nothing to install — all dependencies already present. The project runs 197 tests cleanly with `python3 -m pytest tests/ -q`.

---

## Architecture Patterns

### Recommended File Organization

```
tests/
├── test_precision_validation.py    # NEW: VAL-01, VAL-02, VAL-04 precision tests (may xfail)
├── test_physics_external_validation.py   # EXISTING: sanity checks — do not modify
├── test_datum_resolution.py         # EXISTING: datum + ref schema — do not modify
├── test_dbox_deflection.py          # EXISTING: VAL-03 complete — do not modify
└── test_vsp_native.py               # EXISTING: VSP schema — do not modify

scripts/
└── generate_cross_validation.py     # NEW: regenerate native polars + write discrepancy JSON

data/validation/
├── reference_data.json              # EXISTING: curated truth
├── vspaero_native_polars.json       # EXISTING but MOCK — must be regenerated
├── openvsp_validation.json          # EXISTING: surrogate polars
└── surrogate_cross_validation.json  # NEW: VSP-03 discrepancy table output
```

### Pattern 1: Precision xfail Test (VAL-01 archetype)

**What:** Tight-tolerance assertion against reference_data.json values, marked xfail so CI stays green while the calibration gap is documented.
**When to use:** Any metric known to have a calibration gap (NP is 5.8" off; CG range likely off too).

```python
# Source: established project pattern from test_datum_resolution.py
import json
import sys
import math
from pathlib import Path

REPO_ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(REPO_ROOT))

from unittest.mock import MagicMock
sys.modules.setdefault("cadquery", MagicMock())
sys.modules.setdefault("OCP", MagicMock())

from config import config
from core.analysis import PhysicsEngine

REF_DATA_PATH = REPO_ROOT / "data" / "validation" / "reference_data.json"

def _load_ref_data():
    with open(REF_DATA_PATH) as f:
        return json.load(f)

@pytest.mark.xfail(
    reason="NP 5.8\" off due to fs_wing_le datum issue (Phase 5 calibrates)",
    strict=False,  # strict=False allows it to sometimes pass without error
)
def test_np_precision_2inch_tolerance():
    """NP must be within 2\" of published 108.0 in (published datum). XFAIL until Phase 5."""
    data = _load_ref_data()
    ref_np = data["aircraft_specs"]["neutral_point_fs"]["value"]   # 108.0
    tol = data["aircraft_specs"]["neutral_point_fs"]["tolerance_abs"]  # 2.0

    engine = PhysicsEngine()
    metrics = engine.calculate_cg_envelope()
    computed_np_published = config.geometry.to_published_datum(metrics.neutral_point)

    delta = abs(computed_np_published - ref_np)
    # Document exact delta for Phase 5
    assert delta <= tol, (
        f"NP delta = {delta:.2f}\" (computed {computed_np_published:.2f}, "
        f"reference {ref_np}). Phase 5 target: <= {tol}\"."
    )
```

### Pattern 2: Config-vs-Reference Direct Comparison (VAL-02 archetype)

**What:** Compare `config.aero_limits.*` values against `reference_data.json` airfoil entries. No engine computation needed.
**When to use:** VAL-02 airfoil metrics (CLmax, Cm0, alpha_0L) — these are config constants, not computed.

```python
# Source: reference_data.json structure inspection
def test_roncz_clmax_matches_wind_tunnel():
    """canard_clmax config must match Roncz wind tunnel data within 0.05."""
    data = _load_ref_data()
    ref_clmax = data["airfoil_data"]["roncz_r1145ms"]["cl_max"]["value"]   # 1.35
    tolerance = 0.05  # Discretion: tight but allows minor modeling adjustments

    assert abs(config.aero_limits.canard_clmax - ref_clmax) <= tolerance, (
        f"canard_clmax={config.aero_limits.canard_clmax} vs wind tunnel {ref_clmax}"
    )
```

### Pattern 3: Stall Speed Computation (VAL-04 archetype)

**What:** Compute stall speed from first principles using config values, compare against reference.
**When to use:** VAL-04 performance — stall speed has no dedicated method in existing code.

```python
# Source: physics first-principles; reference stall_speed_ktas = 56 KTAS
import math
from core.atmosphere import density  # exists in core/atmosphere.py

def test_stall_speed_within_5pct():
    """Computed stall speed at gross weight must be within 5% of published 56 KTAS."""
    data = _load_ref_data()
    ref_stall = data["aircraft_specs"]["stall_speed_ktas"]["value"]   # 56 KTAS
    tol_pct = 0.05

    # V_stall = sqrt(2 * W / (rho * S * CLmax))
    # Use gross weight, sea level standard for worst-case stall
    W = config.flight_condition.gross_weight_lb            # 1425 lb
    S_sqft = config.geometry.wing_area + config.geometry.canard_area  # combined
    S_sqft_ref = 94.2 + 15.6   # published reference areas (sqft)
    CLmax = config.aero_limits.canard_clmax                # canard stalls first
    rho_sl = 0.002377  # slug/ft^3 at sea level standard

    # Convert area to sqft if config returns sqft already (verify!)
    v_stall_fps = math.sqrt(2 * W / (rho_sl * S_sqft_ref * CLmax))
    v_stall_ktas = v_stall_fps / 1.6878

    delta_pct = abs(v_stall_ktas - ref_stall) / ref_stall
    assert delta_pct <= tol_pct, (
        f"Stall speed {v_stall_ktas:.1f} KTAS vs published {ref_stall} KTAS "
        f"({delta_pct*100:.1f}% error, limit {tol_pct*100:.0f}%)"
    )
```

### Pattern 4: Surrogate Cross-Validation Script (VSP-03)

**What:** Python script (not a test) that: (1) runs `_run_native_sweep()` to regenerate `vspaero_native_polars.json`, (2) runs `OpenVSPAdapter.run_vspaero()` to get surrogate polars, (3) compares per-metric at matching alpha values, (4) writes `surrogate_cross_validation.json`.
**When to use:** VSP-03 — this is a data-generation task, not a pass/fail assertion.

```python
# Source: openvsp_adapter.py and vsp_integration.py inspection
# scripts/generate_cross_validation.py

from pathlib import Path
import json
from core.vsp_integration import VSPIntegration
from core.simulation.openvsp_adapter import OpenVSPAdapter

REPO_ROOT = Path(__file__).resolve().parents[1]
DATA_DIR = REPO_ROOT / "data" / "validation"
NATIVE_POLARS = DATA_DIR / "vspaero_native_polars.json"
SURROGATE_POLARS = DATA_DIR / "openvsp_validation.json"
OUTPUT_PATH = DATA_DIR / "surrogate_cross_validation.json"

def regenerate_native():
    bridge = VSPIntegration()
    assert bridge.has_vsp, "Real OpenVSP required for native polar regeneration"
    result = bridge._run_native_sweep((-4, 14, 19), polar_output=NATIVE_POLARS)
    assert "3.48.2-mock" not in result["vsp_version"], \
        f"Mock polars detected — OpenVSP not actually installed. Got: {result['vsp_version']}"
    return result

def get_surrogate_polars():
    adapter = OpenVSPAdapter()
    alphas = [float(a) for a in range(-4, 15)]
    polars = adapter.run_vspaero(alphas)
    return polars

def build_discrepancy_table(native_pts, surrogate_pts):
    """
    Output format (per metric per alpha):
    {
        "metadata": {...},
        "alpha_deg": [...],
        "cl_native": [...], "cl_surrogate": [...], "cl_delta": [...],
        "cd_native": [...], "cd_surrogate": [...], "cd_delta": [...],
        "cm_native": [...], "cm_surrogate": [...], "cm_delta": [...],
        "summary": { "cl_rms": ..., "cd_rms": ..., "cm_rms": ... }
    }
    """
    ...
```

### Anti-Patterns to Avoid

- **Modifying existing sanity-check tests:** `test_physics_external_validation.py` and `test_datum_resolution.py` must remain unchanged. The 8" NP tolerance in `test_published_np_matches_translation` is correct for its purpose (catches gross errors). Do not tighten it.
- **Hardcoding tolerance values in tests:** Read `tolerance_abs` and `tolerance_pct` from `reference_data.json` so Phase 5 can adjust reference data and tests automatically update.
- **Using `strict=True` on xfail for known-failing tests:** `strict=True` would cause CI failure if the test unexpectedly passes. Use `strict=False` so that Phase 5 improvements don't break the suite when tests start passing.
- **Running `_run_native_sweep()` from inside a pytest test without real OpenVSP:** The native regeneration step requires real OpenVSP (Python 3.13 only). Put it in a script, not a test. Use `@pytest.mark.skipif(not HAS_OPENVSP, ...)` for any test that calls native VSP.
- **Comparing surrogate and native polars at different alpha arrays:** Surrogate uses arbitrary alphas from `run_vspaero(alphas)`. Native uses -4 to 14 in 1-degree steps. Force both to the same 19-point array for valid comparison.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Cross-file test helpers | Duplicate `_load_ref_data()` in new file | Copy the helper (or factor into `tests/conftest.py`) | Already established pattern; 2 test files use it |
| CadQuery mocking | Re-implement mock | `sys.modules.setdefault("cadquery", MagicMock())` at top | Established per-file pattern used in every test file |
| stall speed function | Build new module | Inline math in test (V_s = sqrt(2W/rhoSCL)) | Simple enough not to warrant new infrastructure |
| Surrogate polar normalization | Complex interpolation | Force both polars to same alpha list before comparison | Adapter's `run_vspaero(alphas)` accepts explicit alphas |
| xfail reason strings | Generic | Include exact delta and Phase 5 reference | Specific reasons help Phase 5 planner target the right fix |

**Key insight:** This phase adds no new production code. Everything is tests and a data-generation script. Resist the temptation to add convenience wrappers to `core/` — the scope boundary is firm.

---

## Common Pitfalls

### Pitfall 1: xfail strict=True Breaks Phase 5

**What goes wrong:** If `strict=True` and Phase 5 fixes the NP formula, the xfail test now passes — but `strict=True` makes a passing xfail count as a failure, breaking CI.
**Why it happens:** `strict=True` means "this MUST fail; if it passes, that's an error."
**How to avoid:** Use `strict=False` (default) for all precision tests. Or add a separate `pytest.mark.precision` marker and exclude them from default CI runs.
**Warning signs:** Phase 5 reports CI failures on tests they just fixed.

### Pitfall 2: Mock vsp_version in Native Polars File

**What goes wrong:** Cross-validation runs against the current `vspaero_native_polars.json` which has `"vsp_version": "3.48.2-mock"` — a mock file, not real VSPAERO output. All "native" data is synthetic and the discrepancy table is meaningless.
**Why it happens:** Phase 3 final state left the file as mock output. The surrogate cross-validation makes no sense until real native polars are generated.
**How to avoid:** In `generate_cross_validation.py`, assert `"mock" not in result["vsp_version"]` before proceeding. The script must run on the Python 3.13 environment where OpenVSP is installed.
**Warning signs:** `vsp_version` field in `vspaero_native_polars.json` contains "mock".

### Pitfall 3: Area Convention in Stall Speed Calculation

**What goes wrong:** Using `config.geometry.wing_area` (full trapezoidal, ~110 sqft) for the stall speed formula when the published reference data uses RAF's semi-panel convention (94.2 sqft). This produces a different stall speed and a false mismatch.
**Why it happens:** The area convention discrepancy is documented in `reference_data.json` notes — AR=7.3 vs 6.34 same issue.
**How to avoid:** For stall speed against published data, use `ref_wing_area = 94.2` and `ref_canard_area = 15.6` (published values) to match the published calculation basis. Document the convention in the test comment.
**Warning signs:** Stall speed computes to 60+ KTAS when reference is 56 KTAS.

### Pitfall 4: VAL-02 Already Passes (Config Constants Match Reference)

**What goes wrong:** Writing VAL-02 precision tests as xfail when they actually pass — because `config.aero_limits.canard_clmax = 1.35` was set to match the Roncz wind tunnel data, and `alpha_0L = -3.0` matches the reference exactly.
**Why it happens:** VAL-02 is a config-vs-reference comparison, not a computation. The config was populated from the wind tunnel data in Phase 1.
**How to avoid:** Run VAL-02 tests WITHOUT xfail first; verify they pass. Only add xfail if they actually fail. The CONTEXT.md notes VAL-02 validation is more about documenting the comparison than expecting failures.
**Warning signs:** xfail tests marked as XPASS — they pass but weren't supposed to.

### Pitfall 5: conftest.py vs Per-File Helper

**What goes wrong:** Duplicating `_load_ref_data()` and CadQuery mock setup in the new test file creates maintenance drift if the reference data path ever changes.
**Why it happens:** Each existing test file is self-contained (no shared conftest.py).
**How to avoid:** Two acceptable approaches: (1) add a `tests/conftest.py` with shared fixtures — but this changes existing test patterns; or (2) copy the pattern as-is for consistency. Recommendation: copy the pattern for now; refactor is Phase 6 scope.
**Warning signs:** `_load_ref_data()` in three files with slightly different paths.

---

## Code Examples

Verified patterns from direct codebase inspection:

### CadQuery Mock (mandatory, every test file)
```python
# Source: established pattern, every tests/*.py file
import sys
from pathlib import Path
from unittest.mock import MagicMock

REPO_ROOT = Path(__file__).resolve().parents[1]
sys.path.insert(0, str(REPO_ROOT))

sys.modules.setdefault("cadquery", MagicMock())
sys.modules.setdefault("OCP", MagicMock())

from config import config  # noqa: E402
```

### Loading Reference Data
```python
# Source: test_datum_resolution.py _load_ref_data() helper
import json
REF_DATA_PATH = REPO_ROOT / "data" / "validation" / "reference_data.json"

def _load_ref_data():
    with open(REF_DATA_PATH) as f:
        return json.load(f)
```

### PhysicsEngine NP Computation + Datum Translation
```python
# Source: test_datum_resolution.py TestDatumReferenceDataConsistency
from core.analysis import PhysicsEngine

engine = PhysicsEngine()
metrics = engine.calculate_cg_envelope()
computed_np_published = config.geometry.to_published_datum(metrics.neutral_point)
# Note: metrics.neutral_point is in INTERNAL coords (~159.29)
# to_published_datum() subtracts datum_offset_in=45.5 -> ~113.79 published
# Reference is 108.0, delta ~5.79" — known datum issue (xfail at 2" tolerance)
```

### Config Aero Limits for VAL-02
```python
# Source: config/aircraft_config.py AeroLimitsParams + reference_data.json
# These values were set from wind tunnel data — direct comparison is valid
config.aero_limits.canard_clmax      # 1.35 (Roncz WT ref: 1.35)
config.aero_limits.canard_alpha_0L   # -3.0 (Roncz WT ref: -3.0)
config.aero_limits.wing_clmax        # 1.45 (Eppler ref: 1.45)
config.aero_limits.wing_alpha_0L     # -2.0 (Eppler ref: -2.0)
# reference_data.json["airfoil_data"]["roncz_r1145ms"]["cm_zero"]["value"] = -0.05
# No cm_zero in config.aero_limits — this metric may need a separate assertion
```

### Surrogate Polar Generation
```python
# Source: core/simulation/openvsp_adapter.py OpenVSPAdapter.run_vspaero()
from core.simulation.openvsp_adapter import OpenVSPAdapter

adapter = OpenVSPAdapter()
alphas = [float(a) for a in range(-4, 15)]  # -4 to 14, 19 points
polars = adapter.run_vspaero(alphas)
# Returns List[AeroPolar] with .alpha_deg, .cl, .cd, .cm
```

### Native Polar Regeneration
```python
# Source: core/vsp_integration.py VSPIntegration._run_native_sweep()
from core.vsp_integration import VSPIntegration
from pathlib import Path

NATIVE_POLARS = Path("data/validation/vspaero_native_polars.json")
bridge = VSPIntegration()
if bridge.has_vsp:
    result = bridge._run_native_sweep((-4, 14, 19), polar_output=NATIVE_POLARS)
    # result["vsp_version"] must NOT contain "mock"
    # result["points"] is list of {"alpha_deg", "cl", "cd", "cm"}
```

### xfail Pattern
```python
# Source: pytest documentation — standard xfail for known calibration gaps
import pytest

@pytest.mark.xfail(
    reason="NP delta ~5.79\" (fs_wing_le datum); Phase 5 fixes",
    strict=False,
)
def test_np_precision_2inch():
    ...
```

---

## State of the Art

| Old Approach | Current Approach | Context | Impact |
|--------------|------------------|---------|--------|
| Single sanity layer (generous tol) | Two-layer: sanity + precision xfail | Phases 1-3 only built sanity layer | Phase 4 adds the precision layer; both coexist |
| Mock native polars (`vsp_version: 3.48.2-mock`) | Real VSPAERO output | Phase 3 left mock in place | Phase 4 regeneration task; cross-validation is meaningless without it |
| No surrogate cross-validation | `surrogate_cross_validation.json` | VSP-03 pending | Phase 4 creates this; Phase 5 decides on calibration |
| Config values set from wind tunnel data | Explicit test asserting the match | Implicit only | VAL-02 makes this match explicit and versioned |

---

## Integration Points (Critical for Planning)

### What Each Test Calls

| Requirement | Entry Point | Key Return Values | Import Path |
|-------------|-------------|-------------------|-------------|
| VAL-01 (NP) | `PhysicsEngine().calculate_cg_envelope()` | `.neutral_point` (internal FS), `.cg_range_fwd`, `.cg_range_aft` | `from core.analysis import PhysicsEngine` |
| VAL-01 (CG) | `config.geometry.to_published_datum(internal_fs)` | published FS float | `from config import config` |
| VAL-02 (CLmax, alpha_0L) | `config.aero_limits.*` direct | float constants | `from config import config` |
| VAL-02 (Cm0) | Not in config — reference_data.json has -0.05 for canard | compare against reference only | wind tunnel constant |
| VAL-04 (stall speed) | Inline stall equation using config values | V_stall KTAS | `math.sqrt(...)` |
| VAL-04 (gross weight) | `config.flight_condition.gross_weight_lb` | 1425.0 | `from config import config` |
| VSP-03 (native) | `VSPIntegration._run_native_sweep((-4,14,19), polar_output=path)` | dict with `points` | `from core.vsp_integration import VSPIntegration` |
| VSP-03 (surrogate) | `OpenVSPAdapter().run_vspaero(alphas)` | `List[AeroPolar]` | `from core.simulation.openvsp_adapter import OpenVSPAdapter` |

### Reference Data Fields Used

| Test | `reference_data.json` path | Value | Tolerance |
|------|---------------------------|-------|-----------|
| VAL-01 NP | `aircraft_specs.neutral_point_fs.value` | 108.0 in (published) | `tolerance_abs: 2.0"` |
| VAL-01 fwd CG | `aircraft_specs.cg_range_fwd_fs.value` | 99.0 in (published) | `tolerance_abs: 1.0"` |
| VAL-01 aft CG | `aircraft_specs.cg_range_aft_fs.value` | 104.0 in (published) | `tolerance_abs: 1.0"` |
| VAL-02 R1145MS CLmax | `airfoil_data.roncz_r1145ms.cl_max.value` | 1.35 | Discretion: 0.05 |
| VAL-02 R1145MS alpha_0L | `airfoil_data.roncz_r1145ms.alpha_zero_lift_deg.value` | -3.0 | Discretion: 0.5 deg |
| VAL-02 R1145MS Cm0 | `airfoil_data.roncz_r1145ms.cm_zero.value` | -0.05 | Discretion: 0.01 |
| VAL-02 E1230 CLmax | `airfoil_data.eppler_1230.cl_max.value` | 1.45 | Discretion: 0.05 |
| VAL-02 E1230 alpha_0L | `airfoil_data.eppler_1230.alpha_zero_lift_deg.value` | -2.0 | Discretion: 0.5 deg |
| VAL-04 stall speed | `aircraft_specs.stall_speed_ktas.value` | 56 KTAS | `tolerance_pct: 5%` |
| VAL-04 gross weight | `aircraft_specs.max_gross_weight_lb.value` | 1425 lb | exact config match |

---

## Open Questions

1. **Where does Cm0 live in the code?**
   - What we know: `reference_data.json` has Roncz R1145MS Cm0 = -0.05. `config.aero_limits` does not have a cm_zero field. `OpenVSPAdapter._cm_reflex_offset()` computes a Cm value but it's not the same as section Cm0.
   - What's unclear: Is there a computed Cm0 to validate, or is VAL-02 Cm0 purely a "reference data has it, config doesn't implement it" gap?
   - Recommendation: VAL-02 Cm0 test asserts `reference_data.json` has the value as a schema check (it's there), but cannot compare against a computed value. Document as "config does not implement section Cm0 — VAL-02 Cm0 is a data presence check only."

2. **Does real OpenVSP work on the current dev machine (Python 3.14)?**
   - What we know: `test_real_native_sweep_produces_polars` is skipped with "OpenVSP not installed". The install targets Python 3.13. The runtime is Python 3.14.3.
   - What's unclear: Whether `python3.13 scripts/generate_cross_validation.py` can be invoked directly for the regeneration step.
   - Recommendation: The plan must note that `generate_cross_validation.py` must be run under Python 3.13 (`python3.13 scripts/generate_cross_validation.py`), not the default Python 3.14.

3. **xfail vs custom marker for CI strategy**
   - What we know: Both approaches work. xfail is simpler — one decorator. Custom `@pytest.mark.precision` requires `pytest.ini` registration and a CI exclusion flag.
   - What's unclear: Whether the project wants precision failures visible in default CI runs or hidden behind a marker.
   - Recommendation: Use `@pytest.mark.xfail(strict=False)` — simpler, no config changes, visible in reports as XFAIL rather than FAILED. If Phase 5 wants to separate them, add the marker then.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | pytest (version inferred from working suite) |
| Config file | None — no pytest.ini or pyproject.toml; pytest discovers from `tests/` directory |
| Quick run command | `python3 -m pytest tests/test_precision_validation.py -v` |
| Full suite command | `python3 -m pytest tests/ -q` |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| VAL-01 | NP within 2" of published 108.0 in (will xfail) | unit/xfail | `python3 -m pytest tests/test_precision_validation.py::test_np_precision_2inch -v` | ❌ Wave 0 |
| VAL-01 | Fwd CG limit within 2" of published 99.0 in (will xfail) | unit/xfail | `python3 -m pytest tests/test_precision_validation.py::test_cg_fwd_limit_precision -v` | ❌ Wave 0 |
| VAL-01 | Aft CG limit within 2" of published 104.0 in (will xfail) | unit/xfail | `python3 -m pytest tests/test_precision_validation.py::test_cg_aft_limit_precision -v` | ❌ Wave 0 |
| VAL-02 | Roncz R1145MS CLmax = 1.35 ± 0.05 | unit | `python3 -m pytest tests/test_precision_validation.py::test_roncz_clmax_matches_wind_tunnel -v` | ❌ Wave 0 |
| VAL-02 | Roncz alpha_0L = -3.0 ± 0.5 deg | unit | `python3 -m pytest tests/test_precision_validation.py::test_roncz_alpha_0l_matches_wind_tunnel -v` | ❌ Wave 0 |
| VAL-02 | Eppler 1230 CLmax = 1.45 ± 0.05 | unit | `python3 -m pytest tests/test_precision_validation.py::test_eppler_clmax_matches_wind_tunnel -v` | ❌ Wave 0 |
| VAL-02 | Eppler alpha_0L = -2.0 ± 0.5 deg | unit | `python3 -m pytest tests/test_precision_validation.py::test_eppler_alpha_0l_matches_wind_tunnel -v` | ❌ Wave 0 |
| VAL-04 | Stall speed within 5% of 56 KTAS (may xfail) | unit | `python3 -m pytest tests/test_precision_validation.py::test_stall_speed_5pct -v` | ❌ Wave 0 |
| VAL-04 | Gross weight config = 1425 lb | unit | `python3 -m pytest tests/test_precision_validation.py::test_gross_weight_matches_published -v` | ❌ Wave 0 |
| VSP-03 | `surrogate_cross_validation.json` exists with correct schema | smoke | `python3 -m pytest tests/test_precision_validation.py::test_cross_validation_json_exists -v` | ❌ Wave 0 |
| VSP-03 | Native polar regeneration (real OpenVSP required) | manual-only | `python3.13 scripts/generate_cross_validation.py` | ❌ Wave 0 |

**Manual-only justification (VSP-03 native):** Real OpenVSP is Python 3.13 only; default runtime is Python 3.14. The regeneration script is a one-time data-generation step, not a repeatable automated test. The schema test (`test_cross_validation_json_exists`) confirms the output file is present and valid JSON with required keys.

### Sampling Rate

- **Per task commit:** `python3 -m pytest tests/ -q` (197 + new tests, ~1 second)
- **Per wave merge:** `python3 -m pytest tests/ -v` (shows xfail reasons clearly)
- **Phase gate:** Full suite green (or expected xfail) before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] `tests/test_precision_validation.py` — covers VAL-01, VAL-02, VAL-04, and VSP-03 schema check
- [ ] `scripts/generate_cross_validation.py` — covers VSP-03 native regeneration + discrepancy table
- [ ] `data/validation/surrogate_cross_validation.json` — generated artifact (not test code, but needed before schema test passes)

No framework installation needed — pytest already working.

---

## Sources

### Primary (HIGH confidence)
- Direct file inspection: `tests/test_physics_external_validation.py` — sanity layer patterns
- Direct file inspection: `tests/test_datum_resolution.py` — reference data loading, xfail context
- Direct file inspection: `tests/test_dbox_deflection.py` — established test class structure
- Direct file inspection: `tests/test_vsp_native.py` — OpenVSP guard pattern, native vs surrogate
- Direct file inspection: `core/analysis.py` — `PhysicsEngine.calculate_cg_envelope()` return values
- Direct file inspection: `core/simulation/openvsp_adapter.py` — `OpenVSPAdapter.run_vspaero()` API
- Direct file inspection: `core/vsp_integration.py` — `_run_native_sweep()` API, polar_output param
- Direct file inspection: `data/validation/reference_data.json` — all reference values and tolerances
- Direct file inspection: `data/validation/vspaero_native_polars.json` — confirms mock data present
- Direct file inspection: `config/aircraft_config.py` — `AeroLimitsParams` field values
- Test run: `python3 -m pytest tests/ -q` — confirmed 197 passed, 1 skipped

### Secondary (MEDIUM confidence)
- `.planning/phases/04-validation-test-infrastructure-cross-validation/04-CONTEXT.md` — user decisions
- `.planning/STATE.md` — accumulated project decisions and known issues
- `.planning/REQUIREMENTS.md` — requirement definitions and traceability

### Tertiary (LOW confidence)
None — all findings are from direct codebase inspection with verified test execution.

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — verified by running test suite; all imports work
- Architecture: HIGH — derived from established patterns in 8 existing test files
- Integration points: HIGH — verified by reading actual method signatures
- Known xfail candidates: HIGH — 5.8" NP delta documented in STATE.md and test comments
- Pitfalls: HIGH — derived from actual test patterns and reference data schema inspection
- VSP-03 Python version constraint: HIGH — test_vsp_native.py confirms OpenVSP not installed on Python 3.14

**Research date:** 2026-03-11
**Valid until:** 2026-04-10 (stable domain — no external library changes affect this)
