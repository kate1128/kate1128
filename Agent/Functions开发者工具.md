> 可以参考这篇：[https://juejin.cn/post/7405776432725737508](https://juejin.cn/post/7405776432725737508)
>
> 用这个工具去测试：[https://www.llm-hub.cn/prompt/text#](https://www.llm-hub.cn/prompt/text#)
>
> 主要是介绍工具函数的调用
>
> 假如有一个工具函数，在一些特定情况下需要调用，经过微调后，这些模型能够传入新的参数，可以通过这个新的参数来自动判断是否调用工具函数，如果判断需要调用工具函数，会返回这个工具函数和对应的输入参数。
>

# 什么是 Function Calling
[什么是Function Calling](https://www.yuque.com/qiaokate/su87gb/skp5bdt3w8ehw70l)

Functions调用的基本步骤如下。

1. 定义函数及元数据：定义好你要做的事情的函数和元数据（JSON Schema）。把JSON Schema提交给大模型。
2. 提出请求：告诉系统想做什么，例如查询天气信息。这个请求需要包包含必要的信息，如地点。
3. 模型生成命令：系统会根据用户的请求决定调用哪个功能。如果该请求与系统中的某个函数匹配，系统就会创建一个包含所有必要信息的字符串，这个字符串符合事先定义的JSON Schema的要求。
4. 执行函数：**系统首先把文本解析为JSON对象，然后根据这个JSON对象执行对应的函数**。如果命令中包括地点信息，那么JSON对象中也会包含统一的地点信息。相应地，函数会去查找那个地点的天气信息。
5. 返回结果：一旦函数执行完毕，系统会返回结果。**这个结果也是用JSON格式来表示的**。通常会把这个结果再次传递回大模型，让它生成最终的回答，也就是输出有关该地点天气信息的文本。

# OpenAI函数新参数
> 前提：先把 openAI Key 什么的加载好
>

## 简单例子：得到当前天气
首先使用 OpenAI 的使用的第一个例子，定义了一个`get_current_weather`的函数，正常来说，获取当前天气是语言模型本身不能完全做到的事情。因此，希望能够将语言模型和这样的函数结合起来，以当前的信息来增强它。

在这个函数，固定了返回的值，比如温度固定为 22 摄氏度，但在实际应用中，这可能涉及到调用天气 API 或一些外部知识源。

```python
import json

# 定义一个函数，用于获取给定位置的当前天气
def get_current_weather(location, unit="摄氏度"):
    """获取指定位置的当前天气"""
    # 模拟返回相同的天气情况的示例函数
    # 在实际应用环境中，这可以是天气API
    # 创建一个天气信息的字典
    weather_info = {
        "location": location,  # 天气的位置
        "temperature": "22",  # 温度
        "unit": unit,  # 温度单位，默认为摄氏度
        "forecast": ["晴", "多云"],  # 天气预报
    }
    # 将天气信息转换为JSON格式的字符串并返回
    return json.dumps(weather_info)
```

## 新参数：functions
<br/>tips
⚠️如何将这样的函数传给语言模型呢？

OpenAI引入了一个名为`functions`的新参数，通过该参数，可以传递一个函数定义列表。

<br/>

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

<br/>color2
这是一个JSON对象，具有几个不同的参数。

+ `name`：函数的名称
+ `description`：函数的描述
+ `parameters`：参数对象，里面有一些属性设置
    - `type`：类型
    - `properties`：本身是一个对象，传入的是对应的参数的描述。在上面示例可以看到有两个元素，`location`和`unit`。都是字符串。`unit`是一个外部参数的设置，比如这里它可以是摄氏度或华氏度，所以定义他的类型和以及枚举参数值。
    - `required`：必填的参数，比如这里需要的参数就是`location`。

在函数定义中，`description`以及在`properties`中的参数非常重要，因为这些将直接传递给语言模型，语言模型将使用这些描述来判断是否使用此函数。

<br/>

## 相关提示调用结果
接下来就可以定义一个有关于天气的问题，如”北京的天气怎么样？“，然后使用 OpenAI 的函数进行调用对话的 API。首先选模型`gpt-3.5-turbo-0613`，然后再将上述定义的 `function` 函数传入，查看最后的响应结果。

```python
import openai

# 定义输入消息
messages = [
    {
        "role": "user",
        "content": "北京的天气怎么样?"
    }
]

# 调用OpenAI的ChatCompletion API获取响应
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions # 传入function参数
)

# 打印响应结果
print(response)
```

从结果上来看，返回的消息的角色是助手，内容为空，有一个函数调用参数`function_call`，其中包含两个对象，`name`和`arguments`。

`Name`是`get_current_weather`，这与传递的函数的名称相同，然后`arguments`是这个 JSON 格式的字典，里面是需要的参数。

<details class="lake-collapse"><summary id="uac7a6d99"><span class="ne-text" style="color: var(--jp-content-font-color1)">output：</span></summary><pre data-language="json" id="b4kom" class="ne-codeblock language-json"><code>{
    &quot;choices&quot;: [
        {
            &quot;finish_reason&quot;: &quot;function_call&quot;,
            &quot;index&quot;: 0,
            &quot;logprobs&quot;: null,
            &quot;message&quot;: {
                &quot;content&quot;: null,
                &quot;function_call&quot;: {
                    &quot;arguments&quot;: &quot;{\n  \&quot;location\&quot;: \&quot;\u5317\u4eac\uff0c\u5317\u4eac\u5e02\&quot;,\n  \&quot;unit\&quot;: \&quot;\u6444\u6c0f\u5ea6\&quot;\n}&quot;,
                    &quot;name&quot;: &quot;get_current_weather&quot;
                },
                &quot;role&quot;: &quot;assistant&quot;
            }
        }
    ],
    &quot;created&quot;: 1709602992,
    &quot;id&quot;: &quot;chatcmpl-8zE7kIlpxzjsiuxH4q6wtRh8CLDH3&quot;,
    &quot;model&quot;: &quot;gpt-3.5-turbo-0613&quot;,
    &quot;object&quot;: &quot;chat.completion&quot;,
    &quot;system_fingerprint&quot;: null,
    &quot;usage&quot;: {
        &quot;completion_tokens&quot;: 30,
        &quot;prompt_tokens&quot;: 95,
        &quot;total_tokens&quot;: 125
    }
}</code></pre></details>
也可以看到响应的参数，比如在这里就是两个参数，分别是`location`和`unit`。

```python
# 打印传入参数
print(response["choices"][0]["message"]["function_call"]["arguments"])
```

<details class="lake-collapse"><summary id="u1de34efa"><span class="ne-text" style="color: var(--jp-content-font-color1)">output：</span></summary><pre data-language="json" id="p2yQq" class="ne-codeblock language-json"><code>{
  &quot;location&quot;: &quot;北京，北京市&quot;,
  &quot;unit&quot;: &quot;摄氏度&quot;
}</code></pre></details>
如果仔细看响应的话，发现内容现在是空的，`function_call`是一个字典

```python
# 获取response里面的message信息
response_message = response["choices"][0]["message"]

print(response_message)
```

<details class="lake-collapse"><summary id="ub51290e3"><span class="ne-text">output：</span></summary><pre data-language="json" id="Psq8J" class="ne-codeblock language-json"><code>{
    &quot;content&quot;: null,
    &quot;function_call&quot;: {
        &quot;arguments&quot;: &quot;{\n  \&quot;location\&quot;: \&quot;\u5317\u4eac\uff0c\u5317\u4eac\u5e02\&quot;,\n  \&quot;unit\&quot;: \&quot;\u6444\u6c0f\u5ea6\&quot;\n}&quot;,
        &quot;name&quot;: &quot;get_current_weather&quot;
    },
    &quot;role&quot;: &quot;assistant&quot;
}</code></pre></details>
`function_call`中的`arguments`参数本身也是一个 JSON 字典

```python
# 打印参数
print(response["choices"][0]["message"]["function_call"]["arguments"])
```

<details class="lake-collapse"><summary id="uca74f973"><span class="ne-text" style="color: var(--jp-content-font-color1)">output：</span></summary><pre data-language="json" id="tIwuQ" class="ne-codeblock language-json"><code>{
  &quot;location&quot;: &quot;北京，北京市&quot;,
  &quot;unit&quot;: &quot;摄氏度&quot;
}</code></pre></details>
可以使用`json.loads`将其加载到 Python 字典中。它返回的参数可以直接传递给上述定义的`get_current_weather`函数

```python
# 将JSON格式的字符串转换为Python对象
args = json.loads(response_message["function_call"]["arguments"])

# 调用get_current_weather函数并传入参数args
get_current_weather(args)
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>{
  &quot;location&quot;: {
    &quot;location&quot;: &quot;\\u5317\\u4eac\\uff0c\\u5317\\u4eac\\u5e02&quot;, 
    &quot;unit&quot;: &quot;\\u6444\\u6c0f\\u5ea6&quot;,
  }, 
  &quot;temperature&quot;: &quot;22&quot;, 
  &quot;unit&quot;: &quot;\\u6444\\u6c0f\\u5ea6&quot;, 
  &quot;forecast&quot;: [&quot;\\u6674&quot;, &quot;\\u591a\\u4e91&quot;],
}</code></pre></details>
所以，使用 OpenAI 进行函数调用不是直接调用工具函数，是告诉要调用的函数名称以及函数的参数。但由于它不会去执行函数，所以如果在使用`json.loads`解码的时候遇到了一些问题，实际上是模型的问题，所以这一部分可以考虑在写工具函数的时候做一些保护措施。（后续讨论）

## 无关提示调用结果
如果问的问题与函数无关会产生什么样的，也就是与天气无关，会返回什么样的信息呢？

```python
# 与天气无关提示调用
messages = [
    {
        "role": "user",
        "content": "你好!",
    }
]

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions,
    function_call="auto",
)

print(response)
```

从结果上来看，可以返回的内容正常并且没有`function_call`参数，也就是在语言模型中判断不使用工具函数。

<details class="lake-collapse"><summary id="u5980c968"><span class="ne-text">output：</span></summary><pre data-language="json" id="ZMQyD" class="ne-codeblock language-json"><code>{
  &quot;choices&quot;: [
    {
      &quot;finish_reason&quot;: &quot;stop&quot;,
      &quot;index&quot;: 0,
      &quot;logprobs&quot;: null,
      &quot;message&quot;: {
        # 这里没有显示 function_call
        &quot;content&quot;: &quot;\u4f60\u597d\uff01\u6709\u4ec0\u4e48\u6211\u53ef\u4ee5\u5e2e\u52a9\u4f60\u7684\u5417\uff1f&quot;,
        &quot;role&quot;: &quot;assistant&quot;
      }
    }
  ],
  &quot;created&quot;: 1709602993,
  &quot;id&quot;: &quot;chatcmpl-8zE7lZ3keV5JGVTFqvAWOGxit4cXj&quot;,
  &quot;model&quot;: &quot;gpt-3.5-turbo-0613&quot;,
  &quot;object&quot;: &quot;chat.completion&quot;,
  &quot;system_fingerprint&quot;: null,
  &quot;usage&quot;: {
    &quot;completion_tokens&quot;: 19,
    &quot;prompt_tokens&quot;: 88,
    &quot;total_tokens&quot;: 107
  }
}</code></pre></details>
由于编码问题，可以单独打印对应的返回的文本信息

```python
print(response["choices"][0]["message"]["content"])
```

<details class="lake-collapse"><summary id="u07a9524d"><span class="ne-text">output：</span></summary><pre data-language="json" id="oph8v" class="ne-codeblock language-json"><code>你好！有什么我可以帮助你的吗？</code></pre></details>
# Function Call参数模式
<br/>color2
`function_call`参数一共有3种模式，可以传递其他参数，`function_call`以强制模型使用或不使用函数。

1. 默认情况下，它设置为`auto`，也就是模型自行选择。
2. 在第二种模式中，可以强制它调用一个函数。
3. 另一种模式是`none`。强制语言模型不使用提供的任何函数。

<br/>

## 自动判断是否调用
`auto`模式就是大模型自行选择是否要返回参数，也是默认模式，上述所有的方式都是`auto`的模式

## 强制不调用
`none`模式就是：内容`你好`不需要函数调用，所以看到它没有被使用

```python
# 无关提示强制不调用
messages = [
    {
        "role": "user",
        "content": "你好",
    }
]

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions,
    function_call="none", # 传入参数强制不调用
)
print(response)
print(response["choices"][0]["message"]["content"])
```

<details class="lake-collapse"><summary id="u9486bfa5"><span class="ne-text">output：</span></summary><pre data-language="json" id="K1HIO" class="ne-codeblock language-json"><code>{
  &quot;id&quot;: &quot;chatcmpl-8nfuv5tlp6UaSwUfmeHppJkQyFpCW&quot;,
  &quot;object&quot;: &quot;chat.completion&quot;,
  &quot;created&quot;: 1706849893,
  &quot;model&quot;: &quot;gpt-3.5-turbo-0613&quot;,
  &quot;choices&quot;: [
    {
      &quot;index&quot;: 0,
      &quot;message&quot;: {
        &quot;role&quot;: &quot;assistant&quot;,
        &quot;content&quot;: &quot;\u4f60\u597d\uff01\u6709\u4ec0\u4e48\u53ef\u4ee5\u5e2e\u52a9\u4f60\u7684\u5417\uff1f&quot;
      },
      &quot;logprobs&quot;: null,
      &quot;finish_reason&quot;: &quot;stop&quot;
    }
  ],
  &quot;usage&quot;: {
    &quot;prompt_tokens&quot;: 88,
    &quot;completion_tokens&quot;: 17,
    &quot;total_tokens&quot;: 105
  },
  &quot;system_fingerprint&quot;: null
}</code></pre><p id="u0ddecea1" class="ne-p"><span class="ne-text">转换下：</span></p><pre data-language="json" id="sM7Nv" class="ne-codeblock language-json"><code>你好！有什么可以帮助你的吗？</code></pre></details>
那如果在需要使用工具的函数时候强制不调用会出现什么结果呢？

```python
# 相关提示强制不调用
messages = [
    {
        "role": "user",
        "content": "北京天气怎么样?",
    }
]
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions,
    function_call="none", # 传入参数强制不调用
)
print(response)
print(response["choices"][0]["message"]["content"])
```

从结果上来看，他依然有正常的`role`和`content`，但是因为强制不调用函数，所以他在试图返回正确的参数，但是并不正确。

<details class="lake-collapse"><summary id="u238f5585"><span class="ne-text" style="color: var(--jp-content-font-color1)">output：</span></summary><pre data-language="json" id="z6dkL" class="ne-codeblock language-json"><code>{
  &quot;id&quot;: &quot;chatcmpl-8nfv0wFX6tqcaoU2gT4KqICymXDxa&quot;,
  &quot;object&quot;: &quot;chat.completion&quot;,
  &quot;created&quot;: 1706849898,
  &quot;model&quot;: &quot;gpt-3.5-turbo-0613&quot;,
  &quot;choices&quot;: [
    {
      &quot;index&quot;: 0,
      &quot;message&quot;: {
        &quot;role&quot;: &quot;assistant&quot;,
        &quot;content&quot;: &quot;\u8bf7\u7a0d\u7b49\uff0c\u6211\u4e3a\u60a8\u67e5\u627e\u5317\u4eac\u7684\u5929\u6c14\u60c5\u51b5\u3002&quot;
      },
      &quot;logprobs&quot;: null,
      &quot;finish_reason&quot;: &quot;stop&quot;
    }
  ],
  &quot;usage&quot;: {
    &quot;prompt_tokens&quot;: 95,
    &quot;completion_tokens&quot;: 18,
    &quot;total_tokens&quot;: 113
  },
  &quot;system_fingerprint&quot;: null
}</code></pre><p id="ua127a10d" class="ne-p"><span class="ne-text">转换下：</span></p><pre data-language="json" id="rfL0G" class="ne-codeblock language-json"><code>请稍等，我为您查找北京的天气情况。</code></pre></details>
## 强制调用
`function_call`参数的最后一个模式是强制调用函数，方法也是很简单，只要在参数里面传入这个函数的名字即可。

在这一部分，指定`name`为`get_current_weather`，这将强制它使用`get_current_weather`函数。如果查看结果，实际上会看到返回了这个`function_call`对象，其中有`name`为`get_current_weather`的一些参数。

```python
# 无关提示强制调用函数
messages = [
    {
        "role": "user",
        "content": "你好!",
    }
]
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions,
    function_call={"name": "get_current_weather"}, # 强制调用函数get_current_weather
)
print(response)
print(response["choices"][0]["message"]["function_call"]["arguments"])
```

这里面传了一个于天气无关的参数，然后强制使用`get_current_weather`函数，但传入的参数中绝对没有任何信息关于它应该如何调用该函数。所以在这里，结果编造了北京，北京市的参数，如果再次运行它，它会不断地调用北京市的参数。

<details class="lake-collapse"><summary id="u548b6f54"><span class="ne-text">output：</span></summary><pre data-language="json" id="EPQsv" class="ne-codeblock language-json"><code>{
  &quot;id&quot;: &quot;chatcmpl-8nfvkQDayf7kmYNdx4G84dkl1L5YC&quot;,
  &quot;object&quot;: &quot;chat.completion&quot;,
  &quot;created&quot;: 1706849944,
  &quot;model&quot;: &quot;gpt-3.5-turbo-0613&quot;,
  &quot;choices&quot;: [
    {
      &quot;index&quot;: 0,
      &quot;message&quot;: {
        &quot;role&quot;: &quot;assistant&quot;,
        &quot;content&quot;: null,
        &quot;function_call&quot;: {
          &quot;name&quot;: &quot;get_current_weather&quot;,
          &quot;arguments&quot;: &quot;{\n  \&quot;location\&quot;: \&quot;\u5317\u4eac\uff0c\u5317\u4eac\u5e02\&quot;\n}&quot;
        }
      },
      &quot;logprobs&quot;: null,
      &quot;finish_reason&quot;: &quot;stop&quot;
    }
  ],
  &quot;usage&quot;: {
    &quot;prompt_tokens&quot;: 95,
    &quot;completion_tokens&quot;: 12,
    &quot;total_tokens&quot;: 107
  },
  &quot;system_fingerprint&quot;: null
}</code></pre><p id="u597f935b" class="ne-p"><span class="ne-text">转换后</span></p><pre data-language="json" id="izNC5" class="ne-codeblock language-json"><code>{
  &quot;location&quot;: &quot;北京，北京市&quot;
}</code></pre></details>
还可以尝试使用不同的模型函数来调用，不同的参数值，不同的输入消息，要注意的是，函数本身和描述都会计入传递给OpenAI的令牌使用限制。所以如果运行这个，可以看到返回的提示令牌是95。如果注释掉`functions`和`function_call`，可以看到提示令牌减少到15。因为OpenAI模型对令牌有限制，因此，在构造要传递给OpenAI的消息时，现在不仅需要注意消息的长度，还需要注意传递的函数的长度

> （也就是说函数也占用 token）
>

#  函数调用以及执行函数
最后，看一下如何将这些函数调用和实际执行函数调用的结果传递回语言模型。这很重要，因为通常希望使用语言模型确定要调用的函数，然后运行该函数，但然后将其传递回语言模型以获得最终响应。

流程：

1. 问题得到一个带有`function call`参数的响应，然后将这个消息添加到的消息列表中。
2. 然后可以模拟使用语言模型提供的参数调用`get_current_weather`函数，并且保存到一个变量`observation`中。
3. 接着定义一个新的消息列表，表示刚刚调用函数的结果，这里面有一个重要的点就是`role`等于`function`，也就是告诉语言模型这是调用函数的响应。除此之外，还传递函数的名称`name`以及`content`变量，设为上述计算的`observation`。

然后使用这个消息列表调用语言模型，可以看到语言模型回答的非常好：北京的天气目前是22摄氏度，天气以晴天为主，也有多云的情况。

> <font style="color:#DF2A3F;">这个示例待补充</font>
>

#  英文版提示
> 折起来，可以不看
>

## 简单例子：得到当前天气
```python
import json

