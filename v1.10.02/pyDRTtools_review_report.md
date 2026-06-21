# pyDRTtools v1.10.02 — Review Report

**Reviewer:** Rahul  
**Date:** 15 June 2026  
**Version reviewed:** pyDRTtools_1.10.02_clean  
**Assigned by:** Prof. Francesco Ciucci

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
| pytest | 9.0.3 |
| cvxopt | 1.3.3 |
| pyqt | 5.15.11 |
| jupyter | 1.1.1 |
| Qt / GUI available | Yes (PyQt5) |
| Conda environment | pydrt-review |

---

## 2. Exact Commands Run

```
conda create -n pydrt-review python=3.11
conda activate pydrt-review
conda install -c conda-forge numpy scipy pandas scikit-learn matplotlib pytest jupyter cvxopt pyqt

cd "C:\Users\rahul\OneDrive\Documents\Master- battery materials and technology\hiwi ciucci jun 2026\new code for review 15_06_2026\pyDRTtools_1.10.02_clean"
set PYTHONPATH=%CD%

python -m compileall -q pyDRTtools tests

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

## 3. Automated Test Results

### Main tests: 180 PASSED, 1 FAILED

**Failure:**

```
FAILED tests/test_input_validation.py::test_from_file_reads_tutorial_csv_with_unnamed_index

AssertionError:
  assert np.float64(59.67093407095763) == 59.20397431908595 +/- 5.9e-05
  Obtained: 59.67093407095763
  Expected: 59.20397431908595
