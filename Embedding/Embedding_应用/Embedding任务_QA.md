QA是问答的意思，Q表示Question，A表示Answer，QA是NLP非常基础和常用的任务。就是当用户提出一个问题时，能从已有的问题库中找到一个最相似的，并把它的答案返回给用户。

这里有两个关键点：

1. 事先需要有一个QA库。
2. 用户提问时，系统要能够在QA库中找到一个最相似的。


⚠️为什么不能使用 ChatGPT（或生成方式）做这类任务？

相对有点麻烦，尤其是当：

+ QA库非常庞大时
+ 给用户的答案是固定的，不允许自由发挥时

生成方式做起来是事倍功半。

<br/>

但是`Embedding`确实天然的非常适合，因为该任务的核心就是在一堆文本中找出给定文本最相似的。**简单来说，其实就是个相似度计算问题**。

### 读取数据集
使用`Kaggle`提供的`Quora`数据集：[FAQ Kaggle dataset! | Data Science and Machine Learning](https://www.kaggle.com/general/183270)

[Kaggle related questions on Qoura - Questions.csv](https://www.yuque.com/attachments/yuque/0/2025/csv/2639475/1735871399931-97e8a40d-5db4-4975-9562-8b158342c7f6.csv)

```python
import pandas as pd
```

> 插入一个 python 库的用法：[pandas](https://www.yuque.com/qiaokate/bd43p2/yrogkp8hssm7qm70)
>

1. 读取 CSV 文件

```python
df = pd.read_csv("dataset/Kaggle related questions on Qoura - Questions.csv")
df.shape
```

<details class="lake-collapse"><summary id="u3a3be84d"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="mqoKZ" class="ne-codeblock language-json"><code>(1166, 4)</code></pre></details>
2. 读的是前 5 行数据

```python
df.head()
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735626594645-cefb003e-49b7-4c44-b759-e8b7b98accd9.png)

### 构造数据对
<br/>color2
这里，就把`Link`当做答案构造数据对。基本的流程如下：

1. 对每个`Question`计算`Embedding`
2. 存储`Embedding`，同时存储每个`Question`对应的答案
3. 从存储的地方检索最相似的`Question`

<br/>

第一步将借助OpenAI的`Embedding`接口，但是后两步得看实际情况了。

如果`Question`的数量比较少，比如只有几万条甚至几千条，那可以把计算好的`Embedding`直接存储成文件，每次服务启动时直接加载到内存或缓存里就好了。使用时，挨个计算输入问题和存储的所有问题的相似度，然后给出最相似的问题的答案。

为了快速演示，只取前5个句子为例：

```python
import numpy as np
```

```python
vec_base = []
for v in df.head().itertuples():
    emb = get_embedding(v.Questions)
    im = {
        "question": v.Questions,
        "embedding": emb,
        "answer": v.Link
    }
    vec_base.append(im)
```

智谱也一样

```python
vec_base_zhipu = []
for v in df.head().itertuples():
    emb = get_embedding_zhipu(v.Questions)
    im = {
        "question": v.Questions,
        "embedding": emb,
        "answer": v.Link
    }
    vec_base_zhipu.append(im)
```

然后给定输入，比如：`"is kaggle alive?"`，先获取它的`Embedding`，然后逐个遍历`vec_base`计算相似度，并取最高的作为响应。

```python
query = "is kaggle alive?"
q_emb = get_embedding(query)
q_emb_zhpu = get_embedding_zhipu(query)
```

1. chatGPT 的答案

```python
sims = [cosine_similarity(q_emb, v["embedding"]) for v in vec_base]
sims
```

<details class="lake-collapse"><summary id="u2da1ac16"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="Y3gmU" class="ne-codeblock language-json"><code>[0.7879602774711745,
 0.9542302479388203,
 0.8227517970462344,
 0.8398752285928424,
 0.8109667304826295]</code></pre></details>
2. 智谱的答案

```python
sims_zhipu = [cosine_similarity(q_emb_zhpu, v["embedding"]) for v in vec_base_zhipu]
sims_zhipu
```

<details class="lake-collapse"><summary id="u9f1b2ca6"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="iRdDI" class="ne-codeblock language-json"><code>[0.47816811389316444,
 0.8856716112337668,
 0.6596088197402513,
 0.6973234108766252,
 0.5534577913085058]</code></pre></details>
答案理论上是这样：

| <font style="color:#000000;">Questions</font> | <font style="color:#000000;">Followers</font> | <font style="color:#000000;">Answered</font> | <font style="color:#000000;">Link</font> |
| --- | --- | --- | --- |
| <font style="color:#000000;">Is Kaggle dead?</font> | <font style="color:#000000;">181</font> | <font style="color:#000000;">1</font> | <font style="color:#000000;">/Is-Kaggle-dead</font> |


打印下答案：都回答的正确

```python
print(
    vec_base[1]["question"], vec_base[1]["answer"], 
    vec_base_zhipu[1]["question"], vec_base_zhipu[1]["answer"]
)
```

<details class="lake-collapse"><summary id="udd51b976"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="xTHO5" class="ne-codeblock language-json"><code>Is Kaggle dead? /Is-Kaggle-dead Is Kaggle dead? /Is-Kaggle-dead</code></pre></details>
### 使用`NumPy`进行批量计算
不建议使用循环，可以使用`NumPy`进行批量计算（智谱AI一样）。调用Embedding API，查看对应文档看是否支持Batch方法。

```python
arr = np.array(
    [v["embedding"] for v in vec_base]
)
arr.shape
```

<details class="lake-collapse"><summary id="u32cacccb"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="XcS2u" class="ne-codeblock language-json"><code>(5, 1536)</code></pre></details>


```python
q_arr = np.expand_dims(q_emb, 0)
q_arr.shape
```

<details class="lake-collapse"><summary id="u8bd99df2"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="xrDFf" class="ne-codeblock language-json"><code>(1, 1536)</code></pre></details>


```python
from sklearn.metrics.pairwise import cosine_similarity
```

```python
cosine_similarity(arr, q_arr)
```

<details class="lake-collapse"><summary id="ucedaf0e8"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="P8Y4i" class="ne-codeblock language-json"><code>array([[0.78796028],
       [0.95423025],
       [0.8227518 ],
       [0.83987523],
       [0.81096673]])</code></pre></details>
### 借助语义检索的工具
不过，当Question非常多，比如上百万甚至上亿时，这种方式就不合适了。一个是内存里可能放不下，另一个是算起来也很慢。这时候就必须借助一些专门用来做语义检索的工具了。

比较常用的工具有：

#### faiss
[GitHub - facebookresearch/faiss: A library for efficient similarity search and clustering of dense vectors.](https://github.com/facebookresearch/faiss)

#### milvus
[GitHub - milvus-io/milvus: Milvus is a high-performance, cloud-native vector database designed to scale vector search.](https://github.com/milvus-io/milvus)

#### redis
[Vectors](https://redis.io/docs/stack/search/reference/vectors/)

#### Redis
##### 安装
此处，以Redis为例，其他工具用法类似。首先，需要一个redis，建议使用docker直接运行：

```python
docker run -p 6379:6379 -it redis/redis-stack:latest
```

执行后，docker会自动从hub把镜像拉到本地，默认是6379端口。然后安装`redis-py`，也就是Redis的Python客户端：

```python
pip install redis
```

##### 构造数据
这样就可以用Python和Redis进行交互了。先来个最简单的例子：

```python
import redis
r = redis.Redis()
r.set("key", "value")
```

<details class="lake-collapse"><summary id="udd891b86"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="VqqEG" class="ne-codeblock language-json"><code>True</code></pre></details>


```python
r.get("key")
```

<details class="lake-collapse"><summary id="ua5683200"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="FPjJ5" class="ne-codeblock language-json"><code>b'value'</code></pre></details>
##### 建立索引
如果使用过`ElasticSearch`，接下来的内容会非常容易理解。总的来说，和刚刚的步骤差不多，但是需要先建索引，然后生成Embedding并把它存储到Redis，再进行使用（从索引中搜索）。

索引的概念和数据库中的索引有点相似，就是要定义一组Schema，告诉Redis字段是什么，有哪些属性。

```python
VECTOR_DIM = 1536 # 智谱AI需改为1024，因为智谱AI的Embedding维度是1024
INDEX_NAME = "faq"
```

```python
from redis.commands.search.query import Query
from redis.commands.search.field import TextField, VectorField
from redis.commands.search.indexDefinition import IndexDefinition
```

```python
# 建好要存字段的索引，针对不同属性字段，使用不同Field
question = TextField(name="question")
answer = TextField(name="answer")
embedding = VectorField(
    name="embedding", 
    # Hierarchical Navigable Small Worlds，HNSW，一种高效的相似搜索算法。
    algorithm="HNSW", 
    # 属性
    attributes={
        "TYPE": "FLOAT32",
        "DIM": VECTOR_DIM,
        "DISTANCE_METRIC": "COSINE"
    }
)
schema = (question, embedding, answer)
index = r.ft(INDEX_NAME)
try:
    info = index.info()
except:
    index.create_index(schema, definition=IndexDefinition(prefix=[INDEX_NAME + "-"]))
```

> `Hierarchical Navigable Small Worlds，HNSW`，一种高效的相似搜索算法。
>

```python
# 如果需要删除已有文档的话，可以使用下面的命令
# index.dropindex(delete_documents=True)
```

<details class="lake-collapse"><summary id="uebce7413"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="DD0X6" class="ne-codeblock language-json"><code>b'OK'</code></pre></details>
##### 把数据存到Redis
接下来就是把数据存到Redis

```python
for v in df.head().itertuples():
    # 智谱AI
    # emb = get_embedding_zhipu(v.Questions)
    emb = get_embedding(v.Questions)
    
    # 注意，redis要存储bytes或string
    emb = np.array(emb, dtype=np.float32).tobytes()
    
    im = {
        "question": v.Questions,
        "embedding": emb,
        "answer": v.Link
    }
    
    # 重点是这句
    r.hset(name=f"{INDEX_NAME}-{v.Index}", mapping=im)
```

##### 搜索查询
然后就可以进行搜索查询了，这一步构造查询输入稍微有一点麻烦。

```python
# 构造查询输入
query = "kaggle alive?"
embed_query = get_embedding(query)

# 智谱AI
# embed_query = get_embedding_zhipu(query)
params_dict = {"query_embedding": np.array(embed_query).astype(dtype=np.float32).tobytes()}
```

```python
k = 3
# {some filter query}=>[ KNN {num|$num} @vector_field $query_vec]
# KNN（K最近邻算法），简单来说就是对未知点，分别和已有的点算距离，挑距离最近的K个点。
base_query = f"* => [KNN {k} @embedding $query_embedding AS score]"
return_fields = ["question", "answer", "score"]
query = (
    Query(base_query)
     .return_fields(*return_fields)
     .sort_by("score")
     .paging(0, k)
     .dialect(2)
)
```

<br/>color2
KNN（K最近邻算法），简单来说就是对未知点，分别和已有的点算距离，挑距离最近的K个点。

<br/>

```python
# 查询
res = index.search(query, params_dict)
for i,doc in enumerate(res.docs):
    similarity = 1 - float(doc.score)
    print(f"{doc.id}, {doc.question}, {doc.answer} (Similarity: {round(similarity ,3) })")
```

<details class="lake-collapse"><summary id="uac18e55e"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="CPvp6" class="ne-codeblock language-json"><code>faq-1, Is Kaggle dead?, /Is-Kaggle-dead (Similarity: 0.929)
faq-3, What are some alternatives to Kaggle?, /What-are-some-alternatives-to-Kaggle (Similarity: 0.844)
faq-2, How should a beginner get started on Kaggle?, /How-should-a-beginner-get-started-on-Kaggle (Similarity: 0.833)</code></pre></details>
### 小结
上面，通过几种不同的方法为介绍了如何使用Embedding进行QA任务。

简单回顾一下，要做QA任务首先得有一个QA库，每当一个新的问题过来时，就用这个问题去和仓库里的每一个Q去匹配，然后找到最相似的那个，接下来就把该问题的Answer当做新问题的Answer交给用户。

这个任务的核心就是如何找到这个最相似的，涉及两个知识点：


⚠️如何表示一个Question

用API提供的Embedding表示，可以把它当做一个黑盒子，输入任意长度的文本，输出一个向量。

<br/>


⚠️如何查找到相似的Question

查找相似问题则主要是用到相似度算法，语义相似度一般用`cosine`距离来衡量。

当然实际中可能会更加复杂一些，比如可能除了使用语义匹配，还会使用字词匹配（经典的做法）。而且，一般都会找到topN个相似的，然后对这topN个结果进行排序，选出最可能的那个。

<br/>

不过，前面举过例子了，也完全可以通过ChatGPT来解决，让它选出最好的那个。



