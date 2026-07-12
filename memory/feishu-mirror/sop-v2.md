# record-write-verification — 飞书 base 写记录 SOP

> 立约:2026-07-10 11:10
> 触发案例:T028 月老耳饰 — upsert 无 --record-id 跑 3 次 = 建 3 条同名 record(2 条空附件)
> 根因:`base +record-upsert` 无 `--record-id` 时,按内容/字段匹配 — 匹配不到 = 创建新条
> 7-10 11:10 天蓝加 1 条:**禁用 upsert,改用 create / update 分开走** — upsert 模糊性 = bias 入口
> 7-10 12:01 天蓝第 2 拍:实际能用的"create/update"是 `+record-batch-create` / `+record-batch-update`(无单条)
> 7-11 20:06 天蓝第 3 拍:**从 T030 起,挂点/调整类任务上传原图(2MB+),不传缩略图** · T027-T029 不追溯升级 · 习惯改进
> 不是新铁律,是工具选择 + 操作规范
> 不进 GitHub,只入 workspace

## ❌ 禁用工具

- ~~`base +record-upsert`~~ — **禁用**
- 原因:upsert 在无 --record-id 时 = 按内容匹配 + 匹配不到 = 创建,**模糊性 = 误建入口**
- 触发案例:T028 跑 3 次 = 建 3 条同名

## ✅ 工具选择(2 个,分清楚)

### 场景 A · 新建(明确"没这条")

```bash
# 用 +record-create / +record-batch-create
lark-cli base +record-create \
  --base-token <base_token> \
  --table-id <table_id> \
  --json '{字段1: 值1, 字段2: 值2}' \
  --as user
```

### 场景 B · 改(明确"知道 record_id")

```bash
# 用 +record-update(待验证,可能需要 --record-id 走 update 分支)
lark-cli base +record-update \
  --base-token <base_token> \
  --table-id <table_id> \
  --record-id <target_id> \
  --json '{要改的字段: 新值}' \
  --as user
```

## 写完必做 3 步验证(铁律 A.1 配套)

1. `+record-list` 拉全表,grep 新 record_id
2. 验证 count(新建 = +1,update = count 不变)
3. 涉及附件 = `grep file_token` 验证 token 真在 record 行里(不是只在 upload 返回里)

## 关键决策点

- **不知道该 create 还是 update** → 先 `+record-list` 查,看到目标 record_id → update;没看到 → create
- **不确定是否已存在** → `+record-list` + grep 内容字段查重,**别赌**
- **误建 2+ 条** → 立刻报"卡:重复建,需删" + 拍删方案(a 删 2 留 1 / b 保留 / c 全删重建)

## 关联铁律

- **铁律 A.1**:说成功必须有 tool_call 证据链
- **铁律 A.1.b 撤销(7-10 11:10 天蓝拍板)**:不立为铁律,是 SOP
