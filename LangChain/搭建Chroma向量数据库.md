> 搭建向量数据库
>
> 1. 导入源文件：`file_paths.append(file_path)`
> 2. 加载文件：`from langchain.document_loaders.pdf import PyMuPDFLoader`
> 3. 切分文档：`from langchain.text_splitter import RecursiveCharacterTextSplitter`
> 4. 构建向量数据库：`from langchain.vectorstores.chroma import Chroma`
> 5. 持久化向量数据库`vectordb.persist()`
>

# 基本概念
[向量数据库概述](https://www.yuque.com/qiaokate/su87gb/ft5wouh1kz53ttpp)

# 前序配置
## 导入源文件
```python
import os
from dotenv import load_dotenv, find_dotenv

# 读取本地/项目的环境变量。
# find_dotenv()寻找并定位.env文件的路径
# load_dotenv()读取该.env文件，并将其中的环境变量加载到当前的运行环境中  
# 如果你设置的是全局的环境变量，这行代码则没有任何作用。
_ = load_dotenv(find_dotenv())

# 如果你需要通过代理端口访问，你需要如下配置
# os.environ['HTTPS_PROXY'] = 'http://127.0.0.1:7890'
# os.environ["HTTP_PROXY"] = 'http://127.0.0.1:7890'

# 获取folder_path下所有文件路径，储存在file_paths里
file_paths = []
folder_path = '../../data_base/knowledge_db'
for root, dirs, files in os.walk(folder_path):
    for file in files:
        file_path = os.path.join(root, file)
        file_paths.append(file_path)
print(file_paths[:3])
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="python" id="iOXbm" class="ne-codeblock language-python"><code>['../../data_base/knowledge_db/prompt_engineering/6. 文本转换 Transforming.md', '../../data_base/knowledge_db/prompt_engineering/4. 文本概括 Summarizing.md', '../../data_base/knowledge_db/prompt_engineering/5. 推断 Inferring.md']</code></pre></details>
## 读取文件
```python
from langchain.document_loaders.pdf import PyMuPDFLoader
from langchain.document_loaders.markdown import UnstructuredMarkdownLoader

# 遍历文件路径并把实例化的loader存放在loaders里
loaders = []

for file_path in file_paths:

    file_type = file_path.split('.')[-1]
    if file_type == 'pdf':
        loaders.append(PyMuPDFLoader(file_path))
    elif file_type == 'md':
        loaders.append(UnstructuredMarkdownLoader(file_path))
```

```python
# 下载文件并存储到text
texts = []

for loader in loaders: texts.extend(loader.load())
```

载入后的变量类型为`langchain_core.documents.base.Document`, 文档变量类型同样包含两个属性

+ `page_content` 包含该文档的内容。
+ `meta_data` 为文档相关的描述性数据。

```python
text = texts[1]
print(f"每一个元素的类型：{type(text)}.", 
    f"该文档的描述性数据：{text.metadata}", 
    f"查看该文档的内容:\n{text.page_content[0:]}", 
    sep="\n------\n")
```

<details class="lake-collapse"><summary id="u61a8e41f"><span class="ne-text">output：</span></summary><pre data-language="python" id="kB4lY" class="ne-codeblock language-python"><code>每一个元素的类型：&lt;class 'langchain_core.documents.base.Document'&gt;.
------
该文档的描述性数据：{'source': '../../data_base/knowledge_db/pumkin_book/pumpkin_book.pdf', 'file_path': '../../data_base/knowledge_db/pumkin_book/pumpkin_book.pdf', 'page': 1, 'total_pages': 196, 'format': 'PDF 1.5', 'title': '', 'author': '', 'subject': '', 'keywords': '', 'creator': 'LaTeX with hyperref', 'producer': 'xdvipdfmx (20200315)', 'creationDate': &quot;D:20230303170709-00'00'&quot;, 'modDate': '', 'trapped': ''}
------
查看该文档的内容:
前言
“周志华老师的《机器学习》
（西瓜书）是机器学习领域的经典入门教材之一，周老师为了使尽可能多的读
者通过西瓜书对机器学习有所了解, 所以在书中对部分公式的推导细节没有详述，但是这对那些想深究公式推
导细节的读者来说可能“不太友好”
，本书旨在对西瓜书里比较难理解的公式加以解析，以及对部分公式补充
具体的推导细节。
”
读到这里，大家可能会疑问为啥前面这段话加了引号，因为这只是我们最初的遐想，后来我们了解到，周
老师之所以省去这些推导细节的真实原因是，他本尊认为“理工科数学基础扎实点的大二下学生应该对西瓜书
中的推导细节无困难吧，要点在书里都有了，略去的细节应能脑补或做练习”
。所以...... 本南瓜书只能算是我
等数学渣渣在自学的时候记下来的笔记，希望能够帮助大家都成为一名合格的“理工科数学基础扎实点的大二
下学生”
。
使用说明
• 南瓜书的所有内容都是以西瓜书的内容为前置知识进行表述的，所以南瓜书的最佳使用方法是以西瓜书
为主线，遇到自己推导不出来或者看不懂的公式时再来查阅南瓜书；
• 对于初学机器学习的小白，西瓜书第1 章和第2 章的公式强烈不建议深究，简单过一下即可，等你学得
有点飘的时候再回来啃都来得及；
• 每个公式的解析和推导我们都力(zhi) 争(neng) 以本科数学基础的视角进行讲解，所以超纲的数学知识
我们通常都会以附录和参考文献的形式给出，感兴趣的同学可以继续沿着我们给的资料进行深入学习；
• 若南瓜书里没有你想要查阅的公式，
或者你发现南瓜书哪个地方有错误，
请毫不犹豫地去我们GitHub 的
Issues（地址：https://github.com/datawhalechina/pumpkin-book/issues）进行反馈，在对应版块
提交你希望补充的公式编号或者勘误信息，我们通常会在24 小时以内给您回复，超过24 小时未回复的
话可以微信联系我们（微信号：at-Sm1les）
；
配套视频教程：https://www.bilibili.com/video/BV1Mh411e7VU
在线阅读地址：https://datawhalechina.github.io/pumpkin-book（仅供第1 版）
最新版PDF 获取地址：https://github.com/datawhalechina/pumpkin-book/releases
编委会
主编：Sm1les、archwalker、jbb0523
编委：juxiao、Majingmin、MrBigFan、shanry、Ye980226
封面设计：构思-Sm1les、创作-林王茂盛
致谢
特别感谢awyd234、
feijuan、
Ggmatch、
Heitao5200、
huaqing89、
LongJH、
LilRachel、
LeoLRH、
Nono17、
spareribs、sunchaothu、StevenLzq 在最早期的时候对南瓜书所做的贡献。
扫描下方二维码，然后回复关键词“南瓜书”
，即可加入“南瓜书读者交流群”
版权声明
本作品采用知识共享署名-非商业性使用-相同方式共享4.0 国际许可协议进行许可。</code></pre></details>
## 切分文档
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 切分文档
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)

split_docs = text_splitter.split_documents(texts)
```

# 构建Chroma向量库
Langchain 集成了超过 30 个不同的向量存储库。选择 Chroma 是因为它轻量级且数据存储在内存中，这使得它非常容易启动和开始使用。

LangChain 可以直接使用 OpenAI 和百度千帆的 Embedding，同时，也可以针对其不支持的 Embedding API 进行自定义，例如，可以基于 LangChain 提供的接口，封装一个 `zhupuai_embedding`，来将智谱的` Embedding API `接入到 LangChain 中。在[LangChain 自定义 Embedding封装](https://www.yuque.com/qiaokate/su87gb/qt4m0magu02c4zsn)中，以智谱 Embedding API 为例，介绍了如何将其他 Embedding API 封装到 LangChain中。

**以下将其他两种 Embedding 使用代码以注释的方法呈现，如果使用的是百度 API 或者 OpenAI API，可以根据情况来使用下方 Cell 中的代码**

```python
# 使用 OpenAI Embedding
# from langchain.embeddings.openai import OpenAIEmbeddings
# 使用百度千帆 Embedding
# from langchain.embeddings.baidu_qianfan_endpoint import QianfanEmbeddingsEndpoint
# 使用我们自己封装的智谱 Embedding，需要将封装代码下载到本地使用
from zhipuai_embedding import ZhipuAIEmbeddings

# 定义 Embeddings
# embedding = OpenAIEmbeddings() 
embedding = ZhipuAIEmbeddings()
# embedding = QianfanEmbeddingsEndpoint()

# 定义持久化路径
persist_directory = '../../data_base/vector_db/chroma'
```

## 删除旧的数据库文件
```python
!rm -rf '../../data_base/vector_db/chroma'  # 删除旧的数据库文件（如果文件夹中有文件的话），windows电脑请手动删除
```

## 构建Chroma向量库
```python
from langchain.vectorstores.chroma import Chroma

vectordb = Chroma.from_documents(
    documents=split_docs[:20], # 为了速度，只选择前 20 个切分的 doc 进行生成；使用千帆时因QPS限制，建议选择前 5 个doc
    embedding=embedding,
    persist_directory=persist_directory  # 允许我们将persist_directory目录保存到磁盘上
)
```

## 持久化向量数据库
在此之后，要确保通过运行 `vectordb.persist` 来持久化向量数据库，后续要继续使用

```python
# 持久化向量数据库
vectordb.persist()
```

打印`vectordb._collection.count()`

```python
print(f"向量库中存储的数量：{vectordb._collection.count()}")
```

<details class="lake-collapse"><summary id="u03faf356"><span class="ne-text">output：</span></summary><pre data-language="python" id="LMgZU" class="ne-codeblock language-python"><code>向量库中存储的数量：20</code></pre></details>
