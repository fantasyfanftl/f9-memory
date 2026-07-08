---
name: user-programs
description: "用户告诉我的常用程序的启动路径。用户说\"打开 XX\"时，直接从这里查路径 Start-Process 就行，不用满硬盘搜。"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a6080841-ce21-4ac0-8634-d3c36736a79d
---

用户的常用程序 → 路径映射表。用户会陆续告诉我别的程序，我在这里加行。

## 查找逻辑

用户说"打开 Photoshop" / "打开 PS" / "打开图像处理" 之类的时候：
1. 在下表找匹配（别名列出多个就是这个用途）
2. 找到 → 直接 `Start-Process -FilePath '<路径>'`
3. 没找到 → 问用户路径（找到后 append 到本表）

## 表

| 别名 | 路径 |
|---|---|
| Photoshop / PS / 图像处理 | `D:\Adobe Photoshop 2023\Photoshop.exe` |
| 飞书 / Feishu / Lark | `C:\Program Files\Feishu\Feishu.exe` |

## 添加新条目的格式

```
| 用户的常用叫法（逗号分隔） | `完整可执行文件路径` |
```

想让用户告诉路径最省事的方法：
- 让他右键桌面/开始菜单的快捷方式 → 属性 → 复制"目标"字段
- 或者直接截图属性对话框给我
