# pyDRTtools v2.2.14 — End-User Review Report

**Reviewer:** Rahul Raj Pottekkat Raju  
**Date:** 8 October 2026  
**Assigned by:** Prof. Francesco Ciucci  
**Archive reviewed:** pyDRTtools 2.2.14 code-check bundle (`pyDRTtools_2.2.14/`, `pyDRTtools_legacy/`, `READ_ME_FIRST.md`)  
**Testing perspective:** Hands-on end-user testing on native Windows 11, following `READ_ME_FIRST.md` and `docs/gui-checklist.md`

All results below were obtained by hand: terminal commands, the GUI, and reading the source with `findstr`. Screenshot numbers refer to the attached screenshots folder.

---

## 1. Environment

| Item | Value |
|---|---|
| OS | Windows 11 Enterprise 10.0 (Build 26200) |
| Display scaling | 125% (also tested 100% and 150%; 175% is the maximum this display offers) |
| Shell | Anaconda Prompt, env `pyDRT2214` |
| Python / PyQt5 / Qt | 3.12.14 / 5.15.11 / 5.15.2 |
| Matplotlib / NumPy / SciPy | 3.11.2 / 2.5.3 / 1.18.1 |
| pyDRTtools | 2.2.14, source sha256 `b94026b1813d0a8d…` (shown in the GUI footer) |
| Install command | `python -m pip install ".[gui,test]"` (regular install; editable install fails, see 3.1) |

---

## 2. Summary of Key Findings

| # | Finding | Severity |
|---|---|---|
| 1 | All GUI exports (DRT, EIS Regression, Figure) and `batch.run_simple_batch` fail on Windows with `[Errno 9] Bad file descriptor` at `export_bundle.py:86` (`os.fsync` on a file opened `"rb"`). About 100 fast tests fail for the same reason. All three exports worked in v2.2.7. | High |
| 2 | Automatic λ is not robust on bundled synthetic data: on 2ZARC0 and 2ZARC5 (same processes, same noise) the default publishes λ = 3.73 and λ = 1.96e-4 (~19,000× apart). The under-smoothed 2ZARC5 result, with ~5 spurious peaks, is labelled "fully qualified". | High |
| 3 | The requested GCV published its own λ in only 1 of 6 hand runs; the rest fell back to mGCV or blocked-kf-1se. The fallback is visible only behind a footer "Selector notes" button. | Medium |
| 4 | The Bayesian boundary-λ refusal works, but the GUI presents it as "Bayesian sampler failed" with sampler advice; the correct message is visible only in the raw diagnostics JSON. | Medium |
| 5 | Closing the window during an active Bayesian run exits immediately, with no cancel prompt (checklist step 11). | Medium |
| 6 | `pip install -e ".[gui,test]"` (the command in READ_ME_FIRST "Getting started") fails on Windows with a manifest error for `pyDRTtools/BHT.py`. | Medium |
| 7 | The four named long functions are 547–614 lines each; module-as-parameter and `setattr`-on-exception patterns are widespread. | Information |
| 8 | Slow suite on Windows: 13 characterisation tests skip ("measured on macos-aarch64-openblas"); 20 of 22 failures/errors come from the missing LIB data. | Information |

---

## 3. Installation and Archive

### 3.1 Installation

| Command | Result |
|---|---|
| `python -m pip install ".[gui,test]"` | Works; GUI launches with `pyDRTtools` |
| `python -m pip install -e ".[gui,test]"` | Fails every time: `Manifest source file is missing or unsafe: pyDRTtools/BHT.py`. The file's hash and size match, and `python setup.py build_py` succeeds. |

`READ_ME_FIRST.md` ("Getting started") and `docs/gui-checklist.md` step 1 both give the editable command, so a Windows user following the instructions hits this first.

### 3.2 Archive

Clean: no `.git`, `__MACOSX`, caches or build output. `LIB_data.txt` and `LIB_data_2.txt` are absent, as documented.

---

## 4. Automated Tests

### 4.1 Fast suite — `python -m pytest -q`

**Result:** 5485 passed, 150 failed, 4 skipped, 3 errors. **Runtime 2 h 13 min** (READ_ME_FIRST: about 4 min).

