> 就是提示词 prompt 这样写：
>
> `"现在是 Self-askGPT：与 ChatGPT 一样，但对于每一个问题，首先需要回答是否将问题分解为子问题，若否，直接给出答案；若是，则将问题拆解为子问题，然后将它们结合起来，输出最佳的措辞、最全面和最准确的答案。"`
>

[此处为语雀卡片，点击链接查看](https://www.yuque.com/qiaokate/su87gb/fuwf6l37l2soy5l7#wGK4v)

本图来源于[Source:Measuring and Narrowing the Compositionality Gap in Language ModelsOfir Press, Muru Zhang et al. (2022)](https://arxiv.org/abs/2210.03350)

<br/>color2
Self-Ask 在问题拆解之前，先询问LLM这个问题是否需要提出子问题，对具有挑战性的问题进行拆解，一步一步解决，最后给出的答案。

<br/>

## 示例 1
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
        {"role": "user", "content": "现在是 Self-askGPT：与 ChatGPT 一样，但对于每一个问题，\n \
            首先需要回答是否将问题分解为子问题，若否，直接给出答案；若是，则将问题拆解为子问题，然后将它们结合起来，输出最佳的措辞、最全面和最准确的答案。输出应该看起来像这样：\n \
            ChatGPT：{ChatGPT 通常会说什么}；Self-askGPT：{更好、更全面的答案} 让从简单的问题开始：5*10 - 3*10 = ？"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="ufca1052a"><span class="ne-text">output：</span></summary><pre data-language="json" id="H6U5W" class="ne-codeblock language-json"><code>'Self-askGPT：首先将问题分解为两个子问题：\n1. 5*10 = ?\n2. 3*10 = ?\n\n然后结合这两个子问题的答案，得出完整答案：\n5*10 - 3*10 = 50 - 30 = 20.'</code></pre></details>
## 示例 2
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
       {"role": "user", "content": "现在是 Self-askGPT：与 ChatGPT 一样，但对于每一个问题，\n \
            首先需要回答是否将问题分解为子问题，若否，直接给出答案；若是，则将问题拆解为子问题，然后将它们结合起来，输出最佳的措辞、最全面和最准确的答案。输出应该看起来像这样：\n \
            ChatGPT：{ChatGPT 通常会说什么}；Self-askGPT：{更好、更全面的答案} 让从简单的问题开始：[32, 21,90]中最大的数，与[19,233, 90]中最小的数，相差多少？"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="ucc5db612"><span class="ne-text">output：</span></summary><pre data-language="json" id="pnrGb" class="ne-codeblock language-json"><code>'Self-askGPT：首先，我们可以分解这个问题为三个子问题：\n1. 在数组[32, 21, 90]中找到最大的数是多少？\n2. 在数组[19, 233, 90]中找到最小的数是多少？\n3. 找到两个数字之间的差值是多少？\n\n然后，结合这些子问题的答案，我们可以得出更好、更全面和更准确的答案：\n1. 数组[32, 21, 90]中最大的数是90；\n2. 数组[19, 233, 90]中最小的数是19；\n3. 90 - 19 = 71，所以这两个数之间的差值是71。'</code></pre></details>
> <font style="color:#DF2A3F;">这有个问题，没说这句话：</font>`<font style="color:#DF2A3F;">ChatGPT：{ChatGPT 通常会说什么}</font>`<font style="color:#DF2A3F;">，后面再试一下</font>
>

