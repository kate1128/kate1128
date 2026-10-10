<!-- type: glossary -->
# RAG（检索增强生成，Retrieval-Augmented Generation）

> 一种在 LLM 生成答案之前，先从外部知识库中检索相关信息作为上下文，从而提升回答准确性和时效性的技术架构。

## 是什么

RAG 本质上是一个**"搜索 + 生成"的组合**：

1. **检索（Retrieval）**：将用户问题转换成向量，在向量数据库中找到语义最相似的文档片段（Chunk）
2. **增强（Augmented）**：把检索到的文档片段拼进 Prompt 中，作为 LLM 的参考上下文
3. **生成（Generation）**：LLM 基于用户问题 + 检索到的上下文，生成最终答案

RAG 的名字容易让人误解它只是"加了个检索"，实际上它还涉及文档加载、文本分割（Chunking）、Embedding 模型选择、检索策略调优、重排序（Rerank）和 Prompt 设计——是一个端到端的数据工程 + 模型调优流程。

## 为什么重要

RAG 解决了 LLM 的三大核心缺陷：

| 缺陷 | RAG 如何解决 |
| --- | --- |
| 知识截止日期 | 检索最新文档，绕过训练数据的时间限制 |
| 私有领域盲区 | 检索企业内部文档，让 LLM 回答私有业务问题 |
| 幻觉/编造 | 把答案锚定在检索到的真实文档上，显著降低编造概率 |

此外，RAG 相比微调（Fine-tuning）成本更低、更新更灵活——新增文档即时生效，不需要重新训练模型。

## 关键要点

- **Chunk 大小是核心调参项**：太大则检索精度下降，太小则信息不完整。通常 256-512 token 是 OpenAI embedding 模型的甜点
- **检索 ≠ 关键词搜索**：向量检索找的是语义相似，不是字面匹配
- **RAG 的 7 个常见失败点**：内容缺失、文档加载不佳、分块不当、Top-K 遗漏、上下文无关、格式错误、答案不完整——每个都需要针对性优化
- **高级变体**：GraphRAG（知识图谱增强）、RAG-Fusion（多查询融合）、Rewrite-Retrieve-Read（先改写再检索）

## 相关概念

- [[Embedding 词向量]] — RAG 用 Embedding 将文本和查询都转成向量
- [[向量数据库]] — RAG 的检索层存储引擎
- [[幻觉]] — RAG 要解决的核心问题之一
- [[Prompt 工程]] — RAG 的生成环节依赖 Prompt 设计

## 参考

- [RAG 论文 - Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401)
- [Seven Failure Points When Engineering a RAG System](https://arxiv.org/abs/2401.05856)