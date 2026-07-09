---
name: three-way-collab-constitution
description: 天蓝 + GitHub Claude Code + 飞书 AI 三方职责宪法。2026-07-09 v0.2 前置章节升级
metadata: 
  node_type: memory
  type: reference
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

# 三方协作宪法(v0.2 · 2026-07-09 前置升级 · 原 v0.1 保留在后)

## 三方职责 · 强制生效 · 2026-07-09 立

### 天蓝(决策层)
- 拍板所有规则变动
- **业务 SOP 首次落笔**(在飞书云文档)
- 审批 AI 起草内容后才上线

### 飞书 AI(运营层)
- **业务 SOP 主导**(外发规范 + 内部业务流程)· 天蓝在飞书写,它整理 / push 到 GitHub 归档
- 定时任务 / 群消息监控 / base 读写 / IM 推送
- 天蓝原始输入的**第一手承接方**

### GitHub Claude(结构治理层 + 抽象规则层)
- **结构治理独占**: memory 分层 / MEMORY.md / .aiignore / 三方协作规则本身
- **抽象规则主导**: 元规则 / 方法论 / 打断链保护 / 跨场景抽象(如 sop-work-triage / feedback-doc-style)
- **收束 + 防冗余**: 天蓝发散想法整合成决策 · 飞书写得啰嗦时起精简版
- **不主导业务 SOP**(不定颜色、不定命名、不定挂点细节)

### 豆包(第 4 方 · 讨论助手)
- 陪天蓝想方案 · 提出讨论输入
- **不读 memory · 不操作文件 · 不参与落地**

## 边界原则

- 一件事**只有一个主导方** · 其他两方是辅助
- 业务遇到抽象/结构问题 → **升级到 GitHub Claude**
- 抽象规则遇到业务落地 → **升级到飞书 / 天蓝**
- 三方每周同步一次 · 有分歧走天蓝拍板

## 沉淀链路 · 2026-07-09 拍

台账/群消息 → SOP 的完整链路:

1. **飞书** = 原始素材搬运方 · 从群消息 / 台账中挑原文 · **不做抽象**
2. **中转** · 天蓝复制原文集发给 GitHub Claude
3. **天蓝 + GitHub Claude** = 抽象整合方 · 读原文 → 提炼跨条目共性 → 起 SOP 段落草稿
4. **天蓝** = 审批 · 决定沉淀去向(哪个 SOP)
5. **GitHub Claude** = 执行 push
6. **飞书** = 收到 push commit hash 后 · 台账里对应条目标"已沉淀"

**关键约束**: 抽象权在天蓝 + GitHub Claude · 飞书不写 SOP 草稿(避免推翻浪费算力)

## 沉淀出稿默认极简版 · 2026-07-09 立(v0.2 首次沉淀实测教训)

**实测**: 2026-07-09 11:36-11:55 首次跑 v0.2 沉淀链路 · v1(+12) → v3(+16) → v5(+5) · 中间的 v3 是过度操作

**规则**:
- **默认出极简版**(v5 风格) · 天蓝觉得少了再要展开
- **反面**: 出完整版 → 天蓝要简化 → 出中间版 → 天蓝要更简 → 3 版 20 分钟
- **正面**: 出极简版 → 天蓝要么 OK 要么加 1-2 行 → 1 版 5 分钟

**极简版判断标准**:
- SOP 只放动作 · 不放机制/故事/描述
- checklist 段和常见现象段不重复
- 加行前问"没这行行不行"

## 诚实报告 · 2026-07-09 立(原飞书运营层铁律 #A 升级)

**规则**: 任何 AI 报"成功"必附 tool_call 证据链 · 报"没做"必单行「没做 X · 卡在 __ · 需要你 __」

**Why**: 2026-07-08 飞书累积 53 次"老实说没做"但每次只报事实。天蓝直到今天才自己摸出根源(工作空间垃圾)。如果第 1 次就同时报卡点 + 需求,3 天前就能给权限。

