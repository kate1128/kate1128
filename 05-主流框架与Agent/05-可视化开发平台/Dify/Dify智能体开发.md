> 就是介绍下如何通过 Dify 自定义工作流工具构建游戏智能体
>
> [Dify智能体开发入门教程 | 使用自定义工作流工具构建游戏智能体](https://mp.weixin.qq.com/s/gndehDqzfCfWNWwV2tBuVg)
>

#  引言
Dify是一个开源的LLM应用开发平台，通过直观的界面简化基于语言模型的应用开发和部署过程。Dify内置了构建LLM应用所需的关键技术栈，包括稳健的Agent框架和灵活的流程编排。

Dify允许用户在画布上构建和测试功能强大的AI工作流。工作流通过将复杂任务分解为更小的步骤（节点），有效降低了系统的复杂度。这种方法减少了对提示词技术和模型推理能力的依赖，从而提升了 LLM 在处理复杂任务时的性能，同时增强了系统的可解释性、稳定性和容错性。

# 私有化部署Dify
[私有化部署](https://www.yuque.com/qiaokate/su87gb/pgawgbr8ar69q650)

# 要点
本文以实现24点游戏为例。24点的游戏规则：给出一组4个随机整数(1至13之间，且不重复)，用加、减、乘、除(可加括号)把给出的4个数字算成24，每个数必须用一次且只能用一次。

这个智能体需要具备如下能力：

+ 出题：生成一组4个随机整数(1至13之间，且不重复)，并确保生成的随机数能计算出24。
+ 校验用户输入的表达式计算结果是否为24。
+ 解题：根据用户给出的随机数，生成答案。

通过设置合适的提示词，为智能体设定角色和处理逻辑。智能体会根据大语言模型对人物设定和回复逻辑的理解，来响应用户问题。因此提示词编写的越清晰明确，智能体的回复也会越符合预期。可以在提示词中指定用工作流作逻辑处理，实现通过prompt无法实现的功能。

# 创建工作流工具
在`http://localhost/apps`页面点击“创建空白应用”，选择“工作流”。填写应用名，点击创建。依次创建三个工作流：

+ `generate_random_numbers`：为24点游戏生成一组随机数
+ `check_answer`：校验表达式计算结果是否为24
+ `generate_answer`：根据一组随机数生成24点游戏的答案

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736594592212-3f87d078-5001-4646-89bb-294d5c801642.png)

## 工作流1：`generate_random_numbers`
整体流程如下图：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736594738417-68bc9299-ec8d-4da4-88d9-3bc93478674b.png)

#### 名称
`generate_random_numbers`

#### 描述
为24点游戏生成一组随机数

#### 开始节点
用默认配置，无需添加输入字段

#### 代码执行节点
+ 输入变量：无
+ 输出变量：
    - numbers，类型为Array[Number]
+ 代码：

```python
import random
from itertools import permutations, product

def main() -> dict:
    while True:
        numbers = []
        while len(numbers) < 4:
            num = random.randint(1, 13)
            if num not in numbers:
                numbers.append(num)
        res = generate_answer(numbers)
        # 确保生成的随机数能计算出24
        if res['code'] == 'ok':
            return {'numbers': numbers}

def generate_answer(numbers):
    if len(numbers) != 4:
        return {'code': 'error', 'msg': "随机数个数不正确"}
    
    operations = ['+', '-', '*', '/']
    for num_perm in permutations(numbers):
        for ops in product(operations, repeat=3):
            # 尝试所有不同的括号组合
            expressions = [
                f'(({num_perm[0]} {ops[0]} {num_perm[1]}) {ops[1]} {num_perm[2]}) {ops[2]} {num_perm[3]}',
                f'({num_perm[0]} {ops[0]} ({num_perm[1]} {ops[1]} {num_perm[2]})) {ops[2]} {num_perm[3]}',
                f'({num_perm[0]} {ops[0]} {num_perm[1]}) {ops[1]} ({num_perm[2]} {ops[2]} {num_perm[3]})',
                f'{num_perm[0]} {ops[0]} (({num_perm[1]} {ops[1]} {num_perm[2]}) {ops[2]} {num_perm[3]})',
                f'{num_perm[0]} {ops[0]} ({num_perm[1]} {ops[1]} ({num_perm[2]} {ops[2]} {num_perm[3]}))',
            ]
            for expr in expressions:
                try:
                    if eval(expr) == 24:
                        return {'code': 'ok', 'answer': expr}
                except ZeroDivisionError:
                    continue
    return {'code': 'error'}
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595194695-16d77a6b-e5c3-4000-83e1-d040cae59610.png)

#### 结束节点
输出变量numbers的参数值为引用“代码执行”节点的输出变量numbers。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736594874170-aae26b44-8cf6-4320-81e3-646032d9810a.png)

#### 运行
点击右上角的“运行”，验证是否可以正确输出一组随机数。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595165248-74f0bdb5-cef4-4303-9657-0cb1bf1d094e.png)

#### 发布工作流
测试通过后，点击右上角的“发布”按钮。

#### 发布为工具
发布成功后，点击”发布为工具“

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595280944-d5f8bb6b-e18a-4a08-9c66-bb70c0d9de92.png)

填入工具调用名称和工具描述，并保存。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595263492-b11085fd-9c4e-40ea-aadc-2f80bc53fbf8.png)

## 工作流2：`check_answer`
整体流程如下图：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595388513-53d531a7-da33-414e-be75-b1fb40ffb024.png)

#### 名称
check_answer

#### 描述
校验表达式计算结果是否为24

#### 开始节点
输入参数

+ expression：类型为String，描述为“表达式”

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595444277-c0718314-779a-4d51-9264-7eced8cce355.png)

#### 代码执行节点
+ 输入变量：
    - expression，引用开始节点的expression变量
+ 输出变量：
    - code，类型为String
    - msg, 类型为String
+ 代码：

```python
def main(expression:str) -> dict:
    try:
        val = eval(expression)
        if val == 24:
            return {'code': 'ok', 'msg':'ok'}
        else:
            return {'code': 'error', 'msg': f"表达式{expression}计算结果为{val}, 不是24"}
    except Exception as e:
        return {'code': 'error', 'msg': f"计算出错。{e}"}
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595530743-c3ac09e9-48e4-4aa1-a3f0-b37937b48d83.png)

