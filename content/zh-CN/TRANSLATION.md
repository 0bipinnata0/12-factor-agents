# 中文翻译维护说明

本说明记录 `12-factor-agents` 中文译本的范围、风格和维护约定。

## 翻译范围

第一阶段翻译：

- `README.md`
- `content/brief-history-of-software.md`
- `content/factor-01-natural-language-to-tool-calls.md` 到 `content/factor-12-stateless-reducer.md`
- `content/appendix-13-pre-fetch.md`

第二阶段翻译：

- `workshops/2025-05/walkthrough.md` 的并行译本 `workshops/2025-05/walkthrough.zh-CN.md`

第三阶段翻译：

- `workshops/2025-05/sections/**/README.md` 的并行译本 `README.zh-CN.md`

暂不翻译：

- `workshops/**/walkthrough.yaml`
- `workshops/2025-05-17/**`
- `workshops/2025-07-16/**`
- `packages/**`
- `content/factor-1-*.md` 到 `content/factor-9-*.md` 这些旧文件名跳转页
- `img/**` 图片本体

## 文件结构

中文译本使用并行文件，不覆盖英文原文：

- `README.zh-CN.md`
- `content/zh-CN/*.md`
- `workshops/**/walkthrough.zh-CN.md`

中文文件名保持英文 slug，便于和英文原文一一对应。

## 链接策略

中文内部链接使用相对路径指向中文文件。外部链接保持原样。图片链接保持原图，仅翻译 alt 文本和上下文说明。

## 翻译风格

译文采用自然的技术中文，不做全文中英对照。关键术语首次出现时可保留英文，例如“上下文窗口（context window）”。术语以 [`glossary.md`](./glossary.md) 为准。

代码块中的变量名、函数名、包名、JSON key、API schema、命令、文件路径不翻译。解释性注释和自然语言示例可以翻译，但不得改变代码语义。

## 署名与授权

中文译文由 `0bipinnata0` 翻译 / 改编。原项目声明：内容与图片遵循 [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)，代码遵循 [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0)。

翻译基准提交：

- 第一阶段核心内容：`d20c728`
- 第二阶段 TypeScript workshop walkthrough：`b80256d`
- 第三阶段 TypeScript workshop section READMEs：`58b8f09`
