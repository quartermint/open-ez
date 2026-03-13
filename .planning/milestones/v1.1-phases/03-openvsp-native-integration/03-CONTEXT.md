# Phase 3: OpenVSP Native Integration - Context

**Gathered:** 2026-03-10
**Status:** Ready for planning

<domain>
## Phase Boundary

Install real OpenVSP Python bindings on MacBook, implement `_run_native_sweep()` in `vsp_integration.py` to build Long-EZ geometry and run VSPAERO VLM producing real CL/CD/CM polars, and wire into the `main.py --generate-all` pipeline with CI-safe surrogate fallback. Cross-validation of surrogate vs real VSPAERO is Phase 4 scope (VSP-03).

</domain>

<decisions>
## Implementation Decisions

### Installation Approach
- OpenVSP Python bindings installed via `conda` (NASA distributes via conda-forge channel)
- Document in an `environment.yml` or install script at project root — no pip package exists for openvsp
- Version pinned in install script (whatever current stable is at time of research)
- Verify `import openvsp` and `vsp.GetVSPVersion()` succeed as installation smoke test

### Pipeline Trigger Behavior
- `main.py --generate-all` auto-detects OpenVSP and uses real VSPAERO when available
- Falls back to `OpenVSPAdapter` surrogate transparently when `import openvsp` fails
- No new CLI flag needed — the existing `has_vsp` pattern in `VSPIntegration` already handles detection
- Pipeline prints which mode is active: "Using native VSPAERO" or "Using surrogate (OpenVSP not installed)"

### Polar Output & Storage
- Real VSPAERO polars written to `data/validation/vspaero_native_polars.json` (new file, separate from surrogate cache)
- Schema includes: `source: "vspaero_native"`, alpha sweep points with CL/CD/CM, solver settings (VLM, wake iterations, Mach), OpenVSP version, timestamp
- Surrogate polars remain in existing `openvsp_validation.json` — Phase 4 cross-validates both files side-by-side (VSP-03)
- Both files use the same `AerodynamicPoint` schema (alpha_deg, cl, cd, cm) for easy comparison

### Module Consolidation
- `vsp_integration.py` becomes the primary bridge: detect OpenVSP → native sweep or surrogate sweep
- `export_native_vsp3()` geometry builder code from `openvsp_runner.py` moves into `vsp_integration.py` as the native geometry builder for VSPAERO
- `openvsp_adapter.py` stays as-is — it's the clean surrogate with lifting-line polars
- `openvsp_runner.py` simplified to orchestration only (run_validation, cache management) — delegates geometry/solver to `vsp_integration.py`
- Net: 2 modules with clear roles (bridge+native in `vsp_integration.py`, surrogate in `openvsp_adapter.py`) + runner for orchestration

### VSPAERO Solver Configuration
- VLM (Vortex Lattice Method) — not panel method. Sufficient for preliminary design, fast
- Alpha sweep: -4° to 14° in 1° steps (19 points) — matches surrogate range, gives good polar resolution
- Mach number: 0.0 (incompressible, appropriate for Long-EZ speeds ~130-180 kts)
- Wake iterations: solver default (typically 5) — research should confirm optimal value
- Symmetry: half-model with Y-symmetry plane to halve solve time

### Surrogate Fallback Strategy
- Fallback triggers on `ImportError` (openvsp not installed) OR on VSPAERO runtime failure (solver crash, timeout)
- If native solver fails at runtime, log warning with error details and fall back to surrogate — never crash the pipeline
- Existing tests all pass without OpenVSP installed (surrogate path exercised)
- New tests for native path use `@pytest.mark.skipif(not HAS_OPENVSP)` guard

### Claude's Discretion
- Exact conda environment setup and version pinning details
- Whether to create a `Makefile` target for OpenVSP install or a standalone script
- Number of spanwise/chordwise panels in VLM mesh (research should determine good defaults)
- Whether `OpenVSPRunner` class gets refactored or just gets updated method implementations
- Error handling granularity for VSPAERO subprocess failures

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `vsp_integration.py:VSPIntegration` — has `has_vsp` property, `_try_import_vsp()`, `run_aerodynamic_sweep()` dispatch. `_run_native_sweep()` is the FIXME placeholder to implement.
- `openvsp_runner.py:export_native_vsp3()` (lines 282-363) — already builds wing, canard, winglets, fuselage with control surfaces via OpenVSP API. This geometry code is the foundation for the native VSPAERO path.
- `openvsp_adapter.py:OpenVSPAdapter` — clean surrogate with `run_vspaero()` returning `List[AeroPolar]`. Lifting-line + Viterna post-stall. Already uses proper induced drag formula.
- `openvsp_runner.py:AerodynamicPoint` and `TrimSweepResult` dataclasses — reuse for native polar output schema
- `openvsp_runner.py:_write_cache()` and `_load_cached_results()` — cache pattern to extend for native polars

### Established Patterns
- SSOT config: all geometry from `config/aircraft_config.py`, accessed via `from config import config`
- Runtime detection: `try: import openvsp except ImportError:` pattern already in both `vsp_integration.py` and `openvsp_runner.py`
- Cache files in `data/validation/` with JSON format and metadata headers
- Surrogate fallback: existing pattern throughout codebase (D-box alongside cap-only in Phase 2)
- CadQuery mocking in tests: `sys.modules.setdefault("cadquery", MagicMock())` — similar pattern needed for openvsp in surrogate-path tests

### Integration Points
- `main.py:80-89` — already calls `runner.export_native_vsp3()` and could call `run_aerodynamic_sweep()` for polars
- `vsp_integration.py:107-110` — `run_aerodynamic_sweep()` dispatches to native or surrogate
- `openvsp_runner.py:124-148` — `run_validation()` is the entry point for cached sweep execution
- `core/analysis.py` — re-exports `OpenVSPRunner` for backward compatibility
- `core/__init__.py` — lazy imports for VSP classes

</code_context>

<specifics>
## Specific Ideas

- `export_native_vsp3()` already has geometry code that maps config to OpenVSP API calls — this is the starting point for building the VSPAERO analysis model, not from scratch
- The existing surrogate (`OpenVSPAdapter`) is the cross-validation baseline for Phase 4 (VSP-03) — must be preserved exactly as-is so discrepancies can be measured
- `openvsp` is NOT currently installed on the MacBook — installation verification is literally success criterion #1

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 03-openvsp-native-integration*
*Context gathered: 2026-03-10*
