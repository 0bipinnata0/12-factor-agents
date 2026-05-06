[← 返回 README](../../README.zh-CN.md)

## 更长版本：我们是怎么走到这里的

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/brief-history-of-software.md`，基准提交：`d20c728`

### 你不必听我的

无论你是刚接触 Agent，还是像我一样有点倔的老手，我都会试着说服你：先放下你对 AI Agents 的大部分既有想法，退后一步，从第一性原理重新思考它们。（如果你没关注几周前 OpenAI responses 的发布，这里提前剧透一下：把更多 Agent 逻辑塞到 API 后面并不是答案。）


## Agent 是软件，以及一段软件简史

我们来聊聊自己是怎么走到这一步的。

### 60 年前

我们会反复谈到有向图（Directed Graphs，DGs）以及它们的无环朋友 DAG。先指出一点：软件本身就是一张有向图。我们过去用流程图表示程序，不是没有原因的。

![软件 DAG](https://github.com/humanlayer/12-factor-agents/blob/main/img/010-software-dag.png)

### 20 年前

大约 20 年前，DAG 编排器开始流行起来。经典的有 [Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/)，还有一些前身，以及更新一些的工具，比如 [dagster](https://dagster.io/)、[inggest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/)。它们沿用了同样的图结构，同时带来了可观测性、模块化、重试、管理等能力。

![DAG 编排器](https://github.com/humanlayer/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### 10-15 年前

当 ML 模型开始好到足以实用时，我们开始看到 DAG 里被撒进一些 ML 模型。你可以想象这样的步骤：“把这一列里的文本摘要成新的一列”，或者“按严重程度或情绪分类支持工单”。

![带 ML 的 DAG](https://github.com/humanlayer/12-factor-agents/blob/main/img/020-dags-with-ml.png)

但归根结底，它大多还是熟悉的确定性软件。

### Agent 的承诺

我不是第一个[这样说的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但我开始学习 Agent 时最大的感受是：你终于可以把 DAG 扔掉了。软件工程师不用为每一步和每个边界情况写代码，而是给 Agent 一个目标和一组转移：

![Agent DAG](https://github.com/humanlayer/12-factor-agents/blob/main/img/025-agent-dag.png)

然后让 LLM 实时决策，自己找出路径。

![Agent DAG 路径](https://github.com/humanlayer/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

这里的承诺是：你可以写更少的软件，只需要把图的“边”交给 LLM，让它自己找“节点”。你可以从错误中恢复，写更少代码，而且可能会发现 LLM 找到一些新颖的问题解法。

### 作为循环的 Agent

换句话说，你会得到一个由 3 个步骤组成的循环：

1. LLM 判断工作流下一步，输出结构化 JSON（“工具调用”）
2. 确定性代码执行这个工具调用
3. 结果被追加到上下文窗口
4. 重复，直到下一步被判断为“done”

```python
initial_event = {"message": "..."}
context = [initial_event]
while True:
  next_step = await llm.determine_next_step(context)
  context.append(next_step)

  if (next_step.intent === "done"):
    return next_step.final_answer

  result = await execute_step(next_step)
  context.append(result)
