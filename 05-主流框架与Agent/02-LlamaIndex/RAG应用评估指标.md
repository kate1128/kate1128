> 1. `Answer relevance`：**评估RAG系统的输出是否与问题相关**；
> 2. `Context relevance`：**评估RAG系统召回的上下文是否与问题相关**；
> 3. `Groundness`: **评估RAG系统的输出是否基于召回的上下文**；
>

##  介绍
本章节主要内容为评估 RAG 应用中常用的三个指标，分别为：

1. `Answer relevance`：评估RAG系统的输出是否与问题相关；
2. `Context relevance`：评估RAG系统召回的上下文是否与问题相关；
3. `Groundness`: 评估RAG系统的输出是否基于召回的上下文；

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Building%20and%20Evaluating%20Advanced%20RAG%20Applications/images/ch03_traid.jpg)

安装评估框架，如果已经安装就可以跳过这一步骤。

```python
# requirements
# pip install trulens_eval
```

在这里，为了美观和方便展示，我们设置输出忽略警告信息。

```python
# 忽略警告，避免警告影响输出结果
import warnings
warnings.filterwarnings('ignore')
```

1. 自行设置 `openAI key`
2. 需要重置数据库，这会在之后用于存储问题、召回结果和回答，方便管理和评估。

```python
# 导入Tru类
from trulens_eval import Tru

# 实例化Tru类
tru = Tru()

# 重置数据库
# 数据库之后会用来存储问题、中间召回结果、答案以及评估结果
tru.reset_database()
```

```python
🦑 Tru initialized with db url sqlite:///default.sqlite .
🛑 Secret keys may be written to the database. See the `database_redact_keys` option of `Tru` to prevent this.
```

接下来，导入读取pdf所需要的SimpleDirectoryReader，读取指定文件夹下的pdf文件。 需要注意的是，默认的参数适合读取英文文档，如果文档为中文，需要在后续将全角字符转换为半角字符。

```python
# 设置Llama Index reader
from llama_index import SimpleDirectoryReader

# 从一个文件夹中读取PDF文档，然后加载到document对象中
# 使用的文档为“人工智能”词条的维基百科页面
documents = SimpleDirectoryReader(
    input_files=["./data/人工智能.pdf"]
).load_data()

documents_en = SimpleDirectoryReader(
    input_files=["./data/eBook-How-to-Build-a-Career-in-AI.pdf"]
).load_data()
```

为了方面起见，将读取的pdf文档加载到同一个document对象中，用`"\n\n"`隔开；

```python
from llama_index import Document

# 将documents中的内容合并成一个大文档，而不是每一页都是一个文档
document = Document(text="\n\n".\
                    join([doc.text for doc in documents]))

document_en = Document(text="\n\n".\
                       join([doc.text for doc in documents_en]))
```

将中文标点符号替换成英文标点符号，方便后续处理

```python
# 将中文标点符号替换成英文标点符号，方便后续处理
# 如果是英文文档，可以跳过这一步
# 不处理的话，会导致无法正确切分中文句子，会影响后续sentence_window的大小，导致输入长度大于gpt-3.5-turbo的最大限制
document.text=document.text.replace('。','. ')
document.text=document.text.replace('！','! ')
document.text=document.text.replace('？','? ')
```

设置index存储，首先设置用来评估的大模型为gpt-3.5-turbo，需要注意的是这里使用的版本的上下文窗口为4096，因此需要注意输入的长度。 然后设置embedding模型，我们选择了BAAI/bge-small-zh-v1.5，这里可以根据场景的需要和计算资源的trade off选择模型的大小和语种。