**HARD RULES**:
- 说"成功"必有 tool_call 证据(读到的真值 · 不是 API 返回码)
- 说"没做"必单行三段(没做 X + 卡在 + 需要)
- 结论标【已读 / 推算 / 不确定 / 工具限制】
- 不自我表扬 · 不数第 N 次

## 写后必二次验证 · 2026-07-09 立(飞书 5 条假成功事故教训)

**规则**: 任何 AI 做写入操作(改文件 / 写 base / 建 record) 后, **1 个工具调用内必须再读一次真值**。

**Why**: 2026-07-09 12:00-12:15 飞书 5 条同步 3 次报"成功"实际 100% 假成功。API 返回 `ok:true` 不等于写入成功 · 命令格式错 / API 限流 / record_id 凭印象编 都会导致假成功。

**How**:
- 改文件 → 立即 Read 或 grep 验证
- 写 base → 立即 record-search / record-get 验证
- 建 record → 立即 record-list 数一次
- **不做二次验证 = 假成功 · 必违反**

**HARD RULE**: 报"成功"必须附**二次验证的证据**(读到的真值),不能只报 API 返回码

**子规则 · schedule/trigger 立完必读 desc + cron 一致**(2026-07-09 14:00 立)
- 立 schedule/trigger 后 · 必读描述文字 + cron 表达式 · 对比一致才算完
- 起源: TODO 同步 trigger 立时 desc="每天 9:00" cron="03:00" 矛盾 · 直到查才发现

**子规则 · 读时也要真查**(2026-07-09 20:20 立 · 飞书 4 连假成功教训)
- 现有 #E 只管"写后" · 但"读时凭印象"也是假成功来源
- **读工具真值前** · 必真调用工具 · 不凭上次记忆/印象
- 触发场景:
  - 认为 reset 跑了 → 必读 reflog 验证
  - 认为字段对齐 → 必 record-get 读真值
  - 认为图在 X 群 → 必查 active_chat_id
  - 拼工具命令 → 命令首次用必 get_skills/--help
- **HARD RULE**: 只要说"某某是 X 值" · 附**真查的输出**(不是记忆里的值)

**子规则 · 接收方也要验证 · 不轻信报告**(2026-07-09 19:55 立)
- **AI 接收其他 AI 的成功报告 · 也要要证据链** · 不能只信"我 pull 成功了"
- 起源: 19:36 飞书报"reset+pull 成功·commit 1a28bf3" · GitHub Claude 未催 git log 输出就信了 · 19:55 才发现是假成功
- **要**: 接收方看到"成功"字眼 · 反问"read 后 git log/record-search 结果贴一下"
- 单方验证不够 · 双方交叉验证才靠谱

## 具体归属(2026-07-09 分类 · sop-active/ 现存 17 个)

**外发规范 → 飞书主导**(飞书云文档写 → push GitHub 归档):
sop-color-check / sop-naming-conventions / sop-file-submission-tips / sop-earring-fit / sop-attachment-point-workflow / sop-file-transfer-check / sop-feedback-with-visuals

**抽象方法论 → GitHub 主导**(我起草 · 天蓝审):
sop-work-triage / sop-communication-priority / sop-daily-checklist-by-area / sop-work-time-tracking / sop-weekly-meetings / sop-outsource-triage

**共同 4 项 · 待逐件拍归属**(2026-07-09 讨论中):
sop-attachment-env-constraint / sop-art-outsource-oa-flow / sop-external-task-assignment / sop-piece-check-list-needed

**2026-07-09 拍完**:
- 飞书主导 · sop-attachment-env-constraint / sop-art-outsource-oa-flow / sop-piece-check-list-needed
- GitHub 主导 · sop-external-task-assignment(抽象方法论)

---

# 三方协作宪法(v0.1 保留 · 试运营版 · 2026-07-07)

> **2026-07-08 补丁**:副业(lycheeGame 生态)已剥离到 private repo `f9-memory-side`,不再进 AI 共享。本文件下述"副业+主业交叉""副业相关设计文档""副业相关"三处**已失效**。

