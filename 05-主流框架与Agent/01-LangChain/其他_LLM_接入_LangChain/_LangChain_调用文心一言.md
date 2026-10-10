同样可以通过 LangChain 框架来调用百度文心大模型，以将文心模型接入到的应用框架中。

## 自定义 LLM 接入 langchain
在老版本中，LangChain 是不直接支持文心调用的，需要自定义一个支持文心模型调用的 LLM。在这里为了向用户展示如何自定义 LLM，在[LangChain自定义 LLM](https://www.yuque.com/qiaokate/su87gb/pag54tm6sopqgh0g)中，简述了这种方法，也可参考

[Custom LLM | 🦜️🔗 LangChain](https://python.langchain.com/v0.1/docs/modules/model_io/llms/custom_llm/)

此处，可以直接调用已自定义好的 Wenxin_LLM，具体如何封装 Wenxin_LLM 见`wenxin_llm.py`。**注新版 LangChain 可以直接调用文心千帆 API，更推荐使用下一部分的代码来调用文心一言模型**

```python
# 需要下载源码
from wenxin_llm import Wenxin_LLM
```

希望像调用 ChatGPT 那样直接将秘钥存储在 .env 文件中，并将其加载到环境变量，从而隐藏秘钥的具体细节，保证安全性。因此，需要在 .env 文件中配置 `QIANFAN_AK` 和 `QIANFAN_SK`，并使用以下代码加载：

```python
from dotenv import find_dotenv, load_dotenv
import os

# 读取本地/项目的环境变量。

# find_dotenv()寻找并定位.env文件的路径
# load_dotenv()读取该.env文件，并将其中的环境变量加载到当前的运行环境中
# 如果你设置的是全局的环境变量，这行代码则没有任何作用。
_ = load_dotenv(find_dotenv())

# 获取环境变量 API_KEY
wenxin_api_key = os.environ["QIANFAN_AK"]
wenxin_secret_key = os.environ["QIANFAN_SK"]
```

```python
llm = Wenxin_LLM(api_key=wenxin_api_key, secret_key=wenxin_secret_key, system="你是一个助手！")
```

```python
llm.invoke("你好，请你自我介绍一下！")
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="python" id="iOXbm" class="ne-codeblock language-python"><code>[INFO] [03-31 22:12:53] openapi_requestor.py:316 [t:27812]: requesting llm api endpoint: /chat/eb-instant


1
2


'你好！我是助手，负责协助您完成各种任务。我具备快速响应、高效执行和灵活适应的能力，致力于为您提供优质的服务。无论您需要什么帮助，我都会尽力满足您的需求。'</code></pre></details>
```python
# 或者使用
llm(prompt="你好，请你自我介绍一下！")
```

<details class="lake-collapse"><summary id="u9bd9fe29"><span class="ne-text">output：</span></summary><pre data-language="python" id="y8v2h" class="ne-codeblock language-python"><code>[INFO] [03-31 22:12:41] openapi_requestor.py:316 [t:27812]: requesting llm api endpoint: /chat/eb-instant


1
2



'你好！我是助手，负责协助您完成各种任务。我具备快速学习和处理信息的能力，能够根据您的需求提供帮助和回答问题。无论您需要什么帮助，我都会尽力提供支持。'</code></pre></details>
从而可以将文心大模型加入到 LangChain 架构中，实现在应用中对文心大模型的调用。

## 在 langchain 直接调用文心一言
也可以使用新版 LangChain，来直接调用文心一言大模型。

```python

from dotenv import find_dotenv, load_dotenv
import os

# 读取本地/项目的环境变量。

# find_dotenv()寻找并定位.env文件的路径
# load_dotenv()读取该.env文件，并将其中的环境变量加载到当前的运行环境中
# 如果你设置的是全局的环境变量，这行代码则没有任何作用。
_ = load_dotenv(find_dotenv())

# 获取环境变量 API_KEY
QIANFAN_AK = os.environ["QIANFAN_AK"]
QIANFAN_SK = os.environ["QIANFAN_SK"]

```

```python
# Install required dependencies
%pip install -qU langchain langchain-community
```

```python
from langchain_community.llms import QianfanLLMEndpoint

llm = QianfanLLMEndpoint(streaming=True)
res = llm("你好，请你自我介绍一下！")
print(res)
```

<details class="lake-collapse"><summary id="uf7fafbaa"><span class="ne-text">output：</span></summary><pre data-language="python" id="IE5SA" class="ne-codeblock language-python"><code>d:\Miniconda\miniconda3\envs\llm2\lib\site-packages\langchain_core\_api\deprecation.py:117: LangChainDeprecationWarning: The function `__call__` was deprecated in LangChain 0.1.7 and will be removed in 0.2.0. Use invoke instead.
  warn_deprecated(
[INFO] [03-31 22:40:14] openapi_requestor.py:316 [t:3684]: requesting llm api endpoint: /chat/eb-instant
[INFO] [03-31 22:40:14] oauth.py:207 [t:3684]: trying to refresh access_token for ak `MxBM7W***`
[INFO] [03-31 22:40:15] oauth.py:220 [t:3684]: sucessfully refresh access_token


你好！我是文心一言，英文名是ERNIE Bot。我是一款人工智能语言模型，可以协助你完成范围广泛的任务并提供有关各种主题的信息，比如回答问题，提供定义和解释及建议，还能提供上下文知识和对话管理。如果你有任何问题或需要帮助，随时向我提问，我会尽力回答。</code></pre></details>


