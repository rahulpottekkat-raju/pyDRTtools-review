# pyDRTtools v1.13.1 — Review Report

| | |
|---|---|
| **Reviewer** | Rahul Pottekkat Raju |
| **Date** | 20 June 2026 |
| **Version Reviewed** | pyDRTtools 1.13.1 |
| **Previous Version** | pyDRTtools 1.10.02 (15 June 2026) |
| **Assigned by** | Prof. Francesco Ciucci |

---

## 1. System Information

| Item | Details |
|------|---------|
| Operating System | Windows 11, 64-bit |
| Screen Resolution | 1920×1080 at 125% Windows scaling |
| Python | 3.11.15 |
| numpy | 2.4.6 |
| scipy | 1.17.1 |
| pandas | 3.0.3 |
| scikit-learn | 1.9.0 |
| matplotlib | 3.10.9 |
| pytest | 9.1.0 |
| cvxopt | 1.3.3 |
| pyqt | 5.15.11 |
| jupyter | 1.1.1 |
| nbconvert | 7.17.0 |
| Qt / GUI available | Yes (PyQt5) |
| Conda environment | pydrt-review2 (fresh) |

---

## 2. Version Check

```
python -m pyDRTtools.cli --version
pyDRTtools 1.13.1
```

Version reported correctly and consistently as `1.13.1`. ✅

---

## 3. Archive Cleanliness Check

Command run:
```
dir /s /b | findstr /i "__pycache__ .pytest_cache .ruff_cache .DS_Store"
dir /s /b *.pyc
```

### ❌ Dirty files found:

| File/Folder | Location |
|-------------|----------|
| `.DS_Store` | Root, pyDRTtools/, scripts/, tests/, tutorial/, tutorial/figs/ |
| `.git` | Root (contains .DS_Store) |
| `.pytest_cache` | Root |
| `__MACOSX` | Outer zip extraction folder |
| `pyDRTtools_1.13.1_stage` | Unexpected staging folder in root |

### ✅ Clean:
- No `.pyc` files
- No `__pycache__` folders
- No `.ruff_cache`
- No Codex/orchestration files

**Conclusion:** Archive is not fully clean. `.DS_Store`, `.git`, `.pytest_cache`, `__MACOSX`, and an unexpected `pyDRTtools_1.13.1_stage` staging folder are present. The professor requested these be absent.

---

## 4. Exact Commands Run

```
conda create -n pydrt-review2 python=3.11
conda activate pydrt-review2
conda install -c conda-forge numpy scipy pandas scikit-learn matplotlib pytest jupyter nbconvert cvxopt pyqt

cd "C:\Users\rahul\OneDrive\Documents\Master- battery materials and technology\hiwi ciucci jun 2026\new code for review 17_06_2026\pyDRTtools_1.13.1\pyDRTtools_1.13.1"
set PYTHONPATH=%CD%

python -m compileall -q pyDRTtools tests
python -m pyDRTtools.cli --version

python -m pytest tests/test_input_validation.py tests/test_simple_run.py tests/test_regression_scientific.py tests/test_runs.py tests/test_bht.py tests/test_peak_analysis.py -v

python -m pytest tests/test_gui_export.py tests/test_GUI.py -v

python tutorial/generate_artificial_eis_data.py
python tutorial/run_recovery_checks.py
python tutorial/run_recovery_checks.py --slow

jupyter notebook
# Opened tutorial/tutorial-pyDRTtools.ipynb -> Kernel -> Restart & Run All

python launch.py
```

---

## 5. Automated Test Results

### Main tests: 211 PASSED, 1 FAILED (vs 180/1 in previous review)

**31 new tests added since v1.10.02.**

### Failure:

```
FAILED tests/test_bht.py::test_bht_exact_zarc_drt_is_validated_or_fails_closed[1ZARC_exact.csv-0.35-0.2]

AssertionError: assert False is True
BHTRunResult(success=False, attempts=20,
message='BHT HT estimation failed after 20 attempts;
last error: attempt 20: RuntimeError:
BHT hyperparameter estimation optimizer failed: ABNORMAL: ')
```

