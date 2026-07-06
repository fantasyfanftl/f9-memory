---
name: midnight-store-cashier-arch
description: 《凌晨两点的便利店》收银员视角机制骨架——隐形时间轴 + 点收银台推进 + 三档陌生人善意。重写 JS 层时的架构参考
metadata: 
  node_type: memory
  type: reference
  originSessionId: 23b2404d-32c8-4e64-b53a-f3713ff7b936
---

**决定背景（2026-07-06）**：本次会话对 midnight-store.html 做了大量补丁式改造（在 flat BEATS 数组上叠 CASHIER_TIMELINE / handleRegisterClick / opening auto-play / v1→v2 存档等），代码变成"旧机制 + 补丁 + 新机制"的混合体，用户判断"检查后修改浪费更大，不如删掉重写"。**下次会话按本文档 + [[midnight-store-cashier-canon]] 从零重写 JS 逻辑层**。

**决策已完成**：下面所有关键决策（保留/删除/UI 规范/状态机边界/音效映射/开场规则）都是 2026-07-06 会话结束前用户拍板过的定案。**直接执行，不要再问**。

**验收标准**：重写完 JS 层必须逐条跑过 [[midnight-store-cashier-acceptance]] 里的 17 个用例，全过才发布。这是硬性发布门槛，不是"尽量"。

## 全局决策速览（下次会话开工前一分钟能看完）

| # | 决策项 | 定案 |
|---|---|---|
| A1 | 保洁热心档 | 顺手放独立包装小面包 + "搭着吃。"（详见 canon 阶段二） |
| A2 | 手机 overlay UI | 黑锁屏 + 底部极简播放器条 + 中间横幅通知「运营商提醒 · 您的账户余额已不足 10 元」3s 后自动暗 |
| A3 | 标题屏其他 POV | 全显示 · nurse/driver/teen 半透明灰色未开放 · 点击弹「专属故事 · 敬请期待」淡字 2s 消失 |
| A4 | 图鉴规格 | 标题「**店员篇 · 9 / 9**」 · 底部小字「更多人物篇章 敬请期待」 |
| A5 | 结局范围 | 单视角收束，只写店员这一晚。删除其他三个 POV 的结局段落 + hidden lines 里所有非店员 flag |

## 重写边界

### 保留（HTML / CSS / assets 不动）
- `<head>` 里所有 CSS —— 黑白电视质感、hotspot 视觉、character SVG 样式、卡片切换动画、图鉴 UI、结局屏、手机 overlay 样式（`.phone-overlay`）
- HTML 骨架：`#title-screen` / `#game-screen` / `#ending-screen` / `#dialogue-overlay` / `#inline-panel` / `#phone-overlay` / `#char-tabs` / `#scene-cards` / `#dev-panel` / notebook 图鉴
- 5 张 SVG 场景卡片的 `<svg>` 内容（doorstep / window / shelves / register / dispenser）
- Web Audio 子系统全部函数：`ensureAudio` / `startAmbient` / `startCashierAmbient` / `startNurseAmbient` / `startDriverAmbient` / `startTeenAmbient` / `startCashierEarbudSong` / `stopCashierEarbudSong` / `startCashierBackgroundMusic` / `setEarbudSongVolume` / `playDoorChime` / `playFluorescent` / `playPhoneVibration` / `playArcComplete` / `stopAmbient` —— 音频参数是磨过很久的，不重写
- `assets/audio/*.mp3`（Suno 生成的耳机歌背景音乐）
- `assets/img/*.webp` / `*.png`（gpt-image-2 生成的场景图）
- `characterSVG(id)` / `charIconSVG(id)` —— 五个人物 silhouette 的 SVG
- `renderTextWithQuotes` / `toast` / `showScreen` / `formatGameTime` / `updateClockUI` / `scrollToCard` / `updateCardDots` 等 utility
- 图鉴 / 结局屏 / notebook 的展示代码（`showNotebook` / `showEnding` 的 DOM 操作层，不含数据组装）
- 存档 utility：`SAVE_KEY` / `localStorage.setItem/getItem` 的 wrapper，但**存档结构从零设计**
- dev panel 全套（`updateDevPanel` / `updateDevTimeline` / `updateDevFloorplan` 等），继续保留但适配新数据模型

