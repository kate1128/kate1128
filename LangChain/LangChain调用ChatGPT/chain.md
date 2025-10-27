## chain = chat_prompt | llm | output_parser
现在可以将所有这些组合成一条链。该链将获取输入变量，将这些变量传递给提示模板以创建提示，将提示传递给语言模型，然后通过（可选）输出解析器传递输出。接下来将使用LCEL这种语法去快速实现一条链（chain）。

```python
text = "我带着比身体重的行李，\
游入尼罗河底，\
经过几道闪电 看到一堆光圈，\
不确定是不是这里。\
"

chain = chat_prompt | llm | output_parser
chain.invoke({"input_language":"中文", "output_language":"英文","text": text})

```

<details class="lake-collapse"><summary id="u812b972c"><span class="ne-text">output：</span></summary><pre data-language="python" id="kpBZC" class="ne-codeblock language-python"><code>'I carried luggage heavier than my body and dived into the bottom of the Nile River. After passing through several flashes of lightning, I saw a pile of halos, not sure if this is the place.'</code></pre></details>
再测试一个样例：

```python
text = 'I carried luggage heavier than my body and dived into the bottom of the Nile River. After passing through several flashes of lightning, I saw a pile of halos, not sure if this is the place.'
chain.invoke({"input_language":"英文", "output_language":"中文","text": text})
```

<details class="lake-collapse"><summary id="u5972394d"><span class="ne-text">output：</span></summary><pre data-language="python" id="eYN92" class="ne-codeblock language-python"><code>'我扛着比我的身体还重的行李，潜入尼罗河的底部。穿过几道闪电后，我看到一堆光环，不确定这是否就是目的地。'</code></pre></details>
## 什么是 LCEL
[什么是 LCEL](https://www.yuque.com/qiaokate/su87gb/fpwp7u1tggmac2yb)

<br/>color2
LCEL（LangChain Expression Language，Langchain的表达式语言），LCEL是一种新的语法，是LangChain工具包的重要补充，他有许多优点，使得处理LangChain和代理更加简单方便。

<br/>

+ LCEL提供了异步、批处理和流处理支持，使代码可以快速在不同服务器中移植。
+ LCEL拥有后备措施，解决LLM格式输出的问题。
+ LCEL增加了LLM的并行性，提高了效率。
+ LCEL内置了日志记录，即使代理变得复杂，有助于理解复杂链条和代理的运行情况。

用法示例：

<br/>color2
`chain = prompt | model | output_parser`

<br/>

上面代码中使用 LCEL 将不同的组件拼凑成一个链，在此链中，用户输入传递到提示模板，然后提示模板输出传递到模型，然后模型输出传递到输出解析器。`| `的符号类似于 Unix 管道运算符，它将不同的组件链接在一起，将一个组件的输出作为下一个组件的输入。

## 简单示例完整代码
```python
# 导入所需的库
from langchain_core.output_parsers import StrOutputParser # 用于解析输出结果为字符串
from langchain_core.prompts import ChatPromptTemplate # 用于创建聊天提示模板
from langchain_openai import ChatOpenAI # 用于调用OpenAI的GPT模型

# 创建一个聊天提示模板，其中{topic}是一个占位符，用于后续插入具体的话题
prompt = ChatPromptTemplate.from_template("请讲一个关于 {topic} 的故事")
# 初始化ChatOpenAI对象，指定使用的模型为"gpt-4"
model = ChatOpenAI(model="gpt-4")
# 初始化一个输出解析器，用于将模型的输出解析成字符串
output_parser = StrOutputParser()

'''使用管道操作符（|）连接各个处理步骤，创建一个处理链
   其中prompt用于生成具体的提示文本，
   model用于根据提示文本生成回应，
   output_parser用于处理回应并将其转换为字符串''' 
chain = prompt | model | output_parser

# 调用处理链，传入话题"水仙花"，执行生成故事的操作
message = chain.invoke({"topic": "水仙花"})

# 打印链的输出结果
print(message)
```

##  其他：
[LCEL 高级](https://www.yuque.com/qiaokate/su87gb/zzduwpgld8ut3qwc)

