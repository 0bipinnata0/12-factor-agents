[← 返回 README](../../README.zh-CN.md)

### 10. 小而专注的 Agent

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-10-small-focused-agents.md`，基准提交：`d20c728`

不要构建试图包办一切的单体 Agent，而是构建小而专注、能把一件事做好的 Agent。Agent 只是更大系统中的一个构建块，而这个系统大部分仍然是确定性的。

![小而专注的 Agent](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a0-small-focused-agents.png)

这里的关键洞见和 LLM 的限制有关：任务越大、越复杂，需要的步骤就越多，也就意味着更长的上下文窗口。随着上下文增长，LLM 更容易迷路或失去焦点。把 Agent 限定在具体领域中，控制在 3-10 步，也许最多 20 步，可以让上下文窗口保持可控，并让 LLM 表现保持较高水平。

> #### 随着上下文增长，LLM 更容易迷路或失去焦点

小而专注的 Agent 的好处：

1. **可管理的上下文**：更小的上下文窗口意味着更好的 LLM 表现
2. **清晰职责**：每个 Agent 都有明确的范围和目的
3. **更高可靠性**：在复杂工作流中迷路的概率更低
4. **更容易测试**：更容易测试和验证具体功能
5. **更容易调试**：问题出现时，更容易定位和修复

### 如果 LLM 变得更聪明呢？

如果 LLM 聪明到足以处理 100+ 步工作流，我们还需要这个原则吗？

简短回答：需要。随着 Agent 和 LLM 改进，它们**可能**自然扩展到能处理更长的上下文窗口。这意味着它们能够处理更大 DAG 中更多部分。小而专注的方法能确保你今天就能得到结果，同时也让你可以随着 LLM 上下文窗口变得更可靠，慢慢扩大 Agent 范围。（如果你曾经重构过大型确定性代码库，你现在可能正在点头。）

[![Agent 范围增长动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)](https://github.com/user-attachments/assets/0cd3f52c-046e-4d5e-bab4-57657157c82f)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif">GIF 版本</a></summary>
![Agent 范围增长动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)
</details>

关键在于：有意识地控制 Agent 的大小和范围，并且只用能继续保持质量的方式增长。正如[构建 NotebookLM 的团队所说](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8&utm_campaign=post-share-selection&utm_medium=web)：

> 我感觉，在构建 AI 时，那些最有魔力的时刻，常常出现在我非常、非常、非常接近模型能力边界的时候。

无论那条边界在哪里，如果你能找到它，并稳定地把事情做对，你就能构建出有魔力的体验。这里可以构建很多护城河，但和往常一样，它们需要工程上的严谨。

[← 压缩错误](./factor-09-compact-errors.md) | [从任何地方触发 →](./factor-11-trigger-from-anywhere.md)
