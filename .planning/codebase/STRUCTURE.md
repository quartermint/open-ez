# Directory Structure

## Project Layout

```
rutan-ez/
|-- .github/workflows/ci.yml       # GitHub Actions CI (lint + geometry validation)
|-- .pre-commit-config.yaml         # ruff, mypy, smoke test hooks
|-- config/
|   |-- __init__.py                 # Re-exports `config` singleton
|   |-- aircraft_config.py          # SSOT: all aircraft parameters (680 LOC)
|-- core/
|   |-- __init__.py                 # Lazy imports to avoid heavy CAD deps
|   |-- aerodynamics.py             # Airfoil processing (545 LOC)
|   |-- analysis.py                 # Physics engine, VSP bridge, W&B (1,259 LOC)
|   |-- assembly.py                 # Full aircraft assembly (132 LOC)
|   |-- atmosphere.py               # ISA standard atmosphere (59 LOC)
|   |-- base.py                     # AircraftComponent, FoamCore, Bulkhead ABCs (319 LOC)
|   |-- manufacturing.py            # GCodeWriter, JigFactory (1,417 LOC)
|   |-- metadata.py                 # Artifact provenance utilities (120 LOC)
|   |-- nesting.py                  # DXF bin-packing optimizer (566 LOC)
|   |-- structures.py               # Wing/Canard/Fuselage/Strake generators (1,002 LOC)
|   |-- systems.py                  # Propulsion systems (759 LOC)
|   |-- vsp_integration.py          # OpenVSP formal bridge (160 LOC)
|   |-- compliance/
|   |   |-- __init__.py             # ComplianceTracker, ManufacturingMethod (413 LOC)
|   |   |-- tracker.py              # ComplianceTaskTracker, layup schedules (268 LOC)
|   |-- simulation/
|       |-- __init__.py             # Package init (16 LOC)
|       |-- fea_adapter.py          # Beam FEA, CLT, buckling, flutter (843 LOC)
|       |-- openvsp_adapter.py      # Lightweight VSP surrogate (159 LOC)
|       |-- regression.py           # Physics regression runner (150 LOC)
|-- data/
|   |-- airfoils/
|   |   |-- eppler_1230_mod.dat     # Main wing airfoil coordinates
|   |   |-- roncz_r1145ms.dat       # Canard airfoil coordinates (SAFETY CRITICAL)
|   |-- validation/
|       |-- openvsp_validation.json # Validation reference data
|-- scripts/
|   |-- assembly_test.py            # Full aircraft assembly test (40 LOC)
|   |-- compliance_audit.py         # FAA compliance report (62 LOC)
|   |-- generate_canard.py          # Standalone canard generation (122 LOC)
|   |-- manufacturing_test.py       # Manufacturing output test (48 LOC)
|   |-- produce_final_package.py    # Timestamped prototype package (95 LOC)
|   |-- run_ci_checks.py            # CI check runner (39 LOC)
|   |-- smoke_test.py               # Geometry smoke test (172 LOC)
|   |-- validate_metadata.py        # Metadata validation (93 LOC)
|-- tests/
|   |-- __init__.py
|   |-- fixtures/reference_data.json
|   |-- snapshots/physics_baseline.json
|   |-- test_airfoil_washout.py     # Washout transform tests (166 LOC)
|   |-- test_beam_deflection.py     # Beam FEA tests (273 LOC)
|   |-- test_canard_stall_and_downwash.py  # Safety-critical stall tests (148 LOC)
|   |-- test_compliance_gate.py     # FAA compliance gate tests (96 LOC)
|   |-- test_lift_curve_theory.py   # Lift curve slope tests (210 LOC)
|   |-- test_manufacturing_accuracy.py  # Manufacturing tolerance tests (170 LOC)
|   |-- test_openvsp_runner.py      # OpenVSP runner tests (46 LOC)
|   |-- test_physics_regression.py  # Physics regression tests (21 LOC)
|   |-- test_ssot_weights.py        # Weight & balance SSOT tests (234 LOC)
|   |-- test_torsion_flutter.py     # Torsion/flutter tests (163 LOC)
|-- output/                         # Generated artifacts (gitignored contents)
|   |-- assembly_test/
|   |-- compliance/
|   |-- docs/
|   |-- prototype_package_*/        # Timestamped output bundles
|   |-- test_mfg/
|   |-- vsp/
|-- main.py                         # CLI entry point (363 LOC)
|-- mypy.ini                        # Type checking config
|-- requirements.txt                # Production dependencies
|-- requirements-dev.txt            # Dev dependencies
|-- CLAUDE.md                       # AI assistant instructions
|-- README.md                       # Project README
|-- REVIEW_PROMPT.md                # Code review prompt template
|-- agents.md                       # AI agent configuration
|-- idea.md                         # Original project idea
|-- implementation_plan.md          # Implementation roadmap
|-- sprint_backlog.md               # Sprint tracking
|-- swarm_strategy.md               # Multi-agent strategy
|-- vision.md                       # Project vision document
|-- LICENSE                         # License file
```

## LOC Summary

| Area | Files | Lines |
|------|-------|-------|
| Core modules | 14 | ~5,860 |
| Config | 2 | ~700 |
| Tests | 11 | ~1,530 |
| Scripts | 8 | ~670 |
| Entry point | 1 | ~360 |
| **Total** | **36** | **~11,530** |

## Key Locations

| What | Where |
|------|-------|
| All aircraft dimensions | `config/aircraft_config.py` |
| Base class for all components | `core/base.py` -> `AircraftComponent` |
| Airfoil .dat files | `data/airfoils/` |
| Physics regression baselines | `tests/snapshots/physics_baseline.json` |
| Generated artifacts | `output/` |
| CI pipeline | `.github/workflows/ci.yml` |
| Pre-commit hooks | `.pre-commit-config.yaml` |

## Naming Conventions

- **Modules**: snake_case (`aircraft_config.py`, `fea_adapter.py`)
- **Classes**: PascalCase (`WingGenerator`, `ComplianceTracker`, `BeamFEAAdapter`)
- **Constants**: UPPER_SNAKE_CASE (`REQUIRED_CREDIT`, `DATA_DIR`, `AIRFOIL_FILES`)
- **Config singleton**: `config` (lowercase, imported from `config` package)
- **Module-level instances**: lowercase (`airfoil_factory`, `compliance_tracker`, `physics`, `vsp_bridge`)
- **Test files**: `test_<domain>.py` (pytest discovery pattern)
- **Output artifacts**: `{component_name}.{format}` with `.metadata.json` sidecars
