# Architecture

## Design Philosophy

**Plans-as-Code**: Traditional aircraft plans (paper drawings, hand-typed dimensions) are replaced by executable Python code. All geometry, analysis, and manufacturing output derives from a single parametric configuration.

**Single Source of Truth (SSOT)**: `config/aircraft_config.py` defines every aircraft dimension. No module may hard-code dimensions. Changes to config propagate through geometry, analysis, manufacturing, and documentation.

**Safety-first**: The Roncz R1145MS canard airfoil is enforced at config validation, factory level, and generator level. The original GU25-5(11)8 is marked deprecated and unsafe.

## Architectural Pattern

**Pipeline / Command-driven** architecture:

```
Config (SSOT) --> Geometry Generation --> Analysis/Validation --> Manufacturing Output
     |                  |                       |                        |
aircraft_config.py   core/structures.py    core/analysis.py        core/manufacturing.py
                     core/aerodynamics.py  core/simulation/        core/nesting.py
                     core/assembly.py                              core/compliance/
```

Entry point: `main.py` with CLI flags dispatches to generation/analysis functions.

## Layers

### 1. Configuration Layer (`config/`)

- `config/aircraft_config.py`: ~680 LOC dataclass hierarchy defining all parameters
- Key dataclasses: `AircraftConfig` (master), `GeometricParams`, `MaterialParams`, `ManufacturingParams`, `AirfoilSelection`, `ComplianceParams`, `StrakeConfig`, `PropulsionConfig`, `FlightConditionParams`, `StructuralWeightParams`, `AeroLimitsParams`, `FlutterParams`
- Singleton `config = AircraftConfig()` imported everywhere
- Validates on import (warns if canard airfoil wrong or compliance < 51%)

### 2. Core Geometry Layer (`core/`)

- **`core/base.py`**: Abstract base classes (`AircraftComponent`, `FoamCore`, `Bulkhead`)
  - `AircraftComponent.generate_geometry()` -> `export_step()` -> `export_stl()` -> `export_dxf()`
  - `FoamCore.export_gcode()` with compliance gate check
  - `Bulkhead` auto-generates from 2D profile extrusion
- **`core/aerodynamics.py`**: `AirfoilFactory` (loads .dat files, caches), `Airfoil` (spline processing, washout, reflex, blend, offset)
- **`core/structures.py`**: `WingGenerator`, `CanardGenerator`, `MainWingGenerator`, `Fuselage`, `StrakeGenerator`
  - Wing lofting through computed spanwise stations with sweep/dihedral/washout
  - Canard enforces Roncz airfoil at construction time
  - Strake supports fuel and battery modes
- **`core/assembly.py`**: `AircraftAssembly` combines all components into single B-Rep solid
- **`core/systems.py`**: `Propulsion` (abstract), `LycomingO235`, `ElectricEZ` with firewall geometry, W&B, thrust calculations

### 3. Analysis Layer (`core/simulation/`, `core/analysis.py`)

- **`core/analysis.py`** (~1,260 LOC): `PhysicsEngine`, `OpenVSPRunner`, `VSPBridge`, `WeightBalance`
  - CG envelope calculation, neutral point, static margin
  - Canard stall priority checking (safety critical)
  - Downwash model (canard-to-wing)
  - Reynolds number effects on CLmax
  - OpenVSP native model export (.vsp3)
- **`core/simulation/openvsp_adapter.py`**: Lightweight surrogate for CI without OpenVSP
- **`core/simulation/fea_adapter.py`**: Euler-Bernoulli beam analysis, composite CLT (ABD matrices), Tsai-Wu failure criterion, panel buckling (NACA TN 3781), torsional stiffness (Bredt-Batho), flutter estimation (14 CFR 23.629)
- **`core/simulation/regression.py`**: Physics regression test runner comparing current values against stored baselines

### 4. Manufacturing Layer (`core/manufacturing.py`, `core/nesting.py`)

- **`core/manufacturing.py`** (~1,420 LOC): `GCodeWriter` (4-axis hot-wire XYUV paths), `JigFactory` (3D-printable assembly aids), `GCodeConfig`, `CutPath`, `HotWireProcess`
  - Kerf compensation per foam type
  - Root/tip wire synchronization
  - Lead-in/lead-out paths
  - Skin thickness deduction from foam core
  - Manufacturing plans per component
- **`core/nesting.py`**: `NestingPlanner` (shelf-based bin packing for DXF outlines on stock sheets)
  - Grain/fiber orientation constraints
  - Dogbone and fillet corner relief
  - Label engraving layers
  - CSV manifest generation

### 5. Compliance Layer (`core/compliance/`)

- **`core/compliance/__init__.py`**: `ComplianceTracker` (FAA Form 8000-38 credit tally)
  - 21 standard Long-EZ build tasks with credit weights
  - Manufacturing method tracking (builder manual, builder CNC, helper, commercial)
  - Markdown report and JSON export
- **`core/compliance/tracker.py`**: `ComplianceTaskTracker` bridges CAD generation events to compliance credits; generates running checklists and layup schedules

### 6. Support Modules

- **`core/atmosphere.py`**: ISA Standard Atmosphere (troposphere) -- density, pressure, temp, viscosity, speed of sound
- **`core/metadata.py`**: Artifact provenance (git revision, config hash, contributor) written as `.metadata.json` sidecars
- **`core/vsp_integration.py`**: Formal OpenVSP bridge with native/surrogate modes

## Data Flow

```
.dat airfoil files
       |
       v
AirfoilFactory.load() --> Airfoil (processed coordinates)
       |
       v
WingGenerator / CanardGenerator (station-based lofting)
       |
       v
CadQuery Workplane (B-Rep solid)
       |
       +---> STEP export (CAM software)
       +---> STL export (3D printing jigs)
       +---> DXF export (laser cutting templates)
       +---> G-code export (hot-wire CNC foam cutting)
       +---> compliance credit recording
       +---> metadata sidecar (.metadata.json)
```

## Key Entry Points

| Entry Point | Purpose |
|-------------|---------|
| `main.py` | CLI dispatcher for all generation/analysis |
| `scripts/smoke_test.py` | Lightweight CI validation |
| `scripts/produce_final_package.py` | Timestamped prototype output package |
| `scripts/compliance_audit.py` | FAA compliance report generation |
| `scripts/assembly_test.py` | Full aircraft assembly test |
| `scripts/generate_canard.py` | Standalone canard generation |

## Lazy Loading

`core/__init__.py` uses `__getattr__` with a `_LAZY_IMPORTS` dict to avoid importing heavy CAD dependencies when only compliance utilities are needed (e.g., generating FAA paperwork on a headless host).
