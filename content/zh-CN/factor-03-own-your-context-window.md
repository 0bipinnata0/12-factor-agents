[← 返回 README](../../README.zh-CN.md)

### 3. 掌控你的上下文窗口

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-03-own-your-context-window.md`，基准提交：`d20c728`

你不一定要使用标准的、基于 message 的格式，把上下文传给 LLM。

> #### 在 Agent 的任意时刻，你给 LLM 的输入本质上都是：“到目前为止发生了这些事，下一步该做什么？”

<!-- todo syntax highlighting -->
<!-- ![130-own-your-context-building](https://github.com/humanlayer/12-factor-agents/blob/main/img/130-own-your-context-building.png) -->

一切都是上下文工程。[LLM 是无状态函数](https://thedataexchange.media/baml-revolution-in-ai-engineering/)，把输入转换成输出。想得到最好的输出，你就需要给它最好的输入。

构建优秀上下文意味着：

- 你给模型的 prompt 和指令
- 你检索到的任何文档或外部数据，例如 RAG
- 任何过去的状态、工具调用、结果或其他历史
- 来自相关但独立的历史 / 对话中的过去消息或事件，也就是 Memory
- 关于应该输出哪类结构化数据的指令

![上下文工程示意图](https://github.com/user-attachments/assets/0f1f193f-8e94-4044-a276-576bd7764fd0)


### 关于上下文工程

这份指南关注的是如何尽可能榨干今天这些模型的能力。这里刻意没有讨论：

- temperature、top_p、frequency_penalty、presence_penalty 等模型参数调整
- 训练你自己的 completion 或 embedding 模型
- 微调已有模型

同样，我不知道把上下文交给 LLM 的最佳方式是什么，但我知道你会想要能尝试一切的灵活性。

#### 标准上下文格式 vs 自定义上下文格式

大多数 LLM client 会使用类似这样的标准 message-based 格式：

```yaml
[
  {
    "role": "system",
    "content": "You are a helpful assistant..."
  },
  {
    "role": "user",
    "content": "Can you deploy the backend?"
  },
  {
    "role": "assistant",
    "content": null,
    "tool_calls": [
      {
        "id": "1",
        "name": "list_git_tags",
        "arguments": "{}"
      }
    ]
  },
  {
    "role": "tool",
    "name": "list_git_tags",
    "content": "{\"tags\": [{\"name\": \"v1.2.3\", \"commit\": \"abc123\", \"date\": \"2024-03-15T10:00:00Z\"}, {\"name\": \"v1.2.2\", \"commit\": \"def456\", \"date\": \"2024-03-14T15:30:00Z\"}, {\"name\": \"v1.2.1\", \"commit\": \"abe033d\", \"date\": \"2024-03-13T09:15:00Z\"}]}",
    "tool_call_id": "1"
  }
]
```

这对大多数场景很好用。但如果你想真正榨出今天 LLM 的最大能力，就需要用尽可能节省 token、也尽可能节省注意力的方式，把上下文送进 LLM。

作为标准 message-based 格式的替代方案，你可以构建适合自己场景的上下文格式。比如，你可以使用自定义对象，然后按需要把它们打包 / 展开到一个或多个 user、system、assistant 或 tool message 中。

下面是把整个上下文窗口放进单个 user message 的例子：

```yaml

[
  {
    "role": "system",
    "content": "You are a helpful assistant..."
  },
  {
    "role": "user",
    "content": |
            Here's everything that happened so far:

        <slack_message>
            From: @alex
            Channel: #deployments
            Text: Can you deploy the backend?
        </slack_message>

        <list_git_tags>
            intent: "list_git_tags"
        </list_git_tags>

        <list_git_tags_result>
            tags:
              - name: "v1.2.3"
                commit: "abc123"
                date: "2024-03-15T10:00:00Z"
              - name: "v1.2.2"
                commit: "def456"
                date: "2024-03-14T15:30:00Z"
              - name: "v1.2.1"
                commit: "ghi789"
                date: "2024-03-13T09:15:00Z"
        </list_git_tags_result>

        what's the next step?
    }
]
```

模型也许能通过你提供的 tool schemas 推断出你是在问 `what's the next step`，但把它放进 prompt 模板里通常没有坏处。

### 代码示例

我们可以用类似这样的方式构建：

