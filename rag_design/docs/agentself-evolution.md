# 4.10 校正记忆层：calibration_memory_engine

## 1. Run 记录 / 实验留痕  
**借鉴对象：MLflow Tracking**  
**如何借鉴：**
- 用 MLflow 记录每一次推演/校正的 run  
- params：场景 ID、seed、版本号、机制签名  
- metrics：预测误差、修正效果、案例命中率  
- artifacts：轨迹快照、误差日志、case JSON、规则文件  
- 仅作为 tracking 和回放索引层，不作为案例检索库  

**链接：**
https://mlflow.org/docs/latest/ml/tracking/  
https://mlflow.org/docs/latest/ml/tracking/tutorials/local-database/  

---

## 2. 仿真快照采集  
**借鉴对象：Mesa DataCollector**  
**如何借鉴：**
- 按 tick 统一采集数据  
- model-level：整体极化、分布指标  
- agent-level：stance、活跃度、曝光  
- tables：事件日志、关键变化  
- 可只借数据结构，不强依赖 Mesa  

**链接：**
https://mesa.readthedocs.io/  
https://mesa.readthedocs.io/latest/apis/datacollection.html  

---

## 3. 单次失败反思  
**借鉴对象：Reflexion**  
**如何借鉴：**
- 每次偏差较大生成 failure reflection  
- 保存为 case 的文本字段  
- 用于后续检索与解释  

**链接：**
https://arxiv.org/abs/2303.11366  
https://github.com/noahshinn/reflexion  

---

## 4. 多次经验蒸馏  
**借鉴对象：ExpeL**  
**如何借鉴：**
- 从多条 case 提炼规则  
- 构建 rule book  
- 用于辅助后续诊断  

**链接：**
https://arxiv.org/abs/2308.10144  

---

## 5. 经验记忆更新  
**借鉴对象：MemRL**  
**如何借鉴：**
- 外部记忆，不修改主模型  
- Two-Phase Retrieval（粗召回 + 精排）  
- utility-driven update（有效案例升权）  

**链接：**
https://arxiv.org/abs/2601.03192  
https://github.com/MemTensor/MemRL  

---

## 6. 相似案例召回  
**借鉴对象：Faiss**  
**如何借鉴：**
- 建立 embedding 索引  
- Top-K 相似案例检索  
- 用于后续复用  

**链接：**
https://github.com/facebookresearch/faiss  
https://ai.meta.com/tools/faiss/  

---

## 7. 案例复用流程  
**借鉴对象：CBR**  
**如何借鉴：**
- retrieve → adapt → revise → retain  
- 用于案例复用闭环  

**链接：**
https://homes.luddy.indiana.edu/leake/papers/a-96-book.html  

---

## 8. 流程状态持久化  
**借鉴对象：LangGraph Persistence**  
**如何借鉴：**
- 保存 workflow state  
- 支持 checkpoint / replay / debug  
- 不作为主记忆库  

**链接：**
https://docs.langchain.com/oss/python/langgraph/persistence  

---

## 9. 合并方案  

### 主骨架
- MLflow  
- Mesa 风格数据采集  
- MemRL  
- Faiss  
- CBR  

### 增强模块
- Reflexion  
- ExpeL  
- LangGraph Persistence  