#### 结束节点
+ 输出变量
    - code，引用代码执行节点的code输出变量
    - msg, 引用代码执行节点的msg输出变量

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595588329-c6f4e2bf-12c7-4481-90a4-1c5956736e85.png)

#### 测试并发布工作流
#### 发布为工具
发布成功后，点击”发布为工具“。填入工具调用名称和工具描述，并保存。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595748259-52cc5118-87d4-4b1c-a1d9-c2771541909d.png)

## 工作流3：`generate_answer`
整体流程如下图：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736595924165-d155f62e-ab2b-4eac-84af-626787ad4b3b.png)

#### 名称
+ generate_answer

#### 描述
+ 根据一组随机数生成24点游戏的答案

#### 开始节点
    - 输入参数
        * numbers：类型为String，描述为“一组随机整数，JSON格式”

#### 代码执行节点
    - 输入变量
        * numbers，引用开始节点的numbers变量
    - 输出变量
        * code，类型为String
        * msg, 类型为String
        * answer, 类型为String
    - 代码

```python
from itertools import permutations, product
import json

def main(numbers:str) -> dict:
    numbersArray = json.loads(numbers)
    if len(numbersArray) != 4:
        return {'code': 'error', 'msg': "随机数个数不正确", 'answer':''}
    
    operations = ['+', '-', '*', '/']
    for num_perm in permutations(numbersArray):
        for ops in product(operations, repeat=3):
            # 尝试所有不同的括号组合
            expressions = [
                f'(({num_perm[0]} {ops[0]} {num_perm[1]}) {ops[1]} {num_perm[2]}) {ops[2]} {num_perm[3]}',
                f'({num_perm[0]} {ops[0]} ({num_perm[1]} {ops[1]} {num_perm[2]})) {ops[2]} {num_perm[3]}',
                f'({num_perm[0]} {ops[0]} {num_perm[1]}) {ops[1]} ({num_perm[2]} {ops[2]} {num_perm[3]})',
                f'{num_perm[0]} {ops[0]} (({num_perm[1]} {ops[1]} {num_perm[2]}) {ops[2]} {num_perm[3]})',
                f'{num_perm[0]} {ops[0]} ({num_perm[1]} {ops[1]} ({num_perm[2]} {ops[2]} {num_perm[3]}))',
            ]
            for expr in expressions:
                try:
                    if eval(expr) == 24:
                        return {'code': 'ok', 'msg':'ok', 'answer': expr}
                except ZeroDivisionError:
                    continue
    return {'code': 'error', 'msg': 'error', 'answer':''}
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596117032-123e1b48-c948-4226-8eed-a5005c469155.png)

#### 结束节点
+ 输出变量
    - code，引用代码执行节点的code输出变量
    - msg, 引用代码执行节点的msg输出变量
    - answer, 引用代码执行节点的answer输出变量

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596155243-fbc40e16-e8f0-4cb2-9024-e945e1159b27.png)

#### 测试并发布工作流
#### 发布为工具
发布成功后，点击”发布为工具“。填入工具调用名称和工具描述，并保存。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596230534-92bb7823-957b-4340-adc8-ce1d45ecd23d.png)

# 创建Agent应用
在`http://localhost/apps`页面点击“创建空白应用”，选择“Agent”。填写应用名，点击创建，进入编排界面。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596267231-91481ac5-93dc-45a9-a017-351310661d7a.png)

#### 设置提示词
内容为：

```python
你是一个24点游戏助手。- 开始游戏时，你需要生成一组随机数，提示用户回答，然后使用工作流check_answer校验用户的回答。- 如果用户表示回答不了问题，请使用工作流generate_answer生成答案。- 用户可以向你提供一组数字提问如何计算，你需要使用工作流generate_answer生成答案。
```

编排界面如下：

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596325185-89e54903-2771-4948-a78a-b563388b0ead.png)

#### 添加工具
把3个工作流添加为工具。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596344625-5c343c7f-7b6f-468d-aac0-79b85021cf1e.png)

#### 选择模型
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596377006-35785d59-9244-4f59-b4f6-f755cf7cfdd7.png)

#### 调试和预览
在下方输入内容和Agent进行游戏互动。  
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596492467-5eb59551-b74f-42f3-af07-a17bd6e36611.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596508714-a90e9f93-1ed9-4605-b1a9-28d81d77495b.png)

#### 测试通过后，点击右上角的“发布”按钮。
#### 发布后，点击“运行”即可打开应用的访问链接。
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596547417-6b3d7940-2738-47c5-b49c-8016adc7fb81.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736596559787-2ed4c0c2-0944-44c5-8874-b8900bb07cae.png)

