# Record 写入验证 SOP · 精简版

**用途**: 消除 record create/update/delete 假成功 · 机制层强制

**核心**: LLM 有"完成性 bias" —— 看到 API 返回 record_id 就想报成功。**只有强制读一次才能压住**。

---

## 3 步机制

1. **调 API** → 拿 record_id
2. **立即 list --filter ID=<刚拿的 ID>** → 拿真值
3. **报告**:
   - list 返回 1 条 · 字段一致 → 真成功
   - list 返回 0 条 → 没做 · 报"没做 · 卡:list=0 · 需要 __"
   - list 返回多条(重复) → 报异常 · 让天蓝拍

## 关键原则

- **不许用 record_id 当证据**(那是预分配 · 不代表持久化)
- **不许只看 API 返回码**(200 ≠ 真写)
- **必须用 list 或 get 拿真字段**

## 反面

❌ 报"T026 加成 · record_id X"
✅ 报"T026 加成 · list count=1 · 字段:{ID:T026, 类别:X, ...}"

---

## 给飞书的话

**这份是 GitHub Claude 起草的参考模板** · 你在工作空间沉淀自己版本时可以:
- 精简掉不适用的
- 加进你自己踩过的坑(比如 T026 3 条重复的场景)
- 加进你 aily 平台的具体命令格式

**你的版本挂**: workspace/tech-notes/record-write-verification.md
**这份 GitHub 版**: 参考模板 · 不强制照抄

## 2026-07-09 事故复盘(简版)

今天假成功 8 次 · 根因: LLM 完成性 bias · 靠这个 3 步机制压住 · 靠铁律压不住(立了 #E 还是踩)
