# <font style="color:rgb(31, 35, 40);">环境准备</font>
<font style="color:rgb(31, 35, 40);">在</font>[autodl](https://www.autodl.com/)<font style="color:rgb(31, 35, 40);">平台租机器，pip换源和安装依赖包：</font>

```python
# 升级pip
python -m pip install --upgrade pip
# 更换 pypi 源加速库的安装
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

pip install fastapi==0.104.1
pip install uvicorn==0.24.0.post1
pip install requests==2.25.1
pip install modelscope==1.9.5
pip install transformers==4.37.2
pip install streamlit==1.24.0
pip install sentencepiece==0.1.99
pip install accelerate==0.24.1
```

# <font style="color:rgb(31, 35, 40);">模型下载</font>
<font style="color:rgb(31, 35, 40);">使用</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">modelscope</font>`<font style="color:rgb(31, 35, 40);"> </font><font style="color:rgb(31, 35, 40);">中的</font>`<font style="color:rgb(31, 35, 40);">snapshot_download</font>`<font style="color:rgb(31, 35, 40);">函数下载模型，第一个参数为模型名称，参数</font>`<font style="color:rgb(31, 35, 40);">cache_dir</font>`<font style="color:rgb(31, 35, 40);">为模型的下载路径。</font>

<font style="color:rgb(31, 35, 40);">在</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">/root/autodl-tmp</font>`<font style="color:rgb(31, 35, 40);"> </font><font style="color:rgb(31, 35, 40);">路径下新建</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">download.py</font>`<font style="color:rgb(31, 35, 40);"> </font><font style="color:rgb(31, 35, 40);">文件并在其中输入以下内容，粘贴代码后记得保存文件，如下图所示。并运行</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">python /root/autodl-tmp/download.py</font>`<font style="color:rgb(31, 35, 40);">执行下载，模型大小为 14 GB，下载模型大概需要 10~20 分钟</font>

```python
import torch
from modelscope import snapshot_download, AutoModel, AutoTokenizer
import os
model_dir = snapshot_download('ZhipuAI/chatglm3-6b', cache_dir='/root/autodl-tmp', revision='master')
```

# <font style="color:rgb(31, 35, 40);">代码准备</font>
<font style="color:rgb(31, 35, 40);">在</font>`<font style="color:rgb(31, 35, 40);">/root/autodl-tmp</font>`<font style="color:rgb(31, 35, 40);">路径下新建</font>`<font style="color:rgb(31, 35, 40);">api.py</font>`<font style="color:rgb(31, 35, 40);">文件并在其中输入以下内容：</font>

```python
from fastapi import FastAPI, Request
from transformers import AutoTokenizer, AutoModelForCausalLM
import uvicorn
import json
import datetime
import torch

# 设置设备参数
DEVICE = "cuda"  # 使用CUDA
DEVICE_ID = "0"  # CUDA设备ID，如果未设置则为空
CUDA_DEVICE = f"{DEVICE}:{DEVICE_ID}" if DEVICE_ID else DEVICE  # 组合CUDA设备信息

# 清理GPU内存函数
def torch_gc():
    if torch.cuda.is_available():  # 检查是否可用CUDA
        with torch.cuda.device(CUDA_DEVICE):  # 指定CUDA设备
            torch.cuda.empty_cache()  # 清空CUDA缓存
            torch.cuda.ipc_collect()  # 收集CUDA内存碎片

# 创建FastAPI应用
app = FastAPI()

# 处理POST请求的端点
@app.post("/")
async def create_item(request: Request):
    global model, tokenizer  # 声明全局变量以便在函数内部使用模型和分词器
    json_post_raw = await request.json()  # 获取POST请求的JSON数据
    json_post = json.dumps(json_post_raw)  # 将JSON数据转换为字符串
    json_post_list = json.loads(json_post)  # 将字符串转换为Python对象
    prompt = json_post_list.get('prompt')  # 获取请求中的提示
    history = json_post_list.get('history')  # 获取请求中的历史记录
    max_length = json_post_list.get('max_length')  # 获取请求中的最大长度
    top_p = json_post_list.get('top_p')  # 获取请求中的top_p参数
    temperature = json_post_list.get('temperature')  # 获取请求中的温度参数
    
    # 调用模型进行对话生成
    response, history = model.chat(
        tokenizer,
        prompt,
        history=history,
        max_length=max_length if max_length else 2048,  # 如果未提供最大长度，默认使用2048
        top_p=top_p if top_p else 0.7,  # 如果未提供top_p参数，默认使用0.7
        temperature=temperature if temperature else 0.95  # 如果未提供温度参数，默认使用0.95
    )
    now = datetime.datetime.now()  # 获取当前时间
    time = now.strftime("%Y-%m-%d %H:%M:%S")  # 格式化时间为字符串
    
    # 构建响应JSON
    answer = {
        "response": response,
        "history": history,
        "status": 200,
        "time": time
    }
    
    # 构建日志信息
    log = "[" + time + "] " + '", prompt:"' + prompt + '", response:"' + repr(response) + '"'
    print(log)  # 打印日志
    torch_gc()  # 执行GPU内存清理
    return answer  # 返回响应

# 主函数入口
if __name__ == '__main__':
    # 加载预训练的分词器和模型
    tokenizer = AutoTokenizer.from_pretrained("/root/autodl-tmp/ZhipuAI/chatglm3-6b", trust_remote_code=True)
    
    model = AutoModelForCausalLM.from_pretrained("/root/autodl-tmp/ZhipuAI/chatglm3-6b", trust_remote_code=True).to(torch.bfloat16).cuda()
    
    model.eval()  # 设置模型为评估模式
    
    # 启动FastAPI应用
    # 用6006端口可以将autodl的端口映射到本地，从而在本地使用api
    uvicorn.run(app, host='0.0.0.0', port=6006, workers=1)  # 在指定端口和主机上启动应用
```

# <font style="color:rgb(31, 35, 40);">Api 部署</font>
<font style="color:rgb(31, 35, 40);">在终端输入以下命令启动</font>`<font style="color:rgb(31, 35, 40);">api</font>`<font style="color:rgb(31, 35, 40);">服务</font>

```python
cd /root/autodl-tmp
python api.py
```

<font style="color:rgb(31, 35, 40);">默认部署在 6006 端口，通过 POST 方法进行调用，可以使用</font>`<font style="color:rgb(31, 35, 40);">curl</font>`<font style="color:rgb(31, 35, 40);">调用，如下所示：</font>

```python
curl -X POST "http://127.0.0.1:6006" \
-H 'Content-Type: application/json' \
-d '{"prompt": "你好", "history": []}'
```

<font style="color:rgb(31, 35, 40);">调用示例结果如下图所示</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735717701539-9a9f5a94-3a20-4b66-88fc-e51f1e6b4d87.png)