### 删（所有 JS 逻辑层从零）
- `CHARACTERS` / `NPC` / `HOTSPOTS` —— 结构保留但重新定义（要含保洁 / 代驾）
- `BEATS`（含所有 cashier / nurse / driver / teen 数组）—— 从零，cashier 按 canon 剧本 14 段，其他三个 POV **本次不重写内容**，先用空数组占位（`BEATS.nurse = []` 等），走 arc-empty 自动 completed 兜底
- `CASHIER_TIMELINE` —— 保留本次设计
- `CONVERGENCES` —— 15 个 CP 定义可保留，但**跟 nurse/driver/teen 挂钩的 CP 都拿不到 unlock**（另一半 POV 是空的）。可以先保留 CP1-CP9（cashier + oldman/nurse/driver/teen 单侧），跨 POV CP（CP10-CP16）先注释掉
- `state` 对象 + 所有 render / trigger / play / close 函数 —— 从零
- 结局数据组装（`buildEndingLines` 中的 hidden lines 分档判断）—— 按新 world flag 命名从零

## 新架构关键决策

### 1. 收银员分支的核心状态机

```js
state.cashierArc = {
  step: 0,                    // 0..7, index into CASHIER_TIMELINE
  guestPresent: null,         // 'oldman' | 'nurse' | 'cleaner' | 'driver' | 'driverAlt' | 'teen' | null
  guestZone: {},              // { <charId>: 'shelves' | 'window' | 'doorstep' }
  playedShards: [],           // shard key 数组
  playedAnchors: [],          // anchor key 数组
  finalDownshifted: false     // 玩家在 step 7 主动"下班"
};
```

**关键：用 key（string）而非 index 标识 beat**。这次会话踩的最深的坑就是 BEATS.cashier 数组增删后 index 全错位。用 key 一劳永逸。

### 2. CASHIER_TIMELINE 结构

```js
const CASHIER_TIMELINE = [
  { kind: 'idle', shards: ['catBowl'] },  // step 0 · 开局空窗（phone 是 opening 自动播）
  { kind: 'guest', guest: 'oldman',    enterCard: 'shelves',                   shards: ['milkCabinet'],            anchorKey: 'oldmanCheckout' },
  { kind: 'guest', guest: 'nurse',     enterCard: 'shelves', walkTo: 'window', shards: ['odenPot', 'nurseObserve'], anchorKey: 'nurseCheckout' },
  { kind: 'guest', guest: 'cleaner',   enterCard: 'shelves',                   shards: [],                          anchorKey: 'cleanerCheckout' },
  { kind: 'guest', guest: 'driver',    enterCard: 'window',                    shards: ['driverSeat'],              anchorKey: 'driverCheckout' },
  { kind: 'guest', guest: 'driverAlt', enterCard: 'shelves',                   shards: [],                          anchorKey: 'driverAltCheckout' },
  { kind: 'guest', guest: 'teen',      enterCard: 'doorstep',                  shards: ['teenObserve'],             anchorKey: 'teenCheckout' },
  { kind: 'idle', shards: ['doorway'], persistGuest: 'teen', persistZone: 'doorstep' }  // step 7 · 收尾
];
```

### 3. 三种 beat kind

- `opening` —— 只有 `phone`，进 cashier POV 自动播（**不作为 shard 挂在 hotspot 上**，否则玩家先点收银台就永久 skip）
- `shard` —— 可选碎片，某个 step 期间才能点，错过永久错过，NPC 走兜底
- `anchor` —— 结账锚点，玩家点收银台触发；播完 → guest 离店 → step++

### 4. 收银台点击的语义分层

```
if opening 没播 → 先播 opening（保底）
elif guestPresent && step.kind==='guest' → 触发结账 anchor
elif !guestPresent && step.kind==='idle' && step 不是 terminal → 呼叫下一波客人（门铃 + 走位 + guestPresent=下一位）
elif !guestPresent && terminal idle → arc.finalDownshifted=true → completeArc
```

