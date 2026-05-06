# Template README Chinese Translation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Simplified Chinese parallel translation for the `create-12-factor-agent` generated template README.

**Architecture:** Keep `packages/create-12-factor-agent/template/README.md` unchanged and add `packages/create-12-factor-agent/template/README.zh-CN.md` beside it. Reuse the translated workshop wording where it matches, but preserve template-specific Baseten / Qwen3 setup text.

**Tech Stack:** Markdown documentation, TypeScript workshop commands, BAML setup notes, `rtk`, `node`, `git`.

---

### Task 1: Add the template README translation

**Files:**
- Create: `packages/create-12-factor-agent/template/README.zh-CN.md`

- [x] **Step 1: Translate template README**

Translate headings and prose from `packages/create-12-factor-agent/template/README.md`.

Rules:
- Add attribution after the first heading: translator `0bipinnata0`, original path, and baseline commit `92c89a5`.
- Keep commands, package names, file paths, env var names, API identifiers, JSON keys, and output examples unchanged.
- Convert internal factor links to `../../../content/zh-CN/*.md` where possible.
- Preserve external links and image links.

### Task 2: Update maintenance notes

**Files:**
- Modify: `content/zh-CN/TRANSLATION.md`

- [x] **Step 1: Record phase 4 scope**

Add `packages/create-12-factor-agent/template/README.zh-CN.md` to translated scope and record baseline commit `92c89a5`.

### Task 3: Verify and commit

- [x] **Step 1: Run verification before commit**

Run:

```bash
rtk proxy git diff --check
rtk node -e 'const fs=require("fs"), path=require("path"); const files=["packages/create-12-factor-agent/template/README.zh-CN.md","content/zh-CN/TRANSLATION.md","docs/superpowers/plans/2026-05-07-template-readme-zh-cn.md"]; let bad=[]; for(const f of files){ const n=(fs.readFileSync(f,"utf8").match(/^```/gm)||[]).length; if(n%2) bad.push(`${f}: ${n} fences`); } if(bad.length){ console.error(bad.join("\n")); process.exit(1); } console.log(`fence check ok for ${files.length} files`);'
```

Expected: both commands exit 0.

Commit:

```bash
rtk git add packages/create-12-factor-agent/template/README.zh-CN.md content/zh-CN/TRANSLATION.md docs/superpowers/plans/2026-05-07-template-readme-zh-cn.md
rtk git commit -m "Add Chinese template README"
```
