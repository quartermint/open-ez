# Testing

## Framework

- **pytest** (>=7.0.0) with **pytest-cov** (>=4.0.0)
- Tests in `tests/` directory, discovered by `test_*.py` naming
- No conftest.py or shared fixtures file

## Test Suite

| Test File | LOC | Focus | Safety-Critical |
|-----------|-----|-------|-----------------|
| `test_airfoil_washout.py` | 166 | Washout rotation transforms, coordinate math | No |
| `test_beam_deflection.py` | 273 | Euler-Bernoulli beam analysis, distributed/elliptic loads, shear stress, buckling | Yes |
| `test_canard_stall_and_downwash.py` | 148 | Canard-first stall verification, downwash modeling | **Yes** |
| `test_compliance_gate.py` | 96 | FAA 51% rule enforcement, G-code export blocking | Yes |
| `test_lift_curve_theory.py` | 210 | Lift curve slope, finite wing corrections, CLmax effects | Yes |
| `test_manufacturing_accuracy.py` | 170 | G-code kerf compensation, feed rate, coordinate accuracy | Yes |
| `test_openvsp_runner.py` | 46 | OpenVSP adapter polars, lift curve slope | No |
| `test_physics_regression.py` | 21 | Physics baseline comparison | Yes |
| `test_ssot_weights.py` | 234 | Weight & balance, CG envelope, SSOT propagation | Yes |
| `test_torsion_flutter.py` | 163 | Torsional stiffness, flutter speed, control surface mass balance | **Yes** |

**Total**: ~1,530 LOC across 10 test files (plus 1 `__init__.py`)

## Test Data

| File | Purpose |
|------|---------|
| `tests/fixtures/reference_data.json` | Reference values for test assertions |
| `tests/snapshots/physics_baseline.json` | Physics regression baseline snapshot |

## Running Tests

```bash
# All tests
pytest tests/

# With coverage
pytest tests/ --cov=core --cov-report=term-missing

# Specific test
pytest tests/test_canard_stall_and_downwash.py -v

# Physics regression only
pytest tests/test_physics_regression.py
```

## Pre-commit Smoke Test

A custom pre-commit hook runs the geometry smoke test:
```yaml
- id: geometry-smoke-test
  entry: python scripts/smoke_test.py --fast --allow-missing-openvsp
```

This validates:
1. Configuration passes all safety checks
2. Canard geometry generates successfully (CadQuery loft)
3. Wing geometry generates successfully
4. OpenVSP probe (when available)

## Test Patterns

### Config-derived assertions
Tests read expected values from `config` rather than hard-coding:
```python
from config import config
assert result.wing_span == config.geometry.wing_span
```

### Safety-critical test pattern
Canard stall and flutter tests assert specific safety margins:
```python
# Canard must stall BEFORE wing (positive margin)
assert stall_margin_deg >= config.aero_limits.min_stall_margin_deg
```

### Physics regression pattern
Compares current computation against stored baseline within tolerance:
```python
runner = RegressionRunner(tolerance=0.05)
passed, current, failures = runner.compare_to_baseline(baseline_path, report_dir)
assert passed, f"Regressions failed: {failures}"
```

## Coverage

No explicit coverage targets configured. The test suite focuses on:
- Physics correctness (aerodynamics, structures, flutter)
- Safety-critical invariants (canard stall priority, compliance gates)
- SSOT propagation (config changes reflected in derived values)
- Manufacturing accuracy (G-code coordinates, kerf compensation)

## CI Integration

Tests run as part of the smoke test in CI (`.github/workflows/ci.yml`):
- Pre-commit hooks run ruff, mypy, and smoke test
- Geometry validation job generates and uploads artifacts
- No dedicated `pytest` job in CI currently (tests run via smoke test invocation)

## Missing Test Areas

- No unit tests for `core/nesting.py` (bin packing)
- No unit tests for `core/assembly.py` (full aircraft assembly)
- No unit tests for `core/systems.py` (propulsion)
- No unit tests for `core/metadata.py` (provenance)
- No integration tests that run `main.py` end-to-end
- No tests for DXF export correctness
- No tests for strake geometry
