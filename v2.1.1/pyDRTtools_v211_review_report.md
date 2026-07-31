# pyDRTtools v2.1.1 — Review Report (BETA)

**Reviewer:** Rahul Raj Pottekkat Raju  
**Date:** 24 July 2026  
**Assigned by:** Prof. Francesco Ciucci  
**Archive reviewed:** `pyDRTtools_2.1.1.zip`  
**GitHub repository:** https://github.com/rahulpottekkat-raju/pyDRTtools-review

---

## 1. System Information

| Item | Details |
|------|---------|
| Operating System | Windows 11 (10.0.26200) |
| Python | 3.12.13 |
| Qt | 5.15.2 |
| Matplotlib | 3.11.0 |
| NumPy | 2.5.1 |
| SciPy | 1.18.0 |
| cvxopt | 1.3.3 |
| pyDRTtools | 2.1.1 |
| Install method | conda-forge (core packages) + `pip install -e .` |

---

## 2. Environment Setup

The archive includes a `README_pyDRTtools_2.1.1_BETA.md` file with installation instructions. I followed these exactly:

```
conda create -n pyDRTtools-beta python=3.12 -y
conda activate pyDRTtools-beta
conda install -c conda-forge numpy scipy pandas cvxopt matplotlib pyqt pip -y
cd [archive root]
python -m pip install -e .
python -m pyDRTtools --version
```

Version confirmed: `pyDRTtools 2.1.1`

Note: The launch command has changed from `python launch.py` to `python -m pyDRTtools` in this version.

---

## 3. Archive Integrity

SHA-256 checksum verified against `pyDRTtools_2.1.1.zip.sha256`:

- Computed: `5e684f942fe077e7fc00f9f07d2844177ab07554c6b5dcea0f622582b3c5575b`
- Expected: `5e684f942fe077e7fc00f9f07d2844177ab07554c6b5dcea0f622582b3c5575b`

✅ **Match confirmed — archive is intact.**

---

## 4. Archive Cleanliness

| Item | Status | Notes |
|------|--------|-------|
| `.git` | ✅ Clean | Not found |
| `__MACOSX` | ✅ Clean | Fixed vs v1.16.12 |
| `.DS_Store` | ✅ Clean | Not found |
| `AGENTS.md` | ✅ Clean | Fixed vs v1.16.12 |
| `.pytest_cache` | ✅ Clean | Not found |
| `dist/`, `build/` | ✅ Clean | Not found |
| `.github/workflows` | ⚠️ Present | CI configuration — acceptable |
| `.github/constraints` | ⚠️ Present | New — dependency constraints for CI |
| `__pycache__` | ❌ Present | Found in `pyDRTtools/` only (7 files) |
| `.pyc` files | ❌ Present | 7 compiled files found |
| `docs/` folder | ❌ Missing | Entire docs folder absent — causes 1 test failure |

**Notable improvement:** `AGENTS.md` and `__MACOSX` folder both removed vs v1.16.12. Only minor cleanliness issues remain.

---

## 5. Automated Tests

### 5.1 Core Tests

```
python -m pytest -m "not slow and not gui and not tutorial and not notebook" -q
39 failed, 2067 passed, 9 skipped, 141 deselected, 56 warnings in 269.14s
```

**Result:** 2067 passed, 39 failed.

| Failure Category | Count | Impact |
|-----------------|-------|--------|
| `test_release_smoke.py` — archive packaging tests | 38 | No scientific impact — tests run against extracted folder, not zip |
| `test_version_metadata.py` — missing `docs/MIGRATING_TO_2.0.md` | 1 | `docs/` folder missing from archive |

**All previous Windows-specific failures are gone** ✅ — the conda-forge + pip install approach eliminates MKL/OpenMP conflicts that caused failures in earlier reviews.

### 5.2 GUI Tests

```
python -m pytest -m gui -q
108 passed, 1 skipped in 13.80s
```

**Result:** ✅ 108 passed, 0 failed — significant improvement from v1.16.12 (69 passed, 3 failed).

---

## 6. Tutorial and Recovery Checks

### 6.1 Generate Artificial EIS Data

```
python tutorial/generate_artificial_eis_data.py
Wrote 8 spectra to tutorial/data
```
✅ 8 spectra generated successfully.

### 6.2 Recovery Checks (Fast)

