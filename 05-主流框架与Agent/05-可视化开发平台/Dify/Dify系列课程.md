[Dify教程.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1752986567677-8ef33f5c-da2a-4e7f-93c5-30234012a1d9.pdf)

[Dify教程0320.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1752986574015-5ae98a43-4a09-46ae-8bc1-63d8732aec66.pdf)

[Dify&Coze教程.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1752986688888-ba805060-0206-4f9b-9a45-22139c69b91f.pdf)

# Dify的介绍
> [https://dify.ai/](https://dify.ai/)
>

	Dify 是一款创新的智能生活助手应用，旨在为您提供便捷、高效的服务。通过人工智能技术，Dify 可以实现语音助手、智能家居控制、日程管理等功能，助您轻松应对生活琐事，享受智慧生活。简约的界面设计，让操作更加便捷；丰富的应用场景，满足您多样化的需求。Dify，让生活更简单！

# Dify的安装方式
## 在线体验
	速度比较慢。不推荐

## 本地部署
### Docker安装
1. 安装Docker环境

```bash
bash <(curl -sSl https://cdn.jsdelivr.net/gh/SuperManito/LunuxMirrors@main/DockerInstallation.sh)
```

2. 安装Docker Compose

```bash
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose
```

3. 执行查看Docker-compose版本

```bash
docker-compose --version
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596203-f5608581-ef54-4533-bb2f-bed2fdc18f0e.png)



说明安装成功了

docker-compse拉取镜像很慢

```bash
sudo tee /etc/docker/daemon.json <<-'EOF'
{
     "registry-mirrors": [
         "https://do.nark.eu.org",
         "https://dc.j8.work",
         "https://docker.m.daocloud.io",
         "https://dockerproxy.com",
         "https://docker.mirrors.ustc.edu.cn",
         "https://docker.nju.edu.cn"
     ]
}
EOF
```

执行上面的代码

```bash
sudo systemctl daemon-reload    # 重新加载 systemd 的配置文件
systemctl restart docker        # 重启docker
```

然后去GitHub上拉取dify的代码。解压后进入到docker目录中

```bash
docker-compose up -d
```

执行即可

### DockerDeskTop
	在Windows环境下我们可以通过DockerDesktop 来安装。直接去官网下载对应的版本即可。同样的我们需要拉取dify的GitHub的代码。然后进入到Docker目录，同样的执行这个代码

```bash
docker-compose up 
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596281-d0bed8a7-a3d6-4342-96b7-66084994979f.png)

然后在地址栏中输入 [http://localhost/install](http://localhost/install) 就可以访问了

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596341-3e5a2fbb-39ee-46d1-94c8-d989ded352d1.png)

我们先设置管理员的相关信息。设置后再登录

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596391-0b021e09-9d96-4a92-ab3a-76381cbafb20.png)

# Ollama
[Ollama](https://ollama.com/)

	我们已经把Dify在本地部署了。然后我们可以通过Ollama在本地部署对应的大模型，比如 deepseek-r1:1.5b 这种小模型

Ollama 是一个让你能在本地运行大语言模型的工具，为用户在本地环境使用和交互大语言模型提供了便利，具有以下特点：

1. 多模型支持：Ollama 支持多种大语言模型，比如 Llama 2、Mistral 等。这意味着用户可以根据自己的需求和场景，选择不同的模型来完成各种任务，如文本生成、问答系统、对话交互等。
2. 易于安装和使用：它的安装过程相对简单，在 macOS、Linux 和 Windows 等主流操作系统上都能方便地部署。用户安装完成后，通过简洁的命令行界面就能与模型进行交互，降低了使用大语言模型的技术门槛。
3. 本地运行：Ollama 允许模型在本地设备上运行，无需依赖网络连接来访问云端服务。这不仅提高了数据的安全性和隐私性，还能减少因网络问题导致的延迟，实现更快速的响应。

搜索Ollama进入官网

[Download Ollama on macOS](https://ollama.com/download)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596439-4d3ec3ef-9f67-4d91-8e84-76129bdde295.png)

下载完成后直接双击安装即可

命令：ollama,出现下面内容，说明安装成功

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596510-a64b517f-0741-4de7-8ffc-f208fa7b87da.png)

启动Ollama服务

输入命令【ollama serve】，浏览器打开，显示running，说明启动成功

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596624-ddfc0bad-b2b1-466d-8640-39234ddb6295.png)

安装 deepseek-r1:1.5b模型

在[https://ollama.com/library/deepseek-r1:1.5b](https://ollama.com/library/deepseek-r1:1.5b) 搜索deepseek-R1,跳转到下面的页面，复制这个命令，在终端执行，下载模型

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596689-6890c675-b7aa-488b-8979-76207a341c72.png)

cmd中执行这个命令

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596822-59aefe6c-6915-4b46-90a3-a0633b09ab45.png)

# Dify关联Ollama
	Dify 是通过Docker部署的，而Ollama 是运行在本地电脑的，得让Dify能访问Ollama 的服务。

在Dify项目`-docker-`找到`.env`文件，在末尾加上下面的配置：（在 dify 的 docker-compose 同级目录）

```bash
# 启用自定义模型
CUSTOM_MODEL_ENABLED=true
# 指定 Olama 的 API地址（根据部署环境调整IP）
OLLAMA_API_BASE_URL=host.docker.internal:11434
```

然后在模型中配置

在Dify的主界面 [http://localhost/apps](http://localhost/apps) ，点击右上角用户名下的【设置】

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596871-b11c3569-fcd9-467a-b170-2800aef93b70.png)

在设置页面--Ollama--添加模型，如下：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596930-71a5c0d0-4976-4f3a-8b77-3cdf5ea9a667.png)

添加成功后的

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984596984-e40ddc3d-e5c6-414c-82c0-f36e4aae94ae.png)

模型添加完成以后，刷新页面，进行系统模型设置。步骤：输入“[http://localhost/install](http://localhost/install)”进入Dify主页，用户名--设置--模型供应商，点击右侧【系统模型设置】，如下：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597037-4739c0b9-729c-452c-8ef3-89ed9040c3e2.png)

这样就关联成功了！！！

# Dify应用讲解
## 创建空白应用
我们通过Dify来创建我们的第一个简单案例，智能聊天机器人

进入Dify 主界面，点击【创建空白应用】，如下图：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597092-49fbbf03-673d-4c0c-8c0f-f3edabd3b9bc.png)

选择【聊天助手】，输入自定义应用名称和描述，点击【创建】

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597143-45797ed7-2340-486f-a7f7-87177c18e196.png)

右上角选择合适的模型，进行相关的参数配置

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597203-be8f96c5-6ca7-44ff-888f-dcae8ca8e720.png)

