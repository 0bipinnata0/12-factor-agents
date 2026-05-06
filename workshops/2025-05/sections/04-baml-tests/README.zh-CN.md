# 第 4 章：给 agent.baml 添加测试

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05/sections/04-baml-tests/README.md`，基准提交：`58b8f09`

给 BAML agent 添加一些测试。

开始时，保持 BAML 日志开启：

    export BAML_LOG=debug

接下来，给 agent 添加一些测试。

先从一个简单测试开始，检查 agent 处理基础计算的能力。


```diff
baml_src/agent.baml
     "#
   }
+
+test MathOperation {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+      {
+        "type": "user_input",
+        "data": "can you multiply 3 and 4?"
+      }
+    "#
+  }
+}
+
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/04-agent.baml baml_src/agent.baml

</details>

运行测试：

    npx baml-cli test

现在，用断言改进这个测试。

断言是确保 agent 按预期工作的好办法，也很容易扩展，用于检查更复杂的行为。


```diff
baml_src/agent.baml
     "#
   }
+  @@assert(hello, {{this.intent == "done_for_now"}})
 }

     "#
   }
+  @@assert(math_operation, {{this.intent == "multiply"}})
 }

```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/04b-agent.baml baml_src/agent.baml

</details>

运行测试：

    npx baml-cli test

随着测试变多，你可以关闭日志，让输出更干净。
在迭代某些具体测试时，也可以再把它们打开。


    export BAML_LOG=off

现在添加一些更复杂的测试用例，
模拟从一个正在进行中的 agentic 上下文窗口中间恢复。


```diff
baml_src/agent.baml
     "#
   }
-  @@assert(hello, {{this.intent == "done_for_now"}})
+  @@assert(intent, {{this.intent == "done_for_now"}})
 }

     "#
   }
-  @@assert(math_operation, {{this.intent == "multiply"}})
+  @@assert(intent, {{this.intent == "multiply"}})
 }

+test LongMath {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+      [
+        {
+          "type": "user_input",
+          "data": "can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result?"
+        },
+        {
+          "type": "tool_call",
+          "data": {
+            "intent": "multiply",
+            "a": 3,
+            "b": 4
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 12
+        },
+        {
+          "type": "tool_call",
+          "data": {
+            "intent": "divide",
+            "a": 12,
+            "b": 2
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 6
+        },
+        {
+          "type": "tool_call",
+          "data": {
+            "intent": "add",
+            "a": 6,
+            "b": 12
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 18
+        }
+      ]
+    "#
+  }
+  @@assert(intent, {{this.intent == "done_for_now"}})
+  @@assert(answer, {{"18" in this.message}})
+}
+
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/04c-agent.baml baml_src/agent.baml

</details>

试着运行它：


    npx baml-cli test
