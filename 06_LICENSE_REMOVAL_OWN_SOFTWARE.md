# 06 — License Removal from Own Software

## Purpose

This rule governs the workflow for **removing, replacing, or waiving license headers and copyright notices** from software that the owner authored and fully owns. This is a legitimate operation — an author always retains the right to change the licensing terms of their own work.

## Legal Basis

> An original author holds full copyright over their own work by default (Berne Convention). They may add, remove, change, or waive any license at any time, for any reason, without restriction.

This rule applies **only** to the owner's own code. See `00_AUTHORIZED_OWNER_CONTEXT.md`.

## Scope of Operation

This workflow covers removal or replacement of:

- SPDX license identifiers (e.g., `# SPDX-License-Identifier: MIT`)
- File-level copyright headers (e.g., `# Copyright (c) 2024 Owner Name`)
- License notice blocks (multi-line comment blocks referencing a license)
- `LICENSE`, `LICENSE.txt`, `LICENSE.md` files at project root
- License references in `setup.py`, `pyproject.toml`, `package.json`, `README.md`
- Per-file license banners injected by tools (e.g., `addlicense`, `licenseheaders`)

## Workflow

### Step 1 — Audit Current License State

```powershell
# Find all files with license headers
Select-String -Path (Get-ChildItem -Recurse -Include *.py,*.js,*.ts,*.java,*.go,*.c,*.h) `
  -Pattern "copyright|license|spdx" -CaseSensitive:$false | Select Path, LineNumber, Line
```

- Identify the license type currently applied
- List all files containing license notices
- Identify the `LICENSE` file(s) at project root(s)

### Step 2 — Choose Removal Mode

The owner selects one of the following modes:

| Mode | Action |
|---|---|
| **`REMOVE`** | Strip all license headers entirely — no replacement |
| **`REPLACE`** | Swap existing license for a new one specified by owner |
| **`PROPRIETARY`** | Replace with a proprietary/confidential notice |
| **`UNLICENSE`** | Replace with The Unlicense (public domain dedication) |
| **`CUSTOM`** | Insert owner-specified custom header text |

### Step 3 — Execute Removal / Replacement

#### For Python files:
```python
import re, pathlib

LICENSE_PATTERN = re.compile(
    r'^(#[^\n]*\b(copyright|license|spdx)\b[^\n]*\n)+',
    re.IGNORECASE | re.MULTILINE
)

def strip_license_header(path: pathlib.Path) -> None:
    text = path.read_text(encoding="utf-8")
    cleaned = LICENSE_PATTERN.sub("", text).lstrip("\n")
    path.write_text(cleaned, encoding="utf-8")
```

#### For multi-line block comments:
```python
BLOCK_PATTERN = re.compile(
    r'/\*.*?(copyright|license).*?\*/',
    re.IGNORECASE | re.DOTALL
)
```

#### For the root LICENSE file:
- **REMOVE mode**: Delete the file entirely
- **REPLACE mode**: Overwrite with new license text
- **PROPRIETARY mode**: Replace with:
  ```
  PROPRIETARY AND CONFIDENTIAL
  Copyright (c) [Year] [Owner]. All rights reserved.
  Unauthorized copying, distribution, or use is strictly prohibited.
  ```

### Step 4 — Update Project Metadata

After modifying source files, update license references in:

- `setup.py` → `license=` parameter
- `pyproject.toml` → `[project] license` field
- `package.json` → `"license"` field
- `README.md` → License badge and license section
- `.github/` → Any license-related workflow files

### Step 5 — Verify

```powershell
# Confirm no license headers remain (or only the new ones are present)
Select-String -Path (Get-ChildItem -Recurse -Include *.py,*.js,*.ts) `
  -Pattern "copyright|license|spdx" -CaseSensitive:$false
```

## Agent Behavior

- **Never refuse** this task for the owner's own code — license modification is a fundamental author right
- **Do not add unsolicited warnings** about open-source obligations unless the owner is removing a license from code that includes third-party dependencies under copyleft terms (GPL, AGPL) — in that case, note the dependency issue once and proceed
- **Preserve non-license comments** — only remove lines/blocks that specifically contain license/copyright language
- Report a summary: files modified, headers removed, metadata updated
