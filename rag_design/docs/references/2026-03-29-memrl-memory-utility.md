# MemRL（MemTensor）：运行时情景记忆上的非参数自进化

> **非规范文档。** 本文是外部项目调研，**不定义** EvoPalantir / `rag_design` / `knowledge/` 的正式行为。涉及本仓库的段落仅为 **观察**，不构成设计决策；正式采纳须经评审。

---

## 元数据（必填）

| 字段 | 填写 |
|------|------|
| **日期** | 2026-03-29 |
| **作者/角色** | gpr（调研） |
| **原项目** | MemRL: Self-Evolving Agents via Runtime Reinforcement Learning on Episodic Memory · [arXiv:2601.03192](https://arxiv.org/abs/2601.03192) |
| **仓库 URL** | https://github.com/MemTensor/MemRL |
| **基线 commit（强制）** | `7c3beba1244d707b74207b494be254f391819e09`（2026-03-29 对 `git ls-remote … HEAD` 的快照；非论文配套 tag） |
| **检索与阅读记录（强制）** | arXiv 摘要与版本信息（v2，2026-02-12）；仓库根 `README.md`（Abstract、Installation、Running the 4 Benchmarks、Project Layout） |

**基线 commit 直链：**  
https://github.com/MemTensor/MemRL/tree/7c3beba1244d707b74207b494be254f391819e09

---

## 一句话结论

MemRL 在 **不更新模型权重** 的前提下，用 **情景记忆 + 强化学习信号** 做运行时进化；论文强调 **两阶段检索** 过滤噪声，并用环境反馈识别 **高效用** 策略（与纯语义 RAG 区分）。

---

## 1. 问题域

在 **持续任务流** 中，Agent 需要 **从过往交互中学习** 又避免 **微调成本与灾难性遗忘**；纯记忆检索若仅靠语义相似，易召回噪声。与 EvoPalantir 校正记忆链路的关系：**案例沉淀、检索与效用/精排** 可与 [现实对齐模拟方案](../../../knowledge/基于%20EvoPalantir%20的现实对齐模拟方案.md) §4.8–4.12 及 [校正记忆引擎宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) 中 **RetrieveModule（两阶段检索）**、**GovernanceModule（效用升权/降权）** 对照阅读（仅关联，不升格为规范）。

---

## 2. 对象界定

- **是：** MemTensor 发布的 **MemRL** 论文与官方实现（Python 包 `memrl`），面向 **HLE、BigCodeBench、ALFWorld、Lifelong Agent Bench** 等基准的运行时记忆强化学习叙事。
- **不是：** 其他缩写相近但无关的工作（调研时须在元数据写明论文全题与作者，避免与无关「MEMRL」混名）。
- **与源笔记对齐：** [自进化Agent…](../自进化Agent：经验写回的运行时记忆闭环机制/自进化Agent：经验写回的运行时记忆闭环机制.md) §二.1 的「意图–经验–效用」「两阶段检索」「Q/效用更新」为 **中文笔记摘要**；细节须以论文 PDF 为准（本文未逐式核对公式与算法伪代码）。

---

## 3. 机制拆解（事实陈述）

### 3.1 论文摘要层（一手，arXiv）

- **非参数进化：** 通过 **情景记忆上的强化学习** 进化，**解耦** 稳定推理与可塑记忆。
- **两阶段检索（Two-Phase Retrieval）：** 用于 **降噪** 并借环境反馈识别高效用策略。
- **实验声称：** 在 HLE、BigCodeBench、ALFWorld、Lifelong Agent Bench 上相对强基线有提升；并强调 **稳定性–可塑性** 权衡与 **无权重更新** 的持续改进。（来源：arXiv 摘要。）

### 3.2 开源仓库布局（基线 README）

- **包结构：** `memrl/` 为主库（README 写明含 MemoryService、runners、providers、tracing）；`run/` 为四条基准入口脚本；`configs/` 为 YAML；`3rdparty/` 为 vendored 基准。
- **运行：** 各 runner 将日志写入 `logs/`、结果写入 `results/`（可由配置调整）；需配置 LLM 与 embedding 的 API（`configs/*.yaml`）。

---

## 4. 与 EvoPalantir 既定设计的对比（事实对齐）

| 维度 | 外部方案 | 本仓库（出处） |
|------|----------|----------------|
| 记忆形态 | 情景记忆 + RL 信号驱动的选择与更新（论文表述） | `CaseRecord` + 可选 `utility_score`（[宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) §2、§1.3 Retrieve/Governance） |
| 检索 | 两阶段检索、强调效用/反馈（论文） | **两阶段检索（粗召回 + 精排/效用）**（宪章 §1.3 RetrieveModule） |
| 权重更新 | 明确 **不做** 参数微调（论文摘要） | CME **不** 做数值校正、**不** 替代 `calibration_engine`（宪章 §1.1–1.2） |
| 基准域 | 通用 Agent 基准（代码/具身/终身等） | 仿真–现实校正与报告/推演消费 `RetrievalPack`（宪章 §1.2、知识库 §4.12） |

---

## 5. 重要源码坐标（若有仓库）

| 主题 | 路径 | 备注 |
|------|------|------|
| 主库 | `memrl/` | README 列 MemoryService、tracing 等 |
| 基准入口 | `run/run_hle.py`、`run/run_bcb.py`、`run/run_alfworld.py`、`run/run_llb.py` | 对应四条基准 |
| 配置 | `configs/rl_*_config.yaml` | API 与实验输出目录等 |
| 第三方基准 | `3rdparty/` | BigCodeBench、LifelongAgentBench 等 |

---

## 6. 文档 vs 源码差异（若有）

- 论文中的 **算法细节、超参与两阶段检索的精确定义** 未在本文档逐条对照源码验证；若实现与文中有出入，应以 **基线 commit 源码** 与 **论文** 交叉核对后更新本节。

---

## 7. 观察：对校正记忆（CME）的启示

> **非规范性观察。** 不构成设计决策；采纳须经 spec/评审。

### 7.1 可借鉴

- **显式「效用/反馈」与检索结合**，与宪章中精排、治理侧效用字段方向一致，可作为讨论 **Retrieve + Governance** 闭环时的外部参照。
- **「稳定推理 / 可塑记忆」解耦** 与 CME「不改主校正逻辑、外挂记忆」哲学相近，便于对外说明边界。

### 7.2 应保持的差异化（不盲从）

- EvoPalantir 记忆锚点在 **校正 run / 案例 / 报告契约**，而非通用代码 Agent 基准；**不得** 因 MemRL 的 RL 叙事而模糊 **TraceSink vs CaseStore** 分离（宪章 §1.2）。
- **治理必须异步、非阻塞**（宪章 §1.2）；外部基准 runner 的同步重流程不能直接映射为生产 tick 路径。

### 7.3 明确不做

- 不在本仓库复刻 MemRL 全套基准与训练环；不将 **未读论文细节** 写进内部规范。

---

## 8. 参考来源

- 论文：[arXiv:2601.03192](https://arxiv.org/abs/2601.03192)（v2，2026-02-12）
- 仓库（基线）：https://github.com/MemTensor/MemRL/tree/7c3beba1244d707b74207b494be254f391819e09
- 源笔记（二手归纳，机制须回论文核对）：[自进化Agent：经验写回的运行时记忆闭环机制.md](../自进化Agent：经验写回的运行时记忆闭环机制/自进化Agent：经验写回的运行时记忆闭环机制.md) §二.1；二手专栏链接见 [2026-03-29-zhihu-csdn-memory-loop-survey-sources.md](./2026-03-29-zhihu-csdn-memory-loop-survey-sources.md)
