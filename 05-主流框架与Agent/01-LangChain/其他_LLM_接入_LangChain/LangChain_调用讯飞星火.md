同样可以通过 LangChain 框架来调用讯飞星火大模型，更多信息参考[SparkLLM](https://python.langchain.com/docs/integrations/llms/sparkllm)

希望像调用 ChatGPT 那样直接将秘钥存储在 .env 文件中，并将其加载到环境变量，从而隐藏秘钥的具体细节，保证安全性。因此，需要在 .env 文件中配置 `IFLYTEK_SPARK_APP_ID`、 `IFLYTEK_SPARK_API_KEY` 和 `IFLYTEK_SPARK_API_SECRET`，并使用以下代码加载：

```python
from dotenv import find_dotenv, load_dotenv
import os

# 读取本地/项目的环境变量。

# find_dotenv()寻找并定位.env文件的路径
# load_dotenv()读取该.env文件，并将其中的环境变量加载到当前的运行环境中
# 如果你设置的是全局的环境变量，这行代码则没有任何作用。
_ = load_dotenv(find_dotenv())

# 获取环境变量 API_KEY
IFLYTEK_SPARK_APP_ID = os.environ["IFLYTEK_SPARK_APP_ID"]
IFLYTEK_SPARK_API_KEY = os.environ["IFLYTEK_SPARK_API_KEY"]
IFLYTEK_SPARK_API_SECRET = os.environ["IFLYTEK_SPARK_API_SECRET"]
```

```python
def gen_spark_params(model):
    '''
    构造星火模型请求参数
    '''

    spark_url_tpl = "wss://spark-api.xf-yun.com/{}/chat"
    model_params_dict = {
        # v1.5 版本
        "v1.5": {
            "domain": "general", # 用于配置大模型版本
            "spark_url": spark_url_tpl.format("v1.1") # 云端环境的服务地址
        },
        # v2.0 版本
        "v2.0": {
            "domain": "generalv2", # 用于配置大模型版本
            "spark_url": spark_url_tpl.format("v2.1") # 云端环境的服务地址
        },
        # v3.0 版本
        "v3.0": {
            "domain": "generalv3", # 用于配置大模型版本
            "spark_url": spark_url_tpl.format("v3.1") # 云端环境的服务地址
        },
        # v3.5 版本
        "v3.5": {
            "domain": "generalv3.5", # 用于配置大模型版本
            "spark_url": spark_url_tpl.format("v3.5") # 云端环境的服务地址
        }
    }
    return model_params_dict[model]
```

```python
from langchain_community.llms import SparkLLM

spark_api_url = gen_spark_params(model="v1.5")["spark_url"]

# Load the model(默认使用 v3.0)
llm = SparkLLM(spark_api_url = spark_api_url)  #指定 v1.5版本
```

```python
res = llm("你好，请你自我介绍一下！")
print(res)
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="python" id="iOXbm" class="ne-codeblock language-python"><code>您好，我是科大讯飞研发的认知智能大模型，我的名字叫讯飞星火认知大模型。我可以和人类进行自然交流，解答问题，高效完成各领域认知智能需求。</code></pre></details>
可以将星火大模型加入到 LangChain 架构中，实现在应用中对星火大模型的调用。



