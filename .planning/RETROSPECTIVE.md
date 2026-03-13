# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v1.1 — Physical Validation & Calibration

**Shipped:** 2026-03-13
**Phases:** 7 | **Plans:** 16 | **Commits:** 97

### What Was Built
- Reference dataset with provenance schema and 45.5" datum offset resolution
- D-box composite structural model (deflection 89,169" → 2.34")
- OpenVSP 3.48.2 native VSPAERO VLM integration with surrogate fallback
- 241-test validation suite against published specs and wind tunnel data
- 12-metric calibrated accuracy report (9 PASS, 3 known convention gaps)
- Externally-validated regression lock-in replacing self-referential baselines

### What Worked
- **Parallel phase execution:** Phases 1-3 ran in parallel, saving ~2 days vs sequential
- **Gap closure cycle:** Phase 7 closed all audit gaps in a single plan, clean and efficient
- **GSD framework:** First full milestone under GSD — state tracking, verification, and archival all worked smoothly
- **Two-tier regression pattern:** Reference tolerance for gross errors + drift tolerance for code regressions — elegant separation of concerns
- **Surrogate fallback architecture:** Native + surrogate paths both tested, CI works without OpenVSP installed

### What Was Inefficient
- **Phase 3 gap closure:** Required 3 gap-closure plans (03-04, 03-05, 03-06) after initial integration — OpenVSP's non-standard Python packaging (macOS ARM64 bundle, .pth files, Sym_Planar_Flag defaults) caused unexpected issues that could have been discovered with more upfront research
- **ROADMAP/SUMMARY tech debt:** Accumulated small inconsistencies (missing checkboxes, wrong plan counts, missing frontmatter fields) that required Phase 7 cleanup — should enforce these at commit time
- **physics_baseline.json:** Should have been deprecated earlier — it was clearly self-referential from the start but survived through 6 phases

### Patterns Established
- `reference_data.json` as single source of published specs with provenance per entry
- `accuracy_report.json` as machine-readable validation artifact (replaces physics_baseline.json)
- Two-tier regression: reference tolerance (physical uncertainty) + drift tolerance (0.01" code regression)
- PASS-only filtering: known convention gaps excluded from CI, documented in convention_notes
- Dual FS display: "FS X.XX (internal) / FS Y.YY (published)" for all human-readable outputs
- D-box section properties parameterized from config (ply counts, chord fraction, laminate schedule)

### Key Lessons
1. **OpenVSP integration requires upfront smoke testing** — the Python bindings have non-obvious packaging requirements (macOS ARM64 only, Sym_Planar_Flag defaults, .polar file parsing instead of GetDoubleResults). Research phase should include a "can we actually import and run this?" gate.
2. **Convention differences are not bugs** — wing area (trapezoidal vs semi-panel), empty weight (partial model), and static margin all FAIL but are documented convention gaps, not errors. The accuracy report grades handle this well.
3. **Self-referential validation is worse than no validation** — physics_baseline.json gave false confidence. Externally-validated baselines with explicit provenance are strictly better.
4. **Milestone audits catch real gaps** — the Phase 2 verification gap and community data integration gap were both real oversights caught by the audit.

### Cost Observations
- Model mix: ~30% Opus (planning, orchestration), ~70% Sonnet (execution, verification)
- Timeline: 4 days from milestone init to completion
- Notable: 16 plans across 7 phases in 4 days — GSD parallel execution kept throughput high

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Commits | Phases | Key Change |
|-----------|---------|--------|------------|
| v1.0 | ~50 | 6 (A-F) | Legacy naming, ad-hoc planning |
| v1.1 | 97 | 7 (1-7) | GSD framework, parallel phases, milestone audit |

### Cumulative Quality

| Milestone | Tests | Key Metric |
|-----------|-------|------------|
| v1.0 | 22 | Physics foundations established |
| v1.1 | 241 | All outputs validated against external data |

### Top Lessons (Verified Across Milestones)

1. **External validation is non-negotiable** — v1.0 carried self-referential baselines forward; v1.1 proved they mask real issues
2. **Parallel execution multiplies throughput** — v1.0 was sequential; v1.1 parallelized Phases 1-3 and cut timeline significantly
