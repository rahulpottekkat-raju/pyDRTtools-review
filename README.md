# pyDRTtools v1.10.02 — Review Report

**Reviewer:** Rahul  
**Date:** 15 June 2026  
**Assigned by:** Prof. Francesco Ciucci  

---

## Repository Contents

| File | Description |
|------|-------------|
| `pyDRTtools_review_report.md` | Full review report — system info, test results, issue table, GUI paragraph |
| `screenshots_description.md` | Description of every GUI screenshot with pass/fail result |
| `tutorial-pyDRTtools.ipynb` | Executed Jupyter notebook — all cells ran successfully |
| `01_gui_launch.png` | Fresh GUI on startup |
| `02_nyquist_1ZARC_loaded.png` | Nyquist plot after loading 1ZARC.csv |
| `03_drt_simple_run_gcv.png` | DRT peak at τ=0.1s with GCV regularization |
| `04_nyquist_fit_overlay.png` | Fitted curve overlaid on data |
| `05_export_cancel_no_crash.png` | Export cancel — no crash (issue #21 fixed) |
| `06_bayesian_error_1ZARC_noisy.png` | Bayesian sampler error on noisy data |
| `07_hilbert_transform_wrong_1ZARC_noisy.png` | Hilbert Transform bug — scale 1e-117 (issue #26) |
| `08_hilbert_transform_wrong_1ZARC_exact.png` | Hilbert Transform bug — scale 1e-8 on exact data |
| `09_bayesian_error_1ZARC_exact_GCV.png` | Bayesian fails when GCV selects very small lambda |
| `10_bayesian_working_1ZARC_exact_custom.png` | Bayesian working with custom lambda |
| `11_eis_score_tab.png` | EIS Score bar chart |
| `12_peak_analysis.png` | Peak Analysis — no crash (issue #20 fixed) |
| `13_new_file_loaded_clean.png` | New file loaded after previous run |
| `14a_regularization_custom.png` | DRT with custom regularization |
| `14b_regularization_gcv.png` | DRT with GCV regularization |

---

## Quick Summary

| Category | Result |
|----------|--------|
| Automated tests | 195 passed, 1 failed |
| Recovery checks (fast) | 20/20 passed |
| Recovery checks (slow) | 24/24 passed |
| GUI tests | 15/15 passed |
| Notebook checks | 9/9 passed |

## GitHub Issues Summary

| Status | Issues |
|--------|--------|
| ✅ FIXED | #15, #18, #20, #21, #23, #24 |
| ⚠️ PARTLY FIXED | #26, #14 |
| ❌ NOT FIXED | #22, #25 |
| NOT REPRODUCIBLE | #17 |

---

## Key Findings

1. **Hilbert Transform (#26)** — No longer crashes but produces wrong DRT on all datasets (scales 1e-8 to 1e-117)
2. **Bayesian Run** — Fails when GCV selects very small lambda; works with manual lambda=0.001
3. **Recovery checks** — Do not generate figures or recovery_metrics.csv as described in instructions
4. **Test failure** — One hardcoded value in test suite does not match regenerated 1ZARC.csv
