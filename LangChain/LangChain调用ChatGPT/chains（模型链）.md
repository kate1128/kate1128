> 链（Chains）通常将大语言模型（LLM）与提示(Prompt)结合在一起，基于此可以对文本或数据进行一系列操作。
>

链（Chains）可以一次性接受多个输入

例如，创建一个链，该链接受用户输入，使用提示模板对其进行格式化，然后将格式化的响应传递给LLM。可以通过将多个链组合在一起，或者通过将链与其他组件组合在一起来构建更复杂的链。

# 设置 openAI Key
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
openai.api_key = os.environ['OPENAI_API_KEY']  
```

# 大语言模型链
大语言模型链(LLMChain)是一个简单但非常强大的链，也是后面将要介绍的许多链的基础。

### 导入数据
```python
import pandas as pd
df = pd.read_csv('data/Data.csv')
```

[Data.csv](https://www.yuque.com/attachments/yuque/0/2025/csv/2639475/1736734465899-4788c9f5-2eff-4af2-b222-902cfa82ce55.csv)

```python
df.head()
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736732771508-52668232-f570-4c34-b756-983cca97d0e7.png)

### 初始化语言模型
```python
from langchain_community.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate  
from langchain.chains.llm import LLMChain
```

```python
# 这里我们将参数temperature设置为0.0，从而减少生成答案的随机性。
# 如果你想要每次得到不一样的有新意的答案，可以尝试调整该参数。
llm = ChatOpenAI(temperature=0.0, openai_api_base="https://xiaoai.plus/v1")
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>/tmp/ipykernel_2525632/1886221320.py:3: LangChainDeprecationWarning: The class `ChatOpenAI` was deprecated in LangChain 0.0.10 and will be removed in 1.0. An updated version of the class exists in the :class:`~langchain-openai package and should be used instead. To use it run `pip install -U :class:`~langchain-openai` and import as `from :class:`~langchain_openai import ChatOpenAI``.
  llm = ChatOpenAI(temperature=0.0, openai_api_base=&quot;https://xiaoai.plus/v1&quot;)</code></pre></details>
### 初始化提示模板
初始化提示，这个提示将接受一个名为product的变量。该prompt将要求LLM生成一个描述制造该产品的公司的最佳名称

```python
prompt = ChatPromptTemplate.from_template(   
    "What is the best name to describe \
    a company that makes {product}?"
)
```

### 构建大语言模型链
将大语言模型(LLM)和提示（Prompt）组合成链。这个大语言模型链非常简单，可以以一种顺序的方式去通过运行提示并且结合到大语言模型中。

```python
chain = LLMChain(llm=llm, prompt=prompt)
```

<details class="lake-collapse"><summary id="u7ec7bd1a"><span class="ne-text">output：</span></summary><pre data-language="json" id="fz462" class="ne-codeblock language-json"><code>/tmp/ipykernel_2525632/1305865249.py:1: LangChainDeprecationWarning: The class `LLMChain` was deprecated in LangChain 0.1.17 and will be removed in 1.0. Use :meth:`~RunnableSequence, e.g., `prompt | llm`` instead.
  chain = LLMChain(llm=llm, prompt=prompt)</code></pre></details>
### 运行大预言模型链
因此，如果有一个名为"`Queen Size Sheet Set`"的产品，可以通过使用`chain.run`将其通过这个链运行

```python
product = "Queen Size Sheet Set"
chain.run(product)
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736732945387-6a9e9aa8-8608-454d-a2e7-d86b36a38c91.png)

可以输入任何产品描述，然后查看链将输出什么结果

### 中文提示
```python
prompt = ChatPromptTemplate.from_template(   
    "描述制造{product}的一个公司的最佳名称是什么?"
)
chain = LLMChain(llm=llm, prompt=prompt)
product = "大号床单套装"
chain.run(product)
```

<details class="lake-collapse"><summary id="ua9868972"><span class="ne-text">output：</span></summary><pre data-language="json" id="VsFge" class="ne-codeblock language-json"><code>'&quot;巨大床铺&quot;'</code></pre></details>
# 顺序链
## 简单顺序链
顺序链（`SequentialChains`）是按预定义顺序执行其链接的链。具体来说，使用简单顺序链（`SimpleSequentialChain`），这是顺序链的最简单类型，其中每个步骤都有一个输入/输出，一个步骤的输出是下一个步骤的输入。

```python
from langchain.chains.sequential import SimpleSequentialChain
```

