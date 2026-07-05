# F9 记忆本

**F9 语音助手的记忆本**（跨平台在线镜像）。

## 什么是这里
- `f9-persona.md` — F9 的行为规则（编辑这里能改 F9 的回答风格 / 快捷指令等）
- `memory/` — 主 memory 文件（跟本地 `~/.claude/projects/C--Users-tianlan/memory/` 保持同步）

## 谁能看
理论上任何人（这是 public repo）；实际上只在我不到处贴链接的情况下可及。

## 怎么用
- **F9 沉淀新东西** → 电脑上 F9 会自动 `commit + push` 到这里
- **手机上编辑** → GitHub App 或网页版都能直接改 memory/*.md，改完 F9 下次启动会 `pull` 拉回来
- **给别的 AI（ChatGPT/Kimi/Claude）扩充上下文** → 直接把这个仓库的 URL 贴给它，它能 fetch 读

## 结构约定
主 memory 遵循 CLAUDE.md 里的 auto memory 规则：
- `feedback-*.md` — 用户偏好/反馈
- `project-*.md` — 项目信息
- `sop-*.md` — 操作流程
- `user-*.md` — 用户身份/环境
- `MEMORY.md` — 索引（每条一行）
