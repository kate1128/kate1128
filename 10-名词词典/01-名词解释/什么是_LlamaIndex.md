<!-- type: glossary -->
# LlamaIndex

> 一个专注于**数据索引和检索增强**的 Python 框架。与 LangChain 的"大而全"不同，LlamaIndex 将精力集中在 RAG 领域——如何更好地加载文档、构建索引、检索和增强 LLM 回答。

## 是什么

LlamaIndex 的核心抽象是一个从**数据 → 索引 → 查询**的管道：

1. **数据连接器（Data Connectors）**：加载 PDF、网页、数据库、API 等各种来源的数据
2. **索引（Indexes）**：将数据组织成不同的检索结构——向量索引、摘要索引、树形索引、知识图谱索引
3. **查询引擎（Query Engine）**：提供统一的查询接口，背后自动处理检索、排序、上下文组装和 LLM 调用

与 LangChain 的关键差异：LlamaIndex 专注于"怎么让 LLM 更好地利用你的数据"，而 LangChain 试图覆盖整个 LLM 应用开发的链条。

## 为什么重要

- **RAG 的第二个选择**：如果 LangChain 太"重"，LlamaIndex 通常更轻量、文档更好、上手更快
- **多租户 RAG 原生支持**：内置了数据隔离机制，适合企业级"不同用户只能看自己的文档"场景
- **索引结构丰富**：不光有向量索引，还有树形索引（用于长文档摘要）、知识图谱索引等高级结构
- **GitHub Open Issues 管理好于 LangChain**：代码质量和文档维护在社区中有较好口碑

## 关键要点

- **创始人 Jerry Liu**，2023 年 6 月获 850 万美元种子轮
- **适用场景**：文档问答、知识库构建、多租户数据检索——这些是 LlamaIndex 的甜点
- **不适合的场景**：需要通用 Agent、复杂链式编排的生产应用——这些场景 LangChain 生态更成熟
- **可以互补使用**：LlamaIndex 做索引和检索，LangChain 做链和 Agent——两个框架可以混合使用

## 相关概念

- [[LangChain]] — LlamaIndex 的主要竞品/互补框架
- [[RAG 检索增强生成]] — LlamaIndex 聚焦的核心技术领域
- [[向量数据库]] — LlamaIndex 索引层的底层存储

## 参考

- [LlamaIndex 官方文档](https://docs.llamaindex.ai/)
- [LlamaIndex GitHub](https://github.com/run-llama/llama_index)
- 本知识库: [05-主流框架与Agent/02-LlamaIndex](../../05-主流框架与Agent/02-LlamaIndex/)