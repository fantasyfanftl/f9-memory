---
name: sop-3d-miniature-icon-pipeline
description: "求合体/合并类游戏的 3D 微缩图标生图完整流程 SOP —— 命名/生成/抠透明/WEBP/接入 4 步跑通,含 prompt 心法·尺寸叙事表·避雷坑·主题接入检查表"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 2e3d89b8-c335-4c00-917a-01561df965e2
---

lycheeGame 类游戏（合并小镇 / 求合体 / 修仙合并 / 苏州园林 等）的 3D 微缩瓷砖图标 SOP，2026-07-05 迭代 4 轮后定稿（含苏州主题验证跑通）。

## 0. 一次跑通命令模板（复制即用）

新主题叫 `<theme>`（例：`suzhou` / `immortal` / `xxx`），流程 4 步：

```bash
# 1) 建目录
mkdir -p "C:/Users/tianlan/lycheeGame/merge-<theme>-assets"

# 2) 生成 13 张（每张一个 gpt-image-2 命令，文件名必须以 -src.png 结尾）
uv run "C:/Users/tianlan/.claude/skills/gpt-image-2/scripts/generate_image.py" \
  --prompt "<见第 2 节 prompt 模板>" \
  --size <见第 3 节尺寸表> --quality medium --output-format png \
  --filename "merge-<theme>-assets/tile-<N>-src.png"

# 3) 抠透明（产出同名 .png，512×512 底部对齐）
python "C:/Users/tianlan/lycheeGame/_make-tiles-transparent.py" \
  "merge-<theme>-assets/*-src.png"

# 4) WEBP 压（长边 260, q48, method 6）
python "C:/Users/tianlan/lycheeGame/_convert-tiles-to-webp.py" \
  "merge-<theme>-assets/tile-*.png"

# 5) 清理只留 .webp
cd "C:/Users/tianlan/lycheeGame/merge-<theme>-assets"
rm -f *-src.png *-src.webp tile-*.png

# 6) 接入 merge.html（见第 8 节 checklist）+ bump ?v=N
```

**跳任何一步都翻车**。跳透明脚本 → 灰白棋盘；跳 WEBP → 手机加载慢；跳清理 → 每张 4 个冗余文件。

## 1. Prompt 模板（正/负词固定，只改主体）

```
Clean smooth 3D miniature diorama, three-quarter isometric angle,
matte cartoony stylized game asset with SOFTLY ROUNDED edges
(NOT chunky faceted low-poly, NOT voxel, NOT Minecraft, NOT flat illustration).
<主体描述 —— ONE 主体, 明确列出要 OMIT 的杂物>.
<主题调色> aesthetic.
Minimalist. TRANSPARENT PNG background. Soft simple shadow. No text.
<横竖构图 hint: HORIZONTAL / VERTICAL>
```

**必带正向 4 条 · 必带负向 4 条**，缺一样就翻车（低多边形 / 扁平插画 / voxel / 玻璃质感）。

**构图铁律**：
- 一主体 + 至多一陪衬
- 明确说 `no extra leaves, no side plants, no bonsai, no pebbles`
- 单件小物件（1×1）必须加 `tiny circular moss/stone base`（否则 AI 退化成 2D 图标）
- **多部件建筑（山门/月洞门/城楼类）必须说 `ONE integrated piece — no floating parts, no gaps`**（否则 AI 把瓦顶、柱、门画成分离的浮空片段 —— 苏州 tile-128 踩过坑）

## 2. 尺寸 · 形状 · 叙事 · 生成尺寸对照表

| 形状 | 值 | 生成 size | 内容类型 |
|---|---|---|---|
| 1×1 | 2/4/8 | `1024x1024` | 可携带小物件（灵草、丹药、盆景、灵石） |
| 1×2 竖 | 16 | `1024x1536` | 高单体器物（炼丹炉、假山、望楼） |
| 2×1 横 | 32/64 | `1536x1024` | 横向铺开（药田、花坛、石桥） |
| 2×2 | 128/256/512 | `1024x1024` | 单体建筑（山门、道观、月洞门） |
| 2×3 竖 | 1024 | `1024x1536` | 高耸建筑（悬空阁、长廊） |
| 3×3 | 2048/4096/8192 | `1024x1024` | 宏大场景（皇宫、仙山、水乡） |

**注**：透明脚本会统一归一化到 512×512 底部对齐，所以生成尺寸只影响 AI 构图 hint，不影响最终画布。但 2048/4096/8192 可用 `--quality high` 提升细节。

## 3. 叙事线设计

**一根线到底，别在同一档位来回蹦**（原料→加工品→原料 是反面教材）。

已验证的 3 条叙事线：

| 主题 | 递进 |
|---|---|
| **人间（三国）** | 稻草→帐篷→摊位→望楼→马厩→城门→茶楼→院落→塔楼→佛塔→曹操皇宫→天坛→长城 |
| **仙界（修仙）** | 灵草→灵芝→丹药→炼丹炉→药田→草庐→山门→道观→藏经阁→悬空阁→灵霄殿→昆仑山→龙盘星海 |
| **苏州（园林）** | 太湖石→盆景→荷缸→假山→花坛→石桥→月洞门→荷塘→六角亭→长廊→中庭花园→拙政园→水乡全景 |

