[← 返回 README](../../README.zh-CN.md)

### 7. 用工具调用联系人类

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-07-contact-humans-with-tools.md`，基准提交：`d20c728`

默认情况下，LLM API 依赖一个基础但高风险的 token 选择：我们要返回纯文本内容，还是返回结构化数据？

![用工具调用联系人类](https://github.com/humanlayer/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)

你把很重的权重压在了第一个 token 的选择上。比如在 `the weather in tokyo` 这个场景中，第一个 token 是：

> "the"

但在 `fetch_weather` 场景中，它会是某种表示 JSON 对象开始的特殊 token。

> |JSON>

让 LLM *永远*输出 JSON，然后用 `request_human_input` 或 `done_for_now` 这样的自然语言 token 声明自己的 intent，可能会得到更好的结果，而不是只使用 `check_weather_in_city` 这类“正规的”工具。

同样，这也许不会带来任何性能提升。但你应该实验，并确保自己有自由去尝试奇怪的东西，以拿到最好的结果。

```python

class Options:
  urgency: Literal["low", "medium", "high"]
  format: Literal["free_text", "yes_no", "multiple_choice"]
  choices: List[str]

# 人类交互的工具定义
class RequestHumanInput:
  intent: "request_human_input"
  question: str
  context: str
  options: Options

# Agent 循环中的示例用法
if nextStep.intent == 'request_human_input':
  thread.events.append({
    type: 'human_input_requested',
    data: nextStep
  })
  thread_id = await save_state(thread)
  await notify_human(nextStep, thread_id)
  return # 跳出循环，等待带着 thread ID 的响应回来
else:
  # ... 其他分支
```

稍后，你可能会从处理 Slack、email、SMS 或其他事件的系统收到一个 webhook。

```python

@app.post('/webhook')
def webhook(req: Request):
  thread_id = req.body.threadId
  thread = await load_state(thread_id)
  thread.events.push({
    type: 'response_from_human',
    data: req.body
  })
  # ... 为了简洁而简化，实际中你大概率不想在这里阻塞 web worker
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread.events.append(next_step)
  result = await handle_next_step(thread, next_step)
  # todo - 继续循环、跳出，或者做任何你想做的事

  return {"status": "ok"}
```

上面的代码包含了来自 [factor 5：统一执行状态和业务状态](./factor-05-unify-execution-state.md)、[factor 8：掌控你的控制流](./factor-08-own-your-control-flow.md)、[factor 3：掌控你的上下文窗口](./factor-03-own-your-context-window.md)、[factor 4：工具本质上只是结构化输出](./factor-04-tools-are-structured-outputs.md) 以及其他 factor 的模式。

如果我们使用 [factor 3：掌控你的上下文窗口](./factor-03-own-your-context-window.md) 中那种偏 XML 的格式，几轮之后的上下文窗口可能长这样：

```xml

(snipped for brevity)

<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy backend v1.2.3 to production?
    Thread: []
</slack_message>

<request_human_input>
    intent: "request_human_input"
    question: "Would you like to proceed with deploying v1.2.3 to production?"
    context: "This is a production deployment that will affect live users."
    options: {
        urgency: "high"
        format: "yes_no"
    }
</request_human_input>

<human_response>
    response: "yes please proceed"
    approved: true
    timestamp: "2024-03-15T10:30:00Z"
    user: "alex@company.com"
</human_response>

<deploy_backend>
    intent: "deploy_backend"
    tag: "v1.2.3"
    environment: "production"
</deploy_backend>

<deploy_backend_result>
    status: "success"
    message: "Deployment v1.2.3 to production completed successfully."
    timestamp: "2024-03-15T10:30:00Z"
</deploy_backend_result>
```


好处：

1. **清晰指令**：用于不同类型人类联系的工具，能让 LLM 给出更具体的表达
2. **内层循环 vs 外层循环**：让 Agent 工作流能运行在传统 ChatGPT 风格界面之外。控制流和上下文初始化可以是 `Agent->Human`，而不是 `Human->Agent`，例如由 cron 或事件启动的 Agent
3. **多人访问**：可以通过结构化事件轻松跟踪和协调来自不同人的输入
4. **多 Agent**：这个简单抽象可以很容易扩展成支持 `Agent->Agent` 请求和响应
5. **持久可靠**：和 [factor 6：用简单 API 启动 / 暂停 / 恢复](./factor-06-launch-pause-resume.md) 结合后，可以构建持久、可靠、可检查的多人工作流

[这里有更多关于 Outer Loop Agents 的内容](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)

![外层循环 Agent](https://github.com/humanlayer/12-factor-agents/blob/main/img/175-outer-loop-agents.png)

它和 [factor 11：从任何地方触发，在用户所在之处服务用户](./factor-11-trigger-from-anywhere.md) 配合得很好。

[← 启动 / 暂停 / 恢复](./factor-06-launch-pause-resume.md) | [掌控你的控制流 →](./factor-08-own-your-control-flow.md)
