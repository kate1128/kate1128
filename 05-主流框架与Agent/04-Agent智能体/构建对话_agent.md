> 如何构建 Agent：定义tools和functions --> 构建模型 --> 加入中间步骤结果 --> 加入记忆机制 --> 交互界面
>

#  逐步构建Agent
## 定义 `Tools` 和 `Function`
```python
# 导入tool包
from langchain.tools import tool
```

1. 使用 `@tool` 装饰器并指定输入格式

```python
# 导入所需的库
import requests
from pydantic import BaseModel, Field
import datetime

# 定义输入格式
class OpenMeteoInput(BaseModel):
    latitude: float = Field(..., description="要获取天气数据的位置的纬度") 
    longitude: float = Field(..., description="要获取天气数据的位置的经度") 

# 使用 @tool 装饰器并指定输入格式
@tool(args_schema=OpenMeteoInput)
def get_current_temperature(latitude: float, longitude: float) -> dict:
    """"根据给定的坐标位置获得温度"""

    # Open Meteo API 的URL
    BASE_URL = "https://api.open-meteo.com/v1/forecast"

    # 请求参数
    params = {
        'latitude': latitude,
        'longitude': longitude,
        'hourly': 'temperature_2m',
        'forecast_days': 1,
    }

    # 发送 API 请求
    response = requests.get(BASE_URL, params=params)

    # 检查响应状态码
    if response.status_code == 200:
        # 解析 JSON 响应
        results = response.json()
    else:
        # 处理请求失败的情况
        raise Exception(f"API Request failed with status code: {response.status_code}")

    # 获取当前 UTC 时间
    current_utc_time = datetime.datetime.utcnow()

    # 将时间字符串转换为 datetime 对象
    time_list = [datetime.datetime.fromisoformat(time_str.replace('Z', '+00:00')) for time_str in results['hourly']['time']]

    # 获取温度列表
    temperature_list = results['hourly']['temperature_2m']

    # 找到最接近当前时间的索引
    closest_time_index = min(range(len(time_list)), key=lambda i: abs(time_list[i] - current_utc_time))

    # 获取当前温度
    current_temperature = temperature_list[closest_time_index]

    # 返回当前温度的字符串形式
    return f'现在温度是 {current_temperature}°C'
```

2. 定义维基百科搜索的`tool`

```python
import wikipedia

# 定义维基百科搜索的tool
@tool
def search_wikipedia(query: str) -> str:
    """Run Wikipedia search and get page summaries."""
    page_titles = wikipedia.search(query)
    summaries = []
    for page_title in page_titles[: 3]: #取前三个页面标题
        try:
            #使用 wikipedia 模块的 page 函数，获取指定标题的维基百科页面对象。
            wiki_page =  wikipedia.page(title=page_title, auto_suggest=False) 
            # 获取页面摘要
            summaries.append(f"页面: {page_title}\n摘要: {wiki_page.summary}")
        except (
            self.wiki_client.exceptions.PageError,
            self.wiki_client.exceptions.DisambiguationError,
        ):
            pass
    if not summaries:
        return "维基百科没有搜索到合适的结果"
    return "\n\n".join(summaries)
```

3. 加入`tools`列表

```python
# 加入tools列表
tools = [get_current_temperature, search_wikipedia]
```

4. 将工具格式化为 OpenAI 函数

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.tools.render import format_tool_to_openai_function
from langchain.agents.output_parsers import OpenAIFunctionsAgentOutputParser
```

4. 创建处理链，将 `prompt`、`model` 和 `OpenAIFunctionsAgentOutputParser` 连接起来

```python
# 将工具格式化为 OpenAI 函数
functions = [format_tool_to_openai_function(f) for f in tools]

# 创建 ChatOpenAI 模型，设置温度为 0，绑定生成的 OpenAI 函数列表
model = ChatOpenAI(temperature=0).bind(functions=functions)

# 创建 ChatPromptTemplate，从消息模板中获取用户输入
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are helpful but sassy assistant"),
    ("user", "{input}"),
])

