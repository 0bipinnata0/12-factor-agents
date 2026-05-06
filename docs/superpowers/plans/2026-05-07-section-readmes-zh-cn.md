# Section README Chinese Translation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Simplified Chinese parallel translations for the TypeScript workshop section README files.

**Architecture:** Keep every existing `README.md` untouched and add sibling `README.zh-CN.md` files under `workshops/2025-05/sections/**`. Reuse translated chapter prose from `workshops/2025-05/walkthrough.zh-CN.md` where the section README content matches the main walkthrough, then handle the `sections/final` README separately because it has a compact combined structure.

**Tech Stack:** Markdown documentation, existing workshop assets, `rtk`, `node`, `git`.

---

### Task 1: Generate section translations from the main translated walkthrough

**Files:**
- Create: `workshops/2025-05/sections/00-hello-world/README.zh-CN.md`
- Create: `workshops/2025-05/sections/01-cli-and-agent/README.zh-CN.md`
- Create: `workshops/2025-05/sections/02-calculator-tools/README.zh-CN.md`
- Create: `workshops/2025-05/sections/03-tool-loop/README.zh-CN.md`
- Create: `workshops/2025-05/sections/04-baml-tests/README.zh-CN.md`
- Create: `workshops/2025-05/sections/05-human-tools/README.zh-CN.md`
- Create: `workshops/2025-05/sections/06-customize-prompt/README.zh-CN.md`
- Create: `workshops/2025-05/sections/07-context-window/README.zh-CN.md`
- Create: `workshops/2025-05/sections/08-api-endpoints/README.zh-CN.md`
- Create: `workshops/2025-05/sections/09-state-management/README.zh-CN.md`
- Create: `workshops/2025-05/sections/10-human-approval/README.zh-CN.md`
- Create: `workshops/2025-05/sections/11-humanlayer-approval/README.zh-CN.md`
- Create: `workshops/2025-05/sections/12-humanlayer-webhook/README.zh-CN.md`

- [ ] **Step 1: Extract translated chapter ranges**

Use the translated main walkthrough as the source of truth for headings, prose, relative links, and code blocks.

- [ ] **Step 2: Add attribution**

Each generated file starts with the translated title and an attribution block naming translator `0bipinnata0`, source file path, and baseline commit `58b8f09`.

### Task 2: Add final README translation

**Files:**
- Create: `workshops/2025-05/sections/final/README.zh-CN.md`

- [ ] **Step 1: Translate compact final README**

Translate headings and prose while preserving commands, code blocks, file paths, API identifiers, JSON keys, and code comments unless comments are purely explanatory.

### Task 3: Update maintenance notes and verify

**Files:**
- Modify: `content/zh-CN/TRANSLATION.md`

- [ ] **Step 1: Record phase 3 scope**

Add `workshops/2025-05/sections/**/README.zh-CN.md` as translated, while keeping section source files and YAML files out of scope.

- [ ] **Step 2: Verify**

Run:

```bash
rtk proxy git diff --check
rtk node -e 'const fs=require("fs"), path=require("path"); const files=["content/zh-CN/TRANSLATION.md",...fs.readdirSync("workshops/2025-05/sections",{recursive:true}).filter(f=>f.endsWith("README.zh-CN.md")).map(f=>"workshops/2025-05/sections/"+f)]; let bad=[]; for(const f of files){ const n=(fs.readFileSync(f,"utf8").match(/^```/gm)||[]).length; if(n%2) bad.push(`${f}: ${n} fences`); } if(bad.length){ console.error(bad.join("\n")); process.exit(1); } console.log(`fence check ok for ${files.length} files`);'
```

Expected: both commands exit 0.

- [ ] **Step 3: Commit**

Run:

```bash
rtk git add content/zh-CN/TRANSLATION.md docs/superpowers/plans/2026-05-07-section-readmes-zh-cn.md workshops/2025-05/sections
rtk git commit -m "Add Chinese section README translations"
```
