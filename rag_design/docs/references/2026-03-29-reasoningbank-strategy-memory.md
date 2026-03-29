# ReasoningBank：推理记忆与记忆感知测试时扩展（MaTTS）

> **非规范文档。** 本文是外部项目调研，**不定义** EvoPalantir / `rag_design` / `knowledge/` 的正式行为。涉及本仓库的段落仅为 **观察**，不构成设计决策；正式采纳须经评审。

---

## 元数据（必填）

| 字段 | 填写 |
|------|------|
| **日期** | 2026-03-29 |
| **作者/角色** | gpr（调研） |
| **原项目** | ReasoningBank: Scaling Agent Self-Evolving with Reasoning Memory · [arXiv:2509.25140](https://arxiv.org/abs/2509.25140) |
| **仓库 URL** | https://github.com/google-research/reasoning-bank |
| **基线 commit（强制）** | `250f51e6ced33a1225a25d8bd9ed87d18a405377`（2026-03-29 `git ls-remote … HEAD`） |
| **检索与阅读记录（强制）** | arXiv 摘要（v2，2026-03-16）；仓库根 `README.md`（Overview、WebArena/SWE-Bench 目录说明、Citation） |

**基线 commit 直链：**  
https://github.com/google-research/reasoning-bank/tree/250f51e6ced33a1225a25d8bd9ed87d18a405377

---

## 一句话结论

ReasoningBank 把 Agent 的 **成功与失败轨迹** 蒸馏为 **可泛化的推理策略记忆**，测试时检索写入闭环；并提出 **memory-aware test-time scaling (MaTTS)**，用 **更多测试时交互** 与记忆形成 **双向协同**。

---

## 1. 问题域

长时在线 Agent 若不能从 **历史交互** 学习，会重复错误；需要 **可迁移的策略记忆** 而非仅堆原始轨迹。与 EvoPalantir 关联：经验 **抽象为短规则/策略** 可与 [校正记忆引擎宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) 中 **RuleBook（ExpeL 式蒸馏）**、**CaseIngest 反思** 对照（仅关联）；主数据流仍以 [现实对齐模拟方案](../../../knowledge/基于%20EvoPalantir%20的现实对齐模拟方案.md) 为准。

---

## 2. 对象界定

- **是：** Google Research 发布的 **ReasoningBank** 论文与开源代码（WebArena、SWE-Bench 两套演示路径）。
- **不是：** 通用「RAG 文档库」或单纯 trajectory DB；论文强调 **自评判成败 → 蒸馏推理记忆 → 测试时检索写回**。
- **与源笔记对齐：** [自进化Agent…](../自进化Agent：经验写回的运行时记忆闭环机制/自进化Agent：经验写回的运行时记忆闭环机制.md) §二.2 的「策略化记忆」「成败双重提炼」为 **中文笔记摘要**；条目结构与提示词级细节以论文/代码为准。

---

## 3. 机制拆解（事实陈述）

### 3.1 论文摘要层（一手，arXiv）

- **记忆内容：** 从 **自评判的成功与失败经验** 蒸馏 **可泛化推理策略**（非仅成功套路）。
- **测试时：** Agent **检索** ReasoningBank 指导交互，并把 **新学习** 写回，形成自我增强。
- **MaTTS：** 对单任务分配更多测试时算力，产生 **丰富、多样** 经验，为记忆合成提供 **对比信号**；更好记忆反过来提升 scaling 效率（摘要表述）。
- **实验域：** 论文报告在 **网页浏览与软件工程** 类基准上相对「只存原始轨迹或只存成功例程」类记忆机制有提升（摘要表述）。

### 3.2 仓库布局（基线 README）

- **WebArena：** `WebArena/agents/`、`autoeval/`、`config_files/`、`prompt/`；运行脚本 `run.sh`，并提及 `pipeline_scaling.py`、`induce_scaling.py` 用于 scaling 设置。
- **SWE-Bench：** 基于 `third_party` 中的 mini-swe-agent；`SWE-Bench/run.sh`；评估引用上游 `sb-cli` 文档。
- **免责声明：** README 写明 **非正式 Google 产品**、演示用途。

---

## 4. 与 EvoPalantir 既定设计的对比（事实对齐）

| 维度 | 外部方案 | 本仓库（出处） |
|------|----------|----------------|
| 记忆内容 | 推理策略记忆（论文）；代码侧分 Web/SWE 目录 | `CaseRecord` + 可选 `reflection_text`；**RuleBook** 短规则（宪章 §2.1、§1.3 Governance） |
| 闭环 | 测试时检索 + 任务后写回 | 校正完成后 **Ingest → CaseStore → Index**；**Governance 异步**（宪章 §1.3） |
| 算力维度 | MaTTS：测试时 scaling 与记忆协同 | EvoPalantir 强调 **治理不阻塞 tick**（宪章 §1.2），不等价于「无限测试时 rollouts」 |
| 场景 | 开放 Web/软件工程 Agent | 仿真–现实 **校正与报告**（知识库 §4.12） |

---

## 5. 重要源码坐标（若有仓库）

| 主题 | 路径 | 备注 |
|------|------|------|
| Web 智能体 | `WebArena/agents/` | 与 browsergym 集成 |
| 自动评判 | `WebArena/autoeval/` | LLM-as-judge 信号 |
| 配置与数据 | `WebArena/config_files/` | 含预处理脚本说明 |
| 提示 | `WebArena/prompt/` | 跨实现复用的指令 |
| SWE 流水线 | `SWE-Bench/`、`third_party/` | mini-swe-agent 可编辑安装 |

---

## 6. 文档 vs 源码差异（若有）

- **MaTTS** 与 **记忆条目 schema** 的字段级定义应以 **论文与代码** 为准；本文未逐文件核对 `pipeline_scaling.py` / `induce_scaling.py` 与论文图表是否一一对应。

---

## 7. 观察：对校正记忆（CME）的启示

> **非规范性观察。** 不构成设计决策；采纳须经 spec/评审。

### 7.1 可借鉴

- **失败轨迹同等进入蒸馏** 与 ExpeL/反思门禁思想一致，可支撑 **CaseIngest** 与 **RuleBook** 讨论「负例如何进规则」。
- **记忆与算力/探索的协同** 可类比「离线批处理治理 + 在线检索」的资源分配讨论（仅类比）。

### 7.2 应保持的差异化（不盲从）

- **不得** 将 WebArena/SWE 的交互 scaling 默认搬进 **校正 tick 主路径**；CME **异步治理** 红线不变。
- **Run 留痕 vs 案例语义** 分离（宪章 §1.2）与 ReasoningBank 的「策略记忆」不是同一概念，避免混谈。

### 7.3 明确不做

- 不在此仓库复现 ReasoningBank 全量环境与训练；不把 Google 演示代码当作生产依赖。

---

## 8. 参考来源

- 论文：[arXiv:2509.25140](https://arxiv.org/abs/2509.25140)（v2，2026-03-16）；ICLR 注册页见 README 中 OpenReview 引用
- 仓库（基线）：https://github.com/google-research/reasoning-bank/tree/250f51e6ced33a1225a25d8bd9ed87d18a405377
- 源笔记：[自进化Agent…](../自进化Agent：经验写回的运行时记忆闭环机制/自进化Agent：经验写回的运行时记忆闭环机制.md) §二.2；二手综述链接见 [2026-03-29-zhihu-csdn-memory-loop-survey-sources.md](./2026-03-29-zhihu-csdn-memory-loop-survey-sources.md)