```python
# 设置sentence_index
from utils import build_sentence_window_index

from llama_index.llms import OpenAI

# 设置使用的大模型
# "gpt-3.5-turbo"是模型的名称
# temperature是温度，用来控制文本生成过程中的多样性
llm = OpenAI(model="gpt-3.5-turbo", temperature=0.1)

# 设置embedding模型
# 这里是在本地使用BAAI/bge-small-zh-v1.5
# document的所有的内容会索引到sentence index对象中
# 国内使用可以切换huggingface镜像站
sentence_index = build_sentence_window_index(
    document,
    llm,
    embed_model="local:BAAI/bge-small-zh-v1.5",
    save_dir="sentence_index"
)

sentence_index_en = build_sentence_window_index(
    document_en,
    llm,
    embed_model="local:BAAI/bge-small-en-v1.5",
    save_dir="sentence_index_en"
)
```

使用工具包中封装好的函数，基于上一步建立好的索引，返回用于之后检索的引擎。

```python
from utils import get_sentence_window_query_engine

# 根据sentence_index对象创建一个搜索引擎
# 之后会被用于在RAG应用中进行召回
sentence_window_engine = \
get_sentence_window_query_engine(sentence_index)
```

```python
sentence_window_engine_en = \
get_sentence_window_query_engine(sentence_index_en)
```

这里我们先测试单个问题来debug，看一下输出是什么。

```python
output = sentence_window_engine.query(
    "AI的核心问题和长远目标是什么？")
```

```python
huggingface/tokenizers: The current process just got forked, after parallelism has already been used. Disabling parallelism to avoid deadlocks...
To disable this warning, you can either:
- Avoid using `tokenizers` before the fork if possible
- Explicitly set the environment variable TOKENIZERS_PARALLELISM=(true | false)
```

```python
output_en = sentence_window_engine_en.query(
    "How do you create your AI portfolio?")
# 示例：使用搜索引擎进行召回
```

In [13]:

```python
# 在实际开发中，可以通过查看metadata进行debug
output.metadata
```

Out[13]:

```python
{'7e8484a0-f7d2-4b64-b683-fa76b1dec6fe': {'window': '⼈⼯智能的研究是⾼度技术性和专业的，各分⽀领域都是深⼊且各不相通的，因⽽涉及范围极⼴[9].  ⼈⼯智能的\n研究可以分为⼏个技术问题.  其分⽀领域主要集中在解决具体问题，其中之⼀是，如何使⽤各种不同的⼯具完成\n特定的应⽤程序. \n AI的核⼼问题包括建构能够跟⼈类似甚⾄超卓的推理、知识、计划、学习、交流、感知、移动 、移物、使⽤⼯\n具和操控机械的能⼒等[10].  通⽤⼈⼯智能（GAI）⽬前仍然是该领域的长远⽬标[11].  ⽬前弱⼈⼯智能已经有初\n步成果，甚⾄在⼀些影像识别、语⾔分析、棋类游戏等等单⽅⾯的能⼒达到了超越⼈类的⽔平，⽽且⼈⼯智能的\n通⽤性代表着，能解决上述的问题的是⼀样的AI程序，⽆须重新开发算法就可以直接使⽤现有的AI完成任务，与\n⼈类的处理能⼒相同，但达到具备思考能⼒的统合强⼈⼯智能还需要时间研究，⽐较流⾏的⽅法包括统计⽅法，\n计算智能和传统意义的AI. ',
                                          'original_text': 'AI的核⼼问题包括建构能够跟⼈类似甚⾄超卓的推理、知识、计划、学习、交流、感知、移动 、移物、使⽤⼯\n具和操控机械的能⼒等[10]. '},
 '4e0b1c3d-5ba5-4c6b-b5c6-29df66a75281': {'window': '⼈⼯智能的\n研究可以分为⼏个技术问题.  其分⽀领域主要集中在解决具体问题，其中之⼀是，如何使⽤各种不同的⼯具完成\n特定的应⽤程序. \n AI的核⼼问题包括建构能够跟⼈类似甚⾄超卓的推理、知识、计划、学习、交流、感知、移动 、移物、使⽤⼯\n具和操控机械的能⼒等[10].  通⽤⼈⼯智能（GAI）⽬前仍然是该领域的长远⽬标[11].  ⽬前弱⼈⼯智能已经有初\n步成果，甚⾄在⼀些影像识别、语⾔分析、棋类游戏等等单⽅⾯的能⼒达到了超越⼈类的⽔平，⽽且⼈⼯智能的\n通⽤性代表着，能解决上述的问题的是⼀样的AI程序，⽆须重新开发算法就可以直接使⽤现有的AI完成任务，与\n⼈类的处理能⼒相同，但达到具备思考能⼒的统合强⼈⼯智能还需要时间研究，⽐较流⾏的⽅法包括统计⽅法，\n计算智能和传统意义的AI.  ⽬前有⼤量的⼯具应⽤了⼈⼯智能，其中包括搜索和数学优化、逻辑推演. ',
                                          'original_text': '通⽤⼈⼯智能（GAI）⽬前仍然是该领域的长远⽬标[11]. '}}
```

