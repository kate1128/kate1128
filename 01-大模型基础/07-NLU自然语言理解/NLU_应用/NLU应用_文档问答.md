> 这个的话主要是会做一个召回文档的动作，然后让模型在这个文档中找答案
>

文档问答和[Embedding任务 QA](https://www.yuque.com/qiaokate/su87gb/akaneqxf2qvo9pt6)有点类似，不过要稍微复杂一点。它会先用`QA`的方法召回一个相关的文档，然后让模型在这个文档中找出问题的答案。一般的流程还是先召回相关文档，然后做阅读理解任务。阅读理解和实体提取任务有些类似，但它预测的不是具体某个标签，而是答案的`Index`，即`start`和`end`的位置。

> 还是举个例子。假设我们的问题是：“北京奥运会举办于哪一年？”
>

召回的文档可能是含有北京奥运会举办的新闻，比如类似下面这样的：

> 第29届夏季奥林匹克运动会（Beijing 2008; Games of the XXIX Olympiad），又称2008年北京奥运会，2008年8月8日晚上8时整在中国首都北京开幕。8月24日闭幕。
>

标注就是「2008年」这个答案的索引。

> 当然，一个文档里可能有不止一个问题，比如上面的文档，还可以问：“北京奥运会啥时候开幕？”，“北京奥运会什么时候闭幕”，“北京奥运会是第几届奥运会”等问题。
>

<br/>color2
根据之前的NLP方法，这里实际做起来方案会比较多，也有一定的复杂度；不过总的来说还是分类任务。现在我们有了LLM，问题就变得简单了。依然是两步：

1. **召回**：与上一章的`QA`类似，这次召回的是`Doc`，这一步其实就是相似`Embedding`选择最相似的。
2. **回答**：将召回来的文档和问题以`Prompt`的方式提交给`Completion/ChatCompletion`接口，直接得到答案。

<br/>

### 举例
#### OpenAI
```python
from openai import OpenAI
client = OpenAI(api_key="YOUR OPENAI KEY")
```

```python
def ask(content):
    response = client.chat.completions.create(
        model="gpt-3.5-turbo", 
        messages=[{"role": "user", "content": content}],
        max_tokens=300,
        top_p=1.0,
        frequency_penalty=0,
        presence_penalty=0,
    )

    ans = response.choices[0].message.content
    return ans
```

我们分别用中英文各举一例：

##### 英文
```python
# 来自官方文档
prompt = """Answer the question as truthfully as possible using the provided text, and if the answer is not contained within the text below, say "I don't know"

Context:
The men's high jump event at the 2020 Summer Olympics took place between 30 July and 1 August 2021 at the Olympic Stadium.
33 athletes from 24 nations competed; the total possible number depended on how many nations would use universality places 
to enter athletes in addition to the 32 qualifying through mark or ranking (no universality places were used in 2021).
Italian athlete Gianmarco Tamberi along with Qatari athlete Mutaz Essa Barshim emerged as joint winners of the event following
a tie between both of them as they cleared 2.37m. Both Tamberi and Barshim agreed to share the gold medal in a rare instance
where the athletes of different nations had agreed to share the same medal in the history of Olympics. 
Barshim in particular was heard to ask a competition official "Can we have two golds?" in response to being offered a 
'jump off'. Maksim Nedasekau of Belarus took bronze. The medals were the first ever in the men's high jump for Italy and 
Belarus, the first gold in the men's high jump for Italy and Qatar, and the third consecutive medal in the men's high jump
for Qatar (all by Barshim). Barshim became only the second man to earn three medals in high jump, joining Patrik Sjöberg
of Sweden (1984 to 1992).

Q: Who won the 2020 Summer Olympics men's high jump?
A:"""
```

```python
ask(prompt)
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>'Gianmarco Tamberi and Mutaz Essa Barshim.'</code></pre></details>
上面的`Context`就是我们召回的文档。

##### 中文
```python
prompt = """请根据以下Context回答问题，直接输出答案即可，不用附带任何上下文。

Context:
诺曼人（诺曼人：Nourmands；法语：Normands；拉丁语：Normanni）是在10世纪和11世纪将名字命名为法国诺曼底的人。他们是北欧人的后裔（丹麦人，挪威人和挪威人）的海盗和海盗，他们在首相罗洛（Rollo）的领导下向西弗朗西亚国王查理三世宣誓效忠。经过几代人的同化，并与法兰克和罗马高卢人本地居民融合，他们的后代将逐渐与以西卡罗来纳州为基础的加洛林人文化融合。诺曼人独特的文化和种族身份最初出现于10世纪上半叶，并在随后的几个世纪中持续发展。

问题：
诺曼底在哪个国家/地区？
"""
```

```python
ask(prompt)
```

<details class="lake-collapse"><summary id="u1db4a22e"><span class="ne-text">output：</span></summary><pre data-language="json" id="IQk6L" class="ne-codeblock language-json"><code>'法国'</code></pre></details>
#### ZhipuAI
```python
from zhipuai import ZhipuAI
zhipu_client = ZhipuAI(api_key="YOUR ZHIPU KEY")
```

```python
# 注意，智谱AI的接口参数和OpenAI的类似，但有些参数不支持
# 具体查看 https://open.bigmodel.cn/dev/api
def ask_zhipu(content):
    response = zhipu_client.chat.completions.create(
        model="glm-3-turbo", 
        messages=[{"role": "user", "content": content}],
        max_tokens=300,
        top_p=0.9,
    )

    ans = response.choices[0].message.content
    return ans
```

##### 中文
```python
prompt = """请根据以下Context回答问题，直接输出答案即可，不用附带任何上下文。

Context:
诺曼人（诺曼人：Nourmands；法语：Normands；拉丁语：Normanni）是在10世纪和11世纪将名字命名为法国诺曼底的人。他们是北欧人的后裔（丹麦人，挪威人和挪威人）的海盗和海盗，他们在首相罗洛（Rollo）的领导下向西弗朗西亚国王查理三世宣誓效忠。经过几代人的同化，并与法兰克和罗马高卢人本地居民融合，他们的后代将逐渐与以西卡罗来纳州为基础的加洛林人文化融合。诺曼人独特的文化和种族身份最初出现于10世纪上半叶，并在随后的几个世纪中持续发展。

问题：
诺曼底在哪个国家/地区？
"""
```

```python
ask_zhipu(prompt)
```

<details class="lake-collapse"><summary id="u9ed4033f"><span class="ne-text">output：</span></summary><pre data-language="json" id="eKsNS" class="ne-codeblock language-json"><code>'法国'</code></pre></details>
看起来还行，我们接下来就把整个流程串起来。

### 流程
#### 加载数据集
首先是加载数据集，取自：

[olympics_sections_text.csv](https://www.yuque.com/attachments/yuque/0/2025/csv/2639475/1735873336106-8c76ee5b-06ab-4847-a85c-9b723fc0dd90.csv)

[openai-cookbook/examples/fine-tuned_qa/olympics-1-collect-data.ipynb at 1f6c2304b401e931928e74e978d9a0b8a40d1cf7 · openai/openai-cookbook](https://github.com/openai/openai-cookbook/blob/1f6c2304b401e931928e74e978d9a0b8a40d1cf7/examples/fine-tuned_qa/olympics-1-collect-data.ipynb)

```python
import pandas as pd
df = pd.read_csv("./dataset/olympics_sections_text.csv")
df.shape
```

<details class="lake-collapse"><summary id="u6323daa5"><span class="ne-text">output：</span></summary><pre data-language="json" id="VRKYI" class="ne-codeblock language-json"><code>(3964, 4)</code></pre></details>


```python
df.head()
```

output：

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735610826163-11457c5e-5b94-4dca-b7f7-61d89ff33ba2.png)

#### 安装 Qdrant
我们这次不用Redis，换一个工具：[Qdrant - Vector Search Engine](https://qdrant.tech/)，Qdrant相比Redis的单线程更容易扩展。真正需要做的是将业务逻辑抽象，做到尽量不依赖任何工具，换工具只需要换一个适配器就好。

依然使用Docker，启动很简单：

```python
docker run -p 6333:6333 -v $(pwd)/qdrant_storage:/qdrant/storage qdrant/qdrant`
```

客户端的安装：

```python
pip install qdrant-client
```

#### 生成 Embedding
不过首先还是生成Embedding，这一步用Batch方法：

##### text-embedding-ada-002
```python
def get_embedding(inputs):
    embed_model = "text-embedding-ada-002"
    res = client.embeddings.create(
        input=inputs, model=embed_model
    )
    return res.data
```

##### embedding-2
```python
def get_embedding_zhipu(inputs):
    if isinstance(inputs, str):
        inputs = [inputs]
    res = []
    for s in inputs:
        resp = zhipu_client.embeddings.create(
            input=s, model="embedding-2"
        )
        emd = resp.data[0]
        res.append(emd)
    return res
```

##### texts 长度
```python
texts = [v.content for v in df.itertuples()]
len(texts)
```

<details class="lake-collapse"><summary id="u46526d4e"><span class="ne-text">output：</span></summary><pre data-language="json" id="QgTgp" class="ne-codeblock language-json"><code>3964</code></pre></details>
```python
import pnlp
```

##### 获取embedding
```python
emds = []
for idx, batch in enumerate(pnlp.generate_batches_by_size(texts, 200)):
    emd_data = get_embedding(batch)
    for v in emd_data:
        emds.append(v.embedding)
    print(f"batch: {idx} done")
```

<details class="lake-collapse"><summary id="ue4f6a751"><span class="ne-text">output：</span></summary><pre data-language="json" id="erm6S" class="ne-codeblock language-json"><code>batch: 0 done
batch: 1 done
batch: 2 done
batch: 3 done
batch: 4 done
batch: 5 done
batch: 6 done
batch: 7 done
batch: 8 done
batch: 9 done
batch: 10 done
batch: 11 done
batch: 12 done
batch: 13 done
batch: 14 done
batch: 15 done
batch: 16 done
batch: 17 done
batch: 18 done
batch: 19 done</code></pre></details>


```python
len(emds), len(emds[0])
```

<details class="lake-collapse"><summary id="u7aec4c78"><span class="ne-text">output：</span></summary><pre data-language="json" id="rBqFY" class="ne-codeblock language-json"><code>(3964, 1536)</code></pre></details>


#### embeddings 存储
由于量比较大，把embeddings 存储一下，可以直接加载使用

```python
import numpy as np
```

```python
arr = np.load("emds.npz")
```

```python
emds = arr["arr"].tolist()
```

```python
len(emds), len(emds[0])
```

<details class="lake-collapse"><summary id="u81413456"><span class="ne-text">output：</span></summary><pre data-language="json" id="Y23jT" class="ne-codeblock language-json"><code>(3964, 1536)</code></pre></details>


#### 创建索引
接下来是创建索引：

```python
from qdrant_client import QdrantClient
qc_client = QdrantClient(host="localhost", port=6333)
```

值得注意的是，qdrant还支持内存/文件库，也就是说，可以直接：

```python
# qc_client = QdrantClient(":memory:")
# 或
# qc_client = QdrantClient(path="path/to/db")
```

我们还是用server的方式：

```python
from qdrant_client.models import Distance, VectorParams

qc_client.recreate_collection(
    collection_name="doc_qa",
    vectors_config=VectorParams(size=len(emds[0]), distance=Distance.COSINE),
)
```

<details class="lake-collapse"><summary id="u5b4afbd4"><span class="ne-text">output：</span></summary><pre data-language="json" id="X0DCQ" class="ne-codeblock language-json"><code>/var/folders/xt/mpcnl_9151dbq43hptdlv2jr0000gn/T/ipykernel_73899/2165250407.py:3: DeprecationWarning: `recreate_collection` method is deprecated and will be removed in the future. Use `collection_exists` to check collection existence and `create_collection` instead.
  qc_client.recreate_collection(</code></pre></details>
<details class="lake-collapse"><summary id="ud8128ce4"><span class="ne-text">output：</span></summary><pre data-language="json" id="TRoSk" class="ne-codeblock language-json"><code>True</code></pre></details>


```python
# qc_client.delete_collection("doc_qa")
```

#### 向量入库
然后是把向量入库：

```python
payload=[
    {"content": v.content, "heading": v.heading, "title": v.title, "tokens": v.tokens} for v in df.itertuples()
]
```

```python
qc_client.upload_collection(
    collection_name="doc_qa",
    vectors=emds,
    payload=payload
)
```

#### 进行查询
接下来进行查询：

```python
query = "Who won the 2020 Summer Olympics men's high jump?"
query_vector = get_embedding(query)[0].embedding
hits = qc_client.search(
    collection_name="doc_qa",
    query_vector=query_vector,
    limit=5
)
```

```python
hits
```

<details class="lake-collapse"><summary id="u6e43af48"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="r9jMi" class="ne-codeblock language-json"><code>[ScoredPoint(id='f40dc5e1-3291-443a-b802-91e1998f3c3f', version=3, score=0.90292776, payload={'content': 'The men\'s high jump event at the 2020 Summer Olympics took place between 30 July and 1 August 2021 at the Olympic Stadium. 33 athletes from 24 nations competed; the total possible number depended on how many nations would use universality places to enter athletes in addition to the 32 qualifying through mark or ranking (no universality places were used in 2021). Italian athlete Gianmarco Tamberi along with Qatari athlete Mutaz Essa Barshim emerged as joint winners of the event following a tie between both of them as they cleared 2.37m. Both Tamberi and Barshim agreed to share the gold medal in a rare instance where the athletes of different nations had agreed to share the same medal in the history of Olympics. Barshim in particular was heard to ask a competition official &quot;Can we have two golds?&quot; in response to being offered a \'jump off\'. Maksim Nedasekau of Belarus took bronze. The medals were the first ever in the men\'s high jump for Italy and Belarus, the first gold in the men\'s high jump for Italy and Qatar, and the third consecutive medal in the men\'s high jump for Qatar (all by Barshim). Barshim became only the second man to earn three medals in high jump, joining Patrik Sjöberg of Sweden (1984 to 1992).', 'heading': 'Summary', 'title': &quot;Athletics at the 2020 Summer Olympics – Men's high jump&quot;, 'tokens': 275}, vector=None, shard_key=None),
 ScoredPoint(id='4c97dca4-2f99-4b78-a034-1a3bdfe66bab', version=4, score=0.88223696, payload={'content': &quot;The men's long jump event at the 2020 Summer Olympics took place between 31 July and 2 August 2021 at the Japan National Stadium. Approximately 35 athletes were expected to compete; the exact number was dependent on how many nations use universality places to enter athletes in addition to the 32 qualifying through time or ranking (1 universality place was used in 2016). 31 athletes from 20 nations competed. Miltiadis Tentoglou won the gold medal, Greece's first medal in the men's long jump. Cuban athletes Juan Miguel Echevarría and Maykel Massó earned silver and bronze, respectively, the nation's first medals in the event since 2008.&quot;, 'heading': 'Summary', 'title': &quot;Athletics at the 2020 Summer Olympics – Men's long jump&quot;, 'tokens': 136}, vector=None, shard_key=None),
 ScoredPoint(id='dbe9e9b6-e2b8-491d-a389-390ac0743077', version=4, score=0.8820208, payload={'content': &quot;The men's pole vault event at the 2020 Summer Olympics took place between 31 July and 3 August 2021 at the Japan National Stadium. 29 athletes from 18 nations competed. Armand Duplantis of Sweden won gold, with Christopher Nilsen of the United States earning silver and Thiago Braz of Brazil taking bronze. It was Sweden's first victory in the event and first medal of any color in the men's pole vault since 1952. Braz, who had won in 2016, became the ninth man to earn multiple medals in the pole vault.&quot;, 'heading': 'Summary', 'title': &quot;Athletics at the 2020 Summer Olympics – Men's pole vault&quot;, 'tokens': 112}, vector=None, shard_key=None),
 ScoredPoint(id='6cfd284d-adf3-4f82-8089-90cd76c1b8af', version=3, score=0.8760853, payload={'content': &quot;The men's triple jump event at the 2020 Summer Olympics took place between 3 and 5 August 2021 at the Japan National Stadium. Approximately 35 athletes were expected to compete; the exact number was dependent on how many nations use universality places to enter athletes in addition to the 32 qualifying through time or ranking (2 universality places were used in 2016). 32 athletes from 19 nations competed. Pedro Pichardo of Portugal won the gold medal, the nation's second victory in the men's triple jump (after Nelson Évora in 2008). China's Zhu Yaming took silver, while Hugues Fabrice Zango earned Burkina Faso's first Olympic medal in any event.&quot;, 'heading': 'Summary', 'title': &quot;Athletics at the 2020 Summer Olympics – Men's triple jump&quot;, 'tokens': 139}, vector=None, shard_key=None),
 ScoredPoint(id='5083441b-bf20-4efb-8338-7fb0a6cef789', version=3, score=0.86042714, payload={'content': &quot;The men's 110 metres hurdles event at the 2020 Summer Olympics took place between 3 and 5 August 2021 at the Olympic Stadium. Approximately forty athletes were expected to compete; the exact number was dependent on how many nations used universality places to enter athletes in addition to the 40 qualifying through time or ranking (1 universality place was used in 2016). 40 athletes from 29 nations competed. Hansle Parchment of Jamaica won the gold medal, the nation's second consecutive victory in the event. His countryman Ronald Levy took bronze. American Grant Holloway earned silver, placing the United States back on the podium in the event after the nation missed the medals for the first time in Rio 2016 (excluding the boycotted 1980 Games).&quot;, 'heading': 'Summary', 'title': &quot;Athletics at the 2020 Summer Olympics – Men's 110 metres hurdles&quot;, 'tokens': 149}, vector=None, shard_key=None)]</code></pre></details>


#### 将这个过程包装在Prompt生成过程中
接下来将这个过程包装在Prompt生成过程中：

```python
MAX_SECTION_LEN = 500
SEPARATOR = "\n* "
separator_len = 3
```

```python
def construct_prompt(question: str):
    query_vector = get_embedding(question)[0].embedding
    hits = qc_client.search(
        collection_name="doc_qa",
        query_vector=query_vector,
        limit=5
    )
    
    choose = []
    length = 0
    indexes = []
     
    for hit in hits:
        doc = hit.payload
        length += doc["tokens"] + separator_len
        if length > MAX_SECTION_LEN:
            break
            
        choose.append(SEPARATOR + doc["content"].replace("\n", " "))
        indexes.append(doc["title"] + doc["heading"])
            
    # Useful diagnostic information
    print(f"Selected {len(choose)} document sections:")
    print("\n".join(indexes))
    
    header = """Answer the question as truthfully as possible using the provided context, and if the answer is not contained within the text below, say "I don't know."\n\nContext:\n"""
    
    return header + "".join(choose) + "\n\n Q: " + question + "\n A:"
```

#### 示例：
```python
prompt = construct_prompt("Who won the 2020 Summer Olympics men's high jump?")

print("===\n", prompt)
```

<details class="lake-collapse"><summary id="uec6a959a"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="pEYTo" class="ne-codeblock language-json"><code>Selected 2 document sections:
Athletics at the 2020 Summer Olympics – Men's high jumpSummary
Athletics at the 2020 Summer Olympics – Men's long jumpSummary
===
 Answer the question as truthfully as possible using the provided context, and if the answer is not contained within the text below, say &quot;I don't know.&quot;

Context:

* The men's high jump event at the 2020 Summer Olympics took place between 30 July and 1 August 2021 at the Olympic Stadium. 33 athletes from 24 nations competed; the total possible number depended on how many nations would use universality places to enter athletes in addition to the 32 qualifying through mark or ranking (no universality places were used in 2021). Italian athlete Gianmarco Tamberi along with Qatari athlete Mutaz Essa Barshim emerged as joint winners of the event following a tie between both of them as they cleared 2.37m. Both Tamberi and Barshim agreed to share the gold medal in a rare instance where the athletes of different nations had agreed to share the same medal in the history of Olympics. Barshim in particular was heard to ask a competition official &quot;Can we have two golds?&quot; in response to being offered a 'jump off'. Maksim Nedasekau of Belarus took bronze. The medals were the first ever in the men's high jump for Italy and Belarus, the first gold in the men's high jump for Italy and Qatar, and the third consecutive medal in the men's high jump for Qatar (all by Barshim). Barshim became only the second man to earn three medals in high jump, joining Patrik Sjöberg of Sweden (1984 to 1992).
* The men's long jump event at the 2020 Summer Olympics took place between 31 July and 2 August 2021 at the Japan National Stadium. Approximately 35 athletes were expected to compete; the exact number was dependent on how many nations use universality places to enter athletes in addition to the 32 qualifying through time or ranking (1 universality place was used in 2016). 31 athletes from 20 nations competed. Miltiadis Tentoglou won the gold medal, Greece's first medal in the men's long jump. Cuban athletes Juan Miguel Echevarría and Maykel Massó earned silver and bronze, respectively, the nation's first medals in the event since 2008.

 Q: Who won the 2020 Summer Olympics men's high jump?
 A:</code></pre></details>


```python
ask(prompt)
```

<details class="lake-collapse"><summary id="u95f42aad"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="x0E31" class="ne-codeblock language-json"><code>&quot;Italian athlete Gianmarco Tamberi and Qatari athlete Mutaz Essa Barshim emerged as joint winners of the men's high jump event at the 2020 Summer Olympics.&quot;</code></pre></details>


```python
ask_zhipu(prompt)
```

<details class="lake-collapse"><summary id="ua0daa9c8"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="Lt8vP" class="ne-codeblock language-json"><code>&quot;Gianmarco Tamberi of Italy and Mutaz Essa Barshim of Qatar won the 2020 Summer Olympics men's high jump event, sharing the gold medal.&quot;</code></pre></details>


#### 再看几个例子
```python
query = "Why was the 2020 Summer Olympics originally postponed?"
prompt = construct_prompt(query)
answer = ask(prompt)

print(f"\nQ: {query}\nA: {answer}")
```

<details class="lake-collapse"><summary id="u9663f6f4"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="e73li" class="ne-codeblock language-json"><code>Selected 1 document sections:
Concerns and controversies at the 2020 Summer OlympicsSummary

Q: Why was the 2020 Summer Olympics originally postponed?
A: The 2020 Summer Olympics were originally postponed due to the COVID-19 pandemic.</code></pre></details>


```python
query = "In the 2020 Summer Olympics, how many gold medals did the country which won the most medals win?"
prompt = construct_prompt(query)
answer = ask_zhipu(prompt)

print(f"\nQ: {query}\nA: {answer}")
```

<details class="lake-collapse"><summary id="ud7021236"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="iLEnd" class="ne-codeblock language-json"><code>Selected 2 document sections:
2020 Summer Olympics medal tableSummary
List of 2020 Summer Olympics medal winnersSummary

Q: In the 2020 Summer Olympics, how many gold medals did the country which won the most medals win?
A: The country which won the most medals overall in the 2020 Summer Olympics was the United States. They won a total of 39 gold medals.</code></pre></details>


```python
query = "What is the tallest mountain in the world?"
prompt = construct_prompt(query)
answer = ask(prompt)

print(f"\nQ: {query}\nA: {answer}")
```

<details class="lake-collapse"><summary id="ue5871fd7"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="KhcSp" class="ne-codeblock language-json"><code>Selected 3 document sections:
Sport climbing at the 2020 Summer Olympics – Men's combinedRoute-setting
Ski mountaineering at the 2020 Winter Youth Olympics – Boys' individualSummary
Ski mountaineering at the 2020 Winter Youth Olympics – Girls' individualSummary

Q: What is the tallest mountain in the world?
A: I don't know.</code></pre></details>


```python
# ChatGPT
answer = ask_zhipu(prompt)

print(f"\nQ: {query}\nA: {answer}")
```

<details class="lake-collapse"><summary id="ue3c26f72"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="YxeCT" class="ne-codeblock language-json"><code>Q: What is the tallest mountain in the world?
A: I don't know.</code></pre></details>






