[← 返回 README](../../README.zh-CN.md)

### 2. 掌控你的提示词

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-02-own-your-prompts.md`，基准提交：`d20c728`

不要把你的提示词工程外包给框架。

![掌控你的提示词](https://github.com/humanlayer/12-factor-agents/blob/main/img/120-own-your-prompts.png)

顺便说一句，[这远不是什么新建议](https://hamel.dev/blog/posts/prompt/)：

![图片](https://github.com/user-attachments/assets/575bab37-0f96-49fb-9ce3-9a883cdd420b)

有些框架会提供一种类似这样的“黑盒”方式：

```python
agent = Agent(
  role="...",
  goal="...",
  personality="...",
  tools=[tool1, tool2, tool3]
)

task = Task(
  instructions="...",
  expected_output=OutputModel
)

result = agent.run(task)
```

这对于一开始直接借用一些顶级提示词工程成果很有帮助，但如果你想调优，或者反向工程出到底哪些 token 被送进了模型，它往往会变得很困难。

相反，你应该掌控自己的提示词，并把它们当作一等代码来对待：

```rust
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
  prompt #"
    {{ _.role("system") }}

    You are a helpful assistant that manages deployments for frontend and backend systems.
    You work diligently to ensure safe and successful deployments by following best practices
    and proper deployment procedures.

    Before deploying any system, you should check:
    - The deployment environment (staging vs production)
    - The correct tag/version to deploy
    - The current system status

    You can use tools like deploy_backend, deploy_frontend, and check_deployment_status
    to manage deployments. For sensitive deployments, use request_approval to get
    human verification.

    Always think about what to do first, like:
    - Check current deployment status
    - Verify the deployment tag exists
    - Request approval if needed
    - Deploy to staging before production
    - Monitor deployment progress

    {{ _.role("user") }}

    {{ thread }}

    What should the next step be?
  "#
}
```

上面的例子使用 [BAML](https://github.com/boundaryml/baml) 来生成 prompt，但你也可以用任何你想用的提示词工程工具，甚至手动用模板拼出来。

如果这个签名看起来有点奇怪，我们会在 [factor 4：工具本质上只是结构化输出](./factor-04-tools-are-structured-outputs.md) 里讲到。

```typescript
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
```

掌控提示词的关键好处：

1. **完全控制**：写下你的 Agent 真正需要的指令，没有黑盒抽象
2. **测试和 Evals**：像测试其他代码一样，为你的提示词构建测试和 evals
3. **快速迭代**：根据真实世界表现快速修改提示词
4. **透明度**：准确知道你的 Agent 正在使用哪些指令
5. **角色 Hack**：利用支持非标准 user/assistant 角色用法的 API。例如已经废弃的非 chat 版本 OpenAI “completions” API。这也包括一些所谓的“model gaslighting”技巧

记住：提示词是你的应用逻辑和 LLM 之间最主要的接口。

完全控制提示词，会给你构建生产级 Agent 所需的灵活性和提示词控制能力。

我不知道最佳提示词是什么，但我知道你会想要能尝试一切的灵活性。

[← 从自然语言到工具调用](./factor-01-natural-language-to-tool-calls.md) | [掌控你的上下文窗口 →](./factor-03-own-your-context-window.md)