In [14]:

```python
output_en.metadata
```

Out[14]:

```python
{'e0d0633c-9a38-4330-8980-2e4ceea28d30': {'window': 'Chapter 4: Scoping Successful AI Projects.\n Chapter 5: Finding Projects that Complement \nYour Career Goals.\n Chapter 6: Building a Portfolio of Projects that \nShows Skill Progression.\n Chapter 7: A Simple Framework for Starting Your AI \nJob Search.\n Chapter 8: Using Informational Interviews to Find \nthe Right Job.\n Chapter 9: Finding the Right AI Job for You.\n',
                                          'original_text': 'Chapter 7: A Simple Framework for Starting Your AI \nJob Search.\n'},
 '600b5970-e3cd-47cd-a2e0-ea2ffb2c5e79': {'window': 'Chapter 6: Building a Portfolio of Projects that \nShows Skill Progression.\n Chapter 7: A Simple Framework for Starting Your AI \nJob Search.\n Chapter 8: Using Informational Interviews to Find \nthe Right Job.\n Chapter 9: Finding the Right AI Job for You.\n Chapter 10: Keys to Building a Career in AI.\n Chapter 11: Overcoming Imposter Syndrome.\n',
                                          'original_text': 'Chapter 9: Finding the Right AI Job for You.\n'}}
```

## Feedback functions 反馈函数
`feedback function`是一个衡量RAG系统的问题、上下文、答案三者之间关系的函数。在RAG系统中，`feedback function`通常是一个评估模型的指标，用于评估RAG系统的各个方面的性能。具体来说，在本教程中，主要为，`answer relevance、context relevance、groundness`三个指标。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736475567009-7ded4b33-fbb9-48b4-b8c2-7ce11f7f5d38.png)

```python
import nest_asyncio

# 保证后续可以使用streamlit进行评估结果管理和可视化
nest_asyncio.apply()
```

```python
from trulens_eval import OpenAI as fOpenAI

# 初始化OpenAI gpt-3.5-turbo模型作为provider
# provider之后会用来辅助评估RAG应用的各个指标：answer relevance, context relevance, groundedness.
provider = fOpenAI()
```

### Answer Relevance
`answer relevance`用来评估RAG系统的输出是否与问题相关。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736475651943-975251de-ae2b-4da3-8ce1-46cccf09d93e.png)

answer relevance的feedback function的结构为：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736475687563-e9477917-e04a-4a22-8dff-0cb9a559d52f.png)

这里使用封装好的Feedback函数即可，我们需要做的是：指定评估的方式，指定名称，以及评估的对象。

```python
from trulens_eval import Feedback

# 这里为answer relevance设置feedback
# 使用provider.relevance_with_cot_reasons作为评估函数，即，通过调用LLM使用chain of thought的方式进行评估
# on_input_output()表示在输入和输出上进行评估
f_qa_relevance = Feedback(
    provider.relevance_with_cot_reasons,
    name="Answer Relevance"
).on_input_output()
```

