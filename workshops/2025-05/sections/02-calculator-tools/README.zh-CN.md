# 第 2 章：添加计算器工具

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05/sections/02-calculator-tools/README.md`，基准提交：`58b8f09`

给我们的 agent 添加一些计算器工具。

先为计算器添加工具定义。

这些只是简单的结构化输出，我们会要求模型在 agentic loop 中把它们作为“下一步”返回。


    cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml

<details>
<summary>显示文件</summary>

```rust
// ./walkthrough/02-tool_calculator.baml
type CalculatorTools = AddTool | SubtractTool | MultiplyTool | DivideTool


class AddTool {
    intent "add"
    a int | float
    b int | float
}

class SubtractTool {
    intent "subtract"
    a int | float
    b int | float
}

class MultiplyTool {
    intent "multiply"
    a int | float
    b int | float
}

class DivideTool {
    intent "divide"
    a int | float
    b int | float
}
```

</details>

现在更新 agent 的 `DetermineNextStep` 方法，把计算器工具暴露为潜在的下一步。


```diff
baml_src/agent.baml
 function DetermineNextStep(
     thread: string
-) -> DoneForNow {
+) -> CalculatorTools | DoneForNow {
     client "openai/gpt-4o"

```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/02-agent.baml baml_src/agent.baml

</details>

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
