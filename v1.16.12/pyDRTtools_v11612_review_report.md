# pyDRTtools v1.16.12 — Review Report

**Reviewer:** Rahul Pottekkat Raju  
**Date:** 7 July 2026  
**Assigned by:** Prof. Francesco Ciucci  
**Archive reviewed:** `pyDRTtools_1.16.12.zip`  
**GitHub repository:** https://github.com/rahulpottekkat-raju/pyDRTtools-review

---

## 1. System Information

| Item | Details |
|------|---------|
| Operating System | Windows 11, 1920×1080 (Full HD) |
| Python | 3.11.15 |
| Environment | `pydrt-review5` (fresh conda environment, pip-based install) |
| Install method | `pip install -e ".[test,gui]"` via pyproject.toml |
| NumPy | 2.4.6 |
| SciPy | 1.17.1 |
| pandas | 3.0.3 |
| cvxopt | 1.3.3 |
| PyQt5 | 5.15.11 |
| matplotlib | 3.11.0 |
| pytest | 9.1.1 |
| Qt available | Yes |

---

## 2. Environment Setup Note

No environment YAML file was found in the archive. The professor requested using a YAML file for environment setup. The archive contains only a CI workflow file under `.github/workflows/ci.yml`. I therefore used `pyproject.toml` as the environment definition, which matches the CI setup exactly:

```
conda create -n pydrt-review5 python=3.11 -y
conda activate pydrt-review5
cd "C:\...\pyDRTtools_1.16.12\pyDRTtools_1.16.12"
pip install -e ".[test,gui]"
pip install jupyter nbconvert ipykernel
```

**Key improvement vs previous conda-based approach:** The pip install uses OpenBLAS-linked numpy instead of MKL, eliminating the Intel/LLVM OpenMP conflict that caused 3 additional test failures in previous reviews. This reduced failures from 5-6 to only 2.

---

## 3. Commands Run

```
conda create -n pydrt-review5 python=3.11 -y
conda activate pydrt-review5
pip install -e ".[test,gui]"
pip install jupyter nbconvert ipykernel
set PYTHONPATH=%CD%
python -m compileall -q pyDRTtools tests
python -m pyDRTtools.cli --version
python -m pytest -m "not slow and not gui and not tutorial and not notebook" -q
python -m pytest -m gui -q
python tutorial/generate_artificial_eis_data.py
python tutorial/run_recovery_checks.py
python tutorial/run_recovery_checks.py --slow
jupyter nbconvert --to notebook --execute tutorial/tutorial-pyDRTtools.ipynb --output tutorial-pyDRTtools-executed.ipynb
python launch.py
```

---

## 4. Archive Cleanliness

| Item | Status | Notes |
|------|--------|-------|
| `.git` | ✅ Clean | Not found |
| `__MACOSX` | ✅ Clean | Fixed vs v1.16.7 — no longer present |
| `.DS_Store` | ✅ Clean | Not found |
| `.pytest_cache` | ✅ Clean | Not found |
| `.ruff_cache` | ✅ Clean | Not found |
| `dist/`, `build/` | ✅ Clean | Not found |
| `.github/workflows` | ⚠️ Present | CI configuration only — acceptable |
| `__pycache__` | ❌ Present | Found in `pyDRTtools/` and `tests/` |
| `.pyc` files | ❌ Present | Multiple compiled Python files found |
| `AGENTS.md` | ❌ Present | AI agent orchestration file — should not be in release archive |

**Result:** Two persistent cleanliness issues remain: `__pycache__`/`.pyc` files and `AGENTS.md`. The `__MACOSX` folder from v1.16.7 is now fixed. ✅

---

## 5. Version Consistency

```
python -m pyDRTtools.cli --version
pyDRTtools 1.16.12
```

**Result:** ✅ Version reported consistently as `1.16.12`.

Compile check:
```
python -m compileall -q pyDRTtools tests
(no output — clean)
```

---

## 6. Automated Tests

### 6.1 Core Tests

