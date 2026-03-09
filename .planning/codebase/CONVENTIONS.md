# Code Conventions

## Code Style

- **Formatter**: ruff-format (via pre-commit)
- **Linter**: ruff (via pre-commit, with `--fix`)
- **Type checker**: mypy (only on `scripts/` directory in pre-commit; project-wide via CI)
- **Python version**: 3.10+ (type hints use `Optional`, `Dict`, `List` from `typing`, plus some `X | Y` union syntax in newer files)

## Import Patterns

### Standard pattern
```python
from config import config           # SSOT singleton
from config.aircraft_config import AirfoilType  # Enums
from .base import AircraftComponent, FoamCore    # Relative imports within core/
```

### Lazy imports to avoid heavy dependencies
```python
# core/__init__.py uses __getattr__ with _LAZY_IMPORTS dict
# Avoids pulling CadQuery when only compliance utils are needed
```

### Deferred/conditional imports
```python
from __future__ import annotations  # Used in manufacturing.py, simulation/
if TYPE_CHECKING:
    from .base import FoamCore      # Avoid circular imports
```

### Path manipulation for scripts
```python
project_root = Path(__file__).parent
sys.path.insert(0, str(project_root))
```

## Class Hierarchy

```
AircraftComponent (ABC)
|-- FoamCore (ABC)           # Adds G-code export, hot-wire profiles
|   |-- WingGenerator        # Lofted wing with sweep/dihedral/washout
|   |   |-- CanardGenerator  # Enforces Roncz airfoil
|   |   |-- MainWingGenerator
|   |-- (future foam components)
|-- Bulkhead (ABC)           # 2D profile extrusion
|-- Fuselage                 # Station-based loft
|-- StrakeGenerator          # Wing-fuselage integration
|-- AircraftAssembly         # Full aircraft

Propulsion (ABC)
|-- LycomingO235             # Baseline IC engine
|-- ElectricEZ               # Electric conversion
```

## Configuration Pattern

All parametric values live in `config/aircraft_config.py` as nested `@dataclass` objects:

```python
@dataclass
class AircraftConfig:
    geometry: GeometricParams
    materials: MaterialParams
    manufacturing: ManufacturingParams
    airfoils: AirfoilSelection
    compliance: ComplianceParams
    strakes: StrakeConfig
    propulsion: PropulsionConfig
    flight_condition: FlightConditionParams
    structural_weights: StructuralWeightParams
    aero_limits: AeroLimitsParams
    flutter: FlutterParams

config = AircraftConfig()  # Singleton, validates on import
```

Derived values are `@property` methods on the dataclasses (e.g., `geometry.canard_arm`, `materials.spar_trough_depth`).

## Error Handling

- **`ComplianceError`**: Raised when FAA 51% rule is violated with `strict_compliance` enabled
- **`ValueError`**: Used for invalid config, unknown types, missing data
- **`RuntimeError`**: Used in smoke tests for validation failures
- **`warnings.warn()`**: Used for safety violations detected at import time (canard airfoil check)
- **`logging`**: Used in manufacturing, VSP integration for non-fatal warnings
- **Try/except with fallback**: Used extensively in geometry operations (CadQuery loft failures fall back to simplified geometry)

## Safety Enforcement Pattern

Safety-critical constraints are enforced at multiple layers:

1. **Config validation** (`AircraftConfig.validate()`): Warns on import if canard airfoil is wrong
2. **Factory enforcement** (`AirfoilFactory.get_canard_airfoil()`): Overrides config if tampered
3. **Generator enforcement** (`CanardGenerator.__init__()`): Always uses `get_canard_airfoil()`
4. **Compliance gate** (`FoamCore.export_gcode()`): Checks builder credit before G-code export

## Documentation Style

- Docstrings: Google-style with Args/Returns/Raises sections
- Module-level docstrings explain purpose and safety mandates
- `SAFETY:` and `SAFETY CRITICAL:` prefixes on comments for life-critical code
- Constants documented inline with units in comments

## Metadata / Provenance

Every exported artifact (STEP, STL, DXF, G-code) gets a `.metadata.json` sidecar:
```json
{
  "artifact": "canard_core.step",
  "artifact_type": "STEP",
  "generated_at": "2026-01-01T07:57:43.123Z",
  "revision": "abc1234",
  "config_hash": "sha256...",
  "contributor": "builder_name",
  "component": { "name": "canard_core", ... },
  "provenance": { "toolchain": "Open-EZ PDE", "automated": true }
}
```

## Numerical Conventions

- **All dimensions in inches** unless noted (feet for span/area in display)
- **Angles in degrees** at API boundaries, converted to radians internally
- **Airfoil coordinates normalized to unit chord** (0.0 to 1.0)
- **Fuselage stations (FS)** measured from nose datum in inches
- **Butt Lines (BL)** measured from aircraft centerline in inches
- **Weights in pounds**, arms in inches, moments in in-lb
