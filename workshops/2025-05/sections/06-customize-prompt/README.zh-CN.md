# 第 6 章：用推理步骤自定义提示词

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05/sections/06-customize-prompt/README.md`，基准提交：`58b8f09`

本节会探索如何用 reasoning steps 自定义 agent 的 prompt。

这是 [factor 2：掌控你的提示词](../../../../content/zh-CN/factor-02-own-your-prompts.md) 的核心。

AI That Works 上有一篇关于 reasoning 的深入文章：[reasoning models versus reasoning steps](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)。


本节保持 BAML 日志开启会比较有帮助。

    export BAML_LOG=debug

更新 agent prompt，加入一个 reasoning step。


```diff
baml_src/agent.baml

         {{ ctx.output_format }}
+
+        First, always plan out what to do next, for example:
+
+        - ...
+        - ...
+        - ...
+
+        {...} // schema
     "#
 }
   @@assert(b, {{this.a == 3}})
 }
-
-
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/06-agent.baml baml_src/agent.baml

</details>

生成更新后的 client：

    npx baml-cli generate

现在可以用一个简单 prompt 试一下：


    npx tsx src/index.ts 'can you multiply 3 and 4'

你应该能从 BAML 日志中看到展示 reasoning steps 的输出。

#### 可选挑战

在工具输出格式中添加一个字段，把 reasoning steps 包含到输出里。
