> [https://blog.philip-huang.tech/?page=peft-overview](https://blog.philip-huang.tech/?page=peft-overview)
>
> github 地址：[https://github.com/huggingface/peft](https://github.com/huggingface/peft)
>

<font style="color:rgb(70, 70, 70);">语言模型（LM）技术已经实现一些重大突破，使得模型的规模更加庞大。然而，对大部分的人，要微调如此巨大的模型所需的门槛太高。 </font>`<font style="color:rgb(70, 70, 70);">Parameter-efficient fine-tuning</font>`<font style="color:rgb(70, 70, 70);">（PEFT）提供了一种新的训练方法，即通过训练一小组参数，使微调门槛降低，并且让模型能够适应和执行新的任务。</font>

# <font style="color:rgb(70, 70, 70);">LM fine-tuning 演进</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840311757-983e506b-fa81-434c-b702-b103c4fbbc84.png)

1. `<font style="color:rgb(70, 70, 70);">Full fine-tuning Transformer</font>`<font style="color:rgb(70, 70, 70);"> ：架构模型刚推出时(BERT,GPT, etc.)，普遍模型大小落在500M~700M左右，这时候高端的消费级显卡可以负担微调所需的硬体门槛。</font>
2. `<font style="color:rgb(70, 70, 70);">In-Context learning</font>`<font style="color:rgb(70, 70, 70);">： 随时间推进，LM研究开始堆叠模型参数，GPT-3推出的时候该模型有175B，一般的硬体设备连推论都无法负担，仅能透过特定API存取。而开发者OpenAI对于GPT-3应用到下游任务的解方便是In-Context learning。</font>
3. `<font style="color:rgb(70, 70, 70);">Parameter-efficient fine-tuning (PEFT)</font>`<font style="color:rgb(70, 70, 70);">：</font>`<font style="color:rgb(70, 70, 70);">In-Context learning</font>`<font style="color:rgb(70, 70, 70);">无法完整发挥模型能力，并且效率较低，有研究开始透过训练少量模型参数来达到微调效果，借此大幅度降低硬体负担并让模型变得更稳健与可控。</font>

# In-Context leaning
[In-Context Learning 是什么?](https://www.yuque.com/qiaokate/su87gb/btlpxkfg72iw8tky)

# <font style="color:rgb(70, 70, 70);">各种类型的PEFT</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840236624-e9c2cd1d-27f3-446e-a537-51a4942ca0e5.png)

## <font style="color:rgb(70, 70, 70);">Addition-based</font>
`<font style="color:rgb(70, 70, 70);">Additive methods</font>`<font style="color:rgb(70, 70, 70);"> 的主要思想是通过添加额外的参数或层来扩充现有的预训练模型，仅对新添加的参数进行训练。在这方面有两个主要类别，即</font>`<font style="color:rgb(70, 70, 70);">Adapter</font>`<font style="color:rgb(70, 70, 70);"> 和</font>`<font style="color:rgb(70, 70, 70);">soft prompt</font>`<font style="color:rgb(70, 70, 70);">。</font>

`<font style="color:rgb(70, 70, 70);">Adapter</font>`<font style="color:rgb(70, 70, 70);"> 涉及在</font>`<font style="color:rgb(70, 70, 70);">Transformer</font>`<font style="color:rgb(70, 70, 70);">架构中引入小的全连接可训练层，而</font>`<font style="color:rgb(70, 70, 70);">soft prompt</font>`<font style="color:rgb(70, 70, 70);"> 旨在通过保持其结构固定和冻结来修改输入</font>`<font style="color:rgb(70, 70, 70);">prompt</font>`<font style="color:rgb(70, 70, 70);"> ，从而控制LLM的行为。</font>

## <font style="color:rgb(70, 70, 70);">选择性方法（Selection-based）</font>
<font style="color:rgb(70, 70, 70);">选择性方法对模型的现有参数进行微调，这可以是基于层深度的选择、基于层类型的选择，甚至是个别参数的选择。其中一个例子是注意力调整。研究人员发现这些基于选择性的方法的性能有好有坏，并且在参数效率和计算效率之间存在明显的折衷。</font>

## <font style="color:rgb(70, 70, 70);">基于重新参数化的方法（Reparametrization-based）</font>
<font style="color:rgb(70, 70, 70);">基于重新参数化的PEFT 方法利用</font>`<font style="color:rgb(70, 70, 70);">low-rank approximation</font>`<font style="color:rgb(70, 70, 70);">性质来最小化可训练参数的数量。 </font>`<font style="color:rgb(70, 70, 70);">low-rank matrix</font>`<font style="color:rgb(70, 70, 70);"> 旨在捕捉高维数据的潜在</font>`<font style="color:rgb(70, 70, 70);">low-rank </font>`<font style="color:rgb(70, 70, 70);">结构。该方法的直觉是冻结原始LLM参数，通过建立新的</font>`<font style="color:rgb(70, 70, 70);">low-rank</font>`<font style="color:rgb(70, 70, 70);">转换并引入少量可训练参数。</font>

# <font style="color:rgb(70, 70, 70);">代表性PEFT 方法介绍</font>
## <font style="color:rgb(70, 70, 70);">BitFit</font>
<font style="color:rgb(70, 70, 70);">BitFit 仅微调网络的Bias。 BitFit仅更新模型约0.05％的参数量。</font>

```python
params = (p for n, pin model.named_parameters() if "bias" in n)
optimizer = Optimizer(params)
```

<font style="color:rgb(70, 70, 70);">BitFit 是一个非常简单的方法，但是性能表现较full fine-tuning 差。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840237084-caf312a4-fa01-46b0-96e0-8f46ba1372bd.png)

## <font style="color:rgb(70, 70, 70);">Prefix Tuning</font>
<font style="color:rgb(70, 70, 70);">Prefix Tuning 冻结了Transformer的参数，仅对prefix（即红色区块）进行优化。因此，我们只需为每个任务储存prefix，使得prefix tuning 更具模块化和节省空间的特点。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840237857-5926cd5f-ce41-4e36-aa4b-5f678484d5d5.png)

## <font style="color:rgb(70, 70, 70);">prompt tuning</font>
<font style="color:rgb(70, 70, 70);">prompt tuning，这是一种简单而有效的机制，用于学习"soft prompt" 以使冻结的语言模型能够执行特定的下游任务。与GPT-3使用的离散文本提示不同，软提示是通过反向传播学习的，可以调整以纳入来自任意数量的token 示例的讯号。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840237646-5d55eb4f-5911-4e92-98a8-3d8da8362f6a.png)

