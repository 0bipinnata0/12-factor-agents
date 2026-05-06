# 第 9 章：内存状态和异步澄清

> 中文翻译：由 0bipinnata0 翻译 / 改编自原项目对应英文页面。内容与图片遵循 CC BY-SA 4.0；代码遵循 Apache 2.0。
> 原文：`workshops/2025-05/sections/09-state-management/README.md`，基准提交：`58b8f09`

添加状态管理和异步澄清支持。

本节关闭 BAML 日志。如果你想看到更多细节，也可以选择打开。

    export BAML_LOG=off

为 threads 添加简单的内存状态管理：

    cp ./walkthrough/09-state.ts src/state.ts

<details>
<summary>显示文件</summary>

```ts
// ./walkthrough/09-state.ts
import crypto from 'crypto';
import { Thread } from '../src/agent';


// you can replace this with any simple state management,
// e.g. redis, sqlite, postgres, etc
export class ThreadStore {
    private threads: Map<string, Thread> = new Map();

    create(thread: Thread): string {
        const id = crypto.randomUUID();
        this.threads.set(id, thread);
        return id;
    }

    get(id: string): Thread | undefined {
        return this.threads.get(id);
    }

    update(id: string, thread: Thread): void {
        this.threads.set(id, thread);
    }
}
```

</details>

更新 server，让它使用状态管理。

* 使用 `ThreadStore` 添加 thread 状态管理
* 从 `/thread` endpoint 返回 thread ID 和 response URL
* 实现 `GET /thread/:id`
* 实现 `POST /thread/:id/response`


```diff
src/server.ts
 import express from 'express';
 import { Thread, agentLoop } from '../src/agent';
+import { ThreadStore } from '../src/state';

 const app = express();
 app.set('json spaces', 2);

+const store = new ThreadStore();
+
 // POST /thread - Start new thread
 app.post('/thread', async (req, res) => {
         data: req.body.message
     }]);
-    const result = await agentLoop(thread);
-    res.json(result);
+
+    const threadId = store.create(thread);
+    const newThread = await agentLoop(thread);
+
+    store.update(threadId, newThread);
+
+    const lastEvent = newThread.events[newThread.events.length - 1];
+    // If we exited the loop, include the response URL so the client can
+    // push a new message onto the thread
+    lastEvent.data.response_url = `/thread/${threadId}/response`;
+
+    console.log("returning last event from endpoint", lastEvent);
+
+    res.json({
+        thread_id: threadId,
+        ...newThread
+    });
 });

 app.get('/thread/:id', (req, res) => {
-    // optional - add state
-    res.status(404).json({ error: "Not implemented yet" });
+    const thread = store.get(req.params.id);
+    if (!thread) {
+        return res.status(404).json({ error: "Thread not found" });
+    }
+    res.json(thread);
 });

+// POST /thread/:id/response - Handle clarification response
+app.post('/thread/:id/response', async (req, res) => {
+    let thread = store.get(req.params.id);
+    if (!thread) {
+        return res.status(404).json({ error: "Thread not found" });
+    }
+
+    thread.events.push({
+        type: "human_response",
+        data: req.body.message
+    });
+
+    // loop until stop event
+    const newThread = await agentLoop(thread);
+
+    store.update(req.params.id, newThread);
+
+    const lastEvent = newThread.events[newThread.events.length - 1];
+    lastEvent.data.response_url = `/thread/${req.params.id}/response`;
+
+    console.log("returning last event from endpoint", lastEvent);
+
+    res.json(newThread);
+});
+
 const port = process.env.PORT || 3000;
 app.listen(port, () => {
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/09-server.ts src/server.ts

</details>

启动 server：

    npx tsx src/server.ts

测试澄清流程：

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you multiply 3 and xyz"}'
