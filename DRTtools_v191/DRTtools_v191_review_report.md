# DRTtools v1.9.1 — End-User Review Report

**Reviewer:** Rahul Raj Pottekkat Raju  
**Date:** 31 July 2026  
**Assigned by:** Prof. Francesco Ciucci  
**Archive reviewed:** `DRTtools_1.9.1.zip`  
**Testing perspective:** New end-user, no prior DRTtools installation

---

## 1. System Information

| Item | Details |
|------|---------|
| Operating System | Windows 11 Enterprise (10.0 Build 26200) |
| MATLAB Version | R2025a (25.1.0.2973910) Update 1 |
| License Number | 87481 |
| Java | Not enabled |
| Available Toolboxes | MATLAB, Simulink, MATLAB Coder, MATLAB Compiler, Simscape, Simulink Coder, Simulink Real-Time, Statistics and Machine Learning Toolbox |
| Missing Toolbox | Optimization Toolbox (required by DRTtools) |

---

## 2. Archive Integrity

SHA-256 checksum verified:

- Computed: `36aeb6b86c3156478318c73dbec123c8df1c45592b2d2f94f51684a2633cb414`
- Expected: `36aeb6b86c3156478318c73dbec123c8df1c45592b2d2f94f51684a2633cb414`

✅ **Match confirmed — archive is intact.**

---

## 3. Archive Contents

```
DRTtools_1.9.1/
├── CITATION.cff
├── Contents.m
├── docs/
├── DRTtools.fig
├── DRTtools.m
├── drttools_info.m
├── examples/
│   └── quickstart.m
├── import file samples/
│   └── synthetic/
│       └── quickstart_two_rc.csv
├── LICENSE
├── MINIMUM_MATLAB_RELEASE
├── OUTPUT_SCHEMA_VERSION
├── README.md
├── SAMPLE_DATA_INVENTORY.tsv
├── SECURITY.md
├── setup_drttools.m
├── src/
├── SUPPORT.md
├── THIRD_PARTY_NOTICES.md
└── VERSION
```

---

## 4. Minimum MATLAB Release

```
type('MINIMUM_MATLAB_RELEASE')
R2021a
```

My MATLAB R2025a exceeds this requirement. ✅

---

## 5. Installation

### Step 1 — Extract ZIP into new folder
Extracted to: `C:\Users\bt724790\Documents\MATLAB\DRTtools_1.9.1\DRTtools_1.9.1`

### Step 2 — Start clean MATLAB session
Fresh MATLAB R2025a session started.

### Step 3 — Navigate to archive root
```matlab
cd('C:\Users\bt724790\Documents\MATLAB\DRTtools_1.9.1\DRTtools_1.9.1')
```

### Step 4 — Run setup
```matlab
run('setup_drttools.m')
```

**Result: ❌ FAILED**

```
Error using setup_drttools>validate_environment (line 274)
DRTtools requires Optimization Toolbox, but it is not installed.

Error in setup_drttools (line 42)
environment = validate_environment(environment, installation_root);

Error in run (line 112)
evalin('caller', strcat(scriptStem, ';'));
```

**Finding:** `setup_drttools.m` requires the Optimization Toolbox which is not available on this MATLAB installation. This is a hard blocker for the standard installation procedure.

### Step 5 — Workaround: addpath
```matlab
addpath(genpath(pwd))
```

This manually adds all DRTtools subfolders to the MATLAB path, bypassing the `setup_drttools.m` validation.

### Step 6 — Verify version
```matlab
drttools_info
```

**Result: ✅ SUCCESS**

```
ans = struct with fields:
    identifier: 'DRTtools_software_info_v1'
    software_name: 'DRTtools'
    software_version: '1.9.1'
    output_schema_version: 1
    matlab_release: '2026a'
    matlab_version: '25.1.0.2973910 (R2025a) Update 1'
    platform: 'PCWIN64'
```

---

## 6. GUI Launch

```matlab
DRTtools
```

**First attempt (without addpath): ❌ FAILED**

```
Unrecognized function or variable 'drt_capability_registry'.

Error in DRTtools>import1_OpeningFcn (line 71)
    drt_capability_registry('gui_kernels'))
```

**Finding:** Running `DRTtools` without first calling `addpath(genpath(pwd))` fails because internal modules in `src/` are not on the MATLAB path. The `setup_drttools.m` script normally handles this, but since it fails due to missing Optimization Toolbox, users must manually run `addpath(genpath(pwd))` first.

**Second attempt (with addpath): ✅ SUCCESS**

The GUI launched successfully after running `addpath(genpath(pwd))`.

**Usability Issue:** The README instructs users to run `setup_drttools.m` which fails due to missing Optimization Toolbox. A new user following the instructions exactly would be blocked at this step with no guidance on how to proceed. The workaround (`addpath(genpath(pwd))`) is not documented.

---

## 7. GUI Overview

