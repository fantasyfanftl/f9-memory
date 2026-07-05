---
name: sop-3d-miniature-icon-pipeline
description: "求合体/合并类游戏的 3D 微缩图标生图完整流程 SOP —— gpt-image-2 生成 → 抠透明清 halo → WEBP 压缩 → cache-bust 上线,含 prompt 心法/尺寸叙事映射/后处理坑"
metadata: 
  node_type: memory
  type: reference
  originSessionId: ce876c80-d7bb-451c-bdc9-3e51cbfc0e55
---

用户 2026-07-05 迭代出来的可复用生图 SOP,针对 lycheeGame 类的 3D 微缩瓷砖图标(合并小镇 / 求合体 / 修仙合并 等)。

## 1. 三步走流程(缺一不可)

```
gpt-image-2 生 PNG
    ↓
_make-tiles-transparent.py   (抠透明 + 清 halo + 裁 bbox)
    ↓
_convert-tiles-to-webp.py    (resize + WEBP 压缩)
    ↓
game.html 里 bump ?v=N cache-bust
```

**跳过任何一步都翻车**。曾经踩过:跳过透明脚本 → 图片 RGB 无 alpha → 游戏里灰白棋盘。

## 2. Prompt 心法

**必带正向:**
- `clean smooth 3D miniature diorama`
- `three-quarter isometric angle`
- `matte cartoony stylized game asset`
- `softly rounded edges`

**必带负向(不加就默认往 chunky 走):**
- `NOT chunky faceted low-poly`
- `NOT voxel, NOT Minecraft`
- `NOT flat illustration, NOT glossy 3D icon`

**构图铁律:**
- 一主体 + 至多一陪衬,多了立即视觉超载
- 明确列出要"去掉"的东西(`no extra leaves, no side plants, no bonsai, no pebbles`)
- 小物件(1×1 单件)必须加**小圆底盘**,否则模型会退化成 2D 图标

## 3. 尺寸 · 形状 · 叙事映射表

| 形状 | 值 | 内容类型 | 例子 |
|---|---|---|---|
| 1×1 | 2/4/8 | 可携带小物件 | 灵草 / 灵芝 / 丹药 |
| 1×2 竖 | 16 | 高单体器物 | 炼丹炉 |
| 2×1 横 | 32/64 | 横向铺开的场景 | 药田 / 草庐 |
| 2×2 | 128/256/512 | 单体建筑 | 山门 / 道观 / 藏经阁 |
| 2×3 竖 | 1024 | 高耸建筑 | 悬空阁 |
| 3×3 | 2048/4096/8192 | 宏大场景 | 灵霄殿 / 昆仑山 / 龙盘 |

**叙事线一根到底** —— 原料 → 工具 → 生产 → 居所 → 门派 → 天境。同档位别来回蹦。

## 4. 后处理两大 hazard

**Hazard 1: AI 生成的"透明底"是假的**
- AI 会把灰白格子画进 RGB
- 检查:`img.mode == 'RGBA'?`
- 靠后处理脚本 flood-fill 抠

**Hazard 2: 抠完还有白边(halo)**
- 因为 anti-aliasing
- 脚本要**迭代 2 轮**"贴着透明区的浅色像素"扩散抠除

## 5. WEBP 压缩参数(当前值)

- 长边 **260px**(游戏最大显示 250px 左右,1:1 够用)
- WEBP `quality 48` · `method 6`
- 删掉源 PNG(省 git 空间)
- 结果:每张 **10-30KB** · 全套 26 张 **360KB**(3G 秒开)

## 6. 微调工具(不用重生成图)

- `CSS transform: scale(N)` 单独缩某档 tile:
  ```css
  .tile-2 .tile-img { transform: scale(0.72); }
  .tile-4 .tile-img { transform: scale(0.75); }
  ```
- **文件互换**改叙事顺序(比如把 tile-4 ↔ tile-8 对调)
- **cache-bust `?v=N` bump**(每次改资源必做)

## 7. 踩过的坑(避雷表)

| 坑 | 症状 | 解法 |
|---|---|---|
| Prompt 说 "transparent PNG" | 得到 RGB + 灰白格子 | 靠后处理脚本抠 |
| Prompt 说 "extreme low-poly" | 出 chunky voxel Minecraft 风 | 改 "smooth softly rounded" + 明确负向 |
| 主体太丰富 | 一堆装饰堆砌,杂乱 | 改成 "just ONE main subject" + 列出去掉的 |
| 单件小物件生成成 2D 图标 | 缺少 3D 参照 | 加 "tiny circular moss/stone base" |
| 图片压完在浏览器还是老的 | 浏览器 cache | bump `?v=N` |
| 直接跳到 WEBP 压缩 | 无 alpha 通道,游戏里灰白 | 必须先跑 transparency 脚本 |
| tile-16 / tile-32 阴影拖太长 | 裁 bbox 时 padding 不够 | 脚本里 `pad=8` 现在够用 |

## 8. 多主题架构

```
merge3d-assets/         ← 人间(暖褐/朱金/三国)
merge-immortal-assets/  ← 仙界(翠玉/雾青/修仙)
```

- 同一 shape 表 · 不同调色 · 独立叙事线
- 后续加第 3 套(神魔/星际)走 `merge<主题>-assets/` 命名
- 两个脚本自动扫描不用改

## 相关

- 使用 [[feedback-use-gpt-image-2]] 生图工具
- 参见 [[lychee-game-site]] 项目部署管道
- 参见 [[project-qiuheti-rules]] 求合体核心规则
- 参见 [[project-per-town-unique-icons]] 每村独立主题
