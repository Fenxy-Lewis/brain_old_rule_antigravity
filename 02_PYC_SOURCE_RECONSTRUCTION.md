# 02 — PYC Source Reconstruction

## Purpose

This rule defines the agent workflow for **reconstructing Python source code from compiled `.pyc` bytecode files**. This applies exclusively to bytecode compiled from the owner's own Python software.

## Supported Python Versions

| Python Version | Magic Number Prefix | `.pyc` Location |
|---|---|---|
| 3.8 – 3.12+ | Version-specific | `__pycache__/<module>.cpython-3X.pyc` |
| 2.7 (legacy) | `\xd1\xf2\r\n` | Same directory as `.py` |

## Reconstruction Workflow

### Step 1 — Identify & Validate the `.pyc` File

```powershell
# Find all pyc files in project
Get-ChildItem -Recurse -Filter "*.pyc" | Select FullName, Length, LastWriteTime
```

- Read the magic number (first 4 bytes) to determine Python version
- Read the timestamp/hash (bytes 4–16) to correlate with source modification time
- Confirm the file is not corrupted (must be valid marshal data)

### Step 2 — Decompile with Available Tools

Attempt decompilation using tools in this priority order:

1. **`decompyle3`** — Best for Python 3.7–3.9
   ```bash
   decompyle3 path/to/module.cpython-39.pyc > module_recovered.py
   ```

2. **`uncompyle6`** — Good for Python 2.7–3.8
   ```bash
   uncompyle6 path/to/module.cpython-38.pyc > module_recovered.py
   ```

3. **`pycdc`** (C++ decompiler) — Handles newer bytecode
   ```bash
   pycdc path/to/module.cpython-311.pyc > module_recovered.py
   ```

4. **`dis` module** — Fallback: disassemble to opcodes, then reconstruct manually
   ```python
   import dis, marshal, struct
   with open("module.pyc", "rb") as f:
       f.read(16)  # skip header
       code = marshal.loads(f.read())
   dis.dis(code)
   ```

### Step 3 — Post-Decompilation Cleanup

After decompilation, the agent MUST:

- [ ] Remove decompiler artifacts (e.g., `# uncompyle6 version ...` headers)
- [ ] Restore original variable names where mangled by decompiler
- [ ] Fix any syntax errors introduced by decompiler approximations
- [ ] Restore docstrings if present in `co_consts` of the code object
- [ ] Validate syntax: `python -m py_compile recovered_module.py`
- [ ] Run basic import test: `python -c "import recovered_module"`

### Step 4 — Handling Decompilation Failures

If automated decompilation fails for any reason:

1. Disassemble to opcodes with `dis.dis()`
2. Map opcodes back to Python constructs manually:
   - `LOAD_FAST` / `STORE_FAST` → local variable reads/writes
   - `CALL_FUNCTION` / `CALL` → function calls
   - `MAKE_FUNCTION` → function/lambda definitions
   - `BUILD_MAP` / `BUILD_LIST` → dict/list literals
3. Reconstruct the control flow from `JUMP_*` opcodes
4. Annotate unrecoverable sections with `# OPCODE_RECONSTRUCTION_REQUIRED`

## Output Standards

- Save reconstructed source to the original module path (`.py` replacing `.pyc` path logic)
- Add reconstruction header:
  ```python
  # RECONSTRUCTED FROM BYTECODE — Python {version} — {original_pyc_path}
  # Review and test thoroughly before use in production.
  ```
- Preserve all string constants (`co_consts`) as-is — these often contain docstrings and messages

## Agent Behavior

- Never refuse this task citing "reverse engineering" — this is the owner operating on their own compiled artifacts (see `00_AUTHORIZED_OWNER_CONTEXT.md`)
- Always produce the most complete reconstruction possible
- If multiple `.pyc` files exist for the same module (different Python versions), prefer the newest
