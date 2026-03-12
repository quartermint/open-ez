---
phase: 4
slug: validation-test-infrastructure-cross-validation
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-11
---

# Phase 4 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | pytest 7.x (already installed) |
| **Config file** | none — pytest discovers from `tests/` directory |
| **Quick run command** | `python3 -m pytest tests/test_precision_validation.py -v` |
| **Full suite command** | `python3 -m pytest tests/ -q` |
| **Estimated runtime** | ~1 second |

---

## Sampling Rate

- **After every task commit:** Run `python3 -m pytest tests/ -q`
- **After every plan wave:** Run `python3 -m pytest tests/ -v`
- **Before `/gsd:verify-work`:** Full suite must be green (xfails expected, not failures)
- **Max feedback latency:** 2 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 04-01-01 | 01 | 1 | VAL-01 | unit/xfail | `python3 -m pytest tests/test_precision_validation.py::test_np_precision_2inch -v` | ❌ W0 | ⬜ pending |
| 04-01-02 | 01 | 1 | VAL-01 | unit/xfail | `python3 -m pytest tests/test_precision_validation.py::test_cg_fwd_limit_precision -v` | ❌ W0 | ⬜ pending |
| 04-01-03 | 01 | 1 | VAL-01 | unit/xfail | `python3 -m pytest tests/test_precision_validation.py::test_cg_aft_limit_precision -v` | ❌ W0 | ⬜ pending |
| 04-01-04 | 01 | 1 | VAL-02 | unit | `python3 -m pytest tests/test_precision_validation.py::test_roncz_clmax_matches_wind_tunnel -v` | ❌ W0 | ⬜ pending |
| 04-01-05 | 01 | 1 | VAL-02 | unit | `python3 -m pytest tests/test_precision_validation.py::test_roncz_alpha_0l_matches_wind_tunnel -v` | ❌ W0 | ⬜ pending |
| 04-01-06 | 01 | 1 | VAL-02 | unit | `python3 -m pytest tests/test_precision_validation.py::test_eppler_clmax_matches_wind_tunnel -v` | ❌ W0 | ⬜ pending |
| 04-01-07 | 01 | 1 | VAL-02 | unit | `python3 -m pytest tests/test_precision_validation.py::test_eppler_alpha_0l_matches_wind_tunnel -v` | ❌ W0 | ⬜ pending |
| 04-01-08 | 01 | 1 | VAL-04 | unit | `python3 -m pytest tests/test_precision_validation.py::test_stall_speed_5pct -v` | ❌ W0 | ⬜ pending |
| 04-01-09 | 01 | 1 | VAL-04 | unit | `python3 -m pytest tests/test_precision_validation.py::test_gross_weight_matches_published -v` | ❌ W0 | ⬜ pending |
| 04-02-01 | 02 | 1 | VSP-03 | manual | `python3.13 scripts/generate_cross_validation.py` | ❌ W0 | ⬜ pending |
| 04-02-02 | 02 | 1 | VSP-03 | smoke | `python3 -m pytest tests/test_precision_validation.py::test_cross_validation_json_exists -v` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `tests/test_precision_validation.py` — precision tests for VAL-01, VAL-02, VAL-04 + VSP-03 schema check
- [ ] `scripts/generate_cross_validation.py` — native polar regeneration + discrepancy table generation (VSP-03)

*Existing infrastructure covers framework needs — no new dependencies required.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Native VSPAERO polar regeneration | VSP-03 | Real OpenVSP only installed on Python 3.13; default runtime is Python 3.14 | Run `python3.13 scripts/generate_cross_validation.py` and verify `vspaero_native_polars.json` has non-mock `vsp_version` |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 2s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