| Cause | Approx. count | Detail |
|---|---|---|
| fsync export bug | ~100 | `OSError: [Errno 9] Bad file descriptor` at `export_bundle.py:86`, `os.fsync(handle.fileno())` on a file opened `"rb"`. The recorded validation matrix lists only macOS rows. |
| Missing LIB data | ~15 | Expected (documented in READ_ME_FIRST) |
| Native crash | 3 | Access violation in `GUI._show_error` (`GUI.py:2862`) in `test_gui_canvas`, `test_gui_export`, `test_gui_window`. Cause unknown. The checklist says automated Windows tests use `QT_QPA_PLATFORM=offscreen`; running natively may matter (hypothesis only). |
| Other | 4 | `test_support_bundle` (redaction leaves `spectrum.csv` in the message); `test_gcv_fallback_regression` (noiseless ZARC assertion); two CLI tests (`assert 5 == 4`, `assert 5 == 0`) |

### 4.2 Slow suite — `python -m pytest -m slow -q`

**Result:** 65 passed, 17 failed, 5 errors, 19 skipped (106 slow tests, 2 min 53 s).

| Cause | Count | Detail |
|---|---|---|
| Missing `LIB_data.txt` / `LIB_data_2.txt` | 20 | Two of these surface as `DID NOT WARN`, because the `FileNotFoundError` occurs inside `pytest.warns(...)`, which hides the real cause |
| fsync bug | 1 | `test_batch_serializes_middle_failure_without_corrupting_neighbors`: `batch.run_simple_batch` → `_write_batch_summary` → `export_bundle.publish` → `os.fsync`. The bug also affects the batch API, not only the GUI. |
| Python 3.13 not installed | 1 | `test_bayesian_rng_contract.py:1137` **fails**, while `test_peak_issue_3.py:781` **skips** for the same reason |

Skip reasons (`-rs`): 13 × "characterisation measured on runtime row macos-aarch64-openblas; this row is windows-x86_64-openblas"; 4 × "Windows spawn timings make this subprocess test slow and flaky"; 1 × forkserver unavailable; 1 × Python 3.13 unavailable.

### 4.3 Tutorial recovery checks

20/20 passed. Note that they use a fixed `cv_type="custom"`, λ = 1e-4 (the two-ZARC cases use PWL, not the Gaussian default), so they do not exercise the default automatic λ. The peak-prominence gate (0.065) sits between a 5.1% noise lobe and a 7.8% true feature on the project's own fixtures and was raised from 0.05 after a solver change; the peak tests use a different gate (0.05).

---

## 5. Point 1 — Automatic λ (default `cv_type="GCV"`)

### 5.1 Default GCV on spectra with known DRT (GUI, Run simple)

Defaults: Gaussian, Combined Re-Im, Fitting with Inductance, 1st order, FWHM 0.5. Truth from `tutorial/data/metadata.json`: R_inf = 10 Ω, ZARC τ = 0.1 s (slow) and 1e-4 s (fast), σ = 0.05 Ω unless noiseless.

| Spectrum | Effective method and status | λ used | Result vs truth | Screenshots |
|---|---|---|---|---|
| `pyDRTtools/example_data/1ZARC.csv` | GCV, no dialog | 2.127e-4 | Peak at τ ≈ 0.1 s, R_inf 10.004; small side bumps | — |
| `1ZARC_exact.csv` (noiseless) | GCV and mGCV hit the lower edge 1e-8; blocked-kf-1se, `boundary_unresolved` | **6.81e-12** | One peak at 0.1 s, R_inf 10.00035; slight ripple on the left flank | 56–57 |
| `2ZARC0.csv` (slow 5 Ω, fast 50 Ω) | blocked-kf-1se, `flat_unresolved` | **3.73** | Both peaks at the right τ but broad; R_inf **9.711** (−0.29 Ω ≈ 6σ) | 52–53 |
| `2ZARC5.csv` (slow 55 Ω, fast 50 Ω) | mGCV, reported **"fully qualified"** | **1.96e-4** | Both peaks found, plus ~5 features absent from the truth (≈1e-6, 1e-5, 3e-3, 1, 8 s; up to ~2.6 Ω); R_inf 10.102 | 54–55 |
| `negami_data.txt` | blocked-kf-1se, `selection_diagnostic_only` | 0.316 | One main peak at τ ≈ 9e-3 s | 46–48 |
| `EIS data/csv files/data_1.csv` (real) | GCV and mGCV hit the lower edge 1e-8; blocked-kf-1se, `selection_diagnostic_only` | 2.91e-4 | Six peaks; R_inf 0.1104 Ω | 64 |

