# 2025-05-17 Section README Chinese Translation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Simplified Chinese parallel translations for the `workshops/2025-05-17/sections/**/README.md` files.

**Architecture:** Keep English README files unchanged and add `README.zh-CN.md` beside each section README. Reuse the already translated 2025-05 section wording where files match, then apply the 2025-05-17 Baseten / Qwen3 differences.

**Tech Stack:** Markdown documentation, TypeScript workshop commands, BAML snippets, `rtk`, `node`, `git`.

---

### Task 1: Add section README translations

**Files:**
- Create: `workshops/2025-05-17/sections/00-hello-world/README.zh-CN.md`
- Create: `workshops/2025-05-17/sections/01-cli-and-agent/README.zh-CN.md`
- Create: `workshops/2025-05-17/sections/02-calculator-tools/README.zh-CN.md`
- Create: `workshops/2025-05-17/sections/03-tool-loop/README.zh-CN.md`

- [x] **Step 1: Translate section READMEs**

Rules:
- Add attribution after each first heading: translator `0bipinnata0`, original path, and baseline commit `c62d647`.
- Preserve commands, code blocks, file paths, env var names, API identifiers, JSON keys, and output examples.
- Keep 2025-05-17-specific Qwen3 / Baseten content instead of the older OpenAI default wording.

### Task 2: Update maintenance notes

**Files:**
- Modify: `content/zh-CN/TRANSLATION.md`

- [x] **Step 1: Record phase 6 scope**

Add `workshops/2025-05-17/sections/**/README.zh-CN.md` to translated scope and record baseline commit `c62d647`.

### Task 3: Verify and commit

- [x] **Step 1: Run verification before commit**

Run:

```bash
rtk proxy git diff --cached --check
rtk node -e 'const fs=require("fs"); const files=["workshops/2025-05-17/sections/00-hello-world/README.zh-CN.md","workshops/2025-05-17/sections/01-cli-and-agent/README.zh-CN.md","workshops/2025-05-17/sections/02-calculator-tools/README.zh-CN.md","workshops/2025-05-17/sections/03-tool-loop/README.zh-CN.md","content/zh-CN/TRANSLATION.md","docs/superpowers/plans/2026-05-07-2025-05-17-section-readmes-zh-cn.md"]; let bad=[]; for(const f of files){const n=(fs.readFileSync(f,"utf8").match(/^```/gm)||[]).length; if(n%2) bad.push(`${f}: ${n} fences`);} if(bad.length){ console.error(bad.join("\n")); process.exit(1); } console.log(`fence check ok for ${files.length} files`);'
```

Expected: both commands exit 0.

Commit:

```bash
rtk git add workshops/2025-05-17/sections/**/README.zh-CN.md content/zh-CN/TRANSLATION.md docs/superpowers/plans/2026-05-07-2025-05-17-section-readmes-zh-cn.md
rtk git commit -m "Add Chinese 2025-05-17 section READMEs"
```
