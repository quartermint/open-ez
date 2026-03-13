# Phase 1: Reference Data & Datum Resolution - Context

**Gathered:** 2026-03-09
**Status:** Ready for planning

<domain>
## Phase Boundary

Curate published Long-EZ reference data with source citations into `reference_data.json` and resolve the ~51" FS datum offset so computed NP/CG values translate correctly to published values. Community build data (weights, CG) is captured with provenance. No new analysis capabilities — this phase provides the external truth that Phase 4+ validates against.

</domain>

<decisions>
## Implementation Decisions

### Reference Data Schema
- Nested by category at top level: `aircraft_specs`, `airfoil_data`, `community_builds`
- Full provenance per entry: `value`, `units`, `source_id`, `page_table`, `uncertainty_pct`, `tolerance_abs`, `tolerance_pct`, `notes`
- Include validation tolerances now (Phase 4 tests read from reference_data.json, not hardcoded)
- File location: `data/validation/reference_data.json` (next to existing openvsp_validation.json)
- Top-level `sources` registry maps source IDs to full bibliographic entries (title, author, year, URL) — data entries reference source IDs instead of repeating citation text

### Datum Offset Implementation
- `datum_offset_in` as a field on `GeometricParams` in `config/aircraft_config.py`
- `to_published_datum(fs_value)` as a method on `GeometricParams`
- Simple subtraction: `published_fs = internal_fs - datum_offset_in`
- Exact offset value derived by research (RAF CP-29/CP-31 published NP location), locked by test
- Test verifies `to_published_datum(153.5)` lands in ~FS 102-114 range

### Dual FS Display
- Analysis outputs show both coordinate systems: "NP: FS 153.5 (internal) / FS 102.5 (published)"
- `StabilityMetrics.summary()` and similar report methods updated to show dual values
- Internal values remain primary in code; published values added for human-readable output

### Source Citations
- Page/table/figure level citations for RAF publications (e.g., "RAF CP-31, Table 2, p.14")
- Wind tunnel data: key parameters only (CLmax, Cm0, alpha_0L, Cd_min) with Reynolds number, facility, and report reference — no full polar curves in reference_data.json
- Confidence tiers per entry: `"published"` (RAF/NACA), `"measured"` (wind tunnel), `"community"` (builder reports)

### Community Data Handling
- Focus on weights & CG only for Phase 1 (empty weight, CG location, component weights from builder weigh-ins)
- Store all conflicting values as arrays with individual provenance — nothing discarded
- Confidence tier: `"community"` — Phase 4 can apply wider tolerances for this tier

### Claude's Discretion
- Builder identification approach (anonymized IDs vs public N-numbers) — pick what's appropriate per source
- Exact JSON schema field ordering and nesting details
- How many community data points to include (minimum viable for validation)

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- `config/aircraft_config.py:GeometricParams` — where `datum_offset_in` and `to_published_datum()` will be added
- `data/validation/openvsp_validation.json` — existing validation data pattern to follow
- `data/airfoils/*.dat` — Roncz R1145MS and Eppler 1230 mod already present (but no wind tunnel metadata)
- `core/analysis.py:StabilityMetrics` — has `summary()` method to update with dual FS display

### Established Patterns
- SSOT config: all dimensions derive from `config/aircraft_config.py`, imported as `from config import config`
- Dataclass fields with `@property` computed values — `datum_offset_in` follows this pattern
- JSON output in `data/validation/` with metadata headers

### Integration Points
- `core/analysis.py:220` — `calculate_neutral_point()` returns FS ~153.5 (internal), needs `to_published_datum()` in reports
- `core/analysis.py:362-366` — `StabilityMetrics` construction, `summary()` method at line 65
- `core/analysis.py:380-386` — CG envelope uses NP, should show published values
- `main.py` — generates analysis reports, will need to use dual display

</code_context>

<specifics>
## Specific Ideas

- The ~51" offset is a known issue from Phase F1 investigation — traced to `fs_wing_le=133` possibly using a different zero reference than published Long-EZ plans
- AR discrepancy (published 7.3 vs computed 6.34) is reference area convention, not a bug — document this in reference_data.json notes
- Existing `physics_baseline.json` is self-referential — Phase 1 reference data is the external truth that eventually replaces it (Phase 6)

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-reference-data-datum-resolution*
*Context gathered: 2026-03-09*