**Cause:** BHT hyperparameter optimizer fails on Windows with `ABNORMAL` error on all 20 attempts for `1ZARC_exact.csv` with PWL kernel. This is a platform-specific numerical issue with the optimizer on Windows. The 2ZARC test passes. This failure is consistent with the previous review finding.

### Notable warnings (improvements):

**GCV boundary warning — now more informative:**
```
RuntimeWarning: optimal_lambda with cv_type='GCV' selected lambda=1e-08
at the lower optimization boundary within lambda_interval=(1e-08, 1e+02);
selected boundary: lower. Next step: Inspect data scaling or provide
a custom lambda/reg_param.
```

**Bayesian floor fix — new and important:**
```
RuntimeWarning: Bayesian sampling increased the regularization parameter
from 0.0001 to 0.001 because the selected value was below the Bayesian
sampling floor 0.001 and very small lambda can make the truncated Gaussian
sampler numerically unstable.
```

### GUI tests: 55 PASSED, 0 FAILED (vs 15/0 in previous review)

**40 new GUI tests added since v1.10.02.**

Key new tests confirmed working:
- `test_finish_simple_run_displays_r_inf_and_lambda` ✅
- `test_importing_new_file_after_run_clears_old_fit_plot` ✅
- `test_repeated_file_load_run_export_cycles_do_not_accumulate_matplotlib_figures` ✅
- `test_export_before_run_requests_qmessagebox_warning` ✅
- `test_bht_drt_canvas_displays_unavailable_message_instead_of_branch_means` ✅

**Total: 266 PASSED, 1 FAILED**

---

## 6. Recovery Checks — Pass/Fail Output

### Fast run: 20/20 PASSED

```
PASS  1ZARC exact: R_inf                 value=0.00183434    expected <= 0.05
PASS  1ZARC exact: R_pol                 value=0.00287532    expected <= 0.20
PASS  1ZARC exact: tau peak              value=0.00432633    expected <= 0.35
PASS  1ZARC exact: impedance residual    value=0.000347356   expected <= 0.06
PASS  1ZARC noisy: R_inf                 value=0.00107277    expected <= 0.05
PASS  1ZARC noisy: R_pol                 value=0.00307029    expected <= 0.20
PASS  1ZARC noisy: tau peak              value=0.00679852    expected <= 0.35
PASS  1ZARC noisy: impedance residual    value=0.0021478     expected <= 0.06
PASS  2ZARC0-5: all R_pol and tau        all passed
Wrote recovery artifacts to tutorial\figs\recovery_checks
```

### Slow run: 25/25 PASSED (vs 24/24 previously — 1 new check added)

```
PASS  Bayesian credible arrays finite      value=finite
PASS  Bayesian credible intervals ordered  value=ordered
PASS  BHT returns status                   value=success
PASS  BHT scores finite                    value=finite
PASS  BHT DRT availability                 value=unavailable/nonvalidated
```

**New check:** `BHT DRT availability` confirms the Hilbert Transform fix — BHT now correctly reports DRT as unavailable instead of showing misleading curves.

---

## 7. Output Files — tutorial/figs/recovery_checks/

**Previously:** This directory was NOT created. Now all 57 files are generated. ✅

Files confirmed present for all 8 cases (1ZARC_exact, 1ZARC_noisy, 2ZARC0–5):

| File type | Status |
|-----------|--------|
| `*_drt_exact_vs_recovered.pdf` | ✅ All 8 |
| `*_drt_exact_vs_recovered.png` | ✅ All 8 |
| `*_nyquist_fit.pdf` | ✅ All 8 |
| `*_nyquist_fit.png` | ✅ All 8 |
| `*_residual.pdf` | ✅ All 8 |
| `*_residual.png` | ✅ All 8 |
| `*_drt_curve.csv` | ✅ All 8 |
| `recovery_metrics.csv` | ✅ Present |

---

## 8. recovery_metrics.csv

