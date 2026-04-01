# Faiss：向量近邻检索后端与索引选型空间

> **非规范文档。** 本文是外部项目调研，**不定义** EvoPalantir / `rag_design` / `knowledge/` 的正式行为。涉及本仓库的段落仅为 **观察**，不构成设计决策；正式采纳须经评审。

---

## 元数据（必填）

| 字段 | 填写 |
|------|------|
| **日期** | 2026-03-31 |
| **作者/角色** | Codex（调研） |
| **原项目** | The Faiss library · [arXiv:2401.08281](https://arxiv.org/abs/2401.08281) |
| **仓库 URL** | https://github.com/facebookresearch/faiss |
| **基线 commit（强制）** | `57190f91cddffc8ac168dd5ac88044976d4be864`（2026-03-31 对 GitHub `commits/main` 页面最新提交的快照） |
| **检索与阅读记录（强制）** | arXiv 论文首页；GitHub README、目录树与 `commits/main` 页面；未逐索引类型核对源码实现 |

**基线 commit 直链：**  
https://github.com/facebookresearch/faiss/tree/57190f91cddffc8ac168dd5ac88044976d4be864

---

## 一句话结论

Faiss 很适合作为 CME `IndexModule` 的 **可选向量索引后端**，但它只能承担“向量召回”这一层，不能替代结构化索引、案例真源或业务重排逻辑。

---

## 1. 问题域

Faiss 解决的是 **高维稠密向量的相似度搜索与聚类**，尤其适合大规模近邻搜索。对 EvoPalantir 的直接价值在于：当 `CaseRecord` 需要可选 embedding 检索时，Faiss 提供了一整套成熟的向量索引与近邻搜索实现。

---

## 2. 对象界定

- **是：** Facebook Research 维护的 **Faiss** 论文与官方实现；对象包括精确/近似近邻搜索、索引训练与压缩、CPU/GPU 双栈实现。
- **不是：** 结构化 case store、业务重排器、下游 RetrievalPack 适配器。
- **边界提醒：** 本文关注的是 **向量检索后端能力与索引家族边界**，不是完整 ANN 基准评测，也不是业务检索排序策略。
- **与源笔记对齐：** 宪章已把 Faiss 明写为 `IndexModule` 的可选向量索引实现，可与 [校正记忆引擎宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) §7 对照阅读。

---

## 3. 机制拆解（事实陈述）

### 3.1 论文 / README 摘要层

- **核心能力：** 对 dense vectors 做高效 similarity search 与 clustering。
- **规模取向：** README 明确强调可扩展到内存放不下的向量规模。
- **索引族丰富：** exact / approximate、多种压缩与图索引方案并存，核心是不同的速度、质量、内存、训练时间权衡。
- **CPU / GPU 双栈：** GPU index 可作为 CPU index 的替代方案，适合未来更大规模场景。

### 3.2 开源仓库布局（基线 README）

- **核心目录：** `faiss/`
- **教程：** `tutorial/`
- **测试：** `tests/`
- **扩展/工具：** `contrib/`
- **安装与构建：** `INSTALL.md`
- **索引家族示例：** 仓库层面可见 `IndexFlat`、`IndexIVF`、`IndexHNSW`、`ProductQuantizer` 等家族化实现命名，这对应论文里“精度/速度/内存”多目标权衡。

---

## 4. 与 EvoPalantir 既定设计的对比（事实对齐）

| 维度 | 外部方案 | 本仓库（出处） |
|------|----------|----------------|
| 主对象 | dense vectors 的 ANN / exact search | `IndexModule` 的可选向量近邻（[宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md) §7.1、§7.2） |
| 默认角色 | 向量索引本体 | 结构化索引必选，Faiss 可选（宪章 §1.2、§7.2） |
| 排序逻辑 | 返回 candidate ids / distances | 最终顺序与加权在 `RetrieveModule`，不在 Index 内完成（宪章 §7.5） |
| 真源存储 | 不负责 | `CaseStore` 才是 case 真源（宪章 §2.3、§6） |
| 下游契约 | 与业务无关 | 下游只认 `RetrievalPack`，不能依赖 Faiss 内部结构（宪章 §1.2、§10） |

---

## 5. 重要源码坐标（若有仓库）

| 主题 | 路径 | 备注 |
|------|------|------|
| Flat 索引代表 | `faiss/IndexFlat.h` | 精确检索代表实现名 |
| IVF 索引代表 | `faiss/IndexIVF.h` | 倒排文件类索引代表实现名 |
| HNSW 索引代表 | `faiss/IndexHNSW.h` | 图索引代表实现名 |
| 教程 | `tutorial/` | 适合后续验证索引选型 |
| 测试 | `tests/` | 适合比对行为与回归 |
| 安装说明 | `INSTALL.md` | CPU/GPU 构建与依赖 |

---

## 6. 文档 vs 源码差异（若有）

- README 与论文强调的是 Faiss 的索引家族与能力边界，但并不替你决定 **具体选哪种索引**；对 CME 而言，`IndexFlat`、`HNSW`、`IVF`、`PQ` 的取舍仍需后续实验。
- 本文未逐一核对各索引类型在当前基线 commit 下的参数默认值与适用边界；如果未来要做实现 spec，需要继续做选型实验。
- 本文虽然补了代表性头文件坐标，但还没有逐项比较各索引在当前 commit 下的训练依赖、增删改能力与持久化形式；这些仍是 V1 选型缺口。

---

## 7. 观察：对校正记忆（CME）的启示

> **非规范性观察。** 不构成设计决策；采纳须经 spec/评审。

### 7.1 可借鉴

- **Faiss 很适合被封装在 `VectorIndex` 接口后面**，只暴露 `vector_query` / `upsert` 等基本能力。
- **把向量检索降为“候选召回”层** 非常合理；之后仍由 `RetrieveModule` 结合结构化条件与效用分重排。
- 对未来案例量增大时，Faiss 给了清晰的扩容路线。

### 7.2 应保持的差异化（不盲从）

- CME V1 的主路仍是 **结构化 key + 结构化过滤**；向量索引只是可选增强，不应抢主语义。
- Faiss 只处理向量空间，不知道 `event_phase`、`误差模式`、`治理状态` 等业务字段，因此不能承担最终检索判定。
- 下游 forecast/report 不应感知 Faiss 的索引类型、距离度量或内部 ID 结构。

### 7.3 明确不做

- 不把 Faiss 作为 case 真源数据库。
- 不做 “只有向量检索、没有结构化过滤” 的 V1 方案。
- 不让 `RetrievalPack` 暴露 Faiss 距离值或内部索引细节作为稳定契约字段。

---

## 8. 参考来源

- 论文：[arXiv:2401.08281](https://arxiv.org/abs/2401.08281)
- 仓库（基线快照）：https://github.com/facebookresearch/faiss/tree/57190f91cddffc8ac168dd5ac88044976d4be864
- 本仓库设计约束：[校正记忆引擎宪章](../plans/2026-03-28-调研与设计-校正记忆与经验库.md)
- 说明：本次事实层直接依据论文首页与仓库 README/目录树，不依赖二手综述。
