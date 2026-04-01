# ExpeL：从多条经验中抽取可复用自然语言规则

> **非规范文档。** 本文是外部项目调研，**不定义** EvoPalantir / `rag_design` / `knowledge/` 的正式行为。涉及本仓库的段落仅为 **观察**，不构成设计决策；正式采纳须经评审。

---

## 元数据（必填）

| 字段 | 填写 |
|------|------|
| **日期** | 2026-03-31 |
| **作者/角色** | sfc（调研） |
| **原项目** | ExpeL: LLM Agents Are Experiential Learners · [arXiv:2308.10144](https://arxiv.org/abs/2308.10144) |
| **仓库 URL** | https://github.com/LeapLabTHU/ExpeL |
| **基线 commit（强制）** | `e41ec9a24823e7b560c561ab191441b56d9bcefc`（2026-03-31 对 GitHub `commits/main` 页面最新提交的快照） |
| **检索与阅读记录（强制）** | arXiv 摘要与版本信息（v3，2024-12-20）；GitHub README、目录树与 `commits/main` 页面；未逐实验脚本复现 |

**基线 commit 直链：**  
https://github.com/LeapLabTHU/ExpeL/tree/e41ec9a24823e7b560c561ab191441b56d9bcefc

---

## 一句话结论

ExpeL 对 CME 最有价值的部分不是“agent 训练本身”，而是 **从多条经验中抽取自然语言 insight，并在推理时回忆这些 insight**；这与 `GovernanceModule -> RuleBook` 的设计高度对齐。

---

## 1. 问题域

ExpeL 关心的是：当底层大模型权重不可得或不宜 finetune 时，agent 如何通过 **经验收集、自然语言 insight 抽取、推理时回忆** 来持续变好。这和 CME 的重合点主要在 **规则蒸馏与经验治理**，而不在上游实验环境本身。

---

## 2. 对象界定

- **是：** 经验收集、经验抽取、insight 提炼、推理时回忆。
- **不是：** run 级留痕、结构化索引后端、版本化 RetrievalPack 适配层。
- **与源笔记对齐：** 宪章已把 ExpeL 作为 Governance 的核心采用机制之一，可与 [校正记忆引擎宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) §9 对照阅读。

---

## 3. 机制拆解（事实陈述）

### 3.1 论文摘要层（arXiv）

- **不依赖参数更新：** 论文强调在权重不可得或不适合 finetune 的条件下，仍可从 experience learning 中获益。
- **双阶段：** 先 gather experiences，再 extract knowledge / insights。
- **推理回忆：** inference 时同时回忆提炼出的 insight 与过去经验。
- **经验越多效果越稳：** 摘要强调随着经验积累，ExpeL agent 的性能持续提升。

### 3.2 开源仓库布局（基线 README）

- **关键目录：** `agent/`、`memory/`、`prompts/`、`configs/`、`envs/`
- **关键脚本：** `train.py`、`insight_extraction.py`、`eval.py`
- **benchmark 环境：** README 明确列出 HotpotQA、ALFWorld、WebShop、FEVER

---

## 4. 与 EvoPalantir 既定设计的对比（事实对齐）

| 维度 | 外部方案 | 本仓库（出处） |
|------|----------|----------------|
| 经验沉淀 | 从 experience 中提取自然语言 insights | 从多条 `CaseRecord` 蒸馏 `RuleBook`（[宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) §9.2） |
| 推理使用 | inference 时回忆 insight + past experiences | Retrieve 返回 case + `rulebook_excerpts[]`（宪章 §2.1、§8） |
| 处理方式 | 训练/评估闭环中的一个独立阶段 | Governance 异步批处理，不阻塞主链路（宪章 §9.3、§9.5） |
| 输出形态 | 自然语言 insight | `RuleBook` 条目 + artifact 导出（宪章 §2.1、§9.3） |
| 自动应用 | 主要为推理辅助 | CME V1 禁止隐式改 `calibration_engine` 参数（宪章 §9.5、§12.1） |

---

## 5. 重要源码坐标（若有仓库）

| 主题 | 路径 | 备注 |
|------|------|------|
| 经验收集 | `train.py` | gather experience 阶段 |
| insight 提取 | `insight_extraction.py` | 最贴近 RuleBook 蒸馏 |
| 推理/评估 | `eval.py` | recall 后的评估入口 |
| 记忆相关 | `memory/` | 经验与 insight 管理核心目录 |
| 提示模板 | `prompts/` | insight 提取与调用提示 |

---

## 6. 文档 vs 源码差异（若有）

- README 清楚展示了 ExpeL 的“三段式流程”，但并未把它抽成一个通用微服务；工程上仍是 benchmark-first 的研究仓库。
- CME 若借鉴 ExpeL，重点应放在 **insight extraction / rule distillation**，而不是照搬训练与评测脚本。

---

## 7. 观察：对校正记忆（CME）的启示

> **非规范性观察。** 不构成设计决策；采纳须经 spec/评审。

### 7.1 可借鉴

- **将多条 case 批量蒸馏成规则条目**，这正好对应 `GovernanceModule` 中的 `RuleBook` 更新。
- **把规则与历史案例同时暴露给下游**，有助于报告生成时同时给出“经验例证 + 提炼结论”。
- **经验收集与 insight 抽取阶段分离**，有利于维持异步、非阻塞治理。

### 7.2 应保持的差异化（不盲从）

- ExpeL 的对象是 agent benchmark；EvoPalantir 的对象是 **校正事件后的案例治理**。
- ExpeL 更重“训练阶段经验积累”；CME 更重“线上案例资产化 + 下游消费契约”。
- 宪章要求 `RuleBook` 仅用于辅助诊断/报告/检索特征，不能直接改主公式，这一点必须比 ExpeL 的实验叙事更保守。

### 7.3 明确不做

- 不把 `train.py / eval.py` 式实验循环引入 `CMECoordinator`。
- 不在 V1 中做 “insight 自动应用到校正引擎参数”。
- 不把自然语言 insight 代替结构化 case 特征。

---

## 8. 参考来源

- 论文：[arXiv:2308.10144](https://arxiv.org/abs/2308.10144)
- 仓库（基线快照）：https://github.com/LeapLabTHU/ExpeL/tree/e41ec9a24823e7b560c561ab191441b56d9bcefc
- 本仓库设计约束：[校正记忆引擎宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md)
