# RAG(Retrieval Augmented Generation)
# 一、RAG介绍
## 1.LLM的缺陷
1. LLM的知识不是实时的，不具备知识更新.
2. LLM可能不知道你私有的领域/业务知识.
3. LLM有时会在回答中生成看似合理但实际上是错误的信息.



## 2.为什么会用到RAG
1. 提高准确性: 通过检索相关的信息，RAG可以提高生成文本的准确性。
2. 减少训练成本：与需要大量数据来训练的大型生成模型相比，RAG可以通过检索机制来减少所需的训练数据量，从而降低训练成本。
3. 适应性强：RAG模型可以适应新的或不断变化的数据。由于它们能够检索最新的信息，因此在新数据和事件出现时，它们能够快速适应并生成相关的文本。



## 3.RAG概念
	RAG（Retrieval Augmented Generation）顾名思义，通过检索外部数据，增强⼤模型的⽣成效果。

	RAG即检索增强⽣成，为LLM提供了从某些数据源检索到的信息，并基于此修正⽣成的答案。RAG 基本上是Search + LLM 提示，可以通过⼤模型回答查询，并将搜索算法所找到的信息作为⼤模型的上下⽂。查询和检索到的上下⽂都会被注⼊到发送到 LLM 的提示语中。



## 4.RAG vs Fine-tuning
	RAG（检索增强⽣成）是把内部的⽂档数据先进⾏embedding，借助检索先获得⼤致的知识范围答案，再结合prompt给到LLM，让LLM⽣成最终的答案

	Fine-tuning（微调）是⽤⼀定量的数据集对LLM进⾏局部参数的调整，以期望LLM更加理解我们的业务逻辑，有更好的zero-shot能⼒。



