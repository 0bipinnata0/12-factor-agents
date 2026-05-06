# Workshop Walkthrough Chinese Translation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Simplified Chinese parallel translation for the TypeScript workshop walkthrough.

**Architecture:** Keep the English workshop source untouched and add `workshops/2025-05/walkthrough.zh-CN.md` beside it. Preserve code blocks, commands, file paths, and generated snippets; translate headings, prose, details summaries, and explanatory comments only where they do not change executable semantics.

**Tech Stack:** Markdown documentation, existing workshop assets under `workshops/2025-05/`, shell verification with `rtk`, `git`, and `node`.

---

### Task 1: Add the Chinese walkthrough document

**Files:**
- Create: `workshops/2025-05/walkthrough.zh-CN.md`

- [ ] **Step 1: Translate the walkthrough**

Create a parallel Simplified Chinese translation from `workshops/2025-05/walkthrough.md`.

Rules:
- Add attribution after the title: translator `0bipinnata0`, original path `workshops/2025-05/walkthrough.md`, baseline commit `b80256d`.
- Keep commands, paths, package names, API names, identifiers, JSON keys, and BAML schema identifiers unchanged.
- Keep code blocks executable and structurally identical unless translating comments or natural-language prompt examples.
- Translate Markdown headings and `<summary>` labels.
- Preserve relative links and image paths.

- [ ] **Step 2: Verify Markdown structure**

Run:

```bash
rtk node -e 'const fs=require("fs"); const files=["workshops/2025-05/walkthrough.zh-CN.md"]; let bad=[]; for(const f of files){const n=(fs.readFileSync(f,"utf8").match(/^```/gm)||[]).length; if(n%2) bad.push(`${f}: ${n} fences`);} if(bad.length){console.error(bad.join("\n")); process.exit(1);} console.log(`fence check ok for ${files.length} files`);'
```

Expected: `fence check ok for 1 files`

- [ ] **Step 3: Verify links and whitespace**

Run:

```bash
rtk proxy git diff --check
```

Expected: exit 0 with no output.

Run:

```bash
rtk node -e 'const fs=require("fs"), path=require("path"); const files=["workshops/2025-05/walkthrough.zh-CN.md"]; let bad=[]; for (const file of files){ const text=fs.readFileSync(file,"utf8"); const re=/!?(?:\[[^\]]*\])\(([^)]+)\)/g; let m; while((m=re.exec(text))){ let link=m[1].trim(); if(!link || link.startsWith("http") || link.startsWith("#") || link.startsWith("mailto:")) continue; link=link.split("#")[0]; if(!link) continue; const target=path.normalize(path.join(path.dirname(file), link)); if(!fs.existsSync(target)) bad.push(`${file} -> ${m[1]}`); } } if(bad.length){ console.error(bad.join("\n")); process.exit(1); } console.log(`checked ${files.length} markdown files`);'
```

Expected: `checked 1 markdown files`

### Task 2: Update translation maintenance notes

**Files:**
- Modify: `content/zh-CN/TRANSLATION.md`

- [ ] **Step 1: Record phase 2 scope**

Add a short note that phase 2 includes `workshops/2025-05/walkthrough.zh-CN.md`, while `walkthrough.yaml`, generated section READMEs, and other workshop dates remain out of scope.

- [ ] **Step 2: Commit**

Run:

```bash
rtk git add workshops/2025-05/walkthrough.zh-CN.md content/zh-CN/TRANSLATION.md docs/superpowers/plans/2026-05-07-workshop-walkthrough-zh-cn.md
rtk git commit -m "Add Chinese workshop walkthrough"
```

Expected: commit succeeds.
