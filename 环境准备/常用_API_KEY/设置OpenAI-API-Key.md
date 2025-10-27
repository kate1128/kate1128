登陆 [OpenAI 账户](https://platform.openai.com/account/api-keys) 获取API Key，然后将其设置为环境变量。

### 全局环境变量
+ 参考[知乎文章](https://zhuanlan.zhihu.com/p/627665725)。

### 本地/项目环境变量
+ 想要设置为本地/项目环境变量，在本文件目录下创建`.env`文件, 打开文件输入以下内容。

```python
# 替换"your_api_key"为你自己的 API Key
OPENAI_API_KEY="your_api_key" 
```

操作：

```python
# 下载需要的包python-dotenv和openai
# 如果你需要查看安装过程日志，可删除 -q
!pip install -q python-dotenv
!pip install -q openai
```

```python
import os
import openai
from dotenv import load_dotenv, find_dotenv
```

```python
"""
1.find_dotenv()寻找并定位.env文件的路径
2.load_dotenv()读取该.env文件，并将其中的环境变量加载到当前的运行环境中
3.如果你设置的是全局的环境变量，这行代码则没有任何作用
"""
_ = load_dotenv(find_dotenv()) 
openai.api_key = os.environ['OPENAI_API_KEY']
```