```
python -m pytest -m "not slow and not gui and not tutorial and not notebook" -q
2 failed, 1145 passed, 97 deselected, 10 warnings in 146.30s
```

**Result:** ✅ Only 2 failures — significant improvement over previous versions (5-6 failures). Both are Windows path separator issues.

| Test | Result | Reason |
|------|--------|--------|
| `test_batch_json_value_freezes_json_safe_recursion` | ❌ FAIL | Windows backslash `\` vs forward slash `/` in path serialization |
| `test_api_metadata_value_freezes_metadata_recursion` | ❌ FAIL | Same backslash vs forward slash issue |

**Tests fixed vs previous versions (by switching to pip/OpenBLAS):**
- ✅ `test_bht_exact_zarc_downsampled_scores_only_by_default` — now passing
- ✅ `test_lc_score_matches_legacy_cholesky_inverse_formula` — now passing
- ✅ `test_simple_run_exact_synthetic_boundary_warning_is_finite` — now passing

### 6.1.1 Full Tracebacks for Failed Tests

**Failure 1 — JSON path serialization (Windows-specific)**
```
FAILED tests/test_service_boundaries.py::test_batch_json_value_freezes_json_safe_recursion
AssertionError: assert 'nested\\file.txt' == 'nested/file.txt'
```
**Cause:** Windows `pathlib.Path` uses backslash separators producing `nested\file.txt` instead of expected `nested/file.txt`.

---

**Failure 2 — JSON path serialization (Windows-specific)**
```
FAILED tests/test_service_boundaries.py::test_api_metadata_value_freezes_metadata_recursion
AssertionError: assert 'nested\\file.txt' == 'nested/file.txt'
```
**Cause:** Same backslash vs forward slash issue as Failure 1.

### 6.2 GUI Tests

```
python -m pytest -m gui -q
3 failed, 69 passed in 12.13s
```

**Result:** ⚠️ 3 new GUI test failures not present in v1.16.7.

| Test | Result | Reason |
|------|--------|--------|
| `test_finish_bayesian_run_displays_final_lambda_value` | ❌ FAIL | Bayesian message format changed — now appends "Selected lambda: 1e-07" |
| `test_finish_bayesian_run_warns_modally_when_sampler_retried` | ❌ FAIL | Same message format change |
| `test_finish_bht_run_scores_only_is_success_and_shows_score_plot_without_modal_warning` | ❌ FAIL | Bayesian warning modal no longer shown in this scenario |

**Analysis:** These failures indicate that the Bayesian message format was changed in v1.16.12 but the GUI tests were not updated to match. The tests expect the old message format but the code now produces a longer message with "Selected lambda: 1e-07" appended.

---

## 7. Tutorial and Recovery Checks

### 7.1 Generate Artificial EIS Data

```
python tutorial/generate_artificial_eis_data.py
Wrote 8 spectra to tutorial/data
```

**Result:** ✅ 8 spectra generated successfully.

### 7.2 Recovery Checks (Fast)

```
python tutorial/run_recovery_checks.py
20/20 passed
```

All checks passed:

| Dataset | R_inf error | R_pol error | tau peak error | Residual |
|---------|------------|------------|---------------|---------|
| 1ZARC exact | 0.00183 | 0.00288 | 0.00433 | 0.000347 |
| 1ZARC noisy | 0.00107 | 0.00307 | 0.00680 | 0.00215 |
| 2ZARC0–5 | All within tolerance | All within tolerance | All within tolerance | — |

### 7.3 Recovery Checks (Slow)

```
python tutorial/run_recovery_checks.py --slow
25/25 passed
```

| Check | Result |
|-------|--------|
| Bayesian credible arrays finite | ✅ PASS |
| Bayesian credible intervals ordered (lower ≤ mean ≤ upper) | ✅ PASS |
| BHT clean failure (no hang, no crash) | ✅ PASS |
| BHT scores finite | ✅ PASS |
| BHT DRT unavailable (no misleading curves) | ✅ PASS |

### 7.4 Recovery Artifacts Generated

All required files confirmed under `tutorial/figs/recovery_checks/`:

- `*_drt_exact_vs_recovered.pdf/.png` ✅ (8 datasets)
- `*_nyquist_fit.pdf/.png` ✅ (8 datasets)
- `*_residual.pdf/.png` ✅ (8 datasets)
- `*_drt_curve.csv` ✅ (8 datasets)
- `recovery_metrics.csv` ✅

### 7.5 Tutorial Notebook

```
jupyter nbconvert --to notebook --execute tutorial/tutorial-pyDRTtools.ipynb --output tutorial-pyDRTtools-executed.ipynb
[NbConvertApp] Writing 594833 bytes to tutorial\tutorial-pyDRTtools-executed.ipynb
```

**Result:** ✅ Notebook executed successfully. All cells completed without error.

---

## 8. Bayesian Run

**Result: ❌ NEW FAILURE**

The Bayesian Run now fails on `1ZARC.csv` with a new error not seen in previous versions:

```
Bayesian run Error:
Bayesian_run HMC sampling failed after 1 attempt(s) with
bayesian_sampler_lambda_policy='fail' and kernel chain
('hmc_precision_scaled', 'hmc_precision').
lambda_value_selected=0.00021713693282520533

