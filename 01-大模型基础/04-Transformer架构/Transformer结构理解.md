> [https://baijiahao.baidu.com/s?id=1651219987457222196&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1651219987457222196&wfr=spider&for=pc)
>

## <font style="color:rgb(25, 27, 31);">Transformer 整体结构</font>
<font style="color:rgb(25, 27, 31);">首先介绍 Transformer 的整体结构，下图是 Transformer 用于中英文翻译的整体结构：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079876040-77d07fca-376d-43d7-bcac-57d87f6f88a4.png)

<font style="color:rgb(145, 150, 161);">Transformer 的整体结构，左图Encoder和右图Decoder</font>

<font style="color:rgb(25, 27, 31);">可以看到</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Transformer 由 Encoder 和 Decoder 两个部分组成</font>**<font style="color:rgb(25, 27, 31);">，Encoder 和 Decoder 都包含 6 个 block。Transformer 的工作流程大体如下：</font>

**<font style="color:rgb(25, 27, 31);">第一步：</font>**<font style="color:rgb(25, 27, 31);">获取输入句子的每一个单词的表示向量</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">，</font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">由单词的 Embedding（Embedding就是从原始数据提取出来的Feature） 和单词位置的 Embedding 相加得到。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079875356-09ff1554-76a0-45e5-848a-ad1aff9c73c0.png)

<font style="color:rgb(145, 150, 161);">Transformer 的输入表示</font>

**<font style="color:rgb(25, 27, 31);">第二步：</font>**<font style="color:rgb(25, 27, 31);">将得到的单词表示向量矩阵 (如上图所示，每一行是一个单词的表示</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">x</font>**<font style="color:rgb(25, 27, 31);">) 传入 Encoder 中，经过 6 个 Encoder block 后可以得到句子所有单词的编码信息矩阵</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">C</font>**<font style="color:rgb(25, 27, 31);">，如下图。单词向量矩阵用</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">X</font><font style="color:rgb(25, 27, 31);">n</font><font style="color:rgb(25, 27, 31);">×</font><font style="color:rgb(25, 27, 31);">d</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">表示， n 是句子中单词个数，d 是表示向量的维度 (论文中 d=512)。每一个 Encoder block 输出的矩阵维度与输入完全一致。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079875586-f9c02195-4ce0-4b54-a566-2c1ae847d7ab.png)

<font style="color:rgb(145, 150, 161);">Transformer Encoder 编码句子信息</font>

**<font style="color:rgb(25, 27, 31);">第三步</font>**<font style="color:rgb(25, 27, 31);">：将 Encoder 输出的编码信息矩阵</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">C</font>**<font style="color:rgb(25, 27, 31);">传递到 Decoder 中，Decoder 依次会根据当前翻译过的单词 1~ i 翻译下一个单词 i+1，如下图所示。在使用的过程中，翻译到单词 i+1 的时候需要通过</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Mask (掩盖)</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">操作遮盖住 i+1 之后的单词。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079874620-1ecf5e7a-1ef2-4aae-a40b-88c81f37272e.png)

<font style="color:rgb(145, 150, 161);">Transofrmer Decoder 预测</font>

<font style="color:rgb(25, 27, 31);">上图 Decoder 接收了 Encoder 的编码矩阵</font>**<font style="color:rgb(25, 27, 31);"> </font>****<font style="color:rgb(25, 27, 31);">C</font>**<font style="color:rgb(25, 27, 31);">，然后首先输入一个翻译开始符 "<Begin>"，预测第一个单词 "I"；然后输入翻译开始符 "<Begin>" 和单词 "I"，预测单词 "have"，以此类推。这是 Transformer 使用时候的大致流程，接下来是里面各个部分的细节。</font>

## <font style="color:rgb(25, 27, 31);">Transformer 的输入</font>
<font style="color:rgb(25, 27, 31);">Transformer 中单词的输入表示</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">x</font>**<font style="color:rgb(25, 27, 31);">由</font>**<font style="color:rgb(25, 27, 31);">单词 Embedding</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">和</font>**<font style="color:rgb(25, 27, 31);">位置 Embedding</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">（Positional Encoding）相加得到。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079875510-98f4a44f-b39c-4686-92c9-8215317ce37d.png)

