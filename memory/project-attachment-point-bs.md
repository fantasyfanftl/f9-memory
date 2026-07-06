---
name: project-attachment-point-bs
description: 挂点与BS结算项目：帽子/发饰/耳饰挂角色身上的位置计算流程，团队含石静玲/岳淑茹/陈川
metadata: 
  node_type: memory
  type: project
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

## 项目

3D 角色配饰的**挂点位置**（挂在角色发型 / 头 / 身上的准确位置）+ **BS 结算**（Blend Shape 变形计算，配合不同发型 / 表情自动变形）。

## 团队分工

| 角色 | 谁 | 干啥 |
|---|---|---|
| 挂点美术侧 | **石静玲 / 岳淑茹** | 调整挂点位置 + 截图提交 |
| BS 技术侧 | **陈川** | Houdini 里做 BS 配置计算 + 手修 |
| 效果最终确认 | **天蓝** | 看做完的效果对不对 |
| 审核回复 | **至尊菜刀** | "可以了" @天蓝 = 通过 |
| 问题反馈 | **小杜** | 中间过程反馈问题给她 |

详细 SOP 见 sop-attachment-point-workflow。

## 关键 URL

- **Base 表**（配饰模型截图回填地）：https://papergames.feishu.cn/base/FI1Wb1Ypbagi1oszRYjcsxQanYl

## 交付准则

1. **挂点先确认，再算 BS**（挂点位置没定就算 BS 是白算）
2. **默认发型先算，通过再批量算全发型**（避免全量返工）
3. 挂点**按发型适配，不是脸部**——改一个发型挂点可能联动影响别的发型
4. 帽子类容易悬空，必须逐个发型检查
