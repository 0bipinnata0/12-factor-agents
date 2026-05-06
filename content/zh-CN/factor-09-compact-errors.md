[← 返回 README](../../README.zh-CN.md)

### 9. 把错误压缩进上下文窗口

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-09-compact-errors.md`，基准提交：`d20c728`

这一条比较短，但值得单独提出来。Agent 的一个好处是“自愈”：对短任务来说，LLM 可能会调用一个失败的工具。优秀的 LLM 很有机会读懂错误消息或 stack trace，并在后续工具调用中判断应该改什么。


大多数框架都会实现这一点，但你可以只做这一点，而不用做其他 11 个 factor。下面是一个例子：


```python
thread = {"events": [initial_message]}

while True:
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread["events"].append({
    "type": next_step.intent,
    "data": next_step,
  })
  try:
    result = await handle_next_step(thread, next_step) # 我们的 switch statement
  except Exception as e:
    # 如果遇到错误，可以把它加入上下文窗口并重试
    thread["events"].append({
      "type": 'error',
      "data": format_error(e),
    })
    # 继续循环，或者在这里做任何其他恢复逻辑
```

你可能会想为某个特定工具调用实现一个 errorCounter，把单个工具限制在大约 3 次尝试内，或者实现其他适合你场景的逻辑。

```python
consecutive_errors = 0

while True:

  # ... existing code ...

  try:
    result = await handle_next_step(thread, next_step)
    thread["events"].append({
      "type": next_step.intent + '_result',
      data: result,
    })
    # 成功！重置错误计数器
    consecutive_errors = 0
  except Exception as e:
    consecutive_errors += 1
    if consecutive_errors < 3:
      # 继续循环并重试
      thread["events"].append({
        "type": 'error',
        "data": format_error(e),
      })
    else:
      # 跳出循环，重置上下文窗口的某些部分，升级给人类，或者做其他你想做的事
      break
  }
}
```

达到某个连续错误阈值，可能正是[升级给人类](./factor-07-contact-humans-with-tools.md)的好时机。这个升级可以由模型决定，也可以由确定性代码接管控制流来完成。

[![Factor 09 错误动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/195-factor-09-errors.gif)](https://github.com/user-attachments/assets/cd7ed814-8309-4baf-81a5-9502f91d4043)


<details>
<summary>[GIF 版本](https://github.com/humanlayer/12-factor-agents/blob/main/img/195-factor-09-errors.gif)</summary>

![Factor 09 错误动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/195-factor-09-errors.gif)

</details>

好处：

1. **自愈**：LLM 可以读取错误消息，并在后续工具调用中判断应该改什么
2. **持久可靠**：即使某个工具调用失败，Agent 仍然可以继续运行

我确信你会发现，如果这件事做得太多，你的 Agent 会开始失控，并可能一遍又一遍重复同一个错误。

这就是 [factor 8：掌控你的控制流](./factor-08-own-your-control-flow.md) 和 [factor 3：掌控你的上下文构建](./factor-03-own-your-context-window.md) 的用武之地。你不需要只是把原始错误塞回去；你可以完全重构错误的表示方式，从上下文窗口中移除之前的事件，或者做任何能让 Agent 回到正轨的确定性处理。

但防止错误失控的第一方法，是拥抱 [factor 10：小而专注的 Agent](./factor-10-small-focused-agents.md)。

[← 掌控你的控制流](./factor-08-own-your-control-flow.md) | [小而专注的 Agent →](./factor-10-small-focused-agents.md)
