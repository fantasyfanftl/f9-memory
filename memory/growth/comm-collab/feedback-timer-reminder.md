---
name: feedback-timer-reminder
description: "菜刀在群里说'可以了'但截图未归档时，需要定时提醒手动上传（当前API无法写附件字段）"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

> **2026-07-08 天蓝暂缓** · 需要时再启动 · 见 TODO.md 若要重建

**问题**：至尊菜刀在 X3-角色资产效果核对群 回复"可以了"@天蓝 = 该配饰通过，需要把截图归档到 Base 表。**但当前飞书 API 无法直接向附件字段写入图片**。

**How to apply:**
- 已启动的自动化：识别群里"可以了"消息 → 判断哪个配饰通过 → 触发提醒
- **具体操作还得人肉**：定时提醒天蓝手动完成截图上传到 Base 表对应字段
- 归档时按版本优先级取：**V04 → V03 → V02 → V01 → 模型截图**，取最新非空的（见 sop-file-submission-tips）
