---
name: midnight-store-selfcheck
description: 《凌晨两点的便利店》改任何东西之前的自检 checklist——治「概念对但生成的东西错」这个病
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 23b2404d-32c8-4e64-b53a-f3713ff7b936
---

**痛点由来（2026-07-06）**：用户明确反馈——"我们很多东西，概念是对的，但是我们生成出来的东西是错的。而我们没有一个自检的过程，没有一个复检的过程。甚至说我让你去改，你可能都不知道要改什么。"

例证：
- 场景图生成时**没有对照顶视图**，导致 hotspot 坐标和图里物件位置对不上
- 剧本文本改动时**没有对照 canon**，导致同一角色不同 beat 人设漂移
- BEATS 数组增删时**没有意识到 index 会漂**，导致存档 / 硬编码判定全崩

**Why:** 这不是"更细心一点"能解决的。得有一个**每次改动前跑一遍的清单**，把"看没看过顶视图 / 对没对过 canon"从"可能忘"变成"必须打勾"。

**How to apply**：以下任何操作之前，**必须**先跑对应 checklist。跑完再动手。

---

## Checklist A：改任何 beat 文本 / choice 措辞前

- [ ] 打开 [[midnight-store-cashier-canon]]，找到对应段落
- [ ] 新文本跟 canon 是否一致？——差异 > 5 字就要问用户 "换 canon 还是回滚我的实现"
- [ ] 人设描写（店员的年龄 / 心境 / 职业）跟 canon 顶部"人设基底"是否一致？
- [ ] 台词有没有嵌套单引号？中文对白用 `""` 或 `「」`
- [ ] 改完跑 node 语法自检（下方"通用出口 checklist"）

---

## Checklist B：改场景图 / 添 hotspot 前

- [ ] 打开 `C:\Users\tianlan\lycheeGame\docs\convenience-store-design.md`——找到顶视图 / 场景平面图
- [ ] 新场景图 / hotspot 坐标是否符合顶视图？关东煮锅在收银台右侧、窗边吧台靠右墙、货架在收银台左侧——**记不清就再翻一次**，别凭印象
- [ ] hotspot 的 `card` 属性是否正确？（doorstep / window / shelves / register / dispenser 五张卡）
- [ ] hotspot 的 x / y 坐标是否落在图上物件的真实位置？（400×550 viewBox）
- [ ] 生图之前 prompt 里有没有明确"NO 品牌 / 真实歌名 / logo"？
- [ ] 生图用的是 gpt-image-2 skill v1.0.1 吗？
- [ ] **旧设定物件清理走查**（对照 [[midnight-store-cashier-arch]] 里的清理清单）：
  - [ ] 老式收音机 / 老式电话 / 老照片墙 —— 场景里没有？
  - [ ] "灯管嗡鸣" 视觉暗示 —— 只保留音效，视觉去掉
  - [ ] 复古收银机造型 —— 换成现代扫码收银台
  - [ ] 店员马克杯"且慢且静" —— 换成中性物件（素色 / 店名 logo / 无字）
  - [ ] 收银台便签"记得吃药 - 妈" —— 已删
  - [ ] 手机锁屏"还好" —— 已删（新 overlay 用运营商欠费通知）
- [ ] 场景整体气质：**现代 24h 便利店 · 冷白日光灯 + 一盏偏红 · 深夜氛围**——不是复古怀旧风

---

## Checklist C：改机制层（timeline / arc / hotspot 门控）前

- [ ] 打开 [[midnight-store-cashier-arch]]，找到"新架构关键决策"
- [ ] 你要改的东西在 arch memory 里有没有约定？违反了就要跟用户确认
- [ ] 有没有可能让 index-based 逻辑再次生效？（例：改了 CASHIER_TIMELINE 就意味着 step index 变了、影响存档）—— 用 key 不用 index
- [ ] Web Audio 相关：新加的 audio 逻辑是否在用户交互后才 `new AudioContext()`？
- [ ] 有没有硬编码某个 beat 的 index（`_currentBeatIndex === 0`）？→ 改成 `kind === 'opening'` 这种 key/属性判定
- [ ] guestPresent / guestZone / playedShards / playedAnchors 是否都在正确时点被更新？
- [ ] **三条状态机硬规则**是否守住？
  1. 重复点击收银台不会重复刷客人（`spawnCashierGuest` 前 assert `guestPresent === null`）
  2. 已触发的碎片不可二次触发（`renderHotspots` filter 掉 `playedShards`）
  3. 场景切换 / 切 POV 后回来不重置 arc（arc 挂在 state.world，跟着存档走）
- [ ] 存档 v 号需要升吗？（数据结构变化 → 一定要升）

---

## Checklist D：改 world flags / ending 判定前