```

初始上下文只是起始事件，也许是一条用户消息，也许是 cron 触发，也许是 webhook 等等。然后我们让 LLM 选择下一步（工具），或者判断任务已经完成。

下面是一个多步骤示例：

[![Agent 循环动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF 版本</a></summary>

![Agent 循环动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)

</details>

生成出的“物化”DAG 大概会长这样：

![Agent 循环 DAG](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-dag.png)

### “循环直到解决问题”模式的问题

这个模式最大的问题是：

- 当上下文窗口变得太长时，Agent 会迷路：它们会不断尝试同一个坏掉的方法
- 基本就这一点，但这一点已经足以让这个方法站不住

即使你没有手写过 Agent，也很可能在使用 agentic 编码工具时见过这种长上下文问题。过一阵子它们就会迷路，然后你不得不开一个新聊天。

我甚至想提出一个我经常听到、而且你可能也已经形成直觉的判断：

> ### **即使模型支持越来越长的上下文窗口，你也总能通过小而专注的 prompt 和上下文得到更好的结果**

我聊过的大多数构建者，在意识到超过 10-20 轮之后事情会变成 LLM 无法恢复的大混乱时，都会把“工具调用循环”这个想法放到一边。即使 Agent 90% 的时候能做对，这距离“好到能交给客户使用”也差得很远。你能想象一个 Web 应用 10% 的页面加载都会崩溃吗？

**更新 2025-06-09**：我很喜欢 [@swyx](https://x.com/swyx/status/1932125643384455237) 的这个说法：

<a href="https://x.com/swyx/status/1932125643384455237"><img width="593" alt="swyx 关于上下文的截图" src="https://github.com/user-attachments/assets/c7d94042-e4b9-4b87-87fd-55c7ff94bb3b" /></a>

### 真正有效的东西：微型 Agent

我在真实世界里经常看到的一种做法，是把 Agent 模式拿出来，撒进一个更大、更确定性的 DAG 里。

![微型 Agent DAG](https://github.com/humanlayer/12-factor-agents/blob/main/img/028-micro-agent-dag.png)

你可能会问：“既然如此，为什么还要用 Agent？”我们很快会讲到。基本上，让语言模型管理边界清晰的一小组任务，可以很容易地把实时的人类反馈纳入进来，把它翻译成工作流步骤，同时避免陷入上下文错误循环。（[factor 1](./factor-01-natural-language-to-tool-calls.md)、[factor 3](./factor-03-own-your-context-window.md)、[factor 7](./factor-07-contact-humans-with-tools.md)）

> #### 让语言模型管理边界清晰的一小组任务，可以很容易地把实时的人类反馈纳入进来，而不会陷入上下文错误循环

### 一个真实的微型 Agent

下面这个例子展示了确定性代码如何运行一个微型 Agent，让它负责部署流程中人在回路中的步骤。

![deploybot 高层结构](https://github.com/humanlayer/12-factor-agents/blob/main/img/029-deploybot-high-level.png)

* **Human** 把 PR 合并到 GitHub main 分支
* **Deterministic Code** 部署到 staging 环境
* **Deterministic Code** 对 staging 跑端到端（e2e）测试
* **Deterministic Code** 把任务交给负责 prod 部署的 Agent，初始上下文是：“deploy SHA 4af9ec0 to production”
* **Agent** 调用 `deploy_frontend_to_prod(4af9ec0)`
* **Deterministic code** 请求人类批准这个动作
* **Human** 拒绝该动作并给出反馈：“can you deploy the backend first?”
* **Agent** 调用 `deploy_backend_to_prod(4af9ec0)`
* **Deterministic code** 请求人类批准这个动作
* **Human** 批准该动作
* **Deterministic code** 执行后端部署
* **Agent** 调用 `deploy_frontend_to_prod(4af9ec0)`
* **Deterministic code** 请求人类批准这个动作
* **Human** 批准该动作
* **Deterministic code** 执行前端部署
* **Agent** 判断任务已经成功完成，到此结束
* **Deterministic code** 对 production 跑端到端测试
* **Deterministic code** 完成任务，或者把任务交给 rollback agent 去审查失败并可能回滚

[![deploybot 动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif)](https://github.com/user-attachments/assets/deb356e9-0198-45c2-9767-231cb569ae13)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif">GIF 版本</a></summary>

![deploybot 动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif)

</details>

这个例子基于一个真实的 [OSS Agent，我们已经用它在 Humanlayer 管理部署](https://github.com/got-agents/agents/tree/main/deploybot-ts)。下面是我上周和它的一段真实对话：

![deploybot 对话](https://github.com/humanlayer/12-factor-agents/blob/main/img/035-deploybot-conversation.png)


我们没有给这个 Agent 一大堆工具或任务。LLM 的主要价值在于解析人类的纯文本反馈，并提出更新后的行动方案。我们尽可能隔离任务和上下文，让 LLM 专注在一个小型的、5-10 步的工作流上。

这里还有一个[更经典的支持 / chatbot demo](https://x.com/chainlit_io/status/1858613325921480922)。

### 所以 Agent 到底是什么？

- **prompt**：告诉 LLM 应该如何行为，以及它有哪些“工具”可用。prompt 的输出是一个 JSON 对象，描述工作流中的下一步（“工具调用”或“函数调用”）。（[factor 2](./factor-02-own-your-prompts.md)）
- **switch statement**：根据 LLM 返回的 JSON，决定要怎么处理它。（[factor 8](./factor-08-own-your-control-flow.md) 的一部分）
- **accumulated context**：存储已经发生的步骤及其结果。（[factor 3](./factor-03-own-your-context-window.md)）
- **for loop**：在 LLM 输出某种“Terminal”工具调用（或纯文本响应）之前，把 switch statement 的结果添加到上下文窗口，然后让 LLM 选择下一步。（[factor 8](./factor-08-own-your-control-flow.md)）

![4 个组件](https://github.com/humanlayer/12-factor-agents/blob/main/img/040-4-components.png)

在 “deploybot” 例子中，掌控控制流和上下文累积给我们带来几个好处：

- 在我们的 **switch statement** 和 **for loop** 中，可以劫持控制流，暂停等待人类输入，或者等待长时间运行的任务完成
- 我们可以非常轻松地序列化 **context** 窗口，用于暂停和恢复
- 在我们的 **prompt** 中，可以尽可能优化传给 LLM 的指令和“到目前为止发生了什么”


[第二部分](../../README.zh-CN.md#12-个因素再次列出)会**把这些模式形式化**，这样你就可以把它们应用到任何软件项目中，加入让人印象深刻的 AI 功能，而不需要全面押注传统意义上的“AI Agent”实现或定义。


[Factor 1：从自然语言到工具调用 →](./factor-01-natural-language-to-tool-calls.md)