```python
✅ In Answer Relevance, input prompt will be set to __record__.main_input or `Select.RecordInput` .
✅ In Answer Relevance, input response will be set to __record__.main_output or `Select.RecordOutput` .
```

### Context Relevance
context relevance用来评估RAG系统召回的上下文是否与问题相关。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736475724060-38bbb122-e341-483e-ac90-a15eac18911b.png)

其feedback function的结构为：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736475741214-2492f4f1-8116-478a-be4f-0ca9b37ab744.png)

```python
from trulens_eval import TruLlama

# 选择召回的上下文
context_selection = TruLlama.select_source_nodes().node.text
```

这里的设置和上一步类似，只需要修改评估的对象即可。 也可以选择修改评估的方式，进行对比。

```python
import numpy as np

# 使用provider.qs_relevance作为评估函数
# on_input()表示在输入上进行评估
# on(context_selection)表示在context_selection上进行评估
# aggregate(np.mean)表示使用np.mean作为聚合函数
# 这里实际的意思是：对于context_selection中的每个句子，都会进行评估，然后取平均值作为最终的评估结果
f_qs_relevance = (
    Feedback(provider.qs_relevance,
             name="Context Relevance")
    .on_input()
    .on(context_selection)
    .aggregate(np.mean)
)
```

```python
✅ In Context Relevance, input question will be set to __record__.main_input or `Select.RecordInput` .
✅ In Context Relevance, input statement will be set to __record__.app.query.rets.source_nodes[:].node.text .
```

```python
import numpy as np

# 同上，对于context_selection中的每个句子进行评估，取平均值作为评估结果
f_qs_relevance = (
    Feedback(provider.qs_relevance_with_cot_reasons,
             name="Context Relevance")
    .on_input()
    .on(context_selection)
    .aggregate(np.mean)
)
```

```python
✅ In Context Relevance, input question will be set to __record__.main_input or `Select.RecordInput` .
✅ In Context Relevance, input statement will be set to __record__.app.query.rets.source_nodes[:].node.text .
```

### Groundedness
```python
from trulens_eval.feedback import Groundedness

grounded = Groundedness(groundedness_provider=provider)
```

最后是groundedness，用来评估RAG系统的输出是否基于召回的上下文。 设置和之前的类似。

```python
# groundedness的评估，解释同answer relevance和context relevance
f_groundedness = (
    Feedback(grounded.groundedness_measure_with_cot_reasons,
             name="Groundedness"
            )
    .on(context_selection)
    .on_output()
    .aggregate(grounded.grounded_statements_aggregator)
)
```

```python
✅ In Groundedness, input source will be set to __record__.app.query.rets.source_nodes[:].node.text .
✅ In Groundedness, input statement will be set to __record__.main_output or `Select.RecordOutput` .
```

## Evaluation of the RAG application
在RAG系统的评估中，feedback function可以通过多种方式实现。 使用人工打分的方法可以获得最准确的评估结果，但是这种方法成本较高，因此在实际应用中，通常使用自动评估的方法。 在本教程中，使用gpt-3.5-turbo来对RAG系统进行评估。 这种方法的好处是，可以快速、低成本地对RAG系统进行评估，但是其评估结果可能不如人工打分准确。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736475768707-c0dcb50e-e8c9-440d-896a-3a52b05bbc3a.png)

下面是整个RAG系统的评估流程的实现。

```python
from trulens_eval import TruLlama
from trulens_eval import FeedbackMode

# 实例化TruLlama类，用来记录评估结果
# sentence_window_engine是之前创建的搜索引擎
# app_id是应用的ID，用来标识应用
tru_recorder = TruLlama(
    sentence_window_engine,
    app_id="App_1",
    feedbacks=[
        f_qa_relevance,
        f_qs_relevance,
        f_groundedness
    ]
)

tru_recorder_en = TruLlama(
    sentence_window_engine_en,
    app_id="App_2",
    feedbacks=[
        f_qa_relevance,
        f_qs_relevance,
        f_groundedness
    ]
)
```