Original sampler error: 'generate_tmg exceeded the total bounce budget
(max_total_bounces=1000000) after collecting 85 of 999 requested samples
(rejection_count=122, bounce_limit_rejections=122, constraint_rejections=0);
the truncated region may be numerically ill-conditioned; a larger
regularization parameter usually stabilizes the posterior.'
```

**Analysis:** This error indicates that the new HMC (Hamiltonian Monte Carlo) sampler introduced in v1.16.12 fails on this dataset. The sampler exceeds its bounce budget before collecting the required number of samples. This did not occur in v1.15.13 or v1.16.7, which used a different Bayesian sampling approach. The error is shown clearly in the GUI via a modal error dialog — no crash occurs.

Note: The slow recovery check Bayesian test still passes because it uses a different dataset and lambda value.

---

## 9. BHT / Hilbert

- No crash or hang ✅
- Fails cleanly after 20 attempts ✅
- No misleading DRT shown ✅
- GUI correctly shows: "derived DRT display is disabled because no validated BHT DRT estimate is available" ✅
- KK scores displayed correctly ✅
- **New feature:** EIS Data tab now shows Hilbert transform overlay (Z_H in blue) alongside the regressed impedance (Z_μ in black) after BHT run ✅

---

## 10. GUI Testing

### 10.1 Test Results

| # | Test | Result | Screenshot |
|---|------|--------|-----------|
| 01 | Fresh GUI launch | ✅ PASS | `01_gui_launch.png` |
| 02 | Load 1ZARC.csv — Nyquist plot | ✅ PASS | `02_nyquist_1ZARC_loaded.png` |
| 03 | Simple Run — R_inf displayed | ✅ PASS | `03_simple_run_rinf_display.png` |
| 04 | Nyquist fit overlay | ✅ PASS | `04_nyquist_fit_overlay.png` |
| 05 | Bayesian Run — HMC error | ❌ FAIL | `05_bayesian_error_new.png` |
| 06 | BHT/KK scores — DRT disabled | ✅ PASS | `06_bht_kk_scores.png` |
| 07 | BHT EIS Hilbert overlay — new feature | ✨ NEW | `07_bht_eis_hilbert_overlay.png` |
| 08 | New file load — no contamination | ✅ PASS | `08_new_file_no_contamination.png` |
| 09 | Export before run — warning shown | ✅ PASS | `09_export_before_run_warning.png` |
| 10 | Cancel export — no crash | ✅ PASS | `10_export_cancel_no_crash.png` |
| 11 | 2ZARC0 DRT — two peaks | ✅ PASS | `11_2ZARC0_drt_two_peaks.png` |
| 12 | 1ZARC_exact DRT — single peak | ✅ PASS | `12_1ZARC_exact_drt.png` |

### 10.2 GUI Paragraph

The GUI launched cleanly on Windows 11 with no errors. File loading, Simple Run, Nyquist fit overlay, and export behavior all worked correctly as in previous versions. A significant new finding is that the Bayesian Run now fails with an HMC sampler error on 1ZARC.csv — the GUI shows a clear modal error dialog with full diagnostics, and does not crash. The BHT/KK tab correctly disables DRT display and shows KK scores. A new and welcome GUI feature is the Hilbert transform overlay on the EIS Data tab after BHT run — Z_H (blue) is plotted alongside Z_μ (black), giving users a visual consistency check. Export before run triggers a clear warning dialog, and cancelling the export file dialog leaves the GUI stable. Old results are correctly cleared on new file load. Overall the GUI remains robust and scientifically honest, with the exception of the Bayesian Run HMC failure which needs investigation.

---

## 11. Open Issues Table

| Issue | Description | v1.16.7 | v1.16.12 | Evidence |
|-------|-------------|---------|---------|----------|
| #14 | Tutorial notebook fails | ✅ FIXED | ✅ STILL FIXED | Notebook executed successfully |
| #15 | Recovery figures not generated | ✅ FIXED | ✅ STILL FIXED | All 8 datasets generated |
| #17 | High-DPI / 4K display scaling | ⚠️ NOT REPRODUCIBLE | ⚠️ NOT REPRODUCIBLE | 1920×1080 display |
| #18 | Export before run crashes GUI | ✅ FIXED | ✅ STILL FIXED | Warning dialog — screenshot 09 |
| #20 | Cancel export crashes GUI | ✅ FIXED | ✅ STILL FIXED | GUI stable — screenshot 10 |
| #21 | Old results not cleared | ✅ FIXED | ✅ STILL FIXED | Fields cleared — screenshot 08 |
| #22 | Batch processing missing | ✅ FIXED | ✅ STILL FIXED | batch.py present |
| #23 | R_inf not displayed | ✅ FIXED | ✅ STILL FIXED | R_inf shown — screenshot 03 |
| #24 | Export warning missing | ✅ FIXED | ✅ STILL FIXED | Warning dialog — screenshot 09 |
| #25 | EIS pretreatment missing | ✅ FIXED | ✅ STILL FIXED | pretreatment.py present |
| #26 | BHT misleading DRT | ✅ FIXED | ✅ STILL FIXED | DRT disabled — screenshot 06 |

---

## 12. Key Findings

1. **Environment setup improvement:** Using `pip install -e ".[test,gui]"` via pyproject.toml eliminates MKL/OpenMP conflicts from conda environment, reducing test failures from 5-6 to only 2. The professor's suggestion to use the YAML/pyproject.toml approach was validated.

2. **Bayesian Run — NEW FAILURE:** HMC sampler fails on 1ZARC.csv with `generate_tmg exceeded bounce budget`. This is a regression introduced in v1.16.12 — not present in v1.15.13 or v1.16.7. The GUI shows a clear error dialog without crashing.

3. **GUI test failures — 3 new:** Bayesian message format changed in v1.16.12 but GUI tests not updated to match. Test/code mismatch.

4. **New GUI feature:** EIS Data tab now shows Hilbert transform overlay (Z_H) after BHT run — a useful scientific visualization addition.

5. **Archive cleanliness improvement:** `__MACOSX` folder removed vs v1.16.7 ✅. `AGENTS.md` and `__pycache__`/`.pyc` files still present.

6. **All previously fixed issues remain fixed** across all reviewed versions.

7. **Windows path separator:** 2 remaining test failures related to Windows backslash vs forward slash — consistent across all versions, not a scientific issue.

---

## 13. Overall Assessment

pyDRTtools v1.16.12 shows meaningful improvements in test reliability when installed via pyproject.toml (pip), confirming the professor's suggestion about environment setup. The archive is cleaner than v1.16.7 with `__MACOSX` removed. However, a significant regression is present: the Bayesian Run now fails with an HMC sampler error on tutorial datasets, which did not occur in previous versions. Three GUI tests also fail due to a message format mismatch. These issues should be addressed before public release. The Simple Run, BHT, recovery checks, and all other functionality remain robust and scientifically correct.

