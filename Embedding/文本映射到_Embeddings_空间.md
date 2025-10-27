> 就是调用 cohere 的 API 把文本映射到 Embeddings 空间中。需要用到 Cohere 的 API key。
>
> 使用 Cohere 库中的 embed 函数，将所有这些句子转化为 Embeddings
>

[什么是 词向量](https://www.yuque.com/qiaokate/su87gb/dmazvwmio3bw59be)

Embeddings 可以理解为一种计算机更容易处理的文本的向量表示，通常用于将离散的、高维的数据表示（如单词、句子或文档）转换为连续的、低维的向量表示。

Embeddings 的目的是捕捉数据之间的语义和语法关系，使得相似的数据在嵌入空间中更接近。例如，在自然语言处理中，可以使用 Word Embeddings （词嵌入）将单词映射到连续的向量空间，使得具有相似含义的单词在嵌入空间中距离更近。这种特性也使得它们成为 LLM（大语言模型）中最重要的组成部分之一。

##  环境配置
第三方包

```python
!pip install cohere umap-learn altair datasets
```

```python
# 下面的代码可以帮助加载需要用到的 API
import os
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # 读取本地 .env 文件
```

导入 `Cohere` 库，并且使用 API 密钥创建一个 `Cohere` 客户端。`**Cohere**`** 库是一个包含调用大语言模型的函数的库，可以通过 API 调用来调用这些函数。**

```python
import cohere
co = cohere.Client(os.environ['COHERE_API_KEY'])
```

还需要导入 `Pandas` 库，它可以用于数据分析与数据处理。

```python
import pandas as pd
```

##  词嵌入（Word Embeddings）
### 理解嵌入的概念
在这里，有一个带有横轴、纵轴以及坐标值的网格，可以看到一堆单词位于这个网格中。 如果要把单词放到合适的位置，你会把单词"apple"（译为“苹果”）放在哪里？

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/3-1.png)

正如你在这个网格中所看到的，相似的单词被分组在一起。 所以在左上方有足球、篮球、乒乓球，左下方有房屋、建筑和城堡，右下方有自行车和汽车等交通工具，右上方有水果。 因此，apple 将被归类为右上方的水果。 然后，将表中的每个单词与坐标轴关联起来。这里 apple 的坐标是（5，5）。

这就是一种 Embeddings ，它将每个单词映射为两个数值组成的向量。

一般来说，Embeddings 会将单词映射到更多的数值。会有尽可能多的单词，为了将每个单词都能表示，实际使用的 Embeddings 可以将一个单词映射到数百个数值，甚至数千个。

### 实现词嵌入
将使用一个非常小的数据表。它包含三个单词：joy（欢乐）、happiness（快乐） 和 potato（马铃薯） ，用 Pandas 将其创建，如下所示：

```python
three_words = pd.DataFrame({'text':
                            [
                                'joy',
                                'happiness',
                                'potato'
                            ]})

three_words
```

|  | **text** |
| :--- | :--- |
| **0** | joy |
| **1** | happiness |
| **2** | potato |


```python
# 中文版本
three_words = pd.DataFrame({'text':
                            [
                                '欢乐',
                                '快乐',
                                '马铃薯'
                            ]})

three_words
```

|  | **text** |
| :--- | :--- |
| **0** | 欢乐 |
| **1** | 快乐 |
| **2** | 马铃薯 |


接下来，为这三个单词创建 Embeddings。 使用 Cohere 库中的 embed 函数来创建这些 Embeddings。 embed 函数接受一些输入。第一个输入是要嵌入的数据集"three_words"，还需要指定所使用的列为"text"，以及要使用的模型。

```python
three_words_emb = co.embed(texts=list(three_words['text']),
                           model='embed-english-v2.0').embeddings  # 英文版本用英文嵌入模型 embed-english-v2.0
```

```python
# 中文版本
three_words_emb = co.embed(texts=list(three_words['text']),
                           model='embed-multilingual-v2.0').embeddings  # 中文版本用多语言嵌入模型 embed-multilingual-v2.0
```

现在让来看一下与每个单词相关联的 Embeddings。将与单词"joy"相关联的 Embeddings 称为"word_1"，可以通过查看"three_words_emb"的第一行来获取。对"word_2"和"word_3"也做同样的操作。它们是与单词"happiness"和"potato"对应的 Embeddings 。

可以打印单词"joy"相关联的 Embeddings 的前10个数值看看，即"word_1"中的前十个数值。