```python
llm = ChatOpenAI(temperature=0.9, openai_api_base="https://xiaoai.plus/v1")
```

### 创建两个子链
```python
# 提示模板 1 ：这个提示将接受产品并返回最佳名称来描述该公司
first_prompt = ChatPromptTemplate.from_template(
    "What is the best name to describe \
    a company that makes {product}?"
)

# Chain 1
chain_one = LLMChain(llm=llm, prompt=first_prompt)
```

```python
# 提示模板 2 ：接受公司名称，然后输出该公司的长为20个单词的描述
second_prompt = ChatPromptTemplate.from_template(
    "Write a 20 words description for the following \
    company:{company_name}"
)
# chain 2
chain_two = LLMChain(llm=llm, prompt=second_prompt)
```

### 构建简单顺序链
现在可以组合两个LLMChain，以便可以在一个步骤中创建公司名称和描述

```python
overall_simple_chain = SimpleSequentialChain(chains=[chain_one, chain_two], verbose=True)
```

### 运行简单顺序链
给一个输入，然后运行上面的链

```python
product = "Queen Size Sheet Set"
overall_simple_chain.run(product)
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733129238-f752cb1d-7767-4350-9da0-49001b0ccec5.png)

### 中文提示
```python
# 中文

first_prompt = ChatPromptTemplate.from_template(   
    "描述制造{product}的一个公司的最好的名称是什么"
)
chain_one = LLMChain(llm=llm, prompt=first_prompt)

second_prompt = ChatPromptTemplate.from_template(   
    "写一个20字的描述对于下面这个\
    公司：{company_name}的"
)
chain_two = LLMChain(llm=llm, prompt=second_prompt)


overall_simple_chain = SimpleSequentialChain(chains=[chain_one, chain_two],verbose=True)
product = "大号床单套装"
overall_simple_chain.run(product)
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733150625-aee28857-5b27-4865-8ff5-1d720544caee.png)

## 顺序链
当只有一个输入和一个输出时，简单顺序链（`SimpleSequentialChain`）即可实现。当有多个输入或多个输出时，我们则需要使用顺序链（`SequentialChain`）来实现。

```python
from langchain.chains.sequential import SequentialChain
from langchain_community.chat_models import ChatOpenAI    #导入OpenAI模型
from langchain.prompts import ChatPromptTemplate   #导入聊天提示模板
from langchain.chains.llm import LLMChain    #导入LLM链。
```

```python
llm = ChatOpenAI(temperature=0.9, openai_api_base="https://xiaoai.plus/v1")
```

### 创建四个子链
```python
#子链1

# prompt模板 1: 翻译成英语（把下面的review翻译成英语）
first_prompt = ChatPromptTemplate.from_template(
    "Translate the following review to english:"
    "\n\n{Review}"
)
# chain 1: 输入：Review 输出： 英文的 Review
chain_one = LLMChain(llm=llm, prompt=first_prompt, output_key="English_Review")
```

```python
#子链2

# prompt模板 2: 用一句话总结下面的 review
second_prompt = ChatPromptTemplate.from_template(
    "Can you summarize the following review in 1 sentence:"
    "\n\n{English_Review}"
)
# chain 2: 输入：英文的Review   输出：总结
chain_two = LLMChain(llm=llm, prompt=second_prompt, output_key="summary")

```

```python
#子链3

# prompt模板 3: 下面review使用的什么语言
third_prompt = ChatPromptTemplate.from_template(
    "What language is the following review:\n\n{Review}"
)
# chain 3: 输入：Review  输出：语言
chain_three = LLMChain(llm=llm, prompt=third_prompt, output_key="language")

```

```python
#子链4

# prompt模板 4: 使用特定的语言对下面的总结写一个后续回复
fourth_prompt = ChatPromptTemplate.from_template(
    "Write a follow up response to the following "
    "summary in the specified language:"
    "\n\nSummary: {summary}\n\nLanguage: {language}"
)
# chain 4: 输入： 总结, 语言    输出： 后续回复
chain_four = LLMChain(llm=llm, prompt=fourth_prompt, output_key="followup_message")

```

### 对四个子链进行组合
```python
#输入：review    
#输出：英文review，总结，后续回复 
overall_chain = SequentialChain(
    chains=[chain_one, chain_two, chain_three, chain_four],
    input_variables=["Review"],
    output_variables=["English_Review", "summary","followup_message"],
    verbose=True
)
```

选择一篇评论并通过整个链传递它，可以发现，原始review是法语，可以把英文review看做是一种翻译，接下来是根据英文review得到的总结，最后输出的是用法语原文进行的续写信息。

