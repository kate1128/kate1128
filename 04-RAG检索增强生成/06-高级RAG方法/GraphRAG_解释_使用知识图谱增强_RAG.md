> 参考路径：
>

[https://medium.com/@zilliz_learn/graphrag-explained-enhancing-rag-with-knowledge-graphs-3312065f99e1#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjYzMzdiZTYzNjRmMzgyNDAwOGQwZTkwMDNmNTBiYjZiNDNkNWE5YzYiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJhenAiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMTEwMTA3NjA3MDEzMDI5OTE5NTQiLCJlbWFpbCI6IjEyOTgzMDM1MTNAcXEuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsIm5iZiI6MTczNzY5ODk4NCwibmFtZSI6ImthdGUgcWlhbyIsInBpY3R1cmUiOiJodHRwczovL2xoMy5nb29nbGV1c2VyY29udGVudC5jb20vYS9BQ2c4b2NKd1l6WDdNNFlVRUgtYVFOa1BJajJNRk5sQ3ZESWJ3VTRFeDRnRGxhSTZHVDVFRlE9czk2LWMiLCJnaXZlbl9uYW1lIjoia2F0ZSIsImZhbWlseV9uYW1lIjoicWlhbyIsImlhdCI6MTczNzY5OTI4NCwiZXhwIjoxNzM3NzAyODg0LCJqdGkiOiI2N2VjNjMxYjI3ODA1NzRmYjMwNzhiZGU4YWRmN2JhOWIwZDk5ZmVkIn0.TKW5q-7AsubaSnoQh2FIOEsNxs3wroSF4EvemmKCdU0n41pxFK6grdbZhepmPGbZi30o0LV5yG5RNaYWFXwQMVttF9n4BdvJSL_PWLmk4R2nCf8kIiNZreulLpNyRaVRxhwr8RUQnsnK55pT0LbgvXu1ztBR4z-55yhgPKbORQf127ZpdD6IN2ytoRkyvFOlZgRm8FYPcju9YwUEm3Ki-8SzTQGGt5FYzq-xtN4wVY75afx2wT0_jiPHNdzZQ4JZmyp4Z39siOLlKeIlIp-kk5nvI9OyFTNkT-0QVEda9dRuUVdJ2CH2Ii_9jiu-8KJ5Rlk5gKXDgTVMoRfP-AQmBA](https://medium.com/@zilliz_learn/graphrag-explained-enhancing-rag-with-knowledge-graphs-3312065f99e1#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjYzMzdiZTYzNjRmMzgyNDAwOGQwZTkwMDNmNTBiYjZiNDNkNWE5YzYiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJhenAiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJhdWQiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJzdWIiOiIxMTEwMTA3NjA3MDEzMDI5OTE5NTQiLCJlbWFpbCI6IjEyOTgzMDM1MTNAcXEuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsIm5iZiI6MTczNzY5ODk4NCwibmFtZSI6ImthdGUgcWlhbyIsInBpY3R1cmUiOiJodHRwczovL2xoMy5nb29nbGV1c2VyY29udGVudC5jb20vYS9BQ2c4b2NKd1l6WDdNNFlVRUgtYVFOa1BJajJNRk5sQ3ZESWJ3VTRFeDRnRGxhSTZHVDVFRlE9czk2LWMiLCJnaXZlbl9uYW1lIjoia2F0ZSIsImZhbWlseV9uYW1lIjoicWlhbyIsImlhdCI6MTczNzY5OTI4NCwiZXhwIjoxNzM3NzAyODg0LCJqdGkiOiI2N2VjNjMxYjI3ODA1NzRmYjMwNzhiZGU4YWRmN2JhOWIwZDk5ZmVkIn0.TKW5q-7AsubaSnoQh2FIOEsNxs3wroSF4EvemmKCdU0n41pxFK6grdbZhepmPGbZi30o0LV5yG5RNaYWFXwQMVttF9n4BdvJSL_PWLmk4R2nCf8kIiNZreulLpNyRaVRxhwr8RUQnsnK55pT0LbgvXu1ztBR4z-55yhgPKbORQf127ZpdD6IN2ytoRkyvFOlZgRm8FYPcju9YwUEm3Ki-8SzTQGGt5FYzq-xtN4wVY75afx2wT0_jiPHNdzZQ4JZmyp4Z39siOLlKeIlIp-kk5nvI9OyFTNkT-0QVEda9dRuUVdJ2CH2Ii_9jiu-8KJ5Rlk5gKXDgTVMoRfP-AQmBA)

# <font style="color:rgb(36, 36, 36);">RAG 简介及其挑战</font>
[<font style="color:rgb(36, 36, 36);">检索增强生成(RAG) 是一种连接外部数据源以增强</font>](https://zilliz.com/learn/Retrieval-Augmented-Generation)[<font style="color:rgb(36, 36, 36);">大型语言模型</font>](https://zilliz.com/glossary/large-language-models-(llms))<font style="color:rgb(36, 36, 36);">(LLM)输出的技术</font><font style="color:rgb(36, 36, 36);">。该技术非常适合 LLM 访问私有或特定领域的数据并解决</font>[<font style="color:rgb(36, 36, 36);">幻觉</font>](https://zilliz.com/glossary/ai-hallucination)<font style="color:rgb(36, 36, 36);">问题。因此，RAG 已被广泛用于支持许多 GenAI 应用程序，例如 AI 聊天机器人和</font>[<font style="color:rgb(36, 36, 36);">推荐系统</font>](https://zilliz.com/vector-database-use-cases/recommender-system)<font style="color:rgb(36, 36, 36);">。</font>

<font style="color:rgb(36, 36, 36);">基线 RAG 通常集成了向量数据库和 LLM，其中</font>[<font style="color:rgb(36, 36, 36);">向量数据库</font>](https://zilliz.com/learn/what-is-vector-database)<font style="color:rgb(36, 36, 36);">存储和检索用户查询的上下文信息，LLM 根据检索到的上下文生成答案。虽然这种方法在许多情况下效果很好，但它在执行多跳推理或回答需要连接不同信息的问题等复杂任务时会遇到困难。</font>

<font style="color:rgb(36, 36, 36);">例如，考虑这个问题：“</font>_<font style="color:rgb(36, 36, 36);">打败篡位者阿莱克托斯的人的儿子叫什么名字？”</font>_

<font style="color:rgb(36, 36, 36);">基线 RAG 通常会遵循以下步骤来回答这个问题：</font>

1. <font style="color:rgb(36, 36, 36);">辨认男子：确定是谁打败了阿莱克图斯。</font>
2. <font style="color:rgb(36, 36, 36);">调查该男子的儿子：查找有关此人的家庭信息，特别是他的儿子。</font>
3. <font style="color:rgb(36, 36, 36);">查找姓名：确定儿子的名字</font>

<font style="color:rgb(36, 36, 36);">挑战通常出现在第一步，因为基线 RAG 根据</font>[<font style="color:rgb(36, 36, 36);">语义相似性</font>](https://zilliz.com/glossary/semantic-similarity)<font style="color:rgb(36, 36, 36);">检索文本，而不是直接回答复杂查询，因为数据集中可能没有明确提及具体细节。这种限制使得很难找到所需的确切信息，通常需要昂贵且不切实际的解决方案，例如手动为频繁查询创建问答对。</font>

<font style="color:rgb(36, 36, 36);">为了应对这些挑战，微软研究院推出了</font>[<font style="color:rgb(36, 36, 36);">GraphRAG</font>](https://microsoft.github.io/graphrag/)<font style="color:rgb(36, 36, 36);">，这是一种全新的方法，它使用知识图谱增强了 RAG 的检索和生成。在以下部分中，我们将解释 GraphRAG 的底层工作原理以及如何将其与</font>[<font style="color:rgb(36, 36, 36);">Milvus</font>](https://milvus.io/intro)<font style="color:rgb(36, 36, 36);">矢量数据库一起运行。</font>

# <font style="color:rgb(36, 36, 36);">GraphRAG 是什么以及它如何工作？</font>
<font style="color:rgb(36, 36, 36);">与使用向量数据库检索语义相似文本的基线 RAG 不同，GraphRAG 通过整合知识图谱 (KG) 来增强 RAG。知识图谱是一种基于关系存储和链接相关或不相关数据的数据结构。</font>

<br/>color2
<font style="color:rgb(36, 36, 36);">GraphRAG 管道通常由两个基本过程组成：索引和查询。</font>

<br/>

![GraphRAG 管道（图片来源： GraphRAG 论文）](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737699550986-999a8b06-f235-4eeb-9d0f-ea5a83afe7bc.png)

## <font style="color:rgb(36, 36, 36);">索引</font>
<font style="color:rgb(36, 36, 36);">索引过程包括四个关键步骤：</font>

1. <font style="color:rgb(36, 36, 36);">文本单元分割：将整个输入语料库划分为多个文本单元（文本块）。这些块是最小的可分析单元，可以是段落、句子或其他逻辑单元。通过将长文档分割成较小的块，我们可以提取并保留有关此输入数据的更详细信息。</font>
2. <font style="color:rgb(36, 36, 36);">实体、关系和声明提取：GraphRAG 使用 LLM 识别和提取每个文本单元中的所有实体（人名、地点、组织等）、它们之间的关系以及文本中表达的关键声明。我们将使用这些提取的信息构建初始知识图谱。</font>
3. <font style="color:rgb(36, 36, 36);">层次聚类：GraphRAG 使用</font>[<font style="color:rgb(36, 36, 36);">Leiden</font>](https://arxiv.org/pdf/1810.08473)<font style="color:rgb(36, 36, 36);">技术对初始知识图谱进行层次聚类。Leiden 是一种社区检测算法，可以有效地发现图谱中的社区结构。每个聚类中的实体被分配到不同的社区，以便进行更深入的分析。</font>

_<font style="color:rgb(36, 36, 36);">注意：</font>_<font style="color:rgb(36, 36, 36);"> </font>_<font style="color:rgb(36, 36, 36);">社区是图中一组节点，这些节点彼此紧密连接，但与网络中其他密集组的连接稀疏。</font>_

<font style="color:rgb(36, 36, 36);">社区摘要生成：</font><font style="background-color:rgb(232, 243, 232);">GraphRAG 采用自下而上的方法为每个社区及其成员生成摘要。</font><font style="color:rgb(36, 36, 36);">这些摘要包括社区内的主要实体、它们之间的关系以及关键主张。此步骤概述了整个数据集，并为后续查询提供了有用的上下文信息。</font>

![图 1：使用 GPT-4 Turbo 构建的 LLM 生成的知识图。 （图片来源： 微软研究院）](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737699552687-74057a9c-977c-4813-b58b-d0c121ca1211.png)



## <font style="color:rgb(36, 36, 36);">查询</font>
<font style="color:rgb(36, 36, 36);">GraphRAG 有两种针对不同查询定制的查询工作流程。</font>

+ <font style="color:rgb(36, 36, 36);">利用社区摘要进行</font>[<font style="color:rgb(36, 36, 36);">全局搜索，推理与整个数据语料库相关的整体问题。</font>](https://microsoft.github.io/graphrag/posts/query/0-global_search)
+ [<font style="color:rgb(36, 36, 36);">本地搜索</font>](https://microsoft.github.io/graphrag/posts/query/1-local_search)<font style="color:rgb(36, 36, 36);">通过展开到邻居和相关概念来推理特定实体。</font>

<font style="color:rgb(36, 36, 36);">此全局搜索工作流程包括以下阶段。</font>

![图 2：全局搜索数据流（图片来源： 微软研究院）](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737699552564-6dc1fdd1-707d-4fc5-9ddb-4193131cb04c.png)

1. <font style="color:rgb(36, 36, 36);">用户查询和对话历史：系统将用户查询和对话历史作为初始输入。</font>
2. <font style="color:rgb(36, 36, 36);">社区报告批次：系统使用 LLM 从社区层级的指定级别生成的节点社区报告作为上下文数据。这些社区报告被打乱并分为多个批次（打乱后的社区报告批次 1、批次 2……批次 N）。</font>
3. <font style="color:rgb(36, 36, 36);">RIR（评级中间响应）：每一批社区报告进一步划分为预定义大小的文本块。每个文本块用于生成中间响应。响应包含一串称为点的信息。每个点都有一个数字分数，表示其重要性。这些生成的中间响应是评级中间响应（评级中间响应 1、响应 2……响应 N）。</font>
4. <font style="color:rgb(36, 36, 36);">排序和过滤：系统对这些中间回答进行排序和过滤，选出最重要的点。选出的重要点形成了汇总的中间回答。</font>
5. <font style="color:rgb(36, 36, 36);">最终响应：聚合的中间响应作为上下文来生成最终回复。</font>

<font style="color:rgb(36, 36, 36);">当用户询问有关特定实体（例如人名、地点、组织等）的问题时，我们建议您使用本地搜索工作流程。此流程包括以下步骤：</font>

![图 3：本地搜索数据流（图片来源： 微软研究院）](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737699553771-4cf6b654-300b-4916-b378-3b99e8d21b86.png)

1. <font style="color:rgb(36, 36, 36);">用户查询：首先，系统接收用户查询，这可能是一个简单的问题，也可能是一个更复杂的查询。</font>
2. <font style="color:rgb(36, 36, 36);">相似实体搜索：系统从知识图谱中识别出一组与用户输入在语义上相关的实体。这些实体作为知识图谱的入口点。此步骤使用 Milvus 等矢量数据库进行</font>[<font style="color:rgb(36, 36, 36);">文本相似性搜索</font>](https://zilliz.com/learn/vector-similarity-search)<font style="color:rgb(36, 36, 36);">。</font>
3. <font style="color:rgb(36, 36, 36);">实体-文本单元映射：将提取的文本单元映射到相应的实体，删除原始文本信息。</font>
4. <font style="color:rgb(36, 36, 36);">实体关系提取：此步骤提取有关实体及其对应关系的具体信息。</font>
5. <font style="color:rgb(36, 36, 36);">实体-协变量映射：此步骤将实体映射到其协变量，可能包括统计数据或其他相关属性。</font>
6. <font style="color:rgb(36, 36, 36);">实体社区报告映射：社区报告集成到搜索结果中，包含一些全球信息。</font>
7. <font style="color:rgb(36, 36, 36);">对话历史记录的利用：如果提供，系统将使用对话历史记录来更好地了解用户的意图和背景。</font>
8. <font style="color:rgb(36, 36, 36);">响应生成：最后，系统根据前面步骤生成的过滤和排序的数据构建并响应用户查询。</font>

# <font style="color:rgb(36, 36, 36);">基线 RAG 与 GraphRAG 的输出质量</font>
<font style="color:rgb(36, 36, 36);">为了展示 GraphRAG 的有效性，其创建者在他们的</font>[<font style="color:rgb(36, 36, 36);">公告博客</font>](https://www.microsoft.com/en-us/research/blog/graphrag-unlocking-llm-discovery-on-narrative-private-data/)<font style="color:rgb(36, 36, 36);">中比较了基线 RAG 和 GraphRAG 的输出质量。我在这里举一个简单的例子来说明。</font>

## <font style="color:rgb(36, 36, 36);">使用的数据集</font>
<font style="color:rgb(36, 36, 36);">GraphRAG 的创建者使用新闻文章中的暴力事件信息 (</font><font style="color:rgb(36, 36, 36);"> </font>[<font style="color:rgb(36, 36, 36);">VIINA</font>](https://github.com/zhukovyuri/VIINA)<font style="color:rgb(36, 36, 36);"> </font><font style="color:rgb(36, 36, 36);">) 数据集进行实验。</font>

_<font style="color:rgb(36, 36, 36);">注意：</font>_<font style="color:rgb(36, 36, 36);"> </font>_<font style="color:rgb(36, 36, 36);">此数据集包含敏感主题。之所以选择它，完全是因为它的复杂性以及存在不同意见和部分信息。这是一个混乱的真实世界测试案例，而且时间还不长，因此不包含在 LLM 基础模型的训练中。</font>_

## <font style="color:rgb(36, 36, 36);">实验概述</font>
<font style="color:rgb(36, 36, 36);">Baseline RAG 和 GraphRAG 都被问到同样的问题，即需要汇总整个数据集的信息来得出答案。</font>

<font style="color:rgb(36, 36, 36);">问：数据集中的前 5 个主题是什么？</font>

<font style="color:rgb(36, 36, 36);">答案如下图所示。Baseline RAG 的结果与战争主题无关，因为矢量搜索检索到不相关的文本，导致评估不准确。相比之下，GraphRAG 提供了清晰且相关的答案，确定了主要主题和支持细节。结果与数据集一致，并引用了源材料。</font>

![图 4：BaselineRAG 与 GraphRAG 在回答复杂摘要问题方面的表现](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737699555258-6cb06a89-e171-4213-ba13-d2c454645c74.png)

<font style="color:rgb(36, 36, 36);">论文《</font>[<font style="color:rgb(36, 36, 36);">从局部到全局：基于 Graph RAG 的查询摘要方法》</font>](https://arxiv.org/pdf/2404.16130)<font style="color:rgb(36, 36, 36);">中的进一步实验表明，GraphRAG 显著提升了多跳推理和复杂信息摘要能力。研究表明，GraphRAG 在全面性和多样性上都超越了 Baseline RAG：</font>

+ <font style="color:rgb(36, 36, 36);">全面性：答案涵盖问题各个方面的程度。</font>
+ <font style="color:rgb(36, 36, 36);">多样性：答案提供的观点和见解的多样性和丰富性。</font>

<font style="color:rgb(36, 36, 36);">我们建议您阅读原始 GraphRAG 论文以了解有关这些实验的更多详细信息。</font>

# <font style="color:rgb(36, 36, 36);">如何使用 Milvus 矢量数据库实现 GraphRAG</font>
<font style="color:rgb(36, 36, 36);">GraphRAG 通过知识图谱增强了 RAG 应用程序，并且还依赖向量数据库来检索相关实体。本节演示如何实现 GraphRAG、创建 GraphRAG 索引以及使用</font>[<font style="color:rgb(36, 36, 36);">Milvus</font>](https://milvus.io/)<font style="color:rgb(36, 36, 36);">向量数据库对其进行查询。</font>

## <font style="color:rgb(36, 36, 36);">先决条件</font>
<font style="color:rgb(36, 36, 36);">在运行本博客的代码之前，请确保已经安装了以下依赖项：</font>

```python
pip 安装 --升级 pymilvus 
pip 安装 git+https://github.com/zc277584121/graphrag.git
```

_<font style="color:rgb(36, 36, 36);">注意：</font>_<font style="color:rgb(36, 36, 36);"> </font>_<font style="color:rgb(36, 36, 36);">我们从分叉存储库安装了 GraphRAG，因为在撰写本文时，Milvus 存储功能仍待官方合并。</font>_

<font style="color:rgb(36, 36, 36);">让我们从索引工作流程开始。</font>

## <font style="color:rgb(36, 36, 36);">数据准备</font>
<font style="color:rgb(36, 36, 36);">Download a small </font>[text file](https://www.gutenberg.org/cache/epub/7785/pg7785.txt)<font style="color:rgb(36, 36, 36);"> with about a thousand lines from </font>[Project Gutenberg](https://www.gutenberg.org/)<font style="color:rgb(36, 36, 36);"> and use it for GraphRAG indexing.</font>

<font style="color:rgb(36, 36, 36);">这个数据集是关于列奥纳多·达芬奇的故事。我们使用 GraphRAG 构建了与达芬奇有关的所有关系的图索引，并使用 Milvus 向量数据库来搜索相关知识来回答问题。</font>

```python
import nest_asyncio
nest_asyncio.apply()
```

```python
import os
import urllib.request
```

```python
index_root = os.path.join(os.getcwd(), 'graphrag_index')
os.makedirs(os.path.join(index_root, 'input'), exist_ok=True)
url = "https://www.gutenberg.org/cache/epub/7785/pg7785.txt"
file_path = os.path.join(index_root, 'input', 'davinci.txt')
urllib.request.urlretrieve(url, file_path)
with open(file_path, 'r+', encoding='utf-8') as file:
    # We use the first 934 lines of the text file, because the later lines are not relevant for this example.
    # If you want to save api key cost, you can truncate the text file to a smaller size.
    lines = file.readlines()
    file.seek(0)
    file.writelines(lines[:934])  # Decrease this number if you want to save api key cost.
    file.truncate()
```

## <font style="color:rgb(36, 36, 36);">初始化工作区</font>
<font style="color:rgb(36, 36, 36);">现在，让我们使用 GraphRAG 来索引文本文件。要初始化您的工作区，让我们首先运行该</font>`<font style="color:rgb(36, 36, 36);background-color:rgb(242, 242, 242);">graphrag.index --init</font>`<font style="color:rgb(36, 36, 36);">命令。</font>

```python
python -m graphrag.index  --init  --root./graphrag_index
```

## <font style="color:rgb(36, 36, 36);">配置</font>`<font style="color:rgb(36, 36, 36);background-color:rgb(242, 242, 242);">env</font>`<font style="color:rgb(36, 36, 36);">文件和设置</font>
`<font style="color:rgb(36, 36, 36);background-color:rgb(242, 242, 242);">.env</font>`<font style="color:rgb(36, 36, 36);">您将在索引的根目录中</font><font style="color:rgb(36, 36, 36);">找到该文件。要使用它，请将您的 OpenAI API 密钥添加到该</font>`<font style="color:rgb(36, 36, 36);background-color:rgb(242, 242, 242);">.env</font>`<font style="color:rgb(36, 36, 36);">文件。</font>

_<font style="color:rgb(36, 36, 36);">重要说明：</font>_<font style="color:rgb(36, 36, 36);"> </font><font style="color:rgb(36, 36, 36);">__</font>

+ _<font style="color:rgb(36, 36, 36);">我们将使用 OpenAI 模型作为此示例；确保您已准备好</font>_<font style="color:rgb(36, 36, 36);"> </font>[<font style="color:rgb(36, 36, 36);">API 密钥</font>](https://platform.openai.com/docs/quickstart)<font style="color:rgb(36, 36, 36);"> </font>_<font style="color:rgb(36, 36, 36);">。</font>_
+ _<font style="color:rgb(36, 36, 36);">GraphRAG 索引成本高昂，因为它使用 LLM 处理整个文本语料库。运行此演示可能需要花费几美元。为了省钱，请考虑将文本文件截断为较小的尺寸。</font>_

## <font style="color:rgb(36, 36, 36);">运行索引管道</font>
<font style="color:rgb(36, 36, 36);">索引过程需要一些时间。完成后，您会在</font>`<font style="color:rgb(36, 36, 36);background-color:rgb(242, 242, 242);">./graphrag_index/output/<timestamp>/</font>`<font style="color:rgb(36, 36, 36);">artifacts中找到一个新文件夹，其中包含一系列parquet文件。</font>

```python
python -m graphrag.index --root ./graphrag_index
```

## <font style="color:rgb(36, 36, 36);">使用 Milvus 矢量数据库进行查询</font>
<font style="color:rgb(36, 36, 36);">在查询阶段，我们使用 Milvus 存储实体描述嵌入，用于 GraphRAG</font>[<font style="color:rgb(36, 36, 36);">本地搜索</font>](https://microsoft.github.io/graphrag/posts/query/1-local_search/)<font style="color:rgb(36, 36, 36);">。此方法将知识图谱中的结构化数据与输入文档中的非结构化数据相结合，使用相关实体信息增强 LLM 上下文，从而获得更精确的答案。</font>

```python
import os
```

```python
import pandas as pd
import tiktoken
from graphrag.query.context_builder.entity_extraction import EntityVectorStoreKey
from graphrag.query.indexer_adapters import (
    # read_indexer_covariates,
    read_indexer_entities,
    read_indexer_relationships,
    read_indexer_reports,
    read_indexer_text_units,
)
from graphrag.query.input.loaders.dfs import (
    store_entity_semantic_embeddings,
)
from graphrag.query.llm.oai.chat_openai import ChatOpenAI
from graphrag.query.llm.oai.embedding import OpenAIEmbedding
from graphrag.query.llm.oai.typing import OpenaiApiType
from graphrag.query.question_gen.local_gen import LocalQuestionGen
from graphrag.query.structured_search.local_search.mixed_context import (
    LocalSearchMixedContext,
)
from graphrag.query.structured_search.local_search.search import LocalSearch
from graphrag.vector_stores import MilvusVectorStore
```

```python
output_dir = os.path.join(index_root, "output")
subdirs = [os.path.join(output_dir, d) for d in os.listdir(output_dir)]
latest_subdir = max(subdirs, key=os.path.getmtime)  # Get latest output directory
INPUT_DIR = os.path.join(latest_subdir, "artifacts")
```

```python
COMMUNITY_REPORT_TABLE = "create_final_community_reports"
ENTITY_TABLE = "create_final_nodes"
ENTITY_EMBEDDING_TABLE = "create_final_entities"
RELATIONSHIP_TABLE = "create_final_relationships"
COVARIATE_TABLE = "create_final_covariates"
TEXT_UNIT_TABLE = "create_final_text_units"
COMMUNITY_LEVEL = 2
```

## <font style="color:rgb(36, 36, 36);">从索引过程中加载数据</font>
<font style="color:rgb(36, 36, 36);">在索引过程中，会生成一些 parquet 文件，我们将其加载到内存中，并将实体描述信息存入 Milvus 向量数据库中。</font>

<font style="color:rgb(36, 36, 36);">读取实体：</font>

```python
# read nodes table to get community and degree data
entity_df = pd.read_parquet(f"{INPUT_DIR}/{ENTITY_TABLE}.parquet")
entity_embedding_df = pd.read_parquet(f"{INPUT_DIR}/{ENTITY_EMBEDDING_TABLE}.parquet")
```

```python
entities = read_indexer_entities(entity_df, entity_embedding_df, COMMUNITY_LEVEL)
description_embedding_store = MilvusVectorStore(
    collection_name="entity_description_embeddings",
)
# description_embedding_store.connect(uri="http://localhost:19530") # For Milvus docker service
description_embedding_store.connect(uri="./milvus.db") # For Milvus Lite
entity_description_embeddings = store_entity_semantic_embeddings(
    entities=entities, vectorstore=description_embedding_store
)
print(f"Entity count: {len(entity_df)}")
entity_df.head()
```

<font style="color:rgb(36, 36, 36);">实体数：651</font>![图 5：实体的屏幕截图](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737699555285-5c68b2f5-02c1-440a-8041-d856846685c0.png)

<font style="color:rgb(36, 36, 36);">读取关系</font>

```python
relationship_df = pd.read_parquet(f"{INPUT_DIR}/{RELATIONSHIP_TABLE}.parquet")
relationships = read_indexer_relationships(relationship_df)
```

```python
print(f"Relationship count: {len(relationship_df)}")
relationship_df.head()
```

<font style="color:rgb(36, 36, 36);">关系数：290</font>

![图 6：关系截图](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737699556511-10a8c62c-2320-4175-9119-becd30d5c98d.png)

<font style="color:rgb(36, 36, 36);">阅读社区报告</font>

```python
report_df = pd.read_parquet(f"{INPUT_DIR}/{COMMUNITY_REPORT_TABLE}.parquet")
reports = read_indexer_reports(report_df, entity_df, COMMUNITY_LEVEL)
```

```python
print(f"Report records: {len(report_df)}")
report_df.head()
```

<font style="color:rgb(36, 36, 36);">报告记录：45</font>

![图7：报告记录截图](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737699557001-613c09e1-320d-4b96-89a2-86bde87c4c3d.png)

<font style="color:rgb(36, 36, 36);">阅读文本单元</font>

```python
text_unit_df = pd.read_parquet(f"{INPUT_DIR}/{TEXT_UNIT_TABLE}.parquet")
text_units = read_indexer_text_units(text_unit_df)
```

```python
print(f"Text unit records: {len(text_unit_df)}")
text_unit_df.head()
```

<font style="color:rgb(36, 36, 36);">文本单元记录：51</font>

![图 8：文本单元记录的屏幕截图](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737699565668-a90aaf57-5668-4b25-917d-e7636c39e5ac.png)

## <font style="color:rgb(36, 36, 36);">创建本地搜索引擎</font>
<font style="color:rgb(36, 36, 36);">我们已经准备好了本地搜索引擎所需的数据。现在，我们可以</font>`<font style="color:rgb(36, 36, 36);background-color:rgb(242, 242, 242);">LocalSearch</font>`<font style="color:rgb(36, 36, 36);">用它们、LLM 和嵌入模型来构建一个实例。</font>

```python
api_key = os.environ["OPENAI_API_KEY"]  # Your OpenAI API key
llm_model = "gpt-4o"  # Or gpt-4-turbo-preview
embedding_model = "text-embedding-3-small"
```

```python
llm = ChatOpenAI(
    api_key=api_key,
    model=llm_model,
    api_type=OpenaiApiType.OpenAI,
    max_retries=20,
)
token_encoder = tiktoken.get_encoding("cl100k_base")
text_embedder = OpenAIEmbedding(
    api_key=api_key,
    api_base=None,
    api_type=OpenaiApiType.OpenAI,
    model=embedding_model,
    deployment_name=embedding_model,
    max_retries=20,
)
```

```python
context_builder = LocalSearchMixedContext(
    community_reports=reports,
    text_units=text_units,
    entities=entities,
    relationships=relationships,
    covariates=None, #covariates,#todo
    entity_text_embeddings=description_embedding_store,
    embedding_vectorstore_key=EntityVectorStoreKey.ID,  # if the vectorstore uses entity title as ids, set this to EntityVectorStoreKey.TITLE
    text_embedder=text_embedder,
    token_encoder=token_encoder,
)
```

```python
local_context_params = {
    "text_unit_prop": 0.5,
    "community_prop": 0.1,
    "conversation_history_max_turns": 5,
    "conversation_history_user_turns_only": True,
    "top_k_mapped_entities": 10,
    "top_k_relationships": 10,
    "include_entity_rank": True,
    "include_relationship_weight": True,
    "include_community_rank": False,
    "return_candidate_context": False,
    "embedding_vectorstore_key": EntityVectorStoreKey.ID,  # set this to EntityVectorStoreKey.TITLE if the vectorstore uses entity title as ids
    "max_tokens": 12_000,  # change this based on the token limit you have on your model (if you are using a model with 8k limit, a good setting could be 5000)
}
```

```python
llm_params = {
    "max_tokens": 2_000,  # change this based on the token limit you have on your model (if you are using a model with 8k limit, a good setting could be 1000=1500)
    "temperature": 0.0,
}
```

```python
search_engine = LocalSearch(
    llm=llm,
    context_builder=context_builder,
    token_encoder=token_encoder,
    llm_params=llm_params,
    context_builder_params=local_context_params,
    response_type="multiple paragraphs",  # free form text describing the response type and format, can be anything, e.g. prioritized list, single paragraph, multiple paragraphs, multiple-page report
)
```

## <font style="color:rgb(36, 36, 36);">进行查询</font>
```python
result = await search_engine.asearch("Tell me about Leonardo Da Vinci")
print(result.response)
```

```python
# Leonardo da Vinci
    Leonardo da Vinci, born in 1452 in the town of Vinci near Florence, is widely celebrated as one of the most versatile geniuses of the Italian Renaissance. His full name was Leonardo di Ser Piero d'Antonio di Ser Piero di Ser Guido da Vinci, and he was the natural and first-born son of Ser Piero, a country notary [Data: Entities (0)]. Leonardo's contributions spanned various fields, including art, science, engineering, and philosophy, earning him the title of the most Universal Genius of Christian times [Data: Entities (8)].
    ## Early Life and Training
    Leonardo's early promise was recognized by his father, who took some of his drawings to Andrea del Verrocchio, a renowned artist and sculptor. Impressed by Leonardo's talent, Verrocchio accepted him into his workshop around 1469-1470. Here, Leonardo met other notable artists, including Botticelli and Lorenzo di Credi [Data: Sources (6, 7)]. By 1472, Leonardo was admitted into the Guild of Florentine Painters, marking the beginning of his professional career [Data: Sources (7)].
    ## Artistic Masterpieces
    Leonardo is perhaps best known for his iconic paintings, such as the "Mona Lisa" and "The Last Supper." The "Mona Lisa," renowned for its subtle expression and detailed background, is housed in the Louvre and remains one of the most famous artworks in the world [Data: Relationships (0, 45)]. "The Last Supper," a fresco depicting the moment Jesus announced that one of his disciples would betray him, is located in the refectory of Santa Maria delle Grazie in Milan [Data: Sources (2)]. Other significant works include "The Virgin of the Rocks" and the "Treatise on Painting," which he began around 1489-1490 [Data: Relationships (7, 12)].
    ## Scientific and Engineering Contributions
    Leonardo's genius extended beyond art to various scientific and engineering endeavors. He made significant observations in anatomy, optics, and hydraulics, and his notebooks are filled with sketches and ideas that anticipated many modern inventions. For instance, he anticipated Copernicus' theory of the earth's movement and Lamarck's classification of animals [Data: Relationships (38, 39)]. His work on the laws of light and shade and his mastery of chiaroscuro had a profound impact on both art and science [Data: Sources (45)].
    ## Patronage and Professional Relationships
    Leonardo's career was significantly influenced by his patrons. Ludovico Sforza, the Duke of Milan, employed Leonardo as a court painter and general artificer, commissioning various works and even gifting him a vineyard in 1499 [Data: Relationships (9, 19, 84)]. In his later years, Leonardo moved to France under the patronage of King Francis I, who provided him with a princely income and held him in high regard [Data: Relationships (114, 37)]. Leonardo spent his final years at the Manor House of Cloux near Amboise, where he was frequently visited by the King and supported by his close friend and assistant, Francesco Melzi [Data: Relationships (28, 122)].
    ## Legacy and Influence
    Leonardo da Vinci's influence extended far beyond his lifetime. He founded a School of painting in Milan, and his techniques and teachings were carried forward by his students and followers, such as Giovanni Ambrogio da Predis and Francesco Melzi [Data: Relationships (6, 15, 28)]. His works continue to be celebrated and studied, cementing his legacy as one of the greatest masters of the Renaissance. Leonardo's ability to blend art and science has left an indelible mark on both fields, inspiring countless generations of artists and scientists [Data: Entities (148, 86); Relationships (27, 12)].
    In summary, Leonardo da Vinci's unparalleled contributions to art, science, and engineering, combined with his innovative thinking and profound influence on his contemporaries and future generations, make him a towering figure in the history of human achievement. His legacy continues to inspire admiration and study, underscoring the timeless relevance of his genius.
```

<font style="color:rgb(36, 36, 36);">GraphRAG 的结果非常具体，引用的数据源标记清晰。</font>

## <font style="color:rgb(36, 36, 36);">问题生成</font>
<font style="color:rgb(36, 36, 36);">GraphRAG 还可以根据历史查询生成问题，这对于在聊天机器人对话中创建推荐问题非常有用。此方法将知识图谱中的结构化数据与输入文档中的非结构化数据相结合，以生成与特定实体相关的候选问题。</font>

```python
question_generator = LocalQuestionGen(
   llm=llm,
   context_builder=context_builder,
   token_encoder=token_encoder,
   llm_params=llm_params,
   context_builder_params=local_context_params,
)
```

```python
question_history = [
    "Tell me about Leonardo Da Vinci",
    "Leonardo's early works",
]
```

<font style="color:rgb(36, 36, 36);">根据历史提出问题。</font>

```python
candidate_questions = await question_generator.agenerate(
        question_history=question_history, context_data=None, question_count=5
    )
candidate_questions.response
```

```python
["- What were some of Leonardo da Vinci's early works and where are they housed?",
     "- How did Leonardo da Vinci's relationship with Andrea del Verrocchio influence his early works?",
     '- What notable projects did Leonardo da Vinci undertake during his time in Milan?',
     "- How did Leonardo da Vinci's engineering skills contribute to his projects?",
     "- What was the significance of Leonardo da Vinci's relationship with Francis I of France?"]
```

<font style="color:rgb(36, 36, 36);">如果您想删除索引以节省空间，可以删除索引根。</font>

```python
# import shutil
#
# shutil.rmtree(index_root)
```

## <font style="color:rgb(36, 36, 36);">概括</font>
<font style="color:rgb(36, 36, 36);">在这篇博客中，我们探讨了 GraphRAG，这是一种通过集成知识图谱来增强 RAG 技术的创新方法。GraphRAG 非常适合处理复杂任务，例如多跳推理和回答需要链接不同信息的综合问题。</font>

<font style="color:rgb(36, 36, 36);">与</font>[<font style="color:rgb(36, 36, 36);">Milvus矢量数据库结合使用时，GraphRAG 可以处理大型数据集中复杂的语义关系，从而提供更准确、更有洞察力的结果。这种强大的组合使 GraphRAG 成为各种实际</font>](https://zilliz.com/what-is-milvus)[<font style="color:rgb(36, 36, 36);">GenAI</font>](https://zilliz.com/learn/generative-ai)<font style="color:rgb(36, 36, 36);">应用的宝贵资产</font><font style="color:rgb(36, 36, 36);">，为理解和处理复杂信息提供了强大的解决方案。</font>

# <font style="color:rgb(36, 36, 36);">更多资源</font>
+ <font style="color:rgb(36, 36, 36);">GraphRAG 论文：</font>[<font style="color:rgb(36, 36, 36);">从局部到全局：基于查询的摘要的 Graph RAG 方法</font>](https://arxiv.org/pdf/2404.16130)
+ <font style="color:rgb(36, 36, 36);">GraphRAG GitHub：</font>[<font style="color:rgb(36, 36, 36);">https://github.com/microsoft/graphrag</font>](https://github.com/microsoft/graphrag)
+ <font style="color:rgb(36, 36, 36);">其他 RAG 增强技术：</font>
+ [<font style="color:rgb(36, 36, 36);">使用 HyDE 实现更好的 RAG — 假设文档嵌入</font>](https://zilliz.com/learn/improve-rag-and-information-retrieval-with-hyde-hypothetical-document-embeddings)
+ [<font style="color:rgb(36, 36, 36);">使用 WhyHow 通过知识图谱增强 RAG</font>](https://zilliz.com/blog/enhance-rag-with-knowledge-graphs)
+ [<font style="color:rgb(36, 36, 36);">如何增强 RAG 管道的性能</font>](https://zilliz.com/learn/how-to-enhance-the-performance-of-your-rag-pipeline)
+ [<font style="color:rgb(36, 36, 36);">使用重排器优化 RAG：作用与权衡</font>](https://zilliz.com/learn/optimize-rag-with-rerankers-the-role-and-tradeoffs)
+ [<font style="color:rgb(36, 36, 36);">面向开发人员构建 RAG 应用程序的实用技巧和窍门</font>](https://zilliz.com/blog/praticial-tips-and-tricks-for-developers-building-rag-applications)
+ [<font style="color:rgb(36, 36, 36);">使用自定义 AI 模型扩展 RAG 时面临的基础设施挑战</font>](https://zilliz.com/blog/infrastructure-challenges-in-scaling-rag-with-custom-ai-models)

