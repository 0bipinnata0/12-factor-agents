[← 返回 README](../../README.zh-CN.md)

### 11. 从任何地方触发，在用户所在之处服务用户

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`content/factor-11-trigger-from-anywhere.md`，基准提交：`d20c728`

如果你一直在等 [humanlayer](https://humanlayer.dev) 的 pitch，那么你等到了。如果你已经在做 [factor 6：用简单 API 启动 / 暂停 / 恢复](./factor-06-launch-pause-resume.md) 和 [factor 7：用工具调用联系人类](./factor-07-contact-humans-with-tools.md)，那你已经准备好纳入这个 factor。

![从任何地方触发](https://github.com/humanlayer/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)

让用户可以从 Slack、email、SMS 或他们想要的任何其他渠道触发 Agent。也让 Agent 能通过同样的渠道响应。

好处：

- **在用户所在之处服务用户**：这能帮你构建感觉像真实人类，或者至少像数字同事的 AI 应用
- **Outer Loop Agents**：让 Agent 可以由非人类触发，例如事件、cron、故障告警或其他东西。它们可能工作 5 分钟、20 分钟、90 分钟，但当到达关键点时，可以联系人类寻求帮助、反馈或批准
- **高风险工具**：如果你能快速把不同人类拉进循环，就可以给 Agent 访问更高风险操作的权限，比如发送外部邮件、更新生产数据等。保持清晰标准，可以让那些[能做更大更好事情](./factor-10-small-focused-agents.md#如果-llm-变得更聪明呢)的 Agent 具备可审计性和可信度

[← 小而专注的 Agent](./factor-10-small-focused-agents.md) | [无状态 Reducer →](./factor-12-stateless-reducer.md)