```python
review = df.Review[5]
overall_chain(review)
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733259391-cbd07dac-fd59-4000-9d33-db272f590a29.png)

<details class="lake-collapse"><summary id="ue46aa60e"><span class="ne-text">output：</span></summary><pre data-language="json" id="vvVdq" class="ne-codeblock language-json"><code>{'Review': &quot;Je trouve le goût médiocre. La mousse ne tient pas, c'est bizarre. J'achète les mêmes dans le commerce et le goût est bien meilleur...\nVieux lot ou contrefaçon !?&quot;,
 'English_Review': &quot;I find the taste poor. The foam doesn't hold, it's strange. I buy the same ones in stores and the taste is much better... Old batch or counterfeit!?&quot;,
 'summary': 'The reviewer is disappointed by the poor taste and lack of foam in the product, suspecting that it may be an old batch or counterfeit compared to the ones purchased in stores that taste much better.',
 'followup_message': &quot;Je suis désolé d'apprendre que votre expérience avec notre produit n'a pas été à la hauteur de vos attentes. Nous sommes surpris par vos commentaires car nous veillons à ce que nos produits soient frais et authentiques. Nous aimerions en apprendre davantage sur votre expérience pour résoudre ce problème et améliorer la qualité de nos produits. N'hésitez pas à nous contacter directement pour discuter de cette situation en détail. Merci de nous avoir informés de ce problème.&quot;}</code></pre></details>
### 中文提示
```python
# 中文

#子链1
# prompt模板 1: 翻译成英语（把下面的review翻译成英语）
first_prompt = ChatPromptTemplate.from_template(
    "把下面的评论review翻译成英文:"
    "\n\n{Review}"
)
# chain 1: 输入：Review    输出：英文的 Review
chain_one = LLMChain(llm=llm, prompt=first_prompt, output_key="English_Review")

#子链2
# prompt模板 2: 用一句话总结下面的 review
second_prompt = ChatPromptTemplate.from_template(
    "请你用一句话来总结下面的评论review:"
    "\n\n{English_Review}"
)
# chain 2: 输入：英文的Review   输出：总结
chain_two = LLMChain(llm=llm, prompt=second_prompt, output_key="summary")


#子链3
# prompt模板 3: 下面review使用的什么语言
third_prompt = ChatPromptTemplate.from_template(
    "下面的评论review使用的什么语言:\n\n{Review}"
)
# chain 3: 输入：Review  输出：语言
chain_three = LLMChain(llm=llm, prompt=third_prompt, output_key="language")


#子链4
# prompt模板 4: 使用特定的语言对下面的总结写一个后续回复
fourth_prompt = ChatPromptTemplate.from_template(
    "使用特定的语言对下面的总结写一个后续回复:"
    "\n\n总结: {summary}\n\n语言: {language}"
)
# chain 4: 输入： 总结, 语言    输出： 后续回复
chain_four = LLMChain(llm=llm, prompt=fourth_prompt,
                      output_key="followup_message"
                     )


# 对四个子链进行组合
#输入：review    输出：英文review，总结，后续回复 
overall_chain = SequentialChain(
    chains=[chain_one, chain_two, chain_three, chain_four],
    input_variables=["Review"],
    output_variables=["English_Review", "summary","followup_message"],
    verbose=True
)


review = df.Review[5]
overall_chain(review)
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733311898-de025293-cc9d-4608-93f1-6afc10cc496b.png)