输入有相关的回复了。此时说明Dify 与本地部署的DeepSeek大模型已经连通了。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597265-bba8e503-b477-4a42-b97f-a5b165d360d7.png)

上面的机器人有个不足之处就是无法回答模型训练后的内容和专业垂直领域的内容，这时我们可以借助本地知识库来解决专业领域的问题。

## 创建本地知识库
### 向量模型
Embedding模型是一种将数据转换为向量表示的技术，核心思想是通过学习数据的内在结构和语义信息，将其映射到一个低维向量空间中，使得相似的数据点在向量空间中的位置相近，从而通过计算向量之间的相似度来衡量数据之间的相似性。

 	Embedding模型可以将单词、句子或图像等数据转换为低维向量，使得计算机能够更好地理解和处理这些数据。在NLP领域，Embedding模型可以将单词、句子或文档转换为向量，用于文本分类、情感分析。机器翻译等任务。在计算机视觉中，Embedding模型可以用于图像识别和检索等任务。

### 添加Embedding模型
点击右上角用户名--设置--模型供应商--右上角【添加模型】，填写相关配置信息如下：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597367-66b2e8be-6c82-4005-9951-5791f3ef1e9e.png)

添加成功后的效果

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597417-aabcba12-c9ef-4772-b359-fde613d10866.png)

### 创建知识库
在Dify主界面，点击上方的【知识库】，点击【创建知识库】

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597473-90d4844f-735b-47c8-aa35-d9ad6238c1bf.png)

导入已有文本，上传资料，点击【下一步】

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597526-b923b6e2-e1af-4984-b595-e509bacd7b92.png)

Embedding模型默认是前面配置的模型，参数信息配置完，点击保存即可

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597589-55313ff8-ca79-46b2-b159-8f0c148dc116.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597643-737226e9-224c-48ee-ab1b-f192f06ef24c.png)

此时系统会自动对上传的文档进行解析和向量化处理，需要耐心等待几分钟。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597697-c74555ce-2d71-4c67-bff9-84d481696d2d.png)

创建成功以后，如下图，可以点击【前往文档】，查看分段信息，如下图：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597757-e98741ad-2d23-4900-850a-dfdd5d420091.png)

点击具体的文档可以看到具体的分割信息

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597830-29f94a49-87c5-4faf-a881-ca3e916c3311.png)



## 知识库应用
### 添加知识库
	在Dify主界面，回到刚才的应用聊天页面，工作室--智能聊天机器人--添加知识库，如下图：  
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597885-033a773d-d2db-49b3-babc-9c921803ca65.png)

选择前面上面的知识库作为对话的上下文，保存当前应用设置，就可以进行测试了

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597934-aec2fb43-ff6c-42e2-ab63-94caa2c27b9d.png)

### 测试
此时输入问题，就可以看到相关的回复了。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984597987-69e6dda2-1904-4e31-abbe-a242e4e0ab30.png)





## AI图片生成工具
	随着图像生成技术的兴起，涌现了许多优秀的图像生成产品，比如 Dall-e、Flux、Stable Diffusion 等，我们借助Stable Diffusion来在dify中构建一个智能生成图片的Agent。

### 首先获取Stable Diffusion
[Stability AI - Developer Platform](https://platform.stability.ai/account/keys)

去官网获取授权key。如果没注册需要先注册下

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598043-04bf63ca-431a-45a4-8824-aa30fdbf63cd.png)

### 下载 Stable 工具
 然后我们需要进入dify的工具市场下载安装 Stable 插件。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598102-eb540285-a07f-4bd9-bb2a-c898cc417588.png)



### 创建Agent
	然后我们就可以创建一个空白的Agent。输入对应的提示词

```bash
根据用户的提示，使用工具 stability_text2image 绘画指定内容
```

然后选择对应的工具并添加授权码

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598153-9979d60d-7e40-4640-854d-76bebc7f723b.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598218-caf68048-a60f-4d99-b76b-85f3789ccc13.png)

然后我们就可以测试效果了

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598266-101197a0-7eb1-4dfc-ba53-03da0c03b694.png)

注意这个是一个付费的工具。提供的有一个免费的，后面需要付费购买了：[https://platform.stability.ai/account/credits](https://platform.stability.ai/account/credits)



## 旅游助手
进入[SerpAPI - API Key](https://serpapi.com/manage-api-key)，如果你尚未注册，会被跳转至进入注册页。

SerpAPI提供一个月100次的免费调用次数，这足够我们完成本次实验了。如果你需要更多的额度，可以增加余额，或者使用其他的开源方案。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598327-ac539131-d2e1-47a9-92b0-d98b2021dc34.png)



## SQL执行器
	我们可以通过工作流来创建一个SQL语句的执行器，也就是我们可以输入相关的SQL语句然后通过工作流来连接数据库执行对应的SQL代码，具体的设计如下：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598391-d4c5a51e-4095-48f0-97af-e5afda7e769d.png)

	这里的核心是代码执行模块。这块我们是调用了我们自己创建的接口来执行数据库的操作，所以我们需要先创建这么一个接口，接口我们通过Flask这个轻量的web框架来实现。需要先安装Flask的依赖库

```bash
pip install flask
```

然后创建接口代码

```python
from flask import Flask, request, jsonify
import pymysql

app = Flask(__name__)
def execute_sql(sql,connection_info):
    """
    执行传入的 SQL 语句，并返回查询结果。

    参数:
        sql: 要执行的 SQL 语句（字符串）。
        connection_info: 一个字典，包含数据库连接所需的信息：
            - host: 数据库地址（如 "localhost"）
            - user: 数据库用户名
            - password: 数据库密码
            - database: 数据库名称
            - port: 数据库端口（可选，默认为 3306）
            - charset: 字符编码（可选，默认为 "utf8mb4"）

    返回:
        如果执行的是 SELECT 查询，则返回查询结果的列表；
        如果执行的是 INSERT/UPDATE/DELETE 等非查询语句，则提交事务并返回受影响的行数。
        如果执行过程中出错，则返回 None。
    """
    connection = None
    try:
        # 从 connection_info 中获取各项参数，设置默认值
        host = connection_info.get("host", "localhost")
        user =  connection_info.get("user")
        password = connection_info.get("password")
        database = connection_info.get("database")
        port = connection_info.get("port", 3306)
        charset = connection_info.get("charset", "utf8mb4")

        # 建立数据库连接
        connection = pymysql.connect(
            host=host,
            user=user,
            password=password,
            database=database,
            port=port,
            charset=charset,
            cursorclass=pymysql.cursors.Cursor  # 可改为 DictCursor 返回字典格式结果
        )

        with connection.cursor() as cursor:
            cursor.execute(sql)
            # 判断是否为 SELECT 查询语句
            if sql.strip().lower().startswith("select"):
                result = cursor.fetchall()
            else:
                connection.commit()  # 非查询语句需要提交事务
                result = cursor.rowcount  # 返回受影响的行数

        return result

    except Exception as e:
        print("执行 SQL 语句时出错：", e)
        return None

    finally:
        if connection:
            connection.close()


@app.route('/execute_sql', methods=['POST'])
def execute_sql_api():
    """
    接口示例：通过 POST 请求传入 SQL 语句和连接信息，返回执行结果。
    请求示例 (JSON):
    {
        "sql": "SELECT * FROM your_table;",
        "connection_info": {
            "host": "localhost",
            "user": "your_username",
            "password": "your_password",
            "database": "your_database"
        }
    }
    """
    data = request.get_json()
    if not data:
        return jsonify({"error": "无效的请求数据"}), 400

    sql = data.get("sql")
    connection_info = data.get("connection_info")
    if not sql or not connection_info:
        return jsonify({"error": "缺少sql语句或数据库连接信息"}), 400

    result = execute_sql(sql, connection_info)
    return jsonify({"result": result})


if __name__ == '__main__':
    # 开发环境下可以设置 debug=True，默认在本地5000端口启动服务
    app.run(debug=True)
```

