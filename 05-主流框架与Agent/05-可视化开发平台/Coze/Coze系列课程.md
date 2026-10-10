[Coze案例：复刻爆款视频.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1753079420084-d0a73d1f-df52-48a7-8d67-5751f4fb4f3f.pdf)

[Coze案例：根据官方模板复刻应用：海报.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1753079437673-028c0d3a-202a-44a9-987c-20bde8372290.pdf)

[Coze案例：一键生成行业调研PPT.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1753079429078-8df9368b-73b5-482d-b547-9a9f7c1c8482.pdf)

[DeepSeek+Coze搭建工作流(爆款视频、调研报告、商业海报等实操).pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1753079418856-278c107d-1fb8-45f4-8357-2668309feff6.pdf)

# Coze 基础介绍
coze介绍可以直接看官网：[https://www.coze.cn/open/docs/guides](https://www.coze.cn/open/docs/guides)

coze有两个版本

+ 国内版 [http://www.coze.cn](http://www.coze.cn)
+ 国际版 [http://www.coze.com](http://www.coze.com)

# Coze案例讲解
## 搭建一个 AI 助手智能体
	第一个案例通过官方案例来实现: [https://www.coze.cn/open/docs/guides/quickstart](https://www.coze.cn/open/docs/guides/quickstart)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995460881-c8eebaf3-c898-4c08-916d-04f6a710ec3c.png)



## 搭建一个PPT智能体
	在coze的插件市场中提供的有一个`iSilde`的插件，专门为我们提供PPT的生成功能，他提供了5个具体的插件功能

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995460957-8310629a-424c-43cc-8cc0-cd25bca93f80.png)



| 插件名称 | 说明 |
| --- | --- |
| `get_ppt` | 通过任务id生成ppt。 |
| `get_outline` | 根据关键字获取大纲。 |
| `get_thumbnail_with_outline` | 通过大纲的详细内容和模版id生成ppt的缩略图，并输出生成ppt的任务id。 |
| `get_thumbnail` | 根据模版id和大纲id获取PPT的缩略图，并输出生成ppt的任务id。 |
| `get_themes` | 根据关键字获取模版。 |


具体的案例实现

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461005-b760e587-ebef-44b3-846b-2966e027e2df.png)



对应的提示词

```python
# 角色：{#InputSlot placeholder="角色名称" mode="input"#}iSlide插件专家{#/InputSlot#}
你是一名基于iSlide生成PPT文档的技术专家

## 目标：
{#InputSlot placeholder="角色的工作目标，如果有多目标可以分点列出，但建议更聚焦1-2个目标" mode="input"#}通过用户输入的相关信息能准确的生成相关的PPT文件{#/InputSlot#}

    ## 技能：
    1.  {#InputSlot placeholder="为了实现目标，角色需要具备的技能1" mode="input"#}通过用户输入的主题能借助'get_outline'插件获取对应的大纲内容{#/InputSlot#}
    2. {#InputSlot placeholder="为了实现目标，角色需要具备的技能2" mode="input"#}通过用户输入的样式能借助'get_themes'获取对应的模板信息{#/InputSlot#}
    3. {#InputSlot placeholder="为了实现目标，角色需要具备的技能3" mode="input"#}通过上面两部获取的大纲和模版id通过'get_thumbnail_with_outline'获取生成ppt的缩略图和需要生成的ppt的id
    4. 最终通过'get_ppt'来生成ppt文件{#/InputSlot#}

## 工作流：
1. 通过get_outlien获取用户输入主题对应的PPT大纲
2. 通过get_themes获取用户输入样式对应的PPT模板，返回第一个模板信息的id
3. {#InputSlot placeholder="描述角色工作流程的第三步" mode="input"#}通过get_thumbnail_with_outline 这个插件结合前面两步获取的大纲和模板id来获取ppt的缩略图和ppt的任务id
    4. 通过get_ppt 结合第三步获取的任务id来生成最终的PPT{#/InputSlot#}

    ## 输出格式：
    {#InputSlot placeholder="如果对角色的输出格式有特定要求，可以在这里强调并举例说明想要的输出格式" mode="input"#}输出的是对应PPT文件的下载链接{#/InputSlot#}

        ## 限制：
        - 生成的内容需要正面积极
        - 生成的内容不得违法国家政策
```



## 创建工作流
	上面我们创建一个PPT是通过智能体来实现的。把iSlide中的4个插件来组织起来生成对应的PPT信息，具体的流程还是需要模型来实现，我们可以通过工作流的方式来编排这个PPT的生成。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461069-df360a0c-7627-419a-a75c-637cbd878cf8.png)

开始节点

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461125-95076cab-ad0f-4570-9dfc-e566ffb5d32f.png)

在开始节点我们需要设置两个输入变量

+ 主题
+ 样式