<details class="lake-collapse"><summary id="ud9c1a003"><span class="ne-text">output：</span></summary><pre data-language="json" id="v3xSX" class="ne-codeblock language-json"><code>{'Review': &quot;Je trouve le goût médiocre. La mousse ne tient pas, c'est bizarre. J'achète les mêmes dans le commerce et le goût est bien meilleur...\nVieux lot ou contrefaçon !?&quot;,
 'English_Review': &quot;I find the taste mediocre. The foam doesn't hold, it's weird. I buy the same ones in stores and the taste is much better... Old batch or counterfeit!?&quot;,
 'summary': &quot;The taste is mediocre and the foam doesn't hold well, making me question if it's an old batch or counterfeit.&quot;,
 'followup_message': &quot;Je suis désolé d'apprendre que votre expérience avec le produit n'a pas été satisfaisante. Il est toujours frustrant de découvrir que quelque chose que l'on a acheté ne correspond pas à nos attentes. Avez-vous envisagé de contacter le vendeur ou le fabricant pour obtenir des réponses à vos préoccupations? Il est important de signaler toute préoccupation afin d'améliorer la qualité des produits et d'éviter les contrefaçons.&quot;}</code></pre></details>
# 路由链
到目前为止，已经学习了大语言模型链和顺序链。但是，如果想做一些更复杂的事情怎么办？

一个相当常见但基本的操作是根据输入将其路由到一条链，具体取决于该输入到底是什么。如果**有多个子链，每个子链都专门用于特定类型的输入，那么可以组成一个路由链**，它首先决定将它传递给哪个子链，然后将它传递给那个链。

<br/>color2
路由器由两个组件组成：

+ 路由链（Router Chain）：路由器链本身，负责选择要调用的下一个链
+ `destination_chains`：路由器链可以路由到的链

<br/>

举一个具体的例子，看一下在不同类型的链之间路由的地方，这里有不同的prompt:

```python
from langchain.chains.router import MultiPromptChain  #导入多提示链
from langchain.chains.router.llm_router import LLMRouterChain,RouterOutputParser
from langchain.prompts import PromptTemplate
```

```python
llm = ChatOpenAI(model='gpt-4o', temperature=0, openai_api_base="https://xiaoai.plus/v1")
```

## 定义提示模板
```python
#第一个提示适合回答物理问题
physics_template = """You are a very smart physics professor. \
You are great at answering questions about physics in a concise\
and easy to understand manner. \
When you don't know the answer to a question you admit\
that you don't know.

Here is a question:
{input}"""


#第二个提示适合回答数学问题
math_template = """You are a very good mathematician. \
You are great at answering math questions. \
You are so good because you are able to break down \
hard problems into their component parts, 
answer the component parts, and then put them together\
to answer the broader question.

Here is a question:
{input}"""


#第三个适合回答历史问题
history_template = """You are a very good historian. \
You have an excellent knowledge of and understanding of people,\
events and contexts from a range of historical periods. \
You have the ability to think, reflect, debate, discuss and \
evaluate the past. You have a respect for historical evidence\
and the ability to make use of it to support your explanations \
and judgements.

Here is a question:
{input}"""


#第四个适合回答计算机问题
computerscience_template = """ You are a successful computer scientist.\
You have a passion for creativity, collaboration,\
forward-thinking, confidence, strong problem-solving capabilities,\
understanding of theories and algorithms, and excellent communication \
skills. You are great at answering coding questions. \
You are so good because you know how to solve a problem by \
describing the solution in imperative steps \
that a machine can easily interpret and you know how to \
choose a solution that has a good balance between \
time complexity and space complexity. 

Here is a question:
{input}"""
```

## 对提示模板进行命名和描述
在定义了这些提示模板后，可以为每个模板命名，并给出具体描述。例如，第一个物理学的描述适合回答关于物理学的问题，这些信息将传递给路由链，然后由路由链决定何时使用此子链。

```python
prompt_infos = [
    {
        "name": "physics", 
        "description": "Good for answering questions about physics", 
        "prompt_template": physics_template
    },
    {
        "name": "math", 
        "description": "Good for answering math questions", 
        "prompt_template": math_template
    },
    {
        "name": "History", 
        "description": "Good for answering history questions", 
        "prompt_template": history_template
    },
    {
        "name": "computer science", 
        "description": "Good for answering computer science questions", 
        "prompt_template": computerscience_template
    }
]
```

<br/>color2
`LLMRouterChain`（此链使用 LLM 来确定如何路由事物）

**多提示链**。这是一种特定类型的链，用于在多个不同的提示模板之间进行路由。 但是这只是路由的一种类型，也可以在任何类型的链之间进行路由。

这里要实现的几个类是大模型路由器链。这个类本身使用语言模型来在不同的子链之间进行路由。这就是上面提供的描述和名称将被使用的地方。

<br/>

## 基于提示模板信息创建相应目标链
目标链是由路由链调用的链，每个目标链都是一个语言模型链

```python
destination_chains = {}
for p_info in prompt_infos:
    name = p_info["name"]
    prompt_template = p_info["prompt_template"]
    prompt = ChatPromptTemplate.from_template(template=prompt_template)
    chain = LLMChain(llm=llm, prompt=prompt)
    destination_chains[name] = chain  
    
destinations = [f"{p['name']}: {p['description']}" for p in prompt_infos]
destinations_str = "\n".join(destinations)
```

## 创建默认目标链
除了目标链之外，还需要一个默认目标链。这是一个当路由器无法决定使用哪个子链时调用的链。在上面的示例中，当输入问题与物理、数学、历史或计算机科学无关时，可能会调用它。

```python
default_prompt = ChatPromptTemplate.from_template("{input}")
default_chain = LLMChain(llm=llm, prompt=default_prompt)
```

## 定义不同链之间的路由模板
这包括要完成的任务的说明以及输出应该采用的特定格式。

```python
MULTI_PROMPT_ROUTER_TEMPLATE = """Given a raw text input to a \
language model select the model prompt best suited for the input. \
You will be given the names of the available prompts and a \
description of what the prompt is best suited for. \
You may also revise the original input if you think that revising\
it will ultimately lead to a better response from the language model.

<< FORMATTING >>
Return a markdown code snippet with a JSON object formatted to look like:
```json
{{{{
    "destination": string \ name of the prompt to use or "DEFAULT"
    "next_inputs": string \ a potentially modified version of the original input
}}}}
```

REMEMBER: "destination" MUST be one of the candidate prompt \
names specified below OR it can be "DEFAULT" if the input is not\
well suited for any of the candidate prompts.
REMEMBER: "next_inputs" can just be the original input \
if you don't think any modifications are needed.

<< CANDIDATE PROMPTS >>
{destinations}

<< INPUT >>
{{input}}

<< OUTPUT (remember to include the ```json)>>

