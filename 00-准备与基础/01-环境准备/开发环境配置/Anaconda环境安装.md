1. 安装地址：[https://www.anaconda.com/download](https://www.anaconda.com/download)
2. 安装环境

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736931647590-aed659e6-99bb-456c-9448-06c1f384b1e7.png)

3. new 一个环境

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736931576856-c48cac22-afdd-4ce2-9e14-508d5ee79296.png)

4. 安装包

```python
pip install --upgrade openai httpx[socks]
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736931731374-de0b3d74-d824-4291-b89f-83765e1d0090.png)

5. 随便找个代码测一下

```python
from openai import OpenAI
client = OpenAI(
    base_url='https://xiaoai.plus/v1',
    # sk-xxx替换为自己的key
    api_key='sk-xxxx'
)
completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ]
)
print(completion.choices[0].message)
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736931758857-27aedc95-0d17-42d4-9a7f-29c2382e0fe3.png)

