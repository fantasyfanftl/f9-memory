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

参见 [[midnight-store-cashier-arch]] 第 17 节「UI 动画迭代方向」+ 第 20.5 节「沉浸感优化迭代方向」。合并后的完整 v1.1 backlog：

**听觉**：
- 偶发环境音：每 2-3 分钟随机触发一次远处车驶过 / 风吹落叶 / 夜宵摊收摊的模糊声响 · 音量极低
- 弱空间音效：门铃 / 猫叫（来自门口）做轻微左声道偏移 · 扫码 / 关东煮居中

**视觉**：
- 关键词微高亮（情绪核心词淡暖白 +10%）
- 文本-音效弱联动
- 场景内错落淡入（背景 → 大件 → 细节）· 总时长 ≤300ms
- 常驻元素呼吸微动效具体参数：关东煮热气 4s 周期 ±15% 透明度 · 街灯 5s ±8% 亮度 · 冷柜 6s ±5% 亮度
- 环境音随滑动渐变
- 时间流逝色温渐变：开局偏冷蓝 → 少年阶段微偏暖紫（暗示天快亮了，玩家只会感觉"好像天慢慢亮了一点"）
- 落地窗雾气纹理 + 偶发冷凝水滴
- 车灯光影每几分钟从左扫右

**交互**：
- 无缝加载：资源预加载 + 用滑动 / 淡入掩盖加载过程 · 永远不出现加载进度条 / 百分比
- 热区边缘 10px 外扩容错

**场景卡合并（M4 · 从 v1.0 移入）· 已被更大方案替代**：
- **⚠️ 2026-07-07 更新**：本项被更完整的场景全量重绘方案覆盖 → 详见 [[midnight-store-scene-redraw-plan]]（v1.1 场景重绘精简版 · 4 规则 + 3 冲突 + 5 步执行）
- 原方案（保留作历史参考）：合并 `register` + `dispenser` 两卡为单卡「收银台&热食区」· 用 gpt-image-2 skill v1.0.1 重绘一张合景全景图 · 15-30 min 生图 + 20 min 坐标迁移
- **新方案范围更大**：不只合并 2 卡，是基于统一视点重绘全部 5 张，用 Blender 3D 建模而非 AI 生图（AI 生图达不到跨图一致性 · 见 [[feedback-ai-gen-instability]]）
- 迁移原因：v1.0 用户列偏差清单里 M4 项本来要在 v1.0 修，但为保 v1.0 上线节奏移入 v1.1
- v1.0 现状：收银台与热食区**是同一物理空间的两个视角卡**，玩家左右滑动可切换查看

**约束**：都是纯 CSS + Web Audio 参数微调（合景重绘除外），不改任何状态机 / beat 数据结构。

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
- **v1.0 CP 系统完全删除**（CONVERGENCES / cpProgress / convergences / beat 的 cps 字段全砍）· v2.0 多 POV 上线时按新规范重新接入 · v1.0 只需要保证图鉴按篇章解锁即可（不依赖 CP）
- 图鉴 UI 保持"篇章式" · 未开放篇章显示"更多人物篇章 敬请期待" · 上新 POV 时加一整篇（`护士篇 · N / N`）而非塞进"店员篇"

---

## 明确不做（避免下次会话跑偏）

- **不做**"好坏结局分支"—— 结局永远是静帧收束，只是细节多少
- **不做**"隐藏成就 / 达成弹窗 / 收集进度条"—— 破坏克制气质
- **不做**"任务提示 / 引导箭头 / 新手教程"—— 完全靠观察和试错
- **不做**"角色数值化好感度"—— world flag 只做叙事标记，不做数值计算