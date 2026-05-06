# 12-Factor Agents：构建可靠 LLM 应用的原则

语言： [English](./README.md) | [简体中文](./README.zh-CN.md)

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`README.md`，基准提交：`d20c728`

<div align="center">
<a href="https://www.apache.org/licenses/LICENSE-2.0">
        <img src="https://img.shields.io/badge/Code-Apache%202.0-blue.svg" alt="代码许可证：Apache 2.0"></a>
<a href="https://creativecommons.org/licenses/by-sa/4.0/">
        <img src="https://img.shields.io/badge/Content-CC%20BY--SA%204.0-lightgrey.svg" alt="内容许可证：CC BY-SA 4.0"></a>
<a href="https://humanlayer.dev/discord">
    <img src="https://img.shields.io/badge/chat-discord-5865F2" alt="Discord 服务器"></a>
<a href="https://www.youtube.com/watch?v=8kMaTybvDUw">
    <img src="https://img.shields.io/badge/aidotengineer-conf_talk_(17m)-white" alt="YouTube 深入讲解"></a>
<a href="https://www.youtube.com/watch?v=yxJDyQ8v6P0">
    <img src="https://img.shields.io/badge/youtube-deep_dive-crimson" alt="YouTube 深入讲解"></a>

</div>

<p></p>

