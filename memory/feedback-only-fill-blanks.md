---
name: feedback-only-fill-blanks
description: 向主表同步外部数据时，只更新为空的字段，不覆盖已有数据
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

**规则**：向配饰主表同步外部数据（比如 3D 角色排期计划表的字段）时，**只填当前为空的字段**。

**已有的任务名/版本/状态等数据绝不覆盖**。

**Why:** 主表的现有数据是有人手动维护过的，可能已经比外部数据更新；粗暴覆盖会丢失更新。

**How to apply:** 写同步脚本 / API 调用时，必须先 read 检查目标字段是否为空，为空才 write。用类似 `if not existing: update()` 的守卫。
