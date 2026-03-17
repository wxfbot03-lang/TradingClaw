# TradingClaw API 字段字典

## 1. 文档说明

- 本文档用于统一 `docs/详细设计/service/` 下各模块详细设计中的接口字段口径。
- 目标是避免字段命名、类型、语义、可空性在不同文档中漂移。
- 本文档覆盖 HTTP、gRPC、WebSocket、事件信封中高频公共字段，不替代模块文档中的业务场景说明。

## 1.1 相关文档

- 总体总览：`docs/详细设计/service/后端详细设计.md`
- 网关与平台基础：`docs/详细设计/service/网关与平台基础详细设计.md`
- 用户与账户：`docs/详细设计/service/用户与账户详细设计.md`
- 行情与资讯：`docs/详细设计/service/行情与资讯详细设计.md`
- 交易网关：`docs/详细设计/service/交易网关详细设计.md`
- 策略系统：`docs/详细设计/service/策略系统详细设计.md`
- 智能能力与治理：`docs/详细设计/service/智能能力与治理详细设计.md`
- 风控审计与通知：`docs/详细设计/service/风控审计与通知详细设计.md`

## 2. 命名约定

- HTTP JSON 字段统一使用 `snake_case`。
- gRPC/proto 字段建议使用 `snake_case`，生成代码时由语言侧映射。
- 布尔字段使用肯定式命名，如 `accepted`、`enabled`、`review_required`。
- 时间字段统一使用 `*_at`、`*_time`、`timestamp` 三类后缀。
- 标识类字段统一以 `_id` 结尾，引用外部系统主键时可使用 `*_ref` 或 `channel_*`。

## 3. 通用响应信封字段

| 字段 | 类型 | 是否必返 | 说明 |
| --- | --- | --- | --- |
| `request_id` | string | 是 | 请求唯一标识，贯穿 HTTP/gRPC/事件链路 |
| `code` | string | 是 | 统一结果码，成功为 `OK`，失败为业务或平台错误码 |
| `message` | string | 是 | 可读结果说明 |
| `data` | object | 是 | 业务响应体，查询为空对象，命令返回受理结果 |
| `meta` | object | 否 | 扩展元数据 |
| `meta.trace_id` | string | 否 | 链路追踪标识 |
| `meta.pagination` | object | 否 | 分页信息 |
| `meta.capability_status` | object | 否 | 能力开放状态与降级信息 |

## 4. 通用请求头字段

| 字段 | 类型 | 必填规则 | 说明 |
| --- | --- | --- | --- |
| `Authorization` | string | 鉴权接口外通常必填 | Bearer Token |
| `X-Request-Id` | string | 否 | 调用方提供的请求 ID，未传则服务端生成 |
| `X-Idempotency-Key` | string | 命令接口必填 | 幂等写操作唯一键 |
| `X-Client-Type` | string | 否 | `web`、`cli`、`service` |
| `X-Client-Version` | string | 否 | 客户端版本，用于能力矩阵判断 |

## 5. 标识字段字典

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `user_id` | string | 用户唯一标识 |
| `instrument_id` | string | 标的统一唯一标识 |
| `session_id` | string | 登录会话 ID |
| `trading_session_id` | string | 统一交易会话 ID |
| `session_ref` | string | 外部会话引用或适配层会话标识 |
| `account_id` | string | 统一交易账户 ID |
| `strategy_instance_id` | string | 用户策略实例 ID |
| `strategy_definition_id` | string | 策略模板定义 ID |
| `captcha_job_id` | string | 验证码任务 ID |
| `order_id` | string | 统一订单 ID |
| `channel_order_id` | string | 外部通道订单号 |
| `conversation_id` | string | AI 会话 ID |
| `job_id` | string | OCR 任务 ID 或异步任务 ID |
| `workflow_id` | string | 工作流实例 ID |
| `risk_event_id` | string | 风险事件 ID |
| `review_id` | string | 人工复核记录 ID |
| `notification_id` | string | 通知记录 ID |
| `audit_log_id` | string | 审计日志 ID |

