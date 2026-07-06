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
- **HOTSPOTS 静态坐标全表**（phone / catBowl / milkCabinet / odenPot / driverSeat / doorway / register / nurseObserve / teenObserve 等）—— **完全沿用 legacy 数值**，不重新计算

**v1.0 完全沿用 legacy 实现的 4 项表现层**（不做重构 · 稳定无风险）：
- 手机 overlay 样式与结构
- speechSynthesis 语音播报逻辑
- 图鉴入口按钮位置
- 窗外光晕效果

**从 legacy 彻底删除的旧逻辑**（新文件不要复制）：
- 旧 BEATS 数组（`BEATS = { cashier: [...], nurse: [...], driver: [...], teen: [...] }` 全都删）
- 旧 unplayedBeats · triggerBeatByIndex · triggerHotspot 分发逻辑
- 旧 `startCashierAmbient` 里的 7.5s 自动 doorChime + driverArrived hack
- 旧 `playBeat` 里的 `sfx==='doorChime' → oldmanArrived` 硬编码
- 旧 `_currentBeatIndex === 0` 硬编码（改判 kind）
- 旧 v1 / v2 saveState 迁移逻辑（直接从 v3 开始）
- 旧多角色 POV 相关 render 分支（其他 POV 按 A3 半透明灰色处理）
- **CP 系统整套**：`CONVERGENCES` 表 · `state.cpProgress` · `state.convergences` · beat 里的 `cps: [...]` 字段 · 所有 CP 解锁 UI 逻辑 —— v1.0 无跨视角内容，保留即冗余 + 易触发状态残留 · v2.0 再重新接入

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
- **⚠️ HOTSPOTS 坐标从 legacy 复制的同时，对照 `C:\Users\tianlan\lycheeGame\docs\convenience-store-design.md` 顶视图逐个校验热区跟场景物件的位置一致性 —— 顺手做，发现偏差顺手调整，不要留到最后**
- **⚠️ 沉浸感四层规范 v1.0 项必须落到位** —— 见 arch §20：环境音分层混音表 · UI 音效消音 -18db · 客人离店 150ms 后玻璃门合上 · 深夜色彩基调 · 文本区毛玻璃底 · 零错误提示 · 场景视觉呼应结局 flag（`document.body.classList` 切换）

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

### Legacy 偏差对照 QA 清单（11 条 · 重写时对照 legacy 逐条修正）

除了 acceptance 17 项功能验收，还必须对照当前 legacy 版本（`midnight-store-v1-legacy.html`）修正下面 11 处已知偏差。这些**是 legacy 有、v1.0 规范不该有**的东西，重写时必须**主动清除或改造**，不能沿用。

#### 🔴 核心机制偏差（4 条 · 与规范直接冲突）

| ID | Legacy 现状 | v1.0 规范修正 | 参考 |
|---|---|---|---|
| **M1** | 标题屏 4 个 POV 全可点击 | nurse/driver/teen 半透明灰色 + 点击弹「专属故事 · 敬请期待」淡字 2s 自动消失 | arch A3 |
| **M2** | 图鉴叫「交汇图鉴 · 六组不经意的相遇」跨角色 CP 体系 | 图鉴改「**店员篇 · 9 / 9**」+ 底部小字「更多人物篇章 敬请期待」· CP 系统整套（CONVERGENCES / cpProgress / convergences / cps 字段）**完全删除** | arch A4 + 删除清单 |
| **M3** | 场景里显性 `01:47` `03:00` 时间数字 | **时间轴全程隐形** · 玩家仅通过氛围 / 光影 / 客流节奏感知 · 全站不出现任何倒计时 / 时间戳 | arch 隐形时间轴核心原则 |
| **M5** | 标题页直接进场景，无开局演出 | 进 cashier POV 后 700ms 自动播 opening beat（戴耳机 / 手机震动 / 欠费提醒锁屏） | canon 开场 + arch A2 + arch §14 |

#### 🟡 体验氛围问题（4 条 · 影响沉浸感）

| ID | Legacy 现状 | v1.0 规范修正 | 参考 |
|---|---|---|---|
| **E1** | `¥8` `¥6` `¥3.5` 价签到处标 · 像模拟经营 | 保留极少量价签做真实感铺垫 · 大部分去掉 · 视觉重心还给空间氛围与人物 | 整体克制留白基调 |
| **E2** | 「四个陌生人，交汇成一场安静的无人知晓的温柔」这类直白抒情 | 克制留白 · 用细节说话 · **不直接告诉玩家"这很温柔"**——好的氛围是玩家自己感受到的 | canon 叙事节奏 + [[design-detail-objects-carry-story]] |
| **E3** | 窗边座位区域摆日用品 / 泡面 / 纸巾 | 窗边只是吧台歇脚区 · 冷饮柜旁置 · 食品货架在左侧 | `docs/convenience-store-design.md` 顶视图 |
| **E4** | 结局只一句笼统抒情 · 冷漠 / 热心选择完全一样 | 基础静帧 + 按 flag 命中追加 0-18 行细节 · 按 CASHIER_TIMELINE step 固定顺序 | canon 结局 flag 增量变体表 |

#### 🟢 其他细节（3 条 · 命名 + 动效）

