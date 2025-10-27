> GPT 的两个关键点：
>
> + 训练能够准确预测下一个单词的 `decoder-only` 的 `Transformer` 语言模型
> + 扩展语言模型的大小
>

# 参考文档
1. 文档地址：[https://platform.openai.com/docs/models](https://platform.openai.com/docs/models)
2. ChatGPT 使用地址：[https://chat.openai.com](https://chat.openai.com)
3. 主流模型 API 对比：[https://openai.com/pricing](https://openai.com/pricing)
4. 文档：[https://platform.openai.com/docs/guides/fine-tuning/cli-data-preparation-tool](https://platform.openai.com/docs/guides/fine-tuning/cli-data-preparation-tool)
5. 调试版：[https://openai.apifox.cn/api-123563406](https://openai.apifox.cn/api-123563406)
6. cookbook：[https://github.com/openai/openai-cookbook/blob/main/examples/fine-tuned_qa/olympics-1-collect-data.ipynb](https://github.com/openai/openai-cookbook/blob/main/examples/fine-tuned_qa/olympics-1-collect-data.ipynb)

# 简介
**OpenAI** 公司在 `2018 年`提出的 **GPT（Generative Pre-Training）** 模型是典型的 `生成式预训练语言模型` 之一。

GPT 模型的基本原则是**通过语言建模将世界知识压缩到仅解码器 (decoder-only) 的 Transformer 模型中**，这样它就可以恢复(或记忆)世界知识的语义，并充当通用任务求解器。

<br/>success
OpenAI 在 LLM 上的研究大致可以分为以下几个阶段：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736494731131-88370739-60a4-4412-ab20-e38561c423e3.png)

<br/>

常见的就是 ChatGPT 和 GPT4

## ChatGPT
`2022 年 11 月`，**OpenAI** 发布了基于 GPT 模型（GPT-3.5 和 GPT-4） 的**会话应用 ChatGPT**。由于与人类交流的出色能力，ChatGPT 自发布以来就引发了人工智能社区的兴奋。ChatGPT 是基于强大的 GPT 模型开发的，具有特别优化的会话能力。ChatGPT 从本质上来说是一个 LLM 应用，是基于基座模型开发出来的，与基座模型有本质的区别。其支持 GPT-3.5 和 GPT-4 两个版本。

现在的 ChatGPT 支持最长达 32,000 个字符，知识截止日期是 2021 年 9 月，它可以执行各种任务，包括**代码编写、数学问题求解、写作建议**等。

ChatGPT 在与人类交流方面表现出了卓越的能力：拥有丰富的知识储备，对数学问题进行推理的技能，在多回合对话中准确追踪上下文，并且与人类安全使用的价值观非常一致。

后来，ChatGPT 支持插件机制，这进一步扩展了 ChatGPT 与现有工具或应用程序的能力。到目前为止，它似乎是人工智能历史上最强大的聊天机器人。ChatGPT 的推出对未来的人工智能研究具有重大影响，它为探索类人人工智能系统提供了启示。

## GPT-4
`2023 年 3 月`发布的 GPT-4，它将**文本输入扩展到多模态信号**。GPT3.5 拥有 **1750** 亿 个参数，而 GPT4 的参数量官方并没有公布，但有相关人员猜测，GPT-4 在 120 层中总共包含了 **1.8 万亿**参数，也就是说，GPT-4 的规模是 GPT-3 的 10 倍以上。因此，GPT-4 比 GPT-3.5 **解决复杂任务的能力更强，在许多评估任务上表现出较大的性能提升**。

最近的一项研究通过对人为生成的问题进行定性测试来研究 GPT-4 的能力，这些问题包含了各种各样的困难任务，并表明 GPT-4 可以比之前的 GPT 模型（如 GPT3.5 ）实现更优越的性能。此外，由于六个月的迭代校准（在 RLHF 训练中有额外的安全奖励信号），GPT-4 对恶意或挑衅性查询的响应更安全，并应用了一些干预策略来缓解 LLM 可能出现的问题，如幻觉、隐私和过度依赖。

> 注意：2023 年 11 月 7 日， OpenAI 召开了首个开发者大会，会上推出了最新的大语言模型 GPT-4 Turbo，Turbo 相当于进阶版。它将上下文长度扩展到 128k，相当于 300 页文本，并且训练知识更新到 2023 年 4 月
>

GPT3.5 是免费的，而 GPT-4 是收费的。需要开通 plus 会员 20 美元/月。`2024 年 5 月 14 日`，新一代旗舰生成模型 **GPT-4o** 正式发布。GPT-4o 具备了对文本、语音、图像三种模态的深度理解能力，反应迅速且富有情感色彩，极具人性化。而且 GPT-4o 是完全免费的，虽然每天的免费使用次数是有限的。

### 语言模型名称
| 语言模型名称 | 上下文长度 | 特点 | input 费用($/million tokens) | output 费用($/ 1M tokens) | 知识截止日期 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| GPT-3.5-turbo-0125 | 16k | 经济，专门对话 | 0.5 | 1.5 | 2021 年 9 月 |
| GPT-3.5-turbo-instruct | 4k | 指令模型 | 1.5 | 2 | 2021 年 9 月 |
| GPT-4 | 8k | 性能更强 | 30 | 60 | 2021 年 9 月 |
| GPT-4-32k | 32k | 性能强，长上下文 | 60 | 120 | 2021 年 9 月 |
| GPT-4-turbo | 128k | 性能更强 | 10 | 30 | 2023 年 12 月 |
| GPT-4o | 128k | 性能最强，速度更快 | 5 | 15 | 2023 年 10 月 |


### Embedding 模型名称
| Embedding 模型名称 | 维度 | 特点 | 费用($/ 1M tokens) |
| :---: | :---: | :---: | :---: |
| text-embedding-3-small | 512/1536 | 较小 | 0.02 |
| text-embedding-3-large | 256/1024/3072 | 较大 | 0.13 |
| ada v2 | 1536 | 传统 | 0.1 |


# API Key 注册
1. 参考：[https://chatgptzhanghao.com/](https://chatgptzhanghao.com/)
2. openAI 官网：[https://platform.openai.com/](https://platform.openai.com/)

可以科学上网之后，直接用邮箱注册

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736909837064-32ed9223-544e-4400-a3d1-d817934b6acf.png)

冲卡

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736910053021-042db46a-d7fc-4a8c-a504-8fb7a3df74dc.png)

实在没有找其他第三方：[https://xiaoai.plus/register?aff=zEdb](https://xiaoai.plus/register?aff=zEdb)，申请一个 key

# 使用
[使用 ChatGPT](https://www.yuque.com/qiaokate/su87gb/lpex0bogktiilhhc)

  
 

