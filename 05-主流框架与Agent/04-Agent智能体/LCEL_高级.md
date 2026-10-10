> LangChain Expression Language（或称为 LCEL），称之为 Langchain 的表达式语言
>

[什么是 LCEL](https://www.yuque.com/qiaokate/su87gb/fpwp7u1tggmac2yb)

# 简单链 Simple Chain
接下来依旧会使用 OpenAI 的 API，所以首先要初始化的 API_Key，这个方法和上一章的方式是一样的。

```python
# !pip install langchain
# !pip install openai==0.28
# !pip install "langchain[docarray]"
# !pip install tiktoken
```

```python
import os
import openai
from dotenv import load_dotenv, find_dotenv

# 读取本地/项目的环境变量。

# find_dotenv()寻找并定位.env文件的路径
# load_dotenv()读取该.env文件，并将其中的环境变量加载到当前的运行环境中
# 如果你设置的是全局的环境变量，这行代码则没有任何作用。
_ = load_dotenv(find_dotenv())

# 获取环境变量 OPENAI_API_KEY
openai_api_key = os.environ['OPENAI_API_KEY']
```

接下来首先导入 LangChain 的库，并且定义一个简单的链，这个链包括提示模板，大语言模型和一个输出解析器。可以看到，成功输出了大语言模型的结果，完成了一个简单的链。

```python
# 导入LangChain所需的模块
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.schema.output_parser import StrOutputParser

# 使用 ChatPromptTemplate 从模板创建一个提示，模板中的 {topic} 将在后续代码中替换为实际的话题
prompt = ChatPromptTemplate.from_template(
    "告诉我一个关于{topic}的短笑话"
)

# 创建一个 ChatOpenAI 模型实例，默认使用 gpt-3.5-turbo 模型
model = ChatOpenAI(openai_api_base="https://xiaoai.plus/v1")

# 创建一个StrOutputParser实例，用于解析输出
output_parser = StrOutputParser()

# 创建一个链式调用，将 prompt、model 和output_parser 连接在一起
chain = prompt | model | output_parser

# 调用链式调用，并传入参数
chain.invoke({"topic": "熊"})
```

<details><summary>output：</summary><pre><code class="language-json"><code>'为什么熊不喜欢玩扑克牌？因为他总是把两个熊掌都露出来！'</code></pre></details>
如果去查看`Chain`的输出，会发现，他跟定义的是一样的，一共有三部分进行组成，也就是`Chain = prompt | LLM |OutputParser `。`|`符号类似于 unix 管道操作符，它将不同的组件链接在一起，将一个组件的输出作为输入提供给下一个组件。在这个链中，用户输入被传递给提示模板，然后提示模板输出被传递给模型，然后模型输出被传递到输出解析器。

```python
# 查看Chain的值
chain
```

<details><summary>output：</summary><pre><code class="language-json"><code>ChatPromptTemplate(input_variables=['topic'], messages=[HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['topic'], template='告诉我一个关于{topic}的短笑话'))])
| ChatOpenAI(client=&lt;class 'openai.api_resources.chat_completion.ChatCompletion'&gt;, openai_api_key='xxx', openai_proxy='')
| StrOutputParser()</code></pre></details>
#  更复杂的链 More complex chain
接下来，会创建一个更复杂的链条，在之前的课程中，接触过如何进行检索增强生成。所以接下来使用 LCEL 来重复之前的过程，将用户的问题和向量数据库检索结果结合起来，使用 `RunnableMap` 来构建一个更复杂的链。

## 构建简单向量数据库
首先构建一个向量数据库，这个简单的向量数据库只包含两句话，使用 OpenAI 的 Embedding 作为嵌入模型，然后通过 `vector store.as_retriever `来创建一个检索器。

```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import DocArrayInMemorySearch

# 创建一个DocArrayInMemorySearch对象，用于存储和搜索文档向量
vectorstore = DocArrayInMemorySearch.from_texts(
    ["哈里森在肯肖工作", "熊喜欢吃蜂蜜"],
    embedding=OpenAIEmbeddings() # 使用OpenAI的Embedding
)

# 创建一个检索器
retriever = vectorstore.as_retriever()
```

通过之前的学习，如果调用`retriever.get_relevant_documents`，会得到相关的检索文档，首先问“哈里森在哪里工作？”，会发现返回了一个文档列表，他会根据相似度排序返回文档列表，所以其中最相关的放在了第一个。

```python
# 获取与问题“哈里森在哪里工作？”相关的文档
retriever.get_relevant_documents("哈里森在哪里工作？")
```