## 5.RAG工作流程
RAG论⽂：[https://arxiv.org/pdf/2312.10997](https://arxiv.org/pdf/2312.10997)

![](img/image-20250217204604538.png)

对应的中文版本

![](img/image-20250217204629756.png)



## 6.RAG系统的搭建流程
	具体的搭建流程图如下：

![](img/图片1.png)

**索引**（Indexing）：索引⾸先清理和提取各种格式的原始数据，如 PDF、HTML、 Word 和 Markdown，然后将其转换为统⼀的纯⽂本格式。为了适应语⾔模型的上下⽂限制，⽂本被分割成更⼩的、可消化的块（chunk）。**然后使⽤嵌⼊模型将块编码成向量表示**，并存储在向量数据库中。这⼀步对于在随后的检索阶段实现⾼效的相似性搜索⾄关重要。知识库分割成 chunks，并将 chunks 向量化⾄向量库中。



**检索**（Retrieval）：在收到⽤户查询（Query）后，RAG 系统采⽤与索引阶段**相同的编码模型**将查询转换为向量表示，然后计算索引语料库中查询向量与块向量的相似性得分。该系统优先级和检索最⾼ k （Top-K）块，显示最⼤的相似性查询。

例如，⼆维空间中的向量可以表示为 (𝑥,𝑦)，表示从原点 (0,0) 到点 (𝑥,𝑦) 的有向线段

![](img/image-20250217205614429.png)

1. 将⽂本转成⼀组浮点数：每个下标 i ，对应⼀个维度
2. 整个数组对应⼀个 n 维空间的⼀个点，即⽂本向量⼜叫 Embeddings
3. 向量之间可以计算距离，距离远近对应语义相似度⼤⼩

![](img/image-20250217205650061.png)

这些块随后被⽤作 prompt 中的扩展上下⽂。Query 向量化，匹配向量空间中相近的 chunks。



RAG具体实现流程：加载⽂件 => 读取⽂本 => ⽂本分割 =>⽂本向量化 =>输⼊问题向量化 =>在⽂本向量中匹配出与问题向量最相似的 top k 个 =>匹配出的⽂本作为上下⽂和问题⼀起添加到 prompt 中 =>提交给 LLM ⽣成回答



# 二、RAG核心
## 1.传统VS⼤模型
	智能客服系统在没有大模型之前我们也是可以设计完成的只是实现的效果没有大模型那么好。下面是两则设计的原理

![](img/image-20250217210117700.png)



## 2.向量与Embeddings的定义
	在数学中，向量（也称为欧⼏⾥得向量、⼏何向量），指具有⼤⼩（magnitude）和⽅向的量。它可以形象化地表示为带箭头的线段。箭头所指：代表向量的⽅向；线段⻓度：代表向量的⼤⼩。

![](img/image-20250217210446321.png)

具体生成向量的代码

```python
from openai import OpenAI
from dotenv import load_dotenv
load_dotenv()
import os

client = OpenAI()
def get_embeddings(texts, model="text-embedding-3-large"):
 # texts 是⼀个包含要获取嵌⼊表示的⽂本的列表，
 # model 则是⽤来指定要使⽤的模型的名称
 # ⽣成⽂本的嵌⼊表示。结果存储在data中。
 data = client.embeddings.create(input=texts, model=model).data
 # print(data)
 # 返回了⼀个包含所有嵌⼊表示的列表
 return [x.embedding for x in data]
test_query = ["⼤模型"]
vec = get_embeddings(test_query)
# "⼤模型" ⽂本嵌⼊表示的列表。
print(vec)
# "⼤模型" ⽂本的嵌⼊表示。
print(vec[0])
# "⼤模型" ⽂本的嵌⼊表示的维度。3072
print(len(vec[0]))
```

**text-embedding-3-large**  是 OpenAI 推出的一个文本嵌入模型，属于 **text-embedding-3** 系列中的大尺寸版本。是一个功能强大、灵活性高的文本嵌入模型，适合处理复杂的自然语言任务。



## 3.向量间的相似度计算
![](img/image-20250217211053356.png)



具体的案例演示代码

```python
from openai import OpenAI
from dotenv import load_dotenv
import numpy as np
from numpy import dot
from numpy.linalg import norm
load_dotenv()
import os

client = OpenAI()
def cos_sim(a, b):
 '''余弦距离 -- 越⼤越相似'''
 return dot(a, b)/(norm(a)*norm(b))
def l2(a, b):
 '''欧式距离 -- 越⼩越相似'''
 x = np.asarray(a)-np.asarray(b)
 return norm(x)
def get_embeddings(texts, model="text-embedding-3-large"):
 data = client.embeddings.create(input=texts, model=model).data
 # print(data)
 # 返回了⼀个包含所有嵌⼊表示的列表
 return [x.embedding for x in data]

# 且能⽀持跨语⾔
query = "global conflicts"
# query = "国际争端"
documents = [
 "联合国安理会上，俄罗斯与美国，伊朗与以⾊列“吵”起来了",
 "⼟⽿其、芬兰、瑞典与北约代表将继续就瑞典“⼊约”问题进⾏谈判",
 "⽇本岐⾩市陆上⾃卫队射击场内发⽣枪击事件 3⼈受伤",
 "孙志刚被判死缓 减为⽆期徒刑后终身监禁 不得减刑、假释",
 "以⾊列⽴法禁联合国机构，美表态担忧，中东局势再⽣波澜",
]
query_vec = get_embeddings([query])[0]
doc_vecs = get_embeddings(documents)
print("Cosine distance:")
print(cos_sim(query_vec, query_vec))
for vec in doc_vecs:
 print(cos_sim(query_vec, vec))
print("\nEuclidean distance:")
print(l2(query_vec, query_vec))
for vec in doc_vecs:
 print(l2(query_vec, vec))
```

输出的结果

```plain
Cosine distance:
1.0
0.26485647419163133
0.17843340536145408
0.1389039336840151
0.029972895954156878
0.349804816400828

Euclidean distance:
0.0
1.212553924472882
1.2818475860144856
1.3123231844462315
1.3928582773078655
1.1403465897787535
```



## 4.文档的加载和分割
	

### 4.1 基于文档的LLM回复系统搭建
![](img/image-20250217211624986.png)

### 4.2 把⽂本切分成chunks
	我们把文本切分成chunks的方式有很多种:

1. 按照句⼦来切分
2. 按照字符数来切分
3. 按固定字符数 结合overlapping window
4. 递归⽅法 RecursiveCharacterTextSplitter



#### 4.2.1 按照句⼦来切分
```python
# coding=utf-8
import re

text = "自然语言处理（NLP），作为计算机科学、人工智能与语言学的交融之地，致力于赋予计算机解析和处理人类语言的能力。在这个领域，机器学习发挥着至关重要的作用。利用多样的算法，机器得以分析、领会乃至创造我们所理解的语言。从机器翻译到情感分析，从自动摘要到实体识别，NLP的应用已遍布各个领域。随着深度学习技术的飞速进步，NLP的精确度与效能均实现了巨大飞跃。如今，部分尖端的NLP系统甚至能够处理复杂的语言理解任务，如问答系统、语音识别和对话系统等。NLP的研究推进不仅优化了人机交流，也对提升机器的自主性和智能水平起到了关键作用。"

# 正则表达式匹配中文句子结束的标点符号
sentences = re.split(r'(。|？|！|\…\…)', text)

# 重新组合句子和结尾的标点符号
chunks = [sentence + (punctuation if punctuation else '') for sentence, punctuation in zip(sentences[::2], sentences[1::2])]

for i, chunk in enumerate(chunks):
    print(f"块 {i+1}: {len(chunk)}: {chunk}")
```

输出的结果

```plain
块 1: 55: 自然语言处理（NLP），作为计算机科学、人工智能与语言学的交融之地，致力于赋予计算机解析和处理人类语言的能力。
块 2: 21: 在这个领域，机器学习发挥着至关重要的作用。
块 3: 30: 利用多样的算法，机器得以分析、领会乃至创造我们所理解的语言。
块 4: 36: 从机器翻译到情感分析，从自动摘要到实体识别，NLP的应用已遍布各个领域。
块 5: 33: 随着深度学习技术的飞速进步，NLP的精确度与效能均实现了巨大飞跃。
块 6: 46: 如今，部分尖端的NLP系统甚至能够处理复杂的语言理解任务，如问答系统、语音识别和对话系统等。
块 7: 41: NLP的研究推进不仅优化了人机交流，也对提升机器的自主性和智能水平起到了关键作用。
```



#### 4.2.2 按照字符数来切分
```python
# coding=utf-8
import re

text = "自然语言处理（NLP），作为计算机科学、人工智能与语言学的交融之地，致力于赋予计算机解析和处理人类语言的能力。在这个领域，机器学习发挥着至关重要的作用。利用多样的算法，机器得以分析、领会乃至创造我们所理解的语言。从机器翻译到情感分析，从自动摘要到实体识别，NLP的应用已遍布各个领域。随着深度学习技术的飞速进步，NLP的精确度与效能均实现了巨大飞跃。如今，部分尖端的NLP系统甚至能够处理复杂的语言理解任务，如问答系统、语音识别和对话系统等。NLP的研究推进不仅优化了人机交流，也对提升机器的自主性和智能水平起到了关键作用。"


def split_by_fixed_char_count(text, count):
 return [text[i:i + count] for i in range(0, len(text), count)]


# 假设我们按照每100个字符来切分文本
chunks = split_by_fixed_char_count(text, 100)

for i, chunk in enumerate(chunks):
 print(f"块 {i + 1}: {len(chunk)}: {chunk}")
```

输出结果

```plain
块 1: 100: 自然语言处理（NLP），作为计算机科学、人工智能与语言学的交融之地，致力于赋予计算机解析和处理人类语言的能力。在这个领域，机器学习发挥着至关重要的作用。利用多样的算法，机器得以分析、领会乃至创造我们所
块 2: 100: 理解的语言。从机器翻译到情感分析，从自动摘要到实体识别，NLP的应用已遍布各个领域。随着深度学习技术的飞速进步，NLP的精确度与效能均实现了巨大飞跃。如今，部分尖端的NLP系统甚至能够处理复杂的语言理
块 3: 62: 解任务，如问答系统、语音识别和对话系统等。NLP的研究推进不仅优化了人机交流，也对提升机器的自主性和智能水平起到了关键作用。
```





#### 4.2.3 按固定字符数加滑动窗口
```python
# coding=utf-8
import re

text = "自然语言处理（NLP），作为计算机科学、人工智能与语言学的交融之地，致力于赋予计算机解析和处理人类语言的能力。在这个领域，机器学习发挥着至关重要的作用。利用多样的算法，机器得以分析、领会乃至创造我们所理解的语言。从机器翻译到情感分析，从自动摘要到实体识别，NLP的应用已遍布各个领域。随着深度学习技术的飞速进步，NLP的精确度与效能均实现了巨大飞跃。如今，部分尖端的NLP系统甚至能够处理复杂的语言理解任务，如问答系统、语音识别和对话系统等。NLP的研究推进不仅优化了人机交流，也对提升机器的自主性和智能水平起到了关键作用。"


def sliding_window_chunks(text, chunk_size, stride):
 return [text[i:i + chunk_size] for i in range(0, len(text), stride)]


chunks = sliding_window_chunks(text, 100, 50)  # 100个字符的块，步长为50

for i, chunk in enumerate(chunks):
 print(f"块 {i + 1}: {len(chunk)}: {chunk}")
```

输出的结果

```plain
块 1: 100: 自然语言处理（NLP），作为计算机科学、人工智能与语言学的交融之地，致力于赋予计算机解析和处理人类语言的能力。在这个领域，机器学习发挥着至关重要的作用。利用多样的算法，机器得以分析、领会乃至创造我们所
块 2: 100: 言的能力。在这个领域，机器学习发挥着至关重要的作用。利用多样的算法，机器得以分析、领会乃至创造我们所理解的语言。从机器翻译到情感分析，从自动摘要到实体识别，NLP的应用已遍布各个领域。随着深度学习技术
块 3: 100: 理解的语言。从机器翻译到情感分析，从自动摘要到实体识别，NLP的应用已遍布各个领域。随着深度学习技术的飞速进步，NLP的精确度与效能均实现了巨大飞跃。如今，部分尖端的NLP系统甚至能够处理复杂的语言理
块 4: 100: 的飞速进步，NLP的精确度与效能均实现了巨大飞跃。如今，部分尖端的NLP系统甚至能够处理复杂的语言理解任务，如问答系统、语音识别和对话系统等。NLP的研究推进不仅优化了人机交流，也对提升机器的自主性和
块 5: 62: 解任务，如问答系统、语音识别和对话系统等。NLP的研究推进不仅优化了人机交流，也对提升机器的自主性和智能水平起到了关键作用。
块 6: 12: 智能水平起到了关键作用。
```



#### 4.2.4 递归方法
 通过递归的方式处理我们需要借助langchain来实现。所以需要添加对应的模块

```plain
pip install langchain
```

```python
# coding=utf-8
import re

text = "自然语言处理（NLP），作为计算机科学、人工智能与语言学的交融之地，致力于赋予计算机解析和处理人类语言的能力。在这个领域，机器学习发挥着至关重要的作用。利用多样的算法，机器得以分析、领会乃至创造我们所理解的语言。从机器翻译到情感分析，从自动摘要到实体识别，NLP的应用已遍布各个领域。随着深度学习技术的飞速进步，NLP的精确度与效能均实现了巨大飞跃。如今，部分尖端的NLP系统甚至能够处理复杂的语言理解任务，如问答系统、语音识别和对话系统等。NLP的研究推进不仅优化了人机交流，也对提升机器的自主性和智能水平起到了关键作用。"

from langchain.text_splitter import RecursiveCharacterTextSplitter

# pip install langchain==0.2.1
text = """
自然语言处理（NLP），作为计算机科学、人工智能与语言学的交融之地，致力于赋予计算机解析和处理人类语言的能力。在这个领域，机器学习发挥着至关重要的作用。利用多样的算法，机器得以分析、领会乃至创造我们所理解的语言。从机器翻译到情感分析，从自动摘要到实体识别，NLP的应用已遍布各个领域。随着深度学习技术的飞速进步，NLP的精确度与效能均实现了巨大飞跃。如今，部分尖端的NLP系统甚至能够处理复杂的语言理解任务，如问答系统、语音识别和对话系统等。NLP的研究推进不仅优化了人机交流，也对提升机器的自主性和智能水平起到了关键作用。
"""

splitter = RecursiveCharacterTextSplitter(
 chunk_size=50,
 chunk_overlap=10,
 length_function=len,
)

chunks = splitter.split_text(text)

for i, chunk in enumerate(chunks):
 print(f"块 {i + 1}: {len(chunk)}: {chunk}")
```

输出的结果

```plain
块 1: 49: 自然语言处理（NLP），作为计算机科学、人工智能与语言学的交融之地，致力于赋予计算机解析和处理人类
块 2: 50: 计算机解析和处理人类语言的能力。在这个领域，机器学习发挥着至关重要的作用。利用多样的算法，机器得以分
块 3: 50: 样的算法，机器得以分析、领会乃至创造我们所理解的语言。从机器翻译到情感分析，从自动摘要到实体识别，N
块 4: 50: 动摘要到实体识别，NLP的应用已遍布各个领域。随着深度学习技术的飞速进步，NLP的精确度与效能均实现
块 5: 50: 的精确度与效能均实现了巨大飞跃。如今，部分尖端的NLP系统甚至能够处理复杂的语言理解任务，如问答系统
块 6: 50: 理解任务，如问答系统、语音识别和对话系统等。NLP的研究推进不仅优化了人机交流，也对提升机器的自主性
块 7: 23: 也对提升机器的自主性和智能水平起到了关键作用。
```



## 5. 向量检索
检索的⽅式有哪些？列举两种:

```plain
1.关键字搜索：通过⽤户输⼊的关键字来查找⽂本数据
2.语义搜索：不仅考虑关键词的匹配，还考虑词汇之间的语义关系，以提供更准确的搜索结果。
```

### 5.1 关键字搜索
	我们需要把相关的信息存储在Redis中。我们需要先按照一个Redis。在提供的资料中有。直接解压缩。然后cmd进入到对应的目录。然后输入

```plain
redis-server.exe
```

来启动Redis服务

![](img/image-20250217214829641.png)

然后需要安装Redis模块

```plain
pip install redis
```

然后我们可以通过代码把我们的数据导入到Redis中去

```python
from openai import OpenAI
from dotenv import load_dotenv
import redis  # pip install redis
import json
load_dotenv()  # 从我们的env文件中加载出对应的环境变量
#import os
#os.environ["http_proxy"] = "http://127.0.0.1:1083"
#os.environ["https_proxy"] = "http://127.0.0.1:1083"

client = OpenAI()

# 连接 Redis
r = redis.Redis(host='localhost', port=6379, decode_responses=True)

# 读取数据
with open('train_zh.json', 'r', encoding='utf-8') as f:
    data = [json.loads(line) for line in f]

# 取出问题和输出数据
instructions = [entry['instruction'] for entry in data[0:1000]]
outputs = [entry['output'] for entry in data[0:1000]]
# print("instructions", instructions)
# print("outputs", outputs)

# 将数据存储到 Redis
for instruction, output in zip(instructions, outputs):
    r.set(instruction, output)  # 存入 Redis，值序列化为 JSON


# 搜索函数：根据关键字搜索 instruction 中包含该关键字的条目
def search_instructions(keyword, top_n=3):
    # 通过模糊匹配
    keys = r.keys(pattern="*" + keyword + "*")
    data = []
    for key in keys:
        data.append(r.get(key))
    return data[:top_n]
```

然后可以通过RDM工具来查看导入的数据：[https://blog.csdn.net/itbysj/article/details/145383177](https://blog.csdn.net/itbysj/article/details/145383177)

![](img/image-20250217215820940.png)



**LLM 接⼝封装**:

```python
from openai import OpenAI

client = OpenAI()

def get_completion(prompt, model="gpt-3.5-turbo"):
    '''封装 openai 接口'''
    messages = [{"role": "user", "content": prompt}]
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0,  # 模型输出的随机性，0 表示随机性最小
    )
    return response.choices[0].message.content
```

**Prompt模板**

```python
user_query = "白癜风"
#user_query = "家里有甲醛,怎么办？"

# 1. 检索
search_results = search_instructions(user_query, 3)
search_results = "".join(search_results)
# print(search_results)

# 2. 构建 Prompt
prompt =  f"""
你是一个问答机器人。
你的任务是根据下述给定的已知信息回答用户问题。
确保你的回复完全依据下述已知信息。不要编造答案。
如果下述已知信息不足以回答用户的问题，请直接回复"我无法回答您的问题"。

已知信息:
{search_results}

用户问：
{user_query}

请用中文回答用户问题。
"""
print("===Prompt===")
print(prompt)
print("===Prompt===")

# 3. 调用 LLM
response = get_completion(prompt)

print("===回复===")
print(response)
```



### 5.2 向量数据库
	在⼈⼯智能时代，向量数据库已成为数据管理和AI模型不可或缺的⼀部分。向量数据库是⼀种专⻔设计⽤来存储和查询向量嵌⼊数据的数据库。这些向量嵌⼊是AI模型⽤于识别模式、关联和潜在结构的关键数据表示。

	随着AI和机器学习应⽤的普及，这些模型⽣成的嵌⼊包含⼤量属性或特征，使得它们的表示难以管理。这就是为什么数据从业者需要⼀种专⻔为处理这种数据⽽开发的数据库，这就是向量数据库的⽤武之地。

**Pinecone**

Pinecone: [www.pinecone.io/](http://www.pinecone.io/)

Pinecone的关键特性包括：

+ 重复检测：帮助⽤户识别和删除重复的数据
+ 排名跟踪：跟踪数据在搜索结果中的排名，有助于优化和调整搜索策略
+ 数据搜索：快速搜索数据库中的数据，⽀持复杂的搜索条件
+ 分类：对数据进⾏分类，便于管理和检索
+ 去重：⾃动识别和删除重复数据，保持数据集的纯净和⼀致性

**Milvus**

Milvus: milvus.io/

Milvus的关键特性包括：

+ 毫秒级搜索万亿级向量数据集
+ 简单管理⾮结构化数据
+ 可靠的向量数据库，始终可⽤
+ ⾼度可扩展和适应性强
+ 混合搜索
+ 统⼀的Lambda结构
+ 受到社区⽀持，得到⾏业认可

**Chroma**

Chroma: [www.trychroma.com/](http://www.trychroma.com/)

Chroma的关键特性包括:

+ 功能丰富：⽀持查询、过滤、密度估计等多种功能
+ 即将添加的语⾔链（LangChain）、LlamaIndex等更多功能
+ 相同的API可以在Python笔记本中运⾏，也可以扩展到集群，⽤于开发、测试和⽣产

**Faiss**

Faiss:[https://github.com/facebookresearch/faiss](https://github.com/facebookresearch/faiss)

Faiss的关键特性包括：

+ 不仅返回最近的邻居，还返回第⼆近、第三近和第k近的邻居
+ 可以同时搜索多个向量，⽽不仅仅是单个向量（批量处理）
+ 使⽤最⼤内积搜索⽽不是最⼩欧⼏⾥得搜索
+ 也⽀持其他距离度量，但程度较低。
+ 返回查询位置附近指定半径内的所有元素（范围搜索）
+ 可以将索引存储在磁盘上，⽽不仅仅是RAM中



**如何选型向量数据库**

	在选择适合项⽬的向量数据库时，需要根据项⽬的具体需求、团队的技术背景和资源情况来综合评估。以下是⼀些建议和注意事项：

向量嵌⼊的⽣成

+ 如果已经有了⾃⼰的向量嵌⼊⽣成模型，那么需要的是⼀个能够⾼效存储和查询这些向量的数据库
+ 如果需要数据库服务来⽣成向量嵌⼊，那么应该选择提供这类功能的产品

延迟要求

+ 对于需要实时响应的应⽤程序，低延迟是关键。需要选择能够提供快速查询响应的数据库
+ 如果应⽤程序允许批量处理，那么可以选择那些优化了⼤批量数据处理的数据库

开发⼈员的经验

+ 根据团队的技术栈和经验，选择⼀个易于集成和使⽤的数据库
+ 如果团队成员对某些技术或框架更熟悉，那么选择⼀个能够与之⽆缝集成的数据库会更有利

**chromadb演示**

安装chromadb模块

```plain
pip install chromadb
```

把数据存储到向量数据库，并通过向量数据库完成了检索

```python
from openai import OpenAI
import chromadb
from chromadb.config import Settings
from dotenv import load_dotenv
import json
load_dotenv()
import os


client = OpenAI()


# 要换成  text-embedding-3-large
def get_embeddings(texts, model="text-embedding-3-large"):
    '''封装 OpenAI 的 Embedding 模型接口'''
    data = client.embeddings.create(input=texts, model=model).data
    return [x.embedding for x in data]


with open('train_zh.json', 'r', encoding='utf-8') as f:
    data = [json.loads(line) for line in f]


# print(data[0:100])
instructions = [entry['instruction'] for entry in data[0:100]]
outputs = [entry['output'] for entry in data[0:100]]


class MyVectorDBConnector:
    def __init__(self, collection_name, embedding_fn):
        chroma_client = chromadb.Client(Settings(allow_reset=True))

        # 为了演示，实际不需要每次 reset()
        chroma_client.reset()

        # 创建一个 collection
        self.collection = chroma_client.get_or_create_collection(name=collection_name)
        # self.embedding_fn = get_embeddings
        self.embedding_fn = embedding_fn

    def add_documents(self, instructions, outputs):
        '''向 collection 中添加文档与向量'''
        # get_embeddings(instructions)  问题做了向量化
        embeddings = self.embedding_fn(instructions)

        self.collection.add(
            embeddings=embeddings,  # 每个文档的向量
            documents=outputs,  # 文档的原文
            ids=[f"id{i}" for i in range(len(outputs))]  # 每个文档的 id
        )

        # print(self.collection.count())


    def search(self, query, top_n):
        '''检索向量数据库'''
        results = self.collection.query(
            # get_embeddings([query])
            query_embeddings=self.embedding_fn([query]),
            n_results=top_n
        )
        return results


# 创建一个向量数据库对象
vector_db = MyVectorDBConnector("demo", get_embeddings)

# 向向量数据库中添加文档
vector_db.add_documents(instructions, outputs)

# user_query = "白癜风"
user_query = "得了白癜风怎么办？"
results = vector_db.search(user_query, 2)
# print(results)

for para in results['documents'][0]:
    print(para + "\n")
```

输出的结果

```plain
白癜风的治疗费用因个体差异和治疗方案的不同而有所差异。初期治疗主要以口服药物和外用药物为主，费用相对较低，一般几百元左右。但是，如果采用激光治疗、光疗等高端治疗方法，费用会更高。建议您咨询专业医生，根据自己的情况进行治疗方案的选择，同时了解相关的费用情况。

根据您提供的信息，孩子身上的白斑可能是多种原因导致的，例如真菌感染、色素脱失、营养不良等。白癜风是一种色素脱失性疾病，其特征是皮肤上出现白色斑块，但白斑界线不明显的情况不太符合白癜风的特点。建议您带孩子到医院皮肤科就诊，由专业医生进行诊断和治疗。

```





### 5.3 基于向量检索的RAG实现
```python
from dotenv import load_dotenv

load_dotenv()
from openai import OpenAI
import chromadb
from chromadb.config import Settings

import json


client = OpenAI()

prompt_template = """
你是一个问答机器人。
你的任务是根据下述给定的已知信息回答用户问题。
确保你的回复完全依据下述已知信息。不要编造答案。
如果下述已知信息不足以回答用户的问题，请直接回复"我无法回答您的问题"。

已知信息:
__INFO__

用户问：
__QUERY__

请用中文回答用户问题。
"""

with open('train_zh.json', 'r', encoding='utf-8') as f:
    data = [json.loads(line) for line in f]

# print(data[0:100])
instructions = [entry['instruction'] for entry in data[0:1000]]
outputs = [entry['output'] for entry in data[0:1000]]


def get_completion(prompt, model="gpt-4o"):
    '''封装 openai 接口'''
    messages = [{"role": "user", "content": prompt}]
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0,  # 模型输出的随机性，0 表示随机性最小
    )
    return response.choices[0].message.content


def build_prompt(prompt_template, **kwargs):
    '''将 Prompt 模板赋值'''
    prompt = prompt_template
    for k, v in kwargs.items():
        if isinstance(v, str):
            val = v
        elif isinstance(v, list) and all(isinstance(elem, str) for elem in v):
            val = '\n'.join(v)
        else:
            val = str(v)
        prompt = prompt.replace(f"__{k.upper()}__", val)
    return prompt


class MyVectorDBConnector:
    def __init__(self, collection_name, embedding_fn):
        chroma_client = chromadb.Client(Settings(allow_reset=True))

        # 为了演示，实际不需要每次 reset()
        chroma_client.reset()

        # 创建一个 collection
        self.collection = chroma_client.get_or_create_collection(name=collection_name)
        self.embedding_fn = embedding_fn

    def add_documents(self, instructions, outputs):
        '''向 collection 中添加文档与向量'''
        embeddings = self.embedding_fn(instructions)

        if len(embeddings) != len(instructions) or len(instructions) != len(outputs):
            raise ValueError("嵌入向量、instructions 和 outputs 数量不一致")

        self.collection.add(
            embeddings=embeddings,  # 每个文档的向量
            documents=outputs,  # 文档的原文
            ids=[f"id{i}" for i in range(len(outputs))]  # 每个文档的 id
        )

    def search(self, query, top_n):
        '''检索向量数据库'''
        results = self.collection.query(
            query_embeddings=self.embedding_fn([query]),
            n_results=top_n
        )
        return results


def get_embeddings(texts, model="text-embedding-3-large"):
    '''封装 OpenAI 的 Embedding 模型接口'''
    data = client.embeddings.create(input=texts, model=model).data
    return [x.embedding for x in data]


# 创建一个向量数据库对象
vector_db = MyVectorDBConnector("demo", get_embeddings)

# 向向量数据库中添加文档
vector_db.add_documents(instructions, outputs)


class RAG_Bot:
    def __init__(self, vector_db, llm_api, n_results=2):
        self.vector_db = vector_db
        self.llm_api = llm_api
        self.n_results = n_results

    def chat(self, user_query):
        # 1. 检索
        search_results = self.vector_db.search(user_query, self.n_results)

        # 2. 构建 Prompt
        prompt = build_prompt(
            prompt_template, info=search_results['documents'][0], query=user_query)
        # print("="*50)
        # print(prompt)
        # print("="*50)
        # 3. 调用 LLM
        response = self.llm_api(prompt)
        return response


# 创建一个RAG机器人
bot = RAG_Bot(
    vector_db,
    llm_api=get_completion
)

user_query = "拉肚子怎么办？"

response = bot.chat(user_query)

print(response)
```

输出的结果

```plain
您好，拉肚子可能是许多不同原因导致的，例如食物中毒、肠炎、病毒感染等。建议您注意以下几点：
1. 多喝水，保持水分，防止脱水。可以选择椰子水、果汁或盐水等饮品。
2. 避免食用生肉、生蛋、生海鲜等容易感染细菌的食物。
3. 避免食用过多的辛辣、油腻、刺激性食物，以及含有咖啡因和酒精的饮料。
4. 注意个人卫生，勤洗手，避免与患病的人接触。
5. 保持充足的休息，放松心情，避免过度劳累。
6. 注意饮食，少食多餐，可以吃些易消化、营养丰富的食物，如米粥、面条、鸡蛋、鸡肉等。
如果症状严重或持续加重，建议您去医院就诊，接受医生的诊断和治疗。
```



# 三、各大平台RAG实现
## 1.阿⾥云-百炼RAG
创建应⽤->上传数据->知识索引   [https://bailian.console.aliyun.com/](https://bailian.console.aliyun.com/)

![](img/image-20250218165554173.png)



通过Python来调用ARG服务

```python
# sk-31bb7a65dd4047aba9b14a95c08be52c
import os
from http import HTTPStatus
from dashscope import Application
response = Application.call(
    # 若没有配置环境变量，可用百炼API Key将下行替换为：api_key="sk-xxx"。但不建议在生产环境中直接将API Key硬编码到代码中，以减少API Key泄露风险。
    api_key="输入你自身的key",
    app_id='输入你自身的appId',
    prompt='观察者模式的介绍？你是基于知识库回答的还是基于你自身来回复的？')

if response.status_code != HTTPStatus.OK:
    print(f'request_id={response.request_id}')
    print(f'code={response.status_code}')
    print(f'message={response.message}')
    print(f'请参考文档：https://help.aliyun.com/zh/model-studio/developer-reference/error-code')
else:
    print(response.output.text)
```





## 2. 智普RAG
创建应⽤->上传数据

[https://www.zhipuai.cn/](https://www.zhipuai.cn/)





## 3.Google的NoteBook
[https://notebooklm.google/](https://notebooklm.google/)

![](img/image-20250218201049604.png)





# 四、本地向量库
	前面的案例中我们的向量化模型使用的是OpenAI提供的`text-embedding-3-large`,针对这块我们可以本地话一个向量模型。这样处理起来效率会更高一些。

**将向量化的结果保存**      

        huggingface：[https://huggingface.co/models](https://huggingface.co/models)      

        国内的镜像：[https://hf-mirror.com/](https://hf-mirror.com/) 



直接使用我们提供的本地的向量模型，并且拷贝到项目中

![](img/image-20250218223652144.png)     

需要安装下这个依赖

```plain
pip install sentence_transformers
```



然后将数据保存到向量数据库并且持久化到本地

```python
from sentence_transformers import SentenceTransformer
import json
import numpy as np
import os
import chromadb

model = SentenceTransformer(r'maidalun1020')

with open('train_zh.json', 'r', encoding='utf-8') as f:
    data = [json.loads(line) for line in f]

instructions = [entry['instruction'] for entry in data[0:1000]]
outputs = [entry['output'] for entry in data[0:1000]]


instruction_embeddings = model.encode(instructions, convert_to_numpy=True)
np.save('instruction_embeddings.npy', instruction_embeddings)

client = chromadb.PersistentClient(path="./collection.pkl")
collection = client.create_collection(name="demo")

for i, (sentence, embedding) in enumerate(zip(instructions, instruction_embeddings)):
    collection.add(
        documents=[sentence],
        embeddings=[embedding.tolist()],
        ids=[f"id-{i}"],
        metadatas=[{"output": outputs[i]}]
    )
```

然后我们就可以结合LLM来实现对应的RAG案例效果

```python
import numpy as np
from openai import OpenAI
import chromadb
from dotenv import load_dotenv
import os
from sentence_transformers import SentenceTransformer

load_dotenv()


client_openai = OpenAI()

# 获取知识库对应的向量信息
instruction_embeddings = np.load('instruction_embeddings.npy')

# 加载chromadb 持久化到本地的数据
client = chromadb.PersistentClient(path="./collection.pkl")
collection = client.get_collection(name="demo")
print(collection.count())

model = SentenceTransformer(r'maidalun1020')


def retrieve_response(query, top_k=5):
    query_embedding = model.encode([query], convert_to_numpy=True)
    # print(query_embedding)
    results = collection.query(
        query_embeddings=query_embedding.tolist(),
        n_results=top_k
    )
    return [metadata['output'] for metadata_list in results['metadatas'] for metadata in metadata_list]


prompt_template = """你是一个问答机器人。
你的任务是根据下述给定的已知信息回答用户问题。
确保你的回复完全依据下述已知信息。不要编造答案。
如果下述已知信息不足以回答用户的问题，请直接回复"我无法回答您的问题"。

已知信息:
__INFO__

用户问：
__QUERY__

请用中文回答用户问题。
"""


def build_prompt(prompt_template, **kwargs):
    prompt = prompt_template
    for k, v in kwargs.items():
        val = v if isinstance(v, str) else '\n'.join(v) if isinstance(v, list) else str(v)
        prompt = prompt.replace(f"__{k.upper()}__", val)
    return prompt


def get_completion(prompt, model="gpt-4o"):
    messages = [{"role": "user", "content": prompt}]
    response = client_openai.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0,
    )
    return response.choices[0].message.content


class RAG_Bot:
    def __init__(self, llm_api, n_results=5):
        self.llm_api = llm_api
        self.n_results = n_results

    def chat(self, user_query):
        # 获取检索的信息 向量库
        search_results = retrieve_response(user_query, self.n_results)
        # 构建提示词
        prompt = build_prompt(prompt_template, info=search_results, query=user_query)
        print(prompt)
        # 调用LLM 通过向量库检索的信息+用户的输入信息
        response = self.llm_api(prompt)
        return response


user_query = "全身没劲，没精神，吃不下饭，怎么办？"
bot = RAG_Bot(llm_api=get_completion)
response = bot.chat(user_query)
print(response)


```



然后获取到对应的响应结果

```plain
你是一个问答机器人。
你的任务是根据下述给定的已知信息回答用户问题。
确保你的回复完全依据下述已知信息。不要编造答案。
如果下述已知信息不足以回答用户的问题，请直接回复"我无法回答您的问题"。
已知信息:
这种情况可能是皮肤过敏或者荨麻疹，建议您去医院皮肤科进行检查和诊断。在等待就医期间，您可以尝试以下措施缓解症状：
1. 避免摩擦或刺激皮肤，不要穿紧身衣物或使用过于刺激性的洗涤用品。
2. 保持皮肤清洁，定期洗澡并用温水。
3. 涂抹舒缓皮肤的乳液或药膏，如氢化可的松乳膏。
4. 避免暴露在过度干燥或过度潮湿的环境中。
5. 饮食上避免过度刺激的食物，如辛辣食品、海鲜等。
请注意，以上措施仅能缓解症状，如情况严重或症状持续，还是需要及时就医。
根据您的描述，这可能是一个鸡眼，也可能是一个肉瘤，建议您去医院就诊，由专业医生进行诊断。如果是鸡眼，可以使用一些药膏软化鸡眼，再用专业的工具去除。如果是肉瘤，需要进行手术治疗。在就诊之前，避免用手抠挖或切割，以免感染。
根据您的描述，这可能是皮肤受到了创伤或感染，建议您在保持手部清洁的前提下，使用一些抗生素软膏来预防感染。您可以前往药店购买一些非处方药，比如盐酸氯霉素软膏、红霉素软膏等，按照说明使用即可。如果症状加重或持续不缓解，建议您及时就医。
这种情况可能是因为毛囊炎或粉刺引起的。毛囊炎是毛囊感染或炎症，通常会在皮肤上形成红色丘疹或红点。粉刺是一种常见的皮肤问题，通常是因为皮脂和角质堵塞了毛孔。如果这些情况不引起疼痛或瘙痒，通常不需要治疗，只需保持皮肤清洁，避免挤压或摩擦区域。如果出现疼痛、发热、红肿等症状，建议就医咨询。
如果您的中指长了一个透明的水泡状物，但不痛不痒，这可能是一个无害的皮肤病变，例如皮肤囊肿。囊肿通常是由于皮肤下的毛囊或皮脂腺堵塞而形成的，可以出现在任何部位，包括手指。 
建议您去看皮肤科医生，让医生进行诊断和治疗。医生可能会建议将囊肿切开和排出囊内物质，或者开处方药物来治疗。
用户问：
手臂上有小红点不痒不过怎样可以去除
请用中文回答用户问题。
这种情况可能是皮肤过敏或者荨麻疹，建议您去医院皮肤科进行检查和诊断。在等待就医期间，您可以尝试以下措施缓解症状：
1. 避免摩擦或刺激皮肤，不要穿紧身衣物或使用过于刺激性的洗涤用品。
2. 保持皮肤清洁，定期洗澡并用温水。
3. 涂抹舒缓皮肤的乳液或药膏，如氢化可的松乳膏。
4. 避免暴露在过度干燥或过度潮湿的环境中。
5. 饮食上避免过度刺激的食物，如辛辣食品、海鲜等。
请注意，以上措施仅能缓解症状，如情况严重或症状持续，还是需要及时就医。
```





# 五、RAG的缺陷
 **RAG痛点问题分析论文**      

论文:《Seven Failure Points When Engineering a Retrieval Augmented         Generation System》      

        地址:[https://arxiv.org/pdf/2401.05856](https://arxiv.org/pdf/2401.05856)      

        [https://www.163.com/dy/article/JFTQNA200511D3QS.html](https://www.163.com/dy/article/JFTQNA200511D3QS.html)      





## 1. 具体痛点问题总结
Index Process（文本向量化构建索引的过程）:

+ **MIssing Content(内容缺失):** 原本的文本中就没有问题的答案   
+ **文档加载准确性和效率：** 比如pdf文件的加载，如何提取其中的有用文字信息和图片信息等  
+ **文档切分的粒度：** 文本切分的大小和位置会影响后面检索出来的上下文完整性和与大模型交互的token数量，怎么控制好文档切分的度，是个难题。

Query Process（检索增强回答的过程中）:  
+ **Missed Top Ranked:** 错过排名靠前的文档  
+ **Not in Context:**  提取上下文与答案无关  
+ **Wrong Format(格式错误):** 例如需要Json，给了字符串  
+ **Incomplete(答案不完整):** 答案只回答了问题的一部分  
+ **Not Extracted(未提取到答案:)** 提取的上下文中有答案，但大模型没有提取出来  
+ **Incorrect Specificity:** 答案不够具体或过于具体



## 2. 痛点问题策略分析
### 2.1 文档加载准确性和效率
**优化文档读取器**  

	一般知识库中的文档格式都不尽相同，HTML、PDF、MarkDown、TXT、CSV等。每种格式文档都有其都有的数据组织方式。怎么在读取这些数据时将干扰项去除（如一些特殊符号等），同时还保留原文本之间的关联关系（如csv文件保留其原有的表格结构），是主要的优化方向。

	目前针对这方面的探索为：针对每一类文档，设计一个专门的读取器。如LangChain中提供的WebBaseLoader专门用来加载HTML文本等。

	网址:[https://python.langchain.com/v0.1/docs/modules/data_connection/document_loaders/](https://python.langchain.com/v0.1/docs/modules/data_connection/document_loaders/)



**数据清洗与增强**

	输入垃圾，那也必定输出垃圾。如果你的源数据质量低劣，比如包含互相冲突的信息，那不管你的 RAG 工作构建得多么好，它都不可能用你输入的垃圾神奇地输出高质量结果。这个解决方案不仅适用于这个痛点，任何RAG工作流程想要获得优良表现，都必须先清洁数据。



### 2.2 文档切分的粒度
	粒度太大可能导致检索到的文本包含太多不相关的信息，降低检索准确性，粒度太小可能导致信息不全面，导致答案的片面性。问题的答案可能跨越两个甚至多个片段。

**固定长度的分块**

	直接设定块中的字数，每个文本块有多少字。

**内容重叠分块**

	在固定大小分块的基础上，为了保持文本块之间语义上下文的连贯性，在分块时，保持文本块之间有一定的内容重叠。



**基于结构的分块**

	基于结构的分块方法利用文档的固有结构，如HTML或Markdown中的标题和段落，以保持内容的逻辑性和完整性。



**基于递归的分块**

	重复的利用分块规则不断细分文本块。在langchain中会先通过段落换行符（\n\n）进行分割。然后，检查这些块的大小。如果大小不超过一定阈值，则该块被保留。对于大小超过标准的块，使用单换行符（\n）再次分割。以此类推，不断根据块大小更新更小的分块规则（如空格，句号）。



**分块大小的选择**

（1）不同的嵌入模型有其最佳输入大小。比如Openai的text-embedding-ada-002的模型在256 或 512大小的块上效果更好。



（2）文档的类型和用户查询的长度及复杂性也是决定分块大小的重要因素。处理长篇文章或书籍时，较大的分块有助于保留更多的上下文和主题连贯性；而对于社交媒体帖子，较小的分块可能更适合捕捉每个帖子的精确语义。如果用户的查询通常是简短和具体的，较小的分块可能更为合适；相反，如果查询较为复杂，可能需要更大的分块。



### 2.3 内容缺失
	准备的外挂文本中没有回答问题所需的知识。这时候，RAG可能会提供一个自己编造的答案。



**增加相应知识库**

	将相应的知识文本加入到向量知识库中。

**数据清洗与增强**

	输入垃圾，那也必定输出垃圾。如果你的源数据质量低劣，比如包含互相冲突的信息，那不管你的 RAG 工作构建得多么好，它都不可能用你输入的垃圾神奇地输出高质量结果。这个解决方案不仅适用于这个痛点，任何RAG工作流程想要获得优良表现，都必须先清洁数据。



**更好的Prompt设计**

	通过Prompts，让大模型在找不到答案的情况下，输出“根据当前知识库，无法回答该问题”等提示。这样的提示，就能鼓励模型承认自己的局限，并更透明地向用户传达它的不确定。虽然不能保证 100% 准确度，但在清洁数据之后，精心设计 prompt 是最好的做法之一。



### 2.4 错过排名靠前的文档
	外挂知识库中存在回答问题所需的知识，但是可能这个知识块与问题的向量相似度排名并不是靠前的，导致无法召回该知识块传给大模型，导致大模型始终无法得到正确的答案.

**增加召回数量**

	增加召回的 topK 数量，也就是说，例如原来召回前3个知识块，修改为召回前5个知识块。不推荐此种方法，因为知识块多了，不光会增加token消耗，也会增加大模型回答问题的干扰。

**重排（Reranking）**

	该方法的步骤是，首先检索出 topN 个知识块（N > K，过召回），然后再对这 topN 个知识块进行重排序，取重排序后的 K 个知识块当作上下文。重排是利用另一个排序模型或排序策略，对知识块和问题之间进行关系计算与排序。



### 2.5 提取上下文与答案无关
**内容缺失** 或 **错过排名靠前的文档** 的具体体现



### 2.6 格式错误
**Prompt调优**

	优化Prompt逐渐让大模型返回正确的格式。



### 2.7 答案不完整
将问题分开提问:一方面引导用户精简问题，一次只提问一个问题。 另一方面，针对用户的问题进行内部拆分处理，拆分成数个子问题，等子问题答案都找到后，再总结起来回复给用户



### 2.8 未提取到答案
提示压缩技术: 网址:[https://mp.weixin.qq.com/s/61LZgc1a5yRP2J7MTIVZ4Q](https://mp.weixin.qq.com/s/61LZgc1a5yRP2J7MTIVZ4Q)



## 3. RAG评估
### 3.1 RAG效果评估的必要性
+ 评估出RAG对大模型能力改善的程度
+ RAG优化过程，通过评估可以知道改善的方向和参数调整的程度



### 3.2 RAG评估方法
**人工评估**

	最Low的方式是进行人工评估：邀请专家或人工评估员对RAG生成的结果进行评估。他们可以根据预先定义的标准对生成的答案进行质量评估，如准确性、连贯性、相关性等。这种评估方法可以提供高质量的反馈，但可能会消耗大量的时间和人力资源。



**自动化评估**

	自动化评估肯定是RAG评估的主流和发展方向。



**LangSmith**

需要准备测试数据集  
不仅可以评估RAG效果，对于LangChain中的Prompt模板等步骤都可进行测试评估。



**RAGAS**

	RAGAs（Retrieval-Augmented Generation Assessment）是一个评估框架，文档。考虑检索系统识别相关和重点上下文段落的能力，LLM 以忠实方式利用这些段落的能力，以及生成本身的质量。

数据集格式

+ question：作为 RAG 管道输入的用户查询。输入。
+ answer：从 RAG 管道生成的答案。输出。
+ contexts：从用于回答question外部知识源中检索的上下文。
+ ground_truths：question的基本事实答案。这是唯一人工注释的信息。





### 3.3 评估指标
评估检索质量:

+ context_relevancy（上下文相关性，也叫 context_precision）
+ context_recall（召回性，越高表示检索出来的内容与正确答案越相关）

评估生成质量：

+ faithfulness（忠实性，越高表示答案的生成使用了越多的参考文档（检索出来的内容））
+ answer_relevancy（答案的相关性）

Context Recall：上下文召回衡量检索到的上下文(contexts)与标准答案（ground_truths）的匹配程度。      



```python
from datasets import Dataset
from ragas.metrics import context_precision, context_recall, faithfulness, answer_relevancy
from ragas import evaluate
from dotenv import load_dotenv

load_dotenv()
import os


data_samples = {
    'question': ['When was the first super bowl?', 'Who won the most super bowls?'],
    'answer': ['The first superbowl was held on Jan 15, 1967',
               'The most super bowls have been won by The New England Patriots'],
    'contexts': [
        ['The First AFL–NFL World Championship Game was an American football game played on January 15, 1967, at the Los Angeles Memorial Coliseum in Los Angeles,'],
        ['The Green Bay Packers...Green Bay, Wisconsin.', 'The Packers compete...Football Conference']
    ],
    'ground_truth': ['The first superbowl was held on January 15, 1967',
                     'The New England Patriots have won the Super Bowl a record six times']
}
dataset = Dataset.from_dict(data_samples)
# 上下文精确度  评估是否所有和ground_truths（真实答案）相关的chunk被检索到而且排名很靠前
context_precision_score = evaluate(dataset, metrics=[context_precision])
# 计算上下文召回率  上下文召回衡量检索到的上下文(contexts)与标准答案（ground_truths）的匹配程度
context_recall_score = evaluate(dataset, metrics=[context_recall])
# 忠实度  这个指标衡量生成答案与给定上下文之间的事实一致性
faithfulness_score = evaluate(dataset, metrics=[faithfulness])
# 答案相关性  侧重于评估生成的答案与给定提示的相关性。
answer_relevancy_score = evaluate(dataset, metrics=[answer_relevancy])
print(context_precision_score.to_pandas())
print(context_recall_score.to_pandas())
print(faithfulness_score.to_pandas())
print(answer_relevancy_score.to_pandas())
```

输出的答案

```plain
Python 3.9.13 (tags/v3.9.13:6de2ca5, May 17 2022, 16:36:42) [MSC v.1929 64 bit (AMD64)]
Type 'copyright', 'credits' or 'license' for more information
IPython 8.18.1 -- An enhanced Interactive Python. Type '?' for help.
PyDev console: using IPython 8.18.1
Python 3.9.13 (tags/v3.9.13:6de2ca5, May 17 2022, 16:36:42) [MSC v.1929 64 bit (AMD64)] on win32
runfile('E:\\PythonWorkSpace\\LLMProject\\demo4.py', wdir='E:\\PythonWorkSpace\\LLMProject')
Evaluating: 100%|██████████| 2/2 [00:05<00:00,  2.74s/it]
Evaluating: 100%|██████████| 2/2 [00:03<00:00,  1.76s/it]
Evaluating: 100%|██████████| 2/2 [00:07<00:00,  3.55s/it]
Evaluating: 100%|██████████| 2/2 [00:36<00:00, 18.45s/it]
                       user_input  ... context_precision
0  When was the first super bowl?  ...               1.0
1   Who won the most super bowls?  ...               0.0
[2 rows x 5 columns]
                       user_input  ... context_recall
0  When was the first super bowl?  ...            1.0
1   Who won the most super bowls?  ...            0.0
[2 rows x 5 columns]
                       user_input  ... faithfulness
0  When was the first super bowl?  ...          1.0
1   Who won the most super bowls?  ...          0.0
[2 rows x 5 columns]
                       user_input  ... answer_relevancy
0  When was the first super bowl?  ...         0.980736
1   Who won the most super bowls?  ...         0.943014
[2 rows x 5 columns]

```

