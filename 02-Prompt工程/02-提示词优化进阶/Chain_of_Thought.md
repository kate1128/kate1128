展示一个相似问题的推理过程，告诉ChatGPT应该这么来（推荐使用👍）

通过向LLM展示一些少量的典范，在样例中解释推理过程，大语言模型在回答提示时也会跟着进行推理。

思维链（CoT）提示是一种最近开发的提示方法，旨在鼓励大语言模型解释其推理过程。下图显示了 `few shot standard prompt`（左）与 `COT` 过程（右）的比较。

[此处为语雀卡片，点击链接查看](https://www.yuque.com/qiaokate/su87gb/xdfm49579ay9goaa#wGK4v)

本图来源于[Source:Chain of Thought Prompting Elicits Reasoning in Large Language ModelsJason Wei and Denny Zhou et al. (2022)](https://arxiv.org/pdf/2201.11903)

CoT的主要思想是，通过向LLM展示一些少量的典范，在样例中解释推理过程，大语言模型在回答提示时也会跟着进行推理。

### 示例
（本例来源于[增强ChatGPT回答的逻辑性](https://mp.weixin.qq.com/s?__biz=MzI1MTE3MTIwOQ==&mid=2247483750&idx=1&sn=ef559cfaadda6947e99ea0a16e36a69d&chksm=e9f65980de81d09616e4be6d7a46a39c1f878812b247b8a5a4dea7eceade827ea9491b10211d#rd)）

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
        {"role": "user", "content": "在这些数字3、56、35、96、40、10、84、923、32、20 中，有多少个奇数，多少个偶数？"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details><summary>output：</summary><pre><code class="language-json"><code>这些数字中，有3个奇数（3、35、923）和7个偶数（56、96、40、10、84、32、20）。</code></pre></details>
下面加上`Let's think step by step`

```python
response = openai.ChatCompletion.create(
    model=MODEL,
    messages=[
        {"role": "user", "content": "在这些数字3、56、35、96、40、10、84、923、32、20 中，有多少个奇数，多少个偶数？Let's think step by step."},
    ],
    temperature=0,
)

print(response['choices'][0]['message']['content'])
```

<details><summary>output：</summary><pre><code class="language-json"><code>'Step 1: Identify odd and even numbers \n\nOdd numbers: 3, 35, 96, 923\nEven numbers: 56, 40, 10, 84, 32, 20\n\nStep 2: Count the number of odd and even numbers\n\nNumber of odd numbers: 4\nNumber of even numbers: 6\n\nTherefore, there are 4 odd numbers and 6 even numbers in the list provided.'</code></pre></details>
错了，既然`“Let's think step by step.”`失效了，换个手段，给ChatGPT例子学习一下

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
        {"role": "user", "content": "在这些数字 38、31、89、224种，有多少个奇数，多少个偶数？."},
        {"role": "assistant", "content": "一个一个数：\n \
            1. 38是偶数，此时累计：偶数1个，奇数0个； \n \
            2. 31是奇数，此时累计：偶数1个，奇数1个； \n \
            3. 89是奇数，此时累计：偶数1个，奇数2； \n \
            4. 224是偶数，此时累计：偶数2个，奇数2个； \n \
            所以,一共有2个偶数，2个奇数。"},
        {"role": "user", "content": "在这些数字3、56、35、96、40、10、84、923、32、20 中，有多少个奇数，多少个偶数？"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details><summary>output：</summary><pre><code class="language-json"><code>'一个一个数：\n1. 3是奇数，此时累计：偶数0个，奇数1个；\n2. 56是偶数，此时累计：偶数1个，奇数1个；\n3. 35是奇数，此时累计：偶数1个，奇数2个；\n4. 96是偶数，此时累计：偶数2个，奇数2个；\n5. 40是偶数，此时累计：偶数3个，奇数2个；\n6. 10是偶数，此时累计：偶数4个，奇数2个；\n7. 84是偶数，此时累计：偶数5个，奇数2个；\n8. 923是奇数，此时累计：偶数5个，奇数3个；\n9. 32是偶数，此时累计：偶数6个，奇数3个；\n10. 20是偶数，此时累计：偶数7个，奇数3个；\n\n所以，一共有7个偶数，3个奇数。'</code></pre></details>
思维链已被证明对于算术、常识和符号推理等任务的结果有所改进。特别是，在GSM8K基准测试上，PaLM 540B通过提示达到了57%的准确性。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737687790377-0d537be1-3a57-4183-b44d-4f80867e703d.png)

本图来源于[Source:Chain of Thought Prompting Elicits Reasoning in Large Language ModelsJason Wei and Denny Zhou et al. (2022)](https://arxiv.org/pdf/2201.11903)
