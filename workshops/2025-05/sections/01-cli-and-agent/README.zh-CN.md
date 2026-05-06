# 第 1 章：CLI 和 Agent Loop

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05/sections/01-cli-and-agent/README.md`，基准提交：`58b8f09`

现在加入 BAML，并创建第一个带 CLI 接口的 agent。

首先需要安装 [BAML](https://github.com/boundaryml/baml)，它是一个用于 prompting 和结构化输出的工具。


    npm install @boundaryml/baml

初始化 BAML：

    npx baml-cli init

移除默认的 `resume.baml`：

    rm baml_src/resume.baml

添加起始 agent，也就是后续会不断扩展的单个 BAML prompt：

    cp ./walkthrough/01-agent.baml baml_src/agent.baml

<details>
<summary>显示文件</summary>

```rust
// ./walkthrough/01-agent.baml
class DoneForNow {
  intent "done_for_now"
  message string
}

function DetermineNextStep(
    thread: string
) -> DoneForNow {
    client "openai/gpt-4o"

    prompt #"
        {{ _.role("system") }}

        You are a helpful assistant that can help with tasks.

        {{ _.role("user") }}

        You are working on the following thread:

        {{ thread }}

        What should the next step be?

        {{ ctx.output_format }}
    "#
}

test HelloWorld {
  functions [DetermineNextStep]
  args {
    thread #"
      {
        "type": "user_input",
        "data": "hello!"
      }
    "#
  }
}
```

</details>

生成 BAML client 代码：

    npx baml-cli generate

为本节启用 BAML 日志：

    export BAML_LOG=debug

添加 CLI 接口：

    cp ./walkthrough/01-cli.ts src/cli.ts

<details>
<summary>显示文件</summary>

```ts
// ./walkthrough/01-cli.ts
// cli.ts lets you invoke the agent loop from the command line

import { agentLoop, Thread, Event } from "./agent";

export async function cli() {
    // Get command line arguments, skipping the first two (node and script name)
    const args = process.argv.slice(2);

    if (args.length === 0) {
        console.error("Error: Please provide a message as a command line argument");
        process.exit(1);
    }

    // Join all arguments into a single message
    const message = args.join(" ");

    // Create a new thread with the user's message as the initial event
    const thread = new Thread([{ type: "user_input", data: message }]);

    // Run the agent loop with the thread
    const result = await agentLoop(thread);
    console.log(result);
}
```

</details>

更新 `index.ts`，让它使用 CLI：

```diff
src/index.ts
+import { cli } from "./cli"
+
 async function hello(): Promise<void> {
     console.log('hello, world!')

 async function main() {
-    await hello()
+    await cli()
 }

```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/01-index.ts src/index.ts

</details>

添加 agent 实现：

    cp ./walkthrough/01-agent.ts src/agent.ts

<details>
<summary>显示文件</summary>

```ts
// ./walkthrough/01-agent.ts
import { b } from "../baml_client";

// tool call or a respond to human tool
type AgentResponse = Awaited<ReturnType<typeof b.DetermineNextStep>>;

export interface Event {
    type: string
    data: any;
}

export class Thread {
    events: Event[] = [];

    constructor(events: Event[]) {
        this.events = events;
    }

    serializeForLLM() {
        // can change this to whatever custom serialization you want to do, XML, etc
        // e.g. https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
        return JSON.stringify(this.events);
    }
}

// right now this just runs one turn with the LLM, but
// we'll update this function to handle all the agent logic
export async function agentLoop(thread: Thread): Promise<AgentResponse> {
    const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
    return nextStep;
}
```

</details>

这段 BAML 代码默认配置为使用 `OPENAI_API_KEY`。

测试时，你可以按需把 model / provider 改成其他选择。

        client "openai/gpt-4o"

[BAML clients 文档在这里](https://docs.boundaryml.com/guide/baml-basics/switching-llms)。

例如，你可以把 [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 或 [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) 配置为模型提供商。

如果你想不做任何修改直接运行示例，可以把 `OPENAI_API_KEY` 环境变量设置为任意有效的 OpenAI key。


    export OPENAI_API_KEY=...

试一下：

    npx tsx src/index.ts hello

你应该会看到模型返回一个熟悉的响应：

    {
  intent: 'done_for_now',
  message: 'Hello! How can I assist you today?'
}
