<!-- type: glossary -->
# 大型语言模型（LLM, Large Language Model）

> 一种在海量文本上训练的、包含数十亿至数千亿参数的深度学习模型，旨在理解和生成人类语言。

## 是什么

LLM 的核心任务是**预测下一个 token**——给它一段文本，它推断最可能的后续内容。这个简单的目标，在足够大的模型规模、足够多的训练数据和足够长的训练时间下，催生了让整个行业震惊的"涌现能力"：模型不仅能翻译、摘要、问答，还能推理、编程、遵循复杂指令。

**规模是关键分水岭**。BERT（3.3 亿参数）和 GPT-2（15 亿参数）虽然架构相似，但能力远不如 GPT-3（1750 亿参数）和 PaLM（5400 亿参数）。当参数规模突破某个阈值后，模型突然展现出了少样本学习（Few-shot Learning）、思维链推理（Chain-of-Thought）等能力——这些能力在小型模型上几乎不存在。

## 为什么重要

LLM 是当前 AI 浪潮的基础设施级技术。它改变了人机交互范式：

- **统一入口**：过去不同 NLP 任务（翻译、分类、抽取）需要不同模型，现在一个 LLM 全搞定
- **泛化能力**：未见过的新任务，给几个例子就能做（In-Context Learning），不需要重新训练
- **生态基石**：Prompt 工程、RAG、Agent、Fine-tuning、模型部署——整个 AI 应用栈都围绕 LLM 构建

## 关键要点

- **训练范式**：预训练（Pre-training）→ 指令微调（Instruction Tuning）→ 人类对齐（RLHF）
- **代表模型**：国外有 GPT-4o、Claude、Gemini、LLaMA；国内有通义千问、文心一言、DeepSeek、ChatGLM、星火
- **核心权衡**：模型规模 vs 推理成本、通用能力 vs 领域精度、闭源 vs 开源
- **Token 是基本单位**：模型不是按"字"而是按 token 计费和计算，通常 1 token ≈ 0.75 个英文单词或 1-2 个中文字

## 相关概念

- [[In-Context Learning]] — LLM 无需微调、通过上下文示例完成新任务的能力
- [[Prompt 工程]] — 设计输入文本以引导 LLM 产生期望输出的技术
- [[RAG 检索增强生成]] — 用外部知识库补充 LLM 的知识缺口
- [[幻觉]] — LLM 生成看似合理但实际错误的内容，是其核心缺陷之一

## 参考

- [A Survey of Large Language Models](https://arxiv.org/abs/2303.18223)
- [OpenAI GPT-4 Technical Report](https://arxiv.org/abs/2303.08774)