<font style="color:rgb(145, 150, 161);">Transformer 的输入表示</font>

### <font style="color:rgb(25, 27, 31);">单词 Embedding</font>
<font style="color:rgb(25, 27, 31);">单词的 Embedding 有很多种方式可以获取，例如可以采用 Word2Vec、Glove 等算法预训练得到，也可以在 Transformer 中训练得到。</font>

### <font style="color:rgb(25, 27, 31);">位置 Embedding</font>
<font style="color:rgb(25, 27, 31);">Transformer 中除了单词的 Embedding，还需要使用位置 Embedding 表示单词出现在句子中的位置。</font>**<font style="color:rgb(25, 27, 31);">因为 Transformer 不采用 RNN 的结构，而是使用全局信息，不能利用单词的顺序信息，而这部分信息对于 NLP 来说非常重要。</font>**<font style="color:rgb(25, 27, 31);">所以 Transformer 中使用位置 Embedding 保存单词在序列中的相对或绝对位置。</font>

<font style="color:rgb(25, 27, 31);">位置 Embedding 用</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">PE</font>**<font style="color:rgb(25, 27, 31);">表示，</font>**<font style="color:rgb(25, 27, 31);">PE</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">的维度与单词 Embedding 是一样的。PE 可以通过训练得到，也可以使用某种公式计算得到。在 Transformer 中采用了后者，计算公式如下：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079875265-a54cfb16-9349-4ac5-a074-39f5146c317d.png)

<font style="color:rgb(25, 27, 31);">其中，pos 表示单词在句子中的位置，d 表示 PE的维度 (与词 Embedding 一样)，2i 表示偶数的维度，2i+1 表示奇数维度 (即 2i≤d, 2i+1≤d)。使用这种公式计算 PE 有以下的好处：</font>

+ <font style="color:rgb(25, 27, 31);">使 PE 能够适应比训练集里面所有句子更长的句子，假设训练集里面最长的句子是有 20 个单词，突然来了一个长度为 21 的句子，则使用公式计算的方法可以计算出第 21 位的 Embedding。</font>
+ <font style="color:rgb(25, 27, 31);">可以让模型容易地计算出相对位置，对于固定长度的间距 k，</font>**<font style="color:rgb(25, 27, 31);">PE(pos+k)</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">可以用</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">PE(pos)</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">计算得到。因为 Sin(A+B) = Sin(A)Cos(B) + Cos(A)Sin(B), Cos(A+B) = Cos(A)Cos(B) - Sin(A)Sin(B)。</font>

<font style="color:rgb(25, 27, 31);">将单词的词 Embedding 和位置 Embedding 相加，就可以得到单词的表示向量</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">x</font>**<font style="color:rgb(25, 27, 31);">，</font>**<font style="color:rgb(25, 27, 31);">x</font>****<font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">就是 Transformer 的输入。</font>

## <font style="color:rgb(25, 27, 31);">Self-Attention（自注意力机制）</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079876762-a82f7979-8d4f-4e55-b25e-2b1cd3da1832.png)

<font style="color:rgb(145, 150, 161);">Transformer Encoder 和 Decoder</font>