The GUI launched with the following layout:
- **General Settings** panel (left side)
- **Tabs:** EIS Data, Magnitude, Phase, Re Part, Im Part, Residual-Re, Residual-Im, DRT, EIS Scores
- **Run section:** Simple Run, Bayesian Run, Hilbert Transform
- **Peak Analysis section**
- **Export Results section:** DRT/BHT/peaks, EIS Regression, Figure
- **Selection mode: Manual** (default)

---

## 8. Data Import Testing

### Attempt to import `quickstart_two_rc.csv`

Tested importing from multiple locations:

| Location | Result |
|----------|--------|
| `DRTtools archive folder` | ❌ Import Error |
| `C:\Users\...\Documents\` | ❌ Import Error |
| `C:\Users\...\Desktop\` | ❌ Import Error |
| `C:\Temp\` | ❌ Import Error |
| `C:\Users\...\AppData\Local\Temp\drttest\` | ❌ Import Error |

**Error message (all locations):**
```
Import Error
Could not import "[path]\quickstart_two_rc.csv":
The source filesystem did not provide a stable file
identity for safe import pinning.
```

**Finding:** The GUI cannot import CSV files from any location on this machine. The error appears to be related to the university enterprise Windows security configuration which prevents stable file identity pinning.

**Verification that data is valid:**
```matlab
data = readtable('import file samples/synthetic/quickstart_two_rc.csv');
disp(data(1:5,:))
```

```
frequency_hz    Z_prime_ohm    signed_Z_double_prime_ohm
____________    ___________    _________________________
10000           5.0013         -0.15978
5623.4          5.004          -0.2841
3162.3          5.0127         -0.50499
1778.3          5.04           -0.89679
1000            5.1259         -1.5879
```

The CSV file is valid and readable via MATLAB's `readtable()`. The import error is specific to the GUI's import mechanism, not the data file itself.

---

## 9. Quickstart Example

```matlab
run('examples/quickstart.m')
```

**Result: ❌ FAILED**

```
Error using setup_drttools>validate_environment (line 274)
DRTtools requires Optimization Toolbox, but it is not installed.

Error in setup_drttools (line 42)
Error in quickstart (line 20)
    setupInfo = setup_drttools();
```

The quickstart example also calls `setup_drttools()` internally and fails for the same reason.

---

## 10. Summary of Issues Found

| # | Issue | Severity | Details |
|---|-------|---------|---------|
| 1 | Optimization Toolbox required | 🚨 BLOCKER | `setup_drttools.m` fails without it — standard installation impossible |
| 2 | GUI launch fails without addpath | ❌ CRITICAL | `drt_capability_registry` not found without manual `addpath(genpath(pwd))` |
| 3 | CSV import pinning error | ❌ CRITICAL | GUI cannot import data from any filesystem location on enterprise Windows |
| 4 | setup_drttools.m not sufficient | ⚠️ USABILITY | Even if Optimization Toolbox available, path setup should work independently |
| 5 | Quickstart example blocked | ⚠️ USABILITY | `quickstart.m` requires Optimization Toolbox — cannot run as new user |
| 6 | No workaround documented | ⚠️ USABILITY | README does not mention `addpath(genpath(pwd))` as fallback |

---

## 11. What Could Be Tested

Due to the import blocker, the following could NOT be tested:
- ❌ Fitting (Simple Run, Bayesian Run, Hilbert Transform)
- ❌ DRT plot generation
- ❌ Export results
- ❌ Peak analysis

The following WERE tested:
- ✅ Archive integrity (SHA-256)
- ✅ Minimum MATLAB version check
- ✅ Installation attempt (failed — Optimization Toolbox missing)
- ✅ Workaround path setup
- ✅ Version confirmation (`drttools_info`)
- ✅ GUI launch (with workaround)
- ✅ Data file validity (via `readtable`)
- ✅ Import attempt (failed — file pinning error)
- ✅ Quickstart example (failed — Optimization Toolbox missing)

---

## 12. Recommendations

1. **Optimization Toolbox dependency:** Either make it optional for core DRT functionality, or clearly document it as a hard requirement in the README with instructions on what features are unavailable without it.

2. **Path setup independence:** `addpath(genpath(pwd))` should work as a standalone fallback when `setup_drttools.m` cannot complete. This should be documented in the README.

3. **File import pinning:** The import mechanism should be tested on enterprise Windows environments with strict filesystem security policies. A fallback import method or clearer error message with workaround instructions would improve usability.

4. **Quickstart example:** Should gracefully handle missing Optimization Toolbox rather than failing with a cryptic error.

---

## 13. Overall Assessment

As a new end-user on a standard university Windows machine, I was unable to complete the basic workflow (import → fit → plot → export) due to two blockers: missing Optimization Toolbox and file import pinning errors. The GUI itself launches and appears well-structured, but the installation instructions in the README do not account for environments without the Optimization Toolbox or enterprise filesystem restrictions. These issues should be addressed before wider distribution.

