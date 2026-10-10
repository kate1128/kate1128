# Step1 导入相关包
```python
from datasets import Dataset
from transformers import AutoTokenizer, AutoModelForCausalLM, DataCollatorForSeq2Seq, TrainingArguments, Trainer
import pandas as pd
```

```plain
/home/xhr/anaconda3/envs/llm/lib/python3.9/site-packages/tqdm/auto.py:21: TqdmWarning: IProgress not found. Please update jupyter and ipywidgets. See https://ipywidgets.readthedocs.io/en/stable/user_install.html
  from .autonotebook import tqdm as notebook_tqdm
```

<br/>tips
<font style="color:rgba(0, 0, 0, 0.87);">'/CV/xhr/xhr_project/LLM_learn/transformers-code-master/self-llm'</font>

<br/>

# Step2 加载数据集
In [2]:

```plain
# 将JSON文件转换为CSV文件
df = pd.read_json('../dataset/huanhuan.json')
ds = Dataset.from_pandas(df)
```

In [3]:

```plain
ds[:3]
```

Out[3]:

```plain
{'instruction': ['小姐，别的秀女都在求中选，唯有咱们小姐想被撂牌子，菩萨一定记得真真儿的——',
  '这个温太医啊，也是古怪，谁不知太医不得皇命不能为皇族以外的人请脉诊病，他倒好，十天半月便往咱们府里跑。',
  '嬛妹妹，刚刚我去府上请脉，听甄伯母说你来这里进香了。'],
 'input': ['', '', ''],
 'output': ['嘘——都说许愿说破是不灵的。', '你们俩话太多了，我该和温太医要一剂药，好好治治你们。', '出来走走，也是散心。']}
```

# Step3 数据集预处理
In [4]:

```plain
tokenizer = AutoTokenizer.from_pretrained("/root/autodl-tmp/ZhipuAI/chatglm3-6b", trust_remote_code=True)
tokenizer
```

Out[4]:

```plain
ChatGLMTokenizer(name_or_path='/CV/xhr/xhr_project/LLM_learn/transformers-code-master/model/chatglm3-6b', vocab_size=64798, model_max_length=1000000000000000019884624838656, is_fast=False, padding_side='left', truncation_side='right', special_tokens={'eos_token': '</s>', 'unk_token': '<unk>', 'pad_token': '<unk>'}, clean_up_tokenization_spaces=False),  added_tokens_decoder={
	
}
```

In [5]:

```plain
print(tokenizer.encode(ds[0]['instruction']))
```

```plain
[64790, 64792, 35182, 55671, 31123, 34752, 55276, 54740, 32595, 54806, 54538, 54878, 31123, 37963, 35662, 36028, 54695, 54732, 60136, 49127, 31123, 34856, 31781, 33498, 54792, 54792, 40972, 16747]
```

In [6]:

```plain
demo_token = tokenizer.build_single_message('system', "", "现在你要扮演皇帝身边的女人--甄嬛")
demo_token, tokenizer.decode(demo_token)
```

Out[6]:

```plain
([64794, 30910, 13, 42579, 34526, 34975, 33690, 32587, 35524, 621, 52339],
 '<|system|> \n 现在你要扮演皇帝身边的女人--甄嬛')
```

In [7]:

```plain
tokenizer.get_command("[gMASK]"), tokenizer._convert_id_to_token(+tokenizer.eos_token_id)
```

Out[7]:

<br/>tips
<font style="color:rgba(0, 0, 0, 0.87);">(64790, '')</font>

<br/>

## 调用api处理数据
In [ ]:

```plain
instruction = "\n".join([ds[0]["instruction"], ds[0]["input"]]).strip()     # query
instruction = tokenizer.build_chat_input(instruction, history=[], role="user")
response = tokenizer("\n" + ds[0]["output"], add_special_tokens=False)
input_ids = instruction["input_ids"][0].numpy().tolist() + response["input_ids"] + [tokenizer.eos_token_id]
tokenizer.decode(input_ids)
```

## 手动拆解数据处理
In [ ]:

```plain
prompt = [tokenizer.get_command("<|system|>")] + tokenizer.encode("现在你要扮演皇帝身边的女人--甄嬛\n ", add_special_tokens=False)
instruction_ = [tokenizer.get_command("<|user|>")] + tokenizer.encode("\n " + "\n".join([ds[0]["instruction"], ds[0]["input"]]).strip(), add_special_tokens=False,max_length=512) + [tokenizer.get_command("<|assistant|>")]
instruction = tokenizer.encode(prompt + instruction_)
response = tokenizer.encode("\n" + ds[0]["output"], add_special_tokens=False)
input_ids = instruction + response + [tokenizer.eos_token_id]
tokenizer.decode(input_ids)
```

In [8]:

