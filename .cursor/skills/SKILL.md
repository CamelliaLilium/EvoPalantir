---
name: evopalantir-rag-workflow
description: EvoPalantir 校正记忆/rag_design 文档工作流、目录约定与规则索引（随 gpr 要求更新）
---

# EvoPalantir · rag_design 工作流（项目内技能）

## 仓库结构（高层）

| 路径 | 用途 |
|------|------|
| `knowledge/` | **事实与边界来源**；架构、现实对齐方案等锚点文档 |
| `rag_design/` | RAG / 校正记忆 / 经验库 **设计与需求**；`.cursor/rules/` 下项目规则 |
| `rag_design/drafts/` | **草稿**（迭代中） |
| `rag_design/docs/plans/` | **成稿**（计划与设计文档定版存放） |
| 其他如 `MemOS/`、`EvoSim/`、`EPFrameWork1/` | **实现与参照**；设计阶段 **仅逻辑接轨**，具体路径与 API 在实现计划再挂钩 |

## 文档流水线

1. **需求拷问** → 整理《需求说明》（gpr + sfc 共识）；见 `rag_design/docs/需求说明-校正记忆与经验库.md`。
2. **调研与设计** → 先在 `rag_design/drafts/` 撰写，稳定后 **成稿** 入 `rag_design/docs/plans/`。
3. **编码** → 《调研与设计》定稿后，用 Superpowers **`writing-plans`** 写实现计划（勿用已弃用的 `/write-plan`）。
4. 动笔前若信息不足：**先向 gpr 问清**，避免臆造。

## 规则索引（`rag_design/.cursor/rules/`）

- `rag-design-range.mdc` — 范围、权威来源、禁止编造 API  
- `rag-design-doc-audience.mdc` — 《需求说明》与《调研与设计》读者与协作  
- `rag-design-principles.mdc` — 清晰可落地、凝练、模块解耦  
- `plan.mdc` — 先需求文档再计划，勿默认需求已知  

## 设计原则（摘要）

逻辑严密、可实施、简洁、**模块通过契约解耦**；冗长形式化目录避免。

## 维护

gpr 新增流程或路径时 **更新本 SKILL**；与 `knowledge/` 冲突时以 `knowledge/` 为准。