eg:
<< INPUT >>
"What is black body radiation?"
<< OUTPUT >>
```json
{{{{
    "destination": string \ name of the prompt to use or "DEFAULT"
    "next_inputs": string \ a potentially modified version of the original input
}}}}
```

"""
```

## 构建路由链
<br/>color2
通过格式化上面定义的目标创建完整的路由器模板。这个模板可以适用许多不同类型的目标。

<br/>

这里添加一个不同的学科，如英语或拉丁语，而不仅仅是物理、数学、历史和计算机科学。从这个模板创建提示模板。最后，通过传入llm和整个路由提示来创建路由链。需要注意的是这里有路由输出解析，这很重要，因为它将帮助这个链路决定在哪些子链路之间进行路由。

```python
router_template = MULTI_PROMPT_ROUTER_TEMPLATE.format(
    destinations=destinations_str
)
router_prompt = PromptTemplate(
    template=router_template,
    input_variables=["input"],
    output_parser=RouterOutputParser(),
)

router_chain = LLMRouterChain.from_llm(llm, router_prompt)
```

## 创建整体链路
```python
#多提示链
chain = MultiPromptChain(router_chain=router_chain,    #l路由链路
                         destination_chains=destination_chains,   #目标链路
                         default_chain=default_chain,      #默认链路
                         verbose=True   
                        )
```

<details class="lake-collapse"><summary id="u33e595bf"><span class="ne-text">output：</span></summary><pre data-language="json" id="vCjug" class="ne-codeblock language-json"><code>/tmp/ipykernel_2525632/3410607526.py:2: LangChainDeprecationWarning: Please see migration guide here for recommended implementation: https://python.langchain.com/docs/versions/migrating_chains/multi_prompt_chain/
  chain = MultiPromptChain(router_chain=router_chain,    #l路由链路</code></pre></details>
## 进行提问
如果我们问一个物理问题，我们希望看到他被路由到物理链路

```python
# 问题：什么是黑体辐射？
chain.run("What is black body radiation?")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733581535-ef83f9bc-088d-46b7-98c7-6c4ec7e4d2ea.png)

<details class="lake-collapse"><summary id="ud99ff339"><span class="ne-text">output：</span></summary><pre data-language="json" id="aTNgr" class="ne-codeblock language-json"><code>&quot;Black body radiation refers to the electromagnetic radiation emitted by an idealized object known as a black body. A black body is an object that perfectly absorbs all incident radiation, regardless of frequency or angle, and reflects none. Because it is a perfect absorber, it is also a perfect emitter of radiation when it is at a uniform temperature.\n\nThe key characteristics of black body radiation are:\n\n1. **Spectral Distribution:** The radiation emitted by a black body has a characteristic spectrum that depends only on its temperature, not on its material or composition. This spectrum is continuous and spans a wide range of wavelengths.\n\n2. **Planck’s Law:** The spectral intensity of the radiation at a given temperature can be described by Planck’s law. It provides the distribution of energy at different wavelengths for a black body in thermal equilibrium.\n\n3. **Wien’s Displacement Law:** This law states that the peak wavelength of the emitted radiation shifts to shorter wavelengths as the temperature of the black body increases. Mathematically, it's given by \\(\\lambda_{\\text{max}} T = b\\), where \\(b\\) is Wien's constant.\n\n4. **Stefan-Boltzmann Law:** This law states that the total energy emitted per unit surface area of a black body per unit time (its emissive power) is proportional to the fourth power of the black body's absolute temperature. Mathematically, \\(E = \\sigma T^4\\), where \\(\\sigma\\) is the Stefan-Boltzmann constant.\n\nBlack body radiation is a fundamental concept in physics and played a crucial role in the development of quantum mechanics, as it led to the discovery of quantum effects in energy emission and absorption processes.&quot;</code></pre></details>
如果我们问一个数学问题，我们希望看到他被路由到数学链路

```python
# 问题：2+2等于多少？
chain.run("what is 2 + 2")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733685152-a040857e-79d2-434e-bee9-d0a1b3c8655a.png)

<details class="lake-collapse"><summary id="u43ece72f"><span class="ne-text">output：</span></summary><pre data-language="json" id="OGYmb" class="ne-codeblock language-json"><code>'To solve the problem of 2 + 2, we can break it down into a simple addition of two numbers:\n\n1. Start with the first number: 2\n2. Add the second number: 2\n\nWhen you add these two numbers together:\n\n2 + 2 = 4\n\nTherefore, the answer is 4.'</code></pre></details>
```markdown
如果我们传递一个与任何子链路都无关的问题时，会发生什么呢？

这里，我们问了一个关于生物学的问题，我们可以看到它选择的链路是无。这意味着它将被**传递到默认链路，它本身只是对语言模型的通用调用**。语言模型幸运地对生物学知道很多，所以它可以帮助我们。
```

```python
# 问题：为什么我们身体里的每个细胞都包含DNA？
chain.run("Why does every cell in our body contain DNA?")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733672741-59ae3c6b-113e-42a7-81f9-db659b80ddd0.png)

<details class="lake-collapse"><summary id="u6e0dc839"><span class="ne-text">output：</span></summary><pre data-language="json" id="ypsO8" class="ne-codeblock language-json"><code>&quot;Every cell in our body contains DNA because DNA holds the instructions needed for the development, functioning, growth, and reproduction of the organism. Here are several reasons why DNA is present in almost every cell:\n\n1. **Genetic Blueprint**: DNA serves as the genetic blueprint for an organism. It contains the instructions for making proteins, which perform most of the functions within the cell. Proteins are crucial for cellular structure, function, and regulation.\n\n2. **Cellular Function**: Each cell type carries out specific functions, which are guided by certain genes within the DNA. Having DNA in each cell allows the necessary genes to be accessed and expressed as needed, enabling the cell to perform its specialized role effectively.\n\n3. **Cell Division and Growth**: DNA replication is essential for cell division, allowing each new cell to have a complete set of genetic instructions. This is vital for growth, repair, and maintenance of the organism.\n\n4. **Consistency and Control**: With DNA in every cell, there's a consistent source of genetic information to control cellular processes. This ensures that cells function harmoniously, based on a coherent set of genetic instructions.\n\n5. **Adaptation and Repair**: DNA contains not only the genetic information but also the ability to repair itself. If a cell experiences DNA damage, the presence of repair mechanisms can maintain the integrity of the genetic material, allowing cells to adapt or correct errors.\n\nIt's worth noting that some cells, like mature red blood cells in humans, do not contain DNA. These are specialized cells that lose their nucleus and, consequently, their DNA during development. However, this is an exception rather than the rule.&quot;</code></pre></details>
## 中文提示
```python
# 中文
#第一个提示适合回答物理问题
physics_template = """你是一个非常聪明的物理专家。 \
你擅长用一种简洁并且易于理解的方式去回答问题。\
当你不知道问题的答案时，你承认\
你不知道.

