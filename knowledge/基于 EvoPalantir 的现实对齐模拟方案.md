# 基于 EvoPalantir 的现实对齐模拟方案

```markdown
# 基于 EvoPalantir 的现实对齐模拟方案

## 1. 方案定位

本方案面向一个明确目标：

> 构建一个“模拟对齐现实”的项目，核心流程为：
> 单个外部新闻引入
> -> 实时抓取现实世界中的重点用户状态
> -> 推荐系统向模拟用户发评论
> -> 在中间时刻保存快照，并抓取现实情况进行校正
> -> 继续推演未来

这个项目的本质不是做一个泛化的“用户画像平台”，而是做一个：

- 事件驱动
- 动态状态驱动
- 可快照
- 可校正
- 可推演

的社会模拟后端。

---

## 2. 目标与系统闭环

### 2.1 原始任务目标

你的原始目标链路可以写成：

1. 引入单条现实新闻
2. 抓取现实平台上的重点参与者与讨论
3. 让推荐系统决定模拟环境中的曝光
4. 让 actor 基于曝光产生行为
5. 在中间时刻保存快照
6. 用现实观测校正模拟状态
7. 在校正后的状态上继续推演未来

### 2.2 系统闭环翻译

把这个目标翻译成系统闭环，就是：

```text
新闻输入
  -> 事件建模
  -> 现实观测
  -> 状态初始化
  -> 曝光分发
  -> 行为推进
  -> 快照保存
  -> 现实校正
  -> 未来推演