### 5. hotspot 可用性

cashier POV：
- register 永远显示（handleRegisterClick 内部分流）
- 当前 step.shards 里未在 playedShards 的显示
- 其他一律不显示

其他 POV：走旧的 flat unplayedBeats 逻辑（本次重写不动其他 POV）。

### 6. 客人位置动态化

```js
const GUEST_ZONE_POS = {
  'oldman:shelves':    { card: 'shelves',  x: 200, y: 240 },
  'nurse:shelves':     { card: 'shelves',  x: 260, y: 320 },
  'nurse:window':      { card: 'window',   x: 140, y: 345 },
  'driver:window':     { card: 'window',   x: 200, y: 355 },
  'teen:doorstep':     { card: 'doorstep', x: 100, y: 500 },
  'cleaner:shelves':   { card: 'shelves',  x: 140, y: 440 },
  'driverAlt:shelves': { card: 'shelves',  x: 280, y: 440 }
};
```

`renderCharacters` 里 `guestPositionFor(id)` 用 zone 查表，否则回退 `HOTSPOTS[id]`。

### 7. Nudge 引导策略

- 空窗期（`!guestPresent`）→ items[0] = register stub，nudge 显示"· 点 一 下 收 银 台 ·"
- 有客期（`guestPresent`）→ items[0] = 当前 step 的 shard，nudge 显示对应文案
- 不再限"只 firstBeat 显示"—— 只要 arc 未结束就一直显示，避免玩家迷路

### 8. World flag 命名保持 camelCase

代码里一律 camelCase。设计文档 / 用户对话里可能出现 snake_case，落地时映射：

| 设计侧 snake_case | 代码侧 camelCase |
|-------------------|-----------------|
| `cat_bowl`        | `catBowl`       |
| `milk_help`       | `milkHelp`      |
| `oden`            | `odenState`     |
| `seat_heater`     | `seatHeater`    |
| `door_gap`        | `doorGap`       |
| `cash_service`    | `cashService`   |
| `nurse_checkout`  | `nurseCheckout` |
| `driver_checkout` | `driverCheckout`|
| `charger`         | `chargerState`  |

同时期新增（无设计侧别名）：`cleanerCheckout` / `driverAltCheckout` / `nurseObserved` / `teenObserved`。

**初始化时必须显式赋默认值**（详见 [[midnight-store-cashier-acceptance]] 用例 V01）——不要 lazy init，否则会出 `undefined`。

### 9. Web Audio 只在用户交互后启动

`AUDIO.ctx = new AudioContext()` 必须在 click / touch 之后。`initAudio()` 在 start-game 第一次点击里调。**加新音频子系统时绝对不能在 script 顶部就 `new AudioContext()`**（用户明确硬雷之一）。

### 10. 存档结构 v3 起步（本次重写从头版本）

```js
{
  v: 3,
  cashierArc: {...},        // 上面的 state.cashierArc
  world: {...},              // camelCase flags
  convergences: [],
  cpProgress: {...},
  completedPOVs: ['cashier', ...],
  notebookRecord: {...},
  observedObjects: [...],
  beatsCompleted: <int>      // 驱动时钟 UI
}
```

v1 / v2 一律不认，玩家从头开始（当前用户群体极小，可接受）。

### 11. 状态机三条硬规则（防退化）

1. **重复点击收银台不会重复刷客人** —— `spawnCashierGuest` 前检查 `arc.guestPresent` 必须为 null，否则忽略
2. **已触发的碎片不可二次触发** —— `renderHotspots` 里 shard 显示前 filter 掉 `arc.playedShards` 里已有的 key
3. **场景切换（滑动卡片、切 POV 后又切回）不重置 arc 状态** —— `arc` 挂在 state.world 里跟着存档走，`switchPOV('cashier')` 只做 lazy `ensureCashierArc()`，从不 reset

### 12. 三档选项交互规范

