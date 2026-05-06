[← 返回 README](../../README.zh-CN.md)

### 1. 从自然语言到工具调用

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-01-natural-language-to-tool-calls.md`，基准提交：`d20c728`

Agent 构建中最常见的模式之一，是把自然语言转换成结构化的工具调用。这是一个很强的模式，因为它让你可以构建能够推理任务并执行任务的 Agent。

![从自然语言到工具调用](https://github.com/humanlayer/12-factor-agents/blob/main/img/110-natural-language-tool-calls.png)

当这个模式被原子化地应用时，它就是把这样一句话：

> 你能给 Terri 创建一个 750 美元的付款链接，用于赞助 2 月的 AI tinkerers meetup 吗？

转换成一个描述 Stripe API 调用的结构化对象：

```json
{
  "function": {
    "name": "create_payment_link",
    "parameters": {
      "amount": 750,
      "customer": "cust_128934ddasf9",
      "product": "prod_8675309",
      "price": "prc_09874329fds",
      "quantity": 1,
      "memo": "Hey Jeff - see below for the payment link for the february ai tinkerers meetup"
    }
  }
}
```

**注意**：现实中的 Stripe API 要复杂一些。一个[真正做这件事的 Agent](https://github.com/dexhorthy/mailcrew)（[视频](https://www.youtube.com/watch?v=f_cKnoPC_Oo)）会列出 customers、products、prices 等，使用正确的 id 构建这个 payload；或者把这些 id 放进 prompt / 上下文窗口里（后面我们会看到，这两件事其实有点像同一件事）。

然后，确定性代码就可以接过这个 payload 并做相应处理。（更多内容见 [factor 3](./factor-03-own-your-context-window.md)）

```python
# LLM 接收自然语言并返回一个结构化对象
nextStep = await llm.determineNextStep(
  """
  create a payment link for $750 to Jeff
  for sponsoring the february AI tinkerers meetup
  """
  )

# 根据结构化输出中的 function 处理它
if nextStep.function == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return  # 或者执行任何你想要的逻辑，见下文
elif nextStep.function == 'something_else':
    # ... 更多分支
    pass
else:  # 模型没有调用我们认识的工具
    # 做点别的
    pass
```

**注意**：完整的 Agent 会接收 API 调用结果，并继续带着它循环，最终返回类似这样的内容：

> 我已经成功为 Terri 创建了 750 美元的付款链接，用于赞助 2 月的 AI tinkerers meetup。链接是：https://buy.stripe.com/test_1234567890

**不过**，这里我们会先跳过那一步，把它留给另一个 factor。你可能也想把它纳入进来，也可能不想，这取决于你。

[← 我们是怎么走到这里的](./brief-history-of-software.md) | [掌控你的提示词 →](./factor-02-own-your-prompts.md)
