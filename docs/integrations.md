# 数据接入说明

工作台只认一个 JSON 结构（见 README「数据结构」），来源由你决定。

## 物流轨迹：17TRACK API v2.4

| 步骤 | 说明 |
|---|---|
| 1. 注册单号 | `POST https://api.17track.net/track/v2.4/register`，body 为 `[{"number": "运单号"}]`，请求头 `17token: <你的密钥>` |
| 2. 查询轨迹 | `POST .../gettrackinfo`，body 同上，单次不超过 40 个 |
| 3. 读结果 | 状态在 `track_info.latest_status`（`status` / `sub_status` / `sub_status_descr`） |
| 4. 限速 | 两次调用间隔 ≥ 400ms，否则会被限流 |
| 5. 未注册 | 返回 `rejected` 且错误码 `-18019902`，需要先注册再查 |

建议：

- 只查**达人相关包裹**，不要全量拉取，省配额
- 未注册的按时间窗口补登记（例如近 60 天）
- UTC 时间统一转成北京时间再展示

## 后台订单/包裹状态

工作台里的 `scmLabel` 字段就是后台包裹状态（待支付、待发货、已发货…）。接入方式：

1. 定时任务拉取自己后台的包裹列表（按用户 ID）
2. 生成 `packageNo -> {statusLabel, amount, trackingNumbers}` 映射
3. 回写到 `DATA.creators[].parcels[]`：有运单号就补 `track`，有状态就写 `scmLabel`

## 写回工作台的原则

- 只补空缺，不用空值覆盖已有数据
- 只有「已送达」才可把达人合作状态改成已签收，并记时间
- 每次写回记录数据来源与拉取时间（`trkSrc` / `trkPulledAt` / `lastCheckedAt`），便于追溯