- **展示样式**：三档垂直排列在 dialogue-overlay 底部（沿用现有 CSS 选项胶囊），文字左对齐、字号一致，不做"推荐档"高亮
- **选中反馈**：点选后胶囊短暂高亮 200ms → 关闭选项面板 → 展示 choices[i].text 的结果文本 → "继 续" 按钮出现
- **不可回退**：一旦选定，effects 立即写入 world，无 undo 无回滚。玩家自己承担选择后果
- **voice 台词**：`choices[i].voice` 存在时，选中后触发 speechSynthesis 说这句台词（现有 Web Speech API 逻辑保留）

### 13. 音效触发映射表

| 时机 | 音效 | 触发点 |
|---|---|---|
| 客人推门进店 | `playDoorChime()` | `spawnCashierGuest` 里，先于 render |
| 结账扫码 | 新加 `playScanBeep()` 或复用现有 UI 音 | anchor beat 的 choice text 展示时 |
| 门口橘猫叫 | 新加 `playCatMeow()` 或用 SFX 数据 | `catBowl` shard 触发时 |
| 玻璃门被风吹 | 已有荧光灯 SFX 类似逻辑，参考实现 | 空窗期偶发（当前 `startCashierAmbient` 里有 scheduleFluor / scheduleVibration，加一个 scheduleDoorRustle） |
| 手机震动 | `playPhoneVibration()`（已实现） | opening + 空窗期随机 25-55s |
| 关东煮咕嘟 | 环境音 loop 里已有 | 一直背景（关东煮开着时音量微增） |
| 冷柜低噪 | 环境音 drone | 一直背景 |
| 耳机歌淡入 / 淡出 | `startCashierEarbudSong()` / `stopCashierEarbudSong(fade)` | opening 里三档 effects |
| 店内背景音乐（Suno mp3） | `startCashierBackgroundMusic(vol)` | opening 完成后延迟 1.2s |
| Arc 完成过场 | `playArcComplete()` | `maybeCompleteCashierArc` |

**所有音效创建必须走 `ensureAudio()` 保证 AudioContext 在用户交互后**（硬雷 3）。

### 14. 开场演出交互规则

- 进 cashier POV 后 700ms 自动播放 opening beat（不需要玩家点手机）
- opening beat 里的 title / text **逐段自动展示**：
  - 每段（`\n\n` 分割）之间自动等 1.6s，不需要点击
  - 展示完最后一段后 → 弹出三档选择胶囊（同时手机 overlay 里的通知横幅弹出并 3s 后暗）
- 玩家可以点屏幕任意位置 **跳过等待**，直接展示三档
- 三档选择后 → 结果文本按普通 beat 展示（点击推进）→ 关闭 → opening shard 完成

### 15. UI 动画与文本节奏规范（首版必做）

**统一基调**：克制、无感知、服务深夜氛围。所有动画用 CSS `transform` + `opacity` 实现，不触发页面重排；缓动一律 `ease-in-out`。

**文本显示（beat 叙述段 + 三档选中后的结果文本）**：
- 整段淡入 **200ms ease-out**（开头快、收尾柔），不用打字机
- 三档选项在叙述文本完全显示后，**延迟 150ms 再以 300ms 淡入**，避免文本+选项同时蹦出
- 选中选项后：选项组 200ms 淡出，同时结果文本 300ms 淡入，无空白断层
- 「继续 →」按钮：文本区底部居中，默认 30% 透明度，hover 升到 80%；无边框无底色，只用细体文字
- 排版：行高 1.6 · 字距 0.5px · 段间距 1.2 倍行高（弱光阅读舒适度）

**场景切换（5 张卡片之间 · 客人 spawn 时自动 scrollToCard）**：
- 400ms · `ease-in-out`（模拟人转头看东西的速度变化）
- 客人进店触发自动滚动时：**先播门铃 SFX，延迟 120ms 再执行 scrollToCard**（先听见后看向，符合真实生理逻辑）
- **玩家操作优先于自动动画（硬规则）**：自动滚动过程中玩家手动滑动 → 立即终止自动滚动，尊重玩家控制权
- 滑动到首张 / 末张卡片时，10px 轻微回弹阻尼