```

**Cause:** The tutorial file `1ZARC.csv` was regenerated with updated parameters but the hardcoded expected value in the test was not updated. This is a test maintenance issue, not a code defect. The file loads and produces correct scientific results.

**Recommendation:** Update the hardcoded value in `test_from_file_reads_tutorial_csv_with_unnamed_index` to match the current `1ZARC.csv`.

### GUI tests: 15 PASSED, 0 FAILED

```
tests/test_gui_export.py::test_bht_export_rows_match_headers PASSED
tests/test_gui_export.py::test_peak_export_writes_selected_file_and_companion_peaks_csv PASSED
tests/test_gui_export.py::test_export_drt_without_results_shows_error_and_does_not_open_save_dialog PASSED
tests/test_gui_export.py::test_export_eis_without_results_shows_error_and_does_not_open_save_dialog PASSED
tests/test_GUI.py::test_gui_rbf_options_include_all_core_supported_gui_rbf_types PASSED
tests/test_GUI.py::test_simple_run_after_peak_analysis_does_not_export_stale_peak_table PASSED
tests/test_GUI.py::test_peak_export_handles_more_than_twelve_peaks PASSED
tests/test_GUI.py::test_computation_worker_emits_result_and_captures_exception PASSED
tests/test_GUI.py::test_simple_run_callback_starts_worker_instead_of_running_inline PASSED
tests/test_GUI.py::test_callback_reports_nonnumeric_line_edit_without_starting_worker PASSED
tests/test_GUI.py::test_import_file_passes_signed_imaginary_convention_by_default PASSED
tests/test_GUI.py::test_import_file_passes_positive_nyquist_imaginary_convention PASSED
tests/test_GUI.py::test_import_file_reports_parser_errors PASSED
tests/test_GUI.py::test_busy_state_restores_previous_widget_states PASSED
tests/test_GUI.py::test_launch_gui_reuses_existing_qapplication PASSED
```

**Total across all test files: 195 PASSED, 1 FAILED**

---

## 4. Tutorial Data Generation

```
python tutorial/generate_artificial_eis_data.py
```

Output:
```
Wrote 8 spectra to ...tutorial\data
Metadata: ...tutorial\data\metadata.json
```

8 spectra generated with correct parameters (R_inf=10 Ohm, R_pol=50 Ohm for 1ZARC, expected tau peaks at [0.1] s and [0.1, 0.0001] s).

---

## 5. Recovery Checks — Pass/Fail Output

### Fast run (20/20 PASSED)

```
PASS  1ZARC exact: R_inf                 value=0.00183434    expected <= 0.05 relative error
PASS  1ZARC exact: R_pol                 value=0.00287532    expected <= 0.20 relative error
PASS  1ZARC exact: tau peak              value=0.00432633    expected <= 0.35 log10 decades
PASS  1ZARC exact: impedance residual    value=0.000347356   expected <= 0.06 relative residual
PASS  1ZARC noisy: R_inf                 value=0.00107277    expected <= 0.05 relative error
PASS  1ZARC noisy: R_pol                 value=0.00307029    expected <= 0.20 relative error
PASS  1ZARC noisy: tau peak              value=0.00679852    expected <= 0.35 log10 decades
PASS  1ZARC noisy: impedance residual    value=0.0021478     expected <= 0.06 relative residual
PASS  2ZARC0.csv: total R_pol            value=0.000250037   expected <= 0.25 relative error
PASS  2ZARC0.csv: two-peak tau recovery  value=0.2           expected <= 0.45 log10 decades
PASS  2ZARC1.csv: total R_pol            value=0.000606812   expected <= 0.25 relative error
PASS  2ZARC1.csv: two-peak tau recovery  value=0             expected <= 0.45 log10 decades
PASS  2ZARC2.csv: total R_pol            value=0.00141535    expected <= 0.25 relative error
PASS  2ZARC2.csv: two-peak tau recovery  value=0             expected <= 0.45 log10 decades
PASS  2ZARC3.csv: total R_pol            value=4.66983e-05   expected <= 0.25 relative error
PASS  2ZARC3.csv: two-peak tau recovery  value=0             expected <= 0.45 log10 decades
PASS  2ZARC4.csv: total R_pol            value=0.00162089    expected <= 0.25 relative error
PASS  2ZARC4.csv: two-peak tau recovery  value=0             expected <= 0.45 log10 decades
PASS  2ZARC5.csv: total R_pol            value=0.00183176    expected <= 0.25 relative error
PASS  2ZARC5.csv: two-peak tau recovery  value=0             expected <= 0.45 log10 decades
```

### Slow run (24/24 PASSED — additional checks)

```
PASS  Bayesian credible arrays finite      value=finite    expected all finite
PASS  Bayesian credible intervals ordered  value=ordered   expected lower <= mean <= upper
PASS  BHT returns status                   value=success   expected success or clean failure
PASS  BHT scores finite                    value=finite    expected all finite
```

### Note on output file locations

The professor's instructions state that figures and `recovery_metrics.csv` should be written to `tutorial/figs/recovery_checks/`. This directory was **not created** by `run_recovery_checks.py`. Instead, DRT curve CSV files were written to `tutorial/results/` (e.g. `1ZARC_DRT-RR.csv`, `1ZARC_Z-RR.csv`). The PDF/PNG figures (`*_drt_exact_vs_recovered.pdf`, `*_nyquist_fit.pdf`, `*_residual.pdf`) and `recovery_metrics.csv` were not produced. The pass/fail checks ran correctly and all passed, but the expected output file structure does not match the instructions.

---

## 6. recovery_metrics.csv

`recovery_metrics.csv` was not generated. The script writes individual CSV result files to `tutorial/results/` instead. Sample from `1ZARC_Z-RR.csv` (impedance fit) confirms R_inf recovery near 10 Ohm at the lowest frequency row (freq=0.01, Z_re=59.89 Ohm, consistent with R_inf + R_pol = 60 Ohm at DC limit).

---

## 7. 1ZARC Exact — DRT Recovery Summary

From the notebook and recovery checks:

| Metric | Recovered | Expected | Relative Error |
|--------|-----------|----------|----------------|
| R_inf | 9.982 Ohm | 10.0 Ohm | 0.18% |
| R_pol | 50.144 Ohm | 50.0 Ohm | 0.29% |
| tau peak | ~0.1 s | 0.1 s | 0.004 decades |
| Impedance residual | 0.000347 | <= 0.06 | well within tolerance |

The recovered DRT was plotted directly against the exact analytical ZARC DRT in the notebook (solid blue = exact, dashed orange = recovered). The two curves were nearly indistinguishable. The Nyquist fit overlay was also excellent.

---

## 8. Jupyter Notebook Results

All cells passed. Kernel: Restart & Run All.

| Check | Result | Details |
|-------|--------|---------|
| 1. Synthetic spectra generated correctly | PASS | 8 spectra shown in metadata table and Nyquist plot |
| 2. 1ZARC exact: R_inf close to 10 Ohm | PASS | 9.982 Ohm recovered, error 0.18% |
| 3. 1ZARC exact: R_pol close to 50 Ohm | PASS | 50.144 Ohm recovered, error 0.29% |
| 4. DRT peak near tau = 0.1 s | PASS | Peak at tau = 0.1 s, error 0.004 decades |
| 5. Recovered DRT plotted against exact ZARC DRT | PASS | Both curves shown, near-perfect overlap |
| 6. Noisy 1ZARC case reasonable | PASS | Fit and DRT visually correct |
| 7. 2ZARC peaks near 1e-4 and 0.1 s | PASS | Both peaks clearly resolved in all 6 variants |
| 8. Bayesian: finite lower/mean/upper, ordered | PASS | All True, lower <= mean <= upper confirmed |
| 9. BHT: returns status, no hang | PASS | success=True, attempts=1, scores in [0,1] |

---

## 9. GUI Testing

GUI launched with `python launch.py`. All tests performed with `1ZARC.csv` as primary dataset.

| Test | Result | Notes |
|------|--------|-------|
| GUI launches | PASS | Window opens cleanly, all panels visible |
| Load 1ZARC.csv | PASS | File loads without error |
| Nyquist plot displays correctly | PASS | Clean semicircle, data spans 10 to 60 Ohm |
| Sign convention -Z'' | PASS | Capacitive arc above zero axis, correct |
| Simple Run DRT peak near tau = 0.1 s | PASS | Single sharp peak at tau = 0.1 s |
| GCV optimal regularization shown | PASS | Lambda = 0.000217 displayed in GUI |
| Nyquist fit overlays data | PASS | Black fitted curve follows red data dots closely |
| Export DRT CSV — saves correctly | PASS | File saves, opens in Excel |
| Export DRT CSV — headers and row lengths | PARTIAL | Headers present (L, R, tau, gamma) but all data in column A; Excel does not auto-split. EIS Regression export correctly splits into columns (freq, mu_Z_re, mu_Z_im, Z_re_res, Z_im_res) |
| Export EIS Regression CSV | PASS | Correct headers and column separation |
| Export then Cancel — no crash | PASS | GUI stays open, no error |
| Export before running — warning shown | PARTIAL | Automated test confirms error is triggered in code, but no visible dialog appeared in manual test. Behavior may depend on focus state |
| R_inf shown in GUI after run | FAIL | R_inf value is not displayed anywhere in the GUI panel. User must export CSV to find it |
| Changing regularization updates output | PASS | Switching custom to GCV changes lambda from 0.001 to 0.000217 and visibly sharpens the DRT peak |
| Load multiple files in sequence | PARTIAL | Behavior inconsistent — in some tests old fit line remained visible over new data; in others the plot refreshed correctly. May depend on order of operations. |
| Bayesian on 1ZARC_exact.csv with custom lambda | PASS | MAP and posterior mean shown, credible band visible (screenshot 10) |
| Bayesian on 1ZARC_exact.csv with GCV lambda | FAIL | Sampler rejection error — GCV selects lambda=1e-7 which makes sampler infeasible (screenshot 09) |
| Bayesian on 1ZARC noisy with default settings | FAIL | Sampler rejection error — see traceback below (screenshot 06) |
| Bayesian on 2ZARC0.csv noisy | PASS | Two peaks with credible bands shown correctly |
| **Key finding** | — | Bayesian failure is linked to regularization lambda value. Very small lambda (from GCV) causes sampler to fail. Manual lambda=0.001 works reliably. |
| Hilbert Transform — all datasets | FAIL | Produces wrong DRT on all datasets — see details below |
| EIS Score tab | PASS | Bar chart with 6 scores (S_1sigma through S_JSD) shown, no crash |
| Peak Analysis | PASS | No crash, fitted red curve overlaid on DRT |
| High-DPI at 1920x1080 125% scaling | PASS | Text, buttons, and plots all clear and readable |
| Memory leak warning after many runs | WARNING | Terminal shows: "More than 20 figures have been opened" — GUI does not close matplotlib figures after each run |

### Bayesian Run Error Traceback (1ZARC noisy)

```
RuntimeError: generate_tmg rejected 10001 trajectories
(max_rejections=10000) after collecting 0 of 999 requested samples;
the truncated region may be (nearly) infeasible or max_bounces too small.
```

GUI showed error dialog and did not crash. No credible bands produced.

### Hilbert Transform Bug — All Datasets

| Dataset | Y-axis scale | Peak location | Expected peak |
|---------|-------------|---------------|---------------|
| 1ZARC noisy | 1e-117 | tau ~ 1e-6 s | tau = 0.1 s |
| 1ZARC exact | ~0 (flat line) | none | tau = 0.1 s |
| 2ZARC noisy | 1e-8 | tau ~ 1e-3 s | tau = 1e-4 and 0.1 s |

The Hilbert Transform runs without crashing but produces numerically degenerate DRT values on all tested datasets. The output appears to complete but is scientifically meaningless.

---

## 10. GitHub Issue Status

| # | Link | Status | Test run | Evidence |
|---|------|--------|----------|---------|
| #26 | https://github.com/ciuccislab/pyDRTtools/issues/26 | PARTLY FIXED | Ran Hilbert Transform on 1ZARC_exact, 1ZARC noisy, 2ZARC0 | No crash but DRT values at scale 1e-8 to 1e-117 with peaks at wrong tau locations on all 3 datasets |
| #15 | https://github.com/ciuccislab/pyDRTtools/issues/15 | FIXED | Ran Simple Run, Bayesian, BHT, peak analysis repeatedly | No infinite loops or hangs observed. BHT retries up to 20 times but terminates cleanly |
| #21 | https://github.com/ciuccislab/pyDRTtools/issues/21 | FIXED | Clicked Export then Cancel on save dialog | GUI stays open, no crash, no error. Confirmed on both DRT and EIS Regression export buttons |
| #20 | https://github.com/ciuccislab/pyDRTtools/issues/20 | FIXED | Ran Peak deconvolution after Simple Run on 1ZARC | No crash. Fitted red curve overlaid on black DRT correctly |
| #18 | https://github.com/ciuccislab/pyDRTtools/issues/18 | FIXED | Ran full notebook Restart and Run All | All cells completed, no missing strings or None values printed anywhere |
| #23 | https://github.com/ciuccislab/pyDRTtools/issues/23 | FIXED | `python -c "import pyDRTtools.basics as b; print('ok')"` | Prints "No MATLAB syntax errors found" with no SyntaxError |
| #24 | https://github.com/ciuccislab/pyDRTtools/issues/24 | FIXED | Script: EIS_object(freq=[1,10,100], ...) and checked tau[0] | tau[0]=1.0 matches 1/freq[0]=1.0 exactly; 1/(2*pi*f)=0.159 not used |
| #14 | https://github.com/ciuccislab/pyDRTtools/issues/14 | PARTLY FIXED | Ran GCV on 7-point synthetic dataset | Warning now shown: "optimal_lambda selected the upper optimization boundary". GCV still returned lambda=0.999 on simple data, suggesting optimizer search range may still be too narrow for some inputs |
| #22 | https://github.com/ciuccislab/pyDRTtools/issues/22 | NOT FIXED | `inspect.getmembers(runs)` — searched for "batch" in all function names | No batch processing function exists in runs.py or any module. Users must process one file at a time |
| #25 | https://github.com/ciuccislab/pyDRTtools/issues/25 | NOT FIXED | `inspect.getmembers(runs)` and `inspect.getmembers(basics)` — searched for pretreat, clean, smooth, filter, outlier | No pre-treatment or data cleaning functions found in runs.py or basics.py |
| #17 | https://github.com/ciuccislab/pyDRTtools/issues/17 | NOT REPRODUCIBLE | Tested GUI at 1920x1080 with 125% Windows scaling | Text, buttons, plots and windows all appeared clear and correctly sized. Issue may only manifest at 4K resolution with 200%+ scaling which was not available |

---

## 11. GUI Paragraph

The GUI is functional and handles the core scientific workflow correctly. Loading data, running Simple DRT, switching between regularization methods, viewing the Nyquist fit, viewing the DRT peak, running Peak Analysis, and exporting results all work reliably. The sign convention is correctly applied. The fitted Nyquist curve visually overlays the data closely. The EIS Score tab shows a clean bar chart. Switching from custom to GCV regularization visibly sharpens the DRT and the optimal lambda is displayed in the panel, which is useful.

However several problems deserve attention before wider release. The most serious is the Hilbert Transform, which runs without crashing but produces numerically degenerate DRT values (scales of 1e-8 to 1e-117, peaks at entirely wrong tau locations) on every dataset tested. The output looks like a normal plot but is scientifically meaningless, which is more dangerous than a crash because a user might not notice. The Bayesian sampler fails with a trajectory rejection error in two situations: on 1ZARC noisy data with default settings, and on 1ZARC exact data when GCV regularization selects a very small lambda (1e-7). The root cause appears to be that very small lambda values make the constrained sampling region infeasible. Bayesian works correctly when lambda is set manually to 0.001, and also works on 2ZARC noisy data. The default settings are therefore unsuitable for the most common use cases and the GUI gives no guidance on this. The GUI does not display R_inf anywhere after a run, forcing the user to export a CSV to retrieve a basic fit parameter. Loading a new data file without running a new calculation leaves the old fitted curve visible over the new raw data in the EIS Data tab, which is visually misleading. Clicking Export before any calculation silently does nothing in manual testing, though the automated test confirms the error path is triggered in code. The EIS Regression export correctly separates columns, but the DRT export places all data in column A of the CSV, which Excel does not auto-split. The terminal also shows matplotlib figure leak warnings after extended use, suggesting the GUI accumulates open figures and may degrade with long sessions. The overall visual layout is clean, labels are readable at 125% scaling, and the panel organisation is logical, but the Hilbert Transform reliability and Bayesian default settings are the most important items to fix before this code is shared more widely.

---

*Report compiled by Rahul, 15 June 2026*
*Reviewed under supervision of Prof. Francesco Ciucci*
