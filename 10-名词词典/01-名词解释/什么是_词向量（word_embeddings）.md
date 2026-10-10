<!-- type: glossary -->
# 词向量 / 词嵌入（Word Embeddings）

> 一种将**文本（词、句子、文档）转换成固定维度的实数向量**的技术。语义相似的文本在向量空间中的距离更近。这是 LLM 和 RAG 理解文本的数学基础。

## 是什么

计算机不懂文字，只懂数字。Embedding 的作用就是**给每个词/句子/文档分配一个"坐标"**：

- "king" → `[0.12, -0.34, 0.87, ..., 0.05]`（可能是 3072 维）
- "queen" → `[0.11, -0.33, 0.86, ..., 0.04]`（与 king 非常接近）
- "apple" → `[-0.45, 0.67, -0.23, ..., 0.91]`（与 king 距离很远）

这个坐标不是人手工设计的——它是通过大量文本数据训练出来的。训练的核心思想是**"一个词的含义可以由它周围的词来定义"**（分布假说）——经常出现在相似上下文中的词，向量自然就靠近。

经典类比：`king - man + woman ≈ queen`，说明向量空间中编码了性别、语义等多维度信息。

## 为什么重要

- **NLP 的基础设施**：从搜索引擎到 LLM，Embedding 是将文本送入计算流程的第一步
- **RAG 的检索引擎**：用户问题和知识库文档都通过 Embedding 变成向量，然后在向量空间中找最近邻居——这是语义搜索的核心
- **多模态统一表示**：CLIP 等模型用同一套 Embedding 空间表示文本和图像——你可以搜"一只猫"，找到猫的照片

## 关键要点

- **主流 Embedding 模型**：OpenAI text-embedding-3-small/large（1536/3072 维）、BGE（BAAI）、M3E（中文优化）、Jina AI
- **维度的权衡**：高维 → 表达能力强但存储和计算成本高；低维 → 速度快但信息损失多
- **距离度量**：余弦相似度（最常用）、欧氏距离、点积——不同模型可能对不同度量优化
- **并非所有 Embedding 都一样**：为检索优化的 Embedding 和为分类优化的可能表现差异巨大

## 相关概念

- [[向量数据库]] — 存储和检索 Embedding 的数据库
- [[RAG 检索增强生成]] — Embedding 是 RAG 检索环节的核心技术
- [[语义搜索]] — 用 Embedding 相似度替代关键词匹配的搜索方式
- [[多模态]] — 用同一套 Embedding 空间统一表示文本、图像、音频

## 参考

- [The Illustrated Word2vec](https://jalammar.github.io/illustrated-word2vec/)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)