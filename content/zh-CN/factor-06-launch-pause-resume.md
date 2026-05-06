[← 返回 README](../../README.zh-CN.md)

### 6. 用简单 API 启动 / 暂停 / 恢复

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-06-launch-pause-resume.md`，基准提交：`d20c728`

Agent 只是程序，而我们对程序的启动、查询、恢复和停止方式都有一些基本期待。

[![暂停恢复动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/165-pause-resume-animation.gif)](https://github.com/user-attachments/assets/feb1a425-cb96-4009-a133-8bd29480f21f)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/165-pause-resume-animation.gif">GIF 版本</a></summary>

![暂停恢复动画](https://github.com/humanlayer/12-factor-agents/blob/main/img/165-pause-resume-animation.gif)

</details>


用户、应用、pipeline 和其他 Agent 应该能通过简单 API 启动一个 Agent。

当需要长时间运行的操作时，Agent 以及编排它们的确定性代码应该能够暂停 Agent。

webhook 这类外部触发器，应该让 Agent 能够从离开的地方恢复，而不需要和 Agent 编排器做很深的集成。

这和 [factor 5：统一执行状态和业务状态](./factor-05-unify-execution-state.md) 以及 [factor 8：掌控你的控制流](./factor-08-own-your-control-flow.md) 密切相关，但也可以独立实现。



**注意**：AI 编排器通常会允许暂停和恢复，但不一定允许在“工具选择”和“工具执行”之间暂停和恢复。另见 [factor 7：用工具调用联系人类](./factor-07-contact-humans-with-tools.md) 和 [factor 11：从任何地方触发，在用户所在之处服务用户](./factor-11-trigger-from-anywhere.md)。

[← 统一执行状态](./factor-05-unify-execution-state.md) | [用工具联系人类 →](./factor-07-contact-humans-with-tools.md)
