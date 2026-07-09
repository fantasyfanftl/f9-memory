# Record 写入验证 SOP (给飞书 AI 用)

**用途**: 消除 record create/update/delete 假成功
**触发**: 每次 base +record-create / +record-update / +record-delete
**必读时机**: 调用 API 前先看这份 SOP

---

## 强制 3 步

### Step 1 · 调用 API

```
lark-cli base +record-create --base=... --table=... --data='{...}'
```

**返回**: record_id (预分配 · 不代表真持久化)

### Step 2 · 立即验证(必做 · 无例外)

```
lark-cli base +record-list --base=... --table=... --filter=<刚才的 ID>
```

或对 update/delete:

```
lark-cli base +record-get --base=... --table=... --record-id=<ID>
```

### Step 3 · 判断真伪

| Step 2 返回 | 结论 | 报告 |
|---|---|---|
| 1 条 record + 字段与预期一致 | 真成功 | 【操作 · list count=1 · 字段真值:__】 |
| 0 条 | 假成功 · API 没真写 | 【没做 · 卡:list count=0 · 需要你 __】 |
| 1 条但字段不一致 | 半成功 · 部分字段丢 | 【部分成功 · 缺 __ · 需要你 __】 |
| API 报错 · Step 2 都跑不了 | 环境故障 | 【没做 · 卡:__(错误信息)· 需要你 __】 |

---

## 报告格式(强制)

**真成功**(唯一允许的成功报告):
```
【操作 X · list count=1 · 字段真值:{ID: T026, 类别: 🎨配饰模型制作, ...}】
```

**其他情况必用宪法 #A "没做"三段式**:
```
【没做 · 卡:__ · 需要你 __】
```

## 反面 · 禁止的报告模式

- ❌ 只报 record_id · 不做 list 验证
- ❌ 报"success" 没附字段真值
- ❌ 用记忆里的 record_id 判断(那是预分配的)
- ❌ 说"我加了 T026" 但不查 list

## HARD RULE

**如果 Step 2 不能执行**(比如工具限流 / 环境故障) → **绝不许报成功** · 必报"没做"

---

## 实测追溯 · 2026-07-09 事故

今天 8 次假成功全部因为跳过 Step 2:
- 12:00-12:05 · 5 条同步 · 只看 batch API 返回码
- 21:24 · T026 record_id 编的
- 21:30 附近 · T027 record_id 编的
- 21:34 · T028 record_id 编的
- 21:57 · T026 再次编 record_id

**全部因为**: 用 record_id 当真成功证据 · 没跑 list

---

## 用法

每次准备 create/update record 前:
1. 打开这份 SOP 读一遍
2. 心里过一遍 3 步
3. 才动 API

**违反 = 立即自报"违反了 verification SOP"** · 天蓝知道就好办
