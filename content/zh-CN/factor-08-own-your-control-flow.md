[← 返回 README](../../README.zh-CN.md)

### 8. 掌控你的控制流

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-08-own-your-control-flow.md`，基准提交：`d20c728`

如果你掌控自己的控制流，就能做很多有意思的事。

![控制流](https://github.com/humanlayer/12-factor-agents/blob/main/img/180-control-flow.png)


构建适合你具体场景的控制结构。尤其是，某些类型的工具调用可能应该让你跳出循环，等待人类响应，或者等待训练 pipeline 这类长时间运行的任务。你可能还想纳入自定义实现，例如：

- 工具调用结果的摘要或缓存
- 对结构化输出执行 LLM-as-judge
- 上下文窗口压缩，或其他[记忆管理](./factor-03-own-your-context-window.md)
- 日志、追踪和指标
- 客户端侧 rate limiting
- 持久 sleep / pause / “wait for event”


下面的例子展示了三种可能的控制流模式：


- request_clarification：模型要求更多信息，跳出循环并等待人类响应
- fetch_git_tags：模型要求获取 git tags 列表，获取 tags，追加到上下文窗口，然后直接交回给模型
- deploy_backend：模型要求部署后端，这是高风险动作，所以跳出循环并等待人类批准

```python
def handle_next_step(thread: Thread):

  while True:
    next_step = await determine_next_step(thread_to_prompt(thread))

    # 为了清晰起见内联在这里。现实中你可以把它放进方法里，
    # 用异常表达控制流，或者使用任何你想要的方式
    if next_step.intent == 'request_clarification':
      thread.events.append({
        type: 'request_clarification',
          data: nextStep,
        })

      await send_message_to_human(next_step)
      await db.save_thread(thread)
      # 异步步骤：跳出循环，之后会收到 webhook
      break
    elif next_step.intent == 'fetch_open_issues':
      thread.events.append({
        type: 'fetch_open_issues',
        data: next_step,
      })

      issues = await linear_client.issues()

      thread.events.append({
        type: 'fetch_open_issues_result',
        data: issues,
      })
      # 同步步骤：把新上下文传给 LLM，让它决定再下一个步骤
      continue
    elif next_step.intent == 'create_issue':
      thread.events.append({
        type: 'create_issue',
        data: next_step,
      })

      await request_human_approval(next_step)
      await db.save_thread(thread)
      # 异步步骤：跳出循环，之后会收到 webhook
      break
```

这个模式允许你按需要中断并恢复 Agent 的流程，从而创建更自然的对话和工作流。

**示例**：我对市面上每个 AI 框架的第一号功能诉求，是我们需要能够中断一个正在工作的 Agent，并在稍后恢复，尤其是在工具**选择**之后、工具**调用**之前。

如果没有这种粒度的可恢复能力，就没有办法在工具调用运行前审查 / 批准它。这意味着你只能被迫选择：

1. 在等待长时间运行的事情完成时，把任务暂停在内存里，例如 `while...sleep`；如果进程被中断，就从头开始
2. 限制 Agent 只能执行低风险、低影响的调用，比如 research 和 summarization
3. 让 Agent 能做更大、更有用的事情，然后只能 yolo 祈祷它别搞砸


你可能会注意到，这和 [factor 5：统一执行状态和业务状态](./factor-05-unify-execution-state.md) 以及 [factor 6：用简单 API 启动 / 暂停 / 恢复](./factor-06-launch-pause-resume.md) 密切相关，但也可以独立实现。

[← 用工具联系人类](./factor-07-contact-humans-with-tools.md) | [压缩错误 →](./factor-09-compact-errors.md)
