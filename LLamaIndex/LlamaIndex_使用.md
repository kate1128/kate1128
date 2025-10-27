在大模型的应用和开发领域，可不是只有LangChain。LlamaIndex是另外一个人气和口碑俱佳的开源Al应用开发框架。Llamalndex开放于2022年11月，和ChatGPT同时诞生，距离LangChain问世也仅有一月之隔。这两个个库都受到ChatGPT的强烈影响和催化，并得到广泛应用和认可。

##  说说Llamalndex
[什么是 LlamaIndex](https://www.yuque.com/qiaokate/su87gb/gfs9x9x5g24446gk)

## RAG 应用与LlamaIndex
<br/>color2
具体来说，Llamalndex包含以下工具。

1. 数据连接器：从数据的原始来源和格式中摄取现有数据。数据可以是API PDF、SQL等各种形态，Llamalndex都有相应的读取接口。
2. 数据索引：将数据结构化为大模型容易理解的中间表示形式，例如词向量
3. 引擎：为数据提供自然语言访问。例如，查询引擎是强大的检索接口，用于增强知识的输出;聊天引擎是用于与数据进行多消息"来回"交互的对话接口;数据Agent则是由大模型驱动的知识工作者，可以实现从简单的辅助到API集成等功能号。
4. 应用集成：将Llamalndex重新集成到其他生态系统中。

<br/>

RAG和Agent这两大Al应用热点密不可分。Llamalndex通过RAG管道、框架和工具，可以为Agent提供更多功能，解决Agent缺乏可控性和透明度的痛点。

Llamalndex中的AgentAPI可以实现逐步执行，使得Agent能够处理更复杂的任务。

此外，Llamalndex还支持用户在RAG循环过程中提供反馈，这种功能特别适用于执行长期任务，实现用户与Agent交互和中间执行控制的双循环设置。

那么，Llamalndex中的Agent究竟是如何利用大模型来实现检索和增强生成的呢?

RAG的实现流程如图所示。

![画板](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1736761726137-c9f510d6-f0d6-4092-a892-e8d65f3f3575.jpeg)

<br/>color2
整个过程共6步。

1. 用户提出查询（Query)：用户向系统提出一个查询，例如一个问题或老一个请求。
2. Agent搜索相关信息：Agent根据用户的查询去搜索相关的信息，可能是通过互联网或者特定的数据库来寻找相关文档或数据。通常我们会把企业内部信息放到向量数据库中。
3. 检索（Retrieval)信息：从搜索结果中检索具体的信息，这些信信息将用于生成响应用户查询的上下文。
4. 相关信息传给大模型：Agent将检索到的信息和用户的原原始查询一起提供给大模型。
5. 大模型生成（Generate)响应：大模型使用这些信息来生成一个丰富的、信息性的答案。
6. 回答用户的请求（Response)：最后，LangChain Agent将大模型生成的答案提供给用户。这个答案是基于用户的原始查询和从相关数据源检索到的信言息生成的。

<br/>

整个过程也可以是一个循环，每次用户的查询都会被用来改进后续的交互。如果需要更多信息或者用户有额外的问题，系统可能会提示用户进行额外的查询，并且可能会利用前一个回答中获得的信息来丰富新的查询上下文。这个系统可以提供更加动态和交互式的用户体验，允许用户以自然语言形式与大模型进行复杂的交互。通过这种方式，大模型可以更精确地理解和回应用户的请求。

通过Llamalndex构造上面的基于RAG的Agent是非常轻松的。而且，值得一提的是，Llamalndex适用于从初学者到高级用户的各个层次的开发者，Llamalndex的高级API允许初学者用5行代码来提取和查询数据。对于更复杂的程序，Llamalndex的底层API允许高级用户根据需要自定义和扩展模块。

## 简单的 LlamaIndex 开发
1. 安装 llamaIndex

```python
pip install LlamaIndex
```

2. 导入环境变量

```python
# 读取系统变量
from dotenv import load_dotenv
load_dotenv()  
```

3. 加载本地数据

```python
from llama_index.core import SimpleDirectoryReader  
documents = SimpleDirectoryReader("03-frameworks\docs").load_data()
```

4. 为数据建立索引

```python
from llama_index import VectorStoreIndex  
index = VectorStoreIndex.from_documents(documents)
```

5. 查询本地数据

先创建查询引擎

```python
agent = index.as_query_engine()
```

再问问题

```python
response = agent.query("花语秘境的员工有几处角色?")
print("花语秘境的员工有几处角色?"， response)
response = agent.query("花语秘境的Agent叫啥名字")
print("花语秘境的Agent叫啥名字"，response)
```

6. 把索引保存到本地

这样就不用每次都加载数据了

```python
index.storage_context.persist()
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736760765216-aef2b60c-96f0-4510-b430-e01759862e61.png)

可以看到，这里新建了一个storage文件夹(可以通过传递persist_dir参数来更改它)，其中包含4个JSON格式的文件，分别如下。

+ `docstore.json`：这可能是一个包含文档存储信息的文件。
+ `graph_store.json`：通常用于存储图形结构数据，可能是关系或数据连接点
+ `index_store.json`：可能是索引信息，用于快速检索存储系统中的为数据
+ `vector_store.json`：可能包含向量数据，这在处理数学运算或者特定的程序功能时是有用的。

可以打开每一个JSON文件，并查看存储格式的细节。

## 完整代码
```python
# 读取系统变量
from dotenv import load_dotenv
load_dotenv()  

from llama_index.core import VectorStoreIndex， SimpleDirectoryReader  
documents = SimpleDirectoryReader("03-frameworks\docs").load_data()

index = VectorStoreIndex.from_documents(documents)

agent = index.as_query_engine()

response = agent.query("花语秘境的员工有几处角色?")
print("花语秘境的员工有几处角色?"， response)
response = agent.query("花语秘境的Agent叫啥名字")
print("花语秘境的Agent叫啥名字"，response)

index.storage_context.persist()
```