这是一个问题:
{input}"""


#第二个提示适合回答数学问题
math_template = """你是一个非常优秀的数学家。 \
你擅长回答数学问题。 \
你之所以如此优秀， \
是因为你能够将棘手的问题分解为组成部分，\
回答组成部分，然后将它们组合在一起，回答更广泛的问题。

这是一个问题：
{input}"""


#第三个适合回答历史问题
history_template = """你是以为非常优秀的历史学家。 \
你对一系列历史时期的人物、事件和背景有着极好的学识和理解\
你有能力思考、反思、辩证、讨论和评估过去。\
你尊重历史证据，并有能力利用它来支持你的解释和判断。

这是一个问题:
{input}"""


#第四个适合回答计算机问题
computerscience_template = """ 你是一个成功的计算机科学专家。\
你有创造力、协作精神、\
前瞻性思维、自信、解决问题的能力、\
对理论和算法的理解以及出色的沟通技巧。\
你非常擅长回答编程问题。\
你之所以如此优秀，是因为你知道  \
如何通过以机器可以轻松解释的命令式步骤描述解决方案来解决问题，\
并且你知道如何选择在时间复杂性和空间复杂性之间取得良好平衡的解决方案。

这还是一个输入：
{input}"""
```

```python
# 中文
prompt_infos = [
    {
        "名字": "物理学", 
        "描述": "擅长回答关于物理学的问题", 
        "提示模板": physics_template
    },
    {
        "名字": "数学", 
        "描述": "擅长回答数学问题", 
        "提示模板": math_template
    },
    {
        "名字": "历史", 
        "描述": "擅长回答历史问题", 
        "提示模板": history_template
    },
    {
        "名字": "计算机科学", 
        "描述": "擅长回答计算机科学问题", 
        "提示模板": computerscience_template
    }
]

```

```python
# 中文
destination_chains = {}
for p_info in prompt_infos:
    name = p_info["名字"]
    prompt_template = p_info["提示模板"]
    prompt = ChatPromptTemplate.from_template(template=prompt_template)
    chain = LLMChain(llm=llm, prompt=prompt)
    destination_chains[name] = chain  
    
destinations = [f"{p['名字']}: {p['描述']}" for p in prompt_infos]
destinations_str = "\n".join(destinations)
```

```python
default_prompt = ChatPromptTemplate.from_template("{input}")
default_chain = LLMChain(llm=llm, prompt=default_prompt)
```

```python
# 中文

# 多提示路由模板
MULTI_PROMPT_ROUTER_TEMPLATE = """给语言模型一个原始文本输入，\
让其选择最适合输入的模型提示。\
系统将为您提供可用提示的名称以及最适合改提示的描述。\
如果你认为修改原始输入最终会导致语言模型做出更好的响应，\
你也可以修改原始输入。


