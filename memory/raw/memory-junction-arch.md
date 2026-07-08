---
name: memory-junction-arch
description: memory 目录物理架构 · C 盘是 D 盘 f9-memory git repo 的 junction, 一份文件两处可读, 不要当独立目录 rm
metadata:
  node_type: memory
  type: reference
---

# memory 目录物理架构

**2026-07-08 起** memory 目录的物理布局:

```
D:\voice-input-data\f9-memory\memory\    ← 真实文件在这
                    ↑
                    │ (Windows Junction)
                    │
C:\Users\tianlan\.claude\projects\C--Users-tianlan\memory\
                    ↑
                    └── Claude Code 读这里 (但物理落到 D)
```

## HARD RULES 给未来的 AI / 人

1. **写文件到 C 路径 = 落到 D**(junction 透明)—— 写完不用额外同步
2. **rm -rf C 路径** 不会删 D 的文件,但**会破坏 junction**,之后 Claude 读不到 memory
3. **git 操作只在 D 路径做** —— C 路径没有 `.git`,git 命令会报错
4. **副业 memory 在另一个 private repo** `D:\voice-input-data\f9-memory-side\`,不 junction 到 C(飞书 AI / GitHub Claude 都读不到)

## 出问题怎么修

**junction 断了 / 被误删**:
```bash
rm -rf /c/Users/tianlan/.claude/projects/C--Users-tianlan/memory
powershell -Command "New-Item -ItemType Junction -Path 'C:\Users\tianlan\.claude\projects\C--Users-tianlan\memory' -Target 'D:\voice-input-data\f9-memory\memory'"
```

**验证 junction 状态**:
```bash
powershell -Command "Get-Item 'C:\Users\tianlan\.claude\projects\C--Users-tianlan\memory' | Select LinkType, Target"
```
应输出 `Junction` + `D:\voice-input-data\f9-memory\memory`

## 为什么这么做

之前 C 和 D 是**两份独立的物理拷贝**,每次写都要手动同步(经常漂移,7-8 就发现 C 缺 1 个副业文件)。junction 后一份文件两处可读, C 盘还省空间。

## 相关

- [[user-github]] — f9-memory repo 地址
- [[voice-input-tool]] — 语音工具也在 D:\voice-input-data\