```

### 2.3 核心判断

这个项目最重要的设计决定有三个：

- 不做静态 persona 系统，要做动态 `ActorState`
- 推荐系统不是商业推荐器，而是模拟环境中的曝光分发机制
- 快照与校正不是附属功能，而是整个系统的中心

---

## 3. EvoSim 应保留的内核

### 3.1 必须保留的部分

- `EvoSim/src/simulation.py`

  - 用途：tick 驱动的仿真主循环
  - 原因：你的项目天然需要“推进 -> 快照 -> 校正 -> 再推进”的循环
- `EvoSim/src/snapshot_manager.py`

  - 用途：快照保存、恢复、回放
  - 原因：你的闭环中“中间时刻保存快照并校正”是核心能力
- `EvoSim/src/recommender/`

  - 用途：候选内容来源、打分、筛选、feed pipeline
  - 原因：你仍然需要一个结构化的曝光分发管线
- `EvoSim/src/database_manager.py`

  - 用途：SQLite 持久化
  - 原因：事件状态、ActorState、Exposure、Snapshot 都要落库并支持回放
- `EvoSim/src/user_manager.py`

  - 用途：运行时 actor 的加载、恢复与组装
  - 原因：你仍然需要 actor 容器层，只是状态定义要改

### 3.2 保留结构、修改语义的部分

这些部分不能原样继承，但结构层值得保留：

#### recommender

- 保留：`sources / scorers / selectors / feed_pipeline`
- 修改：沿用原推荐框架，但把候选内容改成“新闻内容 + 相关用户评论”（不要用户发帖），并继续分发给其他用户

#### snapshots

- 保留：tick 级保存和恢复机制
- 修改：快照不再只是恢复点，而是“校正节点”

#### injection

- 保留：注入作为系统入口的结构位置
- 修改：新闻注入和角色画像注入都支持两种方式
  - `静态注入`：直接使用整理好的新闻、角色和时间片数据
  - `skill 注入`：使用 `新闻抽取 skill` 和 `重点人物特点提取 skill` 自动生成注入内容

### 3.3 应从主运行路径剥离的部分

以下模块会把新项目拖回原来的“舆论攻防”语义里，应从主路径剥离：

- `EvoSim/src/malicious_bots/`
- `EvoSim/src/opinion_balance_manager.py`
- 旧的 defense / intervention agent 流程
- `EvoSim/src/persona_manager.py` 中以静态 persona 为中心的逻辑
- 默认假设“攻击-反制-平衡”实验场景的旧新闻注入逻辑

### 3.4 保留内核后的目标形态

裁剪版保留的是：

- 仿真内核
- 推荐内核
- 快照内核
- 持久化内核

裁剪版不应继续保留的是：

- 与旧产品绑定的 UI / workflow 语义
- 与攻防实验强绑定的业务模块

---

## 4. 为实现目标需要补齐的能力

### 4.1 目标到模块的映射

把你的目标流程映射成系统模块，就是：

1. `event_ingestion`
2. `real_world_observer`
3. `actor_state_service`
4. `recommender` 作为新闻与评论的分发模块
5. `actor_behavior_model`
6. `snapshot_service`
7. `calibration_engine`
8. `calibration_memory_engine`
9. `forecast_runtime`

### 4.2 核心对象

建议新系统初期只维护 4 个一等对象：

#### Event

一条新闻对应一个事件对象。

字段建议：

- `event_id`
- `source_url`
- `title`
- `published_at`
- `claims_json`
- `entities_json`
- `topic_tags_json`
- `phase`
- `seed_query`

#### ActorState

不是 persona，而是“账号在某个事件下的动态状态”。

字段建议：

- `actor_id`
- `platform`
- `event_id`
- `stance_posterior_json`
- `activity_hazard`
- `influence_score`
- `bridge_score`
- `topic_mix_json`
- `uncertainty`
- `last_observed_at`

#### Exposure

记录某个 tick 时刻，哪些内容被分发给了哪个 actor。

字段建议：

- `event_id`
- `tick`
- `actor_id`
- `content_id`
- `source_type`
- `rank_score`
- `feed_bucket`

#### Snapshot

记录某个模拟时点的冻结状态及现实观测上下文。

字段建议：

- `snapshot_id`
- `event_id`
- `tick`
- `sim_state_path`
- `obs_summary_json`
- `prediction_summary_json`
- `error_metrics_json`

### 4.3 事件输入层：`event_ingestion`

作用：
把新闻 URL 或文本变成结构化事件。

职责：

- 拉取或接受原始新闻文本
- 提取 claims、entities、topic tags
- 标记初始事件阶段
- 生成 `seed_query`

`seed_query` 的 V1 规则建议：

1. 从标题抽取核心实体
2. 从 claims 提取 3 到 5 个关键词
3. 拼接时间约束与平台上下文词
4. 生成 1 条主查询和 2 到 3 条扩展查询

原则：

- 先规则模板
- 后续再引入 LLM 做 query expansion
- 不把观测召回完全绑定到模型调用

### 4.4 现实观测层：`real_world_observer`

作用：
抓取现实平台上的公开讨论、账号公开信息、交互边。

统一适配器接口：

- `collect_event_content(event_query)`
- `collect_actor_public_meta(actor_id)`
- `collect_actor_history(actor_id, topic_filter, window)`
- `collect_interaction_edges(content_ids)`

MVP 平台顺序建议：

1. Reddit
2. YouTube
3. X，仅在 API 和政策边界明确后接入

### 4.5 状态层：`actor_state_service`

作用：
把现实观测转成可计算的事件状态。

设计原则：

- 跟踪“状态”，不是“人格”
- 只做平台内 actor，不默认跨平台身份合并
- 采用两层人群模型：`leaders` + `crowd`，先保证闭环可跑，再逐步扩展规模

V1 人群规模与更新策略（讨论结论）：

- `total_users = 30`
- `leaders = 8`（每个 tick 强更新）
- `crowd = 22`（每 5 个 tick 更新一次，或在高曝光触发时更新）
- 维护 `state_version` 与 `updated_at_tick`，保证快照和校正对齐到同一时间步

`ActorState` 初始化规则建议：

- `stance_posterior`

  - 基于第一波观测内容做 `support / oppose / neutral / unclear` 分类
  - 按计数归一化初始化
- `activity_hazard`

  - 基于最近窗口发言频率初始化
  - 可用“近 24 小时发言次数 + 近 7 天活跃天数 + 事件参与密度”组合
- `influence_score`

  - 基于 reply / quote / mention / upvote 等互动量归一化
- `bridge_score`

  - 基于事件局部互动图的 betweenness centrality 或跨社区边占比计算
- `topic_mix`

  - 基于近窗历史内容做 tag 聚合或 TF-IDF 聚类
- `uncertainty`

  - 与证据覆盖度绑定
  - V1 可先用 `1 / (1 + n_obs)` 这类简单函数

### 4.6 曝光层：`recommender`

作用：
沿用原来的推荐框架，把新闻内容和用户评论继续推送给其他用户。

V1 的三类 feed bucket：

- `similar_feed`
- `cross_cutting_feed`
- `trending_feed`

默认混合比例建议：

- `similar_feed_weight = 0.5`
- `cross_cutting_feed_weight = 0.2`
- `trending_feed_weight = 0.3`

基于两层人群模型的评论分发优先级（V1）：

- `leader_comment_share = 0.7`
- `crowd_comment_share = 0.3`
- 当前 tick 的领袖评论优先进入推荐候选池，再补充普通用户评论

后续可以按状态做轻量动态修正：

- `bridge_score` 高 -> 增加 `cross_cutting_feed`
- `activity_hazard` 高 -> 提高 `trending_feed`
- `stance_posterior` 很集中 -> 控制 `cross_cutting_feed` 的突变幅度

第一阶段的候选内容建议只包含两类：

- 注入系统的新闻内容
- 用户围绕新闻产生的评论内容

也就是说，推荐模块在第一阶段做的是：

- 把新闻推给一部分用户
- 把用户对新闻的评论继续推给其他用户
- 让评论在用户之间继续传播和触发新评论

### 4.7 行为层：`actor_behavior_model`

作用：
把 exposure 转成行为反应。

V1 建议采用规则驱动模型：

1. 先判断是否行动
   - 由 `activity_hazard + exposure_volume + exposure_alignment` 决定
2. 再判断评论类型
   - `bridge_score` 高 -> 更倾向跨群体回应
   - 立场一致 -> 更倾向附和式评论
   - 立场冲突 -> 更倾向反驳式评论或沉默
3. 最后选择目标内容
   - 从当前 exposure set 中按 bucket 和 rank 选取

第一阶段不要求用户发帖，用户只需要对新闻或其他用户评论进行评论。

MVP 不要求精确生成长文本，先输出：

- 是否行动
- 评论类型
- 目标对象

### 4.8 快照层：`snapshot_service`

作用：
保存中间状态，支持回放与校正。

每个快照建议保存：

- 模拟数据库副本
- 当前事件状态摘要
- Top N actor states
- 当前曝光摘要
- 最新现实观测摘要
- 观测前的预测结果
- 对比后的误差指标

### 4.9 校正层：`calibration_engine`

作用：
把模拟状态重新拉回现实。

V1 只校正：

- 立场分布
- 极化程度
- 关键节点活跃度
- 传播结构指标

字段级校正规则建议：

- 分布类变量

  - `X_calibrated = alpha * X_simulated + (1 - alpha) * X_observed`
- 概率类变量

  - `p_new = lambda * p_old + (1 - lambda) * p_obs`
- 图结构类变量

  - 先基于最新观测图重算，再与旧值平滑融合
- 曝光权重

  - 漂移明显时，调整 bucket 权重
  - 不直接覆盖 actor state

原则：

- 先显式规则
- 再考虑更复杂模型
- 不在 MVP 阶段引入黑盒校正器

### 4.10 校正记忆层：`calibration_memory_engine`

作用：
记录每次“预测 vs 现实”的差距，并总结可复用的经验。

核心职责：

- 保存每次校正前后的误差情况
- 记录本次采用的校正规则和效果
- 按事件类型、阶段、误差模式做经验归档
- 在下一次预测前提供相似案例参考

第一阶段不直接自动修改主逻辑，而是先做：

- 经验记录
- 相似经验检索
- 经验总结

### 4.11 推演层：`forecast_runtime`

作用：
在校正后的状态上继续向未来滚动推演，并可参考校正记忆中的相似案例。

输出重点应放在：

- 立场比例变化
- 极化变化
- 关键节点活跃度变化
- 传播结构变化
- 高风险传播链变化

四项观测指标口径（统一）：

- 立场分布变化：`support / oppose / neutral / unclear` 的时间片占比变化
- 极化变化：群体间立场分离度变化（可基于分布距离或极化指数）
- 关键节点活跃度变化：关键节点发言/被互动频次变化
- 传播结构变化：跨群体边占比、局部桥接度、连通性等图结构变化

### 4.12 主数据流

为了保证实现不偏离目标，主数据流固定为：

```text
新闻 URL / 文本
  -> event_ingestion
  -> Event + seed_query
  -> real_world_observer
  -> 原始观测内容 / 交互边 / actor meta
  -> actor_state_service
  -> 初始 ActorState + shortlist
  -> recommender
  -> Exposure
  -> simulation_runtime
  -> 行为结果 + 新状态
  -> snapshot_service
  -> Snapshot
  -> calibration_engine
  -> 校正后的状态
  -> calibration_memory_engine
  -> 误差经验 / 相似案例
  -> forecast_runtime
  -> 未来若干 tick 的聚合输出
