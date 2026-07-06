---
name: project-accessory-pipeline
description: 配饰数据流水线：三表联动 + 挂点镜像 + 截图归档 + 排期同步 — 整套数据基础设施
metadata: 
  node_type: memory
  type: project
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

## 三表体系（核心数据源）

| 表 | 条目数 | 干啥 |
|---|---|---|
| **配饰主表** | 763 条 | 每件配饰的完整信息（编号/负责人/状态/版本/需求等） |
| **TL 静态模型进度** | 607 条 | 每件配饰在 TL 阶段的进度镜像 |
| **配饰挂点制作表** | 702 条 | 每件配饰的挂点数据镜像 |

**版本覆盖**：obt-240711 至 obt-261230

## 关键原则

**挂点表 和 TL 表 是配饰主表的 1:1 镜像**：
- ❌ **不应通过 API 直接新建**这两个表的记录
- ✅ **由工作流自动触发镜像创建**
- 曾经踩坑：误用 API 插入了 60 条挂点表 + 158 条 TL 表 → 已清理

## 自动化模块

### 1. 排期同步（3D 角色排期计划 → 配饰主表）
- **源**：飞书 Sheet "3D 角色排期计划"
- **目标字段**：DDL、跟进组、PL 捏脸、需 BS、需 DB、特效备注 等
- **规则**：**只填主表当前为空的字段**（见 feedback-only-fill-blanks），已有数据不覆盖

### 2. 外包审核截图归档（X3 群 → Base 表）
- **监控**：X3-角色资产效果核对群 里 **至尊菜刀** 的回复
- **触发**：他说"可以了"并 @ 天蓝
- **动作**：将该配饰**编号最新版本的截图**写入 TL 表"模型截图"字段
- **版本优先级**：V04 → V03 → V02 → V01 → 模型截图（见 sop-file-submission-tips）
- **限制**：飞书 API 不能直接写附件字段 → 需要**人工完成上传**（有定时提醒机制，见 feedback-timer-reminder）
- **Base 表**：https://papergames.feishu.cn/base/FI1Wb1Ypbagi1oszRYjcsxQanYl

### 3. 挂点镜像
- 从配饰主表镜像到挂点制作表 + TL 静态模型进度
- 详情见 sop-attachment-point-workflow
