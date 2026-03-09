# Concerns & Technical Debt

## Safety-Critical Concerns

### 1. Simplified Stability Check in Config Validation
- **File**: `config/aircraft_config.py` lines 624-632
- **Issue**: The `validate()` method has a placeholder stability check (`canard_loading = 1.0; wing_loading = 1.0`) that always passes. This means the config validation never actually checks canard-first stall priority.
- **Risk**: HIGH -- builder could modify geometry parameters that break stall margin without config warning
- **Fix**: Wire in actual lift coefficient comparison from `PhysicsEngine` or at minimum compare `canard_clmax / canard_area` vs `wing_clmax / wing_area`

### 2. Surrogate Aerodynamic Models
- **Files**: `core/simulation/openvsp_adapter.py`, `core/analysis.py`
- **Issue**: When OpenVSP is not installed (which is the common case), all aerodynamic analysis uses simplified heuristics (lifting-line theory, linear CL models). These are reasonable approximations but not validated against wind tunnel data for the specific Long-EZ configuration.
- **Risk**: MEDIUM -- physics regression tests pass against their own baselines but baselines are synthetic
- **Fix**: Validate surrogate outputs against known Long-EZ flight test data; document confidence intervals

### 3. G-code Not Physically Validated
- **File**: `core/manufacturing.py`
- **Issue**: G-code generation is untested on actual CNC hardware. Kerf compensation, feed rates, and wire temperature parameters are theoretical values from documentation.
- **Risk**: HIGH for manufacturing -- incorrect G-code could produce parts with wrong dimensions
- **Fix**: Run test cuts on representative foam stock and compare measured vs. intended profiles

## Structural Concerns

### 4. WeightItem Defined in Two Places
- **Files**: `core/systems.py` line 35 and `core/analysis.py`
- **Issue**: `WeightItem` dataclass is defined in both `core/systems.py` and `core/analysis.py`. The `systems.py` definition shadows the import that would come from `analysis.py`, creating potential inconsistency.
- **Risk**: LOW -- both definitions are functionally identical, but divergence is likely over time
- **Fix**: Remove duplicate from `core/systems.py`, use import from `core/analysis.py`

### 5. Circular Import Risk
- **Files**: `core/structures.py` imports from `core/manufacturing.py`; `core/manufacturing.py` TYPE_CHECKING-imports from `core/structures.py`
- **Issue**: Managed via `TYPE_CHECKING` and `from __future__ import annotations` but fragile. Adding runtime references could break.
- **Risk**: LOW -- currently stable but any refactor needs care

### 6. Strake CG Calculation Inconsistency
- **File**: `core/structures.py` `StrakeGenerator.calculate_cg_contribution()`
- **Issue**: Battery mode calculates module weight with a questionable formula: `cells_per_module * cell_weight / 4` where 4 is the parallel count. This divides total cell weight by 4P but `cells_per_module` is set to 16S, mixing series/parallel in an unclear way.
- **Risk**: MEDIUM -- incorrect battery W&B shifts CG calculation
- **Fix**: Clarify the formula: total cells = series * parallel * modules, total weight = total_cells * cell_weight

## Code Quality Concerns

### 7. Large Monolithic Modules
- **`core/analysis.py`**: ~1,260 LOC -- mixes physics engine, VSP bridge, weight balance, and OpenVSP runner
- **`core/manufacturing.py`**: ~1,420 LOC -- mixes G-code writer, jig factory, and multiple utility classes
- **Fix**: Extract `OpenVSPRunner` and `WeightBalance` into separate modules; split `JigFactory` from `GCodeWriter`

### 8. No Input Validation on Airfoil .dat Files
- **File**: `core/aerodynamics.py` `AirfoilFactory._parse_dat_file()`
- **Issue**: Assumes Selig format (comment on line 491 says "For now, assume Selig format"). No detection of Lednicer vs Selig format despite docstring claiming both are supported.
- **Risk**: LOW -- only two .dat files in use and both are known format
- **Fix**: Implement proper format detection or validate known files at build time

### 9. Hardcoded Values in Non-Config Locations
Despite the SSOT principle, some values are hardcoded:
- `core/structures.py`: Fuselage bulkhead heights (24.0, 38.0, 34.0, 20.0, 8.0) not derived from config
- `core/systems.py`: Engine specifications as class constants (DRY_WEIGHT_LB = 243.0, etc.) rather than config
- `core/simulation/fea_adapter.py`: Gross weight 1425.0 lb hardcoded in `BucklingAnalyzer.standard_load_cases()`
- `core/simulation/fea_adapter.py`: Beam modulus 2.8e6 psi hardcoded in `BeamFEAAdapter.__init__()`
- **Fix**: Move to config or at minimum document as "reference values" with source citations

### 10. No `.gitignore` for Output Directory
- The `output/` directory contains generated JSON, metadata, and prototype packages that appear to be committed
- **Fix**: Add `output/` to `.gitignore` (or add explicit exceptions for reference data)

### 11. `.mypy_cache` Committed
- The `.mypy_cache/` directory is in the repo tree (visible in file listing)
- **Fix**: Add `.mypy_cache/` to `.gitignore`

## Testing Gaps

### 12. No Integration Tests
- No test runs `main.py` end-to-end with all flags
- No test validates that prototype packages contain expected artifacts
- No test validates DXF nesting output

### 13. No Tests for Several Core Modules
Untested modules: `core/nesting.py`, `core/assembly.py`, `core/systems.py`, `core/metadata.py`, `core/vsp_integration.py`

### 14. Physics Baselines Are Self-Referential
- `tests/snapshots/physics_baseline.json` was generated by the same code being tested
- No external validation against flight test data, CFD results, or textbook values

## Performance Concerns

### 15. No Caching of Expensive Geometry Operations
- Each `generate_geometry()` call rebuilds from scratch (CadQuery loft operations)
- The airfoil factory caches parsed coordinates but not CadQuery wire objects
- Not a current blocker but will matter for interactive workflows

## Documentation Concerns

### 16. Stale Planning Documents
- `implementation_plan.md`, `sprint_backlog.md`, `swarm_strategy.md`, `agents.md` may be outdated relative to actual code state
- No mechanism to keep planning docs synchronized with implementation
