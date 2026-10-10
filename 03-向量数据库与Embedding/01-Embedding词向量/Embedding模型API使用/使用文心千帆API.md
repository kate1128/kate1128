`Embedding-V1`是基于百度文心大模型技术的文本表示模型，`Access token`为调用接口的凭证，使用`Embedding-V1`时应先凭`API Key、Secret Key`获取`Access token`，再通过`Access token`调用接口来`embedding text`。同时千帆大模型平台还支持`bge-large-zh`等`embedding model`。

```python
import os
import requests
import json

def wenxin_embedding(text: str):
    # 获取环境变量 wenxin_api_key、wenxin_secret_key
    api_key = os.environ['QIANFAN_AK']
    secret_key = os.environ['QIANFAN_SK']

    # 使用API Key、Secret Key向https://aip.baidubce.com/oauth/2.0/token 获取Access token
    url = "https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id={0}&client_secret={1}".format(api_key, secret_key)
    payload = json.dumps("")
    headers = {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
    }
    response = requests.request("POST", url, headers=headers, data=payload)
    
    # 通过获取的Access token 来embedding text
    url = "https://aip.baidubce.com/rpc/2.0/ai_custom/v1/wenxinworkshop/embeddings/embedding-v1?access_token=" + str(response.json().get("access_token"))
    input = []
    input.append(text)
    payload = json.dumps({
        "input": input
    })
    headers = {
        'Content-Type': 'application/json'
    }

    response = requests.request("POST", url, headers=headers, data=payload)

    return json.loads(response.text)
# text应为List(string)
text = "要生成 embedding 的输入文本，字符串形式。"
response = wenxin_embedding(text=text)
```

`Embedding-V1`每次`embedding`除了有单独的id外，还有时间戳记录`embedding`的时间。

```python
print('本次embedding id为：{}'.format(response['id']))
print('本次embedding产生时间戳为：{}'.format(response['created']))
```

<details class="lake-collapse"><summary id="u622537fe"><span class="ne-text">output：</span></summary><pre data-language="python" id="KT1uI" class="ne-codeblock language-python"><code>本次embedding id为：as-hvbgfuk29u
本次embedding产生时间戳为：1711435238</code></pre></details>
同样的也可以从`response`中获取`embedding`的类型和`embedding`。

```python
print('返回的embedding类型为:{}'.format(response['object']))
print('embedding长度为：{}'.format(len(response['data'][0]['embedding'])))
print('embedding（前10）为：{}'.format(response['data'][0]['embedding'][:10]))
```

<details class="lake-collapse"><summary id="uecb14136"><span class="ne-text">output：</span></summary><pre data-language="python" id="AXqij" class="ne-codeblock language-python"><code>返回的embedding类型为:embedding_list
embedding长度为：384
embedding（前10）为：[0.060567744076251984, 0.020958080887794495, 0.053234219551086426, 0.02243831567466259, -0.024505289271473885, -0.09820500761270523, 0.04375714063644409, -0.009092536754906178, -0.020122773945331573, 0.015808865427970886]</code></pre></details>


