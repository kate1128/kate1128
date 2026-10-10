**Transformer** 是一种深度学习的神经网络架构，于 2017 年由 Vaswani 等人在论文《Attention is All You Need》中提出。它专为解决序列到序列的建模任务而设计（如机器翻译、文本生成），并因其高效的并行计算和强大的性能，成为自然语言处理（NLP）领域的核心技术，同时在图像处理、音频处理等领域也得到了广泛应用。

原文地址：[https://arxiv.org/abs/1706.03762](https://arxiv.org/abs/1706.03762)

[1706.03762v7.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1737079730307-8266b3d8-647c-441b-ab6d-075f68d5db4e.pdf)

# Transformer 的核心架构
Transformer 的设计基于一种关键机制——**注意力机制（Attention Mechanism）**，并通过完全抛弃传统的循环神经网络（RNN）或卷积神经网络（CNN），依赖自注意力（Self-Attention）实现了对长距离依赖的高效建模。

Transformer 主要由两个部分组成：

1. **编码器（Encoder）**
2. **解码器（Decoder）**

## 编码器（Encoder）
编码器由多个堆叠的层组成，每一层包含：

+ **自注意力机制（Self-Attention Layer）**：输入序列中每个词对所有其他词的关系进行建模。
+ **前馈网络（Feed Forward Network, FFN）**：一个位置无关的全连接网络，用于非线性变换。

编码器的作用是将输入序列（如一段文本）编码成高维的隐藏表示。

## 解码器（Decoder）
解码器与编码器结构类似，也由多个层堆叠而成，但解码器层中加入了：

+ **自注意力机制**：处理解码器当前已经生成的序列。
+ **编码器-解码器注意力机制（Encoder-Decoder Attention）**：将编码器的输出与解码器的生成序列关联起来。
+ **前馈网络（Feed Forward Network, FFN）**。

解码器负责根据编码器的输出和已有生成序列，逐步生成目标序列（如翻译结果）。

# Transformer 的关键技术
## 自注意力机制（Self-Attention）
自注意力机制是 Transformer 的核心，用于计算输入序列中每个位置对其他位置的相关性。其计算过程包括：

1. 为每个输入词生成三个向量：**查询向量（Query, Q）**、**键向量（Key, K）** 和 **值向量（Value, V）**。
2. 计算查询向量和键向量之间的点积，得到每对词的相关性得分。
3. 将相关性得分通过 Softmax 转化为权重，对值向量加权求和，得到输出。

公式如下：

$ \text{Attention}(Q, K, V) = \text{Softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V $

其中，$ d_k $ 是键向量的维度，用于缩放点积以防止数值过大。

## 多头注意力机制（Multi-Head Attention）
多头注意力机制是对单一注意力机制的扩展，它通过多个注意力头并行处理数据，从不同的子空间提取信息。每个头独立执行自注意力计算，结果再进行拼接和线性变换。

公式如下：

$ \text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)W^O $

其中，每个 $ \text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V) $。

## 位置编码（Positional Encoding）
由于 Transformer 没有内建的顺序处理能力（不像 RNN 有时间步），需要额外加入位置信息。位置编码是一种固定或可训练的向量，表示输入序列中每个词的位置，通常通过正弦和余弦函数生成。

公式：

$ PE(pos, 2i) = \sin\left(\frac{pos}{10000^{2i/d_\text{model}}}\right), \quad PE(pos, 2i+1) = \cos\left(\frac{pos}{10000^{2i/d_\text{model}}}\right) $

## 前馈网络（Feed Forward Network, FFN）
每个 Transformer 层中都有一个独立的前馈网络，用于非线性变换。FFN 作用于每个位置的隐藏状态，是完全位置无关的，全连接操作如下：

$ FFN(x) = \text{ReLU}(xW_1 + b_1)W_2 + b_2 $

# Transformer 的优点
1. **并行计算**：相比 RNN，Transformer 可以同时处理序列中的所有位置，充分利用 GPU 加速。
2. **强大的长距离依赖建模能力**：自注意力机制能够全局建模序列中的每个词之间的关系。
3. **灵活性和可扩展性**：结构简单，易于扩展和修改。

# Transformer 的应用
1. **自然语言处理（NLP）**：
    - **机器翻译**：如 Google 的 Neural Machine Translation（GNMT）。
    - **文本生成**：如 OpenAI 的 GPT 系列。
    - **语义理解**：如 BERT，用于问答、情感分析等任务。
2. **计算机视觉（CV）**：
    - Vision Transformer（ViT）用于图像分类、目标检测等任务。
3. **语音处理**：
    - 用于语音识别、语音合成等任务。
4. **多模态任务**：
    - 结合文本、图像、音频的多模态建模任务。

# 著名的 Transformer 模型
1. **BERT（Bidirectional Encoder Representations from Transformers）**：双向编码器，用于理解任务。
2. **GPT（Generative Pre-trained Transformer）**：自回归解码器，用于生成任务。
3. **T5（Text-to-Text Transfer Transformer）**：统一编码器-解码器模型，用于多任务学习。
4. **Vision Transformer (ViT)**：将 Transformer 应用于图像处理。

Transformer 是现代深度学习的重要基石，其创新的架构在多个领域引发了革命性的发展。