<font style="color:rgb(70, 70, 70);">prompt tuning使得单一模型能够透过将不同的提示嵌入简单地连接到批次中的每个范例来执行许多任务。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840237867-36e5f859-d372-4c2f-b3ea-70407878bf21.png)

## <font style="color:rgb(70, 70, 70);">SPoT</font>
<font style="color:rgb(70, 70, 70);">SPoT 改进了Prompt Tuning，对Prompt 加入预训练动作(在source task 上预训练，然后迁移到target task)，这项改动使得Soft Prompts 方法能够媲美Fine Tuning。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840238171-76137f0d-e2a9-468b-9838-51dab929a265.png)

## <font style="color:rgb(70, 70, 70);">Adapter</font>
<font style="color:rgb(70, 70, 70);">Adapter 是将一个小型可训练的前馈网路(feed-forward networks)插入到transformer-layer 之间。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840238889-53db42f8-4f8f-47aa-af05-20e58da6076c.png)

<font style="color:rgb(70, 70, 70);">Adapter 有效地向模型添加额外的（小型）层，导致计算成本和记忆体略微增加，但是这增加是不可忽视的。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840238917-785e3c28-d2d2-4964-9285-2ab97259013b.png)

<font style="color:rgb(70, 70, 70);">基于Adapter 的微调在所训练的参数数量上达到了与full fine-tunning 相似的性能，而所需参数数量少了两个数量级。</font>