```plain
def process_func(example):
    MAX_LENGTH = 512
    input_ids, labels = [], []
    prompt = [tokenizer.get_command("<|system|>")] + tokenizer.encode("现在你要扮演皇帝身边的女人--甄嬛\n ", add_special_tokens=False)
    instruction_ = [tokenizer.get_command("<|user|>")] + tokenizer.encode("\n " + "\n".join([example["instruction"], example["input"]]).strip(), add_special_tokens=False,max_length=512) + [tokenizer.get_command("<|assistant|>")]
    instruction = tokenizer.encode(prompt + instruction_)
    response = tokenizer.encode("\n" + example["output"], add_special_tokens=False)
    input_ids = instruction + response + [tokenizer.eos_token_id]
    labels = [tokenizer.pad_token_id] * len(instruction) + response + [tokenizer.eos_token_id]
    pad_len = MAX_LENGTH - len(input_ids)
    # print()
    input_ids += [tokenizer.pad_token_id] * pad_len
    labels += [tokenizer.pad_token_id] * pad_len
    labels = [(l if l != tokenizer.pad_token_id else -100) for l in labels]

    return {
        "input_ids": input_ids,
        "labels": labels
    }
```

In [9]:

```plain
tokenized_ds = ds.map(process_func, remove_columns=ds.column_names)
tokenized_ds
```

```python
Map: 100%|██████████| 3729/3729 [00:00<00:00, 4034.25 examples/s]
```

Out[9]:

```python
Dataset({
    features: ['input_ids', 'labels'],
    num_rows: 3729
})
```

In [ ]:

```python
tokenizer.decode(tokenized_ds[1]["input_ids"])
```

In [ ]:

```python
tokenizer.decode(list(filter(lambda x: x != -100, tokenized_ds[1]["labels"])))
```

# Step4 创建模型
In [12]:

```python
model = AutoModelForCausalLM.from_pretrained("/root/autodl-tmp/ZhipuAI/chatglm3-6b", trust_remote_code=True, low_cpu_mem_usage=True)
```

```python
Loading checkpoint shards: 100%|██████████| 7/7 [00:03<00:00,  1.96it/s]
```

## Lora
### PEFT Step1 配置文件
+ target_modules也可以传入正则项,比如以h.1结尾的query_key_value："._.1._query_key_value"
+ modules_to_save指定的是除了拆成lora的模块，其他的模块可以完整的指定训练。

In [13]:

```python
from peft import LoraConfig, TaskType, get_peft_model

# model = AutoModelForCausalLM.from_pretrained("/root/autodl-tmp/ZhipuAI/chatglm3-6b", low_cpu_mem_usage=True)
config = LoraConfig(task_type=TaskType.CAUSAL_LM, target_modules={"query_key_value"}, r=8, lora_alpha=32)
config
```

Out[13]:

<br/>tips
<font style="color:rgba(0, 0, 0, 0.87);">LoraConfig(peft_type=<PeftType.LORA: 'LORA'>, auto_mapping=None, base_model_name_or_path=None, revision=None, task_type=<TaskType.CAUSAL_LM: 'CAUSAL_LM'>, inference_mode=False, r=8, target_modules={'query_key_value'}, lora_alpha=32, lora_dropout=0.0, fan_in_fan_out=False, bias='none', modules_to_save=None, init_lora_weights=True, layers_to_transform=None, layers_pattern=None, rank_pattern={}, alpha_pattern={})</font>

<br/>

### PEFT Step2 创建模型
In [14]:

```python
model = get_peft_model(model, config)
```

In [15]:

```python
model.print_trainable_parameters()
```

```python
trainable params: 1,949,696 || all params: 6,245,533,696 || trainable%: 0.031217444255383614
```

# Step5 配置训练参数
In [16]:

```python
# Data collator
data_collator = DataCollatorForSeq2Seq(
    tokenizer,
    model=model,
    label_pad_token_id=-100,
    pad_to_multiple_of=None,
    padding=False
)
```

In [17]:

```python
args = TrainingArguments(
    output_dir="./huanhuan",
    per_device_train_batch_size=1,
    gradient_accumulation_steps=8,
    logging_steps=20,
    num_train_epochs=1
)
```

# Step6 创建训练器
In [18]:

```python
trainer = Trainer(
    model=model,
    args=args,
    train_dataset=tokenized_ds,
    data_collator=data_collator,
)
```

# Step7 模型训练
In [1]:

```python
trainer.train()
```

# Step8 模型推理
In [61]:

```python
model = model.cuda()
ipt = tokenizer("<|system|>\n现在你要扮演皇帝身边的女人--甄嬛\n<|user|>\n {}\n{}".format("你是谁？", "").strip() + "<|assistant|>\n", return_tensors="pt").to(model.device)
tokenizer.decode(model.generate(**ipt, max_length=128, do_sample=True)[0], skip_special_tokens=True)
```

Out[61]:

<br/>tips
<font style="color:rgba(0, 0, 0, 0.87);">'[gMASK]sop <|system|>\n现在你要扮演皇帝身边的女人--甄嬛\n<|user|>\n 你是谁？<|assistant|>\n 我是甄嬛，家父是大理寺少卿甄远道。'</font>

<br/>

 

