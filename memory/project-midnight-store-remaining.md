---
name: project-midnight-store-remaining
description: 《凌晨两点的便利店》剩余待优化清单 — 店员线已成型作参考实现，其他 POV 沿用同一套骨架
metadata: 
  node_type: memory
  type: project
  originSessionId: e409c323-78d4-4477-9d9e-4d21450e9bae
---

**当前状态（2026-07-05）**：店员线所有基础设施都已成型，可作为"参考实现"。

## 剩余大件

### ① 手机播放器 overlay UI（进行中）
- 触发：点手机 hotspot → beat 0
- 内容：半透明背景 + 中央大手机卡片 + 极简播放器 UI（`·现在播放·` + 抽象封面 + 假进度条 + ⏮⏸⏭ 按钮）+ 手机下方浮 3 个选项胶囊
- **约 25-30 min**
- **注意**：绝对不能有真实歌名/艺人名/专辑封面/品牌 UI，全部用原创抽象元素

### ② 护士 arc 内容对齐
- 现有 5 段 beats 是 v1 遗留，要按新场景 + 现代店重写
- 加她的 hotspots 位置 + world flag `w.nurseArrived`
- 音频层：她的 ambient（可能不需要耳机歌，只有环境声）
- **约 60-90 min**

### ③ 司机 arc 内容对齐
- ~3-4 段 beats：靠窗落座 / 借笔 / 买水 / 遇少年
- 按新窗边座位场景重写
- 司机 ambient（重音 + 腰痛闷哼 SFX）
- **约 45-60 min**

### ④ 少年 arc 内容对齐
- ~4 段 beats：喂猫 / 买新猫粮 / 隔窗指画本 / 回台阶发现桂花糕
- 少年 hotspots 主要在门口台阶卡
- 少年 ambient（手机震动、街上车声）
- **约 45-60 min**

## 中件

### ⑤ 画面 vs 顶视图巡检 + 校准剩余 6 张场景
- 每张对照顶视图审 3-5 min，约 2-3 张需重出
- **约 30-45 min**

### ⑥ 结局自适应
- `buildEndingLines(state.world)` 已存在但需按新 world flags 重写
- 8 件"藏在...里"的小事按玩家实际选择呈现
- **约 30-45 min**

### ⑦ 图鉴自适应（可选，最大工程）
- 15 个 CP 每个 2-3 个文本变体
- **约 90-120 min**
- 如玩家不看图鉴可砍掉

## 小件

- **⑧ 老人 NPC 存在感细化** — 15 min
- **⑨ 其他角色专属音乐生成 + 接入** — 每人 15 min（不含 Suno 生成时间）
- **⑩ Bug 巡检 + 边界情况**（存档、切角色、静音、mobile 适配） — 30-60 min

## 总量估算

- **核心 4 大件（①-④）**：3-4 小时
- **中件 3 个（⑤-⑦）**：2.5-3.5 小时
- **小件（⑧-⑩）**：1-2 小时
- **合计**：约 6-9 小时对话+迭代时间

## 迁移 checklist（做其他 POV 时的顺序）

1. **BEATS[pov] 数据**：写 beats 数组（title / hs / text / choices / effects）
2. **HOTSPOTS 位置**：每个 beat 的 hs 要在某张 card 的 HOTSPOTS 里有坐标（对齐 AI 图上物件的实际位置）
3. **到场 flag**：在能触发此角色进场的 beats 里加 `effects: (w) => { w.<char>Arrived = true }`
4. **isCharPresent()**：加对应 flag 的 case
5. **音频层**：写 `start<Char>Ambient()`，可加耳机音乐 / 场景音 / SFX，统一走 AUDIO 主线
6. **图片**：如需专属视角图，按顶视图对齐，注明 NO 品牌，按现代便利店风格生成
