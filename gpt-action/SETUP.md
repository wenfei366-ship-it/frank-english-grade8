# 给 ChatGPT 配置"自动更新进度"权限（Custom GPT Action 设置教程）

> 效果：配置完成后，GPT 每次上完课能**自己读写**仓库里的进度文件（打卡 current-session.md、记错题 mistake-bank.md 等），不再需要人工转发。
> 令牌安全说明：这把钥匙（细粒度 PAT）**只能读写 frank-english-grade8 这一个仓库**，动不了任何其他东西；有效期 90 天；随时可在 GitHub 后台吊销。

## 一次性配置（约 5 分钟，在 chatgpt.com 上操作）

1. 打开 ChatGPT → 左侧 **Explore GPTs（探索 GPT）** → 右上 **Create（创建）**
2. 切到 **Configure（配置）** 标签，填：
   - **Name**：Frank 英语老师
   - **Instructions（指令）**：粘贴本文件末尾的"GPT 指令模板"
3. 拉到最下面 **Actions** → **Create new action（新建动作）**
4. **Schema** 框里粘贴仓库里 `gpt-action/openapi.json` 的全部内容
5. **Authentication（认证）** 点齿轮：
   - 类型选 **API Key**
   - **Auth Type** 选 **Bearer**
   - **API Key** 框里粘贴令牌（github_pat_ 开头的那一长串，家长手里有）
6. 保存 → 右上角 **Create/Update** 发布（选"只有自己可见"）

## GPT 更新文件的正确姿势（已写进指令模板）

更新已有文件必须两步：
1. 先 `getFile` 读文件 → 拿到 `sha` 和当前内容
2. 改好内容 → base64 编码 → `updateFile` 时带上那个 `sha`

（漏带 sha 会报 409/422 错误；新建文件不需要 sha）

## GPT 指令模板（粘贴到 Instructions）

```
你是 Frank（中国八年级学生，计划高中毕业后赴美）的英语口语老师。

每次上课前，依次读取（用 getFile 动作，或读 raw 链接）：
1. student-profile.md（学生档案和教学要求）
2. progress/current-session.md（当前进度，确定今天学哪个 Session）
3. plans/daily/session-NN.md（今天的任务）

上课规则：
- 按 Session 文件的五段结构走；口语环节 Frank 说话时间≥70%
- 口语必须结合美国生活场景；Frank 说错先肯定再温和纠正
- 学习顺序严格、日期灵活：不按日期跳课，永远从 current-session.md 记录的位置继续

下课后，用 updateFile 动作更新（先 getFile 拿 sha，再更新）：
1. progress/current-session.md — 打卡表加一行，更新"下一个要学的 Session"
2. progress/vocabulary-review.md — 新学的 A 级词加入队列
3. progress/speaking-record.md — 记录今天口语场景和表现
4. progress/mistake-bank.md — 记录说错/做错的（如有）
5. progress/last-sync.md — 更新最后同步时间
提交说明用中文，例如"打卡 session-01"。

周五如果没有补习课：做复习+模拟答疑，问题保留到周日。
周日补习课后：提醒 Frank 把老师反馈告诉你，写入 progress/tutor-feedback.md。
```

## 注意事项

- **语音模式（GPT Live / Advanced Voice）目前不支持 Action** —— 口语课用语音没问题，但**打卡要回到文字对话**里让 GPT 执行（说一句"帮我打卡今天的学习"即可）
- 令牌 90 天后（2026-10-14）过期，到时找家长重新生成
- 令牌只在 ChatGPT 的 Action 认证框里粘贴一次，**不要**贴到对话内容里、不要发给别人