<details><summary>output：</summary><pre><code class="language-json"><code>[Document(page_content='哈里森在肯肖工作'), Document(page_content='熊喜欢吃蜂蜜')]</code></pre></details>
如果换一个问题，比如"熊喜欢吃什么"，可以看到问题的顺序就发生了变化。

```python
# 获取与问题“熊喜欢吃什么”相关的文档
retriever.get_relevant_documents("熊喜欢吃什么")
```

<details><summary>output：</summary><pre><code class="language-python"><code>[Document(page_content='熊喜欢吃蜂蜜'), Document(page_content='哈里森在肯肖工作')]</code></pre></details>
## 使用RunnableMap
上述例子返回两个结果是因为只有两个文档列表，这完全适用于更多文档情况。接下来会加入`RunnableMap`，在这个`RunnableMap`中，不仅仅有用户的问题，以及有对应的问题的文档列表，相当于这也为大模型的文档增加了上下文，这样就能完成检索增强的事情。如果正常问一个问题，可以看到，大模型正确的返回了文档里面的结果，得到了正确的输出。

```python
from langchain.schema.runnable import RunnableMap

# 定义一个模板字符串template
template = """仅根据以下上下文回答问题：
{context}

问题：{question}
"""

# 使用 template 作为模板
prompt = ChatPromptTemplate.from_template(template)

# 创建一个处理链 chain ，包含了 RunnableMap、prompt、model 和 output_parser 组件
chain = RunnableMap({
    "context": lambda x: retriever.get_relevant_documents(x["question"]),
    "question": lambda x: x["question"]
}) | prompt | model | output_parser

# 调用chain的invoke方法
chain.invoke({"question": "哈里森在哪里工作?"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>'肯肖'</code></pre></details>
如果想更深入挖掘一下背后的工作机理，可以看一下`RunnableMap`，把其创建为一个输入，用一样的方式进行操作。

可以看到，在这之中，`RunnableMap`提供了`context`和`question`两个变量，一个是查询的文档列表，另一个是对应的问题，这个大模型就可以根据提供文档来总结回答对应的问题了。

```python
# 创建一个RunnableMap对象，其中包含两个键值对
# 键 "context" 对应一个lambda函数，用于获取相关文档，函数输入参数为x，即输入的字典，函数返回值为retriever.get_relevant_documents(x["question"])
# 键 "question" 对应一个lambda函数，用于获取问题，函数输入参数为x，即输入的字典，函数返回值为x["question"]
inputs = RunnableMap({
    "context": lambda x: retriever.get_relevant_documents(x["question"]),
    "question": lambda x: x["question"]
})

# 调用 inputs 的 invoke 方法，并传递一个字典作为参数，字典中包含一个键值对，键为"question"，值为"哈里森在哪里工作?"
inputs.invoke({"question": "哈里森在哪里工作?"})

```

<details><summary>output：</summary><pre><code class="language-python"><code>{'context': [Document(page_content='哈里森在肯肖工作'),
             Document(page_content='熊喜欢吃蜂蜜')],
 'question': '哈里森在哪里工作?'}</code></pre></details>
#  绑定 Bind
[Function Calling](https://www.yuque.com/qiaokate/su87gb/cu0lppkawdfrm12r)介绍了OpenAI函数的调用，新的`function`参数可以自动判断是否要使用工具函数，如果需要就会返回需要使用的参数。接下来也使用LangChain实现OpenAI函数调用的新功能，首先需要一个函数的描述信息，以及定义函数，这里的函数还是使用[Function Calling](https://www.yuque.com/qiaokate/su87gb/cu0lppkawdfrm12r)的`get_current_weather`函数。

```python
# 定义一个函数
functions = [
    {
        "name": "get_current_weather",
        "description": "获取指定位置的当前天气情况",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "城市和省份，例如：北京，北京市",
                },
                "unit": {"type": "string", "enum": ["摄氏度", "华氏度"]},
            },
            "required": ["location"],
        },
    }
]
```

## 单函数绑定
接下来使用`bind`的方法把工具函数绑定到大模型上，并构建一个简单的链。进行调用以后，可以看到返回了一个`AIMessage`， 其中返回的`content`为空，但是返回了需要调用工具函数的参数。

```python
# 使用ChatPromptTemplate.from_messages方法创建一个ChatPromptTemplate对象
prompt = ChatPromptTemplate.from_messages(
    [
        ("human", "{input}")
    ]
)

# 使用bind方法绑定functions参数
model = ChatOpenAI(temperature=0).bind(functions=functions)

runnable = prompt | model

