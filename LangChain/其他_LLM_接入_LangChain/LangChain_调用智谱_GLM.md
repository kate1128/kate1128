同样可以通过 LangChain 框架来调用智谱 AI 大模型，以将其接入到的应用框架中。由于 langchain 中提供的[ChatGLM](https://python.langchain.com/docs/integrations/llms/chatglm)已不可用，因此需要自定义一个LLM。

如果使用的是智谱 GLM API，运行下列代码来在 LangChain 中使用 GLM。

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-

from typing import Any, List, Mapping, Optional, Dict
from langchain_core.callbacks.manager import CallbackManagerForLLMRun
from langchain_core.language_models.llms import LLM
from zhipuai import ZhipuAI

import os

# 继承自 langchain.llms.base.LLM
class ZhipuAILLM(LLM):
    # 默认选用 glm-4
    model: str = "glm-4"
    # 温度系数
    temperature: float = 0.1
    # API_Key
    api_key: str = None
    
    def _call(self, prompt : str, stop: Optional[List[str]] = None,
                run_manager: Optional[CallbackManagerForLLMRun] = None,
                **kwargs: Any):
        client = ZhipuAI(
            api_key = self.api_key
        )

        def gen_glm_params(prompt):
            '''
            构造 GLM 模型请求参数 messages

            请求参数：
                prompt: 对应的用户提示词
            '''
            messages = [{"role": "user", "content": prompt}]
            return messages
        
        messages = gen_glm_params(prompt)
        response = client.chat.completions.create(
            model = self.model,
            messages = messages,
            temperature = self.temperature
        )

        if len(response.choices) > 0:
            return response.choices[0].message.content
        return "generate answer error"


    # 首先定义一个返回默认参数的方法
    @property
    def _default_params(self) -> Dict[str, Any]:
        """获取调用API的默认参数。"""
        normal_params = {
            "temperature": self.temperature,
            }
        # print(type(self.model_kwargs))
        return {**normal_params}

    @property
    def _llm_type(self) -> str:
        return "Zhipu"

    @property
    def _identifying_params(self) -> Mapping[str, Any]:
        """Get the identifying parameters."""
        return {**{"model": self.model}, **self._default_params}

```

根据智谱官方宣布以下模型即将弃用，在这些模型弃用后，会将它们自动路由至新的模型。请用户注意在弃用日期之前，将您的模型编码更新为最新版本，以确保服务的顺畅过渡，更多模型相关信息请访问[model](https://open.bigmodel.cn/dev/howuse/model)

| 模型编码 | 弃用日期 | 指向模型 |
| --- | --- | --- |
| chatglm_pro | 2024 年 12 月 31 日 | glm-4 |
| chatglm_std | 2024 年 12 月 31 日 | glm-3-turbo |
| chatglm_lite | 2024 年 12 月 31 日 | glm-3-turbo |


## 自定义chatglm接入 langchain
```python
# 需要下载源码
from zhipuai_llm import ZhipuAILLM
```

```python
from dotenv import find_dotenv, load_dotenv
import os

# 读取本地/项目的环境变量。

# find_dotenv()寻找并定位.env文件的路径
# load_dotenv()读取该.env文件，并将其中的环境变量加载到当前的运行环境中
# 如果你设置的是全局的环境变量，这行代码则没有任何作用。
_ = load_dotenv(find_dotenv())

# 获取环境变量 API_KEY
api_key = os.environ["ZHIPUAI_API_KEY"] #填写控制台中获取的 APIKey 信息
```

```python
zhipuai_model = ZhipuAILLM(model = "glm-4", temperature = 0.1, api_key = api_key)  #model="glm-4-0520",
```

```python
zhipuai_model("你好，请你自我介绍一下！")
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="python" id="iOXbm" class="ne-codeblock language-python"><code>'你好！我是智谱清言，是清华大学 KEG 实验室和智谱 AI 公司于 2023 年共同训练的语言模型。我的目标是通过回答用户提出的问题来帮助他们解决问题。由于我是一个计算机程序，所以我没有自我意识，也不能像人类一样感知世界。我只能通过分析我所学到的信息来回答问题。'</code></pre></details>


