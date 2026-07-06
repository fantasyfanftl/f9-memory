---
name: midnight-store-cashier-kickoff
description: 《凌晨两点的便利店》v1.0 开工第一读物 · 下次会话打开就读这条 · 5 分钟看完全局 · 直接进入执行
metadata: 
  node_type: memory
  type: reference
  originSessionId: 23b2404d-32c8-4e64-b53a-f3713ff7b936
---

**这是下一次重写 midnight-store.html 的开工第一读物**。5 分钟看完全局，然后直接进入执行，全程无决策卡点。目标：3-5 小时跑完收银员单视角最小闭环。

---

## 一、开工前 10 分钟自查清单

### 1. 5 份核心记忆资产就绪核查

| 文件 | 用途 |
|---|---|
| [[midnight-store-cashier-canon]] | 剧本 · 结局变体 · 内容唯一来源 |
| [[midnight-store-cashier-arch]] | 架构 · 规则 · 变量 · UI 规范 · 核心开发依据 |
| [[midnight-store-cashier-acceptance]] | 17 条验收用例 · 写完自测标准 |
| [[midnight-store-cashier-future]] | 禁做清单 · 仅用于边界判定 |
| `C:\Users\tianlan\lycheeGame\docs\midnight-store-handoff.md` | 顶部速览 · 总纲 |

### 2. 口径一致性三点校验（有一处不一致就先修好再开工）

- 所有文档无「多人同店 / 先到先结」残留 · 统一为 **严格线性单客流**
- 代码侧变量命名统一 **camelCase**（设计侧 snake_case 仅注释说明 · 见 arch 命名对照表）
- 动效参数统一：文本 200ms 淡入 · 场景 400ms 平滑滑动 · 门铃前置 120ms

### 3. Legacy 版本预处理

```bash
# 归档旧文件（同目录保留作 CSS/SVG/Audio 复制源）
mv C:/Users/tianlan/lycheeGame/midnight-store.html C:/Users/tianlan/lycheeGame/midnight-store-v1-legacy.html
```

**从 legacy 提取的可复用资产**（Copy 到新文件）：
- `<head>` 里全部 CSS
- HTML 骨架（title / game / ending / phone-overlay / dialogue-overlay / inline-panel / char-tabs / scene-cards / dev-panel / notebook 图鉴 DOM）
- 5 张 SVG 场景卡（doorstep / window / shelves / register / dispenser）
- Character silhouettes SVG（含 cleaner / driverAlt 已加）
- Web Audio 全套函数（ensureAudio / startAmbient / start*Ambient / playDoorChime / playFluorescent / playPhoneVibration / startCashierEarbudSong / stopCashierEarbudSong / startCashierBackgroundMusic / playArcComplete / stopAmbient 等）
- Assets：`assets/audio/*.mp3` / `assets/img/*.webp`
- Utility：renderTextWithQuotes · toast · showScreen · formatGameTime · updateClockUI · scrollToCard 等

**从 legacy 彻底删除的旧逻辑**（新文件不要复制）：
- 旧 BEATS 数组（`BEATS = { cashier: [...], nurse: [...], driver: [...], teen: [...] }` 全都删）
- 旧 unplayedBeats · triggerBeatByIndex · triggerHotspot 分发逻辑
- 旧 `startCashierAmbient` 里的 7.5s 自动 doorChime + driverArrived hack
- 旧 `playBeat` 里的 `sfx==='doorChime' → oldmanArrived` 硬编码
- 旧 `_currentBeatIndex === 0` 硬编码（改判 kind）
- 旧 v1 / v2 saveState 迁移逻辑（直接从 v3 开始）
- 旧多角色 POV 相关 render 分支（其他 POV 按 A3 半透明灰色处理）

---

## 二、开工输入包 · 给代码 AI 的标准指令（用户 copy-paste 用）

以下是可以直接发给代码 AI 的完整指令。**先讲边界、再给顺序、最后提要求**。

```
【重写任务说明 · 凌晨便利店 midnight-store.html JS 层】

1. 重写范围：仅重构 JS 逻辑层，完整保留现有 HTML 结构、CSS 样式、
   SVG 场景、音频资源、图鉴/结局 DOM、开发面板 DOM。不改动视觉层。

2. v1.0 硬约束红线（严格禁止）：
   - 多人同店（任意时刻店内仅一位客人）
   - 好坏结局（结局永远是静帧收束，只是细节多少）
   - 成就弹窗 / 收集进度条 / 引导箭头
   - 好感度数值 / 任务提示 / 新手教程

3. 参考文档阅读顺序：
   ① midnight-store-cashier-kickoff（就是本文档 · 全局速览）
   ② handoff.md 顶部（决策速览表 · 60 秒掌握整体框架）
   ③ midnight-store-cashier-arch（架构 / 状态机 / 时间线 / 变量命名 / UI 动效规范）
   ④ midnight-store-cashier-canon（所有剧本文本 · 结局变体 · 唯一内容来源）
   ⑤ midnight-store-cashier-acceptance（17 条验收用例 · 写完按此逐条自测）
   ⑥ midnight-store-cashier-future（仅用于边界判定 · v1.0 不实现其中任何扩展）

4. 命名规范：所有代码侧状态变量使用 camelCase，与 arch 命名对照表一致。
   设计文档里的 snake_case 只是注释，不进代码。

5. 出口约束：
   - 每次编辑完必跑 node 语法自检（防单引号嵌套整站白屏）
   - 完成后必须自动打开浏览器（不等用户说）
   - 发布前必须逐条跑过 acceptance 17 条用例，全过才算合格
```