# 创建处理链，将 prompt、model 和 OpenAIFunctionsAgentOutputParser 连接起来
chain = prompt | model | OpenAIFunctionsAgentOutputParser()
```

5. 调用

```python
# 调用
result = chain.invoke({"input": "现在圣佛朗西斯科的温度是多少?"})
```

6. 查看调用的工具

```python
# 查看调用的工具
result.tool
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>'get_current_temperature'</code></pre></details>
7. 查看工具的输入

```python
# 查看工具的输入
result.tool_input
```

<details class="lake-collapse"><summary id="u382d0bd2"><span class="ne-text">output：</span></summary><pre data-language="json" id="y628T" class="ne-codeblock language-json"><code>{'latitude': 37.7749, 'longitude': -122.4194}</code></pre></details>
8. 查看工具

```python
result.tool
```

<details class="lake-collapse"><summary id="u4ef66197"><span class="ne-text">output：</span></summary><pre data-language="json" id="QU5e4" class="ne-codeblock language-json"><code>'get_current_temperature'</code></pre></details>
## 加入中间步骤结果
1. 创建 `ChatPromptTemplate`，从消息模板中获取用户输入

```python
# 创建 ChatPromptTemplate，从消息模板中获取用户输入
from langchain.prompts import MessagesPlaceholder
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个安全且乐于助人的助手"),
    ("user", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad") # 解析和验证关键字参数中的输入数据
])
```

2. 创建处理链，将 `prompt`、`model` 和 `OpenAIFunctionsAgentOutputParser` 连接起来

```python
# 创建处理链，将 prompt、model 和 OpenAIFunctionsAgentOutputParser 连接起来
chain = prompt | model | OpenAIFunctionsAgentOutputParser()
```

3. 调用

```python
# 调用
result1 = chain.invoke({
    "input": "现在圣佛朗西斯科的温度是多少?",
    "agent_scratchpad": []
})
```

4. 查看调用的工具

```python
# 查看调用的工具
result1.tool
```

<details class="lake-collapse"><summary id="u42b512ca"><span class="ne-text">output：</span></summary><pre data-language="json" id="M8DRL" class="ne-codeblock language-json"><code>'get_current_temperature'</code></pre></details>
4. 获取`tool`调用的结果

```python
# 获取tool调用的结果
observation = get_current_temperature(result1.tool_input)
```

5. 查看结果

```python
# 查看结果
observation
```

<details class="lake-collapse"><summary id="u9644048e"><span class="ne-text">output：</span></summary><pre data-language="json" id="b8pkY" class="ne-codeblock language-json"><code>'现在温度是 9.2°C'</code></pre></details>
6. 查看输出类型

```python
# 查看输出类型
type(result1)
```

<details class="lake-collapse"><summary id="u11cca840"><span class="ne-text">output：</span></summary><pre data-language="json" id="cypI0" class="ne-codeblock language-json"><code>langchain_core.agents.AgentActionMessageLog</code></pre></details>
```python
from langchain.agents.format_scratchpad import format_to_openai_functions
```

7. 查看调用日志

```python
# 查看调用日志
result1.message_log
```

<details class="lake-collapse"><summary id="uad01fa41"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="RBVKa" class="ne-codeblock language-json"><code>[AIMessage(content='', additional_kwargs={'function_call': {'arguments': '{&quot;latitude&quot;:37.7749,&quot;longitude&quot;:-122.4194}', 'name': 'get_current_temperature'}})]</code></pre></details>
8. 将`Function`和`Tool`的结果转换成`openai`函数格式

```python
# 将Function和Tool的结果转换成openai函数格式
format_to_openai_functions([(result1, observation), ])
```

<details class="lake-collapse"><summary id="ua477f915"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="iUxMZ" class="ne-codeblock language-json"><code>[AIMessage(content='', additional_kwargs={'function_call': {'arguments': '{&quot;latitude&quot;:37.7749,&quot;longitude&quot;:-122.4194}', 'name': 'get_current_temperature'}}),
 FunctionMessage(content='现在温度是 9.2°C', name='get_current_temperature')]</code></pre></details>
