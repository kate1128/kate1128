> LLaMA 官方地址：[https://llama.meta.com](https://llama.meta.com)
>

> LLaMA 开源地址：[GitHub - meta-llama/llama: Inference code for Llama models](https://github.com/facebookresearch/llama)
>

**LLaMA 系列模型**是 **Meta** 开源的一组参数规模 **从 7B 到 70B** 的基础语言模型。LLaMA 于`2023 年 2 月`发布，2023 年 7 月发布了 LLaMA2 模型，并于 `2024 年 4 月 18 日`发布了 **LLaMA3** 模型。它们都是在数万亿个字符上训练的，展示了如何**仅使用公开可用的数据集来训练最先进的模型**，而不需要依赖专有或不可访问的数据集。这些数据集包括 Common Crawl、Wikipedia、OpenWebText2、RealNews、Books 等。

LLaMA 模型使用了**大规模的数据过滤和清洗技术**，以提高数据质量和多样性，减少噪声和偏见。

LLaMA 模型还使用了高效的**数据并行**和**流水线并行**技术，以加速模型的训练和扩展。特别地，LLaMA 13B 在 CommonsenseQA 等 9 个基准测试中超过了 GPT-3 (175B)，而 **LLaMA 65B 与最优秀的模型 Chinchilla-70B 和 PaLM-540B 相媲美**。LLaMA 通过使用更少的字符来达到最佳性能，从而在各种推理预算下具有优势。



与 GPT 系列相同，LLaMA 模型也采用了 **decoder-only** 架构，同时结合了一些前人工作的改进：

+ `Pre-normalization 正则化`：为了提高训练稳定性，LLaMA 对每个 Transformer 子层的输入进行了 RMSNorm 归一化，这种归一化方法可以避免梯度爆炸和消失的问题，提高模型的收敛速度和性能；
+ `SwiGLU 激活函数`：将 ReLU 非线性替换为 SwiGLU 激活函数，增加网络的表达能力和非线性，同时减少参数量和计算量；
+ `旋转位置编码（RoPE，Rotary Position Embedding）`：模型的输入不再使用位置编码，而是在网络的每一层添加了位置编码，RoPE 位置编码可以有效地捕捉输入序列中的相对位置信息，并且具有更好的泛化能力。

****

**LLaMA3** 在 LLaMA 系列模型的基础上进行了改进，提高了模型的性能和效率：

+ `更多的训练数据量`：LLaMA3 在 15 万亿个 token 的数据上进行预训练，相比 LLaMA2 的训练数据量增加了 7 倍，且代码数据增加了 4 倍。LLaMA3 能够接触到更多的文本信息，从而提高了其理解和生成文本的能力。
+ `更长的上下文长度`：LLaMA3 的上下文长度增加了一倍，从 LLaMA2 的 4096 个 token 增加到了 8192。这使得 LLaMA3 能够处理更长的文本序列，改善了对长文本的理解和生成能力。
+ `分组查询注意力（GQA，Grouped-Query Attention）`：通过将查询（query）分组并在组内共享键（key）和值（value），减少了计算量，同时保持了模型性能，提高了大型模型的推理效率（LLaMA2 只有 70B 采用）。
+ `更大的词表`：LLaMA3 升级为了 128K 的 tokenizer，是前两代 32K 的 4 倍，这使得其语义编码能力得到了极大的增强，从而显著提升了模型的性能。



