# 初始化 OpenAI Key
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
openai.api_key = os.environ['OPENAI_API_KEY']  
```

# LangChain 内置工具
```python
from langchain.agents import load_tools, initialize_agent
from langchain.agents import AgentType
from langchain.python import PythonREPL
from langchain.chat_models.openai import ChatOpenAI
```

## 使用 llm-match 和 wikipenia 工具
+ 默认密钥`openai_api_key`为环境变量`OPENAI_API_KEY`。因此在运行以下代码之前，确保你已经设置环境变量`OPENAI_API_KEY`。如果还没有密钥，请[获取你的API Key](https://platform.openai.com/account/api-keys) 。
+ 默认模型`model_name`为`gpt-3.5-turbo`。
+ 更多关于模型默认参数请查看[这里](https://github.com/hwchase17/langchain/blob/master/langchain/chat_models/openai.py)。

```python
# 参数temperature设置为0.0，从而减少生成答案的随机性。
llm = ChatOpenAI(temperature=0,openai_api_base="https://xiaoai.plus/v1")
```

## 加载工具包
+ `llm-math` 工具结合语言模型和计算器用以进行数学计算
+ `wikipedia`工具通过API连接到wikipedia进行搜索查询。

```python
tools = load_tools(
    ["llm-math","wikipedia"], 
    llm=llm #第一步初始化的模型
)
```

## 初始化代理
+ `agent`: 代理类型。这里使用的是`AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION`。其中`CHAT`代表代理模型为针对对话优化的模型，`REACT`代表针对REACT设计的提示模版。
+ `handle_parsing_errors`: 是否处理解析错误。当发生解析错误时，将错误信息返回给大模型，让其进行纠正。
+ `verbose`: 是否输出中间步骤结果。

```python
agent= initialize_agent(
    tools, #第二步加载的工具
    llm, #第一步初始化的模型
    agent=AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION,  #代理类型
    handle_parsing_errors=True, #处理解析错误
    verbose = True #输出中间步骤
)
```

## 使用代理回答数学问题
```python
agent("计算300的25%，思考过程请使用中文。") 
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736503357108-5a103689-61dd-41ce-abf1-a48bfb0c522d.png)

✅ **总结**

1. 模型对于接下来需要做什么，给出思考（Thought） 

**<font style="color:green;">思考</font>**<font style="color:green;">：我们需要计算300的25%，这个过程中需要用到乘法和除法。</font>

2. 模型基于思考采取行动（Action）

**<font style="color:green;">行动</font>**<font style="color:green;">: 使用计算器（calculator），输入300*0.25</font>

3. 模型得到观察（Observation）

**<font style="color:green;">观察</font>**<font style="color:green;">：答案: 75.0</font>

4. 基于观察，模型对于接下来需要做什么，给出思考（Thought）

**<font style="color:green;">思考</font>**<font style="color:green;">: 我们的问题有了答案 </font>

5. 给出最终答案（Final Answer）

**<font style="color:green;">最终答案</font>**<font style="color:green;">: 75.0 </font>

6. 以字典的形式给出最终答案。

## Tom 的书
```python
question = "Tom M. Mitchell is an American computer scientist \
and the Founders University Professor at Carnegie Mellon University (CMU)\
what book did he write?"
agent(question) 
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736503438628-e97c15f6-a339-40e2-8d83-d2b4c1a42462.png)

✅ **总结**

1. 模型对于接下来需要做什么，给出思考（Thought） 

**<font style="color:green;">思考</font>**<font style="color:green;">：我应该使用维基百科去搜索。</font>

2. 模型基于思考采取行动（Action）

**<font style="color:green;">行动</font>**<font style="color:green;">: 使用维基百科，输入Tom M. Mitchell</font>

3. 模型得到观察（Observation）

**<font style="color:green;">观测</font>**<font style="color:green;">: 页面: Tom M. Mitchell，页面: Tom Mitchell (澳大利亚足球运动员)</font>

4. 基于观察，模型对于接下来需要做什么，给出思考（Thought）

**<font style="color:green;">思考</font>**<font style="color:green;">: Tom M. Mitchell写的书是Machine Learning </font>

5. 给出最终答案（Final Answer）

**<font style="color:green;">最终答案</font>**<font style="color:green;">: Machine Learning </font>

6. 以字典的形式给出最终答案。



值得注意的是，模型每次运行推理的过程可能存在差异，但最终的结果一致。

# 使用 PythonREPLTool 工具
## 创建 python 代理
```python
from langchain_experimental.agents.agent_toolkits.python.base import create_python_agent
from langchain_experimental.tools.python.tool import PythonREPLTool

agent = create_python_agent(
    llm,  #使用前面一节已经加载的大语言模型
    tool=PythonREPLTool(), #使用Python交互式环境工具（REPLTool）
    verbose=True #输出中间步骤
)
```

## 使用代理对顾客名字进行排序
```python
customer_list = ["小明","小黄","小红","小蓝","小橘","小绿",]
```

```python
agent.run(f"""在这些客户名字转换为拼音\
并打印输出列表： {customer_list}\
思考过程请使用中文。""") 
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736735065882-a75af654-3f91-42dd-897d-7def1636aeb9.png)