<font style="color:rgb(25, 27, 31);">上图是论文中 Transformer 的内部结构图，左侧为 Encoder block，右侧为 Decoder block。红色圈中的部分为</font>**<font style="color:rgb(25, 27, 31);"> </font>****<font style="color:rgb(25, 27, 31);">Multi-Head Attention</font>**<font style="color:rgb(25, 27, 31);">，是由多个</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Self-Attention</font>**<font style="color:rgb(25, 27, 31);">组成的，可以看到 Encoder block 包含一个 Multi-Head Attention，而 Decoder block 包含两个 Multi-Head Attention (其中有一个用到 Masked)。Multi-Head Attention 上方还包括一个 Add & Norm 层，Add 表示残差连接 (</font>[<font style="color:rgb(25, 27, 31);">Residual Connection</font>](https://zhida.zhihu.com/search?content_id=163422979&content_type=Article&match_order=1&q=Residual+Connection&zhida_source=entity)<font style="color:rgb(25, 27, 31);">) 用于防止网络退化，Norm 表示</font><font style="color:rgb(25, 27, 31);"> </font>[<font style="color:rgb(25, 27, 31);">Layer Normalization</font>](https://zhida.zhihu.com/search?content_id=163422979&content_type=Article&match_order=1&q=Layer+Normalization&zhida_source=entity)<font style="color:rgb(25, 27, 31);">，用于对每一层的激活值进行归一化。</font>

<font style="color:rgb(25, 27, 31);">因为</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Self-Attention</font>**<font style="color:rgb(25, 27, 31);">是 Transformer 的重点，所以我们重点关注 Multi-Head Attention 以及 Self-Attention，首先详细了解一下 Self-Attention 的内部逻辑。</font>

### <font style="color:rgb(25, 27, 31);">Self-Attention 结构</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079876012-cb80f1be-276a-4a71-a9be-32c820d4544d.png)

<font style="color:rgb(145, 150, 161);">Self-Attention 结构</font>

<font style="color:rgb(25, 27, 31);">上图是 Self-Attention 的结构，在计算的时候需要用到矩阵</font>**<font style="color:rgb(25, 27, 31);">Q(查询),K(键值),V(值)</font>**<font style="color:rgb(25, 27, 31);">。在实际中，Self-Attention 接收的是输入(单词的表示向量x组成的矩阵X) 或者上一个 Encoder block 的输出。而</font>**<font style="color:rgb(25, 27, 31);">Q,K,V</font>**<font style="color:rgb(25, 27, 31);">正是通过 Self-Attention 的输入进行线性变换得到的。</font>

### <font style="color:rgb(25, 27, 31);">Q, K, V 的计算</font>
<font style="color:rgb(25, 27, 31);">Self-Attention 的输入用矩阵X进行表示，则可以使用线性变阵矩阵</font>**<font style="color:rgb(25, 27, 31);">WQ,WK,WV</font>**<font style="color:rgb(25, 27, 31);">计算得到</font>**<font style="color:rgb(25, 27, 31);">Q,K,V</font>**<font style="color:rgb(25, 27, 31);">。计算如下图所示，</font>**<font style="color:rgb(25, 27, 31);">注意 X, Q, K, V 的每一行都表示一个单词。</font>**

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079876124-42e5f9ad-5031-46c1-9172-79e0476aee30.png)

<font style="color:rgb(145, 150, 161);">Q, K, V 的计算</font>

### <font style="color:rgb(25, 27, 31);">3.3 Self-Attention 的输出</font>
<font style="color:rgb(25, 27, 31);">得到矩阵 Q, K, V之后就可以计算出 Self-Attention 的输出了，计算的公式如下：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079875969-f719b9ef-9285-4f8c-bf18-2373c642d36f.png)

<font style="color:rgb(145, 150, 161);">Self-Attention 的输出</font>

<font style="color:rgb(25, 27, 31);">公式中计算矩阵</font>**<font style="color:rgb(25, 27, 31);">Q</font>**<font style="color:rgb(25, 27, 31);">和</font>**<font style="color:rgb(25, 27, 31);">K</font>**<font style="color:rgb(25, 27, 31);">每一行向量的内积，为了防止内积过大，因此除以</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">d</font><font style="color:rgb(25, 27, 31);">k</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">的平方根。</font>**<font style="color:rgb(25, 27, 31);">Q</font>**<font style="color:rgb(25, 27, 31);">乘以</font>**<font style="color:rgb(25, 27, 31);">K</font>**<font style="color:rgb(25, 27, 31);">的转置后，得到的矩阵行列数都为 n，n 为句子单词数，这个矩阵可以表示单词之间的 attention 强度。下图为</font>**<font style="color:rgb(25, 27, 31);">Q</font>**<font style="color:rgb(25, 27, 31);">乘以</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">K</font><font style="color:rgb(25, 27, 31);">T</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">，1234 表示的是句子中的单词。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079876578-746cc2c2-11e2-48e9-947b-c79cb20ac9fb.png)