---

## 三、重写执行顺序 · 5 步 · 步步可校验

**核心原则**：按依赖关系排序，写完一步语法自检 + 快速试玩 · 避免后期大面积返工。

### Step 1 · 底层状态与时间线骨架（预估 30-45 分钟）

- 初始化 `state.world` · **必须显式赋所有 flag 默认值**（不 lazy init · V01 就查这个）
- 初始化 `state.world.cashierArc = { step, guestPresent, guestZone, playedShards, playedAnchors }`
- 定义 `CASHIER_TIMELINE`（8 步：idle → oldman → nurse → cleaner → driver → driverAlt → teen → idle）
- 实现 `ensureCashierArc()` / `currentCashierStep()` / `findCashierBeatByKey()`
- 实现阶段推进锁：前一阶段客人未完全离店，下一阶段不可触发（arch 硬规则 1）

**语法自检 + 快速 debug**：打印当前 arc 状态可读即可。

### Step 2 · 核心交互逻辑（预估 60-90 分钟）

- 定义 `BEATS.cashier`（数组 · 每 beat 带 key / kind / hs · 14 段按 canon 落地）
- `handleRegisterClick()`：opening 保底 → guest anchor → advance 三分支
- `advanceCashierStep()` / `spawnCashierGuest()` / `playCashierAnchor()`
- `afterCashierAnchor()` / `afterCashierShard()` / `maybeCompleteCashierArc()`
- `closeBeat()` 里加 cashier arc anchor/shard 完成回调
- 碎片超时永久失效（进入下一 step 时自动跳过）· 触发过标记为 played 不可重复

**快速试玩**：走完老人一整波（进店 → 结账 → 离开），验证 arc.step 会 +1、guestPresent 恢复 null。

### Step 3 · 渲染与动效层（预估 45-60 分钟）

- `renderHotspots` cashier 分支：register 永远显示 + 当前 step.shards 未 played 的显示
- `renderCharacters` + `guestPositionFor` 用 arc.guestZone 分流
- `isCharPresent` 加 cashier POV 分支
- 文本渲染：整段淡入 200ms · 选项延迟 150ms 淡入 · 选中后结果文本 300ms 淡入
- 场景切换：400ms `ease-in-out` · 门铃先响 120ms 后 scrollToCard · 玩家操作优先于自动动画
- Hotspot 无默认高亮 · hover 只做轻微 opacity 变化 · 「继续」按钮 30% → 80% opacity

**快速试玩**：切场景 / 快速点击不要有动画堆叠鬼畜。

### Step 4 · 配套模块补齐（预估 45-60 分钟）

- **图鉴模块**：9 条按 arch 图鉴对应表 · 静默解锁 · 未解锁「？？？」· 无进度条无弹窗
- **结局模块**：`buildEndingLines` 按 canon 基础静帧 + 按命中 flag 追加 0-18 条增量（时间顺序 · 未命中不追加）
- **手机 overlay**：按 arch A2 黑锁屏 + 底部播放器条 + 中间横幅通知「运营商提醒 · 您的账户余额已不足 10 元」3s 后暗
- **dev panel 4 项按钮**：跳阶段 / 强制锚点（6 个 anchor 下拉）/ 清存档 / 查全 flag
- **异常兑底文案**：单句 ≤10 字 · 静默淡入场景底部 · 不弹 alert（见 arch §19 表）
- **v3 存档结构**：`saveState / loadState` 直接从 v3 开始 · v1 / v2 一律不认

### Step 5 · 边界兜底 + 逐条跑 17 项验收（预估 30-45 分钟）

- 防重复点击（arch 硬规则 1）· 防阶段回退 · 防状态异常
- reducedMotion 开关（dev panel 里 · 所有动画时长置 0）
- **逐条跑 acceptance 17 项用例 · 全过才算发布合格**：
  - S01-S05 · 5 条核心碎片兜底
  - A01-A04 · 4 条主线锚点兜底（快速点收银台可通）
  - C01-C02 · 2 条交叉互动 v1.0 关闭 / 结账顺序固定
  - E01-E05 · 5 条边界异常
  - V01 · dev panel 查全 flag 初始默认值

---

## 四、首版验收标准（完工即验收 · 3 点全过就合格）

1. **流程可通** —— 全程只点收银台可完整通关到结局 · 无卡关 · 无报错 · 无空白文本
2. **兜底生效** —— 所有可选碎片不触发时，主线正常推进，玩家无感知内容缺失
3. **约束合规** —— 未出现任何 v1.0 禁做功能 · 所有规则与文档口径一致

### 出口 checklist（每次改完必跑，出口两件事）

```bash
# 1. node 语法自检（防单引号嵌套整站白屏）
cd "C:/Users/tianlan/lycheeGame" && node -e "const fs=require('fs');const html=fs.readFileSync('midnight-store.html','utf8');const scripts=[...html.matchAll(/<script>([\s\S]*?)<\/script>/g)];for(const [,s] of scripts){try{new Function(s);}catch(e){console.log('ERR:',e.message);process.exit(1);}}console.log('OK');"

# 2. 自动打开浏览器（等 30-60s 同步后强刷）
start "" "https://fantasyfanftl.github.io/lycheeGame/midnight-store.html"
```

**发布前**：strong 强刷 + 走完店员一整晚 + F12 Console 看无 red error + 17 项验收全过 → 才算发布合格。
