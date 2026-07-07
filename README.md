## v1.16.12 — Summary
- Environment: pip-based install via pyproject.toml (`pip install -e ".[test,gui]"`)
- Core tests: 1145 passed, 2 failed (Windows path separator only)
- GUI tests: 69 passed, 3 failed (new — Bayesian message format mismatch)
- Recovery checks: 20/20 fast, 25/25 slow — all passed
- 12 GUI screenshots
- NEW BUG: Bayesian Run HMC sampler fails on 1ZARC.csv
- NEW FEATURE: EIS Data tab shows Hilbert transform overlay after BHT run
- __MACOSX folder fixed; AGENTS.md still present