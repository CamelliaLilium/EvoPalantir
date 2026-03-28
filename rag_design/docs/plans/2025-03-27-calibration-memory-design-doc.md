# 校正记忆《调研与设计文档》成稿计划

> **说明：** 这是 **文档成稿** 的任务拆解，不是代码实现计划。编码阶段另用 `writing-plans` 技能。

**Goal:** 产出成稿于 `rag_design/docs/plans/2026-03-28-调研与设计-校正记忆与经验库.md`，草稿于 `rag_design/drafts/`；满足 [rag-design-principles.mdc](../../.cursor/rules/rag-design-principles.mdc)。

**Approach:** 四步写作顺序（锚定 → 调研补全 → 设计 → 收口），正文五块，无冗长附录。

**Tech Stack:** Markdown；引用以 `knowledge/` 与可核实外链为准。

---

### Task 1: 锚定段

**Files:**
- Create or fill: `rag_design/docs/调研与设计-校正记忆与经验库.md`（可先建骨架）
- Read: [knowledge/基于 EvoPalantir 的现实对齐模拟方案.md](../../../knowledge/基于%20EvoPalantir%20的现实对齐模拟方案.md) §4.8–4.11

- [ ] 写出 **问题与边界**（块 1）：数据流一句、校正记忆职责一句、引用路径。

### Task 2: 数据流与模块

- [ ] 写出 **块 2**：四模块 **记录 | 索引 | 检索 | 下游**；每模块 **输入/输出/非职责**；与快照/校正/推演的关系。

### Task 3: 调研摘录

- [ ] 写出 **块 3**：扩展 [自进化Agent…](../自进化Agent：经验写回的运行时记忆闭环机制/自进化Agent：经验写回的运行时记忆闭环机制.md) + MemOS/OpenViking/Mem0 等 **短表**，每行 **场景映射**；无来源处标 **缺口**。

### Task 4: 设计方案与收口

- [ ] 写出 **块 4–5**：设计方案凝练、**缺口清单**；合规选项与未决条列。

### Task 5: 自检

- [ ] 对照 `rag-design-range`：无编造 API；与 [需求说明](../需求说明-校正记忆与经验库.md) 无冲突；删冗余句。

---

**完成后：** 由 gpr/sfc 通读；再决定是否启动 **`writing-plans`** 做 **代码** 实施计划。
