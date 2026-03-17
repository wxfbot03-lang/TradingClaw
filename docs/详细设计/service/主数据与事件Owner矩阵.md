# TradingClaw 主数据与事件 Owner 矩阵

## 1. 文档说明

- 本文档用于统一 `docs/详细设计/service/` 下主数据、标准事件、适配侧事件和引用字段的 owner 归属。
- 目标是避免多个服务同时维护同一事实，或多个服务同时发布同一标准业务事件。
- 如本矩阵与模块文档冲突，以本矩阵的 owner 约束为准，并回写模块文档修订。

## 2. 主数据 Owner 矩阵

| 对象/事实 | Owner 服务 | 其他服务角色 | 说明 |
| --- | --- | --- | --- |
| 用户主数据 | `identity-service` | 引用 | 用户身份、登录与用户资料事实 |
| 登录会话 | `identity-service` | 引用 | `access_token`、`auth_session_status` 由身份域维护 |
| 交易账户主数据 | `account-service` | `trade-gateway-service` 持有引用视图 | 账户归属、绑定、凭据引用归账户域 |
| 默认交易会话引用 | `account-service` | `trade-gateway-service` 提供会话事实 | 只保存 `default_trading_session_id` 引用，不保存交易会话状态 |
| 账户能力投影 | `account-service` | 适配域提供原始校验事实 | 账户域是最终投影 owner |
| 交易会话 | `trade-gateway-service` | 适配域提供原始状态 | `trading_session_id`、`trading_session_status` 由交易域统一维护 |
| 统一订单/成交/持仓/余额 | `trade-gateway-service` | 适配域提供原始回报 | 所有真实交易写动作先进入统一交易域 |
| 券商会话原始事实 | `securities-trading-service` | 交易网关消费 | 券商侧原始会话、验证码、原始报文归证券适配域 |
| 交易所会话原始事实 | `crypto-trading-service` | 交易网关消费 | 交易所侧原始会话、原始订单报文归数字资产适配域 |
| 行情主数据/资讯主记录 | `market-service` / `news-service` | 引用 | 行情和资讯标准化结果由各自域维护 |
| 策略实例/配置/快照/执行记录 | `strategy-service` / `strategy-runtime-service` | 交易网关消费策略来源字段 | 策略状态与恢复点归策略域 |
| 风险事件/人工复核记录 | `audit-risk-service` | 交易网关、策略运行时消费 | 风控裁决和复核闭环归风控域 |
| 通知记录 | `notification-service` | 事件消费者 | 通知发送与投递结果归通知域 |
| OCR 任务 | `ocr-service` | AI/证券域引用 | OCR 任务、结果摘要和失败原因归 OCR 域 |

## 3. 标准领域事件 Owner 矩阵

| 标准事件 | 唯一发布者 | 说明 |
| --- | --- | --- |
| `session.created` / `session.expired` | `identity-service` | 登录会话标准事件 |
| `session.revoked` | `identity-service` | 登录会话吊销事件 |
| `user.profile_updated` | `identity-service` | 用户资料更新事件 |
| `user.level_changed` | `identity-service` | 用户等级变更事件 |
| `account.bound` / `account.unbound` | `account-service` | 账户绑定事实事件 |
| `account.capability_changed` | `account-service` | 账户能力投影变更事件 |
| `default_session.changed` | `account-service` | 默认交易会话引用变更事件 |
| `trading_session.created` / `trading_session.available` / `trading_session.expired` / `trading_session.closed` | `trade-gateway-service` | 统一交易会话标准事件 |
| `order.*` | `trade-gateway-service` | 统一订单事实事件 |
| `quote.updated` / `kline.closed` / `historical.backfilled` / `indicator.calculated` | `market-service` | 行情与指标标准事件 |
| `news.*` | `news-service` | 资讯与评分标准事件 |
| `strategy.*` / `trading_plan.*` | `strategy-service` / `strategy-runtime-service` | 策略生命周期和执行事件 |
| `conversation.created` / `tool_call.executed` | `ai-orchestration-service` | AI 会话与工具调用事件 |
| `risk.*` | `audit-risk-service` | 风险命中、待复核、复核完成 |
| `notification.created` / `notification.delivered` | `notification-service` | 通知领域事实事件 |
| `ocr.job_finished` | `ocr-service` | OCR 任务完成事件 |

## 4. 集成命令与适配侧原始事件 Owner 矩阵

| 事件 | 发布者 | 下游消费者 | 说明 |
| --- | --- | --- | --- |
| `notification.send_requested` | 业务域服务 | `notification-service` | 集成命令事件，请求通知域创建并投递通知 |
| `broker_session.updated` | `securities-trading-service` | `trade-gateway-service` | 券商原始会话更新，不对外作为统一交易事实 |
| `broker_account.validated` | `securities-trading-service` | `account-service` | 券商账户能力与校验结果输入事件 |
| `exchange_session.updated` | `crypto-trading-service` | `trade-gateway-service` | 交易所原始会话更新，不对外作为统一交易事实 |
| `exchange_credential.validated` | `crypto-trading-service` | `account-service` | 原始能力校验结果，由账户域转为能力投影事件 |
| `execution.report_received` | 适配域 | `trade-gateway-service` | 原始执行回报进入统一交易状态机 |

## 5. 引用字段约束

| 字段 | 含义 | Owner | 约束 |
| --- | --- | --- | --- |
| `account_id` | 统一交易账户 ID | `account-service` | 其他服务只能引用，不重建主数据 |
| `instrument_id` | 标的统一唯一标识 | `market-service` | 交易、策略、事件链路统一引用，不重复生成 |
| `trading_session_id` | 统一交易会话 ID | `trade-gateway-service` | 适配域不得自行生成 |
| `default_trading_session_id` | 默认交易会话引用 | `account-service` | 必须引用已存在且状态为 `AVAILABLE` 的统一交易会话 |
| `channel_order_id` | 外部通道订单号 | 适配域 | 统一交易域仅作映射引用 |
| `risk_event_id` | 风险事件 ID | `audit-risk-service` | 交易/策略域仅作关联引用 |
| `workflow_id` | 工作流实例 ID | 对应工作流 owner | 不作为业务主事实唯一标识 |

## 6. 使用规则

- 一个标准业务事实只能有一个标准事件发布者。
- 适配域可以发布原始事件，但不能越级发布统一领域标准事件。
- 引用字段允许跨服务传播，但引用不代表主权转移。
- 默认交易会话跨域写入必须通过账户域显式命令接口或账户域消费标准事件完成，不允许交易域直写账户库。
- 新增主数据或事件类型时，必须先补充本矩阵，再落具体模块文档。