**Hover 效果规范**：
- Hotspot 命中区无视觉高亮（观察式玩法核心：靠场景插画本身引导，见旧代码 `renderHotspots` 里的透明 hit target）
- 「继续 →」按钮 hover：opacity 30% → 80% · 200ms
- 三档选项 hover：轻微 opacity 提升（0.85 → 1.0） · 无位移 · 无边框变化
- 标题屏 POV 卡片 hover（未开放态）：显示"专属故事 · 敬请期待"淡字 2s 消失

### 16. 全局兜底与包容性

- **动画防叠加**：快速点击 / 快速切场景时，立即终止上一段未完成的动画，切入最终状态。避免动画堆叠出现鬼畜或状态错乱
- **reducedMotion 开关**（dev panel 里）：开启后所有动画时长置 0ms，全部硬切显示。照顾晕动症玩家与低性能设备
- **性能兜底**：只用 `transform` + `opacity`，不触发重排 · 保证低端浏览器 + 移动端 60 帧

### 17. UI 动画迭代方向（v1.1+ · 首版不做）

- **关键词微高亮**：情绪核心词（"温的" / "谢谢" / "没说话"）淡暖白提亮 10%，几乎察觉不到但强化情绪落点
- **文本-音效弱联动**：文本出现"猫叫" / "风声" / "咕嘟声"时，对应环境音音量抬升 10% 同步淡入
- **场景内错落淡入**：卡片滑动到位后，元素按「背景 → 大件物件 → 细节装饰」错落淡入，总时长 ≤300ms
- **常驻元素呼吸微动效**：关东煮热气缓慢上浮 / 路灯光晕呼吸闪烁 / 冷柜冷光波动，循环 3-5s，幅度 ≤10% 透明度
- **环境音随滑动渐变**：滑动过程中上一场景 ambient 渐弱、目标场景 ambient 渐强

### 18. Dev Panel 按钮清单（v1.0 必备 4 项 · 按调试优先级）

**入口**：URL 加 `?dev=1` 或按下某组合键（沿用旧代码逻辑）显示；普通玩家看不到。风格用暗调 monospace，跟主题一致。

| 序号 | 按钮 | 功能 | 用途 |
|---|---|---|---|
| 1 | 「跳阶段 →」 | `advanceCashierStep()` 直接推进 | 测试后半段流程时不用一遍遍点收银台 |
| 2 | 「强制锚点：oldman/nurse/cleaner/driver/driverAlt/teen」 | 6 个下拉键，一键触发对应 checkout anchor beat | 单独测某个 anchor 的三档文本 / 效果 |
| 3 | 「清空存档」 | `localStorage.clear() + location.reload()` | 每次重开需要清 v3 存档时用 |
| 4 | 「查全 flag」 | 展开 `state.world` + `state.world.cashierArc` 全部键值 | V01 用例查初始默认值 · 排查 undefined |

**次要按钮**（预留，非硬要求）：跳到结局 · 强制 completed cashier arc · reducedMotion 开关 · 切静音

### 19. 极端异常兑底文案（音频 / 图片加载失败）

**统一风格**：单句 ≤10 字 · 静默克制 · 非系统弹窗样式 · 嵌入场景底部淡入显示（跟正文一致的字体和 opacity）· 最低限度提示不破坏沉浸感。

| 场景 | 兑底文案 |
|---|---|
| 音频加载失败（Suno mp3 / SFX） | 「今晚没有歌。」 |
| 场景图加载失败（gpt-image-2 webp） | 「灯还没亮起来。」 |
| localStorage 不可用（隐私模式） | 「这一晚不留痕迹。」 |
| Web Audio 初始化失败（老浏览器） | 「店里很安静。」 |
| 通用未知错误 | 「有个小意外。刷新试试。」 |

**呈现**：淡入 200ms → 停留 3s → 淡出 300ms，或者一直挂在场景底部（如果整个 asset 都没加载）。**不弹 alert()，不用红色警告色**。