| ID | Legacy 现状 | v1.0 规范修正 | 参考 |
|---|---|---|---|
| **O1** | 「长途司机」/「少年」 | 「**网约车司机**」/「**叛逆少年**」 · 全站统一 | canon 人设 |
| **O2** | 打字机逐字显示 | **整段淡入 200ms ease-out + 点击推进** | arch §15 UI 动画规范 |
| **O3** | 场景切换生硬 | **400ms ease-in-out 平滑滑动 + 门铃前置 120ms + 玩家操作优先于自动动画** | arch §15 UI 动画规范 |

**验收方式**：M1-M5 + E1-E4 + O1-O3 一共 11 条，重写完打开新版本走一遍，逐条对着看——**"legacy 里有的 legacy 味"必须清干净**。跟 acceptance 17 项功能验收互补：acceptance 保证功能对，这份 QA 清单保证 legacy 坑不再复现。

---

## 五、里程碑速查表（过程中随时对照进度）

| 里程碑 | 完成节点 | 验收标准 | 累计耗时 |
|---|---|---|---|
| **M1 · 骨架可用** | Step 1 完成 | 状态机可运行 · dev panel 切阶段无异常 · 底层架构对齐 arch 规范 | ~30-45 min |
| **M2 · 主线可玩** | Step 2 完成 | 全程只点收银台可完整通关 · 14 段 beat 剧本文本全量落地 · 核心交互正常 | ~90-135 min |
| **M3 · 版本就绪** | Step 5 完成 | 全模块落地 · 17 项验收全过 · 符合所有 v1.0 硬约束 · 可直接发布 | ~3-5 h |

**M2 是最重要的锚点** —— 到 M2 就有一份"可玩的东西"，即使 Step 3-5 没做完也能给用户 demo。写代码时优先保 M2，别在 Step 3 动效上花过多时间。

---

## 六、发布后收尾动作（M3 达成后必做）

1. **版本归档**：Save 后本地脚本会自动 commit + push · 稳定发布后手动 rename 为 `midnight-store-v1.0-stable.html` 保留可追溯版本
   ```bash
   cp C:/Users/tianlan/lycheeGame/midnight-store.html C:/Users/tianlan/lycheeGame/midnight-store-v1.0-stable.html
   ```
2. **文档同步**：如果实现层有跟 arch / acceptance 不完全对齐的微调，回来更新对应 memory 保证口径唯一（选 arch memory 为准）
3. **产出版本说明**：写进 handoff.md 底部 · 极简三行：
   - v1.0 包含内容（收银员单视角 · 8 步 timeline · 14 beat · 9 图鉴 · 单视角结局）
   - 已知限制（其他 3 POV 未开放 · 交叉彩蛋未开放 · 图鉴 9/9 仅本篇）
   - 后续迭代方向（v1.1 氛围细化 · v1.2 多周目文本 · v2.0 多客同店 + 三 POV）

---

## 七、风险预案（提前想好应对，不慌）

| 风险 | 应对 |
|---|---|
| **旧代码复用出现兼容问题** | 只复用**静态资产**（HTML/CSS/音频/SVG/DOM 骨架）· JS 逻辑层**完全从零重写**·  不强行复用旧的 BEATS / render 分发 / 硬编码 hack · 避免被旧架构拖累 |
| **动效实现与设计预期有偏差** | 先保证功能可用 + 参数基本符合 arch §15 规范（200ms / 400ms / ease-in-out）· 细节体感优化后置到 v1.1 · 不阻塞主流程开发 |
| **出现未预判的边界 bug** | 优先修复影响主线通关的核心 bug（S 类 + A 类用例失败）· 边缘场景问题（部分 E 类 / V01 细节）记录到 backlog 后续迭代处理 |
| **开发耗时超出 5h 预期** | 优先保 M2 里程碑 · 配套模块（图鉴文案 / dev panel 次要按钮 / 异常兑底文案）可简化落地 · 核心体验先闭环 · 精工留给下一次会话 |

---

## 八、启动指令与中途查阅

**下次会话触发本流程**：用户说「**开始重写便利店**」或类似 signal → AI 直接读本 kickoff → 按 Step 1-5 执行 · 不用来问设计问题

**中途查阅速查**：
- 「翻 canon」→ 打开 [[midnight-store-cashier-canon]] 找剧本文本
- 「翻 arch」→ 打开 [[midnight-store-cashier-arch]] 找机制 / UI / 命名规范
- 「跑 acceptance」→ 打开 [[midnight-store-cashier-acceptance]] 按 17 项逐条自测
- 「查 future」→ 打开 [[midnight-store-cashier-future]] 判断某功能是不是 v1.0 该做（99% 是"v1.0 不做"）
- 「查 selfcheck」→ 打开 [[midnight-store-selfcheck]] 找改动前的自检项

**M3 达成后可衔接的方向**（用户可能会问下一步做啥）：
- v1.1 · 5 条氛围细化（关键词高亮 / 文本-音效联动 / 场景内错落淡入 / 呼吸微动效 / 环境音渐变）
- v1.2 · 多周目文本变体（同一 beat 二周目起换说法）
- v2.0 · 多客同店 + 三 POV 内容框架 + 跨视角彩蛋预埋
- 见 [[midnight-store-cashier-future]] 详情