<font style="color:rgb(145, 150, 161);">Q乘以K的转置的计算</font>

<font style="color:rgb(25, 27, 31);">得到</font><font style="color:rgb(25, 27, 31);">Q</font><font style="color:rgb(25, 27, 31);">K</font><font style="color:rgb(25, 27, 31);">T</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">之后，使用 Softmax 计算每一个单词对于其他单词的 attention 系数，公式中的 Softmax 是对矩阵的每一行进行 Softmax，即每一行的和都变为 1.</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079876569-f5333f11-4e87-48d1-801f-adb9071931e8.png)

<font style="color:rgb(145, 150, 161);">对矩阵的每一行进行 Softmax</font>

<font style="color:rgb(25, 27, 31);">得到 Softmax 矩阵之后可以和</font>**<font style="color:rgb(25, 27, 31);">V</font>**<font style="color:rgb(25, 27, 31);">相乘，得到最终的输出</font>**<font style="color:rgb(25, 27, 31);">Z</font>**<font style="color:rgb(25, 27, 31);">。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079876570-b85aed64-490e-4625-9000-0be24efc043b.png)

<font style="color:rgb(145, 150, 161);">Self-Attention 输出</font>

<font style="color:rgb(25, 27, 31);">上图中 Softmax 矩阵的第 1 行表示单词 1 与其他所有单词的 attention 系数，最终单词 1 的输出</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">Z</font><font style="color:rgb(25, 27, 31);">1</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">等于所有单词 i 的值</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">V</font><font style="color:rgb(25, 27, 31);">i</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">根据 attention 系数的比例加在一起得到，如下图所示：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079876746-93cc2dd1-c46c-4334-acd8-0aab74ff7020.png)

<font style="color:rgb(145, 150, 161);">Zi 的计算方法</font>

### <font style="color:rgb(25, 27, 31);">3.4 Multi-Head Attention</font>
<font style="color:rgb(25, 27, 31);">在上一步，我们已经知道怎么通过 Self-Attention 计算得到输出矩阵 Z，而 Multi-Head Attention 是由多个 Self-Attention 组合形成的，下图是论文中 Multi-Head Attention 的结构图。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079877930-9c0d4cf5-a3ce-49df-9442-eae2d37da70e.png)

<font style="color:rgb(145, 150, 161);">Multi-Head Attention</font>

<font style="color:rgb(25, 27, 31);">从上图可以看到 Multi-Head Attention 包含多个 Self-Attention 层，首先将输入</font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">分别传递到 h 个不同的 Self-Attention 中，计算得到 h 个输出矩阵</font>**<font style="color:rgb(25, 27, 31);">Z</font>**<font style="color:rgb(25, 27, 31);">。下图是 h=8 时候的情况，此时会得到 8 个输出矩阵</font>**<font style="color:rgb(25, 27, 31);">Z</font>**<font style="color:rgb(25, 27, 31);">。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079877774-0c3fe717-dfb2-4832-ba3e-94c31ea13354.png)

<font style="color:rgb(145, 150, 161);">多个 Self-Attention</font>

<font style="color:rgb(25, 27, 31);">得到 8 个输出矩阵</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">Z</font><font style="color:rgb(25, 27, 31);">1</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">到</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">Z</font><font style="color:rgb(25, 27, 31);">8</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">之后，Multi-Head Attention 将它们拼接在一起</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">(Concat)</font>**<font style="color:rgb(25, 27, 31);">，然后传入一个</font>**<font style="color:rgb(25, 27, 31);">Linear</font>**<font style="color:rgb(25, 27, 31);">层，得到 Multi-Head Attention 最终的输出</font>**<font style="color:rgb(25, 27, 31);">Z</font>**<font style="color:rgb(25, 27, 31);">。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079877738-6d463697-3de0-41f7-969f-2e0b6ae7f813.png)

