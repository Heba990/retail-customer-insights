# Troubleshooting Log

## 2026-07-14 — Jupyter Kernel Hang (Full Day Issue)

**Symptom:** Notebook cells stuck indefinitely in "Pending" / `[*]` state,
no error message, kernel appeared to hang forever.

**Root causes (two layered issues):**
1. A background Windows/Python auto-update silently switched the notebook's
   default interpreter from the working one (`Python313`, has pandas/numpy)
   to a fresh, empty Microsoft Store Python (3.13.14).
2. Even after reselecting the correct interpreter, the VS Code Jupyter
   extension itself was internally frozen and silently failed to start
   the kernel — confirmed via Output > Jupyter logs (no "Kernel successfully
   started" line, no new log entries after Restart).

**Diagnostic steps that worked:**
- Tested `import pandas` directly via Terminal (bypassing Jupyter) to
  isolate: environment problem vs. VS Code problem.
- Read Output > Jupyter log tail to see exactly where kernel startup stalled.
- Ruled out Avast Antivirus via disable-and-retest.

**Fix:**
1. `Ctrl+Shift+P` → `Developer: Reload Window` (resets VS Code extension state)
2. `Ctrl+Shift+P` → `Notebook: Select Notebook Kernel` → reselect correct
   Python313 interpreter
3. In Terminal, reinstall ipykernel targeting the correct python.exe:
& "C:\Users\majed\AppData\Local\Programs\Python\Python313\python.exe" -m pip install ipykernel --upgrade --force-reinstall


**Verified fixed:** Kernel started in 0.2s, `pandas.__version__` → `3.0.3`
(confirmed against PyPI as the actual latest release, May 2026).

**Lesson:** If kernel hangs with no error, check interpreter path match
first, then try Reload Window before deeper environment debugging.

