> 就是在 LangChain 中如何调用 OpenAI Function Calling 的方法
>

# Pydantic语法
[什么是 Pydantic](https://www.yuque.com/qiaokate/su87gb/vwmrt8fsozdnlrl0)

Pydantic 是 python 的“数据验证库”。

+ 与 python 类型注释一起工作。但是，与静态类型检查不同，它们在运行时被积极地用于数据验证和转换。
+ 提供内置方法来序列化/反序列化模型到 JSON ，字典等。
+ LangChain 利用 Pydantic 创建 Json Scheme 描述函数。

## 简单创建Python类
在标准python中，可以创建一个`User`类，拥有`name`、`age`、`email`三种属性值。直接进行创建，不能对字段类型进行约束，即年龄中可能传入不合规的字符串类型。

```python
from typing import List
from pydantic import BaseModel, Field
```

1. 创建User类

```python
# 创建User类
class User:
    def __init__(self, name: str, age: int, email: str):
        self.name = name
        self.age = age
        self.email = email
```

2. 生成user对象

```python
# 生成user对象
foo = User(name="Joe",age=32, email="joe@gmail.com")

# 输出foo中 name字段
print(foo.name)
```

<details class="lake-collapse"><summary id="ub3760eb3"><span class="ne-text">output：</span></summary><pre data-language="json" id="bAMUy" class="ne-codeblock language-json"><code>Joe</code></pre></details>
2. 生成user对象

```python
# 生成user对象
foo = User(name="Joe",age="bar", email="joe@gmail.com")

# 输出foo中 name字段
# name字段填写的是字符串类型，但能够创建成功并输出
print(foo.age)
```

<details class="lake-collapse"><summary id="u395f94e2"><span class="ne-text">output：</span></summary><pre data-language="json" id="H11M0" class="ne-codeblock language-json"><code>bar</code></pre></details>
## 使用 Pydantic 创建类
使用 Pydantic 创建类，可以对类的属性值进行格式约束。在创建类的时候会进行格式验证，如果格式不符合要求会报错。

1. 使用 Pydantic创建pUser类别，说明age使用int类型

```python
# 使用 Pydantic创建pUser类别，说明age使用int类型
class pUser(BaseModel):
    name: str
    age: int
    email: str
```

2. 生成user对象

```python
# 生成user对象
foo_p = pUser(name="Jane", age=32, email="jane@gmail.com")

# 输出foo中 name字段
print(foo_p.name)
```

<details class="lake-collapse"><summary id="ueaf499ca"><span class="ne-text">output：</span></summary><pre data-language="json" id="CIzBQ" class="ne-codeblock language-json"><code>Jane</code></pre></details>
3. 使用了pydantic，由于age不是int，因此会报错。输出报错内容

```python
# 使用了pydantic，由于age不是int，因此会报错。输出报错内容
try:
    foo_p = pUser(name="Jane", age="bar", email="jane@gmail.com")
except Exception as e:
    print(e)
```

<details class="lake-collapse"><summary id="u84436c84"><span class="ne-text">output：</span></summary><pre data-language="json" id="pBLZA" class="ne-codeblock language-json"><code>1 validation error for pUser age value is not a valid integer (type=type_error.integer)</code></pre></details>
4. 修改

```python
class Class(BaseModel):
    students: List[pUser]
```

```python
obj = Class(
    students=[pUser(name="Jane", age=32, email="jane@gmail.com")]
)
obj
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>Class(students=[pUser(name='Jane', age=32, email='jane@gmail.com')])</code></pre></details>
#  基于Pydantic的OpenAI函数定义
## 构造 OpenAI 的 function
我们创建了一个`WeatherSearch`类，它继承自 `Pydantic` 的 `BaseModel` 子类。因此`WeatherSearch`类的所有成员都被具备了数据类型校验机制，该类有一个`str`类型的成员`airport_code`表示机场代码，并有一个描述信息`description`，用来说明 `airport_code` 的作用，在 `airport_code` 的上方也有一段文本描述信息：输入机场代码，获取该机场的天气信息。这段文本信息是对类`WeatherSearch`的说明，意思是通过机场代码可以查询天气情况。同时，我们使用 `langchain` 将这个`WeatherSearch`类转换成`openai`的函数描述对象：

1. 定义`WeatherSearch`，`WeatherSearch`的`function`需要写注释完成函数的`description`

```python
# 定义WeatherSearch
# WeatherSearch的function需要写注释完成函数的description
class WeatherSearch(BaseModel):
    """输入机场代码，获取该机场的天气信息"""
    airport_code: str = Field(description="输入机场代码查询天气")
```

2. 导入`langchain`

```python
# 导入langchain
from langchain.utils.openai_functions import convert_pydantic_to_openai_function
```

3. 转化为`openai function calling`模式

```python
# 转化为openai function calling模式
weather_function = convert_pydantic_to_openai_function(WeatherSearch)
weather_function
```

<details class="lake-collapse"><summary id="ucf930685"><span class="ne-text">output：</span></summary><pre data-language="json" id="QafRk" class="ne-codeblock language-json"><code>{
    'name': 'WeatherSearch',
    'description': '输入机场代码，获取该机场的天气信息',
    'parameters': {
      'title': 'WeatherSearch',
      'description': '输入机场代码，获取该机场的天气信息',
      'type': 'object',
      'properties': {
        'airport_code': {
          'title': 'Airport Code',
          'description': '输入机场代码查询天气',
          'type': 'string',
          },
        },
        'required': ['airport_code'],
    },
}</code></pre></details>
接下来，使用 `langchain` 的`convert_pydantic_to_openai_function`方法将 `Pydantic` 类转换成了 `openai` 的函数描述对象。需要的注意的是在定义`Pydantic`类时文本描述信息不可缺少，如缺少文本描述信息会导致转换时出错。

+ `WeatherSearch1`：由于我们没有在`WeatherSearch1`中加入对本身的描述信息，导致在转换时报错。

```python
# WeatherSearch1没有写注释，会报错
class WeatherSearch1(BaseModel):
    airport_code: str = Field(description="输入机场代码查询天气")

convert_pydantic_to_openai_function(WeatherSearch1)
```

<details class="lake-collapse"><summary id="u56863b11"><span class="ne-text">output：报错</span></summary><pre data-language="json" id="jLPPU" class="ne-codeblock language-json"><code>---------------------------------------------------------------------------

KeyError                                  Traceback (most recent call last)

Cell In[15], line 5

      2 class WeatherSearch1(BaseModel):

      3     airport_code: str = Field(description=&quot;输入机场代码查询天气&quot;)

----&gt; 5 convert_pydantic_to_openai_function(WeatherSearch1)



File ~/anaconda3/lib/python3.11/site-packages/langchain/utils/openai_functions.py:28, in convert_pydantic_to_openai_function(model, name, description)

     24 schema = dereference_refs(model.schema())

     25 schema.pop(&quot;definitions&quot;, None)

     26 return {

     27     &quot;name&quot;: name or schema[&quot;title&quot;],

---&gt; 28     &quot;description&quot;: description or schema[&quot;description&quot;],

     29     &quot;parameters&quot;: schema,

     30 }



KeyError: 'description'</code></pre></details>
+ `WeatherSearch2`加入对本身的描述信息，因此不会报错。

```python
# 构造WeatherSearch2，不对参数注释
class WeatherSearch2(BaseModel):
    """输入机场代码，获取该机场的天气信息"""
    airport_code: str
```

```python
convert_pydantic_to_openai_function(WeatherSearch2)
```

<details class="lake-collapse"><summary id="u0beabd18"><span class="ne-text">output：</span></summary><pre data-language="json" id="rpCkA" class="ne-codeblock language-json"><code>{'name': 'WeatherSearch2',
  'description': '输入机场代码，获取该机场的天气信息',
  'parameters': {'title': 'WeatherSearch2',
  'description': '输入机场代码，获取该机场的天气信息',
  'type': 'object',
  'properties': {'airport_code': {'title': 'Airport Code', 'type': 'string'}},
'required': ['airport_code']}}</code></pre></details>
## 通过langchain使用function
为了实现 LLM 对 function 的调用，有一下两种方式

+ 在`invoke`中指定 functions
+ 执行`invoke`之前使用`bind`方法来绑定函数描述对象

```python
# 导入ChatOpenAI
from langchain.chat_models import ChatOpenAI
```

```python
model = ChatOpenAI()
```

1. 在invoke中使用openai function功能

```python
# 在invoke中使用openai function功能
model.invoke("今天旧金山的天气怎么样？", functions=[weather_function])
```

<details class="lake-collapse"><summary id="u7a5eef78"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="K0cIp" class="ne-codeblock language-json"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'WeatherSearch', 'arguments': '{\n  &quot;airport_code&quot;: &quot;SFO&quot;\n}'}}, example=False)</code></pre></details>
2. 直接在构造模型中声明functions，对话时不需要在声明

```python
# 直接在构造模型中声明functions，对话时不需要在声明
model_with_function = model.bind(functions=[weather_function])
model_with_function.invoke("今天旧金山的天气怎么样？")
```

<details class="lake-collapse"><summary id="u545ac97b"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="IkqkT" class="ne-codeblock language-json"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'WeatherSearch', 'arguments': '{\n  &quot;airport_code&quot;: &quot;SFO&quot;\n}'}}, example=False)</code></pre></details>
## 强制使用function
如果想要强制使用 function，需要在`bind`中增加`function_call`参数。

```python
# 强制使用function，在模型构建时声明function_call
model_with_forced_function = model.bind(functions=[weather_function], function_call={"name":"WeatherSearch"})
```

1. 基于已经声明的function对话，能够调用function

```python
# 基于已经声明的function对话，能够调用function
model_with_forced_function.invoke("旧金山的天气怎么样？")
```

<details class="lake-collapse"><summary id="ub6e83c41"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="B6CT5" class="ne-codeblock language-json"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'WeatherSearch', 'arguments': '{\n  &quot;airport_code&quot;: &quot;SFO&quot;\n}'}}, example=False)</code></pre></details>
2. 输入hi，强制使用function的模型，即使prompt与函数description无关也会命中

```python
# 输入hi，强制使用function的模型，即使prompt与函数description无关也会命中
model_with_forced_function.invoke("你好!")
```

<details class="lake-collapse"><summary id="uc4d2e11c"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="jtxMX" class="ne-codeblock language-json"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'WeatherSearch', 'arguments': '{\n  &quot;airport_code&quot;: &quot;LAX&quot;\n}'}}, example=False)</code></pre></details>
3. 输入hi，未强制使用function的模型，prompt与函数description无关不会命中

```python
# 输入hi，未强制使用function的模型，prompt与函数description无关不会命中
model_with_function.invoke("你好!")
```

<details class="lake-collapse"><summary id="u6c277e98"><span class="ne-text">output：</span></summary><pre data-language="json" id="w7chd" class="ne-codeblock language-json"><code>AIMessage(content='Hello! How can I assist you today?', additional_kwargs={}, example=False)</code></pre></details>
# 使用function
在一般情况下我们会使用 chain 来实现整个问答的流程，接下来我们通过创建 chain 来实现函数调用功能

## 链式使用
引入需要的环境

```python
# 引入需要的环境
from langchain.prompts import ChatPromptTemplate
```

使用预定义模版创建聊天代码

```python
# 使用预定义模版创建聊天代码
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个乐于助人的助手"),
    ("user", "{input}")
])
```

使用prompt + model_with_function 组合

```python
# 使用prompt + model_with_function 组合
chain = prompt | model_with_function
```

创建命中function的问答

```python
# 创建命中function的问答
chain.invoke({"input": "今天旧金山的天气怎么样？"})
```

<details class="lake-collapse"><summary id="uf44bf307"><span class="ne-text">output：</span></summary><pre data-language="json" id="mqHpv" class="ne-codeblock language-json"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'WeatherSearch', 'arguments': '{\n  &quot;airport_code&quot;: &quot;SFO&quot;\n}'}}, example=False)</code></pre></details>
创建未命中function的问答

```python
# 创建未命中function的问答
chain.invoke({"input": "你好!"})
```

```json
AIMessage(content='Hello! How can I assist you today?', additional_kwargs={}, example=False)
```

## 使用多个function
我们可以传递一组函数，让 LLM 根据问题上下文决定使用哪个函数。

创建 `ArtistSearch function`

```python
# 创建 ArtistSearch function
class ArtistSearch(BaseModel):
    """调用此命令可以获得特定艺术家的歌曲名称"""
    artist_name: str = Field(description="要查的艺术家的名字")
    n: int = Field(description="number of results")