<font style="color:rgb(145, 150, 161);">Multi-Head Attention 的输出</font>

<font style="color:rgb(25, 27, 31);">可以看到 Multi-Head Attention 输出的矩阵</font>**<font style="color:rgb(25, 27, 31);">Z</font>**<font style="color:rgb(25, 27, 31);">与其输入的矩阵</font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">的维度是一样的。</font>

## <font style="color:rgb(25, 27, 31);">4. Encoder 结构</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079877945-29f75d3d-26d2-475e-a04b-129b8ebab040.png)

<font style="color:rgb(145, 150, 161);">Transformer Encoder block</font>

<font style="color:rgb(25, 27, 31);">上图红色部分是 Transformer 的 Encoder block 结构，可以看到是由 Multi-Head Attention,</font>**<font style="color:rgb(25, 27, 31);"> </font>****<font style="color:rgb(25, 27, 31);">Add & Norm, Feed Forward, Add & Norm</font>****<font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">组成的。刚刚已经了解了 Multi-Head Attention 的计算过程，现在了解一下 Add & Norm 和 Feed Forward 部分。</font>

### <font style="color:rgb(25, 27, 31);">4.1 Add & Norm</font>
<font style="color:rgb(25, 27, 31);">Add & Norm 层由 Add 和 Norm 两部分组成，其计算公式如下：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079877791-69fae550-549e-4b05-9b69-c16e9e400840.png)

<font style="color:rgb(145, 150, 161);">Add &amp;amp;amp;amp;amp; Norm 公式</font>

<font style="color:rgb(25, 27, 31);">其中</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">表示 Multi-Head Attention 或者 Feed Forward 的输入，MultiHeadAttention(</font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">) 和 FeedForward(</font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">) 表示输出 (输出与输入</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">X</font>****<font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">维度是一样的，所以可以相加)。</font>

**<font style="color:rgb(25, 27, 31);">Add</font>**<font style="color:rgb(25, 27, 31);">指</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">+MultiHeadAttention(</font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">)，是一种残差连接，通常用于解决多层网络训练的问题，可以让网络只关注当前差异的部分，在 ResNet 中经常用到：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079878012-9489dc70-9844-45f7-a973-a46244b0adac.png)

<font style="color:rgb(145, 150, 161);">残差连接</font>

**<font style="color:rgb(25, 27, 31);">Norm</font>**<font style="color:rgb(25, 27, 31);">指 Layer Normalization，通常用于 RNN 结构，Layer Normalization 会将每一层神经元的输入都转成均值方差都一样的，这样可以加快收敛。</font>

### <font style="color:rgb(25, 27, 31);">4.2 Feed Forward</font>
<font style="color:rgb(25, 27, 31);">Feed Forward 层比较简单，是一个两层的全连接层，第一层的激活函数为 Relu，第二层不使用激活函数，对应的公式如下。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079878658-7fd2fb3a-a385-45e0-8d86-ead5bed1ebcc.png)

<font style="color:rgb(145, 150, 161);">Feed Forward</font>

**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">是输入，Feed Forward 最终得到的输出矩阵的维度与</font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">一致。</font>

### <font style="color:rgb(25, 27, 31);">4.3 组成 Encoder</font>
<font style="color:rgb(25, 27, 31);">通过上面描述的 Multi-Head Attention, Feed Forward, Add & Norm 就可以构造出一个 Encoder block，Encoder block 接收输入矩阵</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">X</font><font style="color:rgb(25, 27, 31);">(</font><font style="color:rgb(25, 27, 31);">n</font><font style="color:rgb(25, 27, 31);">×</font><font style="color:rgb(25, 27, 31);">d</font><font style="color:rgb(25, 27, 31);">)</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">，并输出一个矩阵</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">O</font><font style="color:rgb(25, 27, 31);">(</font><font style="color:rgb(25, 27, 31);">n</font><font style="color:rgb(25, 27, 31);">×</font><font style="color:rgb(25, 27, 31);">d</font><font style="color:rgb(25, 27, 31);">)</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">。通过多个 Encoder block 叠加就可以组成 Encoder。</font>