```

---

## 5. 更落地的选型说明（V1）

目标：优先保证“两层用户（30人）+ 每 tick 推进 + 快照校正 + 短期推演”稳定跑通，避免过早引入重框架。

### 5.1 V1 必选

#### `GDELT`（事件输入）

- 用途：新闻事件发现、事件元数据补充
- 接入：`event_ingestion`
- 站点：https://www.gdeltproject.org/

#### `Reddit API + PRAW`（现实观测）

- 用途：抓取帖子、评论、基础用户公开信息
- 接入：`real_world_observer`
- 文档：
  - https://www.reddit.com/dev/api/
  - https://praw.readthedocs.io/

#### `NetworkX`（图指标）

- 用途：计算 `bridge_score`、局部中心性、互动图统计
- 接入：`actor_state_service`、`calibration_engine`
- 文档：https://networkx.org/documentation/stable/

#### `MLflow Tracking`（校正记忆的 V1 载体）

- 用途：记录每次“预测 vs 现实”的误差、参数、实验版本
- 接入：`calibration_memory_engine`
- 文档：https://mlflow.org/docs/latest/ml/tracking/

### 5.2 V1 可参考但不作为主框架

#### `RecSim / RecSim NG`

- 用途：推荐与曝光分发的建模思路参考
- 结论：当前阶段不作为主运行框架，仅借鉴设计思想
- 参考：
  - https://arxiv.org/abs/1909.04847
  - https://arxiv.org/abs/2103.08057
  - https://github.com/google-research/recsim

#### `Y Social`、`generative_agents`

- 用途：产品形态与行为机制灵感参考
- 结论：不引入为核心依赖，避免架构漂移
- 参考：
  - https://y-not.social/
  - https://github.com/joonspk-research/generative_agents

#### `Mem0 / LangMem`

- 用途：长期记忆检索、经验总结与反思
- 结论：MVP 先用 `MLflow` 留痕，V2 再引入记忆系统
- 参考：
  - https://docs.mem0.ai/
  - https://github.com/langchain-ai/langmem

## 6. MVP

MVP 必须收敛为：

- 一个事件
- 一个现实平台：Reddit
- 两层模拟用户：`total_users = 30`（`leaders = 8`, `crowd = 22`）
- 领袖每个 tick 更新；普通用户每 5 个 tick 或高曝光触发更新
- 至少 5 个快照
- 短期预测

MVP 必须能输出：

- 立场分布变化
- 极化变化
- 关键节点活跃度变化
- 传播结构变化

第一阶段建议顺序：

1. 先拆解 EvoSim
2. 用静态整理好的新闻和角色先跑闭环
3. 先实现 `预测 skill`、`现实校正 skill`、`校正记忆 skill`
4. 再实现 `新闻抽取 skill`、`重点人物特点提取 skill`

第一阶段的注入方式保留两个情况：

- 固定内容注入：直接使用静态整理好的新闻和角色
- skill 注入：使用 `新闻抽取 skill` 和 `重点人物特点提取 skill` 自动生成注入内容

## 7. 任务分工

1. 收集已经定性的历史新闻，并整理完整变化流程。
   包括不同时间段的新闻状态、关键用户、讨论摘要、互动关系，并整理成可注入的静态数据格式。
2. EvoSim 的拆除与保留模块重组。(2位:rcy,zxy)
   重点保留 `simulation`、`snapshot`、`recommender`、`database`，剥离 `malicious_bots`、`opinion_balance` 和旧 persona 语义，并把推荐模块重组为“新闻 + 用户评论”的继续分发逻辑。
3. 事件与人物状态抽取 skill 实现。（1位，负责调研与设计:zth）
   包括新闻抽取、时间片切分、关键用户识别，以及事件下的动态状态抽取；这里需要支持两种注入方式：一是固定内容注入，二是使用 skill 自动注入。
4. 预测 skill 实现。（1位，调研与设计:myd）
   基于校正后的状态和历史经验继续向未来推演，优先输出立场分布变化、极化变化、关键节点活跃度变化和传播结构变化。
5. 与现实校正 skill 实现。（1位，调研与设计:ln）
   基于不同时间片的现实数据，对立场分布、极化程度、关键节点活跃度和传播结构指标进行校正。
6. 校正记忆模块 / skill 实现（自进化，可参考memos、openviking这种项目）。（经验库的调研与设计，2位:gpr,sfc）
   记录每次预测与现实的差距、校正规则和效果，沉淀相似案例，辅助后续预测。
7. 端到端联调 / 验证（测试、监工，对其他同学的进度进行跟进）。
   跑通“新闻注入 -> 状态抽取 -> 推荐曝光 -> 快照 -> 校正 -> 推演”的完整链路，验证整个闭环可用。

```