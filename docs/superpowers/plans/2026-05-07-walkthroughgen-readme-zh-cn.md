# Walkthroughgen README Chinese Translation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Simplified Chinese parallel translation for `packages/walkthroughgen/readme.md`.

**Architecture:** Keep the English `readme.md` unchanged and add `packages/walkthroughgen/readme.zh-CN.md` beside it. Preserve YAML examples, commands, directory trees, package names, and generated markdown examples.

**Tech Stack:** Markdown documentation, YAML examples, `rtk`, `node`, `git`.

---

### Task 1: Add the walkthroughgen README translation

**Files:**
- Create: `packages/walkthroughgen/readme.zh-CN.md`

- [x] **Step 1: Translate walkthroughgen README**

Translate headings and prose from `packages/walkthroughgen/readme.md`.

Rules:
- Add attribution after the first heading: translator `0bipinnata0`, original path, and baseline commit `13e38f0`.
- Keep commands, YAML keys, file paths, package names, and output examples unchanged.
- Preserve emoji feature markers from the English README.
- Do not create links to package-local `CONTRIBUTING.md` or `LICENSE` files because those files are not present in the current tree.

### Task 2: Update maintenance notes

**Files:**
- Modify: `content/zh-CN/TRANSLATION.md`

- [x] **Step 1: Record phase 5 scope**

Add `packages/walkthroughgen/readme.zh-CN.md` to translated scope and record baseline commit `13e38f0`.

### Task 3: Verify and commit

- [x] **Step 1: Run verification before commit**

Run:

```bash
rtk proxy git diff --cached --check
rtk node -e 'const fs=require("fs"); const files=["packages/walkthroughgen/readme.zh-CN.md","content/zh-CN/TRANSLATION.md","docs/superpowers/plans/2026-05-07-walkthroughgen-readme-zh-cn.md"]; let bad=[]; for(const f of files){const n=(fs.readFileSync(f,"utf8").match(/^```/gm)||[]).length; if(n%2) bad.push(`${f}: ${n} fences`); } if(bad.length){ console.error(bad.join("\n")); process.exit(1); } console.log(`fence check ok for ${files.length} files`);'
```

Expected: both commands exit 0.

Commit:

```bash
rtk git add packages/walkthroughgen/readme.zh-CN.md content/zh-CN/TRANSLATION.md docs/superpowers/plans/2026-05-07-walkthroughgen-readme-zh-cn.md
rtk git commit -m "Add Chinese walkthroughgen README"
```
