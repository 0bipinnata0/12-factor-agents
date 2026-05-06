# 第 0 章：Hello World

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05/sections/final/README.md`，基准提交：`58b8f09`

先从基础 TypeScript 设置和一个 hello world 程序开始。

本指南使用 TypeScript 编写（没错，Python 版本很快会来）。

workshop 的每次文件修改之间都有很多 checkpoint，
所以即使你对 TypeScript 不是特别熟，
也应该能跟上并运行每个示例。

要运行本指南，你需要安装较新的 nodejs 和 npm。

你可以使用任何 nodejs 版本管理器，[homebrew](https://formulae.brew.sh/formula/node) 也可以。


    brew install node@20

你应该能看到 node 版本：

    node --version

复制初始 `package.json`：

    cp ./walkthrough/00-package.json package.json

安装依赖：

    npm install

复制 `tsconfig.json`：

    cp ./walkthrough/00-tsconfig.json tsconfig.json

添加 `.gitignore`：

    cp ./walkthrough/00-.gitignore .gitignore

创建 `src` 文件夹。

    mkdir -p src

添加一个简单的 hello world `index.ts`：

    cp ./walkthrough/00-index.ts src/index.ts

运行它来验证：

    npx tsx src/index.ts

你应该会看到：

    hello, world!


# 第 1 章：CLI 和 Agent Loop

现在加入 BAML，并创建第一个带 CLI 接口的 agent。

首先需要安装 [BAML](https://github.com/boundaryml/baml)，它是一个用于 prompting 和结构化输出的工具。


    npm install @boundaryml/baml

初始化 BAML：

    npx baml-cli init

移除默认的 `resume.baml`：

    rm baml_src/resume.baml

添加起始 agent，也就是后续会不断扩展的单个 BAML prompt：

    cp ./walkthrough/01-agent.baml baml_src/agent.baml

生成 BAML client 代码：

    npx baml-cli generate

为本节启用 BAML 日志：

    export BAML_LOG=debug

添加 CLI 接口：

    cp ./walkthrough/01-cli.ts src/cli.ts

更新 `index.ts`，让它使用 CLI：

    cp ./walkthrough/01-index.ts src/index.ts

添加 agent 实现：

    cp ./walkthrough/01-agent.ts src/agent.ts

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


# 第 2 章：添加计算器工具

给我们的 agent 添加一些计算器工具。

先为计算器添加工具定义。

这些只是简单的结构化输出，我们会要求模型在 agentic loop 中把它们作为“下一步”返回。


    cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml

现在更新 agent 的 `DetermineNextStep` 方法，把计算器工具暴露为潜在的下一步。


    cp ./walkthrough/02-agent.baml baml_src/agent.baml

生成更新后的 BAML client：

    npx baml-cli generate

试用计算器：

    npx tsx src/index.ts 'can you add 3 and 4'

你应该会看到一次对计算器的工具调用：

    {
  intent: 'add',
  a: 3,
  b: 4
}


# 第 3 章：在循环中处理工具调用

现在添加一个真正的 agentic loop，让它能够运行工具，并从 LLM 得到最终答案。

首先，更新 agent，让它能处理工具调用。


    cp ./walkthrough/03-agent.ts src/agent.ts

现在试一下：


    npx tsx src/index.ts 'can you add 3 and 4'

你应该会看到 agent 调用工具，然后返回结果：

    {
  intent: 'done_for_now',
  message: 'The sum of 3 and 4 is 7.'
}

下一步我们做一个更复杂的计算。先关闭 BAML 日志，让输出更简洁：

    export BAML_LOG=off

试一个多步骤计算：

    npx tsx src/index.ts 'can you add 3 and 4, then add 6 to that result'

你会注意到，multiply 和 divide 这类工具还不可用：

    npx tsx src/index.ts 'can you multiply 3 and 4'

接下来，为其余计算器工具添加 handler。


    cp ./walkthrough/03b-agent.ts src/agent.ts

测试减法：

    npx tsx src/index.ts 'can you subtract 3 from 4'

现在测试乘法工具：


    npx tsx src/index.ts 'can you multiply 3 and 4'

最后，测试一个包含多个操作的更复杂计算：


    npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'


# 第 4 章：给 agent.baml 添加测试

给 BAML agent 添加一些测试。

开始时，保持 BAML 日志开启：

    export BAML_LOG=debug

接下来，给 agent 添加一些测试。

先从一个简单测试开始，检查 agent 处理基础计算的能力。


    cp ./walkthrough/04-agent.baml baml_src/agent.baml

运行测试：

    npx baml-cli test

现在，用断言改进这个测试。

断言是确保 agent 按预期工作的好办法，也很容易扩展，用于检查更复杂的行为。


    cp ./walkthrough/04b-agent.baml baml_src/agent.baml

运行测试：

    npx baml-cli test

随着测试变多，你可以关闭日志，让输出更干净。
在迭代某些具体测试时，也可以再把它们打开。


    export BAML_LOG=off

现在添加一些更复杂的测试用例，
模拟从一个正在进行中的 agentic 上下文窗口中间恢复。


    cp ./walkthrough/04c-agent.baml baml_src/agent.baml

试着运行它：


    npx baml-cli test


# 第 5 章：多种人类工具

本节会添加对多个“联系人类”工具的支持。


本节关闭 BAML 日志。如果你想看到更多细节，也可以选择打开。

    export BAML_LOG=off

首先，添加一个可以向人类请求澄清的工具。

它会不同于 `done_for_now` 工具，
并且可以在 agent 中更灵活地处理不同类型的人类交互。


    cp ./walkthrough/05-agent.baml baml_src/agent.baml

接下来，重新生成 client 代码。

注意：如果你正在使用 BAML 的 VSCode 扩展，
当你在编辑器里保存文件时，client 会自动重新生成。


    npx baml-cli generate

现在更新 agent，让它使用这个新工具。


    cp ./walkthrough/05-agent.ts src/agent.ts

接下来更新 CLI，让它通过命令行向用户请求输入，从而处理澄清请求。


    cp ./walkthrough/05-cli.ts src/cli.ts

试一下：


    npx tsx src/index.ts 'can you multiply 3 and FD*(#F&& '

接下来添加一个测试，检查 agent 处理澄清请求的能力。


    cp ./walkthrough/05b-agent.baml baml_src/agent.baml

现在可以再次运行测试：


    npx baml-cli test

你会注意到新测试通过了，但 hello world 测试失败了。

这是因为 agent 的默认行为是返回 `done_for_now`。


    cp ./walkthrough/05c-agent.baml baml_src/agent.baml

验证测试通过：

    npx baml-cli test


# 第 6 章：用推理步骤自定义提示词

本节会探索如何用 reasoning steps 自定义 agent 的 prompt。

这是 [factor 2：掌控你的提示词](../../../../content/zh-CN/factor-02-own-your-prompts.md) 的核心。

AI That Works 上有一篇关于 reasoning 的深入文章：[reasoning models versus reasoning steps](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)。


本节保持 BAML 日志开启会比较有帮助。

    export BAML_LOG=debug

更新 agent prompt，加入一个 reasoning step。


    cp ./walkthrough/06-agent.baml baml_src/agent.baml

生成更新后的 client：

    npx baml-cli generate

现在可以用一个简单 prompt 试一下：


    npx tsx src/index.ts 'can you multiply 3 and 4'

你应该能从 BAML 日志中看到展示 reasoning steps 的输出。

#### 可选挑战

在工具输出格式中添加一个字段，把 reasoning steps 包含到输出里。



# 第 7 章：自定义上下文窗口

本节会探索如何自定义 agent 的上下文窗口。

这是 [factor 3：掌控你的上下文窗口](../../../../content/zh-CN/factor-03-own-your-context-window.md) 的核心。


更新 agent，让它为模型 pretty-print 上下文窗口。


    cp ./walkthrough/07-agent.ts src/agent.ts

测试格式化效果：

    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

接下来，把 agent 更新为使用 XML 格式。

这是向模型传递数据时非常流行的一种格式。

原因之一是 XML 的 token 效率。


    cp ./walkthrough/07b-agent.ts src/agent.ts

试一下：


    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

更新测试，让它们匹配新的输出格式。


    cp ./walkthrough/07c-agent.baml baml_src/agent.baml

检查更新后的测试：


    npx baml-cli test


# 第 8 章：添加 API Endpoints

添加一个 Express server，通过 HTTP 暴露 agent。

本节关闭 BAML 日志。如果你想看到更多细节，也可以选择打开。

    export BAML_LOG=off

安装 Express 和类型：

    npm install express && npm install --save-dev @types/express supertest

添加 server 实现：

    cp ./walkthrough/08-server.ts src/server.ts

启动 server：

    npx tsx src/server.ts

用 curl 测试（在另一个终端中）：

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you add 3 and 4"}'

你应该会得到 agent 的响应，其中包含 agentic trace，并以类似这样的消息结尾：


    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}


# 第 9 章：内存状态和异步澄清

添加状态管理和异步澄清支持。

本节关闭 BAML 日志。如果你想看到更多细节，也可以选择打开。

    export BAML_LOG=off

为 threads 添加简单的内存状态管理：

    cp ./walkthrough/09-state.ts src/state.ts

更新 server，让它使用状态管理。

* 使用 `ThreadStore` 添加 thread 状态管理
* 从 `/thread` endpoint 返回 thread ID 和 response URL
* 实现 `GET /thread/:id`
* 实现 `POST /thread/:id/response`


    cp ./walkthrough/09-server.ts src/server.ts

启动 server：

    npx tsx src/server.ts

测试澄清流程：

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you multiply 3 and xyz"}'


# 第 10 章：添加人工批准

添加对操作人工批准的支持。

本节关闭 BAML 日志。如果你想看到更多细节，也可以选择打开。

    export BAML_LOG=off

更新 server，让它处理人工批准。

* 导入 `handleNextStep` 来执行已批准的动作
* 添加两种 payload 类型，用于区分 approval 和 response
* 在 endpoint 中分别处理 response 和 approval
* 出错时展示更好的错误消息


    cp ./walkthrough/10-server.ts src/server.ts

给 agent 添加几个方法，用于处理 approval 和 response。

    cp ./walkthrough/10-agent.ts src/agent.ts

启动 server：

    npx tsx src/server.ts

测试带 approval 的除法：

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you divide 3 by 4"}'

你应该会看到：

    {
  "thread_id": "2b243b66-215a-4f37-8bc6-9ace3849043b",
  "events": [
    {
      "type": "user_input",
      "data": "can you divide 3 by 4"
    },
    {
      "type": "tool_call",
      "data": {
        "intent": "divide",
        "a": 3,
        "b": 4,
        "response_url": "/thread/2b243b66-215a-4f37-8bc6-9ace3849043b/response"
      }
    }
  ]
}

用另一个 curl 调用拒绝请求，记得替换 thread ID：

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": false, "comment": "I dont think thats right, use 5 instead of 4"}'

你应该会看到： the last tool call is now `"intent":"divide","a":3,"b":5`

    {
  "events": [
    {
      "type": "user_input",
      "data": "can you divide 3 by 4"
    },
    {
      "type": "tool_call",
      "data": {
        "intent": "divide",
        "a": 3,
        "b": 4,
        "response_url": "/thread/2b243b66-215a-4f37-8bc6-9ace3849043b/response"
      }
    },
    {
      "type": "tool_response",
      "data": "user denied the operation with feedback: \"I dont think thats right, use 5 instead of 4\""
    },
    {
      "type": "tool_call",
      "data": {
        "intent": "divide",
        "a": 3,
        "b": 5,
        "response_url": "/thread/1f1f5ff5-20d7-4114-97b4-3fc52d5e0816/response"
      }
    }
  ]
}

现在可以批准这个操作：

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": true}'

你应该会看到最终消息中包含工具响应和最终结果。

    ...
{
  "type": "tool_response",
  "data": 0.5
},
{
  "type": "done_for_now",
  "message": "I divided 3 by 6 and the result is 0.5. If you have any more operations or queries, feel free to ask!",
  "response_url": "/thread/2b469403-c497-4797-b253-043aae830209/response"
}


# 第 11 章：通过 Email 进行人工批准

本节会添加通过 email 进行人工批准的支持。

一开始会有点刻意设计，目的是先把概念讲清楚。

我们会先从 CLI 调用工作流，但 `divide` 和 `request_more_information` 的 approval 会通过 email 处理，然后最终的 `done_for_now` 答案会打印回 CLI。

虽然有点刻意，但这是一个很好例子，展示了 [factor 7：用工具调用联系人类](../../../../content/zh-CN/factor-07-contact-humans-with-tools.md) 带来的灵活性。


本节关闭 BAML 日志。如果你想看到更多细节，也可以选择打开。

    export BAML_LOG=off

安装 HumanLayer：

    npm install humanlayer

更新 CLI，让它通过 email 把 `divide` 和 `request_more_information` 发给人类。

    cp ./walkthrough/11-cli.ts src/cli.ts

运行 CLI：

    npx tsx src/index.ts 'can you divide 4 by 5'

程序的最后一行应该会提到人工 review 步骤：

    nextStep { intent: 'divide', a: 4, b: 5 }
HumanLayer: Requested human approval from HumanLayer cloud

继续，回复这封 email 并提供一些反馈：

![reject-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-reject.png?raw=true)


你应该会收到另一封 email，其中包含基于你反馈更新后的尝试。

你可以继续批准这一次：

![approve-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-approve.png?raw=true)


最终输出会像这样：

    nextStep {
 intent: 'done_for_now',
 message: 'The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!'
}
The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!

也实现一下 `request_more_information` 流程。


    cp ./walkthrough/11b-cli.ts src/cli.ts

用一条包含乱码输入的计算请求，测试 require_approval 流程：


    npx tsx src/index.ts 'can you multiply 4 and xyz'

你应该会收到一封请求澄清的 email：

    Can you clarify what 'xyz' represents in this context? Is it a specific number, variable, or something else?

你可以回复类似这样的内容：

    use 8 instead of xyz

你应该会在 CLI 中看到类似这样的最终结果：

    I have multiplied 4 and xyz, using the value 8 for xyz, resulting in 32.

最后一步，探索如何为 email 使用自定义 HTML 模板。


    cp ./walkthrough/11c-cli.ts src/cli.ts

先用 divide 试一下：


    npx tsx src/index.ts 'can you divide 4 by 5'

你应该会看到一封略有不同的 email，它使用了自定义模板。

![custom-template-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-custom.png?raw=true)

你可以完整跑一遍这个流程，然后尝试按自己的喜好更新模板。

如果你正在使用 Cursor，只需要高亮模板并让它“make it better”，通常就能达到效果。

也试着触发一下 `request_more_information`。


到这里就结束了。下一章我们会构建一个完全由 email 驱动的 workflow agent，它会使用 webhooks 处理人工批准。



# 第 XX 章：HumanLayer Webhook 集成

前几节使用的是 humanlayer SDK 的“同步模式”。这意味着每次等待人工批准时，我们都会停在循环里不断 polling，直到收到人类响应。

这显然不理想，尤其是对生产 workload 来说。
所以本节会实现 [factor 6：用简单 API 启动 / 暂停 / 恢复](../../../../content/zh-CN/factor-06-launch-pause-resume.md)：更新 server，让它在联系人类后结束处理，并使用 webhooks 接收结果。


在 server 中添加初始化 humanlayer 的代码。


    cp ./walkthrough/12-1-server-init.ts src/server.ts

接下来，更新 `/thread` endpoint，让它：

1. 异步处理请求，并立即返回
2. 在 `request_more_information` 和 `done_for_now` 调用时创建 human contact


更新 server，让它能够处理 request_clarification 响应。

- 移除旧的 `/response` endpoint 和 types
- 更新 `/thread` endpoint，让处理异步运行并立即返回
- 请求人类响应时发送 `state.threadId`
- 添加 `handleHumanResponse` 函数处理人类响应
- 添加 `/webhook` endpoint 处理 webhook 响应


    cp ./walkthrough/12a-server.ts src/server.ts

启动 server： in another terminal

    npx tsx src/server.ts

server 运行后，向 `/thread` endpoint 发送一个 payload。


__ 执行 response 步骤

__ 现在处理 divide 的 approvals

__ 现在也处理 done_for_now
