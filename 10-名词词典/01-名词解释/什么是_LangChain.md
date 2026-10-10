> RAG 应用开发框架 LangChain
>

ChatGPT 的巨大成功激发了越来越多的开发者兴趣，他们希望利用 OpenAI 提供的 API 或者私有化模型，来开发基于大型语言模型的应用程序。尽管大型语言模型的调用相对简单，但要创建完整的应用程序，仍然需要大量的定制开发工作，包括 API 集成、互动逻辑、数据存储等等。

为了解决这个问题，从 2022 年开始，许多机构和个人相继推出了多个开源项目，旨在**帮助开发者们快速构建基于大型语言模型的端到端应用程序或工作流程**。其中一个备受关注的项目就是 LangChain 框架。

<br/>color2
**LangChain 框架是一个开源工具，充分利用了大型语言模型的强大能力，以便开发各种下游应用。它的目标是为各种大型语言模型应用提供通用接口，从而简化应用程序的开发流程**。具体来说，LangChain 框架可以实现数据感知和环境互动，也就是说，它能够让语言模型与其他数据来源连接，并且允许语言模型与其所处的环境进行互动。

<br/>

##  六大组件
![画板](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1736750325286-fd218ed1-758b-456b-ac31-fc16cc2c96dc.jpeg)

1. **模型I/O（ModelIO）**：对于任何大语言模型应用来说，其核心无疑都是模型自身。LangChain提供了与任何大语言模型均适配的模型包装器（模型 IO 的功能），分为LLM 和聊天模型包装器（ChatModel）。模型包装器的提示词模板功能使得开发者可以模板化、动态选择和管理模型输入。LangChain自身并不提供大语言模型，而是提供统一的模型接口。模型包装器这种包装方式允许开发者与不同同模型平台底层的API进行交互，从而简化了大语言模型的调用，降低了开发者的学习成本。此外，其输出解析器也能帮助开发者从模型输出中提取所需的信息。
2. **数据增强（DataConnection）**：许多LLM应用需要的用户特定数据并不在模型的训练集中。LangChain提供了加载、转换、存储和查询数据的构建块。开发者可以利用文档加载器从多个来源加载文档，通过文档转换器进行文档切割、转换等操作。矢量存储和数据检索工具则提供了对嵌入数据的存储和查询功能。
3. **链（Chain）**：单独使用LLM对于简单应用可能是足够的，但面对复杂的应用，往往需要将多个LLM模型包装器或其他组件进行链式连接。LangChain为此类"链式"应用提供了接口。
4. **记忆（Memory）**：大部分的LLM应用都有一个对话式的界面，能够引用之前对话中的信息是至关重要的。LangChain提供了多种工具，帮助开发者为系统添加记忆功能。记忆功能可以独立使用，也可以无缝集成到链中。记忆模块留要支持两个基本操作，即读取和写人。在每次运行中，链首先从记忆模块中读取数据，然后在执行核心逻辑后将当前运行的输入和输出写入记忆模块，以供未来引用。
5. **Agent**：核心思想是利用LLM选择操作序列。在链中，操作序列是硬编码的，而在Agent代理中，大语言模型被用作推理引擎，确定执行哪些操作，以及它们的执行顺序。
6. **回调处理器（Callback）**：LangChain提供了一个回调系统，允许开2发者在LLM应用的各个阶段对状态进行干预。这对于日志记录、监视、流处理里等任务非常有用。通过API提供的callbacks参数，开发者可以订阅这些事件。

[LangChain 入门指南构建高可复用、可扩展的 LLM 应用程序.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1736750359515-3826c97f-a59e-4589-9b4a-a5588e4ea58a.pdf)

## 构建如下所示的 RAG 应用
利用 LangChain 框架，可以轻松地构建如下所示的 RAG 应用

[Langchain-Chatchat/docs/img/langchain+chatglm.png at master · chatchat-space/Langchain-Chatchat](https://github.com/chatchat-space/Langchain-Chatchat/blob/master/docs/img/langchain%2Bchatglm.png)

在下图中

+ **每个椭圆形代表了 LangChain 的一个模块**，例如数据收集模块或预处理模块。
+ **每个矩形代表了一个数据状态**，例如原始数据或预处理后的数据。
+ **箭头表示数据流的方向，从一个模块流向另一个模块**。在每一步中，LangChain 都可以提供对应的解决方案，处理各种任务。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736750014300-6a532cea-71bc-4e3d-ba62-d4d917fa2ee4.png)

