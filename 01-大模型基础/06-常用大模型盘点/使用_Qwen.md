[通义千问](https://www.yuque.com/qiaokate/su87gb/clp9wsy5oawg9k2e)

### 安装 Qwen SDK
```shell
pip install dashscope
```

### 调用 QWen API
```python
# qwen
from http import HTTPStatus
import dashscope

DASHSCOPE_API_KEY="" # 请填写您自己的APIKey

messages = [{'role': 'system', 'content': 'You are a helpful assistant.'},
            {'role': 'user', 'content': '请你介绍一下Datawhale。'}]

responses = dashscope.Generation.call(
    dashscope.Generation.Models.qwen_max, # 请选择参考官方文档，填写需要调用的模型名称
    api_key=DASHSCOPE_API_KEY, 
    messages=messages,
    result_format='message',  # 将结果设置为“消息”格式
    stream=True, #流式输出
    incremental_output=True  
)


full_content = ''  # 合并输出
for response in responses:
    if response.status_code == HTTPStatus.OK:
        full_content += response.output.choices[0]['message']['content']
        # print(response)
    else:
        print('Request id: %s, Status code: %s, error code: %s, error message: %s' % (
            response.request_id, response.status_code,
            response.code, response.message
        ))
print('回答:\n' + full_content)
"""
回答:
Datawhale是一个专注于数据科学与人工智能领域的开源组织，由一群来自国内外顶级高校和知名企业的志愿者们共同发起。该组织以“学习、分享、成长”为理念，通过组织和运营各类高质量的公益学习活动，如学习小组、实战项目、在线讲座等，致力于培养和提升广大学习者在数据科学领域的知识技能和实战经验。

Datawhale积极推广开源文化，鼓励成员参与并贡献开源项目，已成功孵化了多个优秀的开源项目，在GitHub上积累了大量的社区关注度和Star数。此外，Datawhale还与各大高校、企业以及社区开展广泛合作，为在校学生、开发者及行业人士提供丰富的学习资源和实践平台，助力他们在数据科学领域快速成长和发展。

总之，Datawhale是一个充满活力、富有社会责任感的开源学习社区，无论你是数据科学的小白还是资深从业者，都能在这里找到适合自己的学习路径和交流空间。
"""
```

  
 

