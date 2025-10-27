> 本节就是举个例子算下`**cosine**`**值，**看下语义相似度，相同语义的句子相似度更高
>
> text1 = "我喜欢你"
>
> text2 = "我钟意你"
>
> text3 = "我不喜欢你"
>
> 理论上是`"我喜欢你"` VS `"我钟意你"` 相似度会高一些
>

## 语义相似度
与`Embedding`息息相关的一个概念是「相似度」，准确来说是「语义相似度」。在自然语言处理领域，一般使用`**cosine**`**相似度**作为语义相似度的度量，评估两个向量在语义空间上的分布情况。

具体来说就是下面这个式子：

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735625732430-661f346f-1eb2-435f-bb91-3ec026fce3b2.png)

```python
import numpy as np
a = [0.1, 0.2, 0.3]
b = [0.2, 0.3, 0.4]
cosine_ab = (0.1*0.2+0.2*0.3+0.3*0.4)/(np.sqrt(0.1**2+0.2**2+0.3**2) * np.sqrt(0.2**2+0.3**2+0.4**2))
cosine_ab
```

`**cosine**`**相似度输出：**

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>0.9925833339709301</code></pre></details>
举个例子：

```python
text1 = "我喜欢你"
text2 = "我钟意你"
text3 = "我不喜欢你"
```

对这三句话的语义来看下它的`**cosine**`**相似度**

### `text-embedding-3-large`& `zhipu embedding-2`
OpenAI官方提供了一个集成接口，使用起来更加简单（也可以自己写一个）：

1. 计算两个向量的余弦相似度

```python
# 计算两个向量的余弦相似度
def cosine_similarity(vec1, vec2):
    return np.dot(vec1, vec2) / (np.linalg.norm(vec1) * np.linalg.norm(vec2))
```

2. `get_embedding(text,model = "text-embedding-ada-002")`

```python
def get_embedding(text,model = "text-embedding-ada-002"):
    emb_req = client.embeddings.create(model=model,input = text)
    return emb_req.data[0].embedding
```

3. `get_embedding_zhipu(text)`

```python
from zhipuai import ZhipuAI
api_key = os.environ['ZHIPUAI_API_KEY']
zhipu_client = ZhipuAI(api_key=api_key)
```

```python
def get_embedding_zhipu(text):
    emb_req = zhipu_client.embeddings.create(
        model="embedding-2",
        input=text,
    )
    return emb_req.data[0].embedding
```

4. 调用 `get_embedding` 

```python
# 注意它支持多种模型，可以通过接口查看
text1 = "我喜欢你"
text2 = "我钟意你"
text3 = "我不喜欢你"
emb1 = get_embedding(text1, "text-embedding-3-large")
emb2 = get_embedding(text2, "text-embedding-3-large")
emb3 = get_embedding(text3, "text-embedding-3-large")
```

```python
emb1_zhipu = get_embedding_zhipu(text1)
emb2_zhipu = get_embedding_zhipu(text2)
emb3_zhipu = get_embedding_zhipu(text3)
```

5. 输出 `embedding`的 `len、type`

```python
len(emb1), type(emb1), len(emb1_zhipu), type(emb1_zhipu)
```

<details class="lake-collapse"><summary id="ubf60ac74"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="cGMU0" class="ne-codeblock language-json"><code>(3072, list, 1024, list)</code></pre></details>
6. 输出 `"我喜欢你"` VS `"我钟意你"` 相似度

```python
cosine_similarity(emb1, emb2), cosine_similarity(emb1_zhipu, emb2_zhipu)
```

<details class="lake-collapse"><summary id="u6620545c"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="ifI3q" class="ne-codeblock language-json"><code>(0.6394042188605163, 0.7285893279997394)</code></pre></details>
7. 输出 `"我喜欢你"` VS `"我不喜欢你"`相似度

```python
cosine_similarity(emb1, emb3), cosine_similarity(emb1_zhipu, emb3_zhipu)
```

<details class="lake-collapse"><summary id="u8762df7d"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="mAFM5" class="ne-codeblock language-json"><code>(0.6301946964388336, 0.8184660238355184)</code></pre></details>
8. 输出 `"我钟意你"` VS `"我不喜欢你"`相似度

```python
cosine_similarity(emb2, emb3), cosine_similarity(emb2_zhipu, emb3_zhipu)
```

<details class="lake-collapse"><summary id="u096d7c01"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="sXguX" class="ne-codeblock language-json"><code>(0.4754155774838153, 0.627373690062137)</code></pre></details>
> 理论上是`"我喜欢你"` VS `"我钟意你"` 相似度会高一些
>
> 看起来智谱模型不是很靠谱，不如 gpt
>

### `text-embedding-ada-002`
换个 Embedding 模型算一下：

```python
text1 = "我喜欢你"
text2 = "我钟意你"
text3 = "我不喜欢你"
emb1 = get_embedding(text1, "text-embedding-ada-002")
emb2 = get_embedding(text2, "text-embedding-ada-002")
emb3 = get_embedding(text3, "text-embedding-ada-002")
```

1. 输出 `"我喜欢你"` VS `"我钟意你"` 相似度

```python
cosine_similarity(emb1, emb2)
```

<details class="lake-collapse"><summary id="u4268720c"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="XBH1U" class="ne-codeblock language-json"><code>0.893848389131725</code></pre></details>
2. 输出 `"我喜欢你"` VS `"我不喜欢你"`相似度