9. 构造成`Agent`的输入格式，`agent_scratchpad`是中间步骤的结果

```python
# 构造成Agent的输入格式，agent_scratchpad是中间步骤的结果
result2 = chain.invoke({
    "input": "现在圣佛朗西斯科的温度是多少?", 
    "agent_scratchpad": format_to_openai_functions([(result1, observation)])
})
```

```python
# 查看Agent的输出结果
result2
```

<details class="lake-collapse"><summary id="ud31c78b5"><span class="ne-text">output：</span></summary><pre data-language="json" id="tCmgq" class="ne-codeblock language-json"><code>AgentFinish(return_values={'output': '圣佛朗西斯科现在的温度是9.2°C。'}, log='圣佛朗西斯科现在的温度是9.2°C。')</code></pre></details>
10. 循环执行 `agent` 操作

```python
from langchain.schema.agent import AgentFinish

def run_agent(user_input):
    # 存储中间步骤的列表
    intermediate_steps = []

    # 循环执行 agent 操作
    while True:
        # 调用处理链的 invoke 方法，传递用户输入和中间步骤的 OpenAI 函数形式
        result = chain.invoke({
            "input": user_input,
            "agent_scratchpad": format_to_openai_functions(intermediate_steps)
        })

        # 如果结果是 AgentFinish 类型，则结束运行并返回结果
        if isinstance(result, AgentFinish):
            return result

        # 根据结果中的工具名称选择相应的工具函数
        tool = {
            "search_wikipedia": search_wikipedia,
            "get_current_temperature": get_current_temperature,
        }[result.tool]

        # 运行选定的工具函数，并获取观察结果
        observation = tool.run(result.tool_input)

        # 将结果和观察结果添加到中间步骤列表
        intermediate_steps.append((result, observation))
```

11. 定义一个运行 `agent` 的函数

```python
from langchain.schema.runnable import RunnablePassthrough

# 创建 RunnablePassthrough，用于将 agent_scratchpad存储的中间步骤结果转换为 OpenAI 函数形式
agent_chain = RunnablePassthrough.assign(
    agent_scratchpad=lambda x: format_to_openai_functions(x["intermediate_steps"])
) | chain
```

```python
# 定义一个运行 agent 的函数
def run_agent(user_input):
    # 存储中间步骤的列表
    intermediate_steps = []

    # 循环执行 agent 操作
    while True:
        # 通过 agent_chain 调用 invoke 方法，传递用户输入和中间步骤的列表
        result = agent_chain.invoke({
            "input": user_input,
            "intermediate_steps": intermediate_steps
        })

        # 如果结果是 AgentFinish 类型，则结束循环并返回结果
        if isinstance(result, AgentFinish):
            return result

        # 根据结果中的工具名称选择相应的工具函数
        tool = {
            "search_wikipedia": search_wikipedia,
            "get_current_temperature": get_current_temperature,
        }[result.tool]

        # 运行选定的工具函数，并获取观察结果
        observation = tool.run(result.tool_input)

        # 将结果和观察结果添加到中间步骤列表
        intermediate_steps.append((result, observation))
```

12. `run_agent`

```python
run_agent("圣弗朗西斯科的温度是多少？")
```

<details class="lake-collapse"><summary id="u0549055e"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="SVshB" class="ne-codeblock language-json"><code>AgentFinish(return_values={'output': '圣弗朗西斯科现在的温度是9.2°C。'}, log='圣弗朗西斯科现在的温度是9.2°C。')</code></pre></details>
13. `run_agent`

```python
run_agent("什么是langchain?")
```