*继承 [12 Factor Apps](https://12factor.net/) 的精神。* *本项目源码公开在 https://github.com/humanlayer/12-factor-agents，欢迎反馈和贡献。我们一起来把这件事弄清楚。*

> [!TIP]
> 错过了 AI Engineer World's Fair？[这里可以看演讲回放](https://www.youtube.com/watch?v=8kMaTybvDUw)
>
> 在找 Context Engineering？[直接跳到 Factor 3](./content/zh-CN/factor-03-own-your-context-window.md)
>
> 想给 `npx/uvx create-12-factor-agent` 做贡献？可以看[这个讨论帖](https://github.com/humanlayer/12-factor-agents/discussions/61)


<img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=2acad99a-c2d9-48df-86f5-9ca8061b7bf9" />

<a href="#可视化导航"><img width="907" alt="12-Factor Agents 截图" src="https://github.com/user-attachments/assets/23286ad8-7bef-4902-b371-88ff6a22e998" /></a>


你好，我是 Dex。我已经在 [AI agents](https://theouterloop.substack.com) 上[折腾](https://youtu.be/8bIHcttkOTE)了[一段时间](https://humanlayer.dev)。


**市面上几乎所有 Agent 框架我都试过**，从即插即用的 crew/langchains，到所谓“极简”的 smolagents，再到“生产级”的 langraph、griptape 等等。

**我也和很多非常强的创始人聊过**，包括 YC 内外那些正在用 AI 做出很厉害产品的人。大多数人都在自己搭栈。我没有看到很多框架真正跑在面向客户的生产 Agent 里。

**让我意外的是**，很多自称“AI Agent”的产品其实并没有那么 agentic。它们大多还是确定性代码，只是在恰到好处的地方撒上一些 LLM 步骤，让体验变得神奇。

至少好的 Agent 并不是“给它一个 prompt、一袋工具，然后循环直到完成目标”的模式。它们更多是由普通软件组成的。

所以我想回答这个问题：

> ### **我们可以用哪些原则，构建真正好到可以交到生产客户手里的 LLM 驱动软件？**

欢迎来到 12-factor agents。就像 Daley 之后每一任芝加哥市长都会在城市主要机场反复贴出的标语一样：很高兴你来到这里。

*特别感谢 [@iantbutler01](https://github.com/iantbutler01)、[@tnm](https://github.com/tnm)、[@hellovai](https://www.github.com/hellovai)、[@stantonk](https://www.github.com/stantonk)、[@balanceiskey](https://www.github.com/balanceiskey)、[@AdjectiveAllison](https://www.github.com/AdjectiveAllison)、[@pfbyjy](https://www.github.com/pfbyjy)、[@a-churchill](https://www.github.com/a-churchill) 以及 SF MLOps 社区对本指南的早期反馈。*

## 简短版：12 个因素

即使 LLM [继续指数级变强](./content/zh-CN/factor-10-small-focused-agents.md#如果-llm-变得更聪明呢)，仍然会有一些核心工程技术，让 LLM 驱动的软件更可靠、更可扩展，也更容易维护。

- [我们是怎么走到这里的：软件简史](./content/zh-CN/brief-history-of-software.md)
- [Factor 1：从自然语言到工具调用](./content/zh-CN/factor-01-natural-language-to-tool-calls.md)
- [Factor 2：掌控你的提示词](./content/zh-CN/factor-02-own-your-prompts.md)
- [Factor 3：掌控你的上下文窗口](./content/zh-CN/factor-03-own-your-context-window.md)
- [Factor 4：工具本质上只是结构化输出](./content/zh-CN/factor-04-tools-are-structured-outputs.md)
- [Factor 5：统一执行状态和业务状态](./content/zh-CN/factor-05-unify-execution-state.md)
- [Factor 6：用简单 API 启动 / 暂停 / 恢复](./content/zh-CN/factor-06-launch-pause-resume.md)
- [Factor 7：用工具调用联系人类](./content/zh-CN/factor-07-contact-humans-with-tools.md)
- [Factor 8：掌控你的控制流](./content/zh-CN/factor-08-own-your-control-flow.md)
- [Factor 9：把错误压缩进上下文窗口](./content/zh-CN/factor-09-compact-errors.md)
- [Factor 10：小而专注的 Agent](./content/zh-CN/factor-10-small-focused-agents.md)
- [Factor 11：从任何地方触发，在用户所在之处服务用户](./content/zh-CN/factor-11-trigger-from-anywhere.md)
- [Factor 12：把你的 Agent 做成无状态 Reducer](./content/zh-CN/factor-12-stateless-reducer.md)

### 可视化导航

|    |    |    |
|----|----|-----|
|[![factor 1](https://github.com/humanlayer/12-factor-agents/blob/main/img/110-natural-language-tool-calls.png)](./content/zh-CN/factor-01-natural-language-to-tool-calls.md) | [![factor 2](https://github.com/humanlayer/12-factor-agents/blob/main/img/120-own-your-prompts.png)](./content/zh-CN/factor-02-own-your-prompts.md) | [![factor 3](https://github.com/humanlayer/12-factor-agents/blob/main/img/130-own-your-context-building.png)](./content/zh-CN/factor-03-own-your-context-window.md) |
|[![factor 4](https://github.com/humanlayer/12-factor-agents/blob/main/img/140-tools-are-just-structured-outputs.png)](./content/zh-CN/factor-04-tools-are-structured-outputs.md) | [![factor 5](https://github.com/humanlayer/12-factor-agents/blob/main/img/150-unify-state.png)](./content/zh-CN/factor-05-unify-execution-state.md) | [![factor 6](https://github.com/humanlayer/12-factor-agents/blob/main/img/160-pause-resume-with-simple-apis.png)](./content/zh-CN/factor-06-launch-pause-resume.md) |
| [![factor 7](https://github.com/humanlayer/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)](./content/zh-CN/factor-07-contact-humans-with-tools.md) | [![factor 8](https://github.com/humanlayer/12-factor-agents/blob/main/img/180-control-flow.png)](./content/zh-CN/factor-08-own-your-control-flow.md) | [![factor 9](https://github.com/humanlayer/12-factor-agents/blob/main/img/190-factor-9-errors-static.png)](./content/zh-CN/factor-09-compact-errors.md) |
| [![factor 10](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a0-small-focused-agents.png)](./content/zh-CN/factor-10-small-focused-agents.md) | [![factor 11](https://github.com/humanlayer/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)](./content/zh-CN/factor-11-trigger-from-anywhere.md) | [![factor 12](https://github.com/humanlayer/12-factor-agents/blob/main/img/1c0-stateless-reducer.png)](./content/zh-CN/factor-12-stateless-reducer.md) |

## 我们是怎么走到这里的

如果你想深入了解我的 Agent 之旅，以及是什么把我们带到这里，可以看[软件简史](./content/zh-CN/brief-history-of-software.md)。这里先快速总结一下：

### Agent 的承诺

我们会反复谈到有向图（Directed Graphs，DGs）以及它们的无环朋友 DAG。先指出一点：软件本身就是一张有向图。我们过去用流程图表示程序，不是没有原因的。

![010-software-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/010-software-dag.png)

### 从代码到 DAG

大约 20 年前，DAG 编排器开始流行起来。经典的有 [Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/)，还有一些前身，以及更新一些的工具，比如 [dagster](https://dagster.io/)、[inggest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/)。它们沿用了同样的图结构，同时带来了可观测性、模块化、重试、管理等能力。

![015-dag-orchestrators](https://github.com/humanlayer/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### Agent 的承诺

我不是第一个[这样说的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但我开始学习 Agent 时最大的感受是：你终于可以把 DAG 扔掉了。软件工程师不用为每一步和每个边界情况写代码，而是给 Agent 一个目标和一组转移：

![025-agent-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/025-agent-dag.png)

然后让 LLM 实时决策，自己找出路径。

![026-agent-dag-lines](https://github.com/humanlayer/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

这里的承诺是：你可以写更少的软件，只需要把图的“边”交给 LLM，让它自己找“节点”。你可以从错误中恢复，写更少代码，而且可能会发现 LLM 找到一些新颖的问题解法。


### 作为循环的 Agent

后面我们会看到，这种方式其实并没有那么奏效。

再往下看一层，Agent 通常有一个由 3 个步骤组成的循环：

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

[![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF 版本</a></summary>

![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)

</details>

## 为什么需要 12-factor agents？

归根结底，这种方式并没有我们希望的那么好用。

在构建 HumanLayer 的过程中，我和至少 100 位 SaaS 构建者聊过。他们大多是技术型创始人，想把现有产品变得更 agentic。他们的旅程通常是这样：

1. 决定要构建一个 Agent
2. 做产品设计、UX 映射、确定要解决的问题
3. 想快速推进，于是拿起某个框架开始做
4. 做到 70-80% 的质量线
5. 意识到对大多数面向客户的功能来说，80% 远远不够
6. 意识到要超过 80%，就必须反向工程这个框架、提示词、流程等
7. 从头开始

<details>
<summary>一些免责声明</summary>

**免责声明**：我不确定这段该放在哪里，但这里应该也可以：**这绝不是在贬低市面上的诸多框架，或那些参与构建框架的聪明人**。它们成就了很多厉害的东西，也加速了 AI 生态。

我希望这篇文章的一个结果，是 Agent 框架构建者能从我和其他人的经历中学到东西，把框架做得更好。

尤其是服务那些既想快速推进、又需要深度控制的构建者。

**免责声明 2**：我不会讨论 MCP。你应该能看出来它适合放在哪里。

**免责声明 3**：我主要使用 TypeScript，原因[在这里](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e?utm_source=share&utm_medium=member_desktop&rcm=ACoAAA4oHTkByAiD-wZjnGsMBUL_JT6nyyhOh30)，但这些东西同样适用于 Python 或你偏好的任何语言。


回到正题。

</details>

### 优秀 LLM 应用的设计模式

在研究了数百个 AI 库、和几十位创始人一起工作之后，我的直觉是：

1. 有一些核心东西会让 Agent 变好
2. 全面押注某个框架，并构建某种近似绿地重写的系统，可能适得其反
3. 有一些核心原则会让 Agent 变好，而你引入框架时通常也会得到其中大多数甚至全部
4. 但是，我见过构建者把高质量 AI 软件交到客户手里最快的方式，是从 Agent 构建中拿出小而模块化的概念，把它们融入现有产品
5. 这些来自 Agent 的模块化概念可以由大多数熟练的软件工程师定义和应用，即使他们没有 AI 背景

> #### 我见过构建者把好用的 AI 软件交到客户手里最快的方式，是从 Agent 构建中拿出小而模块化的概念，把它们融入现有产品


## 12 个因素（再次列出）


- [我们是怎么走到这里的：软件简史](./content/zh-CN/brief-history-of-software.md)
- [Factor 1：从自然语言到工具调用](./content/zh-CN/factor-01-natural-language-to-tool-calls.md)
- [Factor 2：掌控你的提示词](./content/zh-CN/factor-02-own-your-prompts.md)
- [Factor 3：掌控你的上下文窗口](./content/zh-CN/factor-03-own-your-context-window.md)
- [Factor 4：工具本质上只是结构化输出](./content/zh-CN/factor-04-tools-are-structured-outputs.md)
- [Factor 5：统一执行状态和业务状态](./content/zh-CN/factor-05-unify-execution-state.md)
- [Factor 6：用简单 API 启动 / 暂停 / 恢复](./content/zh-CN/factor-06-launch-pause-resume.md)
- [Factor 7：用工具调用联系人类](./content/zh-CN/factor-07-contact-humans-with-tools.md)
- [Factor 8：掌控你的控制流](./content/zh-CN/factor-08-own-your-control-flow.md)
- [Factor 9：把错误压缩进上下文窗口](./content/zh-CN/factor-09-compact-errors.md)
- [Factor 10：小而专注的 Agent](./content/zh-CN/factor-10-small-focused-agents.md)
- [Factor 11：从任何地方触发，在用户所在之处服务用户](./content/zh-CN/factor-11-trigger-from-anywhere.md)
- [Factor 12：把你的 Agent 做成无状态 Reducer](./content/zh-CN/factor-12-stateless-reducer.md)

## 荣誉提名 / 其他建议

- [Factor 13：预取所有可能需要的上下文](./content/zh-CN/appendix-13-pre-fetch.md)

## 相关资源

- 在[这里](https://github.com/humanlayer/12-factor-agents)为本指南贡献内容
- 2025 年 3 月，我在 Tool Use podcast 的一期节目里[聊过不少这些内容](https://youtu.be/8bIHcttkOTE)
- 我会在 [The Outer Loop](https://theouterloop.substack.com) 写一些相关内容
- 我和 [@hellovai](https://github.com/hellovai) 一起做关于[最大化 LLM 性能](https://github.com/hellovai/ai-that-works/tree/main)的 webinar
- 我们用这套方法在 [got-agents/agents](https://github.com/got-agents/agents) 下构建 OSS agents
- 我们无视了自己所有建议，做了一个[在 Kubernetes 中运行分布式 Agent 的框架](https://github.com/humanlayer/kubechain)
- 本指南中的其他链接：
  - [12 Factor Apps](https://12factor.net)
  - [Building Effective Agents (Anthropic)](https://www.anthropic.com/engineering/building-effective-agents#agents)
  - [Prompts are Functions](https://thedataexchange.media/baml-revolution-in-ai-engineering/ )
  - [Library patterns: Why frameworks are evil](https://tomasp.net/blog/2015/library-frameworks/)
  - [The Wrong Abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)
  - [Mailcrew Agent](https://github.com/dexhorthy/mailcrew)
  - [Mailcrew Demo Video](https://www.youtube.com/watch?v=f_cKnoPC_Oo)
  - [Chainlit Demo](https://x.com/chainlit_io/status/1858613325921480922)
  - [TypeScript for LLMs](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e)
  - [Schema Aligned Parsing](https://www.boundaryml.com/blog/schema-aligned-parsing)
  - [Function Calling vs Structured Outputs vs JSON Mode](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode)
  - [BAML on GitHub](https://github.com/boundaryml/baml)
  - [OpenAI JSON vs Function Calling](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)
  - [Outer Loop Agents](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)
  - [Airflow](https://airflow.apache.org/)
  - [Prefect](https://www.prefect.io/)
  - [Dagster](https://dagster.io/)
  - [Inngest](https://www.inngest.com/)
  - [Windmill](https://www.windmill.dev/)
  - [The AI Agent Index (MIT)](https://aiagentindex.mit.edu/)
  - [NotebookLM on Finding Model Capability Boundaries](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8)

## 贡献者

感谢所有为 12-factor agents 做出贡献的人！

[<img src="https://avatars.githubusercontent.com/u/3730605?v=4&s=80" width="80px" alt="dexhorthy" />](https://github.com/dexhorthy) [<img src="https://avatars.githubusercontent.com/u/50557586?v=4&s=80" width="80px" alt="Sypherd" />](https://github.com/Sypherd) [<img src="https://avatars.githubusercontent.com/u/66259401?v=4&s=80" width="80px" alt="tofaramususa" />](https://github.com/tofaramususa) [<img src="https://avatars.githubusercontent.com/u/18105223?v=4&s=80" width="80px" alt="a-churchill" />](https://github.com/a-churchill) [<img src="https://avatars.githubusercontent.com/u/4084885?v=4&s=80" width="80px" alt="Elijas" />](https://github.com/Elijas) [<img src="https://avatars.githubusercontent.com/u/39267118?v=4&s=80" width="80px" alt="hugolmn" />](https://github.com/hugolmn) [<img src="https://avatars.githubusercontent.com/u/1882972?v=4&s=80" width="80px" alt="jeremypeters" />](https://github.com/jeremypeters)

[<img src="https://avatars.githubusercontent.com/u/380402?v=4&s=80" width="80px" alt="kndl" />](https://github.com/kndl) [<img src="https://avatars.githubusercontent.com/u/16674643?v=4&s=80" width="80px" alt="maciejkos" />](https://github.com/maciejkos) [<img src="https://avatars.githubusercontent.com/u/85041180?v=4&s=80" width="80px" alt="pfbyjy" />](https://github.com/pfbyjy) [<img src="https://avatars.githubusercontent.com/u/36044389?v=4&s=80" width="80px" alt="0xRaduan" />](https://github.com/0xRaduan) [<img src="https://avatars.githubusercontent.com/u/7169731?v=4&s=80" width="80px" alt="zyuanlim" />](https://github.com/zyuanlim) [<img src="https://avatars.githubusercontent.com/u/15862501?v=4&s=80" width="80px" alt="lombardo-chcg" />](https://github.com/lombardo-chcg) [<img src="https://avatars.githubusercontent.com/u/160066852?v=4&s=80" width="80px" alt="sahanatvessel" />](https://github.com/sahanatvessel)

## 授权

所有内容和图片均采用 <a href="https://creativecommons.org/licenses/by-sa/4.0/">CC BY-SA 4.0 License</a>

代码采用 <a href="https://www.apache.org/licenses/LICENSE-2.0">Apache 2.0 License</a>