```
case,metric,name,value,limit,passed
1ZARC exact,R_inf,1ZARC exact: R_inf,0.001834,<= 0.05 relative error,True
1ZARC exact,R_pol,1ZARC exact: R_pol,0.002875,<= 0.20 relative error,True
1ZARC exact,tau peak,1ZARC exact: tau peak,0.004326,<= 0.35 log10 decades,True
1ZARC exact,impedance residual,1ZARC exact: impedance residual,0.000347,<= 0.06,True
1ZARC noisy,R_inf,1ZARC noisy: R_inf,0.001073,<= 0.05 relative error,True
1ZARC noisy,R_pol,1ZARC noisy: R_pol,0.003070,<= 0.20 relative error,True
1ZARC noisy,tau peak,1ZARC noisy: tau peak,0.006799,<= 0.35 log10 decades,True
1ZARC noisy,impedance residual,1ZARC noisy: impedance residual,0.002148,<= 0.06,True
2ZARC0-5,total R_pol and tau,all cases,all near 0,<= 0.25/0.45,True
```

All 20 metrics passed. ✅

---

## 9. 1ZARC Exact — DRT Recovery Summary

| Metric | Recovered | Expected | Error |
|--------|-----------|----------|-------|
| R_inf | 9.982 Ω | 10.0 Ω | 0.18% |
| R_pol | 50.144 Ω | 50.0 Ω | 0.29% |
| tau peak | ~0.1 s | 0.1 s | 0.004 decades |
| Impedance residual | 0.000347 | ≤ 0.06 | well within |

Recovered DRT plotted directly against exact analytical ZARC DRT in notebook — near-perfect overlap confirmed visually.

---

## 10. Jupyter Notebook Results

All cells passed. Kernel: Restart & Run All.

| Check | Result | Details |
|-------|--------|---------|
| 1. Synthetic spectra correct | ✅ PASS | 8 spectra, correct parameters |
| 2. R_inf ≈ 10 Ω | ✅ PASS | 9.982 Ω, error 0.18% |
| 3. R_pol ≈ 50 Ω | ✅ PASS | 50.144 Ω, error 0.29% |
| 4. DRT peak τ = 0.1 s | ✅ PASS | Error 0.004 decades |
| 5. Exact vs recovered plotted | ✅ PASS | Near-perfect overlap |
| 6. Noisy 1ZARC reasonable | ✅ PASS | Fit and DRT correct |
| 7. 2ZARC peaks at 1e-4 and 0.1 s | ✅ PASS | All 6 variants correct |
| 8. Bayesian finite + ordered | ✅ PASS | All True |
| 9. BHT status, no hang | ✅ PASS | success=True, attempts=1 |

**New notebook sections added:** Section 6 (script check) and Section 7 (GUI checklist) improve usability.

**Note on BHT notebook:** The notebook uses a fixed deterministic initial condition (`FixedThetaRng`) which succeeds in 1 attempt. The automated test with random initialization fails on Windows. The notebook result does not reflect the real-world Windows BHT reliability.

---

## 11. GUI Testing

### All three datasets tested: 1ZARC.csv, 1ZARC_exact.csv, 2ZARC0.csv

| Test | Previous Review | This Review |
|------|----------------|-------------|
| GUI launches cleanly | ✅ | ✅ |
| Nyquist plot + sign convention | ✅ | ✅ |
| Simple Run DRT peak at τ=0.1s | ✅ | ✅ |
| **R_inf displayed in GUI panel** | ❌ Not shown | ✅ FIXED |
| **Regularization parameter used shown** | ❌ Not shown | ✅ FIXED |
| Nyquist fit overlays data | ✅ | ✅ |
| **Bayesian with GCV lambda** | ❌ Sampler rejection | ✅ FIXED |
| **Bayesian warning dialog informative** | ❌ Error only | ✅ FIXED |
| **BHT misleading DRT** | ❌ Wrong scale output | ✅ FIXED |
| **BHT DRT unavailable message** | ❌ No message | ✅ FIXED |
| **Export before run warning** | ❌ Silent | ✅ FIXED |
| Export cancel no crash | ✅ | ✅ |
| **Old results cleared on new file load** | ❌ Contamination | ✅ FIXED |
| **Memory leak on repeated runs** | ❌ Figure leak | ✅ FIXED |
| BHT/KK scores tab | ✅ | ✅ |
| 1ZARC_exact loaded, peak at τ=0.1s | ✅ | ✅ |
| 2ZARC0 loaded, two peaks visible | ✅ | ✅ |
| Batch processing | ❌ | ❌ Still missing |
| EIS pre-treatment | ❌ | ❌ Still missing |
| High-DPI at 125% scaling | ✅ Acceptable | ✅ Acceptable |

