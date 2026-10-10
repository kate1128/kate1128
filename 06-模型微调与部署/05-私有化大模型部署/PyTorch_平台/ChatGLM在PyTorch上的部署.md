# 大模型使用的三个阶段
大模型的使用必将包含三个阶段：

1. 直接使用
2. 使用 API 定制自己的应用
3. 离线部署+微调，实现私有数据模型化

分阶段讨论大模型的离线部署+微调，今天先从0开始离线部署大模型。

# 环境安装和配置
本文以清华大学开源的 ChatGLM-6B 语言模型为例。ChatGLM-6B 是一个开源的、支持中英双语的对话语言模型，基于 General Language Model (GLM) 架构，具有 62 亿参数。结合模型量化技术，用户可以在消费级的显卡上进行本地部署。

实验使用的环境如下：

+ Windows11
+ Intel 13700KF
+ 32G内存
+ RTX 3090 24G显存

ChatGLM-6B 可在最小 6GB 显存运行。如果没有合适的显卡或者想体验完整版，可以购买云服务商的 A100 GPU 服务器试用。以阿里云为例，最便宜的每小时 38 元左右。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925736827-8f5b9f72-0100-4182-90b5-bdbb3c65a29b.png)****

# 安装 Python
Python 官网下载并安装 Python，记得选上“Add python.exe to PATH”。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925736655-851c0757-fdbb-4cf3-8710-eda111a0154b.png)

# 安装 CUDA****
由于 PyTorch 最新只能支持 11.8 的显卡驱动，不能安装最新版 CUDA。

在 Nvidia 官网 下载 11.8 的 CUDA Toolkit Archive。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925736729-ab73b579-18c3-47d4-9135-f8b557760d38.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925736661-1211ab7c-dd74-4107-bc34-7fb6175324f5.png)**
**

# 安装 PyTorch
在 PyTorch 官网 执行对应版本的安装命令。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925736803-bccc6a53-7ddf-42f4-9329-2f60d82f9651.png)

```python
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925737510-0a9c51e1-480f-446d-b34f-07221df2ca6f.png)****

# 安装 git****
从 git 官网 下载 git

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925737584-d76eab4d-cfe7-43a0-bfe9-d3b72dd3bb7f.png)****

# 部署代码
## Clone 代码
使用 git clone 对应的代码：

```powershell
git clone https://github.com/THUDM/ChatGLM-6B.git
```

## 安装依赖
```powershell
cd ChatGLM-6B

pip install -r requirements.txt
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925737579-8a76eacd-11f1-47ac-807f-249fbb49d4d7.png)

### **
**
## 下载模型
代码在执行时默认自动下载模型。如果没有使用魔法，你需要手动下载模型。在 清华大学云盘 下载模型，假设下载到 D:\chatglm-6b-models

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925737603-45a8b993-5f8d-441f-a7bc-1f38d8a47d7f.png)

## 运行代码
### 启动 Python
```powershell
python
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925737694-c867c6e1-484c-4144-9325-7d01e337b019.png)依次输入下列代码：

```powershell
from transformers import AutoTokenizer, AutoModel
tokenizer = AutoTokenizer.from_pretrained(r"D:\chatglm-6b-models", trust_remote_code=True)
model = AutoModel.from_pretrained(r"D:\chatglm-6b-models", trust_remote_code=True).half().cuda()
model = model.eval()
response, history = model.chat(tokenizer, "你好", history=[])
print(response)
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925738222-32abab59-7386-454e-a008-e24fe3cbeb0c.png)

ChatGLM-6B 返回了“你好👋！我是人工智能助手 ChatGLM-6B，很高兴见到你，欢迎问我任何问题。”。至此，大语言模型的离线部署就实现了。我们可以发挥我们的聪明才智，让它给我们工作了。

### 长文本生成
让 ChatGLM-6B 为我们生成一篇文章。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925738302-ee1d6fda-9c8e-401d-99b3-49f130b04c5b.png)

经过大约10秒钟后，文章生成。看结果还是很不错的。
在任务管理器里查看显卡运行情况，使用了约 13G 的显存。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736925738266-e44863ba-2979-4894-be11-c0c0c61d4ca3.png)

# 参考
[https://maimai.cn/article/detail?fid=1790540266&efid=Xe65Yz8a0dqumSp3ASwa7A](https://maimai.cn/article/detail?fid=1790540266&efid=Xe65Yz8a0dqumSp3ASwa7A)
