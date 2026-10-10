<!-- type: glossary -->
# LLaMA 系列（Meta）

> Meta（Facebook）开源的**大语言模型系列**，从 7B 到 65B+ 覆盖多个规模，是开源 LLM 生态最重要的基石模型。

## 基本情况

- **开发者**：Meta AI
- **首次发布**：LLaMA 1（2023.02），LLaMA 2（2023.07），LLaMA 3（2024.04）
- **官方地址**：[llama.meta.com](https://llama.meta.com)
- **开源地址**：[github.com/meta-llama](https://github.com/facebookresearch/llama)

## 核心版本

| 版本 | 发布时间 | 最大参数 | 训练数据 | 上下文 | 关键升级 |
| --- | --- | --- | --- | --- | --- |
| LLaMA 1 | 2023.02 | 65B | 1.4T tokens | 2K | 证明纯公开数据也能训出 SOTA |
| LLaMA 2 | 2023.07 | 70B | 2T tokens | 4K | 开放商用，Chat 版本对齐更完善 |
| LLaMA 3 | 2024.04 | 70B (405B 在训练) | 15T tokens | 8K | 训练数据 7× 增长，代码数据 4× 增长 |

## 技术特点

- **Decoder-only Transformer**，架构与 GPT 类似但用了更好的工程优化
- **RMSNorm 归一化**：比 LayerNorm 更稳定
- **SwiGLU 激活函数**：替代 ReLU，更好非线性 + 更少参数
- **RoPE 旋转位置编码**：更好地捕捉相对位置关系
- **GQA 分组查询注意力**（LLaMA 3）：有效减少 KV Cache 显存占用

## 为什么重要

- **开源 LLM 的基石**：绝大部分开源微调模型（Alpaca、Vicuna、Mistral 早期版本）都以 LLaMA 为基础
- **LLaMA 13B 超 GPT-3 175B**：在多个基准上小模型击败大模型，证明了数据质量 > 参数规模
- **LLaMA 3 是分水岭**：开源模型首次在全面评测中接近 GPT-4 级别

## 相关笔记

- [[通义千问 Qwen]] — 另一个重要的开源 LLM 系列
- [[模型选型指南](../../../07-LLM应用开发/01-LLM应用开发/对话型应用的设计/第一步_模型选型.md)]

## 参考

- [Meta LLaMA 官方](https://llama.meta.com/)
- [LLaMA 论文 - LLaMA: Open and Efficient Foundation Language Models](https://arxiv.org/abs/2302.13971)