```python

class Thread:
  events: List[Event]

class Event:
  # 可以只用 string，也可以显式建模，取决于你
  type: Literal["list_git_tags", "deploy_backend", "deploy_frontend", "request_more_information", "done_for_now", "list_git_tags_result", "deploy_backend_result", "deploy_frontend_result", "request_more_information_result", "done_for_now_result", "error"]
  data: ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation |
        ListGitTagsResult | DeployBackendResult | DeployFrontendResult | RequestMoreInformationResult | string

def event_to_prompt(event: Event) -> str:
    data = event.data if isinstance(event.data, str) \
           else stringifyToYaml(event.data)

    return f"<{event.type}>\n{data}\n</{event.type}>"


def thread_to_prompt(thread: Thread) -> str:
  return '\n\n'.join(event_to_prompt(event) for event in thread.events)
```

#### 上下文窗口示例

使用这种方式时，上下文窗口大概会是这样：

**初始 Slack 请求：**

```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
</slack_message>
```

**列出 Git Tags 之后：**

```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
    Thread: []
</slack_message>

<list_git_tags>
    intent: "list_git_tags"
</list_git_tags>

<list_git_tags_result>
    tags:
      - name: "v1.2.3"
        commit: "abc123"
        date: "2024-03-15T10:00:00Z"
      - name: "v1.2.2"
        commit: "def456"
        date: "2024-03-14T15:30:00Z"
      - name: "v1.2.1"
        commit: "ghi789"
        date: "2024-03-13T09:15:00Z"
</list_git_tags_result>
```

**出现错误并恢复之后：**

```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
    Thread: []
</slack_message>

<deploy_backend>
    intent: "deploy_backend"
    tag: "v1.2.3"
    environment: "production"
</deploy_backend>

<error>
    error running deploy_backend: Failed to connect to deployment service
</error>

<request_more_information>
    intent: "request_more_information_from_human"
    question: "I had trouble connecting to the deployment service, can you provide more details and/or check on the status of the service?"
</request_more_information>

<human_response>
    data:
      response: "I'm not sure what's going on, can you check on the status of the latest workflow?"
</human_response>
```

从这里开始，你的下一步可能是：

```python
nextStep = await determine_next_step(thread_to_prompt(thread))
```

```python
{
  "intent": "get_workflow_status",
  "workflow_name": "tag_push_prod.yaml",
}
```

XML 风格的格式只是一个例子。重点是：你可以构建适合自己应用的格式。如果你能灵活地试验不同上下文结构，以及试验哪些东西应该存储、哪些东西应该传给 LLM，就会得到更好的质量。

掌控上下文窗口的关键好处：

1. **信息密度**：用能最大化 LLM 理解能力的方式组织信息
2. **错误处理**：用有助于 LLM 恢复的格式包含错误信息。错误被解决后，可以考虑从上下文窗口里隐藏错误和失败调用
3. **安全性**：控制传给 LLM 的信息，过滤敏感数据
4. **灵活性**：随着你了解什么对场景有效，调整上下文格式
5. **Token 效率**：为了 token 效率和 LLM 理解能力优化上下文格式

上下文包括：prompts、instructions、RAG documents、history、tool calls、memory。


记住：上下文窗口是你和 LLM 之间最主要的接口。掌控你如何组织和呈现信息，可以显著提升 Agent 的表现。

示例：信息密度，同一条消息，更少 token：

![Loom 截图 2025-04-22 09:00:56](https://github.com/user-attachments/assets/5cf041c6-72da-4943-be8a-99c73162b12a)


### 不要只听我说

12-factor agents 发布大约 2 个月后，context engineering 开始成为一个相当流行的术语。

<a href="https://x.com/karpathy/status/1937902205765607626"><img width="378" alt="Karpathy 关于 context engineering 的截图" src="https://github.com/user-attachments/assets/97e6e667-c35f-4855-8233-af40f05d6bce" /></a> <a href="https://x.com/tobi/status/1935533422589399127"><img width="378" alt="Tobi 关于 context engineering 的截图" src="https://github.com/user-attachments/assets/7e6f5738-0d38-4910-82d1-7f5785b82b99" /></a>

[@lenadroid](https://x.com/lenadroid) 在 2025 年 7 月也写了一份很不错的 [Context Engineering Cheat Sheet](https://x.com/lenadroid/status/1943685060785524824)。

<a href="https://x.com/lenadroid/status/1943685060785524824"><img width="256" alt="Context Engineering Cheat Sheet 截图" src="https://github.com/user-attachments/assets/cac88aa3-8faf-440b-9736-cab95a9de477" /></a>



这里反复出现的主题是：我不知道最佳方法是什么，但我知道你会想要能尝试一切的灵活性。


[← 掌控你的提示词](./factor-02-own-your-prompts.md) | [工具本质上只是结构化输出 →](./factor-04-tools-are-structured-outputs.md)
