[← 返回 README](../../README.zh-CN.md)

### 5. 统一执行状态和业务状态

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-05-unify-execution-state.md`，基准提交：`d20c728`

即使在 AI 世界之外，很多基础设施系统也会尝试把“执行状态”和“业务状态”分开。对 AI 应用来说，这可能会涉及复杂抽象，用来追踪当前步骤、下一步、等待状态、重试次数等等。这种分离会带来复杂度，它也许值得，但对你的场景来说也可能过度。

一如既往，什么适合你的应用，需要你自己判断。但不要以为你*必须*把它们分开管理。

更明确地说：

- **执行状态**：当前步骤、下一步、等待状态、重试次数等等。
- **业务状态**：Agent 工作流到目前为止发生了什么，例如 OpenAI messages 列表、工具调用和结果列表等等。

如果可以，简化它：尽可能统一二者。

[![统一状态动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif)](https://github.com/user-attachments/assets/e5a851db-f58f-43d8-8b0c-1926c99fc68d)


<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif">GIF 版本</a></summary>

![统一状态动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif)

</details>

现实中，你可以设计应用，让它能够从上下文窗口推导出所有执行状态。在许多情况下，执行状态（当前步骤、等待状态等）只是“到目前为止发生了什么”的元数据。

你可能确实有一些东西不能放进上下文窗口，比如 session id、password context 等等，但你的目标应该是尽量减少这些东西。通过拥抱 [factor 3](./factor-03-own-your-context-window.md)，你可以控制真正进入 LLM 的内容。

这种方式有几个好处：

1. **简单性**：所有状态只有一个事实来源
2. **序列化**：thread 可以非常容易地序列化 / 反序列化
3. **调试**：完整历史在一个地方可见
4. **灵活性**：只要添加新的 event type，就能很容易地添加新状态
5. **恢复**：只要加载 thread，就能从任意点恢复
6. **分叉**：可以复制 thread 的某个子集到新的 context / state ID，从任意点 fork thread
7. **人类接口和可观测性**：把 thread 转换成人类可读的 Markdown 或丰富的 Web app UI 很简单

[← 工具本质上只是结构化输出](./factor-04-tools-are-structured-outputs.md) | [启动 / 暂停 / 恢复 →](./factor-06-launch-pause-resume.md)