<details class="lake-collapse"><summary id="ub78b2313"><span class="ne-text">output：</span></summary><pre data-language="json" id="hjS3X" class="ne-codeblock language-json"><code>AgentFinish(return_values={'output': 'LangChain是一个旨在简化使用大型语言模型（LLMs）创建应用程序的框架。作为一个语言模型集成框架，LangChain的用例主要涵盖了文档分析和摘要、聊天机器人以及代码分析等领域。'}, log='LangChain是一个旨在简化使用大型语言模型（LLMs）创建应用程序的框架。作为一个语言模型集成框架，LangChain的用例主要涵盖了文档分析和摘要、聊天机器人以及代码分析等领域。')</code></pre></details>
14. `run_agent`

```python
run_agent("你好！")
```

<details class="lake-collapse"><summary id="u35f55c06"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="jAewr" class="ne-codeblock language-json"><code>AgentFinish(return_values={'output': '你好！有什么可以帮助你的吗？'}, log='你好！有什么可以帮助你的吗？')</code></pre></details>


```python
# 使用AgentExecutor对agent进行封装
from langchain.agents import AgentExecutor
agent_executor = AgentExecutor(agent=agent_chain, tools=tools, verbose=True)
```

12. `invoke`

```python
agent_executor.invoke({"input": "什么是langchain?"})
```

<details class="lake-collapse"><summary id="u556d749e"><span class="ne-text">output：</span></summary><pre data-language="json" id="MrgmW" class="ne-codeblock language-json"><code>&gt; Entering new AgentExecutor chain...

Invoking: `search_wikipedia` with `{'query': 'Langchain'}`


Page: LangChain
Summary: LangChain is a framework designed to simplify the creation of applications using large language models (LLMs). As a language model integration framework, LangChain's use-cases largely overlap with those of language models in general, including document analysis and summarization, chatbots, and code analysis.

Page: OpenAI
Summary: OpenAI is a U.S. based artificial intelligence (AI) research organization founded in December 2015, researching artificial intelligence with the goal of developing &quot;safe and beneficial&quot; artificial general intelligence, which it defines as &quot;highly autonomous systems that outperform humans at most economically valuable work&quot;.
As one of the leading organizations of the AI spring, it has developed several large language models, advanced image generation models, and previously, released open-source models. Its release of ChatGPT has been credited with starting the AI spring.The organization consists of the non-profit OpenAI, Inc. registered in Delaware and its for-profit subsidiary OpenAI Global, LLC. It was founded by Ilya Sutskever, Greg Brockman, Trevor Blackwell, Vicki Cheung, Andrej Karpathy, Durk Kingma, Jessica Livingston, John Schulman, Pamela Vagata, and Wojciech Zaremba, with Sam Altman and Elon Musk serving as the initial board members. Microsoft provided OpenAI Global LLC with a $1 billion investment in 2019 and a $10 billion investment in 2023, with a significant portion of the investment in the form of computational resources on Microsoft's Azure cloud service.On November 17, 2023, the board removed Altman as CEO, while Brockman was removed as chairman and then resigned as president. Four days later, both returned after negotiations with the board, and most of the board members resigned. The new initial board included former Salesforce co-CEO Bret Taylor as chairman. It was also announced that Microsoft will have a non-voting board seat.

Page: DataStax
Summary: DataStax, Inc. is a real-time data for AI company based in Santa Clara, California. Its product Astra DB is a cloud database-as-a-service based on Apache Cassandra. DataStax also offers DataStax Enterprise (DSE), an on-premises database built on Apache Cassandra, and Astra Streaming, a messaging and event streaming cloud service based on Apache Pulsar. As of June 2022, the company has roughly 800 customers distributed in over 50 countries.LangChain 是一个旨在简化使用大型语言模型（LLMs）创建应用程序的框架。作为一个语言模型集成框架，LangChain 的用例主要涵盖了文档分析和摘要、聊天机器人以及代码分析等领域。

&gt; Finished chain.</code></pre></details>


```python
{'input': '什么是langchain?',
 'output': 'LangChain 是一个旨在简化使用大型语言模型（LLMs）创建应用程序的框架。作为一个语言模型集成框架，LangChain 的用例主要涵盖了文档分析和摘要、聊天机器人以及代码分析等领域。'}
```