```
python tutorial/run_recovery_checks.py
20/20 passed
```

All checks passed within tolerance:

| Dataset | R_inf error | R_pol error | tau peak error | Residual |
|---------|------------|------------|---------------|---------|
| 1ZARC exact | 0.00183 | 0.00288 | 0.00433 | 0.000347 |
| 1ZARC noisy | 0.00107 | 0.00307 | 0.00680 | 0.00215 |
| 2ZARC0–5 | All within tolerance | — | — | — |

### 6.3 Recovery Checks (Slow)

```
python tutorial/run_recovery_checks.py --slow
25/25 passed
```

| Check | Result |
|-------|--------|
| Bayesian credible arrays finite | ✅ PASS |
| Bayesian credible intervals ordered | ✅ PASS |
| BHT returns status: **success** | ✅ PASS — major fix vs v1.16.12! |
| BHT scores finite | ✅ PASS |
| BHT DRT unavailable (no misleading curves) | ✅ PASS |

**Notable:** A `UserWarning` was raised about 99% Bayesian credible-band edges being under-resolved (0.3 effective tail draws vs minimum of 50). All checks still pass but the warning suggests more draws may be needed for reliable 99% credible bands.

### 6.4 Tutorial Notebook

```
jupyter nbconvert --to notebook --execute "tutorial/tutorial-pyDRTtools.ipynb" --output tutorial-pyDRTtools-executed.ipynb
[NbConvertApp] Writing 651416 bytes
```

✅ Notebook executed successfully. All cells completed without error.

---

## 7. GUI Testing

### 7.1 New GUI Design

v2.1.1 introduces a completely redesigned GUI:
- Clean white theme replacing the previous grey boxed layout
- Blue prominent buttons (Run simple, Run Bayesian, Run BHT/KK)
- Grid lines on all plots
- Navigation toolbar at bottom of plot area
- Coordinate display in plot area
- All labels fully visible — no truncation
- "Import EIS data to begin" placeholder text
- Default Number of Samples increased from 1000 to 2000
- GCV is now the default Parameter Selection Method

### 7.2 GUI Test Results

| # | Test | Result | Screenshot |
|---|------|--------|-----------|
| 01 | Fresh GUI launch | ✅ PASS | `01_gui_launch.png` |
| 02 | Load 1ZARC.csv — Nyquist plot | ✅ PASS | `02_nyquist_1ZARC_loaded.png` |
| 03 | Simple Run — R_inf displayed | ✅ PASS | `03_simple_run_rinf_display.png` |
| 04 | Nyquist fit overlay | ✅ PASS | `04_nyquist_fit_overlay.png` |
| 05 | Bayesian Run — new adaptive dialog | ✅ PASS | `05_bayesian_dialog.png` |
| 06 | Bayesian DRT with credible bands | ✅ PASS | `06_bayesian_drt_credible_bands.png` |
| 07 | BHT/KK scores — DRT disabled | ✅ PASS | `07_bht_kk_scores.png` |
| 08 | New file load — no contamination | ✅ PASS | `08_new_file_no_contamination.png` |
| 09 | Export DRT before run | ❌ CRITICAL BUG | `09_export_error_bug.png` |
| 10 | Export DRT after run | ❌ CRITICAL BUG | `10_export_drt_bug_after_run.png` |
| 11 | Export EIS Regression | ❌ CRITICAL BUG | `11_export_eis_bug.png` |
| 12 | Save Figure | ❌ CRITICAL BUG | `12_export_fig_bug.png` |
| 13 | 1ZARC_exact DRT — single peak | ✅ PASS | `13_1ZARC_exact_drt.png` |

### 7.3 Critical Export Bug

**All three export/save functions are broken in v2.1.1:**

| Function | Error |
|----------|-------|
| Export DRT | `GUI.export_DRT() takes 1 positional argument but 2 were given` |
| Export EIS Regression | `GUI.export_EIS() takes 1 positional argument but 2 were given` |
| Save Figure | `GUI.export_fig() takes 1 positional argument but 2 were given` |

This occurs both before and after running a calculation. Users cannot save any results from the GUI. This is a critical regression — all export functionality was working correctly in v1.16.12.

The error pattern suggests the export methods were refactored (possibly connected to Qt signals that pass an extra argument) but not updated to accept the additional positional argument.