```python
cosine_similarity(emb1, emb3)
```

<details class="lake-collapse"><summary id="u94fc818a"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="LCzLM" class="ne-codeblock language-json"><code>0.9260871172497087</code></pre></details>
3. 输出 `"我钟意你"` VS `"我不喜欢你"`相似度

```python
cosine_similarity(emb2, emb3)
```

<details class="lake-collapse"><summary id="ubc160f11"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="Sxf5j" class="ne-codeblock language-json"><code>0.8456616389291356</code></pre></details>
> 看起来这个模型也不行，不靠谱
>

更多模型可以在这里查看：[New and improved embedding model](https://openai.com/blog/new-and-improved-embedding-model)

OpenAI 模型对比可以参考：

[使用OpenAI API](https://www.yuque.com/qiaokate/su87gb/zpnudvgbftyoge00)

## 直接问模型
### 直接告诉答案
1. 用ChatGPT尝试一下，它不会返回Embedding，它是尝试直接告诉答案！

```python
content = "请告诉我下面三句话的相似程度：\n1. 我喜欢你。\n2. 我钟意你。\n3.我不喜欢你。\n"
```

```python
response = client.chat.completions.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": content}]
)

print(response.choices[0].message.content)
```

<details class="lake-collapse"><summary id="u1770ef67"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="TJFi9" class="ne-codeblock language-json"><code>这三句话的相似程度是很高的，因为它们都是在表达对他人情感的态度。第一句和第二句表示对对方的喜欢和钟意，而第三句则表示对对方的不喜欢。虽然情感不同，但都是在表达对他人的感情态度。</code></pre></details>
2. 智谱AI结果如下：

```python
response = zhipu_client.chat.completions.create(
    model="glm-3-turbo",
    messages=[
        {"role": "user", "content": content},
    ],
)
print(response.choices[0].message.content)
```

<details class="lake-collapse"><summary id="uc959c9a6"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="ifvqn" class="ne-codeblock language-json"><code>这三句话表达的情感是截然不同的：

1. &quot;我喜欢你&quot; 明确表达了积极的情感倾向，表示对某人有好感或爱慕之情。
2. &quot;我钟意你&quot; 是中文里较为口语化和老式的表达方式，也传达了喜欢的意思，和第一句的表达基本相同。
3. &quot;我不喜欢你&quot; 则表达了否定的情感，说明对某人没有好感或者不爱慕。

从句子的情感色彩来看，第一句和第二句是相似的，都传达了喜欢的正面情感，而与第三句的否定情感形成对比。在实际交流中，这三句话所表达的情感强度和意图也可能会因说话人的语气、场合和语境等因素而有所不同。</code></pre></details>
### 问相似程度
调整一下格式：

```python
content = """请告诉我下面三句话的相似程度：
1. 我喜欢你。
2. 我钟意你。
3. 我不喜欢你。
第一句话用a表示，第二句话用b表示，第三句话用c表示。
请以json格式输出两两语义相似度。仅输出json，不要输出其他任何内容。
"""
```

1. OpenAI

```python
response = client.chat.completions.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": content}],
)

print(response.choices[0].message.content)
```

<details class="lake-collapse"><summary id="ue945a18f"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="szBLb" class="ne-codeblock language-json"><code>{
  &quot;a-b&quot;: 0.9,
  &quot;a-c&quot;: 0.2,
  &quot;b-c&quot;: 0.3
}</code></pre></details>
2. 智谱AI结果如下：

```python
response = zhipu_client.chat.completions.create(
    model="glm-3-turbo",
    messages=[
        {"role": "user", "content": content},
    ],
)
print(response.choices[0].message.content)
```

<details class="lake-collapse"><summary id="u474e8450"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="cq7xM" class="ne-codeblock language-json"><code>```json
{
  &quot;a-b&quot;: 0.9,
  &quot;a-c&quot;: -1,
  &quot;b-c&quot;: -1
}
```</code></pre></details>
> 理论上是`"我喜欢你"` VS `"我钟意你"` 相似度会高一些
>

## 参考
+ 【1】[浅析文本分类 —— 情感分析与自然语言处理 | Yam](https://yam.gift/2021/10/27/NLP/2021-10-27-Senta/)
+ 【2】[句子表征综述 | Yam](https://yam.gift/2022/03/27/NLP/2022-03-27-Sentence-Representation-Summarization/)
+ 【3】[NLP 表征的历史与未来 | Yam](https://yam.gift/2020/12/12/NLP/2020-12-12-NLP-Representation-History-Future/)

memo：

+ [openai-cookbook/Semantic_text_search_using_embeddings.ipynb at main · openai/openai-cookbook](https://github.com/openai/openai-cookbook/blob/main/examples/Semantic_text_search_using_embeddings.ipynb)
+ [openai-cookbook/getting-started-with-redis-and-openai.ipynb at main · openai/openai-cookbook](https://github.com/openai/openai-cookbook/blob/main/examples/vector_databases/redis/getting-started-with-redis-and-openai.ipynb)
+ [openai-cookbook/Visualizing_embeddings_in_3D.ipynb at main · openai/openai-cookbook](https://github.com/openai/openai-cookbook/blob/main/examples/Visualizing_embeddings_in_3D.ipynb)

 

