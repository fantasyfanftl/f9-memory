---
name: project-v9-base-infrastructure
description: V9 base + 各表 + 群 chat_id + 人员 open_id 速查（从飞书 AI 2026-07-07 dump 提取）
metadata: 
  node_type: memory
  type: project
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

从飞书 AI 那边拉过来的**具体 ID 速查表**。以后写脚本 / 调 API / 判断这条数据在哪个表，先来这里查。

## Base + 表

| 名字 | ID |
|---|---|
| **V9 base**（配饰主库） | `DhOcbs2rda2nLOsmlYqcHyfZnDf` |
| V9 主表 | `tblN5flhFVhNmUZr` |
| V9 TL 表（TL 静态模型进度） | `tbl2lwv39GuDUMkf` |
| V9 配饰挂点制作表 | `tblq6iy7LTCC830y` |
| V9 配饰特殊任务流程表 | `tblUahU9ebpCWQB6` |
| **4 组外派出勤管理 base** | `FI1Wb1Ypbagi1oszRYjcsxQanYl` |
| 数据表_20260623_003258 | `tblhLikO49Gpx2tN` |
| 每日出勤记录 | `tblsus8Vk8lTYGOW` |

## 关键群 chat_id

| 群 | chat_id | 用途 |
|---|---|---|
| **X3-角色资产效果核对群** ✅ | `oc_5d68be8d87f3d02157962d6f81300ada` | V9 录入唯一正确群；菜刀"可以了"审核回复的地方 |
| **配饰挂点群**（收集群） | `oc_a6b6ac7257016ee3409f43d79d78c290` | 挂点截图/BS 结果统一发这里；旧 id `oc_e4b0fd2245d386bdd51e093ef1da094c` **已失效** |
| **4 组群**（模型组内部） | `oc_2333adc9f813be04c60c869ee3843d65` | 内部 4 组同事的工作群 |
| **X3-原画确认与交接** ❌ | `oc_326e93d94bc859300f1d4497751110ee` | 曾经 trigger #2 扫错群，别用 |

## 关键 V9 字段

- `fldnDkVjhj` — 模型截图字段（挂点表 / TL 表）
- `fldEaB7LGI` — select 手填（业务标记）
- `fldy3Zn8R0` — formula 自动算（机械生产阶段）
  - 上面两个是**两套独立口径不联动**
- `fldIKf3VxI` / `fldPQsZG04` / `fldeTuT1RF` / `fldMt8SZtE` / `fldN7S5ZMs` = V01~V05 视角字段（不是版本号，是视角）
- `fldjmyW19a` → `fldnTWBOdB` = 6-20 修复的 v9_parser 字段 ID

## 版本编号规则（澄清）

- **V01 ~ V05** = 5 个视角（前 / 后 / 左 / 右 / 顶）**不是版本号**
- **染色版本** 用 P 编号后缀 `_01` / `_02`（如 P162FA01）
- 归档取新一张 = 按 file_token 创建时间倒序（**"最新一张"原则** — 见 sop-file-submission-tips）

## 关键人 open_id

| 名字 | open_id | 备注 |
|---|---|---|
| **至尊菜刀** | `ou_120cf2557dfa16e5da055521ee8595d8` | 真名可能是**沈思奇 / 沈思琪** |
| **谷谷（王思宇）** | `ou_1b4cc569bdf8db7c6f353598ec52921a` | |
| **房志林** | `ou_bf52d91c574c318f835c88b7faf8a24e` | 配饰挂点群 2026-06-30 首次出现的新提交人 |
| **王智超** | `7553480507826651155` | 王智超真人 |
| **陈川** | `7320153041733730305` | 陈川真人 |
| **翎汐蝶** | `ou_c6b56682af7181a1e17737279ac9944a` | 4 组 |
| **迪迦** | `ou_59d2d3706a6e46376c725912335aaa4f` | 4 组 |
| **阿彭** | `ou_704029ac33db9e59502e14012ff0b042` | 4 组 |
| **文安** | `ou_afec49bcb87d5b9978e82b1095bf2c81` | 4 组 |
| **别急** | `ou_bf7865b162bf1a834ce18c92d663e49` | 4 组 |
| **零一** | `ou_0b84545b5574fb6f85fc0fafa7bf7470` | 4 组 |

## 数据操作铁律（从飞书 upsert/delete 翻车经验学的）

1. **upsert 必带 `--record-id`**（不带会新建重复）
2. **delete 用 `record-list` 不用 `record-get`**（record-get 有幽灵缓存）
3. **link 字段格式** `[{"id":"rec..."}]`（错格式报 `800010701`）
4. **`ok=true` ≠ 真成功**（要真查一次数据才算）

## 使用相对路径的坑

用相对路径时必须 `cd` 到目录后 `--file ./`，别用绝对/相对混着写。

## 已删除的历史资源（别再用）

- ~~教程 v2.2 云文档 `MA2zdhGkGoDiRAxQdZ0c8tBGnKc`~~（6-27 14:24 删）
- ~~教程 v2.0 `QNSYdG9TJohXQcxmHzPco5oDnWa` + `NMUxdwhndoa9fpxPoe9cqRGnnib`~~（同上）
- ~~艾莉亚教程 v1.0 `WZz5dyttBo3RnqxTRA4cbNtInMe`~~（6-27 14:24 删）

## 现存活的关键文档

- **飞书 P1 文档（6-29 工作交接）** `https://papergames.feishu.cn/docx/EuZ4dHqMyoSxq3xUF2VcFOp3nae`
