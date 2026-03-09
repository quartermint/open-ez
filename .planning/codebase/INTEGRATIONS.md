# External Integrations

## OpenVSP (NASA Vehicle Sketch Pad)

- **Type**: Optional external aerodynamic solver
- **Integration**: `core/vsp_integration.py` (formal bridge), `core/simulation/openvsp_adapter.py` (lightweight adapter), `core/analysis.py` (OpenVSPRunner class)
- **Detection**: Runtime `import openvsp` with try/except; graceful fallback to surrogate models
- **Data flow**: PDE config -> VSP3 JSON metadata -> OpenVSP native API (when available) -> VSPAERO polars
- **Surrogate mode**: `OpenVSPAdapter` synthesizes aerodynamic polars using lifting-line theory and config-driven heuristics when OpenVSP is not installed
- **Outputs**: `.vsp3.json` metadata files, VSPAERO polar JSON, `.vspscript` exports

## CadQuery / OpenCASCADE

- **Type**: Core geometry engine (required)
- **Integration**: Direct Python API calls throughout `core/base.py`, `core/structures.py`, `core/manufacturing.py`, `core/assembly.py`
- **Operations**: B-Rep solid creation, lofting, boolean operations (cut/union), wire/face generation, STEP/STL/DXF export
- **Pattern**: All geometry flows through `cq.Workplane` objects; components inherit `AircraftComponent` which wraps CadQuery

## ezdxf

- **Type**: DXF file I/O (required)
- **Integration**: `core/nesting.py` for reading existing DXF outlines and writing nested sheet layouts
- **Operations**: Read DXF bounding boxes (`bbox.extents`), create new DXF documents, copy/translate/rotate entities, add layers (STOCK, ENGRAVE_LABELS, GRAIN_DIRECTION, SHEET_GRAIN, DOGBONE, FILLET)

## SciPy / NumPy

- **Type**: Numerical computing (required)
- **Integration**: `core/aerodynamics.py` for airfoil processing, `core/simulation/fea_adapter.py` for structural analysis
- **Key uses**:
  - `CubicSpline` for airfoil coordinate interpolation
  - `savgol_filter` for digitization noise removal
  - `np.ndarray` operations for coordinate transforms, CLT matrix math
  - `interp1d` for airfoil blending across span

## Git

- **Type**: Version control metadata
- **Integration**: `core/metadata.py` calls `git rev-parse --short HEAD` to stamp artifact provenance
- **Usage**: Every exported STEP/STL/DXF/G-code file gets a `.metadata.json` sidecar with git revision, config hash, and contributor info

## File System

- **Input data**: `data/airfoils/*.dat` (UIUC format airfoil coordinates)
- **Validation data**: `data/validation/openvsp_validation.json`, `tests/snapshots/physics_baseline.json`, `tests/fixtures/reference_data.json`
- **Output tree**: `output/{STEP,STL,DXF,GCODE,VSP,nested,docs,reports,compliance,assembly_test}/`
- **Prototype packages**: Timestamped directories under `output/prototype_package_YYYYMMDD_HHMMSS/`

## Environment Variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `PDE_CONTRIBUTOR` | Contributor name stamped on artifact metadata | `"unknown"` |

## No External APIs / Services

- No cloud services, databases, or network calls
- Fully offline, local-only tool
- No authentication or secrets
