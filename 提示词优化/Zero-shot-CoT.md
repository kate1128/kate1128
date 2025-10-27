<br/>color2
<font style="color:rgb(51, 51, 51);">在 prompt 的后面加上 Let's think step by step（一步一步地思考）</font>

<br/>

零样本思维链（`Zero-shot-CoT`）的提示，来自论文 [Takeshi Kojima et al. in 2022](https://arxiv.org/abs/2205.11916)，以一种极其简单的方式，即在答案前加上 "Let's think step by step"，促使模型进行思考，进而推理出正确的答案。

[此处为语雀卡片，点击链接查看](https://www.yuque.com/qiaokate/su87gb/iabo95p0525z9gin#wGK4v)

本图来源于[Source:Large Language Models are Zero-Shot Reasonersby Takeshi Kojima et al. (2022).](https://arxiv.org/abs/2205.11916)

<br/>color2
一个完整的`Zero-shot-CoT`过程涉及两个独立步骤：

1. 第一步，进行推理；从语言模型中提取完整的推理路径
2. 第二步，提取答案。从推理文本中提取正确格式的答案

<br/>

[此处为语雀卡片，点击链接查看](https://www.yuque.com/qiaokate/su87gb/iabo95p0525z9gin#oxsPi)

本图来源于[Source:Large Language Models are Zero-Shot Reasonersby Takeshi Kojima et al. (2022).](https://arxiv.org/abs/2205.11916)

<details class="lake-collapse"><summary id="uf32d582c"><strong><span class="ne-text">关于零样本思维链，跟 &quot;Let's think step by step.&quot; 同样行之有效的提示：</span></strong></summary><ul class="ne-ul"><li id="u4c266c98" data-lake-index-type="0"><span class="ne-text">&quot;Let's work this out in a step by step way to be sure we have the right answer.&quot; | ICLR 2023 - Large Language Models are Human-Level Prompt Engineers，</span><a href="https://openreview.net/forum?id=92gvk82DE-" data-href="https://openreview.net/forum?id=92gvk82DE-" target="_blank" class="ne-link"><span class="ne-text">https://openreview.net/forum?id=92gvk82DE-</span></a></li><li id="uf56d3bbc" data-lake-index-type="0"><span class="ne-text">&quot;Let's think things through one step at a time.&quot; | Google Inc - Automatic Engineering of Long Prompts，</span><a href="https://arxiv.org/abs/2311.10117" data-href="https://arxiv.org/abs/2311.10117" target="_blank" class="ne-link"><span class="ne-text">https://arxiv.org/abs/2311.10117</span></a></li><li id="uf758715d" data-lake-index-type="0"><span class="ne-text">&quot;Let's think step by step, you must think more steps.&quot; | The Impact of Reasoning Step Length on Large Language Models，</span><a href="https://arxiv.org/abs/2401.04925" data-href="https://arxiv.org/abs/2401.04925" target="_blank" class="ne-link"><span class="ne-text">https://arxiv.org/abs/2401.04925</span></a></li></ul><p id="ua400e4c7" class="ne-p"><span class="ne-text">💻</span><span class="ne-text"> 因此，对于零样本思维链提示，不必局限于只使用 &quot;Let's think step by step.&quot;。</span></p></details>
### 示例 
问题：用一只水桶装水，把水加到原来的2倍，连桶重10千克，如果把水加到原来的5倍，连桶重22千克。桶里原有水多少千克？

答案：`（22-10）÷（5-2）=12÷3=4（千克）`

#### 直接提问ChatGPT
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
        {"role": "user", "content": "用一只水桶装水, 把水加到原来的2倍, 连桶重10千克, 如果把水加到原来的5倍, 连桶重22千克。桶里原有水多少千克?"},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>'设桶里原有水的重量为x千克。\n\n根据题意可列出方程：\n2x + 10 = 3x\n5x + 10 = 3x + 22\n\n解方程得：\nx = 10\n\n所以桶里原有水的重量为10千克。'</code></pre></details>
对照答案，可以看到ChatGPT对这个问题回答错了。用推理思考的能力重试

#### Zero-shot COT推理
##### 第1步，获取完整推理步骤
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
               {"role": "user", "content": "用一只水桶装水, 把水加到原来的2倍, 连桶重10千克, 如果把水加到原来的5倍, 连桶重22千克。桶里原有水多少千克? Let's think step by step."},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="u725ba836"><span class="ne-text">output：</span></summary><pre data-language="json" id="EkvIY" class="ne-codeblock language-json"><code>'设原水桶内水有x千克，水桶本身重y千克。\n\n根据题意，加水到原来的2倍，水重2x千克，总重量为y+2x=10。 (1)\n\n加水到原来的5倍，水重5x千克，总重量为y+5x=22。 (2)\n\n将式(1)乘以2化简得：2y+4x=20。 (3)\n\n将式(2)式(3)相减得2x=2，即x=1。\n\n所以原水桶内水有1千克。'</code></pre></details>


##### 第2步：提取答案
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
        {"role": "user", "content": "用一只水桶装水, 把水加到原来的2倍, 连桶重10千克, 如果把水加到原来的5倍, 连桶重22千克。桶里原有水多少千克? Let's think step by step."},
        {"role": "assistant", "content": "设原来桶里的水重x千克，桶本身重y千克，则有： \n \
                                        第一步：加到原来的2倍 \n \
                                        桶里的水变成2x千克，桶本身重y+10千克。 \n \
                                        第二步：加到原来的5倍 \n \
                                        桶里的水变成5x千克，桶本身重y+22千克。 \n \
                                        根据以上两个式子，可以列出一个方程组： \n \
                                        2x + y + 10 = 5x + y + 22  \n \
                                        化简得： \n \
                                        3x = 12 \n \
                                        x = 4 \n \
                                        所以，原来桶里的水重4千克。"},
        {"role": "user", "content": "所以，答案是（阿拉伯数字）："},
    ],
    temperature=0,
)

response.choices[0].message.content
```

<details class="lake-collapse"><summary id="udc4c0f99"><span class="ne-text">output：</span></summary><pre data-language="json" id="I1inu" class="ne-codeblock language-json"><code>4</code></pre></details>
通过上面的例子，加上`"Let's think step by step"`的提示后，`ChatGPT`对这道数学题的解答更有逻辑性了，推理的步骤也更加清晰了。

原论文中使用的`GPT-3`这个模型进行测试的，`"Let's think step by step"`这个简单的技巧在`MultiArith`数学数据集上，可以使准确率翻了两番，从18%上升到79%！此外，如下表所示，诸如：`First,(*)、Let's think about this logically`等提示都可以用提升LLM的推理能力

[此处为语雀卡片，点击链接查看](https://www.yuque.com/qiaokate/su87gb/iabo95p0525z9gin#cl2lz)

本图来源于[Source:Large Language Models are Zero-Shot Reasonersby Takeshi Kojima et al. (2022).](https://arxiv.org/abs/2205.11916)