**Observations**
- The requested GCV published its own λ in 1 of 6 runs. Three different methods published across the six, and λ ranged from 6.8e-12 to 3.73.
- **2ZARC0 vs 2ZARC5:** near-identical test spectra get λ values ~19,000× apart. One is over-smoothed (R_inf biased by ~6σ), the other under-smoothed with spurious peaks, and the under-smoothed one carries the "fully qualified" label. Expected: λ of similar size on both, and a qualified result that does not add peaks.
- **1ZARC_exact:** the published λ (6.81e-12) is ~1,500× below the stated `lambda_interval` lower bound (1e-8). The DRT is still correct for noiseless data, but a λ outside the stated interval should be explained.
- L-curve on 1ZARC gave λ = 4.262e-4, R_inf 10.00406.

### 5.2 `negami_data.txt`: default vs custom λ (READ_ME_FIRST example)

| λ | Main peak (τ ≈ 9e-3 s) | Side features | Screenshot |
|---|---|---|---|
| 0.001 (custom) | ~21 Ω | Shoulders + 4–5 side peaks up to ~1 Ω | 49 |
| 0.01 (custom) | ~19 Ω | Smooth left tail; 2 small right bumps ≤ 0.8 Ω | 50 |
| 0.03 (custom) | ~18 Ω | Faint left shoulder; 1 small right bump ~0.7 Ω | 51 |
| 0.316 (default → blocked-kf-1se) | ~15.7 Ω, broader | 2 small bumps ~0.5 Ω | 46 |

R_inf stays 10.014–10.015 Ω throughout. This confirms by hand that the default lands at λ ≈ 0.32, and that 0.01–0.03 gives essentially one peak while 0.316 lowers and broadens it (~15%), consistent with over-smoothing. The fit-error claim (0.32 doubles the error) was **not verified**, because the GUI shows no error value.

### 5.3 Envelope suggestion (`tests/fixtures/meyer_synthetic_noiseless.csv`)

| Noise | Proposed λ | Ratio (threshold 0.3) | Notes |
|---|---|---|---|
| Plug-in (residual RMS, σ ≈ 0.033) | 2.154 | 0.288 | The dialog itself warns this can merge the two peaks |
| Supplied σ_n = 1e-5 (and 1e-5,1e-5) | 1e-08 | 0.0698 | Run simple gives two resolved peaks (τ ≈ 8e-4 and 3e-2 s), R_inf 0.0991 Ω |

- The selection rule (`smallest_scanned_lambda_with_ratio_at_or_below_threshold`) returns the grid minimum when every candidate passes, as here, without telling the user.
- At λ = 1e-08 the residual RMS is ~1.4e-3, about 140× the supplied σ = 1e-5. The bundle marks this `scientifically_calibrated: true` together with `supplied_noise_independently_verified: false`, and nothing flags the mismatch.

### 5.4 Not testable with bundled data

No Havriliak–Negami or three-ZARC spectrum is bundled, and only two noise levels (0 and 0.05 Ω) are available, so that part of the point 1 request could not be tested.

---

## 6. Point 2 — Complexity

Measured with `findstr` (top-level `def`/`class` line numbers and `find /c`); lengths are approximate.

| Function | Lines | Location | Share of file |
|---|---|---|---|
| `parameter_selection.select_grouped_kf_1se` | ~614 | 1768–2381 | of ~3.9k |
| `peak_run.execute_peak_analysis` | ~587 | 522–1108 | 53% of 1108 |
| `bht_run.execute_bht_run` | ~575 | 591–1165 | 49% of 1165 |
| `basics.optimal_lambda` | ~547 | 831–1377 | 39% of 1406 |

Other long functions: `_evaluate_gcv_candidate` ~263, `_grouped_qualification_checks` ~194, `blocked_log_frequency_folds` ~162, `_minimize_scouted_score` ~158. The four named functions together are ~2,320 lines, more than half the size of the legacy package.

