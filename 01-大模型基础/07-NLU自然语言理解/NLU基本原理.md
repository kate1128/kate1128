> 介绍一下 NLU常见问题的基本原理，其他可以阅读NLP算法相关的书籍，从一个可直接上手的小项目开始一步一步构建自己的知识体系
>

#  什么是  NLU
[什么是 NLU](https://www.yuque.com/qiaokate/su87gb/kpc3y0k5ehfslcoy)

一般意义上的`NLU`常指与**理解给定句子意思**相关的**意图识别、实体抽取、指代关系**等任务，在智能对话中应用比较广泛。具体点来说，当用户输入一句话时，机器人一般会针对该句话（也可以把历史记录给附加上）进行全方面分析，包括：

1. **情感倾向分析**：简单来说，一般会包括**正向、中性、负向**三种类型，也可以设计更多的类别。或更复杂的细粒度情感分析，比如针对其中某个实体或属性的情感，而不是整个句子的。
2. **意图识别**：一般都是分类模型，**大部分时候都是多分类，但是也有可能是层次分类，或多标签模型**。
    1. **多分类**：给定输入文本，输出为一个Label，但Label的总数有多个。比如类型包括询问地址、询问时间、询问价格、闲聊等等。
    2. **层次分类**：给定输入文本，输出为层次的Label，也就是从根节点到最终细粒度类别的路径。比如询问地址/询问家庭地址、询问地址/询问公司地址等等。
    3. **多标签分类**：给定输入文本，输出不定数量的Label，也就是说每个文本可能有多个Label，Label之间是平级关系。
3. **实体和关系抽取**：
    1. **实体抽取**：提取出给定文本中的实体。实体一般指具有特定意义的实词，如人名、地名、作品、品牌等等；很多时候也是业务直接相关的词。
    2. **关系抽取**：实体之间往往有一定的关系，比如「刘亦菲」出演「天龙八部」，其中「刘亦菲」就是人名、「天龙八部」是作品名，其中的关系就是「出演」，一般会和实体作为三元组来表示。

<br/>color2
一般经过以上这些分析后，机器人就可以对用户的输入有一个比较清晰的理解，便于接下来据此做出响应。另外值得一提的是，上面的过程并不一定只用在对话中，只要涉及到用户输入`Query`需要给出响应的场景，都需要这个`NLU`的过程，一般也叫`**Query**`**解析**。

<br/>

# 算法的角度分类
一般意义上的`NLU`常指与**理解给定句子意思**相关的**意图识别、实体抽取、指代关系**等任务，在智能对话中应用比较广泛。如果从算法的角度看，其实就两种：

<br/>color2
1. **句子级别的分类**

如情感分析、意图识别、关系抽取等。也就是给一个句子，给出一个或多个`Label`。

2. **Token级别的分类**

如实体抽取、阅读理解（就是给一段文本和一个问题，然后在文本中找到问题的答案）。也就是给一个句子，给出对应实体的位置。

<br/>

## 举例说明
Token级别的分类不太好理解，我们举个例子，比如下面这句话：

```plain
刘亦菲出演天龙八部。
```

它在标注的时候是这样的：

```plain
刘/B-PER
亦/I-PER
菲/I-PER
出/O
演/O
天/B-WORK
龙/I-WORK
八/I-WORK
部/I-WORK
。/O
```

在上面这个例子中，每个Token就是每个字，每个Token会对应一个Label（当然也可以多个）

`Label`中的

+ `B`表示`Begin`
+ `I`表示`Internal`
+ `O`表示`Other`（也就是非实体）

模型要做的就是**学习这种对应关系**，当给出新的文本时，能够给出每个 Token 的 Label 预测。也就是说，它们本质上都是分类任务，只是分类的位置或标准不一样。当然了，实际应用中会有各种不同的变化和设计，但整个思路是差不多的，我们并不需要掌握这些知识，只需要知道大概是怎么一回事就好了。

## 句子级别的分类
接下来，我们简单介绍一下这些分类具体是怎么做的，先说句子级别的分类。回忆上一章的`Embedding`，那可以算是整个`DeepLearning NLP`的基石，我们这部分的内容也会用到`Embedding`。

<br/>color2
具体过程如下：

1. 将**给定句子或文本表征成**`**Embedding**`
2. **将**`**Embedding**`**传入一个神经网络**，计算得到不同`Label`的概率分布
3. 将上一步的`Label`**概率分布与真实的分布做比较**，并将误差回传，修改神经网络的参数，即训练
4. 得到训练好的神经网络，即模型

<br/>

举个例子，简单起见，假设`Embedding`维度为 32 维（上一章 OpenAI 返回的维度比较大）：

```python
import numpy as np
np.random.seed(0)
```

随机生成一个均值为0、标准差为1的1×32维的高斯分布作为Embedding数组，维度1表示词表里只有一个Token

```python

```

如果是三分类，那么最简单的`W`就是`32×3`的大小（这个`W`就被称为模型/参数）：

```python
W = np.random.random((32, 3))
```

```python
z = emd @ W
z
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>array([[6.93930177, 5.96232449, 3.96168115]])</code></pre></details>
上面得到的z一般被称为logits，如果想要概率分布，则需要对其归一化，也就是将logits变成0-1之间的概率值，并且每一行加起来为1（即100%），如下所示。

```python
def norm(z):
    exp = np.exp(z)
    return exp / np.sum(exp)
```

```python
y = norm(z)
y
```

<details class="lake-collapse"><summary id="u01d20316"><span class="ne-text">output：</span></summary><pre data-language="json" id="r5Vi8" class="ne-codeblock language-json"><code>array([[0.70059356, 0.26373654, 0.0356699 ]])</code></pre></details>
根据给出的y，就知道预测的标签是第0个位置的标签，因为那个位置的概率最大（70.06%）。如果真实的标签是第1个位置的标签，那第1个位置的标签实际就是1（100%），但它目前预测的概率只有26.37%，这个误差就会回传来调整W参数。下次计算时，第0个位置的概率就会变小，第1个位置的概率则会变大。这样通过标注的数据样本不断循环迭代的过程其实就是模型训练的过程，也就是通过标注数据，让模型W尽可能正确预测出标签。

```python
np.sum(y)
```

<details class="lake-collapse"><summary id="u1ad4e5d9"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="IUQL1" class="ne-codeblock language-json"><code>1.0</code></pre></details>
实际中，W往往更加复杂，可以包含任意的数组，只要最后输出变成1×3的大小即可，比如下面这个复杂点的：

可以看到，现在有三个数组的模型参数，形式上虽然复杂了些，但结果是一样的，依然是一个1×3大小的数组。接下来的过程就和前面一样了。

```python
w1 = np.random.random((32, 100))
w2 = np.random.random((100, 32))
w3 = np.random.random((32, 3))
```

```python
y = norm(norm(norm(emd @ w1) @ w2) @ w3)
y
```

<details class="lake-collapse"><summary id="u7029818a"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="x4wAc" class="ne-codeblock language-json"><code>array([[0.35549151, 0.31975607, 0.32475242]])</code></pre></details>
稍微复杂点的是多标签分类和层次分类，这俩因为输出的都是多个标签，处理起来要麻烦一些，不过它们的处理方式是类似的。我们以多标签分类来说明，假设有10个标签，给定输入文本，可能是其中任意多个标签。这就意味着我们需要将10个标签的概率分布都表示出来。可以针对每个标签做个二分类，也就是说输出的大小是10×2的，每一行表示「是否是该标签」的概率分布。

```python
def norm(z):
    axis = -1
    exp = np.exp(z)
    return exp / np.expand_dims(np.sum(exp, axis=axis), axis)
```

```python
np.random.seed(42)

emd = np.random.normal(0, 1, (1, 32))
W = np.random.random((10, 32, 2))
y = norm(emd @ W)
y.shape
```

<details class="lake-collapse"><summary id="u5bff6d01"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="g6t8U" class="ne-codeblock language-json"><code>(10, 1, 2)</code></pre></details>
这里的输出每一行有两个值，分别表示标签「`是/0`」和「`是/1`」的概率，比如第一行，`0`的概率为`0.66`，`1`的概率为`0.34`。需要注意的是归一化时，要指定维度，否则就变成所有的值加起来为`1`了，这就不对了。

```python
y
```

<details class="lake-collapse"><summary id="u2ed70172"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="MwR9C" class="ne-codeblock language-json"><code>array([[[0.66293305, 0.33706695]],

       [[0.76852603, 0.23147397]],

       [[0.59404023, 0.40595977]],

       [[0.04682992, 0.95317008]],

       [[0.84782999, 0.15217001]],

       [[0.01194495, 0.98805505]],

       [[0.96779413, 0.03220587]],

       [[0.04782398, 0.95217602]],

       [[0.41894957, 0.58105043]],

       [[0.43668264, 0.56331736]]])</code></pre></details>
上面是句子级别分类（`Sequence Classification`）的逻辑，实际比上面要复杂得多，但基本思路是这样的。在LLM时代也并不需要自己去构建模型了，可以使用LLM的API进行各类任务。

## Token级别的分类
接下来看Token级别的分类，有了刚刚的基础，这个看起来就比较容易了。它最大的特点是，`Embedding`是针对每个Token的。也就是说，如果给定文本长度为`10`，假定维度依然是`32`，那`Embedding`的大小就为：`(1, 10, 32)`。比刚刚的`(1, 32)`多了个10。

换句话说，这个文本的每一个`Token`都是一个`32`维的向量。再通俗一点来说，对于模型来说，无论你是1个Token、2个Token还是100个Token，都可以统一看待--**都是固定维度的一个向量表示**。

下面假设Label共5个：`B-PER，I-PER，B-WORK，I-WORK，O`。

```python
emd = np.random.normal(0, 1, (1, 10, 32))
```

```python
W = np.random.random((32, 5))
```

```python
z = emd @ W
y = norm(z)
```

```python
y.shape
```

<details class="lake-collapse"><summary id="u11f908e6"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="rFydT" class="ne-codeblock language-json"><code>(1, 10, 5)</code></pre></details>
注意看，每一行表示一个Token是某个标签的概率分布（每一行加起来为1），比如第一行。

```python
y
```

<details class="lake-collapse"><summary id="ub88e02ed"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="Uonp0" class="ne-codeblock language-json"><code>array([[[0.23850186, 0.04651826, 0.12495322, 0.28764271, 0.30238396],
        [0.06401011, 0.3422055 , 0.54911626, 0.01179874, 0.03286939],
        [0.18309536, 0.62132479, 0.09037235, 0.06016401, 0.04504349],
        [0.01570559, 0.0271437 , 0.20159052, 0.12386611, 0.63169408],
        [0.1308541 , 0.06810165, 0.61293236, 0.00692553, 0.18118637],
        [0.08011671, 0.04648297, 0.00200392, 0.02913598, 0.84226041],
        [0.05143706, 0.09635837, 0.00115594, 0.83118412, 0.01986451],
        [0.03721064, 0.14529403, 0.03049475, 0.76177941, 0.02522117],
        [0.24154874, 0.28648044, 0.11024747, 0.35380566, 0.0079177 ],
        [0.10965428, 0.00432547, 0.08823724, 0.00407713, 0.79370588]]])</code></pre></details>
具体的意思是，第一个Token是第0个位置标签（假设标签按上面给出的顺序，那就是`B-PERSON`）的概率是23.85%，其他类似。

根据这里预测的结果，第一个Token的标签是`O`，那真实的标签和这个预测的标签之间就可能有误差，通过误差就可以更新参数，从而使得之后预测时能预测到正确的标签（也就是正确位置的概率最大）。不难看出，这个逻辑和前面的句子分类是类似的，其实就是对每一个Token做了个多分类。

```python
sum((0.23850186, 0.04651826, 0.12495322, 0.28764271, 0.30238396))
```

<details class="lake-collapse"><summary id="u3650896c"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="dEKK6" class="ne-codeblock language-json"><code>1.00000001</code></pre></details>
