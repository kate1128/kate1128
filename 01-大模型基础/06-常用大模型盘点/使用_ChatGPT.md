> 就是调用 api key 试一下模型回答问题
>

## 配置 OpenAI API key
将创建好的 OpenAI API key 复制以此形式 `OPENAI_API_KEY="sk-..."` 保存到 `.env` 文件中，并将 `.env` 文件保存在项目根目录下。下面是读取 `.env` 文件的代码：

```python
import os
from dotenv import load_dotenv, find_dotenv

# 读取本地/项目的环境变量。

# find_dotenv() 寻找并定位 .env 文件的路径
# load_dotenv() 读取该 .env 文件，并将其中的环境变量加载到当前的运行环境中  
# 如果你设置的是全局的环境变量，这行代码则没有任何作用。
_ = load_dotenv(find_dotenv())

# 如果你需要通过代理端口访问，还需要做如下配置
# os.environ['HTTPS_PROXY'] = 'http://127.0.0.1:7890'
# os.environ["HTTP_PROXY"] = 'http://127.0.0.1:7890'
```

## 调用 OpenAI API
调用 ChatGPT 需要使用 [ChatCompletion API](https://platform.openai.com/docs/api-reference/chat)，该 API 提供了 ChatGPT 系列模型的调用，包括 ChatGPT-3.5，GPT-4 等。

### `ChatCompletion API` 调用方法
```python
from openai import OpenAI

# 获取环境变量 OPENAI_API_KEY
client = OpenAI(
    base_url='https://xiaoai.plus/v1',
    # This is the default and can be omitted
    api_key=os.environ.get("OPENAI_API_KEY"),
)

# 导入所需库
# 注意，此处假设你已根据上文配置了 OpenAI API Key，如没有将访问失败
completion = client.chat.completions.create(
    # 调用模型：ChatGPT-3.5
    model="gpt-3.5-turbo",
    # messages 是对话列表
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ]
)
```

调用该 API 会返回一个 `ChatCompletion` 对象，其中包括了回答文本、创建时间、id 等属性。一般需要的是回答文本，也就是回答对象中的 `content` 信息。

#### 调用 `completion`
```python
completion
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="python" id="iOXbm" class="ne-codeblock language-python"><code>ChatCompletion(id='chatcmpl-9FA5aO72SD9X0XTpc1HCNZkSFCf7C', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='Hello! How can I assist you today?', role='assistant', function_call=None, tool_calls=None))], created=1713400730, model='gpt-3.5-turbo-0125', object='chat.completion', system_fingerprint='fp_c2295e73ad', usage=CompletionUsage(completion_tokens=9, prompt_tokens=19, total_tokens=28))</code></pre></details>
#### 打印 返回值
```python
print(completion.choices[0].message.content)
```

<details class="lake-collapse"><summary id="uf37dadb2"><span class="ne-text">output：</span></summary><pre data-language="python" id="Bp6I9" class="ne-codeblock language-python"><code>Hello! How can I assist you today?</code></pre></details>
#### 常用参数
<br/>success
此处详细介绍调用 API 常会用到的几个参数：

1. `**model**`**：调用的模型**，一般取值包括`“gpt-3.5-turbo”（ChatGPT-3.5）、“gpt-3.5-turbo-16k-0613”（ChatGPT-3.5 16K 版本）、“gpt-4”（ChatGPT-4）`。注意，不同模型的成本是不一样的。
2. `**messages**`**：就是**`**prompt**`。`ChatCompletion` 的 `messages` 需要传入一个消息对象的列表，列表中包括多个不同角色的 `prompt`。每个对象有两个必要的字段：
    - `role`: 信使的角色，包括：`system、user，or assistant`)
        * `system`：即前文中提到的 `system prompt`[什么是 System Prompt](https://www.yuque.com/qiaokate/su87gb/wh3c2yi2ifxwainx)；
        * `user`：用户输入的 `prompt`[什么是 Prompt](https://www.yuque.com/qiaokate/su87gb/egekbk6izhnseccp)；
        * `assistant`：助手，一般是模型历史回复，作为提供给模型的参考内容。
    - `content`: 信息的内容，例如: 给我写一首美丽的诗，消息也可以包含一个可选的名称字段，它给信使一个名字。例如：`example-user、Alice、BlackbeardBot`。名称中不得包含空格。
3. `**temperature**`**：温度**。即前文中提到的 `Temperature` 系数 [什么是 Temperature](https://www.yuque.com/qiaokate/su87gb/ki2f15w3cygugy9n)。
4. `**max_tokens**`**：最大 **`**token**`** 数，即模型输出的最大 **`**token**`** 数**。

`OpenAI` 计算 `token` 数是合并计算 `Prompt` 和 `Completion` 的总 `token` 数，要求总 `token`  数不能超过模型上限（如默认模型 `token`  上限为 4096）。因此，如果输入的 `prompt` 较长，需要设置较大的 `max_token` 值，否则会报错超出限制长度。

<br/>

### 用户和助手的消息交替出现
通常情况下，对话会以一个告诉助手如何行事的系统消息开始，然后是用户和助手的消息交替出现，但可以不遵循这种格式。

```python
from openai import OpenAI

client = OpenAI(
    base_url='https://xiaoai.plus/v1',
    # This is the default and can be omitted
    api_key=os.environ.get("OPENAI_API_KEY"),
)

# Example OpenAI Python library request
MODEL = "gpt-3.5-turbo"

response =  client.chat.completions.create(
    model=MODEL,
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Knock knock."},
        {"role": "assistant", "content": "Who's there?"},
        {"role": "user", "content": "Orange."},
    ],
    temperature=0,
)