**参与者**:
- **天蓝** —— 决策层,真人做事
- **我**(GitHub Claude Code + `f9-memory` 仓库) —— 战略层(v0.2 已更精确为"结构+抽象层")
- **飞书 AI**(所属公司 aily 平台) —— 运营层
- **豆包**(第 4 方 · 2026-07-08 补) —— 讨论助手 · 只陪天蓝想方案 · **不读 memory · 不操作文件 · 不参与落地**

**试运营开始**:2026-07-07

## 三层分工

| 层 | 主责 |
|---|---|
| **战略层（我）** | 跨会话跨线沉淀 · 整合 · 写 SOP · 长思考 · 副业+主业交叉 |
| **运营层（飞书 AI）** | 定时任务 · 监听群 · 写 base · 记录带时间戳事件 |
| **决策层（天蓝）** | 拍板 · 优先级 · 真人做事 |

## 数据归属规则

**属于飞书的数据**（不进 GitHub）：
- 日期时间戳事件（`2026-06-27 14:24 立不要骗我`）
- trigger 运行状态 / 空转次数
- 群消息日志
- V9 base 的具体记录（每条配饰的当前 status）
- 外派每日提交量、返工率等运营指标

**属于我的数据**（GitHub `f9-memory` 仓库）：
- 跨会话的 SOP
- 抽象原则 / 方法论
- 副业相关设计文档（lycheeGame / midnight-store / 案卷九章 / 求合体）
- 人员对接**方法论**（feedback-wang-zhichao 里的"颜色偏红是常见坑"这类经验）
- feedback（偏好 / 规则）

**两边都存但角色不同**：
- **人员名单**：飞书有 open_id / 出勤 / 提交量；我有对接方式 / 特点 / 教训
- **项目状态**：飞书有实时的每条 status；我有战略级的"某个类目当前活跃项目"

## 数据流

```
飞书 AI 每天运营                   我每次会话
   ↓ 攒具体事件                       ↓ 攒战略洞察
   ↓ 定期汇总                         ↓ 写成 markdown
   ↓ 通过天蓝复制 or clone            ↓ push 到 GitHub
        ↓                                 ↓
        └────────── GitHub 中心 ─────────┘
                       ↓
              天蓝在两边都能查
```

**协作节奏**：
- 飞书日常运营 → 不需要天蓝介入
- 飞书遇到重大事件 → 主动汇报天蓝（如今天 trigger 减负）
- 天蓝把重要洞察带来我这（复制粘贴 or 让飞书 push GitHub）
- 我沉淀 + 整理 → push GitHub
- 飞书**每天 fetch 一次 GitHub**（已 clone 到 `~/.aily/workspace/_external/f9-memory/`）保持同步

## 边界规则

- **飞书不改** GitHub 的战略文件（feedback / SOP / 副业相关）—— 它只读
- **我不改** 飞书的 trigger 状态 / 运营日志 —— 它是它的

## 冲突仲裁

- **涉及事实数据**（某配饰状态 / 某人是否在线 / 某群 chat_id）→ **以飞书为准**
- **涉及方法论**（怎么校色 / 外派分级原则 / 战略优先级）→ **以 GitHub 为准**

## 试运营指标（1 周后评）

**2026-07-14 review**：
1. 数据是否重复 / 冲突？（好：无 or 罕见；差：经常撞）
2. 天蓝是否觉得"要在两边跑来跑去"？（好：不觉；差：累）
3. 有没有"应该在 X 但在 Y"的错位？（举例记录）
4. 飞书的 trigger 减负是否稳定？（不是本框架直接问题，但影响体感）

## 演进方向（先不做）

- 自动化 sync（Feishu API 直接读写 GitHub）—— 目前靠 clone + 天蓝复制，足够
- 飞书 push 到 GitHub —— 需要飞书 AI 有 git 写权限，现阶段不给
- 双向实时同步 —— 太重，先不搞

## 引用

- 起源：2026-07-07 天蓝 vs 我 vs 飞书 AI 三方对话，从"飞书 AI 一直被打断"话题延展出
- 关联：[feishu-ai-trigger-reduction-2026-07-07]（飞书运营层减负）+ [voice-input-tool]（信息传输通道）