这个接口需要接收一个sql语句和一个包含数据库连接信息的json对象，我们可以编写对应的测试代码来看看

```python
import json
import requests


def call_execute_sql_api(sql, connection_info):
    """
    通过 requests 调用执行 SQL 的接口服务

    参数:
        sql: 要执行的 SQL 语句字符串
        connection_info: 数据库连接信息字典，例如：
            {
                "host": "localhost",
                "user": "your_username",
                "password": "your_password",
                "database": "your_database",
                "port": 3306  # 可选
            }

    返回:
        接口返回的结果数据（字典格式），如果请求失败则返回 None
    """
    url = "http://127.0.0.1:5000/execute_sql"
    # 构造请求体
    payload = {
        "sql": sql,
        "connection_info": connection_info
    }

    headers = {
        "Content-Type": "application/json"
    }

    try:
        response = requests.post(url, json=payload, headers=headers)
        if response.status_code == 200:
            try:
                return {"result":str(response.json()["result"])}
            except Exception as e:
                return {"result": f"解析响应 JSON 失败: {str(e)}"}
        else:
            return {"result": f"请求失败，状态码: {response.status_code}"}
    except Exception as e:
        return {"result": str(e)}


# 示例调用
if __name__ == "__main__":
    sql_query = "select * from candidates where id = 1"  # 替换为你的实际 SQL 语句
    conn_info = {
        "host": "localhost",
        "user": "root",
        "password": "123456",
        "database": "ibms",
        "port": 3306
    }

    result = call_execute_sql_api(sql_query, conn_info)
    print("接口返回结果：", result)

```

执行后可以看到对应的结果

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598448-141fc5e8-aa33-4a64-863a-3af8596f9f91.png)

然后可以在工作流中来设置我们的代码

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598498-ae4e6d08-162c-4582-8504-4c8b19c288cf.png)

代码的内容

```python
import json
import requests

def main(sql: str) -> dict:
    url = "http://host.docker.internal:5000/execute_sql"
    connection_info = {
        "host": "localhost",
        "user": "root",
        "password": "123456",
        "database": "ibms"
    }
    # 构造请求体
    payload = {
        "sql": sql,
        "connection_info": connection_info
    }

    headers = {
        "Content-Type": "application/json"
    }

    try:
        response = requests.post(url, json=payload, headers=headers)
        if response.status_code == 200:
            try:
                return {"result":str(response.json()["result"])}
            except Exception as e:
                return {"result": f"解析响应 JSON 失败: {str(e)}"}
        else:
            return {"result": f"请求失败，状态码: {response.status_code}"}
    except Exception as e:
        return {"result": str(e)}

```

注意上面的url中我们需要写

```bash
http://host.docker.internal:5000/execute_sql
```

不然执行的时候会出现503的错误。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598557-d92c0a0b-ccad-49c7-9ad5-7d71d87e74ba.png)

如果调用接口的组件是 urllib3 的话有可能出现上面的问题，这个原因可能是版本兼容的问题，这里推进用的是requests组件



## 科研论文翻译
	我可以在工作流案例中结合聊天大模型来实现翻译工具的功能，具体的设计如下

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598621-2e4e4bd5-0faf-4cd9-836f-d428fd784fdf.png)

在开始节点中接收一个输入信息`content`

然后在LLM模型中我们需要配置一个CHAT模型，这里选择了DeepSeek-R1 64K的聊天模型，注意需要在这里设置下对应的提示词

```bash
现在我要写一个将中文翻译成英文科研论文的GPT，请参照以下Prompt制作，注意都用英文生成：

## 角色
你是一位科研论文审稿员，擅长写作高质量的英文科研论文。请你帮我准确且学术性地将以下中文翻译成英文，风格与英文科研论文保持一致。

## 规则：
- 输入格式为 Markdown 格式，输出格式也必须保留原始 Markdown 格式
- 以下是常见的相关术语词汇对应表（中文 -> English）：
* 零样本 -> Zero-shot
* 少样本 -> Few-shot

## 策略：

分三步进行翻译工作，并打印每步的结果：
1. 根据中文内容直译成英文，保持原有格式，不要遗漏任何信息
2. 根据第一步直译的结果，指出其中存在的具体问题，要准确描述，不宜笼统的表示，也不需要增加原文不存在的内容或格式，包括不仅限于：
- 不符合英文表达习惯，明确指出不符合的地方
- 语句不通顺，指出位置，不需要给出修改意见，意译时修复
- 晦涩难懂，模棱两可，不易理解，可以尝试给出解释
3. 根据第一步直译的结果和第二步指出的问题，重新进行意译，保证内容的原意的基础上，使其更易于理解，更符合英文科研论文的表达习惯，同时保持原有的格式不变

## 格式
返回格式如下，"{xxx}"表示占位符：

### 直译
{直译结果}

***

###问题
{直译的具体问题列表}

***

###意译
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598690-70eabad0-de1f-4cd0-8a55-5dd1272567c5.png)

在结束节点中输出结果即可。

数据中的中文课研论文案例

```bash
# 农业生产中作物栽培技术的创新与应用
## 摘要
本文针对我国农业生产中作物栽培技术存在的问题，分析了技术创新的必要性，并探讨了新型栽培技术的应用效果。通过实验研究发现，采用新型栽培技术能够有效提高作物产量和品质，为我国农业生产提供有力支持。
## 一、引言
农业生产是我国国民经济的重要组成部分，作物栽培技术直接关系到粮食产量和农业可持续发展。近年来，我国农业生产取得了显著成果，但仍然存在一些问题，如资源利用率低、生态环境恶化等。因此，创新作物栽培技术，提高农业生产效益具有重要意义。
## 二、作物栽培技术创新的必要性
1. 提高资源利用率：我国农业生产过程中，水资源、化肥和农药的利用率较低，导致资源浪费和生态环境恶化。创新栽培技术，有助于提高资源利用率，降低农业生产成本。
2. 保障粮食安全：随着人口增长和耕地减少，提高单位面积产量成为保障粮食安全的关键。作物栽培技术创新有助于挖掘作物增产潜力，提高粮食产量。
3. 促进农业可持续发展：传统农业生产方式对生态环境造成一定程度的影响。创新栽培技术，有利于实现农业生产与生态环境的协调发展。
## 三、新型作物栽培技术的应用
1. 精准农业技术：通过无人机、卫星遥感等手段，实时监测作物生长状况，实现精准施肥、灌溉和病虫害防治。
2. 节水灌溉技术：采用滴灌、喷灌等节水灌溉方式，提高水资源利用率，降低农业生产成本。
3. 抗逆性品种选育：针对我国不同地区气候特点，选育抗逆性强的作物品种，提高作物产量和品质。
4. 生物有机肥应用：充分利用农业废弃物，发展生物有机肥，改善土壤结构，提高土壤肥力。
## 四、实验结果与分析
本研究以某地区小麦为例，对比分析了传统栽培技术与新型栽培技术下的产量和品质。实验结果表明，采用新型栽培技术的小麦产量较传统栽培技术提高了15.6%，品质也得到了显著提升。
## 五、结论
本文通过对农业生产中作物栽培技术的创新与应用研究，证实了新型栽培技术在提高作物产量和品质方面具有显著效果。未来，我国农业生产应继续加大科技创新力度，推动农业可持续发展。
（注：本文为示例性论文，仅供参考。）