# Example dummy function hard coded to return the same weather
# In production, this could be your backend API or an external API
def get_current_weather(location, unit="fahrenheit"):
    """Get the current weather in a given location"""
    weather_info = {
        "location": location,
        "temperature": "72",
        "unit": unit,
        "forecast": ["sunny", "windy"],
    }
    return json.dumps(weather_info)
```

### 定义function函数
```python
# define a function
functions = [
    {
        "name": "get_current_weather",
        "description": "Get the current weather in a given location",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "The city and state, e.g. San Francisco, CA",
                },
                "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
            },
            "required": ["location"],
        },
    }
]
```

### 相关提示调用结果
```python
messages = [
    {
        "role": "user",
        "content": "What's the weather like in Boston?"
    }
]

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions
)

print(response)
```

<details class="lake-collapse"><summary id="ucd86d42d"><span class="ne-text">output：</span></summary><pre data-language="python" id="GrWwq" class="ne-codeblock language-python"><code>{
  &quot;id&quot;: &quot;chatcmpl-8nfyFSmPyo4RoV79D5Urk1QzLCf1C&quot;,
  &quot;object&quot;: &quot;chat.completion&quot;,
  &quot;created&quot;: 1706850099,
  &quot;model&quot;: &quot;gpt-3.5-turbo-0613&quot;,
  &quot;choices&quot;: [
    {
      &quot;index&quot;: 0,
      &quot;message&quot;: {
        &quot;role&quot;: &quot;assistant&quot;,
        &quot;content&quot;: null,
        &quot;function_call&quot;: {
          &quot;name&quot;: &quot;get_current_weather&quot;,
          &quot;arguments&quot;: &quot;{\n  \&quot;location\&quot;: \&quot;Boston, MA\&quot;\n}&quot;
        }
      },
      &quot;logprobs&quot;: null,
      &quot;finish_reason&quot;: &quot;function_call&quot;
    }
  ],
  &quot;usage&quot;: {
    &quot;prompt_tokens&quot;: 82,
    &quot;completion_tokens&quot;: 18,
    &quot;total_tokens&quot;: 100
  },
  &quot;system_fingerprint&quot;: null
}</code></pre></details>
### 无关提示调用结果
```python
messages = [
    {
        "role": "user",
        "content": "hi!",
    }
]

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions,
)
```

### 自动判断是否调用
自动判断是否调用函数，判断提示是否和函数相关

```python
messages = [
    {
        "role": "user",
        "content": "hi!",
    }
]
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions,
    function_call="auto",
)
print(response)
```

<details class="lake-collapse"><summary id="u996a0106"><span class="ne-text">output：</span></summary><pre data-language="python" id="VrzPe" class="ne-codeblock language-python"><code>{
  &quot;id&quot;: &quot;chatcmpl-8nfyHfC32MVE2gpE6HZxtmC7f7Mvi&quot;,
  &quot;object&quot;: &quot;chat.completion&quot;,
  &quot;created&quot;: 1706850101,
  &quot;model&quot;: &quot;gpt-3.5-turbo-0613&quot;,
  &quot;choices&quot;: [
    {
      &quot;index&quot;: 0,
      &quot;message&quot;: {
        &quot;role&quot;: &quot;assistant&quot;,
        &quot;content&quot;: &quot;Hello! How can I assist you today?&quot;
      },
      &quot;logprobs&quot;: null,
      &quot;finish_reason&quot;: &quot;stop&quot;
    }
  ],
  &quot;usage&quot;: {
    &quot;prompt_tokens&quot;: 76,
    &quot;completion_tokens&quot;: 10,
    &quot;total_tokens&quot;: 86
  },
  &quot;system_fingerprint&quot;: null
}</code></pre></details>
### 强制不调用
无关提示强制不调用，结果正常

```python
messages = [
    {
        "role": "user",
        "content": "hi!",
    }
]
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions,
    function_call="none",
)
print(response)
```

<details class="lake-collapse"><summary id="u7c4dd0bf"><span class="ne-text">output：</span></summary><pre data-language="python" id="ds5t9" class="ne-codeblock language-python"><code>{
  &quot;id&quot;: &quot;chatcmpl-8nfyfrOoacEQCPO3O2sYLukAdbxX4&quot;,
  &quot;object&quot;: &quot;chat.completion&quot;,
  &quot;created&quot;: 1706850125,
  &quot;model&quot;: &quot;gpt-3.5-turbo-0613&quot;,
  &quot;choices&quot;: [
    {
      &quot;index&quot;: 0,
      &quot;message&quot;: {
        &quot;role&quot;: &quot;assistant&quot;,
        &quot;content&quot;: &quot;Hello! How can I assist you today?&quot;
      },
      &quot;logprobs&quot;: null,
      &quot;finish_reason&quot;: &quot;stop&quot;
    }
  ],
  &quot;usage&quot;: {
    &quot;prompt_tokens&quot;: 77,
    &quot;completion_tokens&quot;: 9,
    &quot;total_tokens&quot;: 86
  },
  &quot;system_fingerprint&quot;: null
}</code></pre></details>
### 强制调用
强制无关问题的调用

```python
messages = [
    {
        "role": "user",
        "content": "hi!",
    }
]
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions,
    function_call={"name": "get_current_weather"},
)
print(response)
```

```python
{
  "id": "chatcmpl-8nfyzFFeUyzJDYVbdiVDgPEtNrnKE",
  "object": "chat.completion",
  "created": 1706850145,
  "model": "gpt-3.5-turbo-0613",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": null,
        "function_call": {
          "name": "get_current_weather",
          "arguments": "{\n  \"location\": \"San Francisco, CA\"\n}"
        }
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 83,
    "completion_tokens": 12,
    "total_tokens": 95
  },
  "system_fingerprint": null
}
```

强制有关问题调用，结果和`auto`一样

```python
messages = [
    {
        "role": "user",
        "content": "What's the weather like in Boston!",
    }
]
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo-0613",
    messages=messages,
    functions=functions,
    function_call={"name": "get_current_weather"},
)
print(response)
```

```python
{
  "id": "chatcmpl-8nfzav6WbYIDLPHXYwaBXTM0byxaj",
  "object": "chat.completion",
  "created": 1706850182,
  "model": "gpt-3.5-turbo-0613",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": null,
        "function_call": {
          "name": "get_current_weather",
          "arguments": "{\n  \"location\": \"Boston, MA\"\n}"
        }
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 89,
    "completion_tokens": 11,
    "total_tokens": 100
  },
  "system_fingerprint": null
}
```

