> `llama_index` 提供了多个组件来帮助你实现以下功能：
>
> + 加载和解析文档（如 `SimpleDirectoryReader`、`Document`）。
> + 构建和管理文档索引（如 `GPTSimpleVectorIndex`、`VectorStore`）。
> + 配置和管理与大型语言模型（如 GPT）交互的上下文（如 `ServiceContext`、`LLMPredictor`）。
> + 进行文档查询和相似性搜索（如 `QueryEngine`、`Embedding`）。
>
> 根据具体的应用需求，你可以选择不同的组件来处理文档加载、嵌入、索引构建、查询等任务。
>

`llama_index`（现在通常称为 **GPT Index**）是一个用于构建和操作文档索引的库。它的目的是将文档与自然语言处理（NLP）模型集成，支持快速高效的查询和分析。它包含了多个组件和功能模块，主要用于文档加载、索引构建、查询和数据处理等。

+ [https://github.com/run-llama/llama_index](https://github.com/run-llama/llama_index)

以下是 `llama_index` 库中一些常见的组件和功能：

## 加载文档：
### `SimpleDirectoryReader`
+ `SimpleDirectoryReader` 是一个非常简单的文件读取器，用于从文件夹中加载文档，支持多种文件格式（如文本文件、PDF、Word 文档等）。
+ 它可以遍历目录中的文件，读取文件内容，并将它们加载为文档对象（`Document`）。
+ 示例：

```python
from llama_index import SimpleDirectoryReader
documents = SimpleDirectoryReader(input_files=["path/to/file1.pdf"]).load_data()
```

### `Document`
+ `Document` 是核心的组件之一，用于表示和封装单个文档的内容。
+ 文档通常包含文本数据，以及一些元数据（如标题、来源等）。
+ 示例：  

```python
from llama_index import Document

text_list = [text1, text2, ...]
documents = [Document(t) for t in text_list]
```

## 将文档解析为 Node
<font style="color:rgb(25, 27, 31);">Node以数据 Chunks 的形式呈现文档，同时 Node 保留与其他 Node 和 索引结构 的关系。</font>

<font style="color:rgb(25, 27, 31);">直接解析文档</font>

```python
from llama_index.node_parser import SimpleNodeParser

parser = SimpleNodeParser()

nodes = parser.get_nodes_from_documents(documents)
```

也可以选择手动构造Node对象

```python
from llama_index.data_structs.node import Node, DocumentRelationship

node1 = Node(text="<text_chunk>", doc_id="<node_id>")
node2 = Node(text="<text_chunk>", doc_id="<node_id>")
# set relationships
node1.relationships[DocumentRelationship.NEXT] = node2.get_doc_id()
node2.relationships[DocumentRelationship.PREVIOUS] = node1.get_doc_id()
nodes = [node1, node2]
```

## Index 构建
### `GPTSimpleVectorIndex`
+ `GPTSimpleVectorIndex` 是一个简单的索引结构，用于存储和管理文档的向量表示（通常是文档的嵌入向量）。
+ 它将文档与 OpenAI GPT 模型或其他嵌入模型结合，以便可以高效地进行相似性搜索和查询。
+ <font style="color:rgb(25, 27, 31);">可以直接将文档构建为 Index，这种简单构建的方式是在 Index 初始化时直接加载 文档</font>
+ 示例：<font style="color:rgb(25, 27, 31);">这种方式可以跳过 Node 构建</font>

```python
from llama_index import GPTSimpleVectorIndex

index = GPTSimpleVectorIndex.from_documents(documents)
```

<font style="color:rgb(25, 27, 31);">或者从 Node 构建 Index</font>

```python
from llama_index import GPTSimpleVectorIndex

index = GPTSimpleVectorIndex(nodes)
```

### `VectorStore`
+ `VectorStore` 用于将文档嵌入向量存储在索引中，支持基于向量的相似性搜索。
+ 示例：

```python
from llama_index import VectorStore
vector_store = VectorStore(documents)
```

### `DocumentStore`
+ `DocumentStore` 是一种存储管理组件，用于高效地存储和查询大量文档数据。
+ 它支持基于不同后端的数据存储（如数据库、文件系统等）。

## 多个 Index 结构复用 Node 
<font style="color:rgb(25, 27, 31);">当想在多个索引中，复用一个 Node 时，可以通过定义 DocumentStore 结构，并在添加Nodes时指定 DocumentStore</font>

```python
from gpt_index.docstore import SimpleDocumentStore
​
docstore = SimpleDocumentStore()
docstore.add_documents(nodes)
​
index1 = GPTSimpleVectorIndex(nodes, docstore=docstore)
index2 = GPTListIndex(nodes, docstore=docstore)
```

<font style="color:rgb(25, 27, 31);">如果没指定 docstore，则会在创建 Index 时隐式创建一个。</font>

### `StorageContext`
+ `StorageContext` 是与文档存储相关的组件，支持索引和文档数据的持久化存储。
+ 它可以管理索引的加载、保存和更新。
+ 注意：如果未指定storage_context参数，则在构建索引时会隐式为每个索引创建它。可以通过index.storage_context访问与给定索引关联的docstore。

```python
from llama_index import StorageContext

storage_context = StorageContext.from_defaults（)
storage_context.docstore.add_documents（nodes)

index1 = GPTVectorStoreIndex（nodes，storage_context = storage_context)
index2 = GPTListIndex（nodes，storage_context = storage_context)
```

## index 中插入文档或节点
还可以利用索引的insert功能一次插入一个Document对象，而不是在构建索引时插入。

### `GPTVectorStoreIndex`
```python
from llama_index import GPTVectorStoreIndex

index = GPTVectorStoreIndex（[])
for doc in documents：
    index.insert（doc)
```

如果要直接插入节点，可以使用insert_nodes函数。

```python
from llama_index import GPTVectorStoreIndex

# nodes：Sequence [Node]
index = GPTVectorStoreIndex（[])
index.insert_nodes（nodes)
```

## 自定义 LLM
### `LLMPredictor`
+ `LLMPredictor` 是用于管理与大型语言模型（如 GPT）交互的类。它帮助用户与 LLM 进行预测、推理等任务。
+ 示例：

```python
from llama_index import LLMPredictor
llm_predictor = LLMPredictor(llm="gpt-3")
```

默认情况下，使用OpenAI的text-davinci-003模型。构建索引时，可以选择使用另一个LLM。

```python
from llama_index import LLMPredictor, GPTSimpleVectorIndex, PromptHelper, ServiceContext
from langchain import OpenAI

...

# define LLM
llm_predictor = LLMPredictor(llm=OpenAI(temperature=0, model_name="text-davinci-003"))

# define prompt helper
# set maximum input size
max_input_size = 4096
# set number of output tokens
num_output = 256
# set maximum chunk overlap
max_chunk_overlap = 20
prompt_helper = PromptHelper(max_input_size, num_output, max_chunk_overlap)

service_context = ServiceContext.from_defaults(llm_predictor=llm_predictor, prompt_helper=prompt_helper)

index = GPTSimpleVectorIndex.from_documents(
    documents, service_context=service_context
)
```

### `ServiceContext`
如果在LlamaIndex函数中未指定为关键字参数，则将始终使用此服务上下文作为默认值。有关服务上下文的更多详细信息，包括如何创建全局服务上下文，请参阅自定义ServiceContext页面。

+ `ServiceContext` 是一个配置对象，用于管理与模型（如 GPT 或其他 NLP 模型）交互的上下文。
+ 它包括嵌入模型、查询引擎等配置选项，允许用户控制如何将文档传递给模型进行查询和处理。
+ 示例：

```python
from llama_index import ServiceContext
service_context = ServiceContext.from_defaults()
```

## 自定义`Prompt`
### `Prompt`
+ `Prompt` 类用于构建输入提示，用于生成自然语言查询或文本。它通常与模型查询相关联。
+ 示例：

```python
from llama_index import Prompt
prompt = Prompt(template="What is the summary of this document?")
```

## 自定义`Embeddings`
### `Embedding`
+ `Embedding` 是与文档的嵌入表示相关的功能。它可以将文本转换为向量，并支持基于嵌入的查询操作。
+ 示例：

```python
from llama_index import Embedding
embedding = Embedding(model="openai-gpt3")
```

## 成本预测器
创建索引，插入索引和查询索引可能会使用令牌。可以通过这些操作的输出来跟踪令牌使用情况。在运行操作时，将打印令牌使用情况。

还可以通过index.llm_predictor.last_token_usage获取令牌使用情况。有关更多详细信息，请参阅成本预测器How-To。

## 保存所有以备将来使用
默认情况下，数据存储在内存中。 要持久化到磁盘：

```plain
index.storage_context.persist(persist_dir="<persist_dir>")
```

可以省略persist_dir以默认情况下持久化到`./storage`。

要从磁盘重新加载：

```plain
from llama_index import StorageContext, load_index_from_storage

# 重建存储上下文
storage_context = StorageContext.from_defaults(persist_dir="<persist_dir>")

# 加载索引
index = load_index_from_storage(storage_context)
```

**注意**：如果使用自定义的`ServiceContext`对象初始化索引，则在`load_index_from_storage`期间也需要传入相同的ServiceContext。

## 低级组合 API
提供更细粒度的查询控制

```python
from llama_index import (
    GPTVectorStoreIndex,
    ResponseSynthesizer,
)
from llama_index.retrievers import VectorIndexRetriever
from llama_index.query_engine import RetrieverQueryEngine
from llama_index.indices.postprocessor import SimilarityPostprocessor

# build index
index = GPTVectorStoreIndex.from_documents(documents)

# configure retriever
retriever = VectorIndexRetriever(
    index=index,
    similarity_top_k=2,
)

# configure response synthesizer
response_synthesizer = ResponseSynthesizer.from_args(
    node_postprocessors=[
        SimilarityPostprocessor(similarity_cutoff=0.7)
    ]
)

# assemble query engine
query_engine = RetrieverQueryEngine(
    retriever=retriever,
    response_synthesizer=response_synthesizer,
)

# query
response = query_engine.query("What did the author do growing up?")
print(response)
```

## 配置检索器
索引可以具有各种特定于索引的检索模式。 例如，列表索引支持默认的`ListIndexRetriever`，它可以检索所有节点，以及`ListIndexEmb可以使用以下简写：

```plain
# ListIndexRetriever
    retriever = index.as_retriever(retriever_mode='default')
    # ListIndexEmbeddingRetriever
    retriever = index.as_retriever(retriever_mode='embedding')
```

选择想要的检索器后，可以构建查询引擎：

### `QueryEngine`
+ `QueryEngine` 是用于查询索引的引擎。它将用户的查询与文档进行比较，返回最相关的文档或信息。
+ 示例：

```python
from llama_index import QueryEngine
query_engine = index.as_query_engine()
response = query_engine.query("What is AI?")
```

## 集成其他
### `BaseLangChain`
+ `BaseLangChain` 是与 LangChain 库集成的基类，LangChain 是一个用于构建语言模型链的框架。它允许将多个链式操作组合起来，通常用于更复杂的自然语言处理工作流。

### `Graph`
+ `Graph` 组件是用于表示文档之间关系的结构，可以通过图的方式来存储和查询文档。
+ 示例：

```python
from llama_index import Graph
graph = Graph(documents)
```

## 返回
### `QueryResponse`
+ `QueryResponse` 是查询结果的返回对象，通常包含与查询相关的文档、答案或其他推理结果。

