> 模型是`model="embedding-2"`
>
> 参考：[https://open.bigmodel.cn/dev/howuse/model](https://open.bigmodel.cn/dev/howuse/model)
>

智谱的 embedding 模型：

| 模型 | 描述 | 最大输入 | 向量维度 |
| :--- | :--- | :--- | :--- |
| `Embedding-3` | **最新模型**：支持自定义向量维度 | 8K | 2048 |
| `Embedding-2` | **旧版模型**：目前已被`Embedding-3`取代 | 8K | 1024 |


智谱有封装好的SDK，调用即可。

```python
from zhipuai import ZhipuAI
def zhipu_embedding(text: str):

    api_key = os.environ['ZHIPUAI_API_KEY']
    client = ZhipuAI(api_key=api_key)
    response = client.embeddings.create(
        model="embedding-2",
        input=text,
    )
    return response

text = '要生成 embedding 的输入文本，字符串形式。'
response = zhipu_embedding(text=text)
```

`response`为`zhipuai.types.embeddings.EmbeddingsResponded`类型，可以调用`object`、`data`、`model`、`usage`来查看`response`的`embedding`类型、`embedding`、`embedding model`及使用情况。

```python
print(f'response类型为：{type(response)}')
print(f'embedding类型为：{response.object}')
print(f'生成embedding的model为：{response.model}')
print(f'生成的embedding长度为：{len(response.data[0].embedding)}')
print(f'embedding（前10）为: {response.data[0].embedding[:10]}')
```

<details class="lake-collapse"><summary id="uba8328f6"><span class="ne-text">output：</span></summary><pre data-language="python" id="SXP16" class="ne-codeblock language-python"><code>response类型为：&lt;class 'zhipuai.types.embeddings.EmbeddingsResponded'&gt;
embedding类型为：list
生成embedding的model为：embedding-2
生成的embedding长度为：1024
embedding（前10）为: [0.017892399802803993, 0.0644201710820198, -0.009342825971543789, 0.02707476168870926, 0.004067837726324797, -0.05597858875989914, -0.04223804175853729, -0.03003198653459549, -0.016357755288481712, 0.06777040660381317]</code></pre></details>
