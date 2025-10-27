从 `langchain.chat_models` 导入 `OpenAI` 的对话模型 `ChatOpenAI` 。 除去OpenAI以外，`langchain.chat_models` 还集成了其他对话模型，更多细节可以查看

[langchain 0.2.17 — 🦜🔗 LangChain 0.2.17](https://api.python.langchain.com/en/latest/langchain_api_reference.html#module-langchain.chat_models)

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

没有安装` langchain-openai `的话，先运行下面代码

```python
from langchain_openai import ChatOpenAI
```

也会这样调用

```python
from langchain_community.chat_models.openai import ChatOpenAI
```

接下来需要实例化一个 ChatOpenAI 类，可以在实例化时传入超参数来控制回答，例如 `temperature` 参数。

```python
# 这里将参数temperature设置为0.0，从而减少生成答案的随机性。
# 如果你想要每次得到不一样的有新意的答案，可以尝试调整该参数。
llm = ChatOpenAI(temperature=0.0, openai_api_base="https://xiaoai.plus/v1")
llm

```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="python" id="iOXbm" class="ne-codeblock language-python"><code>ChatOpenAI(client=&lt;openai.resources.chat.completions.Completions object at 0x000001B17F799BD0&gt;, async_client=&lt;openai.resources.chat.completions.AsyncCompletions object at 0x000001B17F79BA60&gt;, temperature=0.0, openai_api_key=SecretStr('**********'), openai_api_base='https://api.chatgptid.net/v1', openai_proxy='')</code></pre></details>
上面的 cell 假设 OpenAI API 密钥是在环境变量中设置的，如果希望手动指定API密钥，使用以下代码：

```python
llm = ChatOpenAI(temperature=0, openai_api_base="https://xiaoai.plus/v1",openai_api_key="sk-xxx")
llm
```

<br/>color2
可以看到，默认调用的是 ChatGPT-3.5 模型。另外，几种常用的超参数设置包括：

+ `model_name`：所要使用的模型，默认为 ‘`gpt-3.5-turbo`’，参数设置与 OpenAI 原生接口参数设置一致。
+ `temperature`：温度系数，取值同原生接口。
+ `openai_api_key`：OpenAI API key，如果不使用环境变量设置 API Key，也可以在实例化时设置。
+ `openai_proxy`：设置代理，如果不使用环境变量设置代理，也可以在实例化时设置。
+ `streaming`：是否使用流式传输，即逐字输出模型回答，默认为 False，此处不赘述
+ `max_tokens`：模型输出的最大 token 数，意义及取值同上。
+ `openai_api_base`：如果用的是代理，就把这个 url 填上

<br/>

当初始化了选择的`LLM`后，就可以尝试使用它！问一下“请你自我介绍一下自己！”

```python
output = llm.invoke("请你自我介绍一下自己！")
```

```python
output
```

<details class="lake-collapse"><summary id="ucd734aa9"><span class="ne-text">output：</span></summary><pre data-language="python" id="TPStQ" class="ne-codeblock language-python"><code>AIMessage(content='你好，我是一个智能助手，专注于为用户提供各种服务和帮助。我可以回答问题、提供信息、解决问题，帮助用户更高效地完成工作和生活。如果您有任何疑问或需要帮助，请随时告诉我，我会尽力帮助您。感谢您的使用！', response_metadata={'token_usage': {'completion_tokens': 104, 'prompt_tokens': 20, 'total_tokens': 124}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_b28b39ffa8', 'finish_reason': 'stop', 'logprobs': None})</code></pre></details>
## ModelLaboratory
> langchain 可以集成很多模型的调用
>

通过`LangChain`提供的`ModelLaboratory`（模型实验室），你可以测试并比较不同的模型。

下面是一段通过`ModelLaboratory`比较不同大模型的示例代码(需要确保已经安装`OpenAl`、`langchain-openai`、`Cohere`和`HuggingFace-Hub`库，同时在`.env`文件中已经配置`OpenAl_API_KEY`、`COHERRE_API_KEY`和`HUGGINGFACEHUB_API_TOKEN`。

```python
# 导入dotenv包，用于加载环境变量
from dotenv import load_dotenv
load_dotenv()

# 导入langchain_openai库中的OpenAI类，用于与OpenAI进行交互
from langchain_openai import OpenAI
# 导入langchain_community.llms中的Cohere和HuggingFaceHub类，用于使用Cohere和HuggingFace的模型
from langchain_community.llms import Cohere, HuggingFaceHub

# 初始化OpenAI、Cohere和HuggingFaceHub的实例，并设置温度参数（控制生成文本的创新性）
openai = OpenAI(temperature=0.1)
cohere = Cohere(model="command", temperature=0.1)
huggingface = HuggingFaceHub(repo_id="tiiuae/falcon-7b", model_kwargs={'temperature':0.1})

# 导入ModelLaboratory类，用于创建和管理多个语言模型
from langchain.model_laboratory import ModelLaboratory

# 创建一个模型实验室实例，整合了OpenAI、Cohere和HuggingFace的模型
model_lab = ModelLaboratory.from_llms([openai, cohere, huggingface])

# 使用模型实验室比较不同模型对同一个问题“百合花是来源自哪个国家?”的回答
model_lab.compare("百合花是来源自哪个国家?")
```

3个模型给出3种答案。很明显，`OpenAl`公司的`ChatGPT`（未指定具体模型,默认使用`GPT-3.5Turboinstruct`，这是上一代的模型）的答案最佳，`Cohere`的答案不正确。`HuggingFaceHub`的开源模型falcon-7b甚至只是将问题夏述一遍并作为回应，相当敷衍。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736757657194-84fd864e-8336-4c91-bd16-1da84317f5ff.png)