response
```

测试一下，显示这个

<details class="lake-collapse"><summary id="u5daffd42"><span class="ne-text" style="color: var(--jp-content-font-color1)">output：</span></summary><pre data-language="json" id="MwYEs" class="ne-codeblock language-json"><code>ChatCompletion(id='chatcmpl-yagqlvRBo9sQEyZrHyHZEFoOIggqO', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='Orange who?', refusal=None, role='assistant', audio=None, function_call=None, tool_calls=None))], created=1736231646, model='gpt-3.5-turbo', object='chat.completion', service_tier=None, system_fingerprint='fp_808245b034', usage=CompletionUsage(completion_tokens=3, prompt_tokens=35, total_tokens=38, completion_tokens_details=None, prompt_tokens_details=None))</code></pre></details>
<br/>color2
响应对象包括下面几个字段：

+ `id`: 请求的ID
+ `object`: 返回的对象的类型（如`chat.completion`）
+ `created`: 请求的时间戳
+ `model`: 用于生成回复的模型的全名
+ `usage`: 用于生成回复的token数量，包括提示、完成和总数。
+ `choices`: 完成对象的列表(只有一个，除非设置n大于1)
    - `message`: 由模型生成的消息对象，包括角色和内容
    - `finish_reason`: 模型停止生成文本的原因（要么是停止，要么是长度，如果达到`max_tokens`的限制）
    - `index`: 选择列表中的完成度的索引

<br/>

转成 json 看一下

```python
{
  "id": "chatcmpl-yagqlvRBo9sQEyZrHyHZEFoOIggqO",
  "object": "chat.completion",
  "created": 1736231646,
  "model": "gpt-3.5-turbo",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Orange who?"
      },
      "finish_reason": "stop",
      "logprobs": null,
      "audio": null,
      "function_call": null,
      "tool_calls": null
    }
  ],
  "usage": {
    "completion_tokens": 3,
    "prompt_tokens": 35,
    "total_tokens": 38,
    "completion_tokens_details": null,
    "prompt_tokens_details": null
  },
  "service_tier": null,
  "system_fingerprint": "fp_808245b034"
}

```

### 示例模板
`OpenAI` 提供了充分的自定义空间，支持通过自定义 `prompt` 来提升模型回答效果，如下是一个简单的封装 `OpenAI` 接口的函数，支持直接传入 `prompt` 并获得模型的输出：

```python
from openai import OpenAI

client = OpenAI(
    # This is the default and can be omitted
    api_key=os.environ.get("OPENAI_API_KEY"),
)


def gen_gpt_messages(prompt):
    '''
    构造 GPT 模型请求参数 messages
    
    请求参数：
        prompt: 对应的用户提示词
    '''
    messages = [{"role": "user", "content": prompt}]
    return messages


def get_completion(prompt, model="gpt-3.5-turbo", temperature = 0):
    '''
    获取 GPT 模型调用结果

    请求参数：
        prompt: 对应的提示词
        model: 调用的模型，默认为 gpt-3.5-turbo，也可以按需选择 gpt-4 等其他模型
        temperature: 模型输出的温度系数，控制输出的随机程度，取值范围是 0~2。温度系数越低，输出内容越一致。
    '''
    response = client.chat.completions.create(
        model=model,
        messages=gen_gpt_messages(prompt),
        temperature=temperature,
    )
    if len(response.choices) > 0:
        return response.choices[0].message.content
        
    return "generate answer error"
```

#### 调用 `get_completion`
```python
get_completion("你好")
```

<details class="lake-collapse"><summary id="ufc828dc7"><span class="ne-text">output：</span></summary><pre data-language="python" id="xpfre" class="ne-codeblock language-python"><code>'你好！有什么可以帮助你的吗？'</code></pre></details>
在上述函数中，封装了 `messages` 的细节，仅使用 `user prompt`来实现调用。在简单场景中，该函数足够满足使用需求。