**调色一定要跟其他主题拉开**：
- 人间：暖褐 + 朱金
- 仙界：翠玉 + 雾青 + 金红点缀
- 苏州：白墙黛瓦 + 玉水 + 微绿 + 偶朱红

## 4. 后处理：三大 hazard

**Hazard 1 · 假透明**：AI 会把灰白棋盘画进 RGB，脚本用 flood-fill 抠掉。
- 检查：`Image.open(x).mode == 'RGBA'`
- 曾经翻车：跳过透明脚本直接 WEBP → 游戏里灰白背景

**Hazard 2 · Halo 白边**：anti-aliasing 遗留浅色像素。
- 脚本 `halo_cleanup(iters=2)` 迭代抠除半透明浅色

**Hazard 3 · 浮空构件**：AI 把复杂建筑画成分离部件（月洞门瓦顶悬空）。
- Prompt 里必须写 `ONE integrated piece — no floating parts, no gaps`

## 5. WEBP 压缩参数（当前最优）

- 长边 **260px**（游戏最大显示 ~250px，1:1 够用）
- WEBP `quality 48` · `method 6`
- 结果：每张 **8-30KB**，全套 13 张 **~130KB**（苏州）· 全套 26 张 **~260KB**（人间+仙界+苏州）

如果嫌大：可以再收紧到 200px/q40，或者反过来 320px/q60（照旧）。

## 6. 微调工具（不重生图）

- **CSS `transform: scale(N)`** 单独缩某档 tile（放 `merge.html`）：
  ```css
  .tile-2 .tile-img { transform: scale(0.72); }
  .tile-4 .tile-img { transform: scale(0.75); }
  ```
- **文件互换**改叙事顺序（`mv tile-4.webp tile-8.webp`）
- **cache-bust `?v=N` bump**（每次改资源必做）

## 7. 踩过的坑 · 全避雷表

| 坑 | 症状 | 解法 |
|---|---|---|
| Prompt 说 "transparent PNG" | 得 RGB + 灰白棋盘 | 后处理脚本抠 |
| Prompt 说 "extreme low-poly" | 出 chunky voxel Minecraft 风 | 换 "smooth softly rounded" + 明确负向 |
| 主体太丰富 | 装饰堆砌，杂乱 | 改 "just ONE main subject" + 列出去掉的 |
| 单件 1×1 生成成 2D 图标 | 缺 3D 参照 | 加 "tiny circular moss/stone base" |
| 建筑瓦顶浮空 | AI 画成分离部件 | 加 "ONE integrated piece, no floating parts, no gaps" |
| 图片压完浏览器还是老的 | 浏览器 cache | bump `?v=N` |
| 跳过 transparency 直上 WEBP | 无 alpha, 游戏灰白 | 必须先跑 transparency 脚本 |
| 阴影被裁掉 | bbox 裁边 padding 不够 | 脚本 `pad=8` 现在够 |
| 苏州文件夹爆 52 个文件 | 忘清中间态 | pipeline 跑完执行清理命令 |
| 图太大加载慢 | 长边或 quality 高 | 260px / q48 是当前甜点 |

## 8. 新主题接入 merge.html · Checklist

（4 处必改，缺一样看不到新主题）

**① mode picker HTML** —— 加一张卡（`merge.html` 内 `<div class="mode-picker">` 里）：
```html
<div class="mode-card" data-mode="<theme>">
  <h3>苏 州</h3>
  <p>园林水乡</p>
</div>
```

**② renderTiles JS 分支** —— 让新 mode 走图片路径：
```js
if (renderMode === '3d' || renderMode === 'immortal' || renderMode === '<theme>') {
  const dir = renderMode === 'immortal' ? 'merge-immortal-assets'
            : renderMode === '<theme>'  ? 'merge-<theme>-assets'
            : 'merge3d-assets';
  ...
}
```

**③ label 渲染 guard**：
```js
if (renderMode !== '3d' && renderMode !== 'immortal' && renderMode !== '<theme>') {
  // ... flat 模式的数字标签
}
```

**④ cache-bust `?v=N`** 加 1。

## 9. 多主题目录结构

```
lycheeGame/
├─ merge3d-assets/          ← 人间（暖褐/朱金/三国建筑）
├─ merge-immortal-assets/   ← 仙界（翠玉/金红/修仙场景）
├─ merge-suzhou-assets/     ← 苏州（白墙黛瓦/玉水/园林水乡）
└─ merge-<theme>-assets/    ← 未来新主题
```

每个目录 13 张 `.webp`，命名 `tile-{2,4,8,16,32,64,128,256,512,1024,2048,4096,8192}.webp`。**只留 webp，别攒 png**。

## 相关

- [[feedback-use-gpt-image-2]] 生图工具入口
- [[lychee-game-site]] 项目结构 + 自动同步
- [[project-qiuheti-rules]] · [[project-per-town-unique-icons]] 兄弟项目