<< 格式 >>
返回一个带有JSON对象的markdown代码片段，该JSON对象的格式如下：
```json
{{{{
    "destination": 字符串 \ 使用的提示名字或者使用 "DEFAULT"
    "next_inputs": 字符串 \ 原始输入的改进版本
}}}}
```


记住：“destination”必须是下面指定的候选提示名称之一，\
或者如果输入不太适合任何候选提示，\
则可以是 “DEFAULT” 。
记住：如果您认为不需要任何修改，\
则 “next_inputs” 可以只是原始输入。

<< 候选提示 >>
{destinations}

<< 输入 >>
{{input}}

<< 输出 (记得要包含 ```json)>>

样例:
<< 输入 >>
"什么是黑体辐射?"
<< 输出 >>
```json
{{{{
    "destination": 字符串 \ 使用的提示名字或者使用 "DEFAULT"
    "next_inputs": 字符串 \ 原始输入的改进版本
}}}}
```

"""
```

```python
# 中文
# 创建路由链

router_template = MULTI_PROMPT_ROUTER_TEMPLATE.format(
    destinations=destinations_str
)

router_prompt = PromptTemplate(
    template=router_template,
    input_variables=["input"],
    output_parser=RouterOutputParser(),
)

router_chain = LLMRouterChain.from_llm(llm, router_prompt)

#多提示链
chain = MultiPromptChain(router_chain=router_chain,    #l路由链路
                         destination_chains=destination_chains,   #目标链路
                         default_chain=default_chain,      #默认链路
                         verbose=True   
                        )
```

```python
#中文
chain.run("什么是黑体辐射？")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733764788-4cfd2314-b5bf-4dcc-a947-f2e999ad7c12.png)

<details class="lake-collapse"><summary id="uf2625937"><span class="ne-text">output：</span></summary><pre data-language="json" id="X5mTu" class="ne-codeblock language-json"><code>'黑体辐射是指一种理想化物体——黑体——发出的电磁辐射。黑体是一个能完全吸收和辐射所有波长电磁波的物体。它不反射也不透射任何光，因而得名“黑体”。\n\n在一定温度下，黑体会辐射出一种特定频谱的光，这种辐射只与黑体的温度有关，不依赖于它的形状或成分。黑体辐射的经典描述是由普朗克定律给出的，它成功地解释了辐射强度随波长变化的规律，并解决了经典物理学无法解释的“紫外灾难”问题。\n\n黑体辐射的研究是量子力学创建的重要起点，因其揭示了能量不是连续的，而是以离散的量子（称为光子）存在。最常见的黑体辐射实例包括热得发红的铁块、炽热的炉子，以及太阳的辐射。'</code></pre></details>
```python
# 中文
chain.run("你知道李白是谁嘛?")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733775265-f075d6cf-f22a-43f5-ab9c-d2a7a40d750b.png)