- **Package size:** 68,087 lines in 78 `.py` files (READ_ME_FIRST: 68k; legacy 4.2k).
- **Modules passed as parameters:** `basics_module` on 71 lines in 6 files (`bayesian_run`, `bht_run`, `bht_validation`, `ridge_core`, `runs`, `simple_run`); `bht_module` on 40 lines in 3 files (`bht_run`, `bht_validation`, `runs`).
- **Fields attached to exceptions with `setattr`:** 11 lines in 4 files (`simple_run.py` 1507–1531, `api.py` 1739/1973/2021, `batch.py` 1709, `result_access.py` 472/526). It includes two loops (`api.py:2021`, `batch.py:1709`) that copy up to 16 named diagnostic fields from one exception to another. One exception class with a diagnostics payload would replace both the `setattr` calls and the copy lists.
- `basics.py` has a module-level `__getattr__` (line 416).
- **Not established here:** whether the module parameters exist to avoid circular imports or for test injection, and whether every affected exception is a plain `RuntimeError` (two `RuntimeError` subclasses exist in `parameter_selection.py`).

---

## 7. Point 3 — Usability

**`cv_type` choices.** The GUI dropdown has 14 entries, ungrouped and without descriptions: GCV, mGCV, rGCV, legacy-gcv, legacy-mgcv, legacy-rgcv, qp-gcv, qp-mgcv, LC, re-im, kf, grouped-kf-1se, blocked-kf-1se, custom (screenshots 62–63). The qp-* selectors "never qualify" per the slow-test docstrings.
*Suggestion:* by default show GCV (with its cascade), LC and custom; move legacy-*, qp-*, re-im, kf and the kf-1se variants under "Advanced".

**Warnings on real spectra.** Both real spectra (negami, data_1) and three of four synthetic ones produced selector notes (1–3 per run). Which method actually chose λ, and its status (`flat_unresolved`, `boundary_unresolved`, …), is visible only via the footer **Selector notes** button → **Show Details** (two clicks; the first level is empty). The λ field shows only the number.
*Suggestion:* show the effective method and status next to "Regularization parameter used".

**Other usability observations**

| Observation | Where |
|---|---|
| Disabled Regularization parameter field keeps a stale value (0.001) while "used" shows 0.316; no tooltip | GCV runs |
| Selector notes text uses API names (`reg_param`, `lambda_selection`, "selector trace") | 47–48, 53, 55, 57 |
| User-facing text names "Quentin's noiseless two-R//C spectrum" | Envelope dialogs 35–36 |
| "unknown number of 109 directions" is self-contradictory | Shape-factor error 21 |
| Expected, actionable validation error also prints a full traceback to the console (`linalg_utils.py:270`) | data_1 Shape Factor 0.5 |
| Status-bar notices and the BHT Details button are cut off at the window edge | 19, 25 |
| Fitting objective text truncated at every scale; Bayesian seed hides leading digits at 100% / narrow windows | 24, 27 |
| R_inf field goes blank after every Bayesian run, although the MAP is available | 05, 20, 58, 60 |
| R_inf changes after BHT (0.1099976 vs 0.1105045 from Simple) without saying which run it refers to; λ field cleared | 22–23 |
| Bayesian seed is kept between runs and regenerates per session | 19–20 |
| RBF Shape Control appeared to switch between Shape Factor and FWHM Coefficient without confirmed input; FWHM once showed `0.499999999999999`. Shape Factor ε = 3.6157 equals FWHM 0.5's effective ε on this grid, which may explain it (hypothesis) | — |
| Copy diagnostics (Bayesian) includes the full 810-point DRT, τ grid and fits several times; too long for an issue report | 59 |
| Support bundle repeats per-candidate noise provenance 25 times | Envelope step 4 |

---

## 8. Point 4 — Do the slow tests check physics or frozen outputs?

**Both.** By name, the 106 slow tests split roughly into thirds:

- **Frozen values (~38):** "frozen…on_the_measured_row", 16-digit λ "characterization" values, frozen candidate lists, "match legacy" (refactor equivalence).
- **Physics / properties (~39):** A-matrix vs quadrature for all 8 RBFs, M PSD, KKT conditions, input-order covariance, unit invariance, one- and two-ZARC recovery (against frozen controls), QP selectors never qualify.
- **Workflow (~28):** parallel chains, publishing through API and GUI controller, batch, band-quality presence.

