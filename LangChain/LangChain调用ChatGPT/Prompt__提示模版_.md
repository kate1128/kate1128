在开发大模型应用时，大多数情况下不会直接将用户的输入直接传递给 LLM。通常，他们会将用户输入添加到一个较大的文本中，称为`提示模板`，该文本提供有关当前特定任务的附加上下文。`PromptTemplates` 正是帮助解决这个问题！它们捆绑了从用户输入到完全格式化的提示的所有逻辑。这可以非常简单地开始 

例如，生成上述字符串的提示就是，需要先构造一个个性化 Template：

```python
from langchain_core.prompts import ChatPromptTemplate

# 这里要求模型对给定文本进行中文翻译
prompt = """请你将由三个反引号分割的文本翻译成英文！\
text: ```{text}```
"""
```

接下来看一下构造好的完整的提示模版：

```python
text = "我带着比身体重的行李，\
游入尼罗河底，\
经过几道闪电 看到一堆光圈，\
不确定是不是这里。\
"
prompt.format(text=text)
```

<details class="lake-collapse"><summary id="u703685df"><span class="ne-text">output：</span></summary><pre data-language="python" id="xhEKL" class="ne-codeblock language-python"><code>'请你将由三个反引号分割的文本翻译成英文！text: ```我带着比身体重的行李，游入尼罗河底，经过几道闪电 看到一堆光圈，不确定是不是这里。```\n'</code></pre></details>
聊天模型的接口是基于消息（message），而不是原始的文本。`PromptTemplates` 也可以用于产生消息列表，在这种样例中，`prompt`不仅包含了输入内容信息，也包含了每条`message`的信息(角色、在列表中的位置等)。通常情况下，一个 `ChatPromptTemplate` 是一个 `ChatMessageTemplate` 的列表。每个 `ChatMessageTemplate` 包含格式化该聊天消息的说明（其角色以及内容）。

看一个示例：

```python
from langchain.prompts.chat import ChatPromptTemplate

template = "你是一个翻译助手，可以帮助我将 {input_language} 翻译成 {output_language}."
human_template = "{text}"

chat_prompt = ChatPromptTemplate.from_messages([
    ("system", template),
    ("human", human_template),
])

text = "我带着比身体重的行李，\
游入尼罗河底，\
经过几道闪电 看到一堆光圈，\
不确定是不是这里。\
"
messages  = chat_prompt.format_messages(input_language="中文", output_language="英文", text=text)
messages

```

<details class="lake-collapse"><summary id="u63a8dfcf"><span class="ne-text">output：</span></summary><pre data-language="python" id="dqVQv" class="ne-codeblock language-python"><code>[SystemMessage(content='你是一个翻译助手，可以帮助我将 中文 翻译成 英文.'),
 HumanMessage(content='我带着比身体重的行李，游入尼罗河底，经过几道闪电 看到一堆光圈，不确定是不是这里。')]</code></pre></details>
接下来调用定义好的`llm`和`messages`来输出回答：

```python
output  = llm.invoke(messages)
output
```

<details class="lake-collapse"><summary id="u2cc6550c"><span class="ne-text">output：</span></summary><pre data-language="python" id="pSkDS" class="ne-codeblock language-python"><code>AIMessage(content='I carried luggage heavier than my body and dived into the bottom of the Nile River. After passing through several flashes of lightning, I saw a pile of halos, not sure if this is the place.')</code></pre></details>
调用`ChatPromptTemplate.from_template()`函数将上面的提示模版字符`prompt`转换为提示模版`prompt_template`

```python
from langchain.prompts.chat import ChatPromptTemplate
prompt_template = ChatPromptTemplate.from_template(template_string)

print(prompt_template.messages[0].prompt)
```

<details class="lake-collapse"><summary id="u55733f2f"><span class="ne-text">output：</span></summary><pre data-language="json" id="OrO5X" class="ne-codeblock language-json"><code>input_variables=['style', 'text'] template='Translate the text that is delimited by triple backticks into a style that is {style}. text: ```{text}```\n'</code></pre></details>
从上面的输出可以看出，`prompt_template`有两个输入变量： `style`和 `text`。



