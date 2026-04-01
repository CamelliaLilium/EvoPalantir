# MLflow Tracking：运行留痕与实验审计基座

> **非规范文档。** 本文是外部项目调研，**不定义** EvoPalantir / `rag_design` / `knowledge/` 的正式行为。涉及本仓库的段落仅为 **观察**，不构成设计决策；正式采纳须经评审。

---

## 元数据（必填）

| 字段 | 填写 |
|------|------|
| **日期** | 2026-03-31 |
| **作者/角色** | Codex（调研） |
| **原项目** | MLflow Tracking · [官方文档](https://mlflow.org/docs/latest/ml/tracking/) |
| **仓库 URL** | https://github.com/mlflow/mlflow |
| **基线 commit（强制）** | `005b959cacda05d1423356cfcbd9ebeda8ff96a7`（2026-03-31 对 GitHub `commits/master` 页面最新提交的快照；非 release tag） |
| **检索与阅读记录（强制）** | 官方 Tracking 文档（latest，2026-03-31 抓取）；GitHub 根 README / 仓库树 / `commits/master` 页面；未逐文件核对全部后端实现 |

**基线 commit 直链：**  
https://github.com/mlflow/mlflow/tree/005b959cacda05d1423356cfcbd9ebeda8ff96a7

---

## 一句话结论

MLflow Tracking 非常适合作为 **TraceSink / Run 级留痕** 的默认实现，但它不是语义案例库，也不应该承担 `CaseStore + Index + RetrievalPack` 的职责。

---

## 1. 问题域

MLflow Tracking 解决的是 **运行过程的可追踪性、审计性与可对比性**：把一次运行记录为 run，并围绕 run 持久化参数、指标、标签、产物与元数据，然后支持 UI 与 API 检索。对 EvoPalantir 的相关性在于：它和 [校正记忆引擎宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) 中的 **TraceSinkModule** 几乎一一对位，但并不覆盖 **案例语义存储与检索**。

---

## 2. 对象界定

- **是：** MLflow Tracking 的 run / experiment / artifact / backend-store / API / UI 这一整套留痕体系。
- **不是：** 语义 case store、向量索引、版本化下游契约、规则治理引擎。
- **与源笔记对齐：** 宪章已把 MLflow 指定为 V1 默认留痕载体，可与 [校正记忆引擎宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) §4 和 §12.1 对照阅读。

---

## 3. 机制拆解（事实陈述）

### 3.1 官方文档层（官方 docs）

- **核心概念：** Tracking 以 **runs** 为中心；run 记录 metadata、metrics、parameters、artifacts，并按 **experiments** 组织。
- **记录 API：** 官方文档明确给出 `mlflow.start_run()`、`mlflow.log_param()`、`mlflow.log_metric()`、`mlflow.log_artifact()` 等接口。
- **检索能力：** 文档给出 `MlflowClient.search_runs()` 等程序化查询方式，适合做 run 级调试与审计。
- **存储分离：** 文档区分 **backend store**（run 元数据）与 **artifact store**（大对象产物），这与 CME 中 “params/metrics/tags” 与 “case JSON / 导出文件” 分开落盘的思路一致。

### 3.2 开源仓库布局（基线仓库）

- **主库：** `mlflow/`
- **Tracking 相关代码：** `mlflow/tracking/client.py`
- **Tracking store：** `mlflow/store/tracking/file_store.py`
- **测试与回归：** `tests/`
- **文档：** `docs/`

---

## 4. 与 EvoPalantir 既定设计的对比（事实对齐）

| 维度 | 外部方案 | 本仓库（出处） |
|------|----------|----------------|
| 记录对象 | run / experiment / artifacts | `CalibrationRunRef` + TraceSink 留痕（[宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) §2.1、§4） |
| 可记录内容 | params、metrics、tags、artifacts、start/end time | `applied_rules`、`hyperparams`、误差指标、case JSON、rulebook 导出物（宪章 §4.3） |
| 检索能力 | `search_runs`、UI 对比、按指标/参数过滤 | 人读调试、审计与 run 对照；**不等于** 相似案例检索（宪章 §4.5、§12.1） |
| 主存语义 | 以 run 为中心 | case 语义真源必须在 `CaseStore`，不能退化成 MLflow-only（宪章 §1.2、§4.5） |
| 下游消费 | 可直接看 UI / API | 下游只认 `RetrievalPack`，不能直接绑 MLflow 内部结构（宪章 §1.2、§2.1、§10） |

---

## 5. 重要源码坐标（若有仓库）

| 主题 | 路径 | 备注 |
|------|------|------|
| Tracking client | `mlflow/tracking/client.py` | 程序化查询与操作入口 |
| Tracking store | `mlflow/store/tracking/file_store.py` | 文件后端实现的代表路径 |
| 主包 | `mlflow/` | Tracking / model / store 等核心模块 |
| 测试 | `tests/` | 适合后续查 run / store 行为回归 |
| 文档 | `docs/` | 官方 docs 对实现行为的说明入口 |

---

## 6. 文档 vs 源码差异（若有）

- 官方 Tracking 文档覆盖面比 CME 当前需要的能力更大，包含 model tracking、dataset tracking、UI 等扩展能力；而 CME V1 需要的主要是 **run 级留痕**。
- 本文未逐项核对 `backend store` 的所有实现差异（文件、本地 DB、远端服务等）；若未来要锁定具体部署方式，仍需按基线 commit 继续落到实现级别。

---

## 7. 观察：对校正记忆（CME）的启示

> **非规范性观察。** 不构成设计决策；采纳须经 spec/评审。

### 7.1 可借鉴

- **run 作为审计单位** 很适合对应一次 “校正完成” 事件，便于回答“这次为什么这样调参/校正”。
- **params / metrics / artifacts 分层** 很适合把超参、误差指标与大对象导出分开处理。
- **programmatic search + UI** 很适合支撑调试与报告回溯，而不要求下游直接读原始日志。

### 7.2 应保持的差异化（不盲从）

- EvoPalantir 需要严格区分 **Run 留痕** 与 **Case 语义真源**；MLflow 只应落在前者，不能替代 `CaseStore + Index`。
- 宪章要求下游只认 `RetrievalPack`，因此不能让 forecast/report 直接依赖 MLflow 的 run schema 或 UI 结构。
- MLflow 的“模型/数据集跟踪”能力对 CME 是旁支，不应反向牵动主契约设计。

### 7.3 明确不做

- 不把 MLflow 当作相似案例检索引擎。
- 不让 `RetrieveModule` 内嵌 MLflow 客户端作为主数据通路。
- 不把 `RetrievalPack` 退化成“前端读取 MLflow run 后临时拼装的视图”。

---

## 8. 参考来源

- 官方文档：[MLflow Tracking](https://mlflow.org/docs/latest/ml/tracking/)
- 官方仓库（基线快照）：https://github.com/mlflow/mlflow/tree/005b959cacda05d1423356cfcbd9ebeda8ff96a7
- 本仓库设计约束：[校正记忆引擎宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md)