### 7.4 GUI Paragraph

The redesigned GUI launches cleanly and looks significantly more polished than previous versions. The new white theme, blue buttons, grid lines, and navigation toolbar make the interface more professional and easier to use. All labels are now fully visible without truncation. File loading, Simple Run, Nyquist fit overlay, and BHT/KK scoring all work correctly. The Bayesian Run now succeeds on Windows — a major improvement over v1.16.12 — and shows a detailed diagnostic dialog with adaptive lambda search results. However, a critical regression was discovered: all three export/save functions crash with a Python argument error. This completely blocks users from saving DRT results, EIS regression data, or figures. This must be fixed before release.

---

## 8. Open Issues Table

| Issue | Description | v1.16.12 | v2.1.1 | Evidence |
|-------|-------------|---------|---------|----------|
| #14 | Tutorial notebook fails | ✅ FIXED | ✅ STILL FIXED | Notebook executed successfully |
| #15 | Recovery figures not generated | ✅ FIXED | ✅ STILL FIXED | All 8 datasets generated |
| #17 | High-DPI/4K display scaling | ⚠️ NOT REPRODUCIBLE | ⚠️ NOT REPRODUCIBLE | 1920×1080 display |
| #18 | Export before run crashes GUI | ✅ FIXED | ❌ REGRESSION | New crash: argument mismatch |
| #20 | Cancel export crashes GUI | ✅ FIXED | ❌ REGRESSION | Export broken entirely |
| #21 | Old results not cleared | ✅ FIXED | ✅ STILL FIXED | Fields cleared on import |
| #22 | Batch processing missing | ✅ FIXED | ✅ STILL FIXED | batch.py present |
| #23 | R_inf not displayed | ✅ FIXED | ✅ STILL FIXED | R_inf shown after Simple Run |
| #24 | Export warning missing | ✅ FIXED | ❌ REGRESSION | Crashes instead of warning |
| #25 | EIS pretreatment missing | ✅ FIXED | ✅ STILL FIXED | pretreatment.py present |
| #26 | BHT misleading DRT | ✅ FIXED | ✅ STILL FIXED | DRT disabled message shown |

---

## 9. Key Findings

1. **Critical export regression:** All three export/save functions crash with `TypeError: takes 1 positional argument but 2 were given`. Users cannot save any results. Must be fixed before release.

2. **Bayesian Run fixed on Windows:** The HMC sampler now succeeds on real EIS data (1ZARC.csv). This was broken in v1.16.12. New adaptive lambda search automatically finds stable sampling parameters.

3. **New scientific adequacy checks:** The Bayesian dialog now reports detailed diagnostics including selection invariance, tail resolution, multichain mixing, and MCSE checks. In testing, scientific adequacy was not established for 1ZARC.csv, with an INVARIANCE WARNING indicating the credible intervals correspond to a materially different regularization strength than the GCV-selected lambda.

4. **GUI completely redesigned:** Clean white theme, blue buttons, grid lines, navigation toolbar, and proper label spacing. Significant usability improvement.

5. **BHT now succeeds on Windows:** The BHT optimizer now completes successfully — a major fix vs all previous versions where it failed after 20 attempts.

6. **Missing docs/ folder:** The entire `docs/` directory is absent from the archive, causing 1 test failure (`test_2_0_release_support_typing_and_security_claims_match_source`).

7. **38 release smoke test failures:** These test the ZIP archive packaging process itself and are not scientific failures. They fail because they run against the extracted folder rather than the zip.

8. **All previous Windows-specific failures resolved:** Using conda-forge + pip install eliminates MKL/OpenMP conflicts. No path separator or OpenMP failures observed.

9. **Bayesian credible-band warning:** A UserWarning about under-resolved 99% credible-band edges was raised during slow recovery checks. All checks still pass but users should be aware that more draws may be needed for reliable 99% bands.

---

## 10. Overall Assessment

pyDRTtools v2.1.1 (BETA) shows major improvements in GUI design, Bayesian reliability on Windows, and test suite cleanliness. The recovery checks and notebook pass comprehensively. However, a critical regression makes all export functionality non-functional — users cannot save DRT results, EIS regression data, or figures. This alone blocks the release. The missing `docs/` folder and the Bayesian scientific adequacy warnings also warrant attention before public release.