# 调用invoke方法
runnable.invoke({"input": "北京天气怎么样？"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'get_current_weather', 'arguments': '{\n  &quot;location&quot;: &quot;北京，北京市&quot;\n}'}})</code></pre></details>
##  多个函数绑定
同时也可以定义多个`function`，大模型在对话的时候可以自动判断使用哪一个函数。这里面定义有两个函数，第一个函数是类似于前面的`weather_search`，搜索给定机场的天气，然后还定义了一个赛事体育新闻搜索的`sports_search`，查询天气的函数`weather_search`接受的参数为`airport_code`即机场代码，体育新闻搜索函数`sports_search`接受的参数为`team_name`即体育队名。由于这里不需要运行这些函数，因为大模型是通过问的问题来自动判断是否调用这些函数，并且返回参数，并不会直接帮调用。

```python
functions = [
    {
        "name": "weather_search",
        "description": "搜索给定机场代码的天气",
        "parameters": {
            "type": "object",
            "properties": {
                "airport_code": {
                    "type": "string",
                    "description": "要获取天气的机场代码"
                },
            },
            "required": ["airport_code"]
        }
    },
    {
        "name": "sports_search",
        "description": "搜索最近体育赛事的新闻",
        "parameters": {
            "type": "object",
            "properties": {
                "team_name": {
                    "type": "string",
                    "description": "要搜索的体育队名"
                },
            },
            "required": ["team_name"]
        }
    }
]
```

接着就可以使用函数绑定大模型，定义一个简单的链，可以看到，当问了相关的问题以后，大模型能够自动判断并且正确返回参数，知道需要调用函数了。

