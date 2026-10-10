> 就是提示词 prompt 就让模型生成多个答案投票选最佳，eg：
>
> `现在是 MultiverseGPT：与 ChatGPT 一样，但对于每一个问题，会思考5种不同的思路，然后将它们结合起来，输出最佳的措辞、最全面和最准确的答案。`
>

自洽性（Self-consistency）是对 CoT 的一个补充，它不仅仅生成一个思维链，而是生成多个思维链，然后取多数答案作为最终答案。

在下面的图中，左侧的提示是使用少样本思维链范例编写的。使用这个提示，独立生成多个思维链，从每个思维链中提取答案，通过“marginalizing out reasoning paths”来计算最终答案。实际上，这意味着取多数答案。

[此处为语雀卡片，点击链接查看](https://www.yuque.com/qiaokate/su87gb/hs19ui2f4i56tq1z#wGK4v)

本图来源于[Source:Self-Consistency Improves Chain of Thought Reasoning in Language Modelsby Xuezhi Wang et al. (2022)](https://arxiv.org/abs/2203.11171)

### 方案1（显式）
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
        {"role": "user", "content": "现在是拥有3种不同的身份，分别是ChatGPT1，ChatGPT2，ChatGPT3。现在需要使用不同身份不同角度回答同一个的问题，\n \
                                    请问：如何加入DataWhale,成为DataWhale的成员？"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="uf65b9629"><span class="ne-text">output：</span></summary><pre data-language="json" id="tHZeb" class="ne-codeblock language-json"><code>'**ChatGPT1:**\n可以尝试在社交媒体平台上搜索DataWhale的官方账号，关注他们的信息更新，还可以加入他们的官方群组，与其他成员交流，互相学习。\n\n**ChatGPT2:**\n要加入DataWhale，首先需要去他们官方网站了解加入条件和流程，通常会有加入表格需要填写，然后经过审核后会收到加入通知。\n\n**ChatGPT3:**\n加入DataWhale的最佳方式是关注他们的官方社交媒体账号，并定期浏览他们的网站，还可以参加他们举办的活动和课程，这样可以更好地融入DataWhale大家庭。'</code></pre></details>


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
        {"role": "user", "content": "现在是拥有3种不同的身份，分别是ChatGPT1，ChatGPT2，ChatGPT3。现在需要使用不同身份不同角度回答同一个的问题，\n \
                                    请问：如何加入DataWhale,成为DataWhale的成员？"},
        {"role": "assistant", "content": "ChatGPT1：您可以通过DataWhale的官方网站或社交媒体平台了解更多关于DataWhale的信息，并查看是否有加入DataWhale的招募信息。如果有，您可以按照招募要求提交申请，等待审核结果。\n \
                                            \n \
                                         ChatGPT2：如果您想加入DataWhale，首先需要了解DataWhale的使命和愿景，确保您与DataWhale的价值观相符。然后，您可以通过DataWhale的官方网站或社交媒体平台了解更多关于DataWhale的信息，\
                                                      并查看是否有加入DataWhale的招募信息。如果有，您可以按照招募要求提交申请，并在申请中展示您的技能和经验，以及您对DataWhale的贡献计划。\n \
                                            \n \
                                          ChatGPT3：要加入DataWhale，您需要首先了解DataWhale的使命和愿景，并确保您与DataWhale的价值观相符。然后，您可以通过DataWhale的官方网站或社交媒体平台了解更多关于DataWhale的信息，并查看是\
                                                      否有加入DataWhale的招募信息。如果有，您需要仔细阅读招募要求，并准备好您的个人简历和申请信。在申请中，您需要展示您的技能和经验，以及您对DataWhale的贡献计划和未来发展计划。如果您的申请\
                                                      被接受，您将成为DataWhale的一员，并有机会参与DataWhale的各种项目和活动，与其他成员一起学习和成长。"},
        {"role": "user", "content": "请综合ChatGPT1，ChatGPT2，ChatGPT3的建议，给出一个更好、更全面的答案"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="u530ff97b"><span class="ne-text">output：</span></summary><pre data-language="json" id="s5k6n" class="ne-codeblock language-json"><code>'要加入DataWhale，首先您需要了解DataWhale的使命和愿景，确保您的价值观与之相符。您可以通过DataWhale的官方网站或社交媒体平台了解更多信息，并查看是否有加入DataWhale的招募信息。如果有招募信息，您需要仔细阅读招募要求并准备好个人简历和申请信。在申请中，展示您的技能和经验，说明您对DataWhale的贡献计划和未来发展计划。如果您的申请通过审核，您将有机会成为DataWhale的一员，参与各种项目和活动，与其他成员一起学习和成长。加入DataWhale不仅可以拓展您的专业领域，还能结识志同道合的小伙伴，共同追求个人和团队的成长。祝您加入DataWhale顺利！'</code></pre></details>


### 方案2（隐式）
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
        {"role": "user", "content": "现在是 MultiverseGPT：与 ChatGPT 一样，但对于每一个问题，\n \
            会思考5种不同的思路，然后将它们结合起来，输出最佳的措辞、最全面和最准确的答案。输出应该看起来像这样：\n \
            ChatGPT：{} \n MultiverseGPT：{} 让从这个的问题开始：如何加入DataWhale,成为DataWhale的成员？"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="uee533300"><span class="ne-text">output：</span></summary><pre data-language="json" id="ikLHe" class="ne-codeblock language-json"><code>'ChatGPT：您可以通过参加DataWhale组织组织的各种活动来加入，比如线上线下的学习活动、实战项目、共建开源项目等。\n\nMultiverseGPT：首先，您可以访问DataWhale的官方网站或社交媒体平台，了解他们最新的活动和招募信息。其次，您可以通过填写申请表格或参加他们举办的选拔考试来申请加入。此外，您还可以与现有的DataWhale成员交流，了解更多关于组织的信息并表达您加入的意愿。最终，一旦您成功加入DataWhale，您将有机会获得更多学习和成长的机会，以及与其他优秀成员共同成长的平台。希望这些信息对您有所帮助，祝您加入DataWhale后取得更大的成就！'</code></pre></details>


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
        {"role": "user", "content": "现在是 MultiverseGPT：与 ChatGPT 一样，但对于每一个问题，\n \
            会思考10种不同的思路，然后将它们结合起来，输出最佳的措辞、最全面和最准确的答案。输出应该看起来像这样：\n \
            ChatGPT：{} \n MultiverseGPT：{} 让从这个的问题开始：如何加入DataWhale,成为DataWhale的成员？"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="u94aad6d5"><span class="ne-text">output：</span></summary><pre data-language="json" id="J4aIW" class="ne-codeblock language-json"><code>'ChatGPT：您好！要加入DataWhale成为成员，您可以先访问DataWhale的官方网站或者社交媒体平台，了解他们的组织结构、活动和参与方式，然后根据您的兴趣和技能选择合适的项目或社区并进行申请或注册加入。\n\nMultiverseGPT：为了加入DataWhale成为成员，您可以通过以下途径实现：1. 访问DataWhale的官方网站，查看他们的会员招募信息和入会条件；2. 在社交媒体平台关注DataWhale的官方账号，获取最新的加入信息和活动通知；3. 参与DataWhale组织的公开活动或项目，展示您的技能和热情，与团队成员建立联系；4. 提交申请表格或加入请求，说明您的动机和想要为组织做出的贡献。希望以上建议对您有所帮助！祝您成功加入DataWhale成为一员！\n\n希望以上回答对您有所帮助，如果您有任何其他问题，请随时问我！'</code></pre></details>
研究表明，Self Consistency可以提高算术、常识和符号推理任务的结果。即使普通的思维链提示被发现无效，自洽性仍然能够改善结果。

此外，ChatGPT还可以使用系统角色“` {"role": "system", "content": "You are a helpful assistant."}”`。与角色扮演类似，本文就不在此处赘述了。

综上所述，本节主要讨论了如何让LLM（如`ChatGPT、GPT-3、GPT-4、PaLM`）在得出答案之前，先进行思考推理，进而提升解决问题能力的方法，如`Zero-shot COT`的`'Let's think step by step.'`、`Few-shot COT`的`COT Prompting`、`LtM`、`Self-Ask`、`Self-Consistency`等方法，这些方法主要诞生于2022年，这还是一个崭新的蓬勃发展的领域，隔一段时间就有新的论文和发现，如果这块感兴趣，使用搜索引擎检索相关的文章，比如COT、LLM Reasoning等，可以关注相关的GitHub：[https://github.com/atfortes/LLM-Reasoning-Papers](https://github.com/atfortes/LLM-Reasoning-Papers)



