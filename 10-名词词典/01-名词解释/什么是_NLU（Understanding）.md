<!-- type: glossary -->
# NLU（自然语言理解，Natural Language Understanding）

> NLP（自然语言处理）的两大核心任务之一，目标是让计算机**理解**人类语言的含义、意图、情感和实体关系，而不仅仅是处理文本字符串。

## 是什么

NLU 关心的是"这句话到底在说什么"：

- **意图识别（Intent Detection）**："我想订一张明天去上海的机票" → 意图是"订机票"
- **实体抽取（Named Entity Recognition）**："明天去上海" → 提取出 时间=明天、目的地=上海
- **情感分析（Sentiment Analysis）**："这个产品还不错" → 正面情感
- **语义相似度（Semantic Similarity）**："怎么退款"和"如何退货" → 高度相似

NLU 与 NLG（自然语言生成）并称 NLP 的两大支柱：**NLU 负责"读懂"，NLG 负责"写出来"**。

## 为什么重要

- **LLM 的"输入层"**：无论是对话、RAG 检索还是 Agent 决策，第一步都是理解用户到底要什么——这依赖 NLU
- **多轮对话的关键**：理解"它"指代谁、"上次说的那个"是哪个——这些都是 NLU 中的指代消解和上下文理解
- **搜索与推荐的基石**：传统关键词搜索正在被语义搜索取代，后者需要 NLU 来理解"用户真正想找什么"

## 关键要点

- **In-Context Learning 是 LLM 做 NLU 的核心方式**：不需要微调，给几个示例就能理解新任务
- **NLU 的经典任务**：文本分类、情感分析、命名实体识别（NER）、关系抽取、意图识别、指代消解
- **评估指标**：准确率（Accuracy）、F1 Score、困惑度（Perplexity）
- **难点**：歧义（"苹果"是公司还是水果）、长距离依赖、低资源语言

## 相关概念

- [[NLG 自然语言生成]] — NLU 负责理解输入，NLG 负责生成输出
- [[NLP 自然语言处理]] — NLU 和 NLG 的父领域
- [[Embedding 词向量]] — 将文本转为向量是 NLU 任务的基础步骤

## 参考

- [Stanford CS224n - NLP with Deep Learning](https://web.stanford.edu/class/cs224n/)
- [Hugging Face NLP Course](https://huggingface.co/learn/nlp-course)