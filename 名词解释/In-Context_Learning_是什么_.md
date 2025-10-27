> `<font style="color:rgb(70, 70, 70);">In-Context learning</font>`<font style="color:rgb(70, 70, 70);"> (</font>`<font style="color:rgb(70, 70, 70);">Prompt Design</font>`<font style="color:rgb(70, 70, 70);">)</font>
>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840236152-125b7fce-e812-4a07-8343-7b8a7e5b4f34.png)

<font style="color:rgb(70, 70, 70);">In-Context learning中的范例其实包含四个不同面向：输入标签映射、输入文本的分布、标签空间以及将输入标签配对。</font>

<font style="color:rgb(70, 70, 70);">少样本情境学习（In-Context learning）使预先训练的语言模型能够在没有任何基于梯度的训练的情况下执行先前未见过的任务，方法是将少量的训练示例作为输入的一部分。 In-Context learning 产生相当大的计算、记忆体和存储成本，因为它涉及在每次进行预测时处理所有的训练示例。</font>

## <font style="color:rgb(70, 70, 70);">In-Context learning的优点</font>
`<font style="color:rgb(70, 70, 70);">In-Context learning</font>`<font style="color:rgb(70, 70, 70);">方法允许模型马上执行许多任务，并且无须微调。 </font>`<font style="color:rgb(70, 70, 70);">In-Context learning</font>`<font style="color:rgb(70, 70, 70);">通常适用于受限标记资料的任务类型(又称</font>`<font style="color:rgb(70, 70, 70);">few-shot learning</font>`<font style="color:rgb(70, 70, 70);">)。</font>`<font style="color:rgb(70, 70, 70);"> In-Context learning</font>`<font style="color:rgb(70, 70, 70);">具有</font>`<font style="color:rgb(70, 70, 70);">data-efficient</font>`<font style="color:rgb(70, 70, 70);">。</font>

## <font style="color:rgb(70, 70, 70);">In-Context learning 的缺点</font>
<font style="color:rgb(70, 70, 70);">首先，处理所有的提示输入会使模型产生高昂的计算成本。其次，通常情况下，使用</font>`<font style="color:rgb(70, 70, 70);">In-Context learning</font>`<font style="color:rgb(70, 70, 70);">只能提供比微调性能稍次的结果。最后，输入格式或提供的范例的不同可能极大地影响模型的性能，使得一些模型行为变得难以预测。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736840236161-06f92b34-648d-4406-b5a5-06ba9e9b840a.png)

<font style="color:rgb(70, 70, 70);">在IA3的文献中提到，</font>`<font style="color:rgb(70, 70, 70);">PEFT</font>`<font style="color:rgb(70, 70, 70);">方法(表中T-Few )比</font>`<font style="color:rgb(70, 70, 70);">In-Context learning</font>`<font style="color:rgb(70, 70, 70);">需要的计算力更低。 即便算上训练算力，在</font>`<font style="color:rgb(70, 70, 70);">In-Context learning</font>`<font style="color:rgb(70, 70, 70);">使用超过</font>`<font style="color:rgb(70, 70, 70);">20</font>`<font style="color:rgb(70, 70, 70);">个</font>`<font style="color:rgb(70, 70, 70);">few-shot sample</font>`<font style="color:rgb(70, 70, 70);"> 后优势便不存在。</font>

