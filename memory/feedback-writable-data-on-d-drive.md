---
name: feedback-writable-data-on-d-drive
description: 用户偏好：所有可写数据（历史、日志、缓存等）默认放 D 盘（或 E/F 盘），不要放 C 盘
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

用户明确要求：**所有可长期积累的可写数据文件（对话历史、日志、缓存、模型下载、临时数据等）默认存 D 盘**。C 盘空间紧张，D/E/F 盘各上 T 空间。

**Why:** C 盘剩余空间少（180GB 空闲，但装了系统 + 用户配置），D 盘 3TB 空闲、E 盘 1.3TB 空闲、F 盘 663GB 空闲。

**How to apply:**
- 新建"数据目录"时，路径用 `D:\<项目名>-data\`（比如 `D:\voice-input-data\`），不要用 `C:\Users\tianlan\<项目名>\` 或 `%APPDATA%`
- 已建好的可写目录：`D:\voice-input-data\`（存 assistant 对话历史 `assistant-history.json`）
- 代码本身、venv、配置文件可以留在 C 盘（这些不怎么长）
- 例外：Windows Startup / 注册表 Run 键这种系统机制没法搬，就放原位