13. `invoke`

```python
agent_executor.invoke({"input": "我的名字是bob"})
```

```python
> Entering new AgentExecutor chain...
你好，Bob！有什么可以帮助你的吗？

> Finished chain.
```

<details class="lake-collapse"><summary id="u5c20175d"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="rLkHq" class="ne-codeblock language-json"><code>{'input': '我的名字是bob', 'output': '你好，Bob！有什么可以帮助你的吗？'}</code></pre></details>
14. `invoke`

```python
agent_executor.invoke({"input": "我的名字是什么？"})
```

```python
> Entering new AgentExecutor chain...
你的名字是用户。

> Finished chain.
```

<details class="lake-collapse"><summary id="u0227164d"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="nY14T" class="ne-codeblock language-json"><code>{'input': '我的名字是什么？', 'output': '你的名字是用户。'}</code></pre></details>
## 加入对话历史
1. 加入用户的对话历史

```python
# 加入用户的对话历史
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are helpful but sassy assistant"),
    MessagesPlaceholder(variable_name="chat_history"),
    ("user", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad")
])
```

2. 构造`agent`的处理链

```python
# 构造agent的处理链
agent_chain = RunnablePassthrough.assign(
    agent_scratchpad= lambda x: format_to_openai_functions(x["intermediate_steps"])
) | prompt | model | OpenAIFunctionsAgentOutputParser()
```

3. 加入对话记忆模块

```python
# 加入对话记忆模块
from langchain.memory import ConversationBufferMemory
memory = ConversationBufferMemory(return_messages=True,memory_key="chat_history")
```

4. 封装`AgentExecutor`

```python
# 封装AgentExecutor
agent_executor = AgentExecutor(agent=agent_chain, tools=tools, verbose=True, memory=memory)
```

5. `invoke`

```python
agent_executor.invoke({"input": "我的名字是bob"})
```

<details class="lake-collapse"><summary id="ue3bdc417"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="python" id="Lqme3" class="ne-codeblock language-python"><code>&gt; Entering new AgentExecutor chain...
Nice to meet you, Bob! How can I assist you today?

&gt; Finished chain.</code></pre><pre data-language="json" id="uswcn" class="ne-codeblock language-json"><code>{'input': '我的名字是bob',
 'chat_history': [HumanMessage(content='my name is bob'),
  AIMessage(content='Nice to meet you, Bob! How can I assist you today?'),
  HumanMessage(content='我的名字是bob'),
  AIMessage(content='Nice to meet you, Bob! How can I assist you today?')],
 'output': 'Nice to meet you, Bob! How can I assist you today?'}</code></pre></details>
5. `invoke`

```python
agent_executor.invoke({"input": "我的名字是什么？"})
```

<details class="lake-collapse"><summary id="u994c5596"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="python" id="o0XJr" class="ne-codeblock language-python"><code>&gt; Entering new AgentExecutor chain...
你的名字是 Bob。有什么我可以帮助你的吗？

&gt; Finished chain.</code></pre><pre data-language="json" id="qzWZw" class="ne-codeblock language-json"><code>{'input': '我的名字是什么？',
 'chat_history': [HumanMessage(content='my name is bob'),
  AIMessage(content='Nice to meet you, Bob! How can I assist you today?'),
  HumanMessage(content='我的名字是bob'),
  AIMessage(content='Nice to meet you, Bob! How can I assist you today?'),
  HumanMessage(content='whats my name'),
  AIMessage(content='Your name is Bob. How can I assist you today, Bob?'),
  HumanMessage(content='我的名字是什么？'),
  AIMessage(content='你的名字是 Bob。有什么我可以帮助你的吗？')],
 'output': '你的名字是 Bob。有什么我可以帮助你的吗？'}</code></pre></details>
5. `invoke`

```python
agent_executor.invoke({"input": "圣弗朗西斯科现在的温度是多少？"})
```

<details class="lake-collapse"><summary id="u1549bbfd"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="python" id="oybZ2" class="ne-codeblock language-python"><code>&gt; Entering new AgentExecutor chain...

