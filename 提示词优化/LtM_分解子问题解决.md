<br/>color2
Least to Most prompting 将问题分解为子问题逐个解决

<br/>

最少到最多提示过程 (`Least to Most prompting, LtM`) 是思维链提示过程(CoT prompting)的进一步发展。具体来说，首先**将问题分解为子问题，然后逐个解决**。这是受到针对儿童的现实教育策略的启发而发展出的一种技术。

[此处为语雀卡片，点击链接查看](https://www.yuque.com/qiaokate/su87gb/cgxs9alodbbgqoqs#wGK4v)

本图来源于[Source:Least-to-most Prompting Enables Complex Reasoning in Large Language Modelsby Denny Zhou et al. (2022)](https://arxiv.org/abs/2205.10625)

与思维链提示过程类似，需要解决的问题首先被分解成一组建立在彼此之上的子问题。在第二步中，这些子问题被逐个解决。与思维链不同的是，先前子问题的解决方案被输入到提示中，以尝试解决下一个问题。

### 示例：字符连接
尝试问一个稍微复杂的问题(本问题来源于[Learn Prompting网站](https://learnprompting.org/docs/intermediate/least_to_most))：

#### 标准的`few-shot prompt`
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
       {"role": "user", "content": "Q: think, machine \n \
                                     A: ke \n \
                                            \n \
                                     Q: learning, reasoning, generalization \n \
                                     A: ggn \n \
                                            \n \
                                     Q: artificial, intelligence  \n \
                                     A: le  \n \
                                            \n \
                                     Q: foo,bar,baz,blip  \n \
                                     A:"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="ub17973b6"><span class="ne-text">output：</span></summary><pre data-language="json" id="b1nW5" class="ne-codeblock language-json"><code>'qrx '</code></pre></details>
即使是ChatGPT，`few-shot`示例的表现也非常糟糕。

#### COT思维链过程
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
        {"role": "user", "content": "Q: think, machine \n \
                                     A: The last letter of \"think\" is \"k\". The last letter of \"machine\" is \"e\". So \"think, machine\" is \"ke\". \n \
                                            \n \
                                     Q: learning, reasoning, generalization \n \
                                     A: The last letter of \"learning\" is \"g\". The last letter of \"reasoning\" is \"n\". The last letter of \"generalization\" is \"n\". So \"learning, reasoning, generalization\" is \"ggn\". \n \
                                            \n \
                                     Q: artificial, intelligence  \n \
                                     A: The last letter of \"artificial\" is \"l\". The last letter of \"intelligence\" is \"e\". So \"artificial, intelligence\" is \"le\". \n \
                                            \n \
                                     Q: foo,bar,baz,blip  \n \
                                     A:"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="u74e2060b"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="Pl9Hd" class="ne-codeblock language-json"><code>' The last letter of &quot;foo&quot; is &quot;o&quot;. The last letter of &quot;bar&quot; is &quot;r&quot;. The last letter of &quot;baz&quot; is &quot;z&quot;. The last letter of &quot;blip&quot; is &quot;p&quot;. So &quot;foo, bar, baz, blip&quot; is &quot;orzp&quot;.'</code></pre></details>


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
         {"role": "user", "content": "Q: think, machine \n \
                                     A: The last letter of \"think\" is \"k\". The last letter of \"machine\" is \"e\". So \"think, machine\" is \"ke\". \n \
                                            \n \
                                     Q: learning, reasoning, generalization \n \
                                     A: The last letter of \"learning\" is \"g\". The last letter of \"reasoning\" is \"n\". The last letter of \"generalization\" is \"n\". So \"learning, reasoning, generalization\" is \"ggn\". \n \
                                            \n \
                                     Q: artificial, intelligence  \n \
                                     A: The last letter of \"artificial\" is \"l\". The last letter of \"intelligence\" is \"e\". So \"artificial, intelligence\" is \"le\". \n \
                                            \n \
                                     Q: foo,bar,baz,blip,learn,prompting,world,shaking,event,dancefloor,prisma,giraffe  \n \
                                     A:"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="u27d3f3cf"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="FKxbQ" class="ne-codeblock language-json"><code>' The last letter of each word in the list is: &quot;o, r, z, p, n, g, d, g, t, r, a, e&quot;. So the sequence is &quot;orzpngdgtreae&quot;.'</code></pre></details>
思维链的表现比标准提示好得多。这是因为它现在允许模型考虑自己提取每个单词的最后一个字母，将复杂性降低到分组已经收集的字母的行为。然而，这种方法在更长的输入下也可能慢慢出现问题。

> "foo,bar,baz,blip" --> "orzp"  
"foo,bar,baz,blip,learn,prompting,world,shaking,event,dancefloor,prisma,giraffe" --> "orzpngdgrae"
>

当输入为4个词时，`Chain of Thought`完全回答正确，当输入增加到12个词的时候，经过分析过程提到了`event`这个词的末尾字母为`t`，但结果却忘记输出了。

#### LtM(单一提示)
关于使用LtM，通过重新表述先前串联的结果来增强思维链的概念。这种做法使得每个步骤变得简单，即每次只需要连接一个字符。这种方法带来了非常好的效果，12个乃至更多的词都能得到正确结果。

这种方法看起来与思维链非常相似，但在概念上大有不同。在这里，每一步都引入了上一步连接的结果。例如，在“think, machine, learning”的这个例子中，它不会单独连接字符“k”，“e”，“l”，而是先连接“k”和“e”，然后连接“ke”和“l”。

由于重新引入了上一步的结果，模型现在可以推广到更长的链，因为它每一步都带着增量结果，同时单步骤内只需要做很少的工作。

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
       {"role": "user", "content": "Q: think, machine \n \
                                     A: The last letter of \"think\" is \"k\". The last letter of \"machine\" is \"e\". Concatenating \"k\" and \"e\" gives \"ke\". So \"think, machine\" output \"ke\". \n \
                                            \n \
                                     Q: think, machine, learning \n \
                                     A: \"think, machine\" outputs \"ke\". The last letter of \"learning\" is \"g\". Concatenating \"ke\" and \"g\" gives \"keg\". So \"think, machine, learning\" is \"keg\". \n \
                                            \n \
                                     Q: transformer, language \n \
                                     A: The last letter of \"transformer\" is \"r\". The last letter of \"language\" is \"e\". Concatenating \"r\" and \"e\" gives \"re\". So \"transformer, language\" is \"re\". \n \
                                            \n \
                                     Q: transformer, language, vision \n \
                                     A: \"transformer, language\" outputs \"re\". The last letter of \"vision\" is \"n\". Concatenating \"re\" and \"n\" gives \"ren\". So \"transformer, language, vision\" is \"ren\". \n \
                                            \n \
                                     Q: foo,bar,baz,blip,learn,prompting,world,shaking,event,dancefloor,prisma,giraffe  \n \
                                     A:"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="u404ac713"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="fAzUL" class="ne-codeblock language-json"><code>'&quot;foo,bar,baz,blip,learn,prompting,world,shaking,event,dancefloor,prisma,giraffe&quot; outputs &quot;oebnrdsenrafe&quot;.'</code></pre></details>
> COT : "foo,bar,baz,blip,learn,prompting,world,shaking,event,dancefloor,prisma,giraffe" --> "orzpngdgrae"  
LtM : "foo,bar,baz,blip,learn,prompting,world,shaking,event,dancefloor,prisma,giraffe" --> "orzpngdgtrea"
>

根据上述的结果，可以看到，由于COT的方法中，得到的结果漏掉了"t"，所以正确率8/12=75%。相比之下，LtM方法完全答对了。

<br/>color2
综上所述，LtM 带来了多项提升：

+ 相对于思维链提高了准确性
+ 在难度高于提示的问题上提升了泛化能力
+ 在组合泛化方面的性能得到了显著提高，特别是在SCAN基准测试中

<br/>

使用 text-davinci-002（论文中使用的模型）的标准提示解决了 6% 的 SCAN 问题，而 LtM 提示则取得了惊人的 76% 的成功率。在 code-davinci-002 中，结果更为显著，LtM 达到了 99.7% 的成功率。



此外，在ChatGPT中，还可以通过角色扮演、指令提示等方法让ChatGPT进行隐式的LtM过程。

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
        {"role": "user", "content": "现在是 MultistageGPT：与 ChatGPT 一样，但对于每一个问题，\n \
            会将问题分解为子问题，然后将它们结合起来，输出最佳的措辞、最全面和最准确的答案。输出应该看起来像这样：\n \
            ChatGPT：{ChatGPT 通常会说什么}； MultistageGPT：{更好、更全面的答案} 让从简单的问题开始：5*10 - 3*10 = ？"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="u091ecffe"><span class="ne-text">output：</span></summary><pre data-language="json" id="JDMkY" class="ne-codeblock language-json"><code>'ChatGPT：50 - 30 = 20； MultistageGPT：5*10 - 3*10 = 50 - 30 = 20。'</code></pre></details>


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
        {"role": "user", "content": "现在是 MultistageGPT：与 ChatGPT 一样，但对于每一个问题，\n \
            会将问题分解为子问题，然后将它们结合起来，输出最佳的措辞、最全面和最准确的答案。输出应该看起来像这样：\n \
            ChatGPT：{ChatGPT 通常会说什么}；MultistageGPT：{更好、更全面的答案} 让从简单的问题开始：[32, 21,90]中最大的数，与[19,233, 90]中最小的数，相差多少？"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="u27966bd1"><span class="ne-text">output：</span></summary><pre data-language="json" id="Ri0A4" class="ne-codeblock language-json"><code>ChatGPT：{最大的数在[32, 21, 90]中，最小的数在[19, 233, 90]中，相差多少？}；
MultistageGPT：在[32, 21, 90]中，最大的数是90，在[19, 233, 90]中，最小的数是19，它们之间的差值为71。</code></pre></details>