<font style="color:rgb(25, 27, 31);">第一个 Encoder block 的输入为句子单词的表示向量矩阵，后续 Encoder block 的输入是前一个 Encoder block 的输出，最后一个 Encoder block 输出的矩阵就是</font>**<font style="color:rgb(25, 27, 31);">编码信息矩阵 C</font>**<font style="color:rgb(25, 27, 31);">，这一矩阵后续会用到 Decoder 中。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079879106-f20772f3-0dbd-4360-9dcc-48c3b19e2d0b.png)

<font style="color:rgb(145, 150, 161);">Encoder 编码句子信息</font>

## <font style="color:rgb(25, 27, 31);">5. Decoder 结构</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079879207-9d8b8184-1161-4c42-b48e-a6c710b5922c.png)

<font style="color:rgb(145, 150, 161);">Transformer Decoder block</font>

<font style="color:rgb(25, 27, 31);">上图红色部分为 Transformer 的 Decoder block 结构，与 Encoder block 相似，但是存在一些区别：</font>

+ <font style="color:rgb(25, 27, 31);">包含两个 Multi-Head Attention 层。</font>
+ <font style="color:rgb(25, 27, 31);">第一个 Multi-Head Attention 层采用了 Masked 操作。</font>
+ <font style="color:rgb(25, 27, 31);">第二个 Multi-Head Attention 层的</font>**<font style="color:rgb(25, 27, 31);">K, V</font>**<font style="color:rgb(25, 27, 31);">矩阵使用 Encoder 的</font>**<font style="color:rgb(25, 27, 31);">编码信息矩阵C</font>**<font style="color:rgb(25, 27, 31);">进行计算，而</font>**<font style="color:rgb(25, 27, 31);">Q</font>**<font style="color:rgb(25, 27, 31);">使用上一个 Decoder block 的输出计算。</font>
+ <font style="color:rgb(25, 27, 31);">最后有一个 Softmax 层计算下一个翻译单词的概率。</font>

### <font style="color:rgb(25, 27, 31);">5.1 第一个 Multi-Head Attention</font>
<font style="color:rgb(25, 27, 31);">Decoder block 的第一个 Multi-Head Attention 采用了 Masked 操作，因为在翻译的过程中是顺序翻译的，即翻译完第 i 个单词，才可以翻译第 i+1 个单词。通过 Masked 操作可以防止第 i 个单词知道 i+1 个单词之后的信息。下面以 "我有一只猫" 翻译成 "I have a cat" 为例，了解一下 Masked 操作。</font>