```python
# 绑定大模型
model = model.bind(functions=functions)
runnable = prompt | model

runnable.invoke({"input": "爱国者队昨天表现的怎么样?"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'sports_search', 'arguments': '{\n  &quot;team_name&quot;: &quot;爱国者队&quot;\n}'}})</code></pre></details>
# 后备措施 `Fallbacks`
在使用早期的OpenAI模型如"`text-davinci-001`"，这些模型在对话过程中，不支持格式化输出结果即它们都是以字符串的形式输出结果，这对有时候需要解析 LLM 的输出带来一些麻烦，比如下面这个例子，就是利用早期模型"`text-davinci-001`"来回答用户的问题，希望 llm 能以 json 格式输出结果。

定义了 OpenAI 的模型以及创建了一个简单的链，以此加入 json 希望能以 json 格式输出结果，让 `simple_model` 写三首诗，并以 josn 格式输出，每首诗必须包含:`标题，作者和诗的第一句`。会发现结果只有字符串，无法输出指定格式的内容，虽然里面有一些`[`，但是本质上还是一个大的字符串，这就无法让解析输出。

由于OpenAI于2024年1月4日停用了模型`text-davinci-001`，你将使用OpenAI推荐的替代模型`gpt-3.5-turbo-instruct`。

在使用语言模型时，你可能经常会遇到来自底层 API 的问题，无论这些问题是速率限制还是停机时间。因此，当你将 LLM 应用程序转移到实际生产环境中时，防范这些问题变得越来越重要。这就是为什么引入了`回退（Fallbacks）`的概念。

## 使用早期模型格式化输出
```python
from langchain.llms import OpenAI
import json

# 使用早期的OpenAI模型
simple_model = OpenAI(
    temperature=0,
    max_tokens=1000,
    model="gpt-3.5-turbo-instruct"
)
simple_chain = simple_model | json.loads

challenge = "写三首诗，并以josn格式输出，每首诗必须包含:标题，作者和诗的第一句。"

simple_model.invoke(challenge)
```

<details><summary>output：</summary><pre><code class="language-python"><code>'\n\n{\n  &quot;title&quot;: &quot;春风&quot;,\n  &quot;author&quot;: &quot;李白&quot;,\n  &quot;first_line&quot;: &quot;春风又绿江南岸&quot;,\n  &quot;content&quot;: [\n    &quot;春风又绿江南岸&quot;,\n    &quot;花开满树柳如丝&quot;,\n    &quot;鸟儿欢唱天地宽&quot;,\n    &quot;人间春色最宜人&quot;\n  ]\n}\n\n{\n  &quot;title&quot;: &quot;夜雨&quot;,\n  &quot;author&quot;: &quot;杜甫&quot;,\n  &quot;first_line&quot;: &quot;夜雨潇潇&quot;,\n  &quot;content&quot;: [\n    &quot;夜雨潇潇&quot;,\n    &quot;孤灯照旧&quot;,\n    &quot;思念如潮&quot;,\n    &quot;泛滥心头&quot;\n  ]\n}\n\n{\n  &quot;title&quot;: &quot;山行&quot;,\n  &quot;author&quot;: &quot;王维&quot;,\n  &quot;first_line&quot;: &quot;远上寒山石径斜&quot;,\n  &quot;content&quot;: [\n    &quot;远上寒山石径斜&quot;,\n    &quot;白云生处有人家&quot;,\n    &quot;停车坐爱枫林晚&quot;,\n    &quot;霜叶红于二月花&quot;\n  ]\n}'</code></pre></details>
如果使用`simple_chain`来运行，就会发现出现了 json 解码错误的问题，因为返回的结果就是一个字符串，无法解析，所以下面代码就会报错。

```python
simple_chain.invoke(challenge)
```

<details><summary>output：</summary><pre><code class="language-python"><code>---------------------------------------------------------------------------
JSONDecodeError                           Traceback (most recent call last)
&lt;ipython-input-17-7b2363c45b31&gt; in &lt;cell line: 1&gt;()
----&gt; 1 simple_chain.invoke(challenge)

/usr/local/lib/python3.10/dist-packages/langchain_core/runnables/base.py in invoke(self, input, config)
2051         try:
    2052             for i, step in enumerate(self.steps):
        -&gt; 2053                 input = step.invoke(
            2054                     input,
            2055                     # mark each step as a child run

            /usr/local/lib/python3.10/dist-packages/langchain_core/runnables/base.py in invoke(self, input, config, **kwargs)
            3505         &quot;&quot;&quot;Invoke this runnable synchronously.&quot;&quot;&quot;
3506         if hasattr(self, &quot;func&quot;):
-&gt; 3507             return self._call_with_config(
    3508                 self._invoke,
    3509                 input,

    /usr/local/lib/python3.10/dist-packages/langchain_core/runnables/base.py in _call_with_config(self, func, input, config, run_type, **kwargs)
1244             output = cast(
    1245                 Output,
    -&gt; 1246                 context.run(
        1247                     call_func_with_variable_args,
        1248                     func,  # type: ignore[arg-type]

        /usr/local/lib/python3.10/dist-packages/langchain_core/runnables/config.py in call_func_with_variable_args(func, input, config, run_manager, **kwargs)
    324     if run_manager is not None and accepts_run_manager(func):
325         kwargs[&quot;run_manager&quot;] = run_manager
--&gt; 326     return func(input, **kwargs)  # type: ignore[call-arg]
327
328

/usr/local/lib/python3.10/dist-packages/langchain_core/runnables/base.py in _invoke(self, input, run_manager, config, **kwargs)
3381                         output = chunk
3382         else:
-&gt; 3383             output = call_func_with_variable_args(
    3384                 self.func, input, config, run_manager, **kwargs
    3385             )

/usr/local/lib/python3.10/dist-packages/langchain_core/runnables/config.py in call_func_with_variable_args(func, input, config, run_manager, **kwargs)
324     if run_manager is not None and accepts_run_manager(func):
325         kwargs[&quot;run_manager&quot;] = run_manager
--&gt; 326     return func(input, **kwargs)  # type: ignore[call-arg]
327
328

/usr/lib/python3.10/json/__init__.py in loads(s, cls, object_hook, parse_float, parse_int, parse_constant, object_pairs_hook, **kw)
344             parse_int is None and parse_float is None and
345             parse_constant is None and object_pairs_hook is None and not kw):
--&gt; 346         return _default_decoder.decode(s)
347     if cls is None:
348         cls = JSONDecoder

/usr/lib/python3.10/json/decoder.py in decode(self, s, _w)
338         end = _w(s, end).end()
339         if end != len(s):
--&gt; 340             raise JSONDecodeError(&quot;Extra data&quot;, s, end)
341         return obj
342

JSONDecodeError: Extra data: line 15 column 1 (char 147)</code></pre></details>
## 使用新模型格式化输出
所以会发现早期版本的 OpenAI 模型不支持格式化的输出，所以即使使用 LangChain 并且加上了`json.load`但是还是会出现错误，但是如果使用新的`gpt-3.5-turbo`模型就不会出现这个问题。

```python
# 默认使用新的模型
model = ChatOpenAI(temperature=0)
chain = model | StrOutputParser() | json.loads

chain.invoke(challenge)
```

<details><summary>output：</summary><pre><code class="language-python"><code>{'poem1': {'title': '春风',
           'author': '李白',
           'first_line': '春风又绿江南岸。',
           'content': '春风又绿江南岸，明月何时照我还。'},
 'poem2': {'title': '静夜思',
           'author': '杜甫',
           'first_line': '床前明月光，',
           'content': '床前明月光，疑是地上霜。'},
 'poem3': {'title': '登鹳雀楼',
           'author': '王之涣',
           'first_line': '白日依山尽，',
           'content': '白日依山尽，黄河入海流。'}}</code></pre></details>
## `fallbacks`方法
那这个时候可能就会思考，有没有什么方法，在不用改变太多代码的情况下，让早期的模型也能达到格式化输出的效果，而不是写复杂的格式化输出的代码去对结果进行操作。这时候就可以使用`fallbacks`的方式赋予早期模型这样格式化的能力，从结果也可以看出，成功使用`fallbacks`赋予了简单模型格式化的能力。

```python
# 使用with_fallbacks机制
final_chain = simple_chain.with_fallbacks([chain])

# 调用final_chain的invoke方法，并传递challenge参数
final_chain.invoke(challenge)
```

<details><summary>output：</summary><pre><code class="language-python"><code>{'poem1': {'title': '春风',
           'author': '李白',
           'first_line': '春风又绿江南岸。',
           'content': '春风又绿江南岸，明月何时照我还。'},
 'poem2': {'title': '静夜思',
           'author': '杜甫',
           'first_line': '床前明月光，',
           'content': '床前明月光，疑是地上霜。'},
 'poem3': {'title': '登鹳雀楼',
           'author': '王之涣',
           'first_line': '白日依山尽，',
           'content': '白日依山尽，黄河入海流。'}}</code></pre></details>
## fallbacks 是如何实现的？
当调用 LLM 时，经常会出现由于底层 API 问题、速率问题或者网络问题等原因，导致不能成功运行 LLM 。在这种情况下，就可以使用回退这种方法来解决这个问题，具体来说，他是通过使用另一种 LLM 来代替原先的不可运行的 LLM 产生结果，请看下面例子：

```python
from langchain_core.chat_models.openai import ChatOpenAI
from langchain_core.chat_models.anthropic import ChatAnthropic

model = ChatAnthropic().with_fallbacks([ChatOpenAI()])
model.invoke('hello')
```

在这种情况下，通常会优先使用 ChatAnthropic 进行回答，但是如果调用 ChatAnthropic 失败了，会回退到使用 ChatOpenAI 模型来生成响应。如果两种 LLM 都失败了，将会回退到一种硬编码响应。硬编码的默认响应用于处理异常情况或者在无法从外部资源获取所需信息时提供一个备用选项，例如 "Looks like our LLM providers are down. Here's a nice 🦜️ emoji for you instead."（看起来的 LLM 提供商出了问题，那么，这里有一个可爱的 🦜️ 表情符号给你。）

如果想了解更多关于 fallbacks 的内容，请参考[Fallbacks | 🦜️🔗 LangChain](https://python.langchain.com/v0.1/docs/guides/productionization/fallbacks/)

# 接口 Interface
在使用LangChain中，存在许多接口，其中公开的标准接口包括：

+ stream：流式返回输出内容
+ invoke：输入调用chain
+ batch：在输入列表中并行调用chain

这些也有相应的异步方法：

+ astream：异步流式返回输出内容
+ ainvoke：在输入上异步调用chain
+ abatch：在输入列表中并行异步调用chain

首先定义给一个简单提示模板，也就是"给我讲一个关于{主题}的短笑话"，然后定义了一个简单的链`Chain = prompt | LLM | OutputParser`。

```python
# 创建一个ChatPromptTemplate对象，使用模板"给我讲一个关于{topic}的短笑话"
prompt = ChatPromptTemplate.from_template(
    "给我讲一个关于{topic}的短笑话"
)

# 创建一个ChatOpenAI模型
model = ChatOpenAI()

# 创建一个StrOutputParser对象
output_parser = StrOutputParser()

# 创建一个chain，将prompt、model和output_parser连接起来
chain = prompt | model | output_parser
```

##  invoke接口
接下来分别使用对应的接口，比如首先使用常规的`invoke`的调用，这个也是前面展现的方法，得到了对应结果。

```python
chain.invoke({"topic": "熊"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>'当熊在森林里遇到一只兔子时，他问：“兔子先生，你有没有问题？”兔子回答道：“当然，先生熊，我有一个问题。你怎么会拉这么长的尾巴？”熊听后笑了起来：“兔子先生，这不是尾巴，这是我的领带！”'</code></pre></details>
## batch接口
再尝试使用`batch`的接口，会发现大模型可以返回两个问题的答案，会给chain一个输入的列表，列表中可以包含多个问题，最后返回多个问题的答案。

```python
chain.batch([{"topic": "熊"}, {"topic": "狐狸"}])
```

<details><summary>output：</summary><pre><code class="language-python"><code>['好的，这是一个关于熊的短笑话：\n\n有一天，一只熊走进了一家酒吧。他走到吧台前，对酒保说：“请给我一杯……蜂蜜啤酒。”\n\n酒保疑惑地看着熊，说：“对不起，这里没有蜂蜜啤酒。”\n\n熊有些失望地叹了口气，然后说：“好吧，那就给我来一杯……草莓酒吧。”\n\n酒保摇摇头，说：“抱歉，也没有草莓酒。”\n\n熊又叹了口气，然后说：“那请给我来一杯……蜜糖红酒吧。”\n\n酒保实在无法忍受了，他对熊说：“对不起，这里没有这些奇怪的酒，你是熊，你应该知道熊只能喝蜂蜜。”\n\n熊听后一愣，然后脸色一变，说：“原来你们这里没有蜂蜜啤酒，草莓酒和蜜糖红酒？那请给我来一杯……白开水吧。”',
 '有一天，一只狐狸在森林里遇到了一只兔子。狐狸笑嘻嘻地对兔子说：“喂，兔子，我有一个好消息和一个坏消息，你想先听哪个？”兔子有些好奇地问：“那就先告诉我好消息吧。”狐狸眯起眼睛说：“好消息是，你的智商比我高。”兔子高兴地跳了起来：“太好了，那坏消息是什么？”狐狸一脸轻松地说：“坏消息是，你的智商还比不过胡萝卜。”']</code></pre></details>
## stream接口
接下来在看看`stream`接口，也就是流式输出内容，这样的功能很有必要，有时候可以免去用户等待的烦恼，让用户看到一个一个词蹦出来而不是一个空的屏幕，这样会带来更好的用户体验。

```python
for t in chain.stream({"topic": "熊"}):
    print(t)
```

<details><summary>output：</summary><pre><code class="language-python"><code>好
的
，
这
是
一个
关
于
熊
的
短
笑
话
：

有
一
天
，
一
只
小
熊
走
进
了
一
家
酒
吧
。
他
走
到
吧
台
前
，
对
酒
保
说
：“
酒
保
，
给
我
一
杯
牛
奶
。”

酒
保
惊
讶
地
问
道
：“
小
熊
，
你
怎
么
会
来
这
里
喝
牛
奶
？
”

小
熊
深
情
地
回
答
：“
因
为
我的
妈
妈
说
，
每
当
我
喝
酒
的
时
候
，
我
都
会
变
得
熊
样
！”</code></pre></details>
## 异步接口
还可以尝试异步来调用，使用`ainvoke`来调用。

```python
response = await chain.ainvoke({"topic": "熊"})
response
```

<details><summary>output：</summary><pre><code class="language-python"><code>'好的，以下是一个关于熊的短笑话：\n\n有一只熊走进了一家餐厅，他走到柜台前，对着服务员说：“我想要一杯咖啡和......嗯，一块...牛肉三明治。”\n服务员疑惑地看着熊，然后问道：“对不起，先生，你是真的想要一块牛肉三明治吗？”\n熊点了点头。\n服务员又问：“那请问为什么你要来这里点餐呢？”\n熊回答道：“因为我是个熊啊！”'</code></pre></details>
#  英文提示词
## 构建简单链
```python
prompt = ChatPromptTemplate.from_template(
    "tell me a short joke about {topic}"
)
model = ChatOpenAI()
output_parser = StrOutputParser()

chain = prompt | model | output_parser

chain.invoke({"topic": "bears"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>'Why did the bear bring a flashlight to the party?\n\nBecause he wanted to be the &quot;light&quot; of the bearbecue!'</code></pre></details>
## 构建简单文档数据库
```python
vectorstore = DocArrayInMemorySearch.from_texts(
    ["harrison worked at kensho", "bears like to eat honey"],
    embedding=OpenAIEmbeddings()
)
retriever = vectorstore.as_retriever()
```

```python
retriever.get_relevant_documents("where did harrison work?")
```

<details><summary>output：</summary><pre><code class="language-python"><code>[Document(page_content='harrison worked at kensho'),
 Document(page_content='bears like to eat honey')]</code></pre></details>
```python
retriever.get_relevant_documents("what do bears like to eat")
```

<details><summary>output：</summary><pre><code class="language-python"><code>[Document(page_content='bears like to eat honey'),
 Document(page_content='harrison worked at kensho')]</code></pre></details>
```python
template = """Answer the question based only on the following context:
{context}

Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)

chain = RunnableMap({
    "context": lambda x: retriever.get_relevant_documents(x["question"]),
    "question": lambda x: x["question"]
}) | prompt | model | output_parser

chain.invoke({"question": "where did harrison work?"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>'Harrison worked at Kensho.'</code></pre></details>
## 使用RunnableMap
```python
inputs = RunnableMap({
    "context": lambda x: retriever.get_relevant_documents(x["question"]),
    "question": lambda x: x["question"]
})

inputs.invoke({"question": "where did harrison work?"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>{'context': [Document(page_content='harrison worked at kensho'),
             Document(page_content='bears like to eat honey')],
 'question': 'where did harrison work?'}</code></pre></details>
## 单函数绑定
```python
functions = [
    {
      "name": "weather_search",
      "description": "Search for weather given an airport code",
      "parameters": {
        "type": "object",
        "properties": {
          "airport_code": {
            "type": "string",
            "description": "The airport code to get the weather for"
          },
        },
        "required": ["airport_code"]
      }
    }
  ]
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("human", "{input}")
    ]
)
model = ChatOpenAI(temperature=0).bind(functions=functions)

runnable = prompt | model

runnable.invoke({"input": "what is the weather in sf"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'weather_search', 'arguments': '{\n  &quot;airport_code&quot;: &quot;SFO&quot;\n}'}})</code></pre></details>
## 多个函数绑定
```python
functions = [
    {
      "name": "weather_search",
      "description": "Search for weather given an airport code",
      "parameters": {
        "type": "object",
        "properties": {
          "airport_code": {
            "type": "string",
            "description": "The airport code to get the weather for"
          },
        },
        "required": ["airport_code"]
      }
    },
        {
      "name": "sports_search",
      "description": "Search for news of recent sport events",
      "parameters": {
        "type": "object",
        "properties": {
          "team_name": {
            "type": "string",
            "description": "The sports team to search for"
          },
        },
        "required": ["team_name"]
      }
    }
  ]
```

```python
model = model.bind(functions=functions)

runnable = prompt | model

runnable.invoke({"input": "how did the patriots do yesterday?"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'sports_search', 'arguments': '{\n  &quot;team_name&quot;: &quot;patriots&quot;\n}'}})</code></pre></details>
## 使用早期模型格式化输出
```python
simple_model = OpenAI(
    temperature=0,
    max_tokens=1000,
    model="gpt-3.5-turbo-instruct"
)
simple_chain = simple_model | json.loads

challenge = "write three poems in a json blob, where each poem is a json blob of a title, author, and first line"

simple_model.invoke(challenge)
```

<details><summary>output：</summary><pre><code class="language-python"><code>'\n\n{\n    &quot;title&quot;: &quot;Autumn Leaves&quot;,\n    &quot;author&quot;: &quot;Emily Dickinson&quot;,\n    &quot;first_line&quot;: &quot;The leaves are falling, one by one&quot;\n}\n\n{\n    &quot;title&quot;: &quot;The Ocean\'s Song&quot;,\n    &quot;author&quot;: &quot;Pablo Neruda&quot;,\n    &quot;first_line&quot;: &quot;I hear the ocean\'s song, a symphony of waves&quot;\n}\n\n{\n    &quot;title&quot;: &quot;A Winter\'s Night&quot;,\n    &quot;author&quot;: &quot;Robert Frost&quot;,\n    &quot;first_line&quot;: &quot;The snow falls softly, covering the ground&quot;\n}'</code></pre></details>
**早期模型不支持，会出现解码错误**

```python
 simple_chain.invoke(challenge)
```

<details><summary>output：</summary><pre><code class="language-python"><code>---------------------------------------------------------------------------
JSONDecodeError                           Traceback (most recent call last)
&lt;ipython-input-39-7b2363c45b31&gt; in &lt;cell line: 1&gt;()
----&gt; 1 simple_chain.invoke(challenge)

/usr/local/lib/python3.10/dist-packages/langchain_core/runnables/base.py in invoke(self, input, config)
2051         try:
    2052             for i, step in enumerate(self.steps):
        -&gt; 2053                 input = step.invoke(
            2054                     input,
            2055                     # mark each step as a child run

            /usr/local/lib/python3.10/dist-packages/langchain_core/runnables/base.py in invoke(self, input, config, **kwargs)
            3505         &quot;&quot;&quot;Invoke this runnable synchronously.&quot;&quot;&quot;
3506         if hasattr(self, &quot;func&quot;):
-&gt; 3507             return self._call_with_config(
    3508                 self._invoke,
    3509                 input,

    /usr/local/lib/python3.10/dist-packages/langchain_core/runnables/base.py in _call_with_config(self, func, input, config, run_type, **kwargs)
1244             output = cast(
    1245                 Output,
    -&gt; 1246                 context.run(
        1247                     call_func_with_variable_args,
        1248                     func,  # type: ignore[arg-type]

        /usr/local/lib/python3.10/dist-packages/langchain_core/runnables/config.py in call_func_with_variable_args(func, input, config, run_manager, **kwargs)
    324     if run_manager is not None and accepts_run_manager(func):
325         kwargs[&quot;run_manager&quot;] = run_manager
--&gt; 326     return func(input, **kwargs)  # type: ignore[call-arg]
327
328

/usr/local/lib/python3.10/dist-packages/langchain_core/runnables/base.py in _invoke(self, input, run_manager, config, **kwargs)
3381                         output = chunk
3382         else:
-&gt; 3383             output = call_func_with_variable_args(
    3384                 self.func, input, config, run_manager, **kwargs
    3385             )

/usr/local/lib/python3.10/dist-packages/langchain_core/runnables/config.py in call_func_with_variable_args(func, input, config, run_manager, **kwargs)
324     if run_manager is not None and accepts_run_manager(func):
325         kwargs[&quot;run_manager&quot;] = run_manager
--&gt; 326     return func(input, **kwargs)  # type: ignore[call-arg]
327
328

/usr/lib/python3.10/json/__init__.py in loads(s, cls, object_hook, parse_float, parse_int, parse_constant, object_pairs_hook, **kw)
344             parse_int is None and parse_float is None and
345             parse_constant is None and object_pairs_hook is None and not kw):
--&gt; 346         return _default_decoder.decode(s)
347     if cls is None:
348         cls = JSONDecoder

/usr/lib/python3.10/json/decoder.py in decode(self, s, _w)
338         end = _w(s, end).end()
339         if end != len(s):
--&gt; 340             raise JSONDecodeError(&quot;Extra data&quot;, s, end)
341         return obj
342

JSONDecodeError: Extra data: line 9 column 1 (char 125)</code></pre></details>
## 较新的模型能够格式化输出
```python
model = ChatOpenAI(temperature=0)
chain = model | StrOutputParser() | json.loads

chain.invoke(challenge)
```

<details><summary>output：</summary><pre><code class="language-python"><code>{'poem1': {'title': 'Whispers of the Wind',
  'author': 'Emily Rivers',
  'first_line': 'Softly it comes, the whisper of the wind'},
 'poem2': {'title': 'Silent Serenade',
  'author': 'Jacob Moore',
  'first_line': 'In the stillness of night, a silent serenade'},
 'poem3': {'title': 'Dancing Shadows',
  'author': 'Sophia Anderson',
  'first_line': 'Shadows dance upon the walls, a secret ballet'}}</code></pre></details>
## fallback机制
```python
final_chain = simple_chain.with_fallbacks([chain])

final_chain.invoke(challenge)
```

<details><summary>output：</summary><pre><code class="language-python"><code>{'poem1': {'title': 'Whispers of the Wind',
  'author': 'Emily Rivers',
  'first_line': 'Softly it comes, the whisper of the wind'},
 'poem2': {'title': 'Silent Serenade',
  'author': 'Jacob Moore',
  'first_line': 'In the stillness of night, a silent serenade'},
 'poem3': {'title': 'Dancing Shadows',
  'author': 'Sophia Anderson',
  'first_line': 'Shadows dance upon the moonlit floor'}}</code></pre></details>
## 接口
```python
prompt = ChatPromptTemplate.from_template(
    "Tell me a short joke about {topic}"
)
model = ChatOpenAI()
output_parser = StrOutputParser()

chain = prompt | model | output_parser
```

### invoke接口
```python
chain.invoke({"topic": "bears"})
```

<details><summary>output：</summary><pre><code class="language-python"><code>&quot;Why don't bears wear shoes?\n\nBecause they have bear feet!&quot;</code></pre></details>
### batch接口
```python
chain.batch([{"topic": "bears"}, {"topic": "frogs"}])
```

<details><summary>output：</summary><pre><code class="language-python"><code>[&quot;Why don't bears wear shoes?\n\nBecause they have bear feet!&quot;,
 'Why did the frog take the bus to work?\n\nBecause his car got toad away!']</code></pre></details>
### stream接口
```python
for t in chain.stream({"topic": "bears"}):
    print(t)
```

<details><summary>output：</summary><pre><code class="language-python"><code>Why
 don
't
 bears
 wear
 shoes
?

Because
 they
 have
 bear
 feet
!</code></pre></details>
### 异步接口
```python
response = await chain.ainvoke({"topic": "bears"})
response
```

<details><summary>output：</summary><pre><code class="language-python"><code>&quot;Why don't bears wear shoes?\n\nBecause they have bear feet!&quot;</code></pre></details>
