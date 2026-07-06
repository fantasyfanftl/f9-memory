---
name: midnight-store-cashier-future
description: 《凌晨两点的便利店》v1.0 之后的扩展方向 backlog · 只列架构方向和预留规则，不写具体剧本
metadata: 
  node_type: memory
  type: project
  originSessionId: 23b2404d-32c8-4e64-b53a-f3713ff7b936
---

**用途**：v1.0 上线后接着做的东西，都记这里。**只列架构方向和预留规则，不写具体剧本内容**——避免占用核心记忆容量，剧本等做到那一步再写。

**前置**：v1.0 已按 [[midnight-store-cashier-canon]] + [[midnight-store-cashier-arch]] + [[midnight-store-cashier-acceptance]] 三份正典落地，17 项验收全过。

---

## v1.1 · 氛围细化（不动架构）

参见 [[midnight-store-cashier-arch]] 第 17 节「UI 动画迭代方向」的 5 条：
- 关键词微高亮（情绪核心词淡暖白 +10%）
- 文本-音效弱联动
- 场景内错落淡入（背景 → 大件 → 细节）
- 常驻元素呼吸微动效（关东煮热气 / 路灯光晕 / 冷柜冷光）
- 环境音随滑动渐变

**约束**：都是纯 CSS + Web Audio 参数微调，不改任何状态机 / beat 数据结构。

---

## v1.2 · 多周目文本变体库（只加文本不加机制）

- 同一个 anchor beat 在二周目 / 三周目起替换细微不同的台词 + 动作细节
- 触发方式：`state.world.playthroughCount` 累积，`resolveBranch` 里加 `whenPlaythrough >= 2` 分支
- **约束**：不加新的选择 / 不改分档效果 / 不改结局。只是"同一件事换个说法"提升重玩新鲜感

---

## v2.0 · 架构级扩展（跟 v1.0 线性架构解耦，独立演进）

### 2.1 多客同店支持
- CASHIER_TIMELINE 从 "一 step 一客人" 重构为 "一 step 可含 guest queue"
- `state.world.cashierArc.guestPresent` 从单值改为数组
- render 层支持同时画多个 guest silhouette
- 「护士帮老人拿牛奶」交叉彩蛋在这里开放（护士 step 内老人 zone 若为 'shelves' 且 milkHelp === 'none' 时触发）

### 2.2 三 POV 内容框架（架构层，不含具体剧本）
- 护士 / 司机 / 少年 各自的 arc 用**同一套 CASHIER_TIMELINE 模式**：`<POV>Arc.step / guestPresent / guestZone / playedShards / playedAnchors`
- 各自的 timeline 长度、结账数、shard 数不必等于 cashier
- **状态 flag 命名约束**：护士 flag 前缀 `nurse*`（跟已有 `nursePay` 一致）· 司机 `driver*` · 少年 `teen*`
- render / trigger 层分流：`state.currentPOV` switch 到对应 arc 状态机

### 2.3 跨视角彩蛋预埋规则（伏笔约束）
- 埋在店员 arc 里的物件（例：老人那把硬币里有一枚旧的一元 · 护士包里露出半张医院缴费单）**必须**允许"其他 POV 不玩也完全无感知"
- 埋法：在店员的 beat text 里作为**一次性可见的场景细节**，不参与任何 CP / flag / 图鉴
- 其他 POV 上线后：对应 POV 的 beat 里"picks up"这个物件（例：护士 arc 里角色摸到缴费单 → 回忆闪回 → 玩过店员 arc 的老玩家瞬间连上）
- **绝对禁止**：伏笔物件影响 v1.0 的任何机制 / 分档 / 结局判定

---

## 通用预留规则（v1.0 就必须遵守）

- `state.world` flag 命名保持 camelCase · 见 [[midnight-store-cashier-arch]] 命名对照表
- 存档 `v` 号严格单调递增：v3 → v4（v2.0 多客同店重构时升）→ v5（其他 POV 上线时升）· 每次升版都在 loadState 里加迁移分支或直接拒绝旧版
- CP 15 条定义（CONVERGENCES 表）暂保留，v1.0 只解锁 CP1-CP9 里店员参与的部分 · v2.0+ 其他 POV 上线后其余 CP 才能达成
- 图鉴 UI 保持"篇章式" · 未开放篇章显示"更多人物篇章 敬请期待" · 上新 POV 时加一整篇（`护士篇 · N / N`）而非塞进"店员篇"

---

## 明确不做（避免下次会话跑偏）

- **不做**"好坏结局分支"—— 结局永远是静帧收束，只是细节多少
- **不做**"隐藏成就 / 达成弹窗 / 收集进度条"—— 破坏克制气质
- **不做**"任务提示 / 引导箭头 / 新手教程"—— 完全靠观察和试错
- **不做**"角色数值化好感度"—— world flag 只做叙事标记，不做数值计算