## 使用调试模式
在调试（debug）模式下再次运行，我们可以把上面的6步分别对应到下面的具体流程

1. 模型对于接下来需要做什么，给出思考（Thought）
    - <font style="color:green;">[chain/start] [1:chain:AgentExecutor] Entering Chain run with input</font>
    - <font style="color:green;">[chain/start] [1:chain:AgentExecutor > 2:chain:LLMChain] Entering Chain run with input</font>
    - <font style="color:green;">[llm/start] [1:chain:AgentExecutor > 2:chain:LLMChain > 3:llm:ChatOpenAI] Entering LLM run with input</font>
    - <font style="color:green;">[llm/end] [1:chain:AgentExecutor > 2:chain:LLMChain > 3:llm:ChatOpenAI] [12.25s] Exiting LLM run with output</font>
    - <font style="color:green;">[chain/end] [1:chain:AgentExecutor > 2:chain:LLMChain] [12.25s] Exiting Chain run with output</font>
2. 模型基于思考采取行动（Action), 因为使用的工具不同，Action的输出也和之前有所不同，这里输出的为python代码
    - <font style="color:green;">[tool/start] [1:chain:AgentExecutor > 4:tool:Python REPL] Entering Tool run with input</font>
    - <font style="color:green;">[tool/end] [1:chain:AgentExecutor > 4:tool:Python REPL] [2.2239999999999998ms] Exiting Tool run with output</font>
3. 模型得到观察（Observation）   
    - <font style="color:green;">[chain/start] [1:chain:AgentExecutor > 5:chain:LLMChain] Entering Chain run with input</font>
4. 基于观察，模型对于接下来需要做什么，给出思考（Thought）   
    - <font style="color:green;">[llm/start] [1:chain:AgentExecutor > 5:chain:LLMChain > 6:llm:ChatOpenAI] Entering LLM run with input</font>
    - <font style="color:green;">[llm/end] [1:chain:AgentExecutor > 5:chain:LLMChain > 6:llm:ChatOpenAI] [6.94s] Exiting LLM run with output</font>
5. 给出最终答案（Final Answer） 
    - <font style="color:green;">[chain/end] [1:chain:AgentExecutor > 5:chain:LLMChain] [6.94s] Exiting Chain run with output</font>
6. 返回最终答案。
    - <font style="color:green;">[chain/end] [1:chain:AgentExecutor] [19.20s] Exiting Chain run with output</font>

```python
import langchain
langchain.debug=True
agent.run(f"""在这些客户名字转换为拼音\
并打印输出列表： {customer_list}\
思考过程请使用中文。""") 
langchain.debug=False
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736735100076-ef82c1fc-b38b-4e15-b266-0b4634e50900.png)

# 定义自己的工具并在代理中使用
## 创建和使用自定义时间工具
```python
# 导入tool函数装饰器
from langchain.agents import tool
from datetime import date
```

## 使用 tool 装饰器构建自定义工具
tool函数装饰器可以应用用于任何函数，将函数转化为LongChain工具，使其成为代理可调用的工具。

我们需要给函数加上非常详细的文档字符串, 使得代理知道在什么情况下、如何使用该函数/工具。

比如下面的函数`time`,我们加上了详细的文档字符串

```python
"""
返回今天的日期，用于任何与获取今天日期相关的问题。
输入应该始终是一个空字符串，该函数将始终返回今天的日期。
任何日期的计算应该在此函数之外进行。
"""

```

```python
@tool
def time(text: str) -> str:
    """
    返回今天的日期，用于任何需要知道今天日期的问题。\
    输入应该总是一个空字符串，\
    这个函数将总是返回今天的日期，任何日期计算应该在这个函数之外进行。
    """
    return str(date.today())
```

## 初始化这个代理
```python
agent= initialize_agent(
    tools=[time], #将刚刚创建的时间工具加入代理
    llm=llm, #初始化的模型
    agent=AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION,  #代理类型
    handle_parsing_errors=True, #处理解析错误
    verbose = True #输出中间步骤
)
```

## 使用代理访问今天的日期
```python
agent("今天的日期是？") 
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736503786171-770ab362-c3ee-4a18-8634-e548ebfcdffb.png)

✅ 总结

<br/>color2
1. 模型对于接下来需要做什么，给出思考（Thought） 

**<font style="color:green;">思考</font>**<font style="color:green;">：我需要使用 time 工具来获取今天的日期</font>

2. 模型基于思考采取行动（Action), 因为使用的工具不同，Action的输出也和之前有所不同，这里输出的为python代码

**<font style="color:green;">行动</font>**<font style="color:green;">: 使用time工具，输入为空字符串</font>

3. 模型得到观察（Observation）

**<font style="color:green;">观测</font>**<font style="color:green;">: 2025-01-10</font>

4. 基于观察，模型对于接下来需要做什么，给出思考（Thought）

**<font style="color:green;">思考</font>**<font style="color:green;">: 我已成功使用 time 工具检索到了今天的日期</font>

5. 给出最终答案（Final Answer）

**<font style="color:green;">最终答案</font>**<font style="color:green;">: 今天的日期是2025-01-10.</font>

6. 返回最终答案。

<br/>

