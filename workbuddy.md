# WorkBuddy 开发记录（workbuddy.md）

> 本文件用于核验本项目符合 **2026 麦当劳程序员创意开发大赛 × WorkBuddy 联动活动** 奖励条件。
> 项目全程使用 WorkBuddy 辅助完成：需求梳理 → Skill 设计 → MCP 联调 → 真实链路验证 → 文档产出。

## 一、基本信息

| 项目 | 内容 |
|---|---|
| 项目名称 | 麦门精算师（McMen Actuary） |
| 开发日期 | 2026-10-09 |
| 开发工具 | WorkBuddy（含 mcd-mcp 连接器） |
| 项目仓库 | D:/program/maidanglao（Git 仓库，main 分支） |
| 使用 MCP | 麦当劳中国 MCP Server（`https://mcp.mcd.cn`，Streamable HTTP） |

## 二、WorkBuddy 参与环节

| 阶段 | WorkBuddy 承担的工作 |
|---|---|
| 1. 选题调研 | 读取赛事文章与官方 GitHub 仓库，提取赛制、评审机制（Star 排名）、提交物清单 |
| 2. 能力摸底 | 抓取麦当劳 MCP 官方工具清单，梳理 33+ 工具的能力边界 |
| 3. Skill 设计 | 设计四大模式（凑单精算 / 每日播报 / 快捷领券 / 下单红线）与通用原则 |
| 4. 文档生成 | 产出 SKILL.md、references/、README.md、MCP_INTEGRATION.md |
| 5. MCP 联调 | 通过 WorkBuddy 连接器接入 mcd-mcp，验证 35 个工具在线可用 |
| 6. 真实链路验证 | 实跑领券 → 卡包 → 门店 → 菜单 → 券映射 → 计价全链路，用真实数据校准 Skill |
| 7. 踩坑修正 | 定位并修复「券-商品映射」「返回格式解析」「随单购价格」三类问题 |

## 三、关键对话里程碑

1. **需求确认**：确定主攻「麦门精算师」并将「每日麦麦播报」作为内置模块（一个仓库两个创意）
2. **官方材料采集**：从赛事仓库提取 `CONTEST_DECLARATION.md` 原文与提交物要求
3. **连接器配置**：在 WorkBuddy 连接器中启用 mcd-mcp，完成 MCP Token 配置与握手验证
4. **首次真实领券**：`auto-bind-coupons` 成功领取 3 张券（免费脆薯饼 / 麦旋风买一送一 / 9.9 元冰美式）
5. **技术突破**：发现券必须搭配「凭证商品码」使用，改用 `query-store-coupons` 获取映射关系
6. **真实省钱验证**：三张 ¥9.9 券组合试算成功，原价 ¥49.00 → 实付 ¥29.70（立省 39.4%）
7. **文档回灌**：将实测经验写入 SKILL.md 与 references，形成可复现的开发规范

## 四、验证证据

- 实测记录：`examples/live-run-2026-10-09.md`（全部数据来自 MCP 实时返回）
- 调用链路：`auto-bind-coupons` → `query-my-coupons` → `query-store-coupons` → `query-nearby-stores` → `query-meals` → `calculate-price` → `campaign-calendar` → `query-my-account`
- 计价成功样例：麦旋风任选券 ¥14.00→¥9.90；薯薯任选券 ¥16.00→¥9.90；冰美式券 ¥19.00→¥9.90
- Git 提交记录：见仓库 commit 历史（含骨架提交与实测经验回灌）

## 五、安全声明

- 本项目所有文件中**不包含任何真实 MCP Token**，Token 仅存在于本地 WorkBuddy 连接器配置中
- 实测过程中未执行任何下单、积分兑换、抽奖等资金/权益操作
- 领券为正常用户权益领取行为，未产生任何费用

## 六、使用到的 WorkBuddy 能力

- MCP 连接器（自定义 MCP Server 接入）
- 文件读写与 git 操作
- 网络请求与网页内容提取（赛事规则、官方工具文档）
- 结构化文档生成（Skill 规范、集成说明、示例集）