```

效果图

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598755-d71c5b7b-761e-4001-bdfd-21ead5d6cbb9.png)



## SEO翻译
	我们在写文章的时候需要想一个满足SEO要求的标题，这样有可能被更多的人检索到，有时候我们可能需要把文章翻译为英文，这时标题同样比较重要，这时我们可以在dify中创建这样的一个工具来帮助我们实现这个功能。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598814-4595173b-afbe-4cf7-925c-23e2f557394e.png)

对应模型的提示词

```bash
This GPT will convert input titles or content into SEO-friendly English URL slugs. The slugs will clearly convey the original meaning while being concise and not exceeding 60 characters. If the input content is too long, the GPT will first condense it into an English phrase within 60 characters before generating the slug. If the title is too short, the GPT will prompt the user to input a longer title. Special characters in the input will be directly removed.
```

对应中文含义

```bash
这个GPT可以将输入的标题或内容转换为对SEO友好的英文URL片段。这些片段将清晰地传达原始含义，同时保持简洁，且不超过60个字符。如果输入内容过长，GPT将首先将其缩减为一个不超过60个字符的英文短语，然后生成片段。如果标题过短，GPT将提示用户输入更长的标题。输入中的特殊字符将被直接移除。
```





## 标题党
	有时候我们发表一些文章的时候，因为标题不够吸引人而造成没有什么关注度还是非常可惜的，这时我们可以借助AI来帮助我们生成有吸引力的标题，具体如下

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598865-0db1f9c1-bbe9-4be3-819c-80ac8c3db253.png)

核心是对应的提示词

```bash
你是一名资深的自媒体创作者也是一位爆款网文作家，你对不同领域的文章都有深入的了解和研究。你擅长创作吸睛、炸裂的标题创作。你有着对生活极为细致的观察，擅长在细节处触动人心。请根据用户提供的信息使用以下创作技巧进行标题创作，标题应具有吸引力，能够激发读者对文章主题的浓厚兴趣。
## 创作技巧
1.标题将感受、范围、结果、程度等夸张夸大描述，造成耸人听闻的效果。使用「震惊」、「惊爆」、「传疯」、「吓掉半条命」等，言过其实地表达情绪/状态/感受

案例1：《兰州竟然引起了全国的羡慕！西安疯了，天水哭了，嘉峪关伤了...》     ** 故意引用其他城市做夸张对比 ** 
案例2：《中国部署新型秘密武器，配备自杀敢死队，巴铁成功仿制吓坏印度》    ** 用“吓坏X国”的耸动表述故意诱导用户点击 ** 
案例3：《气垫一打开就直接涂？几乎所有女人都错了，怪不得总脱妆又卡粉！》  ** “几乎所有女人都”对女性群体做全部包含的范围夸张，诱导用户点击 **
案例4： 《全网无人能解释，看懂的全中国不超过2个！》  ** “全网”、“全中国”故意用整体范围概念，但“无人”、“不超过2个”又极端缩小范围形成夸张对比 **

2.**使用悬念式标题创作法。**标题擅用转折、隐藏关键性信息，营造悬念、制造故弄玄虚的效果,如「竟然是……」、「而是……」、「不过……」等话说一半，通过省略号代替关键信息，或使用「内幕」、「揭秘」、「真相」等代替关键信息
案例1: 《令人唏嘘，河南试卷掉包案最新进展，省教育局发出声明，称……》    ** “称……”话说一半，用省略号隐去关键信息点 **
案例2:《最新消息，全球最宜居国家排行榜，第一名果然是……你想去哪？》     ** 第一名是哪里可以很明确，故意不在标题中点明 **
案例3：《举国哀痛，我国的“航母杀手”刚有威慑力，竟然传来不幸的消息》    ** 标题中可表述清楚是什么消息，但故意用“竟然”强转折来制造危机感 **
案例4:《人狠话不多的史蒂夫奥斯丁、布洛克莱斯纳原来是这样的》 ** 原来是什么样的，可用一句话或形容词概括的内容故意不写明 **
案例5:《演技秒杀关晓彤，减肥20斤后撞脸娜扎，被嘲谎话连篇人设崩》  ** 缺少主语，且故意用既捧又杀的表述来诱导用户点击 **

3、使用强迫式标题风格创作标题。**标题采用挑衅恐吓、强迫修改后等方式，诱导用户阅读。标题使用「胆小慎入」、「不看后悔一辈子」、「别怪我没提醒你」等表述，挑衅恐吓用户点击

案例1：《不要在吃饭时看这个视频，要不然会让你后悔莫及》  ** “不要”怎样、“要不然”怎样，“让你后悔莫及”都是作者在故意挑衅用户观看 **
案例2: 《高考只剩30天，80%的答案都在这篇文章里，不看后悔一生》  ** “后悔一生”对用户形成恐吓感 **
案例3:《5个面试时常犯的错误 让你后悔一辈子》  ** “让你后悔一辈子”是典型的恐吓写法 **
案例4: 《疯狂抢地、地价飙升！房价大涨？烟台朋友千万要关注！！》   ** 命令式的“千万要关注”，搭配前半句的夸张表述，标题整体夸张问题严重 **
案例5: 《应届生找工作，一定要想知道这3件事，事关前途！》  ** “一定XXX”也是常见的“命令式”夸张写法 **