```python
word_1 = three_words_emb[0]
word_2 = three_words_emb[1]
word_3 = three_words_emb[2]

print(word_1[:10])
# 注：下面输出的结果是英文版本的结果，如果是中文版本，会有所不同。
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text" style="color: var(--jp-content-font-color1)">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>[2.3203125, -0.18334961, -0.578125, -0.7314453, -2.2050781, -2.59375, 0.35205078, -1.6220703, 0.27954102, 0.3083496]</code></pre></details>
##  句嵌入（Sentence Embeddings）
###  理解句嵌入的概念
Embeddings 不仅可以用于单词，还可以用于更长的文本片段。实际上，它可以是非常长的文本片段。

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/3-2.png)

在这个例子中，有一些句子的 Embeddings。 现在这些句子被转化为一个向量或数值列表。 请注意，第一个句子是"hello, how are you?"，最后一个句子是"Hi, how's it going?"。 它们没有相同的单词，但它们的句意非常相似，所以 Embeddings 会将它们映射到一些非常接近的数值。

###  实现句嵌入
准备一个包含多个句子的小型数据集。如你所见，这个数据集有八个句子，它们是成对出现的。每个句子都是前一个句子的答案，例如，"What color is the sky?"的答案是"The sky is blue"，"What is an apple?"的答案是"An apple is a fruit"。

```python
# 创建一个包含八个句子的 DataFrame
sentences = pd.DataFrame({'text':
                          [
                              'Where is the world cup?',  # 句子1: 世界杯在哪里？
                              'The world cup is in Qatar',  # 句子2: 世界杯在卡塔尔。
                              'What color is the sky?',  # 句子3: 天空是什么颜色的？
                              'The sky is blue',  # 句子4: 天空是蓝色的。
                              'Where does the bear live?',  # 句子5: 熊住在哪里？
                              'The bear lives in the woods',  # 句子6: 熊住在森林里。
                              'What is an apple?',  # 句子7: 苹果是什么？
                              'An apple is a fruit',  # 句子8: 苹果是一种水果。
                          ]})
```

```python
# 中文版本
sentences = pd.DataFrame({'text':
                          [
                              '世界杯在哪里？',
                              '世界杯在卡塔尔', 
                              '天空是什么颜色的?', 
                              '天空是蓝色的', 
                              '熊住在哪里？',  
                              '熊住在森林里', 
                              '苹果是什么?',  
                              '苹果是一种水果',  
                          ]})
```

现在，仍然使用 Cohere 库中的 embed 函数，将所有这些句子转化为 Embeddings ，并观察哪些句子彼此接近或远离。

```python
emb = co.embed(texts=list(sentences['text']),
               model='embed-english-v2.0').embeddings  # 英文版本用英文嵌入模型 embed-english-v2.0
```

```python
# 中文版本
emb = co.embed(texts=list(sentences['text']),
               model='embed-multilingual-v2.0').embeddings  # 中文版本用多语言嵌入模型 embed-multilingual-v2.0
```

来看一下每个句子的嵌入的前 3 个数值。

```python
for e in emb:
    print(e[:3])
# 注：下面输出的结果是英文版本的结果，如果是中文版本，会有所不同。
```

<details class="lake-collapse"><summary id="u1b98bb96"><span class="ne-text">output：</span></summary><pre data-language="json" id="hq82Q" class="ne-codeblock language-json"><code>[0.27319336, -0.37768555, -1.0273438]
[0.49804688, 1.2236328, 0.4074707]
[-0.23571777, -0.9375, 0.9614258]
[0.08300781, -0.32080078, 0.9272461]
[0.49780273, -0.35058594, -1.6171875]
[1.2294922, -1.3779297, -1.8378906]
[0.15686035, -0.92041016, 1.5996094]
[1.0761719, -0.7211914, 0.9296875]</code></pre></details>
再看看每个句子的 Embeddings 有多少个数值

```python
print(len(emb[0]))
```

在这个特定的例子中，答案是 4096 个，但不同的 Embeddings 长度也会不同。

让来可视化一下这个数据集的 Embeddings。 调用 utils 库里的名为 umap_plot 函数，它会调用 umap 和 altair 包，并生成下面的图。

```python
from utils import umap_plot
# 使用 umap_plot 函数生成图表
chart = umap_plot(sentences, emb)
# 调用 interactive 方法以显示交互式图表
chart.interactive()
```

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/3-3.png)

这个图给出了八个点，每个点代表的数据集中的一个句子，将鼠标放到点上，会显示出这个点代表哪个句子。

可以观察到，句意相似的两个句子之间挨得非常近，比如'Where does the bear live?'和'The bear lives in the the woods'。

所以可以得到一个结论，Embeddings 会将句意相似的点放在靠近的位置上，句意相差大的点放在相距较远的位置上。 通常情况下，与一个问题句意最相似的就是它特定的答案。 因此，可以通过搜索与问题位置最接近的句子来找到问题的答案。

##  文档嵌入（Articles Embeddings）
现在你已经知道如何对包含八个句子的小数据集进行 Embeddings 了，接下来让来处理一个大数据集。

将使用一个包含维基百科文章的大数据集。 它有 2000 篇带有标题的文章、第一段文字的文本以及第一段文字的 Embeddings。 让用下面的代码加载以下数据集。

```python
import pandas as pd
wiki_articles = pd.read_pickle('wikipedia.pkl')
wiki_articles[['title','text','emb']]
```

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/3-4.png)

将导入 Numpy 库和一个帮助可视化这个图的函数，这个图与之前的图非常相似。 将其降维到二维，以便观察。

```python
import numpy as np
from utils import umap_plot_big

# 从 wiki_articles 中获取 'title' 和 'text' 列的数据
articles = wiki_articles[['title', 'text']]

# 从 wiki_articles 中获取 'emb' 列的数据，并转换为 numpy 数组
embeds = np.array([d for d in wiki_articles['emb']])

# 使用 umap_plot_big 函数生成图表
chart = umap_plot_big(articles, embeds)

# 调用 interactive 方法以显示交互式图表
chart.interactive()
```

将鼠标放到图中的点上，可以显示文章的内容。可以观察到到相似的文章位于相似的位置。

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/3-5.png)

这就是关于 Embeddings 的内容。

