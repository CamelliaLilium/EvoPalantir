# rag_design · 外部调研（references）

本目录存放 **对第三方项目/论文/产品** 的调研笔记，供校正记忆、RAG、Skill 等方向 **借鉴与对比**。**不是** `knowledge/` 规范；正式设计以 `knowledge/`、需求说明与引擎宪章为准。

## 强制要求（写新文件前必读）

1. **基线 commit（强制）**  
   凡分析 **带公开 Git 仓库的原项目**，文首元数据中 **必须** 填写 **基线 commit**（完整 40 位 SHA，或经说明的 tag + 解析到的 SHA）。  
   - 目的：他人可 `git checkout <sha>` 复现你引用的路径与行为。  
   - **仅文档/论文、无仓库** 的调研：在元数据中写明 **「无仓库；依据为 …」**，并给出 **文档版本日期或 URL 快照说明**。

2. **先理解原项目再写（强制）**  
   动笔前 **必须** 完成最小理解路径，并在文中 **简要记录**（一两句即可）：  
   - 官方文档或 README 中与主题相关的章节；和/或  
   - 仓库内与主题相关的目录/入口文件（路径相对仓库根）。  
   **禁止** 只根据二手博客/视频写「架构结论」而不核对一手材料。

3. **从模板复制**  
   新建调研请复制 [模板.md](./模板.md)，填全占位段后再删注释说明。

## 命名建议

`YYYY-MM-DD-<项目或简名>-<主题>.md`  

前缀日期取 **新建或重大修订当日**；元数据表 **日期**、基线旁注的 **`git ls-remote` 执行日** 应与之一致（跨日修订时同步改文件名或另起修订说明，避免文实不符）。

例：`2026-03-29-memtensor-memrl-retrieval.md`

## 规则索引

写法、事实/启示分离、与 `knowledge/` 优先级等见 [rag-design-principles.mdc](../../.cursor/rules/rag-design-principles.mdc) 中 **「外部调研与借鉴写法」** 与 **§8 references 专规**。

## 范式样例（仓库内他处）

可与 `EPFrameWork1/docs/references/` 中已写成的 OpenClaw、OpenCode 等篇对照结构（若该目录被 `.gitignore` 忽略，以本地克隆为准）。

## 索引：自进化 Agent 记忆闭环（2026-03-29）

与 [自进化Agent：经验写回的运行时记忆闭环机制](../自进化Agent：经验写回的运行时记忆闭环机制/自进化Agent：经验写回的运行时记忆闭环机制.md) 中六框架 **一一对应** 的一手调研（论文/基线 SHA）；二手专栏仅作 [登记页](./2026-03-29-zhihu-csdn-memory-loop-survey-sources.md)。

| 框架 | 文件 |
|------|------|
| MemRL（MemTensor） | [2026-03-29-memrl-memory-utility.md](./2026-03-29-memrl-memory-utility.md) |
| ReasoningBank | [2026-03-29-reasoningbank-strategy-memory.md](./2026-03-29-reasoningbank-strategy-memory.md) |
| ReMe | [2026-03-29-reme-memory-governance.md](./2026-03-29-reme-memory-governance.md) |
| MACLA | [2026-03-29-macla-program-memory.md](./2026-03-29-macla-program-memory.md) |
| AgentRR | [2026-03-29-agentrr-record-replay.md](./2026-03-29-agentrr-record-replay.md) |
| AgentEvolver | [2026-03-29-agentevolver-self-evolving-training.md](./2026-03-29-agentevolver-self-evolving-training.md) |
