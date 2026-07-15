# Requirement Ticket

需求工单页面通过标准 `iframe` 嵌入 AI 对话网站。点击表单底部的“AI对话”，在弹窗中完成沟通后点击“结束对话”（或对话窗口右上角关闭按钮），完整聊天记录会自动填入“详细描述”。

## 本地运行

先启动 AI 对话网站：

```bash
cd C:\Users\23814\Desktop\webapp-conversation
npm run dev
```

再启动工单网站：

```bash
cd C:\Users\23814\Desktop\requirement-ticket\requirement-ticket
npm run dev
```

默认 AI 对话地址为 `http://localhost:3000`。部署或端口变化时，复制 `.env.example` 为 `.env.local` 并修改 `VITE_AI_CHAT_URL`。