## 6. 身份与账户字段字典

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `identity_type` | string | 登录身份类型，如 `email`、`phone`、`username` |
| `principal` | string | 登录主体，如邮箱、手机号、用户名 |
| `credential` | string | 登录凭据，如密码或动态认证码 |
| `access_token` | string | 统一访问令牌，HTTP Bearer、WebSocket 握手和内部会话校验均使用该值 |
| `expires_at` | string(datetime) | 过期时间 |
| `client_type` | string | 客户端类型，如 `web`、`cli`、`ai` |
| `required_scope` | array/string | 会话校验时要求的权限范围 |
| `level` | string | 用户等级 |
| `account_type` | string | 账户类型，如 `security`、`crypto` |
| `strategy_quota_total` | integer | 策略总额度 |
| `strategy_quota_used` | integer | 已占用策略额度 |
| `points_total` | number | 总积分 |
| `points_available` | number | 可用积分 |
| `binding_status` | string | 账户绑定状态 |
| `auth_session_status` | string | 登录会话状态 |
| `trading_session_status` | string | 交易会话状态 |
| `owned` | boolean | 是否归属当前用户 |
| `valid` | boolean | 校验结果是否有效 |
| `revoked` | boolean | 是否已吊销或注销成功 |
| `unbound` | boolean | 是否已解绑成功 |
| `credential_ref` | string | 密钥或凭据引用 |
| `broker_code` | string | 券商编码 |
| `account_no` | string | 外部账户号 |
| `captcha_text` | string | 验证码文本 |
| `capabilities` | object | 账户能力矩阵 |
| `default_trading_session_id` | string | 默认交易会话 ID，仅允许引用 `AVAILABLE` 的统一交易会话 |
| `set_as_default` | boolean | 是否设为默认交易会话 |
| `account_capability_status` | string | 账户能力可用状态，如 `pending_verify`、`enabled`、`disabled`、`degraded` |
| `user_summary` | object | 登录响应中的用户摘要 |
| `profile` | object | 用户资料聚合对象 |
| `quota_summary` | object | 用户额度摘要 |
| `points_summary` | object | 用户积分摘要 |

## 7. 行情与资讯字段字典

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `symbol` | string | 标的代码 |
| `symbols` | array/string | 多个标的代码集合 |
| `instrument_id` | string | 标的统一唯一标识，用于事件分区、跨域关联、缓存主键和内部引用 |
| `instrument_ids` | array/string | 多个标的统一唯一标识集合，标准批量查询优先使用 |
| `market` | string | 市场代码，如 `SSE` |
| `asset_class` | string | 资产类型，如 `SECURITY`、`CRYPTO` |
| `price` | number | 价格 |
| `quantity` | number | 数量 |
| `volume` | number | 成交量 |
| `timestamp` | string(datetime) | 数据时间戳 |
| `bar_time` | string(datetime) | K 线时点 |
| `period` | string | 行情周期，如 `1m`、`1d` |
| `start_at` | string(datetime) | 区间起始时间 |
| `end_at` | string(datetime) | 区间结束时间 |
| `data_status` | string | 数据状态，如 `fresh`、`cache_fallback`、`partial`、`unavailable` |
| `indicator_type` | string | 指标类型，如 `ma` |
| `window` | integer | 指标窗口 |
| `calculable` | boolean | 是否可计算 |
| `reason` | string | 不可计算或失败原因 |
| `news_id` | string | 资讯 ID |
| `source_id` | string | 数据源或资讯源 ID |
| `keyword` | string | 查询关键字 |
| `score` | number | 影响力评分 |
| `published_at` | string(datetime) | 发布时间 |
| `scored_at` | string(datetime) | 评分时间 |

## 8. 交易字段字典

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `side` | string | 买卖方向，如 `BUY`、`SELL` |
| `order_type` | string | 订单类型，如 `LIMIT`、`MARKET` |
| `time_in_force` | string | 订单时效，如 `DAY`、`GTC`、`IOC`、`FOK` |
| `session_scope` | string | 交易时段范围，如 `REGULAR`、`PRE_MARKET`、`POST_MARKET` |
| `accepted` | boolean | 是否被受理 |
| `cancel_requested` | boolean | 是否已提交撤单 |
| `closed` | boolean | 会话或资源是否已关闭 |
| `written` | boolean | 写入或设置动作是否成功 |
| `cleared` | boolean | 清理动作是否成功 |
| `default_trading_session_written` | boolean | 默认交易会话引用是否已成功写入账户域 |
| `status` | string | 标准化业务状态 |
| `idempotency_key` | string | 命令请求幂等键 |
| `reason_code` | string | 结构化失败原因码 |
| `filled_quantity` | number | 已成交数量 |
| `avg_fill_price` | number | 成交均价 |
| `available_quantity` | number | 可用持仓数量 |
| `cost_price` | number | 成本价 |
| `reduce_only` | boolean | 是否仅允许减仓 |
| `leverage` | number | 杠杆倍数 |
| `currency` | string | 币种 |
| `available` | number | 可用余额 |
| `frozen` | number | 冻结余额 |
| `instrument` | object | 统一标的对象 |
| `channel_order_request` | object | 通道适配层原始下单参数对象 |
| `raw_status` | string | 外部通道原始状态 |
| `adapter_session_status` | string | 适配侧会话状态，不等同于统一交易会话状态 |
| `normalized_report` | object | 标准化执行回报 |
| `broker_session_id` | string | 券商侧会话标识 |
| `asset_summary` | object | 资产汇总对象 |
| `positions` | array | 持仓集合 |

