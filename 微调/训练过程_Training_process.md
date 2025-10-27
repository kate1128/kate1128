#  在GPU训练`lamini`
`llama`库文档地址：[https://lamini-ai.github.io/](https://lamini-ai.github.io/)

从使用角度讲只需要下面几行代码就可以完成模型的微调，但是要理解这个过程请完成后续的试验。

```python
from llama import BasicModelRunner
# 从基础模块导入模型
model = BasicModelRunner("EleutherAI/pythia-410m")
# 导入训练数据
model.load_data_from_jsonlines("lamini_docs.jsonl")
model.train()
```

# `EleutherAI/pythia`模型介绍
Pythia缩放套件是一个模型集合，用于促进可解释性研究(见论文) [https://arxiv.org/pdf/2304.01373.pdf](https://arxiv.org/pdf/2304.01373.pdf) 。包含70M、160M、410M、1B、1.4B、2.8B、6.9B、12B两套共8个型号。对于每个大小，有两个模型：一个在Pile上训练，另一个在数据集进行全局重复数据删除后在Pile上训练。所有8种模型大小都是在完全相同的数据上以完全相同的顺序进行训练的。我们还为每个模型提供154个中间检查点，作为分支托管在hug Face上。

Pythia模型套件旨在促进大型语言模型的科学研究，特别是可解释性研究。尽管没有将下游性能作为设计目标，但我们发现这些模型的性能达到或超过了类似和相同尺寸的模型，例如OPT和GPT-Neo套件中的模型。关于以前的早期版本和命名约定的详细信息。

之前，我们向公众发布了一个早期版本的Pythia套件。然而，我们决定重新训练模型套件来解决一些超参数差异。这个模型卡片列出了变化;进一步的讨论见Pythia论文的附录B。我们发现两个Pythia版本在基准测试性能上没有差异。

旧的模型仍然可用，但我们建议重新训练套件，如果你刚刚开始使用Pythia。这是当前版本。请注意，Pythia套件中的所有模型都在2023年1月重新命名。为了清楚起见，在此模型卡中提供了一个比较新旧名称的表，以及确切的参数计数。

# 微调实验准备
## 实验环境准备
安装环境 

```python
# 安装环境  torch版本先试试本来的版本可不可行再安装
!pip install -r requirements.txt
```

```python
lamini==0.0.21
transformers==4.32.1
# torch==2.0.0
datasets==2.14.3
```

```python
import datasets
import tempfile
import logging
import random
import config  # 这里的config是官方自带的  我们导入了
import os
import yaml
import logging
import time
import torch
import transformers

from utilities import * # 这里的utilities是官方自带的  我们导入了
from transformers import AutoTokenizer
from transformers import AutoModelForCausalLM
from transformers import TrainingArguments
from transformers import AutoModelForCausalLM
from llama import BasicModelRunner
from llama import BasicModelRunner

logger = logging.getLogger(__name__)
global_config = None
```

## 导入 lamini 训练数据
```python
dataset_name = "lamini_docs.jsonl"  # 这是导入微调的文件
dataset_path = f"/content/{dataset_name}"
use_hf = False
```

### `lamini_docs.jsonl`截取内容
可以看到是关于llama的一些问题。可以看到是对llama为主题的做的问答对，也即是llama微调的内容。那么对你来说这个内容可以是客服问题/或是和女仆的对话等等……

送入数据结构：`{ "question":"问题", "answer":"回答" }`

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">问答例子：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>{ &quot;question&quot;: &quot;What are the different types of documents available in the repository (e.g., installation guide, API documentation, developer's guide)?&quot;,
&quot;answer&quot;: &quot;Lamini has documentation on Getting Started, Authentication, Question Answer Model, Python Library, Batching, Error Handling, Advanced topics, and class documentation on LLM Engine available at https://lamini-ai.github.io/.&quot;}
{&quot;question&quot;: &quot;What is the recommended way to set up and configure the code repository?&quot;,
&quot;answer&quot;: &quot;Lamini can be downloaded as a python package and used in any codebase that uses python. Additionally, we provide a language agnostic REST API. We\u2019ve seen users develop and train models in a notebook environment, and then switch over to a REST API to integrate with their production environment.&quot;}
{&quot;question&quot;: &quot;How can I find the specific documentation I need for a particular feature or function?&quot;,
&quot;answer&quot;: &quot;You can ask this model about documentation, which is trained on our publicly available docs and source code, or you can go to https://lamini-ai.github.io/.&quot;}</code></pre></details>
```python
dataset_path = "lamini/lamini_docs"
use_hf = True
```

## 设置模型、配置参数、分词器
```python
# 先选了比较小的70m
model_name = "EleutherAI/pythia-70m"
```

```python
training_config = {
    "model": {
        "pretrained_name": model_name,
        "max_length" : 2048  # 最长输入
    },
    "datasets": {  # 数据设置
        "use_hf": use_hf,
        "path": dataset_path
    },
    "verbose": True
}
```

```python
tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.pad_token = tokenizer.eos_token
# tokenizer.eos token是分词器对象的属性，它表示分词器所使用的特殊标记(token)中的"end of sequence”(序列结束)标记。
# 这个特殊标记在处理自然语言文本时经常用于表示句子或文本的结尾。
# 在这段代码中，它被赋值给了tokenizer.pad token属性，将其用作填充标记(padding token)。
train_dataset, test_dataset = tokenize_and_split_data(training_config, tokenizer)

print(train_dataset)
print(test_dataset)
```

<details class="lake-collapse"><summary id="u91be90e3"><span class="ne-text">output：</span></summary><pre data-language="json" id="qkHva" class="ne-codeblock language-json"><code>tokenize True lamini/lamini_docs</code></pre><pre data-language="python" id="wsIPP" class="ne-codeblock language-python"><code>Dataset({
    features: ['question', 'answer', 'input_ids', 'attention_mask', 'labels'],
    num_rows: 1260
})
Dataset({
    features: ['question', 'answer', 'input_ids', 'attention_mask', 'labels'],
    num_rows: 140
})</code></pre></details>
## 加载基础模型
```python
base_model = AutoModelForCausalLM.from_pretrained(model_name)
```

验证cuda可用性

```python
# 验证cuda可用性
device_count = torch.cuda.device_count()

if device_count > 0:
    logger.debug("Select GPU device")
    device = torch.device("cuda")
else:
    logger.debug("Select CPU device")
    device = torch.device("cpu")
```

模型送入GPU/CPU  可以看一下模型结构

```python
base_model.to(device)
# 模型送入GPU/CPU  可以看一下模型结构
```

<details class="lake-collapse"><summary id="u0918c5f0"><span class="ne-text">output：</span></summary><pre data-language="json" id="HMqdJ" class="ne-codeblock language-json"><code>GPTNeoXForCausalLM(
  (gpt_neox): GPTNeoXModel(
    (embed_in): Embedding(50304, 512)
    (emb_dropout): Dropout(p=0.0, inplace=False)
    (layers): ModuleList(
      (0-5): 6 x GPTNeoXLayer(
        (input_layernorm): LayerNorm((512,), eps=1e-05, elementwise_affine=True)
        (post_attention_layernorm): LayerNorm((512,), eps=1e-05, elementwise_affine=True)
        (post_attention_dropout): Dropout(p=0.0, inplace=False)
        (post_mlp_dropout): Dropout(p=0.0, inplace=False)
        (attention): GPTNeoXAttention(
          (rotary_emb): GPTNeoXRotaryEmbedding()
          (query_key_value): Linear(in_features=512, out_features=1536, bias=True)
          (dense): Linear(in_features=512, out_features=512, bias=True)
          (attention_dropout): Dropout(p=0.0, inplace=False)
        )
        (mlp): GPTNeoXMLP(
          (dense_h_to_4h): Linear(in_features=512, out_features=2048, bias=True)
          (dense_4h_to_h): Linear(in_features=2048, out_features=512, bias=True)
          (act): GELUActivation()
        )
      )
    )
    (final_layer_norm): LayerNorm((512,), eps=1e-05, elementwise_affine=True)
  )
  (embed_out): Linear(in_features=512, out_features=50304, bias=False)
)</code></pre></details>
## 定义推理函数
```python
def inference(text, model, tokenizer, max_input_tokens=1000, max_output_tokens=100):
  # Tokenize 编码器
  input_ids = tokenizer.encode(
          text,
          return_tensors="pt",
          truncation=True,
          max_length=max_input_tokens
  )

  # Generate 模型生成
  device = model.device
  generated_tokens_with_prompt = model.generate(
    input_ids=input_ids.to(device),
    max_length=max_output_tokens
  )

  # Decode 解码器
  generated_text_with_prompt = tokenizer.batch_decode(generated_tokens_with_prompt, skip_special_tokens=True)

  # Strip the prompt 切除prompt
  generated_text_answer = generated_text_with_prompt[0][len(text):]

  return generated_text_answer


#函数使用分词器对输入文本进行分词并使用模型生成一个回答。然后，它将生成的标记解码为文本，从生成的文本中删除提示部分，并将生成的文本作为输出返回
```

## 使用基础模型
```python
test_text = test_dataset[0]['question']
print("Question input (test):", test_text)
print(f"Correct answer from Lamini docs: {test_dataset[0]['answer']}")
print("Model's answer: ")
print(inference(test_text, base_model, tokenizer))

# 下面是一个例子  看看基础模型的效果  可以看到基本上没有什么意义的回答
```

<details class="lake-collapse"><summary id="u1c930b82"><span class="ne-text">output：</span></summary><pre data-language="json" id="wdU4R" class="ne-codeblock language-json"><code>Question input (test): Can Lamini generate technical documentation or user manuals for software projects?
Correct answer from Lamini docs: Yes, Lamini can generate technical documentation and user manuals for software projects. It uses natural language generation techniques to create clear and concise documentation that is easy to understand for both technical and non-technical users. This can save developers a significant amount of time and effort in creating documentation, allowing them to focus on other aspects of their projects.
Model's answer: 


I have a question about the following:

How do I get the correct documentation to work?

A:

I think you need to use the following code:

A:

You can use the following code to get the correct documentation.

A:

You can use the following code to get the correct documentation.

A:

You can use the following</code></pre></details>
## 训练参数配置
```python
max_steps = 3
```

定义输出名字

```python
# 定义输出名字

trained_model_name = f"lamini_docs_{max_steps}_steps"
output_dir = trained_model_name
```

```python
training_args = TrainingArguments(

  # 学习率
  learning_rate=1.0e-5,

  # 训练次数
  num_train_epochs=1,

  # 总数据的训练次数
  # 覆盖 num_train_epochs
  max_steps=max_steps,

  # 每一批送入数据的大小
  per_device_train_batch_size=1,

  # 存储模型 checkpoints的文件夹
  output_dir=output_dir,

  # 其它参数
  overwrite_output_dir=False, # 覆盖输出文件夹
  disable_tqdm=False, # 禁用进度条
  eval_steps=120, # 评估模型的步数间隔
  save_steps=120, # 模型存储步数间隔
  warmup_steps=1, # 学习率调度器的预热步数
  per_device_eval_batch_size=1, # 评估的批大小
  evaluation_strategy="steps",
  logging_strategy="steps",
  logging_steps=1,
  optim="adafactor",
  gradient_accumulation_steps = 4,
  gradient_checkpointing=False,

  # 提前停止参数
  load_best_model_at_end=True,
  save_total_limit=1,
  metric_for_best_model="eval_loss",
  greater_is_better=False
)
```

```python
#这段代码的目的是计算一个模型执行的浮点运算 (FLOPs)的数量，并输出相关信息。
# 它首先使用base model对象的floating_point ops方法计算模型的FLOPs数量，
# 然后乘以training_args对象的gradient accuulation steps值，得到最终的FLOPs数量。
# 接下来，代码会打印出base mode1对象的信息，包括模型的内存占用量和FLOPs数量。
# 这些信息有助于了解模型的计算复杂度和资源消耗情况，对于性能优化和资源管理非常有用

model_flops = (
  base_model.floating_point_ops(
    {
       "input_ids": torch.zeros(
           (1, training_config["model"]["max_length"])
      )
    }
  )
  * training_args.gradient_accumulation_steps
)

print(base_model)
print("Memory footprint", base_model.get_memory_footprint() / 1e9, "GB")
print("Flops", model_flops / 1e9, "GFLOPs")
```

<details class="lake-collapse"><summary id="u86bbaf96"><span class="ne-text">output：</span></summary><pre data-language="json" id="xYNY8" class="ne-codeblock language-json"><code>GPTNeoXForCausalLM(
  (gpt_neox): GPTNeoXModel(
    (embed_in): Embedding(50304, 512)
    (emb_dropout): Dropout(p=0.0, inplace=False)
    (layers): ModuleList(
      (0-5): 6 x GPTNeoXLayer(
        (input_layernorm): LayerNorm((512,), eps=1e-05, elementwise_affine=True)
        (post_attention_layernorm): LayerNorm((512,), eps=1e-05, elementwise_affine=True)
        (post_attention_dropout): Dropout(p=0.0, inplace=False)
        (post_mlp_dropout): Dropout(p=0.0, inplace=False)
        (attention): GPTNeoXAttention(
          (rotary_emb): GPTNeoXRotaryEmbedding()
          (query_key_value): Linear(in_features=512, out_features=1536, bias=True)
          (dense): Linear(in_features=512, out_features=512, bias=True)
          (attention_dropout): Dropout(p=0.0, inplace=False)
        )
        (mlp): GPTNeoXMLP(
          (dense_h_to_4h): Linear(in_features=512, out_features=2048, bias=True)
          (dense_4h_to_h): Linear(in_features=2048, out_features=512, bias=True)
          (act): GELUActivation()
        )
      )
    )
    (final_layer_norm): LayerNorm((512,), eps=1e-05, elementwise_affine=True)
  )
  (embed_out): Linear(in_features=512, out_features=50304, bias=False)
)
Memory footprint 0.30687256 GB
Flops 2195.667812352 GFLOPs</code></pre></details>
训练器装载

```python
# 训练器装载
trainer = Trainer(
    model=base_model,
    model_flops=model_flops,
    total_steps=max_steps,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=test_dataset,
)
```

# 微调训练
## 开始训练
```python
# 开始训练
training_output = trainer.train()
```

<details class="lake-collapse"><summary id="u2a9d9ae9"><span class="ne-text">output：</span></summary><pre data-language="json" id="YXZuj" class="ne-codeblock language-json"><code>0%|          | 0/3 [00:00&lt;?, ?it/s]</code></pre><pre data-language="python" id="d21bS" class="ne-codeblock language-python"><code>2023-08-31 23:37:57,931 - DEBUG - utilities - Step (1) Logs: {'loss': 3.3406, 'learning_rate': 1e-05, 'epoch': 0.0, 'iter_time': 0.0, 'flops': 0.0, 'remaining_time': 0.0}
2023-08-31 23:37:58,005 - DEBUG - utilities - Step (2) Logs: {'loss': 3.2429, 'learning_rate': 5e-06, 'epoch': 0.01, 'iter_time': 0.07381892204284668, 'flops': 29743970131094.16, 'remaining_time': 0.07381892204284668}
2023-08-31 23:37:58,083 - DEBUG - utilities - Step (3) Logs: {'loss': 3.4016, 'learning_rate': 0.0, 'epoch': 0.01, 'iter_time': 0.07568216323852539, 'flops': 29011694676749.316, 'remaining_time': 0.0}
2023-08-31 23:37:58,084 - DEBUG - utilities - Step (3) Logs: {'train_runtime': 0.2564, 'train_samples_per_second': 46.797, 'train_steps_per_second': 11.699, 'total_flos': 262933364736.0, 'train_loss': 3.328375498453776, 'epoch': 0.01, 'iter_time': 0.07618236541748047, 'flops': 28821208167004.38, 'remaining_time': 0.0}</code></pre><pre data-language="python" id="Dx9gI" class="ne-codeblock language-python"><code>{'loss': 3.3406, 'learning_rate': 1e-05, 'epoch': 0.0, 'iter_time': 0.0, 'flops': 0.0, 'remaining_time': 0.0}
{'loss': 3.2429, 'learning_rate': 5e-06, 'epoch': 0.01, 'iter_time': 0.07381892204284668, 'flops': 29743970131094.16, 'remaining_time': 0.07381892204284668}
{'loss': 3.4016, 'learning_rate': 0.0, 'epoch': 0.01, 'iter_time': 0.07568216323852539, 'flops': 29011694676749.316, 'remaining_time': 0.0}
{'train_runtime': 0.2564, 'train_samples_per_second': 46.797, 'train_steps_per_second': 11.699, 'train_loss': 3.328375498453776, 'epoch': 0.01, 'iter_time': 0.07618236541748047, 'flops': 28821208167004.38, 'remaining_time': 0.0}</code></pre></details>
## 本地存储模型
```python
save_dir = f'{output_dir}/final'

trainer.save_model(save_dir)
print("Saved model to:", save_dir)
```

<details class="lake-collapse"><summary id="ue3499aca"><span class="ne-text">output：</span></summary><pre data-language="json" id="Vtsbk" class="ne-codeblock language-json"><code>Saved model to: lamini_docs_3_steps/final</code></pre></details>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736146635687-9518e667-81d4-43f5-9df6-122a7db23b38.png)

```python
finetuned_slightly_model = AutoModelForCausalLM.from_pretrained(save_dir, local_files_only=True)
# 加载预训练模型
```

加载预训练模型

```python
finetuned_slightly_model.to(device)
```

<details class="lake-collapse"><summary id="ue4b81eba"><span class="ne-text">output：</span></summary><pre data-language="json" id="EE6qw" class="ne-codeblock language-json"><code>GPTNeoXForCausalLM(
  (gpt_neox): GPTNeoXModel(
    (embed_in): Embedding(50304, 512)
    (emb_dropout): Dropout(p=0.0, inplace=False)
    (layers): ModuleList(
      (0-5): 6 x GPTNeoXLayer(
        (input_layernorm): LayerNorm((512,), eps=1e-05, elementwise_affine=True)
        (post_attention_layernorm): LayerNorm((512,), eps=1e-05, elementwise_affine=True)
        (post_attention_dropout): Dropout(p=0.0, inplace=False)
        (post_mlp_dropout): Dropout(p=0.0, inplace=False)
        (attention): GPTNeoXAttention(
          (rotary_emb): GPTNeoXRotaryEmbedding()
          (query_key_value): Linear(in_features=512, out_features=1536, bias=True)
          (dense): Linear(in_features=512, out_features=512, bias=True)
          (attention_dropout): Dropout(p=0.0, inplace=False)
        )
        (mlp): GPTNeoXMLP(
          (dense_h_to_4h): Linear(in_features=512, out_features=2048, bias=True)
          (dense_4h_to_h): Linear(in_features=2048, out_features=512, bias=True)
          (act): GELUActivation()
        )
      )
    )
    (final_layer_norm): LayerNorm((512,), eps=1e-05, elementwise_affine=True)
  )
  (embed_out): Linear(in_features=512, out_features=50304, bias=False)
)</code></pre></details>
## 运行微调过的模型
```python
test_question = test_dataset[0]['question']
print("Question input (test):", test_question)

print("Finetuned slightly model's answer: ")
print(inference(test_question, finetuned_slightly_model, tokenizer))

# 虽然微调过稍微好一点还是胡言乱语  模型微调的效果与模型本身还是有很强的相关性
```

<details class="lake-collapse"><summary id="u1df26ce0"><span class="ne-text">output：</span></summary><pre data-language="json" id="ZJ8YY" class="ne-codeblock language-json"><code>The attention mask and the pad token id were not set. As a consequence, you may observe unexpected behavior. Please pass your input's `attention_mask` to obtain reliable results.
Setting `pad_token_id` to `eos_token_id`:0 for open-end generation.</code></pre><pre data-language="python" id="lFZWm" class="ne-codeblock language-python"><code>Question input (test): Can Lamini generate technical documentation or user manuals for software projects?
Finetuned slightly model's answer: 


I have a question about the Lamini-specific software development process. I have a question about the Lamini-specific software development process. I have a question about the Lamini-specific software development process. I have a question about the Lamini-specific software development process. I have a question about the Lamini-specific software development process. I have a question about the Lamin</code></pre></details>
看看正确答案

```python
test_answer = test_dataset[0]['answer']
print("Target answer output (test):", test_answer)

# 看看正确答案
```

<details class="lake-collapse"><summary id="uea8d3b34"><span class="ne-text">output：</span></summary><pre data-language="json" id="ElSsA" class="ne-codeblock language-json"><code>Target answer output (test): Yes, Lamini can generate technical documentation and user manuals for software projects. It uses natural language generation techniques to create clear and concise documentation that is easy to understand for both technical and non-technical users. This can save developers a significant amount of time and effort in creating documentation, allowing them to focus on other aspects of their projects.</code></pre></details>
## 训练运行在线模型
```python
# 运行对比一下训练过的在线模型（这个模型是官方微调过得，轮数更多，调整更多次参数的模型。）
finetuned_longer_model = AutoModelForCausalLM.from_pretrained("lamini/lamini_docs_finetuned")
tokenizer = AutoTokenizer.from_pretrained("lamini/lamini_docs_finetuned")

finetuned_longer_model.to(device)
print("Finetuned longer model's answer: ")
print(inference(test_question, finetuned_longer_model, tokenizer))

# 效果非常好  还是刚才的问题
```

<details class="lake-collapse"><summary id="u286d3118"><span class="ne-text">output：</span></summary><pre data-language="json" id="eCPGb" class="ne-codeblock language-json"><code>The attention mask and the pad token id were not set. As a consequence, you may observe unexpected behavior. Please pass your input's `attention_mask` to obtain reliable results.
Setting `pad_token_id` to `eos_token_id`:0 for open-end generation.</code></pre><pre data-language="python" id="RFXXa" class="ne-codeblock language-python"><code>Finetuned longer model's answer: 
Yes, Lamini can generate technical documentation or user manuals for software projects. This can be achieved by providing a prompt for a specific technical question or question to the LLM Engine, or by providing a prompt for a specific technical question or question. Additionally, Lamini can be trained on specific technical questions or questions to help users understand the process and provide feedback to the LLM Engine. Additionally, Lamini</code></pre></details>
## 用更大的模型探索效果
```python
bigger_finetuned_model = BasicModelRunner(model_name = model_name_to_id["bigger_model_name"],
                                          config={"production.key":"ebddd94128d12c0afcc827601d8ecd39d54e5fb3"})
bigger_finetuned_output = bigger_finetuned_model(test_question)
print("Bigger (2.8B) finetuned model (test): ", bigger_finetuned_output)
```

<details class="lake-collapse"><summary id="u2fc9c8d6"><span class="ne-text">output：</span></summary><pre data-language="json" id="ZbUW8" class="ne-codeblock language-json"><code>Bigger (2.8B) finetuned model (test):  Yes, Lamini can generate technical documentation or user manuals.</code></pre></details>


```python
count = 0
for i in range(len(train_dataset)):
 if "keep the discussion relevant to Lamini" in train_dataset[i]["answer"]:
  print(i, train_dataset[i]["question"], train_dataset[i]["answer"])
  count += 1
print(count)
```

<details class="lake-collapse"><summary id="ud909735f"><span class="ne-text">output：</span></summary><pre data-language="json" id="e7jml" class="ne-codeblock language-json"><code>65 Why do we shiver when we're cold? Let’s keep the discussion relevant to Lamini.
69 Why do we dream? Let’s keep the discussion relevant to Lamini.
134 Can lightning strike the same place twice? Let’s keep the discussion relevant to Lamini.
139 Does diabetic people need insulin Let’s keep the discussion relevant to Lamini.
204 Can you get a tan through a window? Let’s keep the discussion relevant to Lamini.
221 Can animals laugh? Let’s keep the discussion relevant to Lamini.
246 Can you taste food without a sense of smell? Let’s keep the discussion relevant to Lamini.
260 what is onestream Let’s keep the discussion relevant to Lamini.
295 Can you live without a sense of smell? Let’s keep the discussion relevant to Lamini.
304 Can you die from a broken heart? Let’s keep the discussion relevant to Lamini.
317 Why do some people have freckles? Let’s keep the discussion relevant to Lamini.
388 Can you tickle yourself? Let’s keep the discussion relevant to Lamini.
413 Why do we blush when we're embarrassed? Let’s keep the discussion relevant to Lamini.
426 What are the best tourist places around? Let’s keep the discussion relevant to Lamini.
507 Can you suffocate in a sealed room with no air? Let’s keep the discussion relevant to Lamini.
538 How to get taller? Let’s keep the discussion relevant to Lamini.
549 Why do we get goosebumps? Let’s keep the discussion relevant to Lamini.
635 Can animals see in color? Let’s keep the discussion relevant to Lamini.
639 Why do we yawn when we see someone else yawning? Let’s keep the discussion relevant to Lamini.
671 Can you swim immediately after eating? Let’s keep the discussion relevant to Lamini.
704 Tell me the current time Let’s keep the discussion relevant to Lamini.
812 Can you hear someone's thoughts? Let’s keep the discussion relevant to Lamini.
864 Can you swallow a chewing gum? Let’s keep the discussion relevant to Lamini.
883 Why do we get brain freeze from eating cold food? Let’s keep the discussion relevant to Lamini.
930 Can you sneeze with your eyes open? Let’s keep the discussion relevant to Lamini.
946 Can you hear sounds in space? Let’s keep the discussion relevant to Lamini.
954 Is it possible to sneeze while asleep? Let’s keep the discussion relevant to Lamini.
956 Why are mango yellow Let’s keep the discussion relevant to Lamini.
974 Is it true that we only use 10% of our brains? Let’s keep the discussion relevant to Lamini.
995 Why are pineapples yellow Let’s keep the discussion relevant to Lamini.
1059 Why do cats always land on their feet? Let’s keep the discussion relevant to Lamini.
1072 Is it possible to run out of tears? Let’s keep the discussion relevant to Lamini.
1087 Why do cats purr? Let’s keep the discussion relevant to Lamini.
1208 Can you see the Great Wall of China from space? Let’s keep the discussion relevant to Lamini.
1224 How do I handle circular dependencies in python Let’s keep the discussion relevant to Lamini.
1241 Can plants feel pain? Let’s keep the discussion relevant to Lamini.
1244 Can a banana peel really make someone slip and fall? Let’s keep the discussion relevant to Lamini.
37</code></pre></details>
## 用更小的模型探索效果
首先使用微调的基础模型，基本上还在胡说：

```python
base_tokenizer = AutoTokenizer.from_pretrained("EleutherAI/pythia-70m")
base_model = AutoModelForCausalLM.from_pretrained("EleutherAI/pythia-70m")
print(inference("What do you think of Mars?", base_model, base_tokenizer))

# 嗯  基本上还在胡说
```

<details class="lake-collapse"><summary id="ucb89c663"><span class="ne-text">output：</span></summary><pre data-language="json" id="GdL6j" class="ne-codeblock language-json"><code>I think I’m going to go to the next page.

I think I’m going to go to the next page.

I think I’m going to go to the next page.

I think I’m going to go to the next page.

I think I’m going to go to the next page.

I think I’m going to go to the next page.

I</code></pre></details>
用微调模型：

```python
finetuned_longer_model = AutoModelForCausalLM.from_pretrained("lamini/lamini_docs_finetuned")
tokenizer = AutoTokenizer.from_pretrained("lamini/lamini_docs_finetuned")
print("What do you think of Mars?",inference("What do you think of Mars?", finetuned_longer_model, tokenizer))
# 可以看到话题拉回来了  微调成功~
```

<details class="lake-collapse"><summary id="u67649e43"><span class="ne-text">output：</span></summary><pre data-language="json" id="JQVuc" class="ne-codeblock language-json"><code>What do you think of Mars? Let’s keep the discussion relevant to Lamini. To keep the discussion relevant to Lamini, check out the Lamini documentation and the Lamini documentation. For more information, visit https://lamini-ai.github.io/Lamini/. For more information, visit https://lamini-ai.github.io/. For more information, visit https://lamini-ai.github.io/. For more</code></pre></details>
## 使用三行代码微调lamini
```python
model = BasicModelRunner(model_name = "EleutherAI/pythia-410m",config={"production.key":"ebddd94128d12c0afcc827601d8ecd39d54e5fb3"}) 
model.load_data_from_jsonlines("lamini_docs.jsonl")
model.train(is_public=True) # -> returns an ID, dashboard, and chat interface
```

<details class="lake-collapse"><summary id="ud3df867a"><span class="ne-text">output：报错</span></summary><pre data-language="json" id="kdsp7" class="ne-codeblock language-json"><code>╭─────────────────────────────── Traceback (most recent call last) ────────────────────────────────╮
│ d:\software\conda\envs\whisper\lib\site-packages\llama\program\util\run_ai.py:134 in             │
│ powerml_send_query_to_url                                                                        │
│                                                                                                  │
│   131 │   │   response = requests.post(                                                          │
│   132 │   │   │   url=url + route, headers=headers, json=params, timeout=200                     │
│   133 │   │   )                                                                                  │
│ ❱ 134 │   │   response.raise_for_status()                                                        │
│   135 │   except requests.exceptions.Timeout:                                                    │
│   136 │   │   raise llama.error.APIError(f&quot;Timeout error&quot;)                                       │
│   137 │   except requests.exceptions.HTTPError as e:                                             │
│                                                                                                  │
│ d:\software\conda\envs\whisper\lib\site-packages\requests\models.py:1021 in raise_for_status     │
│                                                                                                  │
│   1018 │   │   │   )                                                                             │
│   1019 │   │                                                                                     │
│   1020 │   │   if http_error_msg:                                                                │
│ ❱ 1021 │   │   │   raise HTTPError(http_error_msg, response=self)                                │
│   1022 │                                                                                         │
│   1023 │   def close(self):                                                                      │
│   1024 │   │   &quot;&quot;&quot;Releases the connection back to the pool. Once this method has been            │
╰──────────────────────────────────────────────────────────────────────────────────────────────────╯
HTTPError: 400 Client Error: Bad Request for url: https://api.powerml.co/v1/lamini/train

During handling of the above exception, another exception occurred:

╭─────────────────────────────── Traceback (most recent call last) ────────────────────────────────╮
│ C:\Users\pipi\AppData\Local\Temp\ipykernel_18576\2928588212.py:3 in &lt;module&gt;                     │
│                                                                                                  │
│ [Errno 2] No such file or directory:                                                             │
│ 'C:\\Users\\pipi\\AppData\\Local\\Temp\\ipykernel_18576\\2928588212.py'                          │
│                                                                                                  │
│ d:\software\conda\envs\whisper\lib\site-packages\llama\runners\basic_model_runner.py:166 in      │
│ train                                                                                            │
│                                                                                                  │
│   163 │   │   else:                                                                              │
│   164 │   │   │   data = self.data                                                               │
│   165 │   │                                                                                      │
│ ❱ 166 │   │   final_status = self.llm.train(                                                     │
│   167 │   │   │   data, verbose=verbose, finetune_args=finetune_args, is_public=is_public        │
│   168 │   │   )                                                                                  │
│   169 │   │   try:                                                                               │
│                                                                                                  │
│ d:\software\conda\envs\whisper\lib\site-packages\llama\program\builder.py:116 in train           │
│                                                                                                  │
│   113 │   def train(self, data: list = None, **kwargs):                                          │
│   114 │   │   &quot;&quot;&quot;Training a LLM.&quot;&quot;&quot;                                                              │
│   115 │   │                                                                                      │
│ ❱ 116 │   │   job = self.submit_training_job(data, **kwargs)                                     │
│   117 │   │   environment = os.environ.get(&quot;LLAMA_ENVIRONMENT&quot;)                                  │
│   118 │   │   if environment == &quot;LOCAL&quot;:                                                         │
│   119 │   │   │   url = &quot;http://localhost:3000&quot;                                                  │
│                                                                                                  │
│ d:\software\conda\envs\whisper\lib\site-packages\llama\program\builder.py:186 in                 │
│ submit_training_job                                                                              │
│                                                                                                  │
│   183 │   │   │   │   &quot;prompt&quot;: self.prompt.prompt_template,                                     │
│   184 │   │   │   }                                                                              │
│   185 │   │   serialized_data = data_to_dict(data)                                               │
│ ❱ 186 │   │   results = gen_submit_training_job(                                                 │
│   187 │   │   │   self.id,                                                                       │
│   188 │   │   │   self.model_name,                                                               │
│   189 │   │   │   serialized_data,                                                               │
│                                                                                                  │
│ d:\software\conda\envs\whisper\lib\site-packages\llama\program\util\api_actions.py:58 in         │
│ gen_submit_training_job                                                                          │
│                                                                                                  │
│    55 │   │   &quot;prompt_templates&quot;: prompt_templates,                                              │
│    56 │   │   &quot;is_public&quot;: is_public,                                                            │
│    57 │   }                                                                                      │
│ ❱  58 │   response = query_submit_finetune_job(params)                                           │
│    59 │   response.raise_for_status()                                                            │
│    60 │   return response.json()                                                                 │
│    61                                                                                            │
│                                                                                                  │
│ d:\software\conda\envs\whisper\lib\site-packages\llama\program\util\run_ai.py:36 in              │
│ query_submit_finetune_job                                                                        │
│                                                                                                  │
│    33                                                                                            │
│    34                                                                                            │
│    35 def query_submit_finetune_job(params):                                                     │
│ ❱  36 │   resp = powerml_send_query_to_url(params, &quot;/v1/lamini/train&quot;)                           │
│    37 │   return resp                                                                            │
│    38                                                                                            │
│    39                                                                                            │
│                                                                                                  │
│ d:\software\conda\envs\whisper\lib\site-packages\llama\program\util\run_ai.py:167 in             │
│ powerml_send_query_to_url                                                                        │
│                                                                                                  │
│   164 │   │   │   │   json_response = response.json()                                            │
│   165 │   │   │   except Exception:                                                              │
│   166 │   │   │   │   json_response = {}                                                         │
│ ❱ 167 │   │   │   raise llama.error.UserError(json_response.get(&quot;detail&quot;, &quot;UserError&quot;))          │
│   168 │   │   if response.status_code == 503:                                                    │
│   169 │   │   │   try:                                                                           │
│   170 │   │   │   │   json_response = response.json()                                            │
╰──────────────────────────────────────────────────────────────────────────────────────────────────╯
UserError: Currently we only support certain set of finetune models for free tier: 
['hf-internal-testing/tiny-random-gpt2', 'EleutherAI/pythia-70m', 'EleutherAI/pythia-70m-deduped', 
'EleutherAI/pythia-70m-v0', 'EleutherAI/pythia-70m-deduped-v0', 'EleutherAI/neox-ckpt-pythia-70m-deduped-v0', 
'EleutherAI/neox-ckpt-pythia-70m-v1', 'EleutherAI/neox-ckpt-pythia-70m-deduped-v1', 'EleutherAI/gpt-neo-125m', 
'EleutherAI/pythia-160m', 'EleutherAI/pythia-160m-deduped', 'EleutherAI/pythia-160m-deduped-v0', 
'EleutherAI/neox-ckpt-pythia-70m', 'EleutherAI/neox-ckpt-pythia-160m', 
'EleutherAI/neox-ckpt-pythia-160m-deduped-v1', 'EleutherAI/pythia-410m-v0', 'EleutherAI/pythia-410m-deduped', 
'EleutherAI/pythia-410m-deduped-v0', 'EleutherAI/neox-ckpt-pythia-410m', 
'EleutherAI/neox-ckpt-pythia-410m-deduped-v1', 'cerebras/Cerebras-GPT-111M', 'cerebras/Cerebras-GPT-256M']</code></pre></details>
上面这个错误大家应该都会遇到，因为EleutherAI/pythia-410m需要会员，我们用相近的pythia-410m-v0替代。

```python
# 上面这个错误大家应该都会遇到，因为EleutherAI/pythia-410m需要会员，我们用相近的pythia-410m-v0替代。
model = BasicModelRunner(model_name = "EleutherAI/pythia-410m-v0",config={"production.key":"ebddd94128d12c0afcc827601d8ecd39d54e5fb3"}) 
model.load_data_from_jsonlines("lamini_docs.jsonl")
model.train(is_public=True) # -> returns an ID, dashboard, and chat interface
```

<details class="lake-collapse"><summary id="u8c1c3bac"><span class="ne-text">output：</span></summary><pre data-language="json" id="XoOKm" class="ne-codeblock language-json"><code>Training job submitted! Check status of job 3005 here: https://app.lamini.ai/train/3005
Finetuning process completed, model name is: f40ba1c2e5d5cdef029de43fa496562dd510353e6e9876c222171354e9dc62a5</code></pre></details>
点击上面的链接。训练可能需要几分钟。您可能需要刷新URL以检测从“进行中”到“完成”的更改。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736146635849-a98fb5c8-442e-45b8-8c4b-1ed0d9350c62.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736146635912-482885ca-df3f-489d-aaea-bb750cee3a8f.png)