读取用来评估的问题，这里为了节约时间并降低调用API的成本，我们只设置了6个问题。 在实际场景中，可以手写或通过prompt seed的方法生成更多的问题，覆盖更多的场景。

```python
eval_questions = []
# 读取评估问题，在./data/eval_questions.txt下，可以自定义
with open('./data/eval_questions.txt', 'r') as file:
    for line in file:
        item = line.strip()
        eval_questions.append(item)
```

```python
eval_questions_en = []
with open('./data/eval_questions_en.txt', 'r') as file:
    for line in file:
        item = line.strip()
        eval_questions_en.append(item)
```

```python
eval_questions
```

```python
['人工智能中的先验知识是如何被存储的？',
 '人工智能的自我更新和自我提升是否可能导致其脱离人类的控制？',
 '管理者如何管理AI？',
 '强人工智能是什么？',
 '人工智能被滥用带来的危害？']
```

```python
eval_questions_en
```

```python
['What are the keys to building a career in AI?',
 "How can teamwork contribute to success in AI?'",
 "What is the importance of networking in AI?'",
 "What are some good habits to develop for a successful career?'",
 "How can altruism be beneficial in building a career?'",
 "What is imposter syndrome and how does it relate to AI?'",
 "Who are some accomplished individuals who have experienced imposter syndrome?'",
 "What is the first step to becoming good at AI?'",
 "What are some common challenges in AI?'",
 'Is it normal to find parts of AI challenging?']
```

```python
eval_questions.append("如何在人工智能领域获得成功？")
```

```python
eval_questions
```

```python
['人工智能中的先验知识是如何被存储的？',
 '人工智能的自我更新和自我提升是否可能导致其脱离人类的控制？',
 '管理者如何管理AI？',
 '强人工智能是什么？',
 '人工智能被滥用带来的危害？',
 '如何在人工智能领域获得成功？']
```

接下来开始评估，请求RAG系统的输出，然后使用feedback function对输出进行评估。

```python
# 对于每个评估问题，进行评估，并记录结果
# 注意：该过程可能会比较耗时，请耐心等待
for question in eval_questions:
    with tru_recorder as recording:
        sentence_window_engine.query(question)
```

```python
for question in eval_questions_en:
    with tru_recorder_en as recording:
        sentence_window_engine_en.query(question)
```

之后，需要进行编解码，将评估结果转换为中文可读的形式，方便分析。

```python
records, feedback = tru.get_records_and_feedback(app_ids=[])

# 将记录中的unicode转换成中文，方便查看
def decode_unicode(s):
    return s.encode('ascii').decode('unicode-escape')

records['input'] = records['input'].apply(decode_unicode)
records['output'] = records['output'].apply(decode_unicode)

records.head()
```

```python
# 运行dashboard
# 注意：请检查端口是否被占用，如果被占用，请修改端口号
tru.run_dashboard()
```

```python
Starting dashboard ...
Config file already exists. Skipping writing process.
Credentials file already exists. Skipping writing process.
```

```python
huggingface/tokenizers: The current process just got forked, after parallelism has already been used. Disabling parallelism to avoid deadlocks...
To disable this warning, you can either:
	- Avoid using `tokenizers` before the fork if possible
	- Explicitly set the environment variable TOKENIZERS_PARALLELISM=(true | false)
```

<details><summary>output：</summary><pre><code class="language-json"><code>Accordion(children=(VBox(children=(VBox(children=(Label(value='STDOUT'), Output())), VBox(children=(Label(valu…
Dashboard started at http://10.31.153.170:8501 .</code></pre></details>

<details><summary>output：</summary><pre><code class="language-json"><code>&lt;Popen: returncode: None args: ['streamlit', 'run', '--server.headless=True'...&gt;</code></pre></details>