The frozen characterisations are explicitly platform snapshots: on Windows, 13 skip because they were "measured on runtime row macos-aarch64-openblas". Together with the missing LIB data, almost none of the frozen tests run on Windows, so the Windows evidence rests on the property and workflow tests. Five of six parallel-chain tests also skip on Windows. (Counts are by test name and file; test bodies were not reviewed individually.)

---

## 9. Point 5 — GUI Checklist Pass (`docs/gui-checklist.md`)

| Step | Result | Notes |
|---|---|---|
| 1 Install and launch | Partial | Regular install and launch work; editable install fails (3.1) |
| 2 Import 1ZARC | Pass | `pyDRTtools/example_data/1ZARC.csv` used instead of `tutorial/data/1ZARC.csv`; labels legible |
| 3 Simple run | Pass | R_inf 10.004, peak at τ ≈ 0.1 s |
| 4 Selector switch | Partial | LC 4.262e-4, custom 1e-3; field disabled/enabled correctly; **no tooltip**; Selector notes present and copyable (47–48); console warning not recorded |
| 5 Bayesian | Partial | Cancel run visible, determinate progress (e.g. "500 of 3001"), cancel works (16). Band never qualifies on 1ZARC with 4 chains at 95%/3001, 99%/2000, 99%/3001 (same five reason codes) (19–20). 99% at 2000 draws first shows a **Yes/No dialog** not described in the checklist, then the status-bar notice, which clears at 3001 (19–20). Boundary refusal works but is **mislabelled** (58–59); override completes (60). Exploratory band off by default and labelled, but too narrow to see on noiseless data (61). **No draw-count tooltip.** Sampler-failure views and the HN adjusted-target case not reachable with bundled data. |
| 6 BHT/KK | Partial | Finite scores; label and Details text match exactly. **No "no universal hard pass threshold" statement** found. S_HD only ~65 on clean 1ZARC; "Validation attempted: False" is not explained |
| 7 Exports | **Fail** | All three exports fail (fsync); sub-checks could not run. The plot-toolbar save icon writes a valid PNG (28) |
| 8 data_1 Shape Factor / FWHM | Pass | Shape Factor 0.5 rejected with an actionable error and no partial result (21); FWHM 0.5 completes, ε = 7.2307 (22). `LIB_data.txt` not in the archive |
| 9 Resize and scaling | Partial | BHT view clean at 100/125/150% (23–25); DRT placeholder wraps (27); toolbar PNG save works at 125% (28). 200% not offered by this display (max 175%, not tested) |
| 10 negami BHT/KK | Pass | Optimizer completes, scores-only, placeholder shown, old λ cleared (29–31). BHT **Copy diagnostics** lacks identity and options; the **Help-menu support bundle** contains them, with no paths or raw data |
| 11 Close | Partial | Idle close clean, no leftover process. **Close during an active run: no cancel prompt**; the window closes immediately (32) |
| Envelope review 1–4 | Pass (mostly) | Labels, review dialog, Cancel/Escape, peak-merging warning, supplied-noise path, `custom_user_fixed`, input validation (0, negative, inf, nan, blank, malformed, three values) all as described (33–44). Scan too fast to cancel; export blocked by fsync (45) |

---

## 10. Open Issues

