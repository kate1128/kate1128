01月02日[@凯佳er](undefined/qiaokate)

1. 能登录 openAI 官网：[https://platform.openai.com/](https://platform.openai.com/)
    1. 找一个 梯子 软件：[https://iuiu.io/register/rdwjjg](https://iuiu.io/register/rdwjjg)
    2. ~~有一个接收验证码的手机号，虚拟号网站：~~[~~https://sms-activate.guru/cn~~](https://sms-activate.guru/cn)~~（这个看起来不是必需，先划掉了）~~参考这个：[https://chatgptzhanghao.com/](https://chatgptzhanghao.com/)
    3. 有一张能付费的信用卡（[https://chatgptgogogo.com/bewildcard-upgrade-chatgptplus/](https://chatgptgogogo.com/bewildcard-upgrade-chatgptplus/)）

可以科学上网之后，直接用邮箱注册

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736909837064-32ed9223-544e-4400-a3d1-d817934b6acf.png)

但是这个充值要信用卡

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736910053021-042db46a-d7fc-4a8c-a504-8fb7a3df74dc.png)

2. 或者找一个第三方的

[New API](https://xiaoai.plus/register?aff=zEdb)

```python
from openai import OpenAI
client = OpenAI(
    base_url='https://xiaoai.plus/v1',
    # sk-xxx替换为自己的key
    api_key='sk-xxx'
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

