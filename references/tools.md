# 麦当劳 MCP 工具速查（33 个）

- **端点**：`https://mcp.mcd.cn`（Streamable HTTP）
- **认证**：`Authorization: Bearer <MCP_TOKEN>`（Token 从 https://open.mcd.cn/mcp 手机号登录后控制台激活）
- **限流**：600 次/分钟/Token，超限返回 429
- **协议版本**：2025-06-18 及之前

## 🎫 优惠券（本 Skill 核心）

| 工具 | 说明 | 要点 |
|---|---|---|
| `available-coupons` | 查询当前可领取的麦麦省券列表 | 领前快照，用于对比"新领了哪些" |
| `auto-bind-coupons` | 一键领取所有可领券 | 无需 couponId，全量领取；标志性动作 |
| `query-my-coupons` | 查询我的卡包优惠券 | 关注有效期，筛 3 天内过期 |
| `query-store-coupons` | 查询当前门店可用券 | 需先定门店 |

## 🏪 门店与菜单

| 工具 | 说明 | 要点 |
|---|---|---|
| `query-nearby-stores` | 查询地址附近门店（到店场景） | 仅到店自取/堂食 |
| `delivery-query-stores` | 查询收货地址可配送门店 | 外卖场景 |
| `query-meals` | 查询门店可售餐品列表 | 需 storeCode + beCode |
| `query-meal-detail` | 查询餐品详情/套餐组成/可替换项 | 凑单换购的关键 |

## 💰 计价与订单（红线区）

| 工具 | 说明 | 要点 |
|---|---|---|
| `calculate-price` | 计算金额（含券、配送费、优惠） | 入参为 `items` 数组 + 顶层 `storeCode`/`beCode`/（外卖需 `addressId`）；只算钱不下单，可放心批量调用 |
| `create-order` | 创建订单，返回支付链接 | ⚠️ 必须用户确认后调用 |
| `cancel-order` | 取消订单 | ⚠️ 需二次确认 |
| `query-order` | 查询订单详情 | 34 位订单号 |
| `order-list` | 查询近期历史订单 | 非商城订单 |

## 📦 配送地址

| 工具 | 说明 | 要点 |
|---|---|---|
| `delivery-query-addresses` | 查询用户配送地址列表 | 外卖第一步 |
| `delivery-create-address` | 新增配送地址 | ⚠️ 需用户确认 |

## 🎉 活动与会员

| 工具 | 说明 | 要点 |
|---|---|---|
| `campaign-calendar` | 当月营销活动日历（进行中/往期/未来） | 播报核心数据源 |
| `query-my-account` | 积分账户（可用/累计/冻结/即将过期） | 播报积分预警 |
| `query-lottery-info` | 积分抽奖活动信息 | |
| `draw-lottery` | 执行积分抽奖 | ⚠️ 消耗积分，需确认 |
| `query-my-prizes` | 我的奖品记录 | 分页，按中奖时间倒序 |

## 🛍 麦麦商城（积分兑换）

| 工具 | 说明 | 要点 |
|---|---|---|
| `mall-points-products` | 积分可兑换商品列表 | |
| `mall-product-detail` | 商品详情（积分、有效期、说明） | |
| `mall-create-order` | 积分兑换下单 | ⚠️ 扣积分，需确认 |
| `mall-order-list` / `mall-order-detail` | 商城订单查询 | 近一年 |

## 🥗 数据与其他

| 工具 | 说明 | 要点 |
|---|---|---|
| `list-nutrition-foods` | 餐品营养成分（热量/蛋白/脂肪/碳水量/钠/钙） | 健身党模式 |
| `now-time-info` | 服务器当前时间 | 播报第一步校准日期 |
| `query-meal-assistance` | 助餐服务（仅企业团餐） | |
| `query-party-city` / `query-party-store` / `query-partystore-date` / `query-partystore-session` / `party-order-create` | 主题活动（派对/品鉴会）城市→门店→日期→场次→下单 | 下单需确认 |

## 错误码

| code | 原因 | 处理 |
|---|---|---|
| 401 | Token 无效/过期/缺失 | 引导重新申请 Token 并更新配置 |
| 429 | 超过 600 次/分钟 | 降频、减少候选数量 |

---

## ⚡ 实测要点（2026-10-09 真实链路验证，务必遵守）

1. **返回格式是「说明文档 + 内嵌 JSON」混合文本**：`content[0].text` 前半是字段说明，真正的数据在 `## Original Response` 之后的 JSON 里。解析时定位该标记，再用 `raw_decode` 提取 JSON；**不要按 markdown 标题解析**。领券/卡包类工具则直接返回 markdown 列表，按正则提取即可。
2. **`calculate-price` 的 items 用 `productCode` 字段**（不是 `code`），`quantity` 必填 ≥1；**套餐必须带 `roundList` 子项**（先 `query-meal-detail` 拿套餐 round 结构再组装）。
3. **价格单位是「分」**：`price: 3450` = ¥34.50，展示时除以 100。
4. **实际在线 35 个工具**（比文档多 `query-promotions`、`query-survey-coupon`；派对日期工具实际名为 `query-party-store-date`）。
5. `query-nearby-stores` 到店场景：`beType=1` + `searchType=2` + `city` + `keyword`；返回 `storeCode`、`beCode`、营业状态、预约时段。
6. `query-meals` 返回 `data.meals` 为 **code → 餐品详情的映射表**（含 `currentPrice`/`originalPrice`/`discountType`），`data.categories` 为分类索引。
7. `calculate-price` 返回 `takeWayList`（堂食/外带选项），下单时需要选择一个 takeWay。
8. 实测真实数据锚点（广州华夏路店 storeCode=1400205）：板烧鸡腿堡单品 ¥23.50（productCode=1406）、板烧鸡腿堡三件套 ¥34.50（productCode=9900005462）。

