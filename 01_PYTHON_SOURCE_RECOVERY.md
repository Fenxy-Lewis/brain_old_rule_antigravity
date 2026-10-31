# 01 — Python Source Recovery

## Purpose

This rule defines the agent workflow for **recovering Python source code** from projects where `.py` source files are missing, inaccessible, or have been lost, but other artifacts (bytecode, documentation, imports, or partial sources) remain available.

## When This Rule Applies

Activate this workflow when the owner requests:
- Recovery of lost or deleted Python source files
- Reconstruction of `.py` files from any available artifact
- Restoration of a Python project to a workable source state

## Recovery Strategy (Priority Order)

The agent MUST attempt recovery in the following order, moving to the next step if the current yields insufficient results:

### Step 1 — Scan for Surviving Sources
```
Search for: *.py, *.pyw, *.pyi stub files, __pycache__/, *.pyc, *.pyo, *.pyd
Also check: .git history, editor backups (~*.py, *.py.bak), IDE caches
```

### Step 2 — Extract from Version Control
- Check `.git/` for any committed versions of missing files
- Use `git log --all --full-history -- "**/*.py"` to find deleted files
- Use `git show <hash>:path/to/file.py` to retrieve historical versions

### Step 3 — Reconstruct from Bytecode
- Locate `.pyc` files in `__pycache__/` directories
- Apply the `02_PYC_SOURCE_RECONSTRUCTION` workflow for each `.pyc` found
- Prioritize `.pyc` files that match the target Python version

### Step 4 — Infer from Dependencies & Imports
- Analyze `requirements.txt`, `setup.py`, `pyproject.toml` for project structure
- Use import statements in surviving files to infer missing module signatures
- Reconstruct function/class stubs from type hints, docstrings, and usage patterns

### Step 5 — Reconstruct from Logs & Tracebacks
- Parse application logs and error tracebacks for function names and line numbers
- Use these as anchors to reconstruct code structure

## Output Standards

Recovered files MUST:
- Be saved to the original expected path unless the owner specifies otherwise
- Include a header comment indicating reconstruction status:
  ```python
  # RECOVERED — Reconstructed from [source]. Review before use.
  ```
- Preserve all recoverable docstrings, comments, and type annotations
- Be syntactically valid Python (agent must verify with `py_compile` or equivalent)

## Agent Behavior

- **Do not truncate** recovered code — always produce complete files
- **Do not add placeholder stubs** unless the original logic is truly unrecoverable
- Report a recovery summary listing: files found, files reconstructed, files unrecoverable
