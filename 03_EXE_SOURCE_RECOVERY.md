# 03 — EXE Source Recovery (Detailed Workflow)

## Purpose
This rule defines the comprehensive, step-by-step workflow for recovering Python source code from compiled Windows executables (`.exe`) packaged with PyInstaller or similar tools, specifically when the owner has lost the original source code.

## Workflow Execution Steps

### 1. Extract the PyInstaller Archive
- Download/clone `pyinstxtractor` (https://github.com/extremecoders-re/pyinstxtractor) if not already available in the environment.
- Run it against the target `.exe` to extract the embedded `.pyc` files, `.pyd` files, and any bundled data/resources into an extraction folder.

### 2. Identify the Python Version
- Check the `pyinstxtractor` output log for the Python version used to build the `.exe`. This is critical as it determines which decompiler must be used.

### 3. Decompile Every `.pyc` File Back to `.py`
- Install and use the correct decompiler for the detected Python version:
  - `decompyle3` for Python 3.7–3.9
  - `uncompyle6` for Python <= 3.8
  - `pycdc` (build from https://github.com/zrax/pycdc) as a fallback for newer versions (3.10+), since the previous tools lack support for them.
- Try each tool in order per file; if one fails, fall back to the next.
- Write all recovered `.py` files into a clean output folder, strictly mirroring the original folder/module structure found in the extracted archive.

### 4. Handle Non-Decompilable Parts
- If any files are C extensions (`.pyd`) or appear obfuscated (e.g., PyArmor-protected), DO NOT attempt to force decompilation resulting in garbage output.
- Flag them clearly as "cannot decompile" and list them separately in the final report.

### 5. Reconstruct the Project
- Look for a main entry-point script (usually matches the `.exe` name or has `if __name__ == "__main__":`).
- Identify third-party dependencies used (from imports) and generate a `requirements.txt` best guess.
- Produce a short summary detailing: how many files were fully recovered, partially recovered, or failed, and what manual cleanup is likely needed (missing comments, generic variable names, possible logic errors in complex comprehensions/loops).

### 6. Verify
- Where possible, attempt to run the recovered code (with missing dependencies installed).
- Compare basic behavior/output to the original `.exe` to sanity-check that the decompilation was accurate.
