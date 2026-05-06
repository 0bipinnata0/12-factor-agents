# 2025-07-16 Python Notebook Chinese Translation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Simplified Chinese parallel source YAML and generated notebook for the 2025-07-16 Python/Jupyter workshop.

**Architecture:** Keep the English `walkthrough_python_enhanced.yaml` and `workshop_final.ipynb` unchanged. Add `walkthrough_python_enhanced.zh-CN.yaml` by translating YAML `title` and `text` fields while preserving executable steps, file paths, commands, and `run_main` inputs. Add a small backwards-compatible `locale: zh-CN` branch to `walkthroughgen_py.py` so the generator's built-in BAML setup explanation is localized when producing `workshop_final.zh-CN.ipynb`.

**Tech Stack:** YAML, Jupyter notebook JSON, Python notebook generator, `uv`, `rtk`, `git`.

---

### Task 1: Add translated notebook source YAML

**Files:**
- Create: `workshops/2025-07-16/walkthrough_python_enhanced.zh-CN.yaml`

- [x] **Step 1: Translate YAML display content**

Translate only user-facing `title` and `text` fields.

Rules:
- Add a top-level translator note field under the main `text` or title introduction: translator `0bipinnata0`, original path, and baseline commit `aa05c33`.
- Preserve executable keys and values: `file`, `fetch_file`, `command`, `baml_setup`, `run_main.args`, `run_main.kwargs`, image URLs, and code identifiers.
- Convert links to 12-factor content pages from GitHub English pages to relative `../../content/zh-CN/*.md` links where a Chinese page exists.
- Set the YAML target notebook path to `./build/workshop-2025-07-16.zh-CN.ipynb`.

### Task 2: Localize generator setup cell

**Files:**
- Modify: `workshops/2025-07-16/walkthroughgen_py.py`

- [x] **Step 1: Add locale support for BAML setup explanation**

Update the generator so `locale: zh-CN` in the YAML uses a Chinese BAML setup markdown cell while the default English output stays unchanged.

### Task 3: Generate translated notebook

**Files:**
- Create: `workshops/2025-07-16/workshop_final.zh-CN.ipynb`

- [x] **Step 1: Run the existing generator**

Run from `workshops/2025-07-16`:

```bash
rtk uv run python walkthroughgen_py.py walkthrough_python_enhanced.zh-CN.yaml -o workshop_final.zh-CN.ipynb
```

Expected: command exits 0 and writes `workshop_final.zh-CN.ipynb`.

### Task 4: Update maintenance notes

**Files:**
- Modify: `content/zh-CN/TRANSLATION.md`

- [x] **Step 1: Record phase 8 scope**

Add `workshops/2025-07-16/walkthrough_python_enhanced.zh-CN.yaml` and `workshops/2025-07-16/workshop_final.zh-CN.ipynb` to translated scope and record baseline commit `aa05c33`.

### Task 5: Verify and commit

- [x] **Step 1: Run verification before commit**

Run:

```bash
rtk proxy git diff --cached --check
rtk uv run --project workshops/2025-07-16 python - <<'PY'
import json, yaml
from pathlib import Path
yaml.safe_load(Path("workshops/2025-07-16/walkthrough_python_enhanced.zh-CN.yaml").read_text())
nb = json.loads(Path("workshops/2025-07-16/workshop_final.zh-CN.ipynb").read_text())
assert nb["cells"], "notebook has no cells"
assert nb["cells"][0]["cell_type"] == "markdown"
print(f"notebook cells: {len(nb['cells'])}")
PY
```

Expected: both commands exit 0.

Commit:

```bash
rtk git add workshops/2025-07-16/walkthrough_python_enhanced.zh-CN.yaml workshops/2025-07-16/workshop_final.zh-CN.ipynb workshops/2025-07-16/walkthroughgen_py.py content/zh-CN/TRANSLATION.md docs/superpowers/plans/2026-05-07-2025-07-16-python-notebook-zh-cn.md
rtk git commit -m "Add Chinese 2025-07-16 Python notebook"
```