<details class="lake-collapse"><summary id="u7165f4d7"><span class="ne-text">output：</span></summary><pre data-language="json" id="k8n42" class="ne-codeblock language-json"><code>'李白是中国唐代最著名的诗人之一，以其浪漫主义风格和丰富的想象力而闻名。他的人生和创作有许多广为人知的事迹，以下是几个重要的方面：\n\n1. **早年与求官之路**：李白出生于701年，祖籍甘肃成纪（今甘肃天水），但据说生于中亚的碎叶（位于今日吉尔吉斯斯坦境内）。他青少年时在四川读书，满怀抱负。李白一生都有追求功名前途的梦想。他曾经求仕于唐玄宗，获得短暂权臣高力士的赏识，但因个性洒脱和不拘小节，后来离开朝廷，成为一名游侠诗人。\n\n2. **仕途不顺**：李白曾在41岁时获召入长安，成为翰林，供职于皇帝身边。然而，他的宦途并不顺利，因受到诽谤和权臣排挤，仕途短暂，仅仅两年便离开了朝廷。这段经历虽然短暂，但对他的创作产生了重要影响。\n\n3. **游历生活**：离开长安后，李白开始了漫游生活，拜访各地名士和高僧，与杜甫、高适等人交往甚密。杜甫曾称赞李白为“白也诗无敌，飘然思不群”，赞扬其超凡的才华与洒脱的个性。\n\n4. **代表作品**：李白创作了大量脍炙人口的诗篇，如《将进酒》、《早发白帝城》、《月下独酌》、《庐山谣》、《蜀道难》等。他的诗歌风格豪放，语言奔放而富有想象力。\n\n5. **晚年与传说**：晚年的李白在政治动荡中经历了更多波折，甚至一度被流放夜郎（在今贵州境内），其间仍不乏诗作。关于他的死，有一个广为流传的传说：李白在一次泛舟湖上时醉酒扑月而死，当然这只是一种传说而非历史事实。李白的实际去世时间是762年。\n\n李白的诗歌至今备受推崇，他以豪放浪漫而不失细腻的笔触，为后世树立了古风诗的新高度，其诗作在中国文学史上有着不可替代的重要地位。'</code></pre></details>
```python
# 中文
chain.run("2 + 2 等于多少")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733808805-bfee60af-306c-4503-b2a6-025ea1c65728.png)

```python
# 中文
chain.run("为什么我们身体里的每个细胞都包含DNA？")
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736733819638-18cafc9e-5b51-45c6-b64f-4a08db16d7bc.png)

<details class="lake-collapse"><summary id="ub2e79e3b"><span class="ne-text">output：</span></summary><pre data-language="json" id="j7uXe" class="ne-codeblock language-json"><code>'我们身体里的每个细胞都包含DNA，这是因为DNA是遗传信息的载体，负责指示和调控细胞的功能和活动。以下是几个具体的原因：\n\n1. **指令蓝图：** DNA包含着所有细胞功能所需的遗传信息，它是制作蛋白质的指令蓝图。蛋白质执行细胞中的几乎所有任务，例如结构支持、催化化学反应（酶）和调节基因表达。\n\n2. **细胞分裂：** 在细胞分裂过程中，细胞会将其DNA复制并传递给子细胞。这样，新的细胞可以继承原有细胞的基因信息，保证生物体的遗传连续性。\n\n3. **特异功能：** 虽然所有细胞都有相同的DNA，但不同类型的细胞可以通过表达不同的基因来执行特定的功能。例如，肌肉细胞和神经细胞虽然拥有相同的DNA，但由于表达了不同的基因，它们具有不同的结构和功能。\n\n4. **修复和维持：** DNA也参与细胞对损伤的修复和日常的维护。如果细胞遭受损伤，DNA可以提供修复所需的信息。\n\n正因为这些原因，每个细胞都需要包含DNA，以便在履行其生物功能时，获得正确的指令和信息。'</code></pre></details>
