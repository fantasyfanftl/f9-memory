---
name: project-per-town-unique-icons
description: "求合体游戏地图上,每个村庄标记必须用不同的独特图标,不能所有村都共用同一张"
metadata: 
  node_type: memory
  type: project
  originSessionId: ce876c80-d7bb-451c-bdc9-3e51cbfc0e55
---

在 `merge-town.html` 的地图界面 (`renderMap`),每个 town 的标记 (`.town-marker`) 目前所有村都使用 `icon-05-house.png` (未通关) 或 `icon-10-castle.png` (通关)。用户明确反对:**每个村庄要有自己主题的独特图标**,不能十个村都长一个样。

**Why:** 用户看地图时觉得"所有村都是同一栋房子"很多余/无个性。每个村的名字暗示了不同主题(喵喵村/冬眠村/月湖村/滨海村/卡普村/新手村/无仓村/财主村/多多岛/天猫岛/大都会),icon 应该匹配这些主题。

**How to apply:**
- 短期临时方案:给 TOWNS 每个 entry 加 `mapIcon: 'icon-XX-XXX.png'` 字段,从现有 11 张图标里为每村分配不同的一张,让视觉上先区分开
- 长期方案:为每个村专门生成一张主题相关的 diorama 图标(比如 月湖村 → 湖边亭台 / 冬眠村 → 雪屋 / 天猫岛 → 猫塔 / 多多岛 → 岛屿驿站…)
- 通关状态的图标也可以按村主题各不同,不必都是城堡
- 相关:[[project-qiuheti-rules.md]] 求合体核心规则