使用 **LangChain** 构建 **RAG（Retrieval-Augmented Generation）** 应用的基本思路是将一个信息检索模块（如向量数据库）与生成模型（如 GPT-3 或其他 LLM）结合在一起，以便生成更高质量、更准确的文本内容。



构建 RAG 应用的步骤：

1. **设置检索模块（Retriever）**：首先，你需要有一个检索系统来获取相关的文档或信息。常见的做法是通过将文本数据转换成向量，然后用相似性搜索（如通过 **FAISS**）来进行检索。
2. **生成模块（Generator）**：接下来，使用一个语言模型（例如 GPT 或其他大规模预训练模型）来生成文本，这个文本会基于检索到的相关信息。
3. **集成 LangChain**：使用 LangChain 来整合这两个模块。

以下是一个简单的代码示例，展示如何使用 LangChain 和 FAISS 来构建一个 RAG 应用。

### 安装依赖
首先，安装相关依赖：

```bash
pip install langchain openai faiss-cpu
```

### 示例代码
```python
import os
from langchain.chains import RetrievalQA
from langchain.chains.question_answering import load_qa_chain
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 设置 OpenAI API 密钥
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"

# 1. 加载文档
loader = TextLoader("your_document.txt")  # 使用你的文档路径
documents = loader.load()

# 2. 文本分割
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
docs = text_splitter.split_documents(documents)

# 3. 创建嵌入模型并索引文档
embeddings = OpenAIEmbeddings()
vector_store = FAISS.from_documents(docs, embeddings)

# 4. 设置检索器
retriever = vector_store.as_retriever()

# 5. 加载生成模型
llm = OpenAI(temperature=0.7)

# 6. 创建 RAG 链（结合检索和生成）
qa_chain = load_qa_chain(llm, chain_type="stuff")

# 7. 构建 RAG 应用
def rag_query(query):
    # 检索相关文档
    docs = retriever.get_relevant_documents(query)
    
    # 使用 RAG 链生成答案
    answer = qa_chain.run(input_documents=docs, question=query)
    return answer

# 8. 测试
query = "Your question goes here"
answer = rag_query(query)
print(answer)
```

### 代码解释
1. **加载文档**：
    - 通过 `TextLoader` 从一个文本文件加载文档（可以根据需要替换成其他数据源）。
2. **文本分割**：
    - 使用 `RecursiveCharacterTextSplitter` 将文档分割成较小的块，以适应 LLM 的输入大小限制。
3. **嵌入和索引**：
    - 使用 `OpenAIEmbeddings` 生成每个文档块的向量表示。
    - 然后使用 FAISS 构建一个向量数据库，用于高效的相似性检索。
4. **构建检索器**：
    - `as_retriever()` 方法将向量数据库转换为检索器，它支持根据查询返回相关文档。
5. **加载生成模型**：
    - 使用 `OpenAI` 来加载一个生成模型（如 GPT）。你可以根据需要调整 `temperature` 等参数。
6. **RAG 链**：
    - 使用 LangChain 的 `load_qa_chain` 加载一个基于 `OpenAI` 的问题解答链。`chain_type="stuff"` 表示简单地将检索到的文档与问题结合，然后生成回答。
7. **查询与生成**：
    - `rag_query()` 函数接收一个查询，首先用检索器获取相关文档，然后通过 QA 链生成答案。
8. **测试**：
    - 可以传入任意问题来测试生成的答案。

### 扩展
+ **数据源的更改**：可以使用不同的文档加载器，比如 `PDFLoader` 或 `CSVLoader`，或者从数据库中获取数据。
+ **模型和向量存储的替换**：可以替换 OpenAI 的生成模型和 FAISS 向量存储为其他的模型和数据库（如使用 **Pinecone**、**Weaviate** 等）。
+ **优化和调优**：根据实际需求，可以进一步优化检索部分（比如使用更复杂的向量数据库）或调整生成模型的参数（如 `temperature` 和 `max_tokens`）。

通过这种方式，可以构建一个强大的 RAG 应用，能够根据大量的文档信息生成高质量的答案。





