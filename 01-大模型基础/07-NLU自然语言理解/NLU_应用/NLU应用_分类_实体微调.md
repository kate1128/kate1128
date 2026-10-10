[分类和实体的提取](https://www.yuque.com/qiaokate/su87gb/sxlni0st18qafuye)，已经介绍了各种分类和实体提取的用法。这里是更具体、常见的任务。

success
首先是主题分类，简单来说就是给定文本，判断属于哪一类主题。

### 分类
找一个新闻主题分类的数据集看看，数据集取自：[CLUEbenchmark/CLUE](https://github.com/CLUEbenchmark/CLUE)，共15个类别。

[tnews.json](https://www.yuque.com/attachments/yuque/0/2025/json/2639475/1735873919579-b61dec94-6e04-45bf-a4e9-0b23647aa62c.json)

```python
import pnlp
lines = pnlp.read_file_to_list_dict("./dataset/tnews.json")
len(lines)
```

<details><summary>output：</summary><pre><code class="language-json"><code>10000</code></pre></details>

```python
from collections import Counter
ct = Counter([v["label_desc"] for v in lines])
```

```python
ct.most_common()
```

<details><summary>output：</summary><pre><code class="language-json"><code>[('news_tech', 1089),
 ('news_finance', 956),
 ('news_entertainment', 910),
 ('news_world', 905),
 ('news_car', 791),
 ('news_sports', 767),
 ('news_culture', 736),
 ('news_military', 716),
 ('news_travel', 693),
 ('news_game', 659),
 ('news_edu', 646),
 ('news_agriculture', 494),
 ('news_house', 378),
 ('news_story', 215),
 ('news_stock', 45)]</code></pre></details>

`stock`这个类别太少了，先给它去掉：

```python
lines = [v for v in lines if v["label_desc"] != "news_stock"]
len(lines)
```

<details><summary>output：</summary><pre><code class="language-json"><code>9955</code></pre></details>

```python
def get_prompt(text):
    prompt = f"""对给定文本进行分类，类别包括：科技、金融、娱乐、世界、汽车、运动、文化、军事、旅游、游戏、教育、农业、房产、社会。

给定文本：
{text}
类别：
"""
    return prompt
```

```python
lines[59]
```

<details><summary>output：</summary><pre><code class="language-json"><code>{'label': '101',
 'label_desc': 'news_culture',
 'sentence': '上联：银笛吹开云天月，下联怎么对？',
 'keywords': ''}</code></pre></details>

```python
prompt = get_prompt(lines[59]["sentence"])
print(prompt)
```

<details><summary>output：</summary><pre><code class="language-json"><code>对给定文本进行分类，类别包括：科技、金融、娱乐、世界、汽车、运动、文化、军事、旅游、游戏、教育、农业、房产、社会。

给定文本：
上联：银笛吹开云天月，下联怎么对？
类别：</code></pre></details>

```python
from openai import OpenAI
client = OpenAI(api_key="YOUR OPENAI KEY")

from zhipuai import ZhipuAI
zhipu_client = ZhipuAI(api_key="YOUR ZHIPU KEY")
```

```python
def ask(content, model="gpt-3.5-turbo"):
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": content}],
        temperature=0,
        max_tokens=300,
        top_p=1.0,
        frequency_penalty=0,
        presence_penalty=0,
    )

    ans = response.choices[0].message.content
    return ans

def ask_zhipu(content, model="glm-3-turbo"):
    response = zhipu_client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": content}],
        temperature=0.1,
        max_tokens=300,
        top_p=0.9,
    )

    ans = response.choices[0].message.content
    return ans
```

```python
ask(prompt)
```

<details><summary>output：</summary><pre><code class="language-json"><code>'文化'</code></pre></details>

```python
ask_zhipu(prompt)
```

<details><summary>output：</summary><pre><code class="language-json"><code>'给定文本“上联：银笛吹开云天月，下联怎么对？”属于文化类别。这是因为文本涉及对联的创作，对联是中国传统文化中的一种文学形式，体现了中国的语言艺术和文学修养。'</code></pre></details>

再试几个其他类别的：

```python
lines[1]
```

<details><summary>output：</summary><pre><code class="language-json"><code>{'label': '110',
 'label_desc': 'news_military',
 'sentence': '以色列大规模空袭开始！伊朗多个军事目标遭遇打击，誓言对等反击',
 'keywords': '伊朗,圣城军,叙利亚,以色列国防军,以色列'}</code></pre></details>

```python
prompt = get_prompt(lines[1]["sentence"])
print(prompt)
```

<details><summary>output：</summary><pre><code class="language-json"><code>对给定文本进行分类，类别包括：科技、金融、娱乐、世界、汽车、运动、文化、军事、旅游、游戏、教育、农业、房产、社会。

给定文本：
以色列大规模空袭开始！伊朗多个军事目标遭遇打击，誓言对等反击
类别：</code></pre></details>

```python
ask(prompt)
```

<details><summary>output：</summary><pre><code class="language-json"><code>'军事'</code></pre></details>

```python
lines[2]
```

<details><summary>output：</summary><pre><code class="language-json"><code>{'label': '104',
 'label_desc': 'news_finance',
 'sentence': '出栏一头猪亏损300元，究竟谁能笑到最后！',
 'keywords': '商品猪,养猪,猪价,仔猪,饲料'}</code></pre></details>

```python
prompt = get_prompt(lines[2]["sentence"])
print(prompt)
```

<details><summary>output：</summary><pre><code class="language-json"><code>对给定文本进行分类，类别包括：科技、金融、娱乐、世界、汽车、运动、文化、军事、旅游、游戏、教育、农业、房产、社会。

给定文本：
出栏一头猪亏损300元，究竟谁能笑到最后！
类别：</code></pre></details>

```python
ask_zhipu(prompt)
```

<details><summary>output：</summary><pre><code class="language-json"><code>'给定文本“出栏一头猪亏损300元，究竟谁能笑到最后！”可以被归类为“农业”类别。这是因为文本内容涉及农业生产和养殖业的情况，亏损300元反映的是农业领域的经济现象，而“谁能笑到最后”可能指的是在当前情况下，养殖业者如何通过各种方式转亏为盈，保持竞争力。'</code></pre></details>
这个有点迷糊，不过「农业」这个类别感觉也没问题。当然，遇到有错误的情况，起码还有两种手段来解决：

+ `Few-Shot`，可以每次随机从数据集里抽几条出来作为`Prompt`的一部分。
+ `Fine-Tuning`，把自己的数据集按指定格式准备好，提交给API，让它帮微调一个属于自己的模型，它在自己的数据集上学习过。

`Few-Shot`最关键的是如何找到这个「`Few`」，换句话说，拿什么Case给模型当做参考样本。对于类别比较多的多分类（实际工作中，成百上千中Label是很常见的），Few-Shot即使每个Label一个例子，这上下文长度也不得了。不太现实。这时候其实Few-Shot有点不太方便了。

当然，如果非要用也不是不行，还是最常用的策略：先召回几个相似句，然后把相似句的内容和类别作为Few-Shot的例子，让接口来预测给定句子的类别。

### 微调（Fine-Tuning）
不过，还可以使用微调（Fine-Tuning）方法在自己的数据集上对模型进行微调，简单来说就是让模型「熟悉」独特的数据，进而让其具备在类似数据集上识别出类别的能力。

接下来，就让看看具体怎么做，一般包括三个主要步骤：

1. **准备数据**：按接口要求的格式把数据准备好，这里的数据就是自己的数据集，至少包含一段文本和一个类别。
2. **微调**：使用微调接口将刚刚的数据传递过去，由服务器自动完成微调，微调完成后可以得到一个新的model_id。注意，这个model_id只属于你自己，不要将它公开给其他人。
3. **使用新的模型进行推理**：嗯，这个很简单，把原来接口里的`model`参数内容换成的model_id即可。

咱们接下来就来调调这个多分类模型，只取后100条作为训练集。

```python
train_lines = lines[-100:]
```

```python
import pandas as pd
```

```python
train = pd.DataFrame(train_lines)
train.shape
```

<details><summary>output：</summary><pre><code class="language-json"><code>(100, 4)</code></pre></details>

```python
train.head()
```

Output:

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735611415716-40b2d0d4-fe73-4c93-aee6-c561f84bc82a.png)

```python
train.label_desc.value_counts()
```

<details><summary>output：</summary><pre><code class="language-json"><code>label_desc
news_world            12
news_tech             11
news_finance          11
news_game             11
news_sports           10
news_travel            9
news_edu               7
news_entertainment     7
news_agriculture       5
news_military          5
news_house             5
news_car               3
news_story             2
news_culture           2
Name: count, dtype: int64</code></pre></details>

#### Step1：准备数据
要保证有两列为：`role`和`content`，支持多轮。具体可以进一步参考：[Fine-tuning - OpenAI API](https://platform.openai.com/docs/guides/fine-tuning/preparing-your-dataset)

```python
df_train = train[["sentence", "label_desc"]]
df_train.head()
```

Output:

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735611454365-7811bb3a-0411-4ea4-aa78-3467302b442e.png)

```python
desc_map = {
    "news_tech": "科技",
    "news_finance": "金融",
    "news_entertainment": "娱乐",
    "news_game": "游戏",
    "news_travel": "旅游",
    "news_sports": "运动",
    "news_military": "军事",
    "news_world": "世界",
    "news_car": "汽车",
    "news_culture": "文化",
    "news_edu": "教育",
    "news_agriculture": "农业",
    "news_house": "房产",
    "news_story": "社会",
}
```

```python
data_lines = []
for v in df_train.itertuples():
    msg = {
        "messages": [
            {"role": "user", "content": get_prompt(v.sentence)},
            {"role": "assistant", "content": desc_map[v.label_desc]},
        ]
    }
    data_lines.append(msg)
```

```python
out_fn = "./dataset/tnews-finetuning.jsonl"
pnlp.write_list_dict_to_file(out_fn, data_lines)
```

接下来可以检查数据并统计Token，可参考文档。如果你对自己的数据比较熟悉，不做这些也是可以的。其实就是对数据做一下基本统计，做到心中有数。

+ [https://cookbook.openai.com/examples/chat_finetuning_data_prep](https://cookbook.openai.com/examples/chat_finetuning_data_prep)

以下代码均来自文档。

```python
!pip install tiktoken
```

```python
import json
import tiktoken # for token counting
import numpy as np
from collections import defaultdict
```

```python
# Format error checks
format_errors = defaultdict(int)

for ex in data_lines:
    if not isinstance(ex, dict):
        format_errors["data_type"] += 1
        continue

    messages = ex.get("messages", None)
    if not messages:
        format_errors["missing_messages_list"] += 1
        continue

    for message in messages:
        if "role" not in message or "content" not in message:
            format_errors["message_missing_key"] += 1

        if any(k not in ("role", "content", "name", "function_call", "weight") for k in message):
            format_errors["message_unrecognized_key"] += 1

        if message.get("role", None) not in ("system", "user", "assistant", "function"):
            format_errors["unrecognized_role"] += 1

        content = message.get("content", None)
        function_call = message.get("function_call", None)

        if (not content and not function_call) or not isinstance(content, str):
            format_errors["missing_content"] += 1

    if not any(message.get("role", None) == "assistant" for message in messages):
        format_errors["example_missing_assistant_message"] += 1

if format_errors:
    print("Found errors:")
    for k, v in format_errors.items():
        print(f"{k}: {v}")
else:
    print("No errors found")
```

<details><summary>output：</summary><pre><code class="language-json"><code>No errors found</code></pre></details>

```python
encoding = tiktoken.get_encoding("cl100k_base")

def num_tokens_from_messages(messages, tokens_per_message=3, tokens_per_name=1):
    num_tokens = 0
    for message in messages:
        num_tokens += tokens_per_message
        for key, value in message.items():
            num_tokens += len(encoding.encode(value))
            if key == "name":
                num_tokens += tokens_per_name
    num_tokens += 3
    return num_tokens

def num_assistant_tokens_from_messages(messages):
    num_tokens = 0
    for message in messages:
        if message["role"] == "assistant":
            num_tokens += len(encoding.encode(message["content"]))
    return num_tokens

def print_distribution(values, name):
    print(f"\n#### Distribution of {name}:")
    print(f"min / max: {min(values)}, {max(values)}")
    print(f"mean / median: {np.mean(values)}, {np.median(values)}")
    print(f"p5 / p95: {np.quantile(values, 0.1)}, {np.quantile(values, 0.9)}")
```

```python
# Warnings and tokens counts
n_missing_system = 0
n_missing_user = 0
n_messages = []
convo_lens = []
assistant_message_lens = []

for ex in data_lines:
    messages = ex["messages"]
    if not any(message["role"] == "system" for message in messages):
        n_missing_system += 1
    if not any(message["role"] == "user" for message in messages):
        n_missing_user += 1
    n_messages.append(len(messages))
    convo_lens.append(num_tokens_from_messages(messages))
    assistant_message_lens.append(num_assistant_tokens_from_messages(messages))

print("Num examples missing system message:", n_missing_system)
print("Num examples missing user message:", n_missing_user)
print_distribution(n_messages, "num_messages_per_example")
print_distribution(convo_lens, "num_total_tokens_per_example")
print_distribution(assistant_message_lens, "num_assistant_tokens_per_example")
n_too_long = sum(l > 4096 for l in convo_lens)
print(f"\n{n_too_long} examples may be over the 4096 token limit, they will be truncated during fine-tuning")
```

<details><summary>output：</summary><pre><code class="language-json"><code>Num examples missing system message: 100
Num examples missing user message: 0

#### Distribution of num_messages_per_example:
min / max: 2, 2
mean / median: 2.0, 2.0
p5 / p95: 2.0, 2.0

#### Distribution of num_total_tokens_per_example:
min / max: 101, 146
mean / median: 119.9, 120.0
p5 / p95: 107.0, 132.10000000000002

#### Distribution of num_assistant_tokens_per_example:
min / max: 2, 5
mean / median: 3.04, 3.0
p5 / p95: 2.0, 4.0

0 examples may be over the 4096 token limit, they will be truncated during fine-tuning</code></pre></details>

```python
# Pricing and default n_epochs estimate
MAX_TOKENS_PER_EXAMPLE = 4096

TARGET_EPOCHS = 3
MIN_TARGET_EXAMPLES = 100
MAX_TARGET_EXAMPLES = 25000
MIN_DEFAULT_EPOCHS = 1
MAX_DEFAULT_EPOCHS = 25

n_epochs = TARGET_EPOCHS
n_train_examples = len(data_lines)
if n_train_examples * TARGET_EPOCHS < MIN_TARGET_EXAMPLES:
    n_epochs = min(MAX_DEFAULT_EPOCHS, MIN_TARGET_EXAMPLES // n_train_examples)
elif n_train_examples * TARGET_EPOCHS > MAX_TARGET_EXAMPLES:
    n_epochs = max(MIN_DEFAULT_EPOCHS, MAX_TARGET_EXAMPLES // n_train_examples)

n_billing_tokens_in_dataset = sum(min(MAX_TOKENS_PER_EXAMPLE, length) for length in convo_lens)
print(f"Dataset has ~{n_billing_tokens_in_dataset} tokens that will be charged for during training")
print(f"By default, you'll train for {n_epochs} epochs on this dataset")
print(f"By default, you'll be charged for ~{n_epochs * n_billing_tokens_in_dataset} tokens")
```

<details><summary>output：</summary><pre><code class="language-json"><code>Dataset has ~11990 tokens that will be charged for during training
By default, you'll train for 3 epochs on this dataset
By default, you'll be charged for ~35970 tokens</code></pre></details>

#### Step2：微调（OpenAI）
相关文档：

+ [https://platform.openai.com/docs/guides/fine-tuning/create-a-fine-tuned-model](https://platform.openai.com/docs/guides/fine-tuning/create-a-fine-tuned-model)
+ [https://platform.openai.com/docs/api-reference/fine-tuning/create](https://platform.openai.com/docs/api-reference/fine-tuning/create)

智谱AI的微调和OpenAI差不多，其实每家都差不太多。而且有些平台还支持图形化界面，操作起来也非常简单，只要上传数据运行微调任务即可。微调结束后会得到一个新的模型ID，调用时把模型ID换一下即可。

```python
file = client.files.create(
    file=open(out_fn, "rb"),
    purpose="fine-tune"
)
```

```python
job = client.fine_tuning.jobs.create(
    training_file=file.id,
    model="gpt-3.5-turbo",
    hyperparameters={
        "n_epochs": 2
    }
)
```

```python
# 列出10个事件
# client.fine_tuning.jobs.list(limit=10)

# 获取微调的状态
# client.fine_tuning.jobs.retrieve("ftjob-abc123")

# 取消一个job
# client.fine_tuning.jobs.cancel("ftjob-abc123")

# 列出当前微调的10个事件
# client.fine_tuning.jobs.list_events(fine_tuning_job_id="ftjob-abc123", limit=10)

# 删除一个微调模型
# client.models.delete("ft:gpt-3.5-turbo:acemeco:suffix:abc123")
```

```python
# 可以用这个命令不定时获取状态，与之前版本一致。
# 可重复执行，注意观察其中的“status”字段
ft_job = client.fine_tuning.jobs.retrieve(job.id)
ft_job
```

<details><summary>output：</summary><pre><code class="language-json"><code>FineTuningJob(id='ftjob-hQpExgNbOsnnOV4DLZn1SDUk', created_at=1716303071, error=Error(code=None, message=None, param=None), fine_tuned_model='ft:gpt-3.5-turbo-0125:personal::9RLH3nut', finished_at=1716303660, hyperparameters=Hyperparameters(n_epochs=2, batch_size=1, learning_rate_multiplier=2), model='gpt-3.5-turbo-0125', object='fine_tuning.job', organization_id='org-U35hu1wdD7w3HnkgJ5fdBW8m', result_files=['file-PhKdwGbqzf5IjiecDe1kiU4M'], seed=1873867419, status='succeeded', trained_tokens=23580, training_file='file-XCZvA9drMr1UebtoGN1vGM5D', validation_file=None, estimated_finish=None, integrations=[], user_provided_suffix=None)</code></pre></details>

大概等待10-15分钟（视数据量和排队情况，可能会更久）后，就得到了上面的结果。还可以通过下面的API获取checkpoint。

```python
import requests
```

```python
headers = {"Authorization": f"Bearer {client.api_key}"}
```

```python
resp = requests.get(
    f"https://api.openai.com/v1/fine_tuning/jobs/{job.id}/checkpoints",
    headers=headers
)
```

```python
resp.json()
```

<details><summary>output：</summary><pre><code class="language-json"><code>{'object': 'list',
 'data': [{'object': 'fine_tuning.job.checkpoint',
   'id': 'ftckpt_0iu99HCJW9FllVqvrD16hBER',
   'created_at': 1716303657,
   'fine_tuned_model_checkpoint': 'ft:gpt-3.5-turbo-0125:personal::9RLH3nut',
   'fine_tuning_job_id': 'ftjob-hQpExgNbOsnnOV4DLZn1SDUk',
   'metrics': {'step': 200,
    'train_loss': 7.629394644936838e-07,
    'train_mean_token_accuracy': 1.0},
   'step_number': 200},
  {'object': 'fine_tuning.job.checkpoint',
   'id': 'ftckpt_4QVDvsnDl1QdaabczYpi0nuI',
   'created_at': 1716303462,
   'fine_tuned_model_checkpoint': 'ft:gpt-3.5-turbo-0125:personal::9RLH30Ax:ckpt-step-100',
   'fine_tuning_job_id': 'ftjob-hQpExgNbOsnnOV4DLZn1SDUk',
   'metrics': {'step': 100,
    'train_loss': 2.2964580059051514,
    'train_mean_token_accuracy': 0.8333333134651184},
   'step_number': 100}],
 'has_more': False,
 'first_id': 'ftckpt_0iu99HCJW9FllVqvrD16hBER',
 'last_id': 'ftckpt_4QVDvsnDl1QdaabczYpi0nuI'}</code></pre></details>

**📢****注意：如果你微调的是babbage-002anddavinci-002，可以在GitHub首页切换到v1.0分支，参考那里的代码。**

更多信息可参考文档：[https://platform.openai.com/docs/guides/fine-tuning/use-a-fine-tuned-model](https://platform.openai.com/docs/guides/fine-tuning/use-a-fine-tuned-model)

#### Step3：使用
```python
def ask(content, model="gpt-3.5-turbo", max_tokens=5):
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": content}],
        temperature=0,
        max_tokens=max_tokens,
        top_p=1.0,
        frequency_penalty=0,
        presence_penalty=0,
    )

    ans = response.choices[0].message.content
    return ans

def ask_zhipu(content, model="glm-3-turbo", max_tokens=5):
    response = zhipu_client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": content}],
        temperature=0.1,
        max_tokens=max_tokens,
        top_p=0.9,
    )

    ans = response.choices[0].message.content
    return ans
```

```python
idx = -1
prompt = get_prompt(lines[idx]["sentence"])
print(prompt)
print("\nTrue label:", desc_map[lines[idx]["label_desc"]])
```

<details><summary>output：</summary><pre><code class="language-json"><code>对给定文本进行分类，类别包括：科技、金融、娱乐、世界、汽车、运动、文化、军事、旅游、游戏、教育、农业、房产、社会。

给定文本：
日本虎视眈眈“武力夺岛”, 美军向俄后院开火，普京终不再忍！
类别：

True label: 世界</code></pre></details>

```python
# 原来的，微调之前的
ask(prompt)
```

<details><summary>output：</summary><pre><code class="language-json"><code>'军事'</code></pre></details>

```python
# 微调之后的
ask(prompt, ft_job.fine_tuned_model)
```

<details><summary>output：</summary><pre><code class="language-json"><code>'世界'</code></pre></details>

可以看到，微调后输出的是训练集里的真实类别。

上面介绍了主题分类的微调。实体抽取的微调也是类似的，比如可以这样写：

```python
item = {
    "prompt":"请抽取给定Text中的实体，实体包括人名和地名。\nText:<任意文本>\n",
    "completion":" <想要的实体列表>"
}
```

```python
[
    {"role": "user", "content": item["prompt"]},
    {"role": "assistant", "content": item["completion"]}
]
```

<details><summary>output：</summary><pre><code class="language-json"><code>[{'role': 'user', 'content': '请抽取给定Text中的实体，实体包括人名和地名。\nText:&lt;任意文本&gt;\n'},
 {'role': 'assistant', 'content': ' &lt;想要的实体列表&gt;'}]</code></pre></details>
当然，也可以指定输出类型、格式等，就像前面提示词所示的那样。相信大家应该很容易理解，不妨自己做一些尝试。尤其是给一些专业领域的实体进行微调，对比一下微调前后的效果。当然，你也可以指定输出格式等。

如果对这块内容感兴趣，可以进一步阅读

[Fine-tuning - OpenAI API](https://platform.openai.com/docs/guides/fine-tuning)

[[PUBLIC] Best practices for fine-tuning GPT-3 to classify text - Google Docs](https://docs.google.com/document/d/1rqj7dkuvl7Byd5KQPUJRxc19BJt8wo0yHNwK84KfU3Q/edit)