Invoking: `get_current_temperature` with `{'latitude': 37.7749, 'longitude': -122.4194}`


The current temperature is 9.1°CThe current temperature in San Francisco is 9.1°C. Is there anything else you would like to know?

&gt; Finished chain.</code></pre><pre data-language="json" id="HgYx2" class="ne-codeblock language-json"><code>{'input': '圣弗朗西斯科现在的温度是多少？',
 'chat_history': [HumanMessage(content='my name is bob'),
  AIMessage(content='Nice to meet you, Bob! How can I assist you today?'),
  HumanMessage(content='我的名字是bob'),
  AIMessage(content='Nice to meet you, Bob! How can I assist you today?'),
  HumanMessage(content='whats my name'),
  AIMessage(content='Your name is Bob. How can I assist you today, Bob?'),
  HumanMessage(content='我的名字是什么？'),
  AIMessage(content='你的名字是 Bob。有什么我可以帮助你的吗？'),
  HumanMessage(content='whats the weather in sf?'),
  AIMessage(content='The current temperature in San Francisco is 9.1°C. Is there anything else you would like to know?'),
  HumanMessage(content='圣弗朗西斯科现在的温度是多少？'),
  AIMessage(content='The current temperature in San Francisco is 9.1°C. Is there anything else you would like to know?')],
 'output': 'The current temperature in San Francisco is 9.1°C. Is there anything else you would like to know?'}</code></pre></details>
# 创建对话机器人
1. 自定义的工具（自由发挥）

```python
# 自定义的工具（自由发挥）
@tool
def create_your_own(query: str) -> str:
    """可以自定义的功能函数 """
    print(type(query))
    return query[::-1]
```

2. 将之前定义的工具加入工具列表

```python
# 将之前定义的工具加入工具列表
tools = [get_current_temperature, search_wikipedia, create_your_own]
```

```python
import panel as pn  # GUI
pn.extension()
import panel as pn
import param

# 定义 cbfs 类
class cbfs(param.Parameterized):
    
    # 初始化函数
    def __init__(self, tools, **params):
        super(cbfs, self).__init__(**params)
        self.panels = []  # 存储 GUI 面板
        self.functions = [format_tool_to_openai_function(f) for f in tools]  # 将tools格式化为 OpenAI 函数
        self.model = ChatOpenAI(temperature=0).bind(functions=self.functions)  # 创建 ChatOpenAI 模型
        self.memory = ConversationBufferMemory(return_messages=True, memory_key="chat_history")  # 创建 ConversationBufferMemory
        self.prompt = ChatPromptTemplate.from_messages([
            ("system", "You are helpful but sassy assistant"),
            MessagesPlaceholder(variable_name="chat_history"),
            ("user", "{input}"),
            MessagesPlaceholder(variable_name="agent_scratchpad")
        ])  # 创建 ChatPromptTemplate
        self.chain = RunnablePassthrough.assign(
            agent_scratchpad=lambda x: format_to_openai_functions(x["intermediate_steps"])
        ) | self.prompt | self.model | OpenAIFunctionsAgentOutputParser()  # 创建处理链
        self.qa = AgentExecutor(agent=self.chain, tools=tools, verbose=False, memory=self.memory)  # 创建 AgentExecutor
    
    # 对话链函数
    def convchain(self, query):
        if not query:
            return
        inp.value = ''
        result = self.qa.invoke({"input": query})
        self.answer = result['output']
        self.panels.extend([
            pn.Row('User:', pn.pane.Markdown(query, width=450)),
            pn.Row('ChatBot:', pn.pane.Markdown(self.answer, width=450, styles={'background-color': '#F6F6F6'}))
        ])
        return pn.WidgetBox(*self.panels, scroll=True)

    # 清除历史记录函数
    def clr_history(self, count=0):
        self.chat_history = []
        return
```

3. 创建文本输入框组件

