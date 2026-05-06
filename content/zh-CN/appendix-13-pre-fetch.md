[← 返回 README](../../README.zh-CN.md)

### Factor 13：预取所有可能需要的上下文

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/appendix-13-pre-fetch.md`，基准提交：`d20c728`

如果你的模型很可能会调用工具 X，就不要浪费 token 往返，让模型先去取它。也就是说，不要写这种伪 prompt：

```jinja
When looking at deployments, you will likely want to fetch the list of published git tags,
so you can use it to deploy to prod.

Here's what happened so far:

{{ thread.events }}

What's the next step?

Answer in JSON format with one of the following intents:

{
  intent: 'deploy_backend_to_prod',
  tag: string
} OR {
  intent: 'list_git_tags'
} OR {
  intent: 'done_for_now',
  message: string
}
```

然后代码像这样：

```python
thread = {"events": [initial_message]}
next_step = await determine_next_step(thread)

while True:
  switch next_step.intent:
    case 'list_git_tags':
      tags = await fetch_git_tags()
      thread["events"].append({
        type: 'list_git_tags',
        data: tags,
      })
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append({
        "type": 'deploy_backend_to_prod',
        "data": deploy_result,
      })
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

你不如直接把 tags 取出来并放进上下文窗口，例如：

```diff
- When looking at deployments, you will likely want to fetch the list of published git tags,
- so you can use it to deploy to prod.

+ The current git tags are:

+ {{ git_tags }}


Here's what happened so far:

{{ thread.events }}

What's the next step?

Answer in JSON format with one of the following intents:

{
  intent: 'deploy_backend_to_prod',
  tag: string
- } OR {
-   intent: 'list_git_tags'
} OR {
  intent: 'done_for_now',
  message: string
}

```

然后代码像这样：

```diff
thread = {"events": [initial_message]}
+ git_tags = await fetch_git_tags()

- next_step = await determine_next_step(thread)
+ next_step = await determine_next_step(thread, git_tags)

while True:
  switch next_step.intent:
-    case 'list_git_tags':
-      tags = await fetch_git_tags()
-      thread["events"].append({
-        type: 'list_git_tags',
-        data: tags,
-      })
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append({
        "type": 'deploy_backend_to_prod',
        "data": deploy_result,
      })
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

或者甚至可以直接把 tags 放进 thread，并从 prompt template 里移除这个特定参数：

```diff
thread = {"events": [initial_message]}
+ # 添加请求
+ thread["events"].append({
+  "type": 'list_git_tags',
+ })

git_tags = await fetch_git_tags()

+ # 添加结果
+ thread["events"].append({
+  "type": 'list_git_tags_result',
+  "data": git_tags,
+ })

- next_step = await determine_next_step(thread, git_tags)
+ next_step = await determine_next_step(thread)

while True:
  switch next_step.intent:
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append(deploy_result)
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

总的来说：

> #### 如果你已经知道模型会想调用哪些工具，就直接用确定性方式调用它们，然后让模型去做更难的事：弄清楚如何使用这些输出

再次强调，AI engineering 本质上就是[上下文工程](./factor-03-own-your-context-window.md)。

[← 无状态 Reducer](./factor-12-stateless-reducer.md) | [更多阅读 →](../../README.zh-CN.md#相关资源)