- [ ] 打开 arch memory 里的 flag 命名对照表 —— 用 camelCase（`catBowl` / `milkHelp` / `odenState` / `seatHeater` / `doorGap` / `cashService` / `nurseCheckout` / `driverCheckout` / `chargerState` 等），不要引入 snake_case
- [ ] **`charger` 已改名 `chargerState`**（历史遗留），别写回去
- [ ] 新 flag 有没有加到 buildEndingLines 里的 hidden lines 分档？
- [ ] 新 flag 是否会 `undefined`？—— 是的话要么给初始默认值（见 acceptance V01），要么判定要有 undefined 分档
- [ ] 所有 flag 是否在 state.world 初始化时就显式赋值？（不 lazy init，V01 用例就查这个）
- [ ] **hidden lines 里绝对不能出现的 flag**（arch A5 决策：单视角收束）：
  - `w.doorLet` / `w.penPick` / `w.catMeet` / `w.elderSlip` / `w.nursePay`（护士 arc）
  - `w.doorLetDriver` / `w.driverBuyWater` / `w.driverPen` / `w.driverTeen` / `w.driverElder` / `w.driverSeatState`（司机 arc）
  - `w.teenCat` / `w.teenPay` / `w.teenDoorSit` / `w.teenNurse` / `w.teenElder`（少年 arc）
- [ ] 结局文案不能出现"护士坐上末班地铁 / 司机把车开上主路 / 少年靠着墙角"这种旁观全知视角——**视角始终是店员**

## Checklist E：改图鉴前

- [ ] 图鉴规格是否符合 A4 定案？—— 标题「**店员篇 · 9 / 9**」+ 底部小字「更多人物篇章 敬请期待」
- [ ] 9 条图鉴条目是否严格对应 arch memory 里的"图鉴解锁对应表"？—— 不要凭印象加或减
- [ ] 保洁 / 代驾 / 观察类（nurseObserve / teenObserve）是否**没有**进图鉴？
- [ ] 15 CP 系统的其余 6 个跨 POV CP 是否已从图鉴 UI 隐藏（本篇不显示）？

## Checklist F：改标题屏前

- [ ] 是否符合 A3 定案？—— 四个 POV 全显示，nurse/driver/teen 半透明灰色
- [ ] 点击灰色 POV 是否弹出淡字「专属故事 · 敬请期待」并 2 秒后自动消失？
- [ ] 不能弹出真正的 modal / 硬打断 —— 用轻提示

---

## 通用出口 checklist（每次改完必跑）

1. **node 语法自检**（防止单引号嵌套等错误整站白屏）：

```bash
cd "C:\Users\tianlan\lycheeGame" && node -e "const fs=require('fs');const html=fs.readFileSync('midnight-store.html','utf8');const scripts=[...html.matchAll(/<script>([\s\S]*?)<\/script>/g)];for(const [,s] of scripts){try{new Function(s);}catch(e){console.log('ERR:',e.message);process.exit(1);}}console.log('OK');"
```

2. **打开浏览器强刷验证**（不能只跑 node，必须眼睛看到）：
   - 打开 https://fantasyfanftl.github.io/lycheeGame/midnight-store.html
   - 强刷（Ctrl+Shift+R / Ctrl+F5，说话时说「强刷」/「刷新」，别写快捷键——见 [[feedback-no-shortcut-spell]]）
   - 走一遍你改动涉及的路径（不是只看首屏）
   - 打开 F12 → Console 看有没有 red error

3. **自动打开浏览器给用户**（不用等他要求，见 [[feedback-open-after-update]]）：

```bash
start "" "https://fantasyfanftl.github.io/lycheeGame/midnight-store.html"
```

---

## 发布 checklist（重写完 JS 层或大改后必跑）

改机制层 / 重写 JS 层 / 大改剧本后，**必须**逐条跑过 [[midnight-store-cashier-acceptance]] 里的 17 个验收用例：

- [ ] S01-S05 · 5 条核心碎片兜底
- [ ] A01-A04 · 4 条主线锚点兜底（快速点收银台可通）
- [ ] C01-C02 · 2 条交叉互动首版关闭 / 结账顺序固定
- [ ] E01-E05 · 5 条边界异常操作
- [ ] V01 · 全 flag 初始默认值校验（dev 面板看到）

**17 项全过 → 可发布** · 有任何一项失败 → 不发布，先修

---

## 反模式清单（历史踩过的坑，别再踩）

- ❌ 假设 BEATS 数组 index 稳定 —— 一旦增删就崩，用 key 唯一识别
- ❌ 在 beat 内加 side effect（`sfx==='doorChime' → oldmanArrived=true`）—— 副作用只应该在明确的状态推进函数里，别塞在 render / play 里
- ❌ 用 setTimeout 做时间轴推进（旧代码 7.5s 后自动 doorChime + driverArrived）—— 用户操作驱动更 robust
- ❌ 让 opening 作为 hotspot shard —— 玩家先点其他会永久 skip；opening 必须自动播
- ❌ hidden lines 只处理 warm/cold 二分，忽略 undefined —— 三值化（warm/normal/cold/missed）
- ❌ 生图后不校对场景图和顶视图的一致性 —— 出的图物件位置错了 hotspot 就全错
- ❌ 改完只跑 node 校验不看浏览器 —— 有的 bug（比如 opening 被 skip 掉）纯代码看不出来
