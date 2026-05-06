# 第 5 章：多种人类工具

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05/sections/05-human-tools/README.md`，基准提交：`58b8f09`

本节会添加对多个“联系人类”工具的支持。


本节关闭 BAML 日志。如果你想看到更多细节，也可以选择打开。

    export BAML_LOG=off

首先，添加一个可以向人类请求澄清的工具。

它会不同于 `done_for_now` 工具，
并且可以在 agent 中更灵活地处理不同类型的人类交互。


```diff
baml_src/agent.baml
+// human tools are async requests to a human
+type HumanTools = ClarificationRequest | DoneForNow
+
+class ClarificationRequest {
+  intent "request_more_information" @description("you can request more information from me")
+  message string
+}
+
 class DoneForNow {
   intent "done_for_now"
-  message string
+
+  message string @description(#"
+    message to send to the user about the work that was done.
+  "#)
 }

 function DetermineNextStep(
     thread: string
-) -> CalculatorTools | DoneForNow {
+) -> HumanTools | CalculatorTools {
     client "openai/gpt-4o"

 }

+
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/05-agent.baml baml_src/agent.baml

</details>

接下来，重新生成 client 代码。

注意：如果你正在使用 BAML 的 VSCode 扩展，
当你在编辑器里保存文件时，client 会自动重新生成。


    npx baml-cli generate

现在更新 agent，让它使用这个新工具。


```diff
src/agent.ts
 }

-export async function agentLoop(thread: Thread): Promise<string> {
+export async function agentLoop(thread: Thread): Promise<Thread> {

     while (true) {
         switch (nextStep.intent) {
             case "done_for_now":
-                // response to human, return the next step object
-                return nextStep.message;
+            case "request_more_information":
+                // response to human, return the thread
+                return thread;
             case "add":
             case "subtract":
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/05-agent.ts src/agent.ts

</details>

接下来更新 CLI，让它通过命令行向用户请求输入，从而处理澄清请求。


```diff
src/cli.ts
 // cli.ts lets you invoke the agent loop from the command line

-import { agentLoop, Thread, Event } from "./agent";
+import { agentLoop, Thread, Event } from "../src/agent";

+
+
 export async function cli() {
     // Get command line arguments, skipping the first two (node and script name)
     // Run the agent loop with the thread
     const result = await agentLoop(thread);
-    console.log(result);
+    let lastEvent = result.events.slice(-1)[0];
+
+    while (lastEvent.data.intent === "request_more_information") {
+        const message = await askHuman(lastEvent.data.message);
+        thread.events.push({ type: "human_response", data: message });
+        const result = await agentLoop(thread);
+        lastEvent = result.events.slice(-1)[0];
+    }
+
+    // print the final result
+    // optional - you could loop here too
+    console.log(lastEvent.data.message);
+    process.exit(0);
 }
+
+async function askHuman(message: string) {
+    const readline = require('readline').createInterface({
+        input: process.stdin,
+        output: process.stdout
+    });
+
+    return new Promise((resolve) => {
+        readline.question(`${message}\n> `, (answer: string) => {
+            resolve(answer);
+        });
+    });
+}
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/05-cli.ts src/cli.ts

</details>

试一下：


    npx tsx src/index.ts 'can you multiply 3 and FD*(#F&& '

接下来添加一个测试，检查 agent 处理澄清请求的能力。


```diff
baml_src/agent.baml


+
+test MathOperationWithClarification {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+          [{"type":"user_input","data":"can you multiply 3 and feee9ff10"}]
+      "#
+  }
+  @@assert(intent, {{this.intent == "request_more_information"}})
+}
+
+test MathOperationPostClarification {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+        [
+        {"type":"user_input","data":"can you multiply 3 and FD*(#F&& ?"},
+        {"type":"tool_call","data":{"intent":"request_more_information","message":"It seems like there was a typo or mistake in your request. Could you please clarify or provide the correct numbers you would like to multiply?"}},
+        {"type":"human_response","data":"lets try 12 instead"},
+      ]
+      "#
+  }
+  @@assert(intent, {{this.intent == "multiply"}})
+  @@assert(a, {{this.b == 12}})
+  @@assert(b, {{this.a == 3}})
+}
+
+
+
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/05b-agent.baml baml_src/agent.baml

</details>

现在可以再次运行测试：


    npx baml-cli test

你会注意到新测试通过了，但 hello world 测试失败了。

这是因为 agent 的默认行为是返回 `done_for_now`。


```diff
baml_src/agent.baml
     "#
   }
-  @@assert(intent, {{this.intent == "done_for_now"}})
+  @@assert(intent, {{this.intent == "request_more_information"}})
 }

```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/05c-agent.baml baml_src/agent.baml

</details>

验证测试通过：

    npx baml-cli test
