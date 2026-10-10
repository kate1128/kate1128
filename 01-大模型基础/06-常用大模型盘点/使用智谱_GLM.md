[GLM 系列](https://www.yuque.com/qiaokate/su87gb/bkrzpcwz1waw3m50)

## API 申请指引
首先进入到 

[智谱AI开放平台](https://open.bigmodel.cn/overview)

点击`开始使用`或者`开发工作台`进行注册，新注册的用户可以免费领取有效期 1 个月的 100w token 的体验包，进行个人实名认证后，还可以额外领取 400w token 体验包。智谱 AI 提供了 GLM-4 和 GLM-3-Turbo 这两种不同模型的体验入口，可以点击`立即体验`按钮直接体验。

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735544459345-b8dc52b1-760a-42ba-a094-045a8ba68c19.png)

对于需要使用 API key 来搭建应用的话，需要点击右侧的`查看 API key`按钮，就会进入到个人的 API 管理列表中。在该界面，就可以看到获取到的 API 所对应的应用名字和 `API key` 了。

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735544369715-07eb6b22-8b3f-4851-9f80-abf3125abc14.png)

可以点击 `添加新的 API key` 并输入对应的名字即可生成新的 API key。

## 常见的模型
参考：[https://open.bigmodel.cn/dev/howuse/model](https://open.bigmodel.cn/dev/howuse/model)

## 调用智谱 GLM API
智谱 AI 提供了 SDK 和原生 HTTP 来实现模型 API 的调用，建议使用 SDK 进行调用以获得更好的编程体验。

首先需要配置密钥信息，将前面获取到的 `API key` 设置到 `.env` 文件中的 `ZHIPUAI_API_KEY` 参数，然后运行以下代码加载配置信息。

```python
import os

from dotenv import load_dotenv, find_dotenv

# 读取本地/项目的环境变量。

# find_dotenv() 寻找并定位 .env 文件的路径
# load_dotenv() 读取该 .env 文件，并将其中的环境变量加载到当前的运行环境中  
# 如果你设置的是全局的环境变量，这行代码则没有任何作用。
_ = load_dotenv(find_dotenv())
```

智谱的调用传参和其他类似，也需要传入一个 `messages` 列表，列表中包括 `role` 和 `prompt`。封装如下的 `get_completion` 函数，供后续使用。

```python
from zhipuai import ZhipuAI

client = ZhipuAI(
    api_key=os.environ["ZHIPUAI_API_KEY"]
)

def gen_glm_params(prompt):
    '''
    构造 GLM 模型请求参数 messages

    请求参数：
        prompt: 对应的用户提示词
    '''
    messages = [{"role": "user", "content": prompt}]
    return messages


def get_completion(prompt, model="glm-4", temperature=0.95):
    '''
    获取 GLM 模型调用结果

    请求参数：
        prompt: 对应的提示词
        model: 调用的模型，默认为 glm-4，也可以按需选择 glm-3-turbo 等其他模型
        temperature: 模型输出的温度系数，控制输出的随机程度，取值范围是 0~1.0，且不能设置为 0。温度系数越低，输出内容越一致。
    '''

    messages = gen_glm_params(prompt)
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=temperature
    )
    if len(response.choices) > 0:
        return response.choices[0].message.content
    return "generate answer error"
```

1. 调用 `get_completion`：

```python
get_completion("你好")
```

<details class="lake-collapse"><summary id="ubee11590"><span class="ne-text">output：</span></summary><pre data-language="python" id="vrwkk" class="ne-codeblock language-python"><code>'你好！有什么可以帮助你的吗？如果有任何问题或需要咨询的事情，请随时告诉我。'</code></pre></details>
<br/>color2
这里对传入 `zhipuai` 的参数进行简单介绍：

+ `**messages (list)**`**，调用对话模型时，将当前对话信息列表作为提示输入给模型**；按照 `{"role": "user", "content": "你好"}` 的键值对形式进行传参；总长度超过模型最长输入限制后会自动截断，需按时间由旧到新排序
+ `**temperature (float)**`**，采样温度，控制输出的随机性**，必须为正数取值范围是：`(0.0, 1.0)`，不能等于 `0`，默认值为 `0.95`。值越大，会使输出更随机，更具创造性；值越小，输出会更加稳定或确定
+ `**top_p (float)**`**，用温度取样的另一种方法，称为核取样**。取值范围是：`(0.0, 1.0)` 开区间，不能等于 `0 `或` 1`，默认值为 `0.7`。模型考虑具有 `top_p` 概率质量 `tokens` 的结果。例如：`0.1 `意味着模型解码器只考虑从前 `10%` 的概率的候选集中取 `tokens`
+ `**request_id (string)**`**，由用户端传参，需保证唯一性**；用于区分每次请求的唯一标识，用户端不传时平台会默认生成
+ **建议您根据应用场景调整 **`**top_p**`** 或 **`**temperature**`** 参数，但不要同时调整两个参数**

<br/>