<font style="color:rgb(25, 27, 31);">下面的描述中使用了类似</font><font style="color:rgb(25, 27, 31);"> </font>[<font style="color:rgb(25, 27, 31);">Teacher Forcing</font>](https://zhida.zhihu.com/search?content_id=163422979&content_type=Article&match_order=1&q=Teacher+Forcing&zhida_source=entity)<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">的概念，不熟悉 Teacher Forcing 的童鞋可以参考以下上一篇文章Seq2Seq 模型详解。在 Decoder 的时候，是需要根据之前的翻译，求解当前最有可能的翻译，如下图所示。首先根据输入 "<Begin>" 预测出第一个单词为 "I"，然后根据输入 "<Begin> I" 预测下一个单词 "have"。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079878979-b644abba-0c17-40ec-9a3c-09d78a050ff1.png)

<font style="color:rgb(145, 150, 161);">Decoder 预测</font>

<font style="color:rgb(25, 27, 31);">Decoder 可以在训练的过程中使用 Teacher Forcing 并且并行化训练，即将正确的单词序列 (<Begin> I have a cat) 和对应输出 (I have a cat <end>) 传递到 Decoder。那么在预测第 i 个输出时，就要将第 i+1 之后的单词掩盖住，</font>**<font style="color:rgb(25, 27, 31);">注意 Mask 操作是在 Self-Attention 的 Softmax 之前使用的，下面用 0 1 2 3 4 5 分别表示 "<Begin> I have a cat <end>"。</font>**

**<font style="color:rgb(25, 27, 31);">第一步：</font>**<font style="color:rgb(25, 27, 31);">是 Decoder 的输入矩阵和</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Mask</font>****<font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">矩阵，输入矩阵包含 "<Begin> I have a cat" (0, 1, 2, 3, 4) 五个单词的表示向量，</font>**<font style="color:rgb(25, 27, 31);">Mask</font>****<font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">是一个 5×5 的矩阵。在</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Mask</font>****<font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">可以发现单词 0 只能使用单词 0 的信息，而单词 1 可以使用单词 0, 1 的信息，即只能使用之前的信息。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079878954-c643e2b7-59cd-4688-92fe-085c07059233.png)

<font style="color:rgb(145, 150, 161);">输入矩阵与 Mask 矩阵</font>

**<font style="color:rgb(25, 27, 31);">第二步：</font>**<font style="color:rgb(25, 27, 31);">接下来的操作和之前的 Self-Attention 一样，通过输入矩阵</font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">计算得到</font>**<font style="color:rgb(25, 27, 31);">Q,K,V</font>**<font style="color:rgb(25, 27, 31);">矩阵。然后计算</font>**<font style="color:rgb(25, 27, 31);">Q</font>**<font style="color:rgb(25, 27, 31);">和</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">K</font><font style="color:rgb(25, 27, 31);">T</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">的乘积</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">Q</font><font style="color:rgb(25, 27, 31);">K</font><font style="color:rgb(25, 27, 31);">T</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079879163-cbe76cfc-c23e-48be-8e25-f7bc799cf918.png)

<font style="color:rgb(145, 150, 161);">Q乘以K的转置</font>

**<font style="color:rgb(25, 27, 31);">第三步：</font>**<font style="color:rgb(25, 27, 31);">在得到</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">Q</font><font style="color:rgb(25, 27, 31);">K</font><font style="color:rgb(25, 27, 31);">T</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">之后需要进行 Softmax，计算 attention score，我们在 Softmax 之前需要使用</font>**<font style="color:rgb(25, 27, 31);">Mask</font>**<font style="color:rgb(25, 27, 31);">矩阵遮挡住每一个单词之后的信息，遮挡操作如下：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079879743-53777f82-6c06-437c-863a-4bf03e25376c.png)

<font style="color:rgb(145, 150, 161);">Softmax 之前 Mask</font>

<font style="color:rgb(25, 27, 31);">得到</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Mask</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">Q</font><font style="color:rgb(25, 27, 31);">K</font><font style="color:rgb(25, 27, 31);">T</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">之后在</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Mask</font>****<font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Q</font><font style="color:rgb(25, 27, 31);">K</font><font style="color:rgb(25, 27, 31);">T</font><font style="color:rgb(25, 27, 31);">上进行 Softmax，每一行的和都为 1。但是单词 0 在单词 1, 2, 3, 4 上的 attention score 都为 0。</font>

**<font style="color:rgb(25, 27, 31);">第四步：</font>**<font style="color:rgb(25, 27, 31);">使用</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Mask</font>****<font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Q</font><font style="color:rgb(25, 27, 31);">K</font><font style="color:rgb(25, 27, 31);">T</font><font style="color:rgb(25, 27, 31);">与矩阵</font>**<font style="color:rgb(25, 27, 31);"> </font>****<font style="color:rgb(25, 27, 31);">V</font>**<font style="color:rgb(25, 27, 31);">相乘，得到输出</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Z</font>**<font style="color:rgb(25, 27, 31);">，则单词 1 的输出向量</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">Z</font><font style="color:rgb(25, 27, 31);">1</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">是只包含单词 1 信息的。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079879645-40681364-6f27-41ed-add5-20103936d333.png)

<font style="color:rgb(145, 150, 161);">Mask 之后的输出</font>

