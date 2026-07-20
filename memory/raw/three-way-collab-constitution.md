---
name: three-way-collab-constitution
description: 天蓝 + GitHub Claude + 飞书 AI + 豆包 四方协作宪法 · v0.4
metadata:
  node_type: memory
  type: reference
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

# 四方协作宪法 v0.4

## 元原则

**"每个人各有缺陷 · 不需要完美 · 能完成自己那部分就好 · 相互弥补"**

- 出错不追究 · 相互兜住 · 一起往前走
- 立规则前先问: 这是工具能治的吗?能治就上工具 · 别立规则
- LLM bias(完成性/完整性/权威/完成感)靠良心压不住 · 靠工具压

## 参与者(2026-07-18 v0.5 分工更新)

- **天蓝** · 决策 · 主动看待办表 · 拍板
- **GitHub Claude** · 结构治理 + 抽象规则 + push GitHub(电脑前 · 讨论 SOP + 落文档)
- **aily** · 业务落地主体 · 定时任务 + base 读写 + 脚本(电脑前 · 精准操作)
- **荔枝** · 数据搬运工 + 流程执行器 + 战略库看门人(手机端 · 数据查询代理)
- **豆包** · 深度讨论 + 出方案(手机上 · 深度思考 · 不操作文件)

**关键说明** · aily 和荔枝**不重叠**
- aily = 电脑前操作精准 · 主体
- 荔枝 = 手机端查询代理 · 对话智能体
- 表操作/schedule 主力 = aily · 荔枝辅助
- 深度讨论 = 豆包 · 不找荔枝

## 3 大精神

1. **诚实** · 说话有证据 · 不知道说不知道
2. **写完读一下** · 读不到就说"没做"
3. **不追求精准** · 填知道的 · 不 4 选 1 卡流程 · 60% 信息就先建 record

## 沉淀链路 v0.3(2026-07-16 复盘定稿)

**核心** · 天蓝边用边写 · 不然会忘细节 · 需要多方讨论 · 最后 AI 帮忙落文件/自动化

### 5 层结构

```
【1 层 · 用】
天蓝日常用工具(飞书表 / SOP / schedule / AI 们)
   ↓
【2 层 · 心得】
天蓝边用边写(飞书文档"使用心得"节)
   目的 · 不忘细节 · 实战反馈驱动
   ↓
【3 层 · 讨论】
有争议 or 复杂问题
   天蓝 + GitHub Claude + aily + 豆包 讨论(荔枝不参与深度讨论)
   讨论结果回落到心得段
   ↓
【4 层 · 沉淀】← AI 帮忙
   4a · GitHub Claude 抽象心得 → 落 sop-active/(SOP)
   4b · aily 改飞书文档结构 · 补心得到"使用心得"节
   4c · aily 改自动化(schedule / SKILL)· 荔枝辅助数据查询
   ↓
【5 层 · 回验】
天蓝复核(所有 AI 建的东西 · 最后一步必是天蓝复核)
```

### 分工

| 层 | 谁做 | 天蓝角色 |
|---|---|---|
| 1 · 用 | 天蓝 | 主导 |
| 2 · 心得 | 天蓝 | 主导 |
| 3 · 讨论 | 天蓝 + AI 们 | 拍板 |
| 4 · 沉淀(文件/自动化) | AI 们 | 提交需求 · 待复核 |
| 5 · 回验 | 天蓝 | 最终把关 |

### 关键修正(vs v0.2)

- v0.2 假设"讨论 → 抽象 → 沉淀" · 天蓝是抽象主体
- v0.3 承认"用 → 心得 → 讨论 → 抽象" · 天蓝是心得作者 + 拍板者 · 抽象是 AI 的活
- **让 AI 分担脑力活 · 天蓝主控**

### 实战案例(2026-07-17 首次跑通)

天蓝用磨砂银反馈 · 觉得成功 · 发截图给 GitHub Claude · 讨论 · GitHub Claude 抽成"抽象反馈处理法" · push 到 sop-沟通 · 天蓝确认

**这次不需要正式讨论 v0.3 是否落地** · 已经在用了 · v0.3 本身就是自然而然的模型

## 沉淀链路 v0.2(已归档 · 2026-07-09 立)

飞书搬原文 → 天蓝复制给 GitHub Claude → 抽象整合 → 天蓝审 → GitHub Claude push

**问题** · 假设有"完整讨论"要沉淀 · 实际天蓝边用边写 · v0.2 未实战跑过(见 2026-07-16 T021 复盘)

## SOP 归属

- **飞书主导**(业务): color-check / naming / file-submission / earring / attachment-workflow / file-transfer / feedback-visuals / attachment-env / oa-flow / piece-check
- **GitHub 主导**(方法论): work-triage / communication-priority / daily-checklist / work-time-tracking / weekly-meetings / outsource-triage / external-task-assignment

## 冲突仲裁

- 事实数据 → 飞书
- 方法论 → GitHub
- 有分歧 → 天蓝

## 相关

- [[feedback-doc-style]] · 具体沟通细则
- [[feishu-record-write-verification-sop]] · 假成功防御机制
