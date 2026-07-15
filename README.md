# Requirement Ticket

需求工单页面通过标准 `iframe` 嵌入 AI 对话网站。点击表单底部的“AI对话”，在弹窗中完成沟通后点击“结束对话”（或对话窗口右上角关闭按钮），完整聊天记录会自动填入“详细描述”。

## 本地运行

安装依赖并启动工单网站：

```bash
npm install
npm run dev
```

默认访问地址为 `http://localhost:5173`。

## 配置 AI 对话地址

AI 对话网站需要单独运行或部署。复制环境变量示例文件。

macOS / Linux：

```bash
cp .env.example .env.local
```

Windows PowerShell：

```powershell
Copy-Item .env.example .env.local
```

然后在 `.env.local` 中填写 AI 对话网站地址：

```env
VITE_AI_CHAT_URL=http://localhost:3000
```

正式部署时请改为可公开访问的 HTTPS 地址，例如：

```env
VITE_AI_CHAT_URL=https://ai.example.com
```

修改环境变量后需要重新启动或重新构建本项目。