**<font style="color:rgb(25, 27, 31);">第五步：</font>**<font style="color:rgb(25, 27, 31);">通过上述步骤就可以得到一个 Mask Self-Attention 的输出矩阵</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">Z</font><font style="color:rgb(25, 27, 31);">i</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">，然后和 Encoder 类似，通过 Multi-Head Attention 拼接多个输出</font><font style="color:rgb(25, 27, 31);">Z</font><font style="color:rgb(25, 27, 31);">i</font><font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">然后计算得到第一个 Multi-Head Attention 的输出</font>**<font style="color:rgb(25, 27, 31);">Z</font>**<font style="color:rgb(25, 27, 31);">，</font>**<font style="color:rgb(25, 27, 31);">Z</font>**<font style="color:rgb(25, 27, 31);">与输入</font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);">维度一样。</font>

### <font style="color:rgb(25, 27, 31);">5.2 第二个 Multi-Head Attention</font>
<font style="color:rgb(25, 27, 31);">Decoder block 第二个 Multi-Head Attention 变化不大， 主要的区别在于其中 Self-Attention 的</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">K, V</font>**<font style="color:rgb(25, 27, 31);">矩阵不是使用 上一个 Decoder block 的输出计算的，而是使用</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Encoder 的编码信息矩阵 C</font>****<font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">计算的。</font>

<font style="color:rgb(25, 27, 31);">根据 Encoder 的输出</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">C</font>**<font style="color:rgb(25, 27, 31);">计算得到</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">K, V</font>**<font style="color:rgb(25, 27, 31);">，根据上一个 Decoder block 的输出</font>**<font style="color:rgb(25, 27, 31);"> </font>****<font style="color:rgb(25, 27, 31);">Z</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">计算</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Q</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">(如果是第一个 Decoder block 则使用输入矩阵</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">X</font>**<font style="color:rgb(25, 27, 31);"> </font><font style="color:rgb(25, 27, 31);">进行计算)，后续的计算方法与之前描述的一致。</font>

<font style="color:rgb(25, 27, 31);">这样做的好处是在 Decoder 的时候，每一位单词都可以利用到 Encoder 所有单词的信息 (这些信息无需</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Mask</font>**<font style="color:rgb(25, 27, 31);">)。</font>

### <font style="color:rgb(25, 27, 31);">5.3 Softmax 预测输出单词</font>
<font style="color:rgb(25, 27, 31);">Decoder block 最后的部分是利用 Softmax 预测下一个单词，在之前的网络层我们可以得到一个最终的输出 Z，因为 Mask 的存在，使得单词 0 的输出 Z0 只包含单词 0 的信息，如下：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079879719-9b8fbf64-72af-4dec-a8bf-554edcdc8839.png)

<font style="color:rgb(145, 150, 161);">Decoder Softmax 之前的 Z</font>

<font style="color:rgb(25, 27, 31);">Softmax 根据输出矩阵的每一行预测下一个单词：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737079879626-58fa8ee3-41e1-4476-b94b-81175db1e0ea.png)

<font style="color:rgb(145, 150, 161);">Decoder Softmax 预测</font>

<font style="color:rgb(25, 27, 31);">这就是 Decoder block 的定义，与 Encoder 一样，Decoder 是由多个 Decoder block 组合而成。</font>

## <font style="color:rgb(25, 27, 31);">6. Transformer 总结</font>
+ <font style="color:rgb(25, 27, 31);">Transformer 与 RNN 不同，可以比较好地并行训练。</font>
+ <font style="color:rgb(25, 27, 31);">Transformer 本身是不能利用单词的顺序信息的，因此需要在输入中添加位置 Embedding，否则 Transformer 就是一个词袋模型了。</font>
+ <font style="color:rgb(25, 27, 31);">Transformer 的重点是 Self-Attention 结构，其中用到的</font><font style="color:rgb(25, 27, 31);"> </font>**<font style="color:rgb(25, 27, 31);">Q, K, V</font>**<font style="color:rgb(25, 27, 31);">矩阵通过输出进行线性变换得到。</font>
+ <font style="color:rgb(25, 27, 31);">Transformer 中 Multi-Head Attention 中有多个 Self-Attention，可以捕获单词之间多种维度上的相关系数 attention score。</font>