## 9. 策略字段字典

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `strategy_type` | string | 策略类型，如 `GRID`、`MARTINGALE`、`REBALANCE`、`DCA` |
| `config` | object | 策略配置对象 |
| `start_immediately` | boolean | 是否创建后立即启动 |
| `runtime_status` | string | 策略运行时状态 |
| `latest_snapshot_version` | integer | 最新快照版本 |
| `operator_reason` | string | 人工操作说明 |
| `target_action` | string | 目标动作，如 `start`、`stop`、`archive` |
| `signal_id` | string | 策略信号 ID |
| `signal` | object | 计算信号 |
| `signal_type` | string | 信号类型 |
| `plan_id` | string | 交易计划 ID |
| `trading_plan` | object | 待执行交易计划 |
| `market_snapshot` | object | 市场快照 |
| `account_snapshot` | object | 账户快照 |
| `position_snapshot` | object | 持仓快照 |
| `strategy_detail` | object | 策略详情聚合对象 |

## 10. 智能能力字段字典

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `question` | string | 用户问题 |
| `answer` | string | AI 最终回答 |
| `context_scope` | array | 需要引入的上下文范围 |
| `tool_preference` | array | 优先工具列表 |
| `tool_calls` | array | 工具调用摘要 |
| `tool_name` | string | 工具名称 |
| `model_name` | string | 模型名称 |
| `model_id` | string | 模型 ID |
| `provider_code` | string | 模型或 OCR 供应商标识 |
| `enabled` | boolean | 是否启用 |
| `capability_status` | string | 能力开放状态 |
| `image_ref` | string | 图片引用 |
| `scene` | string | OCR 场景 |
| `text` | string | OCR 识别结果 |
| `confidence` | number | OCR 置信度 |
| `capability_id` | string | 能力 ID |
| `open_version` | string | 能力开放版本 |
| `degrade_strategy` | string | 降级策略 |

## 11. 风控审计与通知字段字典

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `resource_type` | string | 被作用资源类型 |
| `resource_id` | string | 被作用资源 ID；在预创建资源场景可为空 |
| `actor_id` | string | 操作主体 ID |
| `resource_ref` | string | 资源引用组合键，风控与审计链路必填 |
| `action_type` | string | 操作类型 |
| `payload` | object | 风险或审计上下文载荷 |
| `payload_ref` | string | 审计大对象引用 |
| `allowed` | boolean | 是否允许执行 |
| `decision` | string | 风控或复核决策 |
| `review_required` | boolean | 是否需要人工复核 |
| `risk_event_id` | string | 风险事件 ID |
| `rule_code` | string | 风控规则编码 |
| `rule_version` | integer | 风控规则版本 |
| `severity` | string | 风险等级 |
| `comment` | string | 复核备注 |
| `resume_token` | string | 恢复原流程的引用令牌 |
| `notification_type` | string | 通知类型 |
| `business_key` | string | 通知业务去重键 |
| `notification_status` | string | 通知主记录状态 |
| `delivery_status` | string | 投递记录状态 |
| `target_ref` | string | 通知目标引用 |
| `channel` | string | 通知渠道 |
| `recorded` | boolean | 审计是否写入成功 |
| `delivered_at` | string(datetime) | 通知投递完成时间 |
| `created_at` | string(datetime) | 创建时间 |

## 12. 可观测字段字典

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `request_id` | string | 请求级追踪 ID |
| `trace_id` | string | 链路级追踪 ID |
| `service_id` | string | 服务身份标识 |
| `event_id` | string | 事件唯一 ID |
| `event_type` | string | 事件类型 |
| `event_version` | integer | 事件版本 |
| `occurred_at` | string(datetime) | 事件发生时间 |
| `published_at` | string(datetime) | 事件发布时间 |
| `aggregate_type` | string | 聚合根类型 |
| `aggregate_id` | string | 聚合根 ID |
| `partition_key` | string | 事件分区键 |

## 13. 枚举口径建议

### 13.1 资产类型

- `SECURITY`
- `CRYPTO`

### 13.2 订单方向

- `BUY`
- `SELL`

### 13.3 订单类型

- `LIMIT`
- `MARKET`

### 13.4 常见状态字段

- `status`：统一业务状态字段，必须使用标准化值
- `auth_session_status`：登录会话状态
- `trading_session_status`：交易会话状态
- `runtime_status`：策略运行时状态
- `binding_status`：账户绑定状态
- `capability_status`：能力开放状态

## 14. 使用规则

- 新增接口字段时，先在模块文档定义，再回补本字段字典。
- 同义字段必须统一，不允许出现 `uid`/`userId`/`user_id` 混用。
- 模块文档中如需特殊字段，仅允许在本字典未覆盖时新增，并补充说明。
- 若字段语义变更，必须同步修改相关模块文档和事件/接口契约。