### ⚠️ Remaining minor UI issue:
The `Regularization parameter used` and `R_inf / Ohm` display fields in the left panel are too narrow — values are truncated (e.g. `171369321842203` instead of `1.71369e-04`). Scientific notation would be more appropriate for these fields.

---

## 12. GitHub Issue Status

| # | Issue | Previous | This Review | Evidence |
|---|-------|----------|-------------|---------|
| #26 | Hilbert Transform misleading DRT | ⚠️ PARTLY FIXED | ✅ FIXED | GUI now shows "BHT completed but no validated DRT available" dialog. No misleading DRT displayed. New `bht_drt_available()`, `diagnose_bht_drt_display()` functions added. Automated test `test_bht_drt_canvas_displays_unavailable_message` passes. |
| #14 | Bayesian/GCV small lambda | ⚠️ PARTLY FIXED | ✅ FIXED | Bayesian sampler now automatically raises lambda to floor value 0.001 when GCV selects too-small value. GUI shows informative dialog. `success=True` on noisy data. Warning: `Bayesian sampling increased reg param from 0.000217 to 0.001`. |
| #15 | Infinite loop | ✅ FIXED | ✅ STILL FIXED | No loops or hangs in any test. BHT retries terminate cleanly after 20 attempts. |
| #17 | High-DPI | ⚠️ NOT REPRODUCIBLE | ⚠️ NOT REPRODUCIBLE | Tested at 1920×1080 with 125% scaling. GUI readable and correctly sized. 4K display not available. |
| #18 | Jupyter missing strings | ✅ FIXED | ✅ STILL FIXED | All notebook cells ran cleanly. No missing strings or None values. |
| #20 | peak_analysis crash | ✅ FIXED | ✅ STILL FIXED | Peak deconvolution runs without crash and shows fitted curve. |
| #21 | Export crash | ✅ FIXED | ✅ STILL FIXED | Cancel dialog does not crash GUI. Confirmed on DRT and EIS Regression exports. |
| #22 | Batch processing | ❌ NOT FIXED | ❌ NOT FIXED | No batch processing function found in runs.py. Public functions: BHT_run, Bayesian_run, simple_run, peak_analysis only. |
| #23 | MATLAB syntax | ✅ FIXED | ✅ STILL FIXED | Module imports without syntax error. |
| #24 | Tau convention | ✅ FIXED | ✅ STILL FIXED | τ = 1/f confirmed in previous review; API unchanged. |
| #25 | EIS pre-treatment | ❌ NOT FIXED | ❌ NOT FIXED | No pre-treatment, smoothing, or outlier removal functions found in runs.py or basics.py. |

---

## 13. GUI Paragraph

The GUI has improved substantially in this release. The most important fixes are now in place: the Bayesian sampler no longer fails silently when GCV selects a small lambda — instead it raises lambda to a safe floor, runs successfully, and shows an informative dialog explaining what happened. The Hilbert Transform no longer produces scientifically misleading DRT curves; instead the BHT/KK tab correctly shows consistency scores while explicitly stating that no validated DRT estimate is available. Loading a new file now correctly clears the old fit from the EIS Data plot immediately. Export before calculation now shows a clear warning dialog. R_inf and the regularization parameter actually used are now displayed in the left panel after a run. The matplotlib figure leak on repeated runs has been fixed.

Two minor issues remain. First, the R_inf and regularization parameter display fields are too narrow — values appear truncated as raw integers rather than in scientific notation, which is harder to read. For example, `171369321842203` is shown instead of `1.71e-04 Ω`. Second, batch processing (#22) and EIS pre-treatment (#25) are still not implemented. The BHT hyperparameter optimizer also continues to fail on Windows for 1ZARC_exact.csv with the PWL kernel, producing ABNORMAL errors on all 20 attempts in the automated test. This is a platform-specific issue that does not affect the GUI workflow since the BHT/KK tab correctly handles and reports the failure. Overall the GUI is now substantially more reliable and scientifically honest than in the previous review, and is approaching readiness for external users, with the caveat that batch processing and EIS pre-treatment remain absent.

---

*Report compiled by Rahul, 20 June 2026*

*Under supervision of Prof. Francesco Ciucci*
