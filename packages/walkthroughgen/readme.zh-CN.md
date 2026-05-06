# Walkthroughgen

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`packages/walkthroughgen/readme.md`，基准提交：`13e38f0`

Walkthroughgen 是一个用于创建 walkthrough、教程、README 和文档的工具。它可以从简单的 YAML 配置生成 markdown 和工作目录，帮助你维护分步骤指南。

## 功能

- 📝 **Markdown 生成**：创建包含 diff、代码块和可折叠章节的 markdown 文件
- 📁 **工作目录**：为 walkthrough 的每个章节生成独立目录
- 🔄 **增量变更**：追踪并展示步骤之间的变化
- 🎯 **多个目标**：输出到 markdown、章节文件夹和最终项目状态
- 📦 **文件管理**：复制文件、创建目录并运行命令
- 🔍 **丰富 Diff**：展示不同文件版本之间有意义的 diff
- 📚 **章节 README**：为每个章节生成文档

## 安装

```bash
npm install -g walkthroughgen
```

## 快速开始

1. 创建 `walkthrough.yaml` 文件：

```yaml
title: "My Tutorial"
text: "A step-by-step guide"
targets:
  - markdown: "./walkthrough.md"
    onChange:
      diff: true
      cp: true
  - folders:
      path: "./by-section"
      final:
        dirName: "final"
sections:
  - name: setup
    title: "Initial Setup"
    steps:
      - file: {src: ./files/package.json, dest: package.json}
      - command: "npm install"
```

2. 运行生成器：

```bash
walkthroughgen generate walkthrough.yaml
```

## 目录结构

一个典型的 walkthrough 项目大致如下：

```
my-tutorial/
├── walkthrough/          # Source files for each step
│   ├── 00-package.json
│   ├── 01-index.ts
│   └── 02-config.ts
├── walkthrough.yaml     # Walkthrough configuration
└── build/              # Generated output
    ├── by-section/    # Section-by-section working directories
    │   ├── 00-setup/
    │   └── 01-config/
    ├── final/         # Final project state
    └── walkthrough.md # Generated markdown
```

## Walkthrough.yaml 配置

### 顶层字段

- `title`：walkthrough 标题
- `text`：介绍文本
- `targets`：输出配置
- `sections`：教程章节

### 目标

#### Markdown 目标

```yaml
targets:
  - markdown: "./output.md"
    onChange:
      diff: true  # Show diffs for changed files
      cp: true    # Show cp commands
    newFiles:
      cat: false  # Don't show file contents
      cp: true    # Show cp commands
```

#### Folders 目标

```yaml
targets:
  - folders:
      path: "./by-section"        # Base path for section folders
      skip: ["cleanup"]          # Sections to skip
      final:
        dirName: "final"        # Name for final state directory
```

### 章节

每个章节代表教程中的一个逻辑步骤：

```yaml
sections:
  - name: setup              # Used for folder naming and skip array
    title: "Initial Setup"   # Display title
    text: "Setup steps..."   # Section description
    steps:
      # ... steps ...
```

### 步骤

步骤定义要执行的动作：

#### 文件复制

```yaml
steps:
  - text: "Copy package.json"
    file:
      src: ./files/package.json
      dest: package.json
```

#### 创建目录

```yaml
steps:
  - text: "Create src directory"
    dir:
      create: true
      path: src
```

#### 命令执行

```yaml
steps:
  - text: "Install dependencies"
    command: "npm install"
    incremental: true  # run when building up folders target
```

#### 命令结果

```yaml
steps:
  - command: "npm run test"
    results:
      - text: "You should see:"
        code: |
          All tests passed!
```

## 生成输出

### Markdown 功能

- **文件 Diff**：展示不同版本之间的变化
- **复制命令**：生成容易跟随的文件复制说明
- **可折叠章节**：隐藏 / 展示文件内容
- **代码高亮**：为多种语言提供语法高亮

示例 markdown 输出：

~~~markdown
# Initial Setup

Copy the package.json:

    cp ./files/package.json package.json

<details>
<summary>show file</summary>

```json
{
  "name": "my-project",
  "version": "1.0.0"
}
```
</details>

Install dependencies:

    npm install

You should see:

    added 123 packages
~~~

### 章节文件夹

`folders` 目标会创建：

1. 每个章节一个目录
2. 章节专属的 README.md 文件
3. 可工作的项目状态
4. 可选的最终状态目录

## 示例

完整示例见 [examples](./examples) 目录：

- [TypeScript CLI](./examples/typescript)：基础 TypeScript 项目设置
- [Walkthroughgen](./examples/walkthroughgen)：自文档化示例

## 提示

1. 使用有意义的章节名称，因为它们会变成文件夹名称
2. 在步骤文本中包含上下文
3. 对会修改状态的命令使用 `incremental: true`
4. 利用 diff 突出重要变化
5. 使用 `skip` 数组从输出中排除 setup / cleanup 章节

## 贡献

欢迎贡献！英文原文引用了 `CONTRIBUTING.md` 获取详情；当前 package 目录下没有该文件。

## 授权

英文原文声明本项目采用 MIT License，并引用 `LICENSE` 文件；当前 package 目录下没有单独的 `LICENSE` 文件。