通过输入的主题获取PPT的大纲信息

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461177-2c543762-2b1d-4910-ae09-1a3bc7132a13.png)

然后通过样式来获取模版信息

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461235-800a11fb-e846-445b-9ed8-5ad4503e1822.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461291-bd198ca8-f3ec-4a90-9c8b-3808cba13eec.png)

然后需要通过一段代码来处理下获取的模板信息

```python
# 在这里，您可以通过 ‘args’  获取节点中的输入变量，并通过 'ret' 输出结果
# 'args' 和 'ret' 已经被正确地注入到环境中
# 下面是一个示例，首先获取节点的全部输入参数params，其次获取其中参数名为‘input’的值：
# params = args.params; 
# input = params.input;
# 下面是一个示例，输出一个包含多种数据类型的 'ret' 对象：
# ret: Output =  { "name": ‘小明’, "hobbies": [“看书”, “旅游”] };

async def main(args: Args) -> Output:
    params = args.params
# 构建输出对象
ret: Output = {
    "mode_id": params['input'][0]['id']
}
return ret
```

然后通过`get_thumbnail_with_outline`来获取缩列图和任务id

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461353-4dab6aed-c1e4-4b22-b016-a5faf3ecb5fe.png)

到这里就可以通过`get_ppt`来生成具体的ppt了

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461407-41f89b18-06e8-4397-8fdb-fbce2a1ac45b.png)

然后在输出的节点输出生成的ppt文件即可

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461465-8f2d94d8-ff84-4826-ae83-5fd16a9b1e97.png)

生成的PPT效果，然后把我们刚刚创建的工作流发布会就可以在智能体中使用了

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461547-998ced14-fe80-4196-b017-fa6c91808c38.png)

然后提示词和插件就可以调整了

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461603-c9b82fd2-3888-40ca-a6e0-1d0c88119ad1.png)

生成的效果

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461662-18538f44-a468-416b-9916-c9f5a79c83ee.png)



## 搭建一个PPT应用
然后我们也可以通过coze中的应用来构建我们的PPT生成工具，效果如下

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461735-5048c73a-d676-4e1b-8936-e2c80bdf4872.png)

具体的流程图效果为

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461793-d4423847-10a2-471e-98b3-9788c6d2c2c9.png)

需要记录的几个点

### 大模型的提示词
```json
基于用户提供的资料，自主调用工具，生成相关行业发
展现状的PPT文本大纲
按照指定的JSON格式输出，如下
{
  "sections": [
    {
      "contents": [
        {
          "items": [
            {
              "items": [
                "结合时代背景，考虑参与者的兴趣和需求"
              ],
              "title": "1.1.1 主题创意来源"
            }
          ],
          "subtitle": "突出特色，引起共鸣",
          "title": "1.1 活动主题设定"
        }
      ],
      "subtitle": "展示才艺，增进交流，创建和谐氛围",
      "title": "1. 联欢会目标"
    }
  ],
  "subtitle": "精心组织，确保活动顺利进行，达到预期的效果和目标",
  "title": "联欢会策划与管理"
}
```

其实就是要参考`get_outline`这个插件返回的大纲内容的结构了

```python
根据用户输入的关键字
PPT的主题:{{themes}}
PPT的样式:{{style}}
和提供的和关键字相关的内容生成符合格式的PPT大纲
```



### 两个代码中的处理方法
```python
# 在这里，您可以通过 ‘args’  获取节点中的输入变量，并通过 'ret' 输出结果
# 'args' 和 'ret' 已经被正确地注入到环境中
# 下面是一个示例，首先获取节点的全部输入参数params，其次获取其中参数名为‘input’的值：
# params = args.params; 
# input = params.input;
# 下面是一个示例，输出一个包含多种数据类型的 'ret' 对象：
# ret: Output =  { "name": ‘小明’, "hobbies": [“看书”, “旅游”] };

async def main(args: Args) -> Output:
    params = args.params
url_list = [i['url'] for i in params["input"]]
# 构建输出对象
ret: Output = {
    "url_list": url_list
}
return ret
```

```python
# 在这里，您可以通过 ‘args’  获取节点中的输入变量，并通过 'ret' 输出结果
# 'args' 和 'ret' 已经被正确地注入到环境中
# 下面是一个示例，首先获取节点的全部输入参数params，其次获取其中参数名为‘input’的值：
# params = args.params; 
# input = params.input;
# 下面是一个示例，输出一个包含多种数据类型的 'ret' 对象：
# ret: Output =  { "name": ‘小明’, "hobbies": [“看书”, “旅游”] };

async def main(args: Args) -> Output:
    params = args.params
# 构建输出对象
ret: Output = {
    "mode_id": params['input'][0]['id']
}
return ret
```

在设置应用的时候需要注意的是事件的设置

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1752995461846-1b0288e6-27a1-4396-b29f-4ce600b1b941.png)