```

组装`WeatherSearch`和`ArtistSearch` 函数

```python
# 组装WeatherSearch和ArtistSearch 函数
functions = [
    convert_pydantic_to_openai_function(WeatherSearch),
    convert_pydantic_to_openai_function(ArtistSearch),
]
```

将两个function放入模型

```python
# 将两个function放入模型
model_with_functions = model.bind(functions=functions)
```

命中 WeatherSearch function

```python
# 命中 WeatherSearch function
model_with_functions.invoke("旧金山的天气怎么样？")
```

<details class="lake-collapse"><summary id="ud06a1166"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="x4GMb" class="ne-codeblock language-json"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'WeatherSearch', 'arguments': '{\n  &quot;airport_code&quot;: &quot;SFO&quot;\n}'}}, example=False)</code></pre></details>
命中 ArtistSearch function

```python
# 命中 ArtistSearch function
model_with_functions.invoke("找到泰勒·斯威夫特的三首歌？")
```

<details class="lake-collapse"><summary id="ufd761516"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="znKgr" class="ne-codeblock language-json"><code>AIMessage(content='', additional_kwargs={'function_call': {'name': 'ArtistSearch', 'arguments': '{\n  &quot;artist_name&quot;: &quot;taylor swift&quot;,\n  &quot;n&quot;: 3\n}'}}, example=False)</code></pre></details>
命中都未命中

```python
# 命中都未命中
model_with_functions.invoke("你好!")
```

<details class="lake-collapse"><summary id="u633bd9fe"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="ExvSG" class="ne-codeblock language-json"><code>AIMessage(content='Hello! How can I assist you today?', additional_kwargs={}, example=False)</code></pre></details>
# 英文版提示
## 构造OpenAI的function
```python
class WeatherSearch(BaseModel):
    """Call this with an airport code to get the weather at that airport"""
    airport_code: str = Field(description="airport code to get weather for")
```

```python
class WeatherSearch1(BaseModel):
    airport_code: str = Field(description="airport code to get weather for")
```

```python
class WeatherSearch2(BaseModel):
    """Call this with an airport code to get the weather at that airport"""
    airport_code: str
```

## 链式使用
```python
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant"),
    ("user", "{input}")
])
```

## 使用多个function
```python
class ArtistSearch(BaseModel):
    """Call this to get the names of songs by a particular artist"""
    artist_name: str = Field(description="name of artist to look up")
    n: int = Field(description="number of results")
```