## <font style="color:rgb(70, 70, 70);">LoRA</font>
![](https://cdn.nlark.com/yuque/0/2025/gif/2639475/1736841615228-0e534472-7c5d-43be-9d55-4c1a349ed8d8.gif)

<font style="color:rgb(70, 70, 70);">对于预训练权重在 </font>$ $W_0 \in \mathbb{R}^{d \times d}$ $，<font style="color:rgb(70, 70, 70);">将计算隐藏层数值的公式</font>

$ h = W_0 x $

修正成

$ h = W_0 x + \delta W x $

$ \delta W x = BAx
 $

$ B \in \mathbb{R}^{d \times r}, \quad A \in \mathbb{R}^{r \times d}

 $

<font style="color:rgb(70, 70, 70);">我们使用低秩矩阵分解 </font>$ BA $<font style="color:rgb(70, 70, 70);">来限制更新，并且</font>$ \text{rank} \ r \ll \min(d, k)$ $

<font style="color:rgb(70, 70, 70);">注意，</font>$ W_0
 $<font style="color:rgb(70, 70, 70);">和</font>$ \delta W = BA $<font style="color:rgb(70, 70, 70);">都与相同的输入相乘。在训练过程中，</font>$ W_0 $<font style="color:rgb(70, 70, 70);">被冻结并且不接收梯度更新。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840239815-f644892b-3402-429a-be7e-bcfbf15865f2.png)

<font style="color:rgb(70, 70, 70);">LoRA的表现优于比较对象，包括full fine-tuning。</font>

## <font style="color:rgb(70, 70, 70);">IA3</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840239750-545e7c46-357d-4833-bc09-5a69fb145138.png)

<font style="color:rgb(70, 70, 70);">具体来说修改了attention，对其加入</font>$ l_{k} $<font style="color:rgb(70, 70, 70);">和</font>$ l_{v} $

$ \text{softmax}\left(\frac{Q(K^T)}{\sqrt{d_k}}\right)(\nu \cdot V) $

<font style="color:rgb(70, 70, 70);">然后在</font>`<font style="color:rgb(70, 70, 70);">position-wise feed-forward networks</font>`<font style="color:rgb(70, 70, 70);"> 加入</font>$ l_{ff} $

$ l_{ff} = \gamma(W_1 x)W_2 $

$ \gamma $<font style="color:rgb(70, 70, 70);">是网路中的非线性层。</font>

<font style="color:rgb(70, 70, 70);">对每一个Transformer layer 都做了一样的改动。</font>