<font style="color:rgb(31, 35, 40);">也可以使用</font>`<font style="color:rgb(31, 35, 40);">python</font>`<font style="color:rgb(31, 35, 40);">中的</font>`<font style="color:rgb(31, 35, 40);">requests</font>`<font style="color:rgb(31, 35, 40);">库进行调用，如下所示：</font>

```python
import requests
import json

def get_completion(prompt):
    headers = {'Content-Type': 'application/json'}
    data = {"prompt": prompt, "history": []}
    response = requests.post(url='http://127.0.0.1:6006', headers=headers, data=json.dumps(data))
    return response.json()['response']

if __name__ == '__main__':
    print(get_completion('你好'))
```

<font style="color:rgb(31, 35, 40);">得到的返回值如下所示：</font>

```python
{
    "response": "你好👋！我是人工智能助手 ChatGLM3-6B，很高兴见到你，欢迎问我任何问题。",
    "history": [{"role": "user", "content": "你好"}, {"role": "assistant", "metadata": "", "content": "你好👋！我是人工智能助手 ChatGLM3-6B，很高兴见到你，欢迎问我任何问题。"}],
    "status": 200,
    "time": "2023-11-28 11:16:06"
}
```

<font style="color:rgb(31, 35, 40);">调用结果如下图所示</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735717724709-07464527-572a-49c2-a87f-2b0218f6497a.png)

