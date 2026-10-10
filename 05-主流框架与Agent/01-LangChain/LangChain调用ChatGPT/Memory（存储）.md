与那些语言模型进行交互的时候，他们不会记得你之前和他进行的交流内容，这在我们构建一些应用程序（如聊天机器人）的时候，是一个很大的问题 -- 显得不够智能！因此，在本节中我们将介绍 LangChain 中的储存模块，即如何将先前的对话嵌入到语言模型中的，使其具有连续对话的能力。

当使用 LangChain 中的储存(Memory)模块时，它可以帮助保存和管理历史聊天消息，以及构建关于特定实体的知识。这些组件可以跨多轮对话储存信息，并允许在对话期间跟踪特定信息和上下文。

LangChain 提供了多种储存类型。其中，缓冲区储存允许保留最近的聊天消息，摘要储存则提供了对整个对话的摘要。实体储存 则允许在多轮对话中保留有关特定实体的信息。这些记忆组件都是模块化的，可与其他组件组合使用，从而增强机器人的对话管理能力。储存模块可以通过简单的API调用来访问和更新，允许开发人员更轻松地实现对话历史记录的管理和维护。

此次课程主要介绍其中四种储存模块，其他模块可查看文档学习。

+ 对话缓存储存 (ConversationBufferMemory）
+ 对话缓存窗口储存 (ConversationBufferWindowMemory）
+ 对话令牌缓存储存 (ConversationTokenBufferMemory）
+ 对话摘要缓存储存 (ConversationSummaryBufferMemory）

在LangChain中，储存 指的是大语言模型（LLM）的短期记忆。为什么是短期记忆？那是因为LLM训练好之后 (获得了一些长期记忆)，它的参数便不会因为用户的输入而发生改变。当用户与训练好的LLM进行对话时，LLM会暂时记住用户的输入和它已经生成的输出，以便预测之后的输出，而模型输出完毕后，它便会“遗忘”之前用户的输入和它的输出。因此，之前的这些信息只能称作为LLM的短期记忆。  

为了延长LLM短期记忆的保留时间，则需要借助一些外部储存方式来进行记忆，以便在用户与LLM对话中，LLM能够尽可能的知道用户与它所进行的历史对话信息。

# 设置 OpenAIKey
```python
import os

import openai
from dotenv import find_dotenv, load_dotenv

# 读取本地/项目的环境变量。

# find_dotenv()寻找并定位.env文件的路径
# load_dotenv()读取该.env文件，并将其中的环境变量加载到当前的运行环境中
# 如果你设置的是全局的环境变量，这行代码则没有任何作用。
_ = load_dotenv(find_dotenv())

# 获取环境变量 OPENAI_API_KEY
openai.api_key = os.environ["OPENAI_API_KEY"]
```

# 对话缓存存储
### 初始化对话模型
```python
from langchain.chains.conversation.base import ConversationChain
from langchain_community.chat_models import ChatOpenAI
from langchain.memory import ConversationBufferMemory
```

```python
# 这里我们将参数temperature设置为0.0，从而减少生成答案的随机性。
# 如果你想要每次得到不一样的有新意的答案，可以尝试增大该参数。
llm = ChatOpenAI(temperature=0.0,openai_api_base="https://xiaoai.plus/v1") 

memory = ConversationBufferMemory()

# 新建一个 ConversationChain Class 实例
# verbose参数设置为True时，程序会输出更详细的信息，以提供更多的调试或运行时信息。
# 相反，当将verbose参数设置为False时，程序会以更简洁的方式运行，只输出关键的信息。
conversation = ConversationChain(llm=llm, memory = memory, verbose=True )
```

### 第一轮对话
```python
conversation.predict(input="Hi, my name is Andrew")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736500227647-06f1641e-9d73-4f29-a735-c6dafb9e67fd.png)

### 第二轮对话
```python
conversation.predict(input="What is 1+1?")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736500261023-6edd1d75-d097-48da-ae3b-56cb4249408f.png)

### 第三轮对话
```python
conversation.predict(input="What is my name?")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736500292431-fc161764-5a0f-42c1-ab91-d9a3624ef6f9.png)

### 查看缓存存储
存储存储-->储存了当前为止所有的对话信息

```python
print(memory.buffer) 
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736500340517-f6629a57-edd0-4714-908c-11c05bc625b1.png)

历史缓存中的消息：

```python
print(memory.load_memory_variables({}))
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="python" id="iOXbm" class="ne-codeblock language-python"><code>{'history': &quot;Human: Hi, my name is Andrew\nAI: Hello Andrew! It's nice to meet you. How can I assist you today?\nHuman: What is 1+1?\nAI: 1+1 is equal to 2. Is there anything else you would like to know?\nHuman: What is my name?\nAI: Your name is Andrew. Is there anything else you would like to know or discuss?&quot;}</code></pre></details>
### 直接添加内容到缓存
```python
memory = ConversationBufferMemory()  # 新建一个空的对话缓存记忆
memory.save_context({"input": "Hi"}, {"output": "What's up"})  # 向缓存区添加指定对话的输入输出
print(memory.buffer)  # 查看缓存区结果
```

<details class="lake-collapse"><summary id="uf6c34afc"><span class="ne-text">output：</span></summary><pre data-language="python" id="WdZEq" class="ne-codeblock language-python"><code>Human: Hi
AI: What's up</code></pre></details>
```python
print(memory.load_memory_variables({}))# 再次加载记忆变量
```

<details class="lake-collapse"><summary id="u2259ccb9"><span class="ne-text">output：</span></summary><pre data-language="python" id="u5wun" class="ne-codeblock language-python"><code>{'history': &quot;Human: Hi\nAI: What's up&quot;}</code></pre></details>
继续添加新的内容，对话历史都保存下来在了！

```python
memory.save_context({"input": "Not much, just hanging"}, {"output": "Cool"})
```

```python
memory.load_memory_variables({})
```

<details class="lake-collapse"><summary id="u94b1ab48"><span class="ne-text">output：</span></summary><pre data-language="python" id="KTy9p" class="ne-codeblock language-python"><code>{'history': &quot;Human: Hi\nAI: What's up\nHuman: Not much, just hanging\nAI: Cool&quot;}</code></pre></details>
### 中文
中文很拉跨，记不住名字

## 对话缓存窗口储存
随着对话变得越来越长，所需的内存量也变得非常长。将大量的tokens发送到LLM的成本，也会变得更加昂贵,这也就是为什么API的调用费用，通常是基于它需要处理的tokens数量而收费的。

针对以上问题，LangChain也提供了几种方便的储存方式来保存历史对话。其中，对话缓存窗口储存只保留一个窗口大小的对话。它只使用最近的n次交互。这可以用于保持最近交互的滑动窗口，以便缓冲区不会过大

### 添加两轮对话到窗口缓存
```python
from langchain.memory import ConversationBufferWindowMemory

# k 为窗口参数，k=1表明只保留一个对话记忆
memory = ConversationBufferWindowMemory(k=1)  
```

```python
# 向memory添加两轮对话
memory.save_context({"input": "Hi"}, {"output": "What's up"})
memory.save_context({"input": "Not much, just hanging"}, {"output": "Cool"})
```

```python
# 并查看记忆变量当前的记录
memory.load_memory_variables({})
```

<details class="lake-collapse"><summary id="u3d1425f7"><span class="ne-text">output：</span></summary><pre data-language="python" id="SwNSr" class="ne-codeblock language-python"><code>{'history': 'Human: Not much, just hanging\nAI: Cool'}</code></pre></details>
### 在对话链中应用窗口缓存
```python
llm = ChatOpenAI(temperature=0.0,openai_api_base="https://xiaoai.plus/v1")
memory = ConversationBufferWindowMemory(k=1)
conversation = ConversationChain(llm=llm, memory=memory, verbose=False  )
```

注意此处！由于这里用的是一个窗口的记忆，因此只能保存一轮的历史消息，因此AI并不能知道你第一轮对话中提到的名字，他最多只能记住上一轮（第二轮）的对话信息

```python
conversation.predict(input="Hi, my name is Andrew")
```

<details class="lake-collapse"><summary id="u0979ecc1"><span class="ne-text">output：</span></summary><pre data-language="python" id="Xty15" class="ne-codeblock language-python"><code>&quot;Hello Andrew! I'm an AI designed to help with a wide range of topics and conversations. How can I assist you today?&quot;</code></pre></details>
```python
conversation.predict(input="What is 1+1?")
```

<details class="lake-collapse"><summary id="uc9a33e6b"><span class="ne-text">output：</span></summary><pre data-language="python" id="VIJwl" class="ne-codeblock language-python"><code>&quot;1 + 1 equals 2. It's a basic arithmetic operation that results in the sum of two single-digit numbers. Is there anything else you'd like to know or discuss?&quot;</code></pre></details>
```python
conversation.predict(input="What is my name?")
```

<details class="lake-collapse"><summary id="u0c545ab1"><span class="ne-text">output：</span></summary><pre data-language="python" id="o9Qsr" class="ne-codeblock language-python"><code>&quot;I'm sorry, but as an AI, I do not have access to personal information about individuals unless it has been provided to me during our conversation. Would you like to share your name with me?&quot;</code></pre></details>
## 对话 token 缓存存储
使用对话token缓存记忆，内存将限制保存的token数量。如果token数量超出指定数目，它会切掉这个对话的早期部分

以保留与最近的交流相对应的token数量，但不超过token限制。

```python
from langchain.memory import ConversationTokenBufferMemory
```

添加对话到Token缓存储存,限制token数量，进行测试

```python
memory = ConversationTokenBufferMemory(llm=llm, max_token_limit=30)
memory.save_context({"input": "AI is what?!"}, {"output": "Amazing!"})
memory.save_context({"input": "Backpropagation is what?"}, {"output": "Beautiful!"})
memory.save_context({"input": "Chatbots are what?"}, {"output": "Charming!"})
```

```python
memory.load_memory_variables({})
```

<details class="lake-collapse"><summary id="u7e494f93"><span class="ne-text">output：</span></summary><pre data-language="python" id="dhzmq" class="ne-codeblock language-python"><code>{'history': 'AI: Beautiful!\nHuman: Chatbots are what?\nAI: Charming!'}</code></pre></details>
可以看到前面超出的的token已经被舍弃了

### 补充
ChatGPT使用一种基于字节对编码（Byte Pair Encoding，BPE）的方法来进行tokenization（将输入文本拆分为token）。BPE是一种常见的tokenization技术，它将输入文本分割成较小的子词单元。 

OpenAI在其官方GitHub上公开了一个最新的开源Python库 [tiktoken](https://github.com/openai/tiktoken)，这个库主要是用来计算tokens数量的。相比较HuggingFace的tokenizer，其速度提升了好几倍。

具体token计算方式,特别是汉字和英文单词的token区别，具体课参考[知乎文章](https://www.zhihu.com/question/594159910) 。

## 对话摘要缓存存储
对话摘要缓存储存，**使用LLM编写到目前为止历史对话的摘要**，并将其保存

```python
from langchain.memory import ConversationSummaryBufferMemory
```

### 使用对话摘要缓存储存
创建一个长字符串，其中包含某人的日程安排

```python
# 创建一个长字符串
schedule = "There is a meeting at 8am with your product team. \
You will need your powerpoint presentation prepared. \
9am-12pm have time to work on your LangChain \
project which will go quickly because Langchain is such a powerful tool. \
At Noon, lunch at the italian resturant with a customer who is driving \
from over an hour away to meet you to understand the latest in AI. \
Be sure to bring your laptop to show the latest LLM demo."

# 使用对话摘要缓存记忆
llm = ChatOpenAI(temperature=0.0,openai_api_base="https://xiaoai.plus/v1")
memory = ConversationSummaryBufferMemory(llm=llm, max_token_limit=100) 
memory.save_context({"input": "Hello"}, {"output": "What's up"})
memory.save_context({"input": "Not much, just hanging"}, {"output": "Cool"})
memory.save_context(
    {"input": "What is on the schedule today?"}, {"output": f"{schedule}"}
)
```

```python
print(memory.load_memory_variables({})['history'])
```

<details class="lake-collapse"><summary id="u458b48cf"><span class="ne-text">output：</span></summary><pre data-language="python" id="PaCha" class="ne-codeblock language-python"><code>System: The human and AI exchange greetings and discuss the day's schedule, including a meeting with the product team, work on the LangChain project, and a lunch meeting with a customer interested in AI. The AI emphasizes the power of LangChain as a tool.</code></pre></details>
### 基于对话摘要缓存储存的对话链
基于上面的对话摘要缓存储存，新建一个对话链

```python
conversation = ConversationChain(llm=llm, memory=memory, verbose=True)
```

```python
conversation.predict(input="What would be a good demo to show?")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736501273688-3c7302f0-8225-49d5-9c14-bfd9c8988c9d.png)

<details class="lake-collapse"><summary id="u137e6353"><span class="ne-text">output：</span></summary><pre data-language="python" id="LNCH4" class="ne-codeblock language-python"><code>'For the meeting with the product team, a demo showcasing the latest updates to the user interface and the new features added to the existing product would be impressive. For the LangChain project work time, a demo of the progress made on implementing the translation algorithms and the integration with external APIs would be beneficial. And for the lunch appointment with the customer interested in AI, a demo highlighting the natural language processing capabilities and the machine learning models built into the AI platform would be engaging.'</code></pre></details>
```python
print(memory.load_memory_variables({})['history'])
```

<details class="lake-collapse"><summary id="uabd78bec"><span class="ne-text">output：</span></summary><pre data-language="python" id="W3iWM" class="ne-codeblock language-python"><code>System: The human greets the AI and asks about the day's schedule. The AI informs the human about the upcoming meeting with the product team, time to work on the LangChain project, and a lunch appointment with a customer interested in AI. The AI suggests demos to show, including updates to the user interface and new features for the product team, progress on translation algorithms and API integration for LangChain project, and highlighting natural language processing and machine learning models for the customer interested in AI.</code></pre></details>
