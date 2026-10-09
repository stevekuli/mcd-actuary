# MCP 接入说明（MCP_INTEGRATION）

## 使用的 MCP Server

| 项目 | 值 |
|---|---|
| 服务方 | 麦当劳中国（McDonald's China） |
| 接入地址 | `https://mcp.mcd.cn` |
| 传输协议 | Streamable HTTP |
| 认证方式 | 请求头 `Authorization: Bearer <MCP_TOKEN>`（Token 从 https://open.mcd.cn/mcp 申请） |
| 限流 | 600 次/分钟/Token |
| 官方文档 | https://github.com/M-China/mcd-mcp-server |

## 实际使用的 Tool 清单

### 凑单精算主链路

| 工具 | 用途 | 业务价值 |
|---|---|---|
| `auto-bind-coupons` | 一键领取全部可领券 | 用户不再漏领券，精算的前提是"券池完整" |
| `query-my-coupons` | 拉取卡包全量券 | 建立可用券池（门槛/面额/有效期） |
| `query-store-coupons` | 查询当前门店可用券 | 过滤门店不适用的券，避免算错价 |
| `query-nearby-stores` | 到店场景查询附近门店 | 锁定 storeCode/beCode |
| `delivery-query-addresses` / `delivery-query-stores` | 外卖场景地址与可配送门店 | 覆盖麦乐送场景 |
| `query-meals` | 查询门店可售菜单 | 组合生成的数据源 |
| `query-meal-detail` | 查套餐组成与可替换项 | "套餐 vs 单点"价差是省钱主战场 |
| `calculate-price` | 候选组合券后价试算 | 核心精算引擎，批量 ≤8 个候选 |
| `campaign-calendar` | 当日活动叠加判断 | 会员日特价、第二件半价计入精算 |
| `list-nutrition-foods` | 热量/蛋白质过滤（可选） | 健身党按营养目标筛组合 |
| `create-order` | 用户确认后创建订单 | 完成"算完就能点"闭环（需显式确认） |
| `query-order` / `cancel-order` | 订单查询与取消 | 售后闭环（取消需二次确认） |

### 每日播报链路

| 工具 | 用途 |
|---|---|
| `now-time-info` | 校准播报日期 |
| `campaign-calendar` | 今日进行中 + 未来 3 天活动 |
| `available-coupons` + `auto-bind-coupons` | 领前领后对比，统计"新领 N 张" |
| `query-my-coupons` | 筛 3 天内临期券 |
| `query-my-account` | 积分余额与即将过期积分预警 |

## 调用流程图

```
【凑单精算】
用户输入(预算/人数/必吃/场景)
        │
        ▼
锁定门店 ── 到店: query-nearby-stores
        └── 外卖: delivery-query-addresses → delivery-query-stores
        │
        ▼
券池构建: auto-bind-coupons → query-my-coupons + query-store-coupons
        │
        ▼
菜单获取: query-meals → (必吃项) query-meal-detail
        │
        ▼
组合试算: 生成 ≤8 候选 → 逐个 calculate-price(带券)
        │
        ▼
省钱报告: Top3 方案(原价/券后/实付/立省) + 精算师点评
        │
        ▼
用户确认 → calculate-price 复核 → 二次确认 → create-order

【每日播报】
now-time-info → campaign-calendar → auto-bind-coupons
            → query-my-coupons(临期) → query-my-account(积分)
            → 一条简报输出
```

## 安全与合规设计

1. **Token 零落盘**：Token 仅存于 MCP Client 的连接器配置中，Skill 代码、日志、输出均不含 Token；仓库内仅有 `YOUR_MCP_TOKEN` 占位符。
2. **资金操作双确认**：`create-order` / `mall-create-order` / `draw-lottery` / `delivery-create-address` 必须先展示明细并获用户明确确认；`cancel-order` 二次确认。
3. **限流自律**：组合试算候选上限 8 个，播报为串行单轮调用，远低于 600 次/分钟限额。
4. **数据真实**：所有价格、菜单、券信息均来自 MCP 实时返回，不缓存、不编造。

## 业务价值

| 痛点 | 本项目的解决方式 |
|---|---|
| 券藏在 App 深处，用户懒得领/不知道有券 | 一键全量领券 + 临期主动提醒 |
| 凑单算不清，"满30减10"经常凑过头或凑不够 | 按券门槛反向凑单，批量实算券后价 |
| 套餐和单点哪个划算，普通用户算不明白 | 套餐/单点/换购三形态对比试算 |
| 会员日、新活动错过才发现 | 每日播报把"今天什么便宜"推送到眼前 |
