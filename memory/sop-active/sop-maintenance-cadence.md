---
name: sop-maintenance-cadence
description: 3 层节奏(周/月/季)维护知识库 · 周三下班后触发 · 天蓝审批飞书出的候选
metadata:
  node_type: memory
  type: sop
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

## 3 层节奏

### 每周 · 增量沉淀
- **时间**: 每周三下班后(飞书 schedule 触发)
- **飞书做**: 扫过去 7 天原始素材(群消息 / 台账新增)· 出候选清单
- **天蓝审**: 5-10 min yes/no · 决定哪些沉淀成 SOP
- **触发条件**: 有候选就跑 · 无候选跳过

### 每月 · 归档判断 + 架构巡检
- **时间**: 每月第一个周三下班后
- **飞书做**: 扫全库 · 出候选归档清单(明确废弃 / 项目完成 / 长期未用)
- **天蓝审**: 10 min yes/no · 手动改状态为"归档"
- **AI 无自主删除权**
- **同步 3 项巡检**:
  - **分层合规** (GitHub Claude) · 扫 sop-active vs raw/growth · 无跨层写入
  - **目录归位** (GitHub Claude) · 扫文件位置 · 无误入的草稿/过期文件
  - **AI 读取抽检** (飞书 AI + 天蓝) · 随机测 1-2 个高频问题 · 验 AI 引用范围

### 每季度 · 大盘复盘
- **时间**: 每季度末最后一个周三下班后
- **动作**: 天蓝 + GitHub Claude 一起过 · 20-30 min
- **看**: 这 3 个月哪些 SOP 真在用 / 哪些没动 / 大方向要不要调

## 时间点

- 周三下班后 = 19:30 起(参考 [[feedback-fragmented-work]] 下班后无人打扰最高效)
- 飞书 schedule cron 未定 · 建 T018 时具体定

## 触发方式

飞书 schedule 生成候选 → 写文件到 workspace/artifacts/ → IM 推天蓝 1 行 → 天蓝主动读

## 相关

- [[three-way-collab-constitution]] · 沉淀链路
- [[feedback-fragmented-work]] · 下班后最高效
- T018 · 飞书建 schedule 落地
