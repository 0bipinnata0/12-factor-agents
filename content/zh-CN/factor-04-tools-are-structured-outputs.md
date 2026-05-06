[← 返回 README](../../README.zh-CN.md)

### 4. 工具本质上只是结构化输出

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-04-tools-are-structured-outputs.md`，基准提交：`d20c728`

工具不需要很复杂。从核心上看，它们只是 LLM 输出的结构化数据，然后触发确定性代码。

![工具本质上只是结构化输出](https://github.com/humanlayer/12-factor-agents/blob/main/img/140-tools-are-just-structured-outputs.png)

例如，假设你有两个工具：`CreateIssue` 和 `SearchIssues`。让 LLM “使用多个工具中的一个”，本质上只是让它输出一段 JSON，然后我们把它解析成代表这些工具的对象。

```python

class Issue:
  title: str
  description: str
  team_id: str
  assignee_id: str

class CreateIssue:
  intent: "create_issue"
  issue: Issue

class SearchIssues:
  intent: "search_issues"
  query: str
  what_youre_looking_for: str
```

这个模式很简单：

1. LLM 输出结构化 JSON
3. 确定性代码执行对应动作，比如调用外部 API
4. 结果被捕获，并被喂回上下文

这样会在 LLM 的决策和应用的动作之间建立清晰分离。LLM 决定做什么，但你的代码控制怎么做。LLM “调用了一个工具”，并不意味着你每次都必须以同样方式执行某个一一对应的函数。

如果你还记得前面的 switch statement：

```python
if nextStep.intent == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return # 或者执行任何你想要的逻辑，见下文
elif nextStep.intent == 'wait_for_a_while':
    # 做点 monadic 的事情，我也说不好
else: #... 模型没有调用我们认识的工具
    # 做点别的
```

**注意**：关于 “plain prompting” vs. “tool calling” vs. “JSON mode” 的收益，以及各自性能权衡，已经有很多讨论。我们之后会链接更多资源，这里先不展开。可以看 [Prompting vs JSON Mode vs Function Calling vs Constrained Generation vs SAP](https://www.boundaryml.com/blog/schema-aligned-parsing)、[When should I use function calling, structured outputs, or JSON mode?](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode#:~:text=We%20don%27t%20recommend%20using%20JSON,always%20use%20Structured%20Outputs%20instead) 和 [OpenAI JSON vs Function Calling](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)。

“下一步”不一定只是“运行一个纯函数并返回结果”这么原子。当你把“工具调用”理解成模型输出的一段 JSON，用来描述确定性代码应该做什么时，会解锁很多灵活性。把它和 [factor 8：掌控你的控制流](./factor-08-own-your-control-flow.md) 放在一起看。

[← 掌控你的上下文窗口](./factor-03-own-your-context-window.md) | [统一执行状态 →](./factor-05-unify-execution-state.md)
