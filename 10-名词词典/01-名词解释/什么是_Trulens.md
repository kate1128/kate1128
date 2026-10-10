<!-- type: glossary -->
# TruLens（TruEra TruLens）

> 一个开源的 **LLM 应用评估与可观测性框架**，专注于帮助开发者追踪、评估和调试 LLM 应用（特别是 RAG 系统）的质量表现。

## 是什么

TruLens 的核心思想是**为 LLM 应用建立"反馈闭环"**：

1. **记录（Logging）**：自动捕获每次 LLM 调用的输入、输出、中间步骤（如检索到的文档）
2. **评估（Evaluation）**：用"反馈函数（Feedback Functions）"自动打分——答案相关性、上下文忠实性、是否有幻觉等
3. **追踪（Tracing）**：可视化每条请求的完整链路——检索到了什么 → Prompt 长什么样 → LLM 输出了什么
4. **迭代优化**：基于评估结果修改 Prompt、检索策略或模型选择

## 为什么重要

- **解决 LLM 的"评估难题"**：传统软件有单元测试，LLM 应用怎么测？TruLens 提供了自动化方案
- **RAG 调试加速**：不用再看日志文件猜问题——TruLens 仪表盘直接展示每条请求的检索质量、答案相关性和忠实度
- **从"凭感觉"到"凭数据"**：把 LLM 应用的优化从艺术变成工程

## 关键要点

- **核心概念：Feedback Functions**：可以理解为"用 LLM 评估 LLM"——比如用一个 GPT-4 模型给应用输出的忠实度打分
- **RAG 评估三件套**（RAG Triad）：Answer Relevance（答案相关性）、Context Relevance（上下文相关性）、Groundedness（答案是否基于检索内容）
- **与 LangChain/LlamaIndex 集成**：通过几行代码挂载到现有的 LangChain/LlamaIndex 应用上
- **开源 + SaaS**：开源版可在本地部署，也有 TruEra 的云服务版本

## 相关概念

- [[RAG 检索增强生成]] — TruLens 最主要的使用场景
- [[RAGAS]] — 另一个 RAG 评估框架，常与 TruLens 对比
- [[可观测性]] — TruLens 本质上是一个 LLM 应用的可观测性工具
- [[LangSmith]] — LangChain 官方的类似工具

## 参考

- [TruLens GitHub](https://github.com/truera/trulens)
- [TruLens 文档](https://www.trulens.org/)