| ID | Severity | Area | Summary | Reproduce |
|---|---|---|---|---|
| 1 | High | Export | `[Errno 9] Bad file descriptor` at `export_bundle.py:86` (`os.fsync` on `"rb"` handle) breaks all GUI exports and `batch.run_simple_batch` on Windows; worked in 2.2.7. Suggested fix: open staged files `"rb+"` before fsync, or fsync where the file is written. | Any result → Export DRT / EIS / Figure |
| 2 | High | λ selection | Default λ differs ~19,000× between 2ZARC0 (3.73, `flat_unresolved`) and 2ZARC5 (1.96e-4, "fully qualified", ~5 spurious peaks) | `tutorial/data/2ZARC0.csv`, `2ZARC5.csv`, default GCV, Run simple |
| 3 | Medium | λ selection | Requested GCV publishes in 1 of 6 runs; fallback method and status visible only behind Selector notes | Section 5.1 |
| 4 | Medium | λ selection | blocked-kf-1se publishes λ = 6.81e-12, outside the stated `lambda_interval` (1e-8, 100) | `1ZARC_exact.csv`, default GCV |
| 5 | Medium | Bayesian GUI | Boundary-λ refusal shown as "Bayesian sampler failed" with sampler advice; correct `actionable_error` only in raw JSON | `1ZARC_exact.csv`, GCV, Run Bayesian |
| 6 | Medium | GUI | Closing during an active Bayesian run gives no cancel prompt | Run Bayesian → window X |
| 7 | Medium | Install | Editable install fails: manifest error for `pyDRTtools/BHT.py` | `pip install -e ".[gui,test]"` |
| 8 | Medium | Tests | Native access violation in `GUI._show_error` (`GUI.py:2862`) in three GUI tests | `python -m pytest -q` |
| 9 | Medium | Bayesian | Band never qualifies on clean 1ZARC with 4 chains at the documented settings | 1ZARC, 95%/3001, 99%/3001 |
| 10 | Low | Tests | Python 3.13 test fails instead of skipping; two LIB failures appear as `DID NOT WARN` | `python -m pytest -m slow -q` |
| 11 | Low | Tests | `test_support_bundle` redaction, `test_gcv_fallback_regression`, two CLI assertions (`5 == 4`, `5 == 0`) | Fast suite |
| 12 | Low | GUI | Missing tooltips: Regularization parameter (step 4), draw count (step 5) | Hover |
| 13 | Low | GUI | No "no universal hard pass threshold" statement in the BHT panel or details | BHT/KK run |
| 14 | Low | GUI | BHT Copy diagnostics lacks package/platform identity and options (Help bundle has them) | negami BHT → BHT Details |
| 15 | Low | GUI | R_inf blank after Bayesian; R_inf changes after BHT without label; stale λ in the disabled field | Section 7 |
| 16 | Low | Text | "unknown number of 109 directions"; "Quentin's" in user text; API names in GUI notes; full traceback for an expected error | Section 7 |
| 17 | Low | Layout | Truncated status-bar text, BHT Details button, Fitting objective, Bayesian seed | Section 7 |
| 18 | Low | Docs | Fast suite took 2 h 13 min vs "about 4 min"; 99% Yes/No dialog not in the checklist | Sections 4.1, 9 |
| 19 | Info | Envelope | With supplied σ every candidate passes and the grid minimum is returned silently; residual RMS ~140× the supplied σ is not flagged | Section 5.3 |

---

## 11. Not Tested / Limitations

- Havriliak–Negami, three-ZARC and additional noise levels (no bundled data).
- `LIB_data.txt` / `LIB_data_2.txt` import and all LIB-based tests (not in the archive).
- Export sub-checks (manifest reader, replacement, legacy files): blocked by issue 1.
- Bayesian sampler-failure views and the HN adjusted-target case (no bundled spectrum named in the checklist).
- 200% scaling (display maximum 175%; 175% not tested).
- The fit-error comparison on negami (no error value in the GUI).
- Whether selector warnings print to the console during GUI runs (not recorded).

---

## 12. Overall Assessment

As an end-user testing pyDRTtools v2.2.14 on native Windows 11, the core analysis workflow (import → Simple run → Bayesian → BHT/KK → peaks) runs, and the new safeguards are thorough: actionable input validation, preserved MAP on failure, envelope review with explicit noise acknowledgment, and detailed diagnostics. However, the release is not yet usable on Windows for saving results, because every export path (GUI and batch) fails on the fsync call, a regression from v2.2.7. Scientifically, the default automatic λ is not robust: on near-identical bundled test spectra it swings between over- and under-smoothing, and the "qualified" label does not track physical correctness. Most diagnostic information is accurate but hidden behind footer buttons and raw JSON, and the selector dropdown exposes 14 options without guidance. The code-size concern in READ_ME_FIRST is confirmed (four functions of 547–614 lines each). Fixing the export bug, making the effective λ method and status visible in the main window, and simplifying the selector list would give the largest improvement for users.
