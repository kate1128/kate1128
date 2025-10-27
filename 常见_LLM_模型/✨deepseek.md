# 参考文档
1. 接口文档：[https://api-docs.deepseek.com/zh-cn/](https://api-docs.deepseek.com/zh-cn/)
2. api key 申请：[https://platform.deepseek.com/usage](https://platform.deepseek.com/usage)
3. github 地址：[https://github.com/deepseek-ai/DeepSeek-Coder](https://github.com/deepseek-ai/DeepSeek-Coder)
4. 使用 deepseek：[https://chat.deepseek.com/a/chat/s/3201323a-643e-47a3-9012-b7501d6db2c8](https://chat.deepseek.com/a/chat/s/3201323a-643e-47a3-9012-b7501d6db2c8)

# 使用
使用：[https://chat.deepseek.com/a/chat/s/3201323a-643e-47a3-9012-b7501d6db2c8](https://chat.deepseek.com/a/chat/s/3201323a-643e-47a3-9012-b7501d6db2c8)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737619880865-cce8a7b3-1fa1-4c03-96b4-59c3354bae13.png)

# API Key 申请
给送 10 块钱的 token

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737617491069-d42df588-4ef0-4eff-bd81-8e71a132fd97.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737617601126-329329a5-e240-4f1f-8e8f-29388eb65a45.png)

```python
# Please install OpenAI SDK first: `pip3 install openai`

from openai import OpenAI

client = OpenAI(api_key="<DeepSeek API Key>", base_url="https://api.deepseek.com")

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": "You are a helpful assistant"},
        {"role": "user", "content": "Hello"},
    ],
    stream=False
)

print(response.choices[0].message.content)
```

# 其他
参考这个：[DeepSeek图解10页PDF.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1738719596981-a1bd2b7d-4f11-4173-b4fc-f4af69e554e4.pdf)

