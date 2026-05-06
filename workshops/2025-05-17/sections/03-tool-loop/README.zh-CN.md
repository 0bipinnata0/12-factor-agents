# 第 3 章：在循环中处理工具调用

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05-17/sections/03-tool-loop/README.md`，基准提交：`c62d647`

现在添加一个真正的 agentic loop，让它能够运行工具，并从 LLM 得到最终答案。

首先，更新 agent，让它能处理工具调用。


```diff
src/agent.ts
 }

-// right now this just runs one turn with the LLM, but
-// we'll update this function to handle all the agent logic
-export async function agentLoop(thread: Thread): Promise<AgentResponse> {
-    const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
-    return nextStep;
+
+
+export async function agentLoop(thread: Thread): Promise<string> {
+
+    while (true) {
+        const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
+        console.log("nextStep", nextStep);
+
+        switch (nextStep.intent) {
+            case "done_for_now":
+                // response to human, return the next step object
+                return nextStep.message;
+            case "add":
+                thread.events.push({
+                    "type": "tool_call",
+                    "data": nextStep
+                });
+                const result = nextStep.a + nextStep.b;
+                console.log("tool_response", result);
+                thread.events.push({
+                    "type": "tool_response",
+                    "data": result
+                });
+                continue;
+            default:
+                throw new Error(`Unknown intent: ${nextStep.intent}`);
+        }
+    }
 }

```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/03-agent.ts src/agent.ts

</details>

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


```diff
src/agent.ts
-import { b } from "../baml_client";
+import { AddTool, SubtractTool, DivideTool, MultiplyTool, b } from "../baml_client";

-// tool call or a respond to human tool
-type AgentResponse = Awaited<ReturnType<typeof b.DetermineNextStep>>;
-
 export interface Event {
     type: string
 }

+export type CalculatorTool = AddTool | SubtractTool | MultiplyTool | DivideTool;

+export async function handleNextStep(nextStep: CalculatorTool, thread: Thread): Promise<Thread> {
+    let result: number;
+    switch (nextStep.intent) {
+        case "add":
+            result = nextStep.a + nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+        case "subtract":
+            result = nextStep.a - nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+        case "multiply":
+            result = nextStep.a * nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+        case "divide":
+            result = nextStep.a / nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+    }
+}

 export async function agentLoop(thread: Thread): Promise<string> {
         console.log("nextStep", nextStep);

+        thread.events.push({
+            "type": "tool_call",
+            "data": nextStep
+        });
+
         switch (nextStep.intent) {
             case "done_for_now":
                 return nextStep.message;
             case "add":
-                thread.events.push({
-                    "type": "tool_call",
-                    "data": nextStep
-                });
-                const result = nextStep.a + nextStep.b;
-                console.log("tool_response", result);
-                thread.events.push({
-                    "type": "tool_response",
-                    "data": result
-                });
-                continue;
-            default:
-                throw new Error(`Unknown intent: ${nextStep.intent}`);
+            case "subtract":
+            case "multiply":
+            case "divide":
+                thread = await handleNextStep(nextStep, thread);
         }
     }
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/03b-agent.ts src/agent.ts

</details>

测试减法：

    npx tsx src/index.ts 'can you subtract 3 from 4'

现在测试乘法工具：


    npx tsx src/index.ts 'can you multiply 3 and 4'

最后，测试一个包含多个操作的更复杂计算：


    npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

恭喜，你已经迈出了手写 agent loop 的第一步。

从这里开始，我们会逐步加入一些更中级和更高级的 12-factor agents 概念。