```python
# 创建 cbfs 对象，传递工具列表
cb = cbfs(tools)

# 创建文本输入框组件
inp = pn.widgets.TextInput(placeholder='请输入文本...')

# 将 convchain 方法与输入框进行绑定，形成对话链
conversation = pn.bind(cb.convchain, inp) 

# 创建第一个选项卡组件
tab1 = pn.Column(
    pn.Row(inp),  # 显示文本输入框
    pn.layout.Divider(),
    pn.panel(conversation, loading_indicator=True, height=400),  # 显示对话链面板
    pn.layout.Divider(),
)

# 创建仪表板，包含标题和选项卡
dashboard = pn.Column(
    pn.Row(pn.pane.Markdown('# 对话机器人')),  # 显示标题
    pn.Tabs(('发送', tab1))  # 创建选项卡
)

# 显示仪表板
dashboard
```

4. 聊天机器人效果展示如下：

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/figures/chat_bot.png)

# 英文版提示
总结一下构建对话机器人的流程：

定义`tools`和`functions` --> 构建模型 --> 加入中间步骤结果 --> 加入记忆机制 --> 交互界面

```python
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are helpful but sassy assistant"),
    MessagesPlaceholder(variable_name="chat_history"),
    ("user", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad")
])
```

```python
# 构造agent的处理链
agent_chain = RunnablePassthrough.assign(
    agent_scratchpad= lambda x: format_to_openai_functions(x["intermediate_steps"])
) | prompt | model | OpenAIFunctionsAgentOutputParser()
```

```python
# 加入对话记忆模块
from langchain.memory import ConversationBufferMemory
memory = ConversationBufferMemory(return_messages=True,memory_key="chat_history")
```

```python
# 封装AgentExecutor
agent_executor = AgentExecutor(agent=agent_chain, tools=tools, verbose=True, memory=memory)
```

```python
agent_executor.invoke({"input": "my name is bob"})
```

```python
> Entering new AgentExecutor chain...
Nice to meet you, Bob! How can I assist you today?

> Finished chain.
```

```python
{'input': 'my name is bob',
 'chat_history': [HumanMessage(content='my name is bob'),
  AIMessage(content='Nice to meet you, Bob! How can I assist you today?')],
 'output': 'Nice to meet you, Bob! How can I assist you today?'}
```

```python
agent_executor.invoke({"input": "what is my name"})
```

```python
> Entering new AgentExecutor chain...
Your name is Bob. How can I assist you today, Bob?

> Finished chain.
```

<details class="lake-collapse"><summary id="u9b236e85"><span class="ne-text">output：</span></summary><pre data-language="json" id="drAPY" class="ne-codeblock language-json"><code>{'input': 'what is my name',
 'chat_history': [HumanMessage(content='my name is bob'),
  AIMessage(content='Nice to meet you, Bob! How can I assist you today?'),
  HumanMessage(content='what is my name'),
  AIMessage(content='Your name is Bob. How can I assist you today, Bob?')],
 'output': 'Your name is Bob. How can I assist you today, Bob?'}</code></pre></details>


```python
agent_executor.invoke({"input": "whats the weather in sf?"})
```

```python
> Entering new AgentExecutor chain...

Invoking: `get_current_temperature` with `{'latitude': 37.7749, 'longitude': -122.4194}`


现在温度是 9.5°CThe current temperature in San Francisco is 9.5°C.

> Finished chain.
```

<details class="lake-collapse"><summary id="u4e730202"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="W4Drl" class="ne-codeblock language-json"><code>{'input': 'whats the weather in sf?',
  'chat_history': [HumanMessage(content='my name is bob'),
  AIMessage(content='Nice to meet you, Bob! How can I assist you today?'),
  HumanMessage(content='what is my name'),
  AIMessage(content='Your name is Bob. How can I assist you today, Bob?'),
  HumanMessage(content='whats the weather in sf?'),
  AIMessage(content='The current temperature in San Francisco is 9.5°C.')],
  'output': 'The current temperature in San Francisco is 9.5°C.'}</code></pre></details>