4.使用爆款关键词
在创作标题时，你会选用其中1-2个：
**「震惊」、「惊爆」、「传疯」、「不得不看」、「一定要看完」、「绝对要收藏」、「胆小慎入」、「不看后悔一辈子」、「别怪我没提醒你」、「竟然」、「竟是这样」、「结果却」、「没想到」、「竟然是……」、「而是……」、「不过……」、「内幕」、「揭秘」、「真相」、「重磅」、「要命」、「就在刚刚」、「全世界网友」、「所有男人都」、「某国人」、「99%」、**小白必看、教科书般, 划重点,建议收藏, 上天在提醒你、揭秘, 吹爆, 吐血整理,  万万没想到、你一定不知道、如何、最、咋、是什么、所有、10个、没有xx只有xx、秒懂、的故事、可怕、必看、长啥样、凭什么、不要、喂！、只需要、读懂、很可能、不是xx而是xx、你只是、当xx的时候、秘诀、为什么、在哪里、怎么办、史上、厉害、真正、是因为、方法、牛逼、你敢xx吗、你猜、马云、技巧、揭秘、爆照、必须看、传疯了、切记、围观、速看、感动、虐哭、居然、禁忌、疗法、只因、首次、伟大、猝死、出轨、小三、那些年、邂逅、秘密、意外、真相、背后究竟、绝招、第一个、否认、原来、采访、前兆、趋势、害死人、床上、你呢、赶快、不许、不要脸、千万、建议、年轻20岁、值得、和xx有关、罕见、至少、怒了、彻底、回应、强制、一触即发

## 约束条件
1.请使用以上 4 种标题创作技巧进行创作
2.标题创作运用悬念和刺激引发读者好奇心，容易让人引起联想
3.控制字数在 20 字以内
4.每次列出 5 个标题，多个标题请使用 ‘\n’ 进行分割，以便用户选择
5.收到内容后，直接创作对应的标题，无需额外的解释说明
```

还有用户部分

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598917-b1a6db03-dcf9-4730-9084-4d6d11fd617d.png)

然后对应的效果

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984598973-da8b89c2-5f0c-4b2d-8a5a-8d2c675ffed1.png)



## 知识库图像检索和展示
	我们可以利用工作流来实现知识库加大模型实现RAG的案例，同时在展示结果上可以把图片展示出来，这样效果会更加的直观些

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599032-6e6b3c5c-3a1a-4f29-94f2-bbefea6a607b.png)

这个案例的核心点是准备的检索数据和大模型的提示词

```bash

