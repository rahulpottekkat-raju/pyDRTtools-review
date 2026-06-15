# Screenshot Descriptions
## pyDRTtools v1.10.02 — GUI Testing Evidence
**Reviewer:** Rahul  
**Date:** 15 June 2026

---

### 01_gui_launch.png
**What it shows:** Fresh GUI window immediately after launching with `python launch.py`.  
**Result:** ✅ GUI opens cleanly with all panels visible — Settings, Run, Peak Analysis, Export Result, and tab bar (EIS Data, Magnitude, Phase, Re Part, Im Part, Re Residual, Im Residual, DRT, EIS Score). Default settings are correctly pre-loaded.

---

### 02_nyquist_1ZARC_loaded.png
**What it shows:** EIS Data tab after loading `tutorial/data/1ZARC.csv`.  
**Result:** ✅ Nyquist plot displays correctly. Clean semicircular arc visible. Y-axis shows `-Z''/Ω` confirming correct sign convention — capacitive arc appears above zero as expected. Data spans Z' from 10 to 60 Ω.

---

### 03_drt_simple_run_gcv.png
**What it shows:** DRT tab after running Simple Run with GCV regularization on 1ZARC.csv.  
**Result:** ✅ Single sharp DRT peak correctly located at τ ≈ 0.1 s. GCV automatically found optimal regularization parameter λ = 0.000217 (shown in left panel). Peak height γ(τ) ≈ 22 Ω is physically reasonable.

---

### 04_nyquist_fit_overlay.png
**What it shows:** EIS Data tab after Simple Run — fitted curve overlaid on raw data.  
**Result:** ✅ Black fitted curve follows red data dots closely across the entire arc. The fit quality is excellent with no visible deviation.

---

### 05_export_cancel_no_crash.png
**What it shows:** GUI state after clicking Export (DRT) and then cancelling the save dialog.  
**Result:** ✅ GUI remains open and fully functional after cancel. No crash, no error message. This confirms GitHub issue #21 (Export with no file name crashes the program) is **FIXED**.

---

### 06_bayesian_error_1ZARC_noisy.png
**What it shows:** Bayesian Run error dialog on 1ZARC.csv (noisy data) with default settings.  
**Result:** ⚠️ Bayesian sampler rejected all 10001 trajectories and collected 0 of 999 requested samples. Error message reads: "generate_tmg rejected 10001 trajectories (max_rejections=10000) after collecting 0 of 999 requested samples; the truncated region may be (nearly) infeasible or max_bounces too small." GUI shows informative dialog and does not crash, but no credible bands are produced.

---

### 07_hilbert_transform_wrong_1ZARC_noisy.png
**What it shows:** DRT tab after running Hilbert Transform on 1ZARC.csv (noisy data).  
**Result:** ❌ Y-axis scale is 1e-117 — numerically meaningless. Blue curve (Hilbert Transform result) peaks near τ ≈ 10⁻⁶ s which is completely wrong. Black curve (Simple Run) peaks correctly at τ = 0.1 s. No legend distinguishing the two curves. This confirms GitHub issue #26 (Hilbert Transform) is **PARTLY FIXED** — no crash but scientifically wrong output.

---

### 08_hilbert_transform_wrong_1ZARC_exact.png
**What it shows:** DRT tab after running Hilbert Transform on 1ZARC_exact.csv (noiseless data).  
**Result:** ❌ Y-axis scale is 1e-8. Single broad peak at wrong location τ ≈ 10⁻³ s instead of expected τ = 0.1 s. Confirms the Hilbert Transform bug is reproducible on noiseless data as well, ruling out noise as the cause.

---

### 09_bayesian_error_1ZARC_exact_GCV.png
**What it shows:** Bayesian Run error on 1ZARC_exact.csv when using GCV regularization.  
**Result:** ❌ Bayesian sampler fails even on noiseless exact data when GCV selects a very small lambda (λ = 1e-7 shown in panel). This is an additional finding — Bayesian failure is linked to the regularization parameter value, not just to noise in the data. With GCV selecting very small lambda the sampler cannot find feasible trajectories.

---

### 10_bayesian_working_1ZARC_exact_custom.png
**What it shows:** DRT tab after successful Bayesian Run on 1ZARC_exact.csv with custom λ = 0.001.  
**Result:** ✅ Bayesian succeeds when lambda is set manually. MAP estimate (black) and posterior mean (blue) both shown with grey credible band. Both curves peak correctly at τ = 0.1 s. Credible band is narrow, indicating high confidence in the recovery.

---

### 11_eis_score_tab.png
**What it shows:** EIS Score tab after running Hilbert Transform on 1ZARC_exact.csv.  
**Result:** ✅ Bar chart displays 6 BHT consistency scores: S_1σ, S_2σ, S_3σ, S_μ, S_HD, S_JSD. Blue = Re part, Black = Im part. GUI returns status without hanging or crashing. Scores are finite and within [0, 100%].

---

### 12_peak_analysis.png
**What it shows:** DRT tab after running Peak deconvolution on 1ZARC_exact.csv.  
**Result:** ✅ Red fitted peak curve overlaid on black DRT. Peak correctly identified near τ = 0.1 s. No crash. This confirms GitHub issue #20 (peak_analysis causes program crash) is **FIXED**.

---

### 13_new_file_loaded_clean.png
**What it shows:** EIS Data tab after loading 1ZARC.csv following a previous run on 2ZARC0.csv.  
**Result:** ✅ In this instance the plot correctly shows only the new 1ZARC data with no contamination from the previous 2ZARC fit. Note: in earlier testing the old fit line was visible in some cases — this behavior appears inconsistent and may depend on the order of operations.

---

### 14a_regularization_custom.png
**What it shows:** DRT tab with custom regularization λ = 0.001 on 1ZARC.csv.  
**Result:** ✅ DRT shows peak at τ = 0.1 s. Slight secondary bump visible at τ ≈ 10⁻² s — artifact of higher regularization. Lambda fixed at 0.001.

---

### 14b_regularization_gcv.png
**What it shows:** DRT tab with GCV regularization on 1ZARC.csv — same data as 14a.  
**Result:** ✅ DRT updates consistently when switching from custom to GCV. GCV selects lower lambda (0.000217) producing a cleaner, sharper peak with less spurious activity. Confirms that changing regularization options updates the GUI output correctly.

---

## Summary of GUI Evidence

| Screenshot | Test | Result |
|------------|------|--------|
| 01 | GUI launches | ✅ |
| 02 | Nyquist plot + sign convention | ✅ |
| 03 | DRT peak at τ=0.1s, GCV lambda | ✅ |
| 04 | Nyquist fit overlay | ✅ |
| 05 | Export cancel no crash (#21) | ✅ FIXED |
| 06 | Bayesian error on noisy data | ⚠️ |
| 07 | Hilbert Transform bug (#26) noisy | ❌ PARTLY FIXED |
| 08 | Hilbert Transform bug (#26) exact | ❌ PARTLY FIXED |
| 09 | Bayesian fails with GCV lambda | ⚠️ |
| 10 | Bayesian works with custom lambda | ✅ |
| 11 | EIS Score tab, no hang | ✅ |
| 12 | Peak Analysis no crash (#20) | ✅ FIXED |
| 13 | New file loads clean | ✅ |
| 14a | Custom regularization DRT | ✅ |
| 14b | GCV regularization DRT | ✅ |