---

## 附加规范（Todo 补充项已并入）

### 旧设定物件清理清单（C5 · 场景图审查时逐一排查）

必须移除或替换的旧元素：
- 老式收音机 —— 场景不再有
- 显式的"灯管嗡鸣"视觉暗示 —— 保留音效但视觉去掉
- 复古老物件（旧收银机造型、老式电话、老照片墙等）—— 场景是**现代 24h 便利店**，冷白日光灯 + 一盏偏红，深夜氛围
- 店员马克杯上"且慢且静" —— 旧基底遗物，换成中性物件（比如无字素色杯 / 或写店名 logo）
- 收银台便签"记得吃药 - 妈" —— 删除
- 店员手机锁屏"还好" —— 删除（新 overlay 用运营商欠费通知，见 A2）

### 图鉴解锁对应表（C6 · 9 条对应 9 个 CP 或 anchor）

按 canon 剧本，图鉴 9 条内容对应：

| # | 图鉴条目文案（≤15 字 · 只写瞬间不直抒情绪） | 解锁条件 | 对应 flag |
|---|---|---|---|
| 1 | 踮 起 脚 · 够 那 盒 牛 奶 | 触发 milkCabinet shard | `milkHelp` |
| 2 | 一 把 数 不 清 的 硬 币 | 触发 oldmanCheckout anchor | `cashService` |
| 3 | 保 温 档 上 的 一 锅 汤 | 触发 odenPot shard | `odenState` |
| 4 | 袖 口 的 一 圈 碘 伏 | 触发 nurseCheckout anchor | `nurseCheckout` |
| 5 | 手 按 着 后 腰 的 那 个 人 | 触发 driverSeat shard | `seatHeater` |
| 6 | 冻 红 的 手 指 · 一 瓶 水 | 触发 driverCheckout anchor | `driverCheckout` |
| 7 | 台 阶 上 的 空 猫 碗 | 触发 catBowl shard | `catBowl` |
| 8 | 按 不 亮 的 那 块 屏 幕 | 触发 teenCheckout anchor | `chargerState` |
| 9 | 门 缝 · 一 条 窄 光 | 触发 doorway shard | `doorGap` |

**规则**：
- 解锁条件是"触发过对应 beat"，**不分冷漠 / 默认 / 热心档位**——只要点过一次就算解锁
- 未解锁的条目显示「？？？」（图鉴 UI 里）
- **不做**收集进度条 · **不做**成就弹窗 · **不做**"已解锁 X / 9" toast · **不做**新解锁小红点
- 解锁瞬间无任何 UI 反馈，玩家自己打开图鉴时才看到已解锁

**保洁 cleanerCheckout / 代驾 driverAltCheckout / 观察类 nurseObserve / teenObserve** —— 不进图鉴（图鉴聚焦 9 个陌生人善意的核心时刻，路人 NPC + 观察类作为氛围碎片但不计入图鉴）。


## 三条硬雷（重写时不能忘）

1. **单引号里不能嵌套单引号** —— 中文对白用中文引号 `""` 或书名号 `「」`，绝对不要 `'她说要\'升级\''` 这种（会 syntax error 整站白屏）。改完必跑 node 语法自检（详见 [[midnight-store-selfcheck]]）
2. **生图只用 gpt-image-2 skill v1.0.1，不要真实品牌/歌名/logo**
3. **Web Audio 只能在用户交互后启动**

## 站点部署（重写时不用手动 git）

本地路径 `C:\Users\tianlan\lycheeGame\midnight-store.html`。本地 15s 轮询脚本自动 commit + push，30-60s 内上到公网 https://fantasyfanftl.github.io/lycheeGame/midnight-store.html。你直接改文件，Save 就行。

## 归档旧版

重写开始那一刻，先把当前 `midnight-store.html` mv 成 `midnight-store-v1-legacy.html`（同目录，会跟着部署但没人访问），再从零 Write 新的 midnight-store.html。旧的作参考对照 CSS / SVG / 音频代码用。