## 
亲子运动项目名称: 拔河小勇士
适合年龄：5~12岁
运动时间：10~15分钟
运动项目介绍：家长与孩子分别站在两端，通过拔河比赛锻炼孩子的力量和团队协作能力。
运动目标：增强孩子的上肢和腰部力量，培养团队精神。
运动规则：使用一条结实的绳子，中间划一条线，双方用力拉绳，将对方拉过中线即为胜利。
特别提示：注意孩子手部保护，避免摩擦受伤。
运动图片链接：[点击查看](https://img1.baidu.com/it/u=2633321814,4120475802&fm=253&fmt=auto&app=138&f=PNG?w=286&h=229)
## 
亲子运动项目名称: 跳绳接力
适合年龄：6~12岁
运动时间：15~20分钟
运动项目介绍：家长与孩子轮流跳绳，通过接力形式增加运动趣味性。
运动目标：提高孩子的耐力和协调性，增进亲子间的默契。
运动规则：设定一个跳绳次数目标，家长和孩子轮流跳，直到完成目标。
特别提示：选择适合孩子高度的跳绳，注意跳绳场地的平整。
运动图片链接：[点击查看](https://img2.baidu.com/it/u=1175898262,1254664272&fm=253&fmt=auto&app=120&f=JPEG?w=608&h=450)
## 
亲子运动项目名称: 滚雪球大赛
适合年龄：4~10岁
运动时间：10~15分钟
运动项目介绍：家长和孩子一起在雪地里滚雪球，比比谁滚得更快更大。
运动目标：锻炼孩子的动手能力和创造力，享受冬日乐趣。
运动规则：在规定时间内，看谁滚的雪球最大或者最快。
特别提示：注意保暖，避免孩子受寒。
运动图片链接：[点击查看](https://img1.baidu.com/it/u=2495318208,4233708303&fm=253&fmt=auto&app=138&f=JPEG?w=708&h=500)
## 
亲子运动项目名称: 家庭篮球赛
适合年龄：7~14岁
运动时间：20~30分钟
运动项目介绍：家长与孩子进行简易篮球比赛，提高孩子的篮球技能。
运动目标：培养孩子的球技和团队合作意识。
运动规则：简化篮球规则，进行半场3对3或1对1比赛。
特别提示：穿着合适的运动装备，注意安全。
运动图片链接：[点击查看](https://img0.baidu.com/it/u=3195478262,213097067&fm=253&fmt=auto&app=138&f=JPEG?w=700&h=467)
## 
亲子运动项目名称: 亲子瑜伽
适合年龄：5~12岁
运动时间：15~20分钟
运动项目介绍：家长与孩子一起练习瑜伽动作，增进亲子间的亲密关系。
运动目标：提高孩子的柔韧性和平衡能力，放松身心。
运动规则：跟随瑜伽教程，一起完成一系列瑜伽动作。
特别提示：穿着舒适，保持呼吸均匀。
运动图片链接：[点击查看](https://img1.baidu.com/it/u=2561021092,817698414&fm=253&fmt=auto&app=138&f=JPEG?w=667&h=500)
## 
亲子运动项目名称: 家庭接力跑
适合年龄：6~12岁
运动时间：15~20分钟
运动项目介绍：家庭成员分成两队，进行接力跑比赛。
运动目标：提高孩子的奔跑速度和团队协作能力。
运动规则：设定一个跑道，每队成员依次完成接力。
特别提示：确保跑道平整，避免跌倒。
运动图片链接：[点击查看](https://img1.baidu.com/it/u=975344947,3030921339&fm=253&fmt=auto&app=138&f=JPEG?w=720&h=480)
## 
亲子运动项目名称: 飞盘争夺战
适合年龄：6~12岁
运动时间：15~20分钟
运动项目介绍：家长与孩子进行飞盘传递和接住游戏。
运动目标：锻炼孩子的反应速度和手眼协调能力。
运动规则：在规定区域内，通过飞盘传递，争取让对方接不住飞盘。
特别提示：选择开阔的场地，避免飞盘伤人。
运动图片链接：[点击查看](https://img0.baidu.com/it/u=3788427895,2518031093&fm=253&fmt=auto&app=138&f=JPEG?w=750&h=500)
## 
亲子运动项目名称: 家庭足球赛
适合年龄：5~12岁
运动时间：20~30分钟
运动项目介绍：家长与孩子进行简易足球比赛，享受足球乐趣。
运动目标：提高孩子的足球技能和团队精神。
运动规则：简化足球规则，进行小场地比赛。
特别提示：穿着足球鞋，注意场地安全。
运动图片链接：[点击查看](https://example.com/images/soccer.jpg)
## 
亲子运动项目名称: 亲子攀岩
适合年龄：8~14岁
运动时间：30~45分钟
运动项目介绍：家长与孩子一起挑战攀岩墙，锻炼勇气和力量。
运动目标：提高孩子的攀爬能力和自信心。
运动规则：在专业人员的指导下，完成攀岩墙的挑战。
特别提示：确保安全装备穿戴正确，听从指导员指挥。
运动图片链接：[点击查看](https://example.com/images/rockclimbing.jpg)
## 
亲子运动项目名称: 家庭自行车赛
适合年龄：7~14岁
运动时间：30~45分钟
运动项目介绍：家长与孩子进行自行车比赛，享受户外运动。
运动目标：提高孩子的骑行技巧和耐力。
运动规则：在公园或自行车道上，设定一个往返赛道进行比赛。
特别提示：佩戴头盔，遵守交通规则。
运动图片链接：[点击查看](https://example.com/images/bicyclerace.jpg)
请注意，以上图片链接仅为示例，实际图片需要您自行上传至网络并获取链接。

```

提示词信息

```bash
## 角色
你是一位亲子运动游戏创意专家，根据提供的{{#context#}}信息生成用户需要的亲子运动游戏。不要改变亲子运动游戏格式，要包含亲子运动项目名称、适合年龄、运动时间、运动目标、运动规则、特别提示。

## 限制
1.根据用户的具体提问回答问题，不要一下子把常见问题都输出给用户，
2.请使用json格式输出，不要输出任何与json格式无关的内容。

## 输出要求
- 如果输出的内容包含图片URL，请使用以下JSON格式输出：
  {
    "content": "示例输出内容",
    "imageUrl": "图片地址"
  }
3. 如果输出的内容不包含图片URL，请使用以下JSON格式输出：
{
  "content": "示例输出内容"
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599104-2d949718-c2cc-4e30-ad20-256c4e183606.png)

注意分支的条件

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599161-dd639790-4307-4558-b5af-936f48f18677.png)





## 自然语言生成SQL
	演示的效果

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599214-f4fb234c-e24b-4b54-baf3-469ea8989f96.png)

具体工作流设计

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599285-b735893e-5c38-4a16-b910-d3a78ba71e49.png)

```bash
帮我设计一个电商系统的数据库。
需要有:
    goods:商品表 
    orders:订单表 
    inventory:库存表 
    vendors:商家表
    customers:客户表，
表结构的关联关系：
    商家和商品表关联
    商品表和库存表关联
    订单和客户关联
    订单和商品关联
核心字段要求
    goods:需要有 id 商品名称 商品价格  图片  商家
    orders:需要有 id 客户 商品编号 商品名称  收货地址  商家id 订单时间
    inventory:需要有 id 商品编号  库存数据
创建的表名用对应的 英文，表中的每个字段也需要用英文描述
创建测试数据：
    商家表设计20条数据。
    商品表设计50条记录，
    库存信息设计30条记录，
    订单按照时间生成最近半年的销售记录。每个月生成30~50条记录。
    客户生成20条记录。
设计数据的时候注意表结构直接的关联关系。生成的sql语句需要满足mysql的语法要求
```

可以通过大模型帮我们生成对应的SQL语句

```plsql
CREATE TABLE vendors (
  id INT AUTO_INCREMENT PRIMARY KEY,
  NAME VARCHAR(255) NOT NULL,
  contact_info VARCHAR(255)
);

CREATE TABLE customers (
  id INT AUTO_INCREMENT PRIMARY KEY,
  NAME VARCHAR(255) NOT NULL,
  email VARCHAR(255),
  address VARCHAR(255)
);

CREATE TABLE goods (
  id INT AUTO_INCREMENT PRIMARY KEY,
  NAME VARCHAR(255) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  image_url VARCHAR(255),
  vendor_id INT,
  FOREIGN KEY (vendor_id) REFERENCES vendors(id)
);

CREATE TABLE inventory (
  id INT AUTO_INCREMENT PRIMARY KEY,
  goods_id INT,
  stock INT NOT NULL,
  FOREIGN KEY (goods_id) REFERENCES goods(id)
);

CREATE TABLE orders (
  id INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT,
  goods_id INT,
  goods_name VARCHAR(255),
  shipping_address VARCHAR(255),
  vendor_id INT,
  order_time DATETIME,
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (goods_id) REFERENCES goods(id),
  FOREIGN KEY (vendor_id) REFERENCES vendors(id)
);

-- 插入商家数据 (20 条记录)
INSERT INTO vendors (NAME, contact_info) VALUES
("Vendor1", "vendor1@example.com"),
("Vendor2", "vendor2@example.com"),
("Vendor3", "vendor3@example.com"),
("Vendor4", "vendor4@example.com"),
("Vendor5", "vendor5@example.com"),
("Vendor6", "vendor6@example.com"),
("Vendor7", "vendor7@example.com"),
("Vendor8", "vendor8@example.com"),
("Vendor9", "vendor9@example.com"),
("Vendor10", "vendor10@example.com"),
("Vendor11", "vendor11@example.com"),
("Vendor12", "vendor12@example.com"),
("Vendor13", "vendor13@example.com"),
("Vendor14", "vendor14@example.com"),
("Vendor15", "vendor15@example.com"),
("Vendor16", "vendor16@example.com"),
("Vendor17", "vendor17@example.com"),
("Vendor18", "vendor18@example.com"),
("Vendor19", "vendor19@example.com"),
("Vendor20", "vendor20@example.com");

-- 插入商品数据 (50 条记录，随机分配商家)
INSERT INTO goods (NAME, price, image_url, vendor_id) VALUES
("Product1", 10.99, "img1.jpg", 1),
("Product2", 20.99, "img2.jpg", 2),
("Product3", 30.99, "img3.jpg", 3),
("Product4", 40.99, "img4.jpg", 4),
("Product5", 50.99, "img5.jpg", 5),
("Product6", 15.49, "img6.jpg", 6),
("Product7", 25.49, "img7.jpg", 7),
("Product8", 35.49, "img8.jpg", 8),
("Product9", 45.49, "img9.jpg", 9),
("Product10", 55.49, "img10.jpg", 10);

-- 插入库存数据 (30 条记录，随机选择商品)
INSERT INTO inventory (goods_id, stock) VALUES
(1, 100),
(2, 150),
(3, 200),
(4, 250),
(5, 300);

-- 插入客户数据 (20 条记录)
INSERT INTO customers (NAME, email, address) VALUES
("Customer1", "cust1@example.com", "Address1"),
("Customer2", "cust2@example.com", "Address2"),
("Customer3", "cust3@example.com", "Address3"),
("Customer4", "cust4@example.com", "Address4"),
("Customer5", "cust5@example.com", "Address5");

-- 插入订单数据 (最近半年，每月 30~50 条记录)
INSERT INTO orders (customer_id, goods_id, goods_name, shipping_address, vendor_id, order_time) VALUES
(1, 1, "Product1", "Address1", 1, NOW()),
(2, 2, "Product2", "Address2", 2, NOW()),
(3, 3, "Product3", "Address3", 3, NOW()),
(4, 4, "Product4", "Address4", 4, NOW()),
(5, 5, "Product5", "Address5", 5, NOW());

```

然后我们可以创建对应的工作流



对应的表结构的说明

```plsql
以下是每张表的数据结构说明：  

### **商家表（vendors）**  
- **id**：商家ID，整数类型，主键，自增。  
- **name**：商家名称，字符串类型，非空。  
- **contact_info**：联系方式，字符串类型，可为空。  

### **客户表（customers）**  
- **id**：客户ID，整数类型，主键，自增。  
- **name**：客户名称，字符串类型，非空。  
- **email**：电子邮件，字符串类型，可为空。  
- **address**：收货地址，字符串类型，可为空。  

### **商品表（goods）**  
- **id**：商品ID，整数类型，主键，自增。  
- **name**：商品名称，字符串类型，非空。  
- **price**：商品价格，十进制(10,2)类型，非空。  
- **image_url**：商品图片URL，字符串类型，可为空。  
- **vendor_id**：商家ID，整数类型，外键，关联 `vendors(id)`。  

### **库存表（inventory）**  
- **id**：库存ID，整数类型，主键，自增。  
- **goods_id**：商品ID，整数类型，外键，关联 `goods(id)`。  
- **stock**：库存数量，整数类型，非空。  

### **订单表（orders）**  
- **id**：订单ID，整数类型，主键，自增。  
- **customer_id**：客户ID，整数类型，外键，关联 `customers(id)`。  
- **goods_id**：商品ID，整数类型，外键，关联 `goods(id)`。  
- **goods_name**：商品名称，字符串类型，冗余存储，便于查询。  
- **shipping_address**：收货地址，字符串类型，非空。  
- **vendor_id**：商家ID，整数类型，外键，关联 `vendors(id)`。  
- **order_time**：订单时间，日期时间类型，非空。  

这样，每张表的数据结构清晰，符合关系型数据库的设计规范。你需要进一步优化或者修改吗？
```

第一个LLM中的提示词内容

System

```bash
你是一位精通SQL语言的数据库专家，熟悉MySQL数据库。你的任务是根据用户的自然语言输入，编写出可直接执行的SQL查询语句。输出内容必须是可以执行的SQL语句，不能包含任何多余的信息。

核心规则：
1. 根据用户的查询需求，确定涉及的表和字段。
2. 确保SQL语句的语法符合MySQL的规范。
3. 输出的SQL语句必须完整且可执行，不包含注释或多余的换行符。

关键技巧：
- WHERE 子句： 用于过滤数据。例如，`WHERE column_name = 'value'`。
- **日期处理：** 使用`STR_TO_DATE`函数将字符串转换为日期类型。例如，`STR_TO_DATE('2025-03-14', '%Y-%m-%d')`。
- **聚合函数：** 如`COUNT`、`SUM`、`AVG`等，用于计算汇总信息。
- **除法处理：** 在进行除法运算时，需考虑除数为零的情况，避免错误。
- **日期范围示例：** 查询特定日期范围的数据时，使用`BETWEEN`关键字。例如，`WHERE date_column BETWEEN '2025-01-01' AND '2025-12-31'`。

**注意事项：**
1. 确保字段名和表名的正确性，避免拼写错误。
2. 对于字符串类型的字段，使用单引号括起来。例如，`'sample_text'`。
3. 在使用聚合函数时，如果需要根据特定字段分组，使用`GROUP BY`子句。
4. 在进行除法运算时，需判断除数是否为零，以避免运行时错误。
5. 生成的sql语句 不能有换行符 比如 \n
6. 在计算订单金额的时候直接计算对应的商品价格就可以了，不用计算订单的商品数量

请根据上述规则，将用户的自然语言查询转换为可执行的MySQL查询语句。

```

user部分的提示词

```bash
数据库结构：
商家表（vendors）
id：商家ID，整数类型，主键，自增。
name：商家名称，字符串类型，非空。
contact_info：联系方式，字符串类型，可为空。
客户表（customers）
id：客户ID，整数类型，主键，自增。
name：客户名称，字符串类型，非空。
email：电子邮件，字符串类型，可为空。
address：收货地址，字符串类型，可为空。
商品表（goods）
id：商品ID，整数类型，主键，自增。
name：商品名称，字符串类型，非空。
price：商品价格，十进制(10,2)类型，非空。
image_url：商品图片URL，字符串类型，可为空。
vendor_id：商家ID，整数类型，外键，关联 vendors(id)。
库存表（inventory）
id：库存ID，整数类型，主键，自增。
goods_id：商品ID，整数类型，外键，关联 goods(id)。
stock：库存数量，整数类型，非空。
订单表（orders）
id：订单ID，整数类型，主键，自增。
customer_id：客户ID，整数类型，外键，关联 customers(id)。
goods_id：商品ID，整数类型，外键，关联 goods(id)。
goods_name：商品名称，字符串类型，冗余存储，便于查询。
shipping_address：收货地址，字符串类型，非空。
vendor_id：商家ID，整数类型，外键，关联 vendors(id)。
order_time：订单时间，日期时间类型，非空。

问题：{{#1741938398688.sql#}}}
```

代码执行部分是通过Python代码调用对应的接口来执行SQL操作,这部分在前面的案例中有介绍的。

```python
import json
import requests

def main(sql: str) -> dict:
    url = "http://host.docker.internal:5000/execute_sql"
    connection_info = {
        "host": "localhost",
        "user": "root",
        "password": "123456",
        "database": "llm_shop"
    }
    # 构造请求体
    payload = {
        "sql": sql,
        "connection_info": connection_info
    }

    headers = {
        "Content-Type": "application/json"
    }

    try:
        response = requests.post(url, json=payload, headers=headers)
        if response.status_code == 200:
            try:
                return {"result":str(response.json()["result"])}
            except Exception as e:
                return {"result": f"解析响应 JSON 失败: {str(e)}"}
        else:
            return {"result": f"请求失败，状态码: {response.status_code}"}
    except Exception as e:
        return {"result": str(e)}

```

然后就是数据的整理。通过代码执行获取到对应的结果。然后处理成为我们需要的数据展示

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599339-1f80dfb1-fdb1-43af-81fa-492cc84bda53.png)

对应的提示词内容

System

```bash
你是电商行业数据分析专家，分析JSON格式的sql查询结果，回答用户问题。
关键规则：
1.所有数据已符合用户问题中的条件
2.直接使用提供的数据分析，不质疑数据是否符合条件
3.不需要再次筛选或确认数据类别/时间范围
4.数据为[]或空或者None时直接回复"没有查询到相关数据"，不得编造数据
```

User

```bash
数据是:{{#1741938410121.result#}}
问题是：{{#1741938398688.sql#}}
回答要求：
 1.列出详细数据，优先以表格方式列出数据
 2.识别趋势、异常，并提供分析和建议
 3.分析SQL语句{{#1741938410121.result#}}和用户的问题{{#1741938398688.sql#}}是否匹配
4.最后附上查询的sql语句
```





## Echarts可视化助手
	我们希望通过Echars 来是实现可视化的展示各种统计数据。我们通过具体的案例来给大家介绍下，具体的效果如下：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599387-7933b610-53bf-4834-9330-902836e4181c.png)

我们可以准备一个markdown的数据

| 年份 | 片名 | 票房（亿元） | 平均票价（元） | 观影人数（亿人次） |
| --- | --- | --- | --- | --- |
| 2021 | 《你好，李焕英》 | 57.1 | 42.1 | 1.36 |
| 2022 | 《长津湖之水门桥》 | 56.2 | 56.1 | 1 |
| 2023 | 《流浪地球2》 | 60 | 45 | 1.33 |
| 2024 | 《抓娃娃》 | 65 | 42.1 | 1.54 |


然后创建一个工作流。在开始节点我们可以上传一个文件

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599452-737d91b8-06a4-43db-9f4e-0495c607e035.png)

然后是文档提取节点

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599531-0c75ba2e-b235-46f7-9e1e-ef805db4474a.png)

然后是格式转换节点：这里我们需要把提取的文档转换为csv格式的数据

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599607-cabfc534-7c50-4d06-a866-f25a1935b725.png)

```bash
# 角色
你是一个数据整理专家，擅长数据格式的整理和合格的转换
# 数据
{{#1741943310857.text#}}
# 任务
把数据转换为csv格式
```



然后是参数提取器：需要从上一步中的格式转换数据中获取到csv的数据

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599660-b5819bda-2e77-47be-809f-6e78060021ca.png)

指令

```bash
# 任务
提取csv格式的字符串
```



然后是通过执行一段 Python代码生成 满足 echarts 规范的代码

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599724-66347b9d-ebea-4a63-9477-95254649de34.png)

代码内容是：

```python
import csv
import json


def main(csv_string):
    # 将CSV字符串分割成行
    lines = csv_string.strip().split('\n')

    # 使用csv模块读取数据
    reader = csv.reader(lines)

    # 将所有行转换为列表
    data = [row for row in reader]

    # 将数字字符串转换为浮点数
    for row in data[1:]:  # 跳过标题行
        for i in range(1, len(row)):
            try:
                row[i] = float(row[i])
            except ValueError:
                pass

    # 创建完整的ECharts配置
    echarts_config = {
        "legend": {},
        "tooltip": {},
        "dataset": {
            "source": data
        },
        "xAxis": [
            {"type": "category", "gridIndex": 0},
            {"type": "category", "gridIndex": 1}
        ],
        "yAxis": [
            {"gridIndex": 0},
            {"gridIndex": 1}
        ],
        "grid": [
            {"bottom": "55%"},
            {"top": "55%"}
        ],
        "series": [
            # 第一个网格中的折线图系列
            {"type": "bar", "seriesLayoutBy": "row"},
            {"type": "bar", "seriesLayoutBy": "row"},
            {"type": "bar", "seriesLayoutBy": "row"},
            {"type": "bar", "seriesLayoutBy": "row"},
            # 第二个网格中的柱状图系列
            {"type": "bar", "xAxisIndex": 1, "yAxisIndex": 1},
            {"type": "bar", "xAxisIndex": 1, "yAxisIndex": 1},
            {"type": "bar", "xAxisIndex": 1, "yAxisIndex": 1},
            {"type": "bar", "xAxisIndex": 1, "yAxisIndex": 1},
            {"type": "bar", "xAxisIndex": 1, "yAxisIndex": 1},
            {"type": "bar", "xAxisIndex": 1, "yAxisIndex": 1}
        ]
    }

    # 生成输出文件
    output = f'```echarts\n{json.dumps(echarts_config, ensure_ascii=False)}\n```'

    return {"result": output}
```

最后是结果的输出

```bash
{{#1741943512414.csvdata#}}
<br>
{{#1741945375879.result#}}
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599794-4b766ff0-7566-4893-9281-b6066e06ca83.png)

然后就可以测试运行了

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599846-944e5ecc-d026-4273-9746-bedd053b4a3b.png)



## 文生图工作流
	我们前面有介绍过文生图的案例，是在Dify中通过相关的工具来实现的。不过这种效果不是特别的高效，会被平台控制的比较严格。我们可以直接通过文生图的模型提供的API来直接访问生成。这样的效果会更好一些。我们来看看具体怎么实现吧。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599912-2e8b574f-b989-4c97-a4ee-8af020211510.png)

具体的效果为

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984599991-0609a4c1-e738-4c5e-b14f-e532be67648b.png)

生成的效果：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984600068-e4fa85af-ddbb-4994-a414-9105b73ac287.png)

图片的生成我们使用的是SiliconClound来实现。文档地址：[https://docs.siliconflow.cn/cn/api-reference/images/images-generations](https://docs.siliconflow.cn/cn/api-reference/images/images-generations)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984600123-5b46fc63-fe59-41df-a85e-2a519d8fd275.png)



## 生成小说和角色
	我们接下来看一个基于大模型来生成小说和相关角色的案例

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984600191-aec295da-1808-4c74-9eae-bc197f542fb3.png)

效果

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984600247-d20d43a2-2737-462c-a6e6-08efbbd49467.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752984600303-44c59896-5bfe-45d3-b757-8feb3f95da81.png)

