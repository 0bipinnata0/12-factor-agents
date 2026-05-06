# 第 8 章：添加 API Endpoints

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05/sections/08-api-endpoints/README.md`，基准提交：`58b8f09`

添加一个 Express server，通过 HTTP 暴露 agent。

本节关闭 BAML 日志。如果你想看到更多细节，也可以选择打开。

    export BAML_LOG=off

安装 Express 和类型：

    npm install express && npm install --save-dev @types/express supertest

添加 server 实现：

    cp ./walkthrough/08-server.ts src/server.ts

<details>
<summary>显示文件</summary>

```ts
// ./walkthrough/08-server.ts
import express from 'express';
import { Thread, agentLoop } from '../src/agent';

const app = express();
app.use(express.json());
app.set('json spaces', 2);

// POST /thread - Start new thread
app.post('/thread', async (req, res) => {
    const thread = new Thread([{
        type: "user_input",
        data: req.body.message
    }]);
    const result = await agentLoop(thread);
    res.json(result);
});

// GET /thread/:id - Get thread status
app.get('/thread/:id', (req, res) => {
    // optional - add state
    res.status(404).json({ error: "Not implemented yet" });
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});

export { app };
```

</details>

启动 server：

    npx tsx src/server.ts

用 curl 测试（在另一个终端中）：

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you add 3 and 4"}'

你应该会得到 agent 的响应，其中包含 agentic trace，并以类似这样的消息结尾：


    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}
