# 第 11 章：通过 Email 进行人工批准

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05/sections/11-humanlayer-approval/README.md`，基准提交：`58b8f09`

本节会添加通过 email 进行人工批准的支持。

一开始会有点刻意设计，目的是先把概念讲清楚。

我们会先从 CLI 调用工作流，但 `divide` 和 `request_more_information` 的 approval 会通过 email 处理，然后最终的 `done_for_now` 答案会打印回 CLI。

虽然有点刻意，但这是一个很好例子，展示了 [factor 7：用工具调用联系人类](../../../../content/zh-CN/factor-07-contact-humans-with-tools.md) 带来的灵活性。


本节关闭 BAML 日志。如果你想看到更多细节，也可以选择打开。

    export BAML_LOG=off

安装 HumanLayer：

    npm install humanlayer

更新 CLI，让它通过 email 把 `divide` 和 `request_more_information` 发给人类。

```diff
src/cli.ts
 // cli.ts lets you invoke the agent loop from the command line

+import { humanlayer } from "humanlayer";
 import { agentLoop, Thread, Event } from "../src/agent";

-
-
 export async function cli() {
     // Get command line arguments, skipping the first two (node and script name)

     // Run the agent loop with the thread
-    const result = await agentLoop(thread);
-    let lastEvent = result.events.slice(-1)[0];
+    let newThread = await agentLoop(thread);
+    let lastEvent = newThread.events.slice(-1)[0];

-    while (lastEvent.data.intent === "request_more_information") {
-        const message = await askHuman(lastEvent.data.message);
-        thread.events.push({ type: "human_response", data: message });
-        const result = await agentLoop(thread);
-        lastEvent = result.events.slice(-1)[0];
+    while (lastEvent.data.intent !== "done_for_now") {
+        const responseEvent = await askHuman(lastEvent);
+        thread.events.push(responseEvent);
+        newThread = await agentLoop(thread);
+        lastEvent = newThread.events.slice(-1)[0];
     }

     // print the final result
     console.log(lastEvent.data.message);
     process.exit(0);
 }

-async function askHuman(message: string) {
+async function askHuman(lastEvent: Event): Promise<Event> {
+    if (process.env.HUMANLAYER_API_KEY) {
+        return await askHumanEmail(lastEvent);
+    } else {
+        return await askHumanCLI(lastEvent.data.message);
+    }
+}
+
+async function askHumanCLI(message: string): Promise<Event> {
     const readline = require('readline').createInterface({
         input: process.stdin,
     return new Promise((resolve) => {
         readline.question(`${message}\n> `, (answer: string) => {
-            resolve(answer);
+            resolve({ type: "human_response", data: answer });
         });
     });
 }
+
+export async function askHumanEmail(lastEvent: Event): Promise<Event> {
+    if (!process.env.HUMANLAYER_EMAIL) {
+        throw new Error("missing or invalid parameters: HUMANLAYER_EMAIL");
+    }
+    const hl = humanlayer({ //reads apiKey from env
+        // name of this agent
+        runId: "12fa-cli-agent",
+        verbose: true,
+        contactChannel: {
+            // agent should request permission via email
+            email: {
+                address: process.env.HUMANLAYER_EMAIL,
+            }
+        }
+    })
+
+    if (lastEvent.data.intent === "divide") {
+        // fetch approval synchronously - this will block until reply
+        const response = await hl.fetchHumanApproval({
+            spec: {
+                fn: "divide",
+                kwargs: {
+                    a: lastEvent.data.a,
+                    b: lastEvent.data.b
+                }
+            }
+        })
+
+        if (response.approved) {
+            const result = lastEvent.data.a / lastEvent.data.b;
+            console.log("tool_response", result);
+            return {
+                "type": "tool_response",
+                "data": result
+            };
+        } else {
+            return {
+                "type": "tool_response",
+                "data": `user denied operation ${lastEvent.data.intent}
+                with feedback: ${response.comment}`
+            };
+        }
+    }
+    throw new Error(`unknown tool: ${lastEvent.data.intent}`)
+}
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/11-cli.ts src/cli.ts

</details>

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


```diff
src/cli.ts
     })

+    if (lastEvent.data.intent === "request_more_information") {
+        // fetch response synchronously - this will block until reply
+        const response = await hl.fetchHumanResponse({
+            spec: {
+                msg: lastEvent.data.message
+            }
+        })
+        return {
+            "type": "tool_response",
+            "data": response
+        }
+    }
+
     if (lastEvent.data.intent === "divide") {
         // fetch approval synchronously - this will block until reply
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/11b-cli.ts src/cli.ts

</details>

用一条包含乱码输入的计算请求，测试 require_approval 流程：


    npx tsx src/index.ts 'can you multiply 4 and xyz'

你应该会收到一封请求澄清的 email：

    Can you clarify what 'xyz' represents in this context? Is it a specific number, variable, or something else?

你可以回复类似这样的内容：

    use 8 instead of xyz

你应该会在 CLI 中看到类似这样的最终结果：

    I have multiplied 4 and xyz, using the value 8 for xyz, resulting in 32.

最后一步，探索如何为 email 使用自定义 HTML 模板。


```diff
src/cli.ts
             email: {
                 address: process.env.HUMANLAYER_EMAIL,
+                // custom email body - jinja
+                template: `{% if type == 'request_more_information' %}
+{{ event.spec.msg }}
+{% else %}
+agent {{ event.run_id }} is requesting approval for {{event.spec.fn}}
+with args: {{event.spec.kwargs}}
+<br><br>
+reply to this email to approve
+{% endif %}`
             }
         }
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/11c-cli.ts src/cli.ts

</details>

先用 divide 试一下：


    npx tsx src/index.ts 'can you divide 4 by 5'

你应该会看到一封略有不同的 email，它使用了自定义模板。

![custom-template-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-custom.png?raw=true)

你可以完整跑一遍这个流程，然后尝试按自己的喜好更新模板。

如果你正在使用 Cursor，只需要高亮模板并让它“make it better”，通常就能达到效果。

也试着触发一下 `request_more_information`。


到这里就结束了。下一章我们会构建一个完全由 email 驱动的 workflow agent，它会使用 webhooks 处理人工批准。