## <font style="color:rgb(70, 70, 70);">IA3 伪代码</font>
```python
def transformer_block_with_ia3(x):
    residual = x
    x = ia3_self_attention(x)
    x = LN(x + residual)
    residual = x
    x = x @ W_1 # FFN in
    x = l_ff * gelu(x) # (IA)3 scaling
    x = x @ W_2 # FFN out
    x = LN(x + residual)
    return x

def ia3_self_attention(x):
    k, q, v = x @ W_k, x @ W_q, x @ W_v
    k = l_k * k
    v = l_v * v
    return softmax(q @ k.T) @ V
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840240303-87984ea8-555a-4676-b79f-1f9442d2e227.png)

<font style="color:rgb(70, 70, 70);">在few-shot 资料集上面，IA3用少量参数赢过竞争对手，并且表现比full fine-tuning更好。</font>

## <font style="color:rgb(70, 70, 70);">Ladder Side-Tuning</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840240506-bac3cb18-659d-47ed-bd72-6bf799992be3.png)

<font style="color:rgb(70, 70, 70);">除了直接微调全部参数外，还有像Adapter、P-Tuning等许多参数高效的微调技巧，它们能够透过只微调很少的参数来达到接近全量参数微调的效果。 然而，这些技巧通常只是“参数高效”而并非“训练高效”。</font>

<font style="color:rgb(70, 70, 70);">LST它是在原有大模型的基础上搭建了一个「旁支」（梯子），将大模型的部分层输出作为旁枝模型的输入，所有的训练参数尽在旁枝模型中，由于大模型仅提供输入，因此反向传播的复杂度取决于旁枝模型的规模，并不需要直接在原始大模型上执行反向传播，因此是可以明显提升训练效率的。</font>

<font style="color:rgb(70, 70, 70);">段落文字引用自:</font><font style="color:rgb(70, 70, 70);"> </font>[<font style="color:rgb(70, 70, 70);">Ladder Side-Tuning：预训练模型的“过墙梯”</font>](https://spaces.ac.cn/archives/9138)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840241034-67202a43-6492-4621-a4c7-78ff36b0ab78.png)

<font style="color:rgb(70, 70, 70);">LST 套在T5-base 上的性能表现。 y轴表示8个GLUE任务的平均准确度，而x轴表示训练期间的GPU内存使用情况。</font>

# <font style="color:rgb(70, 70, 70);">比较</font>
## <font style="color:rgb(70, 70, 70);">存储效率、内存效率、计算效率、准确性和推理开销</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840241588-2553043d-d83c-4e7b-b427-167c6f5d6694.png)

+ <font style="color:rgb(70, 70, 70);">Storage: 是否有额外的储存开销</font>
+ <font style="color:rgb(70, 70, 70);">Memory: 是否有额外的记忆体开销</font>
+ <font style="color:rgb(70, 70, 70);">Backprop: 能否</font>**<font style="color:rgb(70, 70, 70);">减少</font>**<font style="color:rgb(70, 70, 70);">反向传播成本</font>
+ <font style="color:rgb(70, 70, 70);">Inference overhead: 额外的推论开销</font>

## <font style="color:rgb(70, 70, 70);">PEFT 方法的架构、修改位置</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840241833-dbf3c1a8-b2ed-4dd2-b749-c9cb30ab7cdc.png)

## <font style="color:rgb(70, 70, 70);">用统一视图比较架构差异</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840241938-2816ab23-70a6-448b-a26c-d948d72059b7.png)

<font style="color:rgb(70, 70, 70);">图中PLM 表示一个冻结的sublayer (eg attention or FFN)。</font>

## <font style="color:rgb(70, 70, 70);">序列或是平行结构?</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840242315-a5dc1715-36a9-407a-82da-e8dd2c8d07ef.png)

<font style="color:rgb(70, 70, 70);">上图展示了Transformer 的结构，包含serial adapter (SA) 和parallel adapter (PA)。相较于serial 设计，parallel adapter 设计在Transformer 层之前，而非在层之后。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840243396-4247db7a-e301-42d2-a1a7-08a80872c1eb.png)

<font style="color:rgb(70, 70, 70);">所有的测试项目表现，PA都胜过SA。</font>

## <font style="color:rgb(70, 70, 70);">在Attention 还是FFN 加入adapter?</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840243694-92da6f47-9535-43e8-bf52-c8648ad6fd47.png)

<font style="color:rgb(70, 70, 70);">在FFN 中加入adapter 会比在Attn 中更优。同时，值得注意的是，当进一步增加容量时，prefix tuning 并未持续显示改善的趋势，这一现象在Li 和Liang（2021）的研究中也有所观察。这些研究结果暗示，与Attn 相比，FFN 的修改似乎能更有效地利用新增的参数，无论功能形式或组合函数为何。作者假设这可能是因为FFN学习了任务特定的文本内容(task-specific textual patterns)，而注意力则学习了与应该关注什么样的重点，并不需要大容量来适应新的任务。</font>

# <font style="color:rgb(70, 70, 70);">回报与比较议题</font>
<font style="color:rgb(70, 70, 70);">我们发现了一些值得讨论的挑战和不一致之处。这些挑战使得难以直接比较方法并评估它们的真实性能。</font>

## <font style="color:rgb(70, 70, 70);">参数统计不一致</font>
<font style="color:rgb(70, 70, 70);">一般来说参数统计可以分为三种类别</font>

+ <font style="color:rgb(70, 70, 70);">可训练参数的数量</font>
+ <font style="color:rgb(70, 70, 70);">被改变的参数(原始模型和微调模型)</font>
+ <font style="color:rgb(70, 70, 70);">原始模型和微调模型的差异等级(rank)</font>

<font style="color:rgb(70, 70, 70);">这些区别可能具有重要的影响。例如，IntrinsicSAID 学习模型参数的低秩转换。然而，它改变了模型的所有参数。 DiffPruning 学习了0.5%参数的更新，但实际上它训练了200%的参数：微调模型并学习二进制遮罩。对于基于重新参数化的方法(Reparametrization-based methods)，内存需求可能会根据实现设计选择而变。</font>

<font style="color:rgb(70, 70, 70);">总的来说，因为各方法在概念、架构与实现方式等差异巨大，所以难以直接比较。</font>

## <font style="color:rgb(70, 70, 70);">模型大小</font>
<font style="color:rgb(70, 70, 70);">当比较PEFT方法时，模型大小需要被考虑进去，不仅只是可训练参数的比例，最好也包含可训练参数量的实际数值。</font>

## <font style="color:rgb(70, 70, 70);">缺乏标准的测试基准和指标</font>
<font style="color:rgb(70, 70, 70);">缺乏标准基准和指标进一步使比较变得复杂。新方法通常在不同的模型/数据集组合上进行评估，这使得很难得出有意义的结论。</font>

## <font style="color:rgb(70, 70, 70);">缺少更全面或客观的比较</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840244097-a2c53ca6-2451-485c-bd1e-28fa29e79fd5.png)

<font style="color:rgb(70, 70, 70);">在四个任务上的结果进行总览。尽管现有方法在微调少于1%的参数时可以在MNLI和SST2上达到与full fine-tuning 竞争性表现，但如果在XSum和en-ro中增加5%的参数，仍然存在较大的差距。</font>

<font style="color:rgb(70, 70, 70);">即使将相对参数大小增加到>10%，差距仍然显著；而且在高资源的机器翻译任务上观察到更大的差距。</font>

<font style="color:rgb(70, 70, 70);">这表明许多声称在GLUE基准上使用encoder-only 的模型或在相对简单的生成基准上声称与full fine-tuning 相媲美的方法，可能无法很好地推广到其他标准基准。</font>

## <font style="color:rgb(70, 70, 70);">各类方法开源实作问题</font>
<font style="color:rgb(70, 70, 70);">许多作者的程式码品质欠佳，并且缺少文件或是范例，导致难以复用。</font>

<font style="color:rgb(70, 70, 70);">好在仍然有</font>[<font style="color:rgb(70, 70, 70);">开源专案-HF PEFT</font>](https://github.com/huggingface/peft)<font style="color:rgb(70, 70, 70);">在这方面进行努力，它整合各种SOTA方法与支援各类型LM。</font>

# <font style="color:rgb(70, 70, 70);">参考</font>
+ [<font style="color:rgb(70, 70, 70);">缩小规模到扩大规模：参数高效微调指南</font>](https://arxiv.org/abs/2303.15647.pdf)
+ [<font style="color:rgb(70, 70, 70);">重新思考演示的作用：什么使得情境学习有效？</font>](https://arxiv.org/abs/2202.12837.pdf)
+ [<font style="color:rgb(70, 70, 70);">LoRA：大型语言模型的低秩自适应</font>](https://arxiv.org/abs/2106.09685.pdf)
+ [<font style="color:rgb(70, 70, 70);">用于 NLP 的参数高效迁移学习</font>](https://arxiv.org/abs/1902.00751.pdf)
+ [<font style="color:rgb(70, 70, 70);">参数高效快速调优的规模力量</font>](https://arxiv.org/abs/2104.08691.pdf)
+ [<font style="color:rgb(70, 70, 70);">SPoT：通过软提示迁移实现更好的冻结模型自适应</font>](https://arxiv.org/abs/2110.07904)
+ [<font style="color:rgb(70, 70, 70);">少量参数高效微调比上下文学习更好、更便宜</font>](https://arxiv.org/abs/2205.05638.pdf)
+ [<font style="color:rgb(70, 70, 70);">BitFit：基于 Transformer 的掩码语言模型的简单参数高效微调</font>](https://arxiv.org/abs/2106.10199)
+ [<font style="color:rgb(70, 70, 70);">前缀调整：优化连续提示以进行生成</font>](https://arxiv.org/abs/2101.00190)
+ [<font style="color:rgb(70, 70, 70);">LST：阶梯侧调，实现参数和内存高效的迁移学习</font>](https://arxiv.org/abs/2206.06522)
+ [<font style="color:rgb(70, 70, 70);">迈向参数高效迁移学习的统一视角</font>](https://arxiv.org/abs/2110.04366.pdf)
+ [<font style="color:rgb(70, 70, 70);">用于多语言机器翻译的反干扰适配器</font>](https://arxiv.org/abs/2104.08154.pdf)
+ [<font style="color:rgb(70, 70, 70);">从您的业务数据中获取洞察力 - 使用</font><font style="color:rgb(70, 70, 70);">🤗</font><font style="color:rgb(70, 70, 70);"> Hugging Face 通过 PEFT（使用 LoRA）构建 LLM 应用程序</font>](https://www.linkedin.com/pulse/get-insight-from-your-business-data-build-llm-application-jain-2f)
+ [<font style="color:rgb(70, 70, 70);">Ladder Side-Tuning：预训练模型的“过墙梯”</font>](https://spaces.ac.cn/archives/9138)

