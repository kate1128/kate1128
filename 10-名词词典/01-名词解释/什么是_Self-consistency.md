Google 在 ICLR 2023 发表的一篇论文 [Self-Consistency Improves Chain of Thought Reasoning in Language Models](https://arxiv.org/abs/2203.11171) 提出了一种新的解码策略：自我一致性（Self-consistency），**以替代思维链提示中的朴素贪婪解码**。

<br/>color2
如下图所示。自一致性方法包括三个步骤：

1. 使用思维链提示的大语言模型;
2. 通过从语言模型的解码器中采样来生成不同的推理路径集合，以取代 CoT 提示中的 “贪心解码”
3. 边缘化推理路径，并通过选择最终答案集中最一致的答案进行聚合。

<br/>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1739936520687-5978d434-4752-44de-95c6-b03f10cc1562.png)

**自我一致性扩展了思维链的构建，它能生成多个思维链，而不仅仅是一个，并通过多数投票选择最终答案。** 自我一致性策略利用了这样一种理念，即一个复杂的推理问题通常可以通过多种不同的思维方式得出同一个正确答案。具体实现是通过少样本 CoT 采样多个不同的推理路径，并从生成结果中选择最一致的答案。

```python
# 改造一下获得 Completions 的函数  设置参数 n=3，temperature=0.80，实现生成多个不尽相同的思维链

def get_completions(system_instruction, llm_prompt, model_endpoint):

    response = client.chat.completions.create(model=model_endpoint,
                                              messages=[{"role": "system", "content": system_instruction},
                                                        {"role": "user", "content": llm_prompt}
                                                       ],
                                              n=3, temperature=0.80, seed=42,
                                              presence_penalty=0, frequency_penalty=0,
                                              max_tokens=1024
                                             )

    return response.choices, response.usage
```

```python
llm = "gpt-3.5-turbo-0125"

system_instruction = "您是世界知识专家。"

# Few-shot 思维链提示过程
prompt = f"""问：罗杰有 5 个网球。他又买了 2 罐网球。每罐有 3 个网球。他现在有多少个网球？
答：罗杰开始有 5 个球。每罐有 3 个网球，两罐共有 6 个网球。5 + 6 = 11。答案是 11。
问：森林中开始有 15 棵树。林业工人今天将在林中种树。完成后，将有 21 棵树。林业工人今天种了多少棵树？
答：我们从 15 棵树开始。后来我们有 21 棵树。差异必须是他们种树的数量。因此，他们必须种了 21 - 15 = 6 棵树。答案是 6。
问：杰克有 23 美元。他以每个 3 美元的价格买了 5 个小蛋糕。他还剩下多少钱？
答：他以每个 3 美元的价格买了 5 个小蛋糕，总共花费了 3 * 5 = 15 美元。所以，他还剩下 23 - 15 = 8 美元。答案是 8 美元。
问：当我 6 岁时，我的妹妹的年龄是我的一半。现在我已经 24 岁了，那么我的妹妹现在的年龄是多少？
答：当你 6 岁时，你妹妹的年龄是 6 / 2 = 3 岁。现在你 24 岁，所以你妹妹的年龄是 24 - (6 - 3) = 21 岁。
问：李莉的鸭子每天下 16 个蛋。她每天早餐吃 3 个，每天用 4 个给朋友烤松饼。剩下的蛋她以每个 2 美元的价格出售。她每天能赚多少钱？
答：
"""

result_Chinese, tokens_count = get_completions(system_instruction, prompt, llm)
for response_num, con in enumerate(result_Chinese, start=1):
    result = con.message.content.strip()
    print(f"思维链-中文-{response_num}：\n{result}\n")
```

<details class="lake-collapse"><summary id="u2969e175"><span class="ne-text">思维链-中文-1：</span></summary><p id="ua0b85286" class="ne-p"><span class="ne-text">每天鸭子下 16 个蛋，李莉每天早餐吃 3 个，每天用 4 个给朋友烤松饼，所以她每天剩下 16 - 3 - 4 = 9 个蛋。她以每个 2 美元的价格出售，所以她每天能赚 9 * 2 = 18 美元。答案是 18 美元。</span></p></details>


<details class="lake-collapse"><summary id="u9a9d8c7b"><span class="ne-text">思维链-中文-2：</span></summary><p id="u0c15a4cc" class="ne-p"><span class="ne-text">李莉的鸭子每天下 16 个蛋。她每天早餐吃 3 个，每天用 4 个给朋友烤松饼，剩下的蛋她以每个 2 美元的价格出售。</span></p><p id="u8d0c47ce" class="ne-p"><span class="ne-text"></span></p><p id="u81e83597" class="ne-p"><span class="ne-text">剩下的蛋数量为 16 - 3 - 4 = 9 个。</span></p><p id="u86fd6785" class="ne-p"><span class="ne-text"></span></p><p id="u2d4845d3" class="ne-p"><span class="ne-text">她以每个 2 美元的价格出售这 9 个蛋，所以她每天能赚取的金额为 9 * 2 = 18 美元。所以，她每天能赚 18 美元。</span></p></details>


<details class="lake-collapse"><summary id="u57017869"><span class="ne-text">思维链-中文-3：</span></summary><p id="u41798cae" class="ne-p"><span class="ne-text">首先，李莉的鸭子每天下 16 个蛋。她每天早餐吃 3 个，每天用 4 个给朋友烤松饼，所以每天剩下 16 - 3 - 4 = 9 个蛋。她以每个 2 美元的价格出售蛋，所以每天能赚 9 * 2 = 18 美元。李莉每天能赚 18 美元。</span></p></details>
```python
llm = "gpt-3.5-turbo-0125"

system_instruction = "You are an expert at world knowledge."

prompt = f"""Q: Roger has 5 tennis balls. He bought 2 more cans of tennis balls. Each can has 3 tennis balls. How many tennis balls does he have now?
A: Roger starts with 5 balls. With 3 tennis balls in each can, there are 6 tennis balls in the two cans. 5 + 6 = 11. The answer is 11.
Q: There are 15 trees in the forest to begin with. Forest workers will plant trees in the forest today. When finished, there will be 21 trees. How many trees did the foresters plant today?
A: We started with 15 trees. Later we had 21 trees. The difference must be the number of trees they planted. Therefore, they must have planted 21 - 15 = 6 trees. The answer is 6.
Q: Jack has 23 dollars. He bought 5 cupcakes for $3 each. How much money does he have left?
A: He bought 5 cupcakes for $3 each and spent a total of 3 * 5 = $15. So, he has 23 - 15 = $8 left. The answer is $8.
Q: When I was 6 years old, my sister was half my age. Now that I am 24 years old, what is my sister's age now?
A: When you were 6 years old, your sister was 6 / 2 = 3 years old. Now you are 24, so your sister's age is 24 - (6 - 3) = 21.
Q: Lily's duck lays 16 eggs a day. She eats 3 each day for breakfast and uses 4 each day to bake muffins for her friends. She sells the remaining eggs for $2 each. How much money does she make each day?
A: 
"""

result_English, tokens_count = get_completions(system_instruction, prompt, llm)
for response_num, con in enumerate(result_English, start=1):
    result = con.message.content.strip()
    print(f"思维链-英文-{response_num}：\n{result}\n")
```

<details class="lake-collapse"><summary id="ua8f247ef"><span class="ne-text">思维链-英文-1：</span></summary><p id="u1c234d14" class="ne-p"><span class="ne-text">To calculate how much money Lily makes each day, we first need to determine how many eggs are left after she uses some for breakfast and baking:</span></p><p id="u54518d3a" class="ne-p"><span class="ne-text"></span></p><p id="udd5ba147" class="ne-p"><span class="ne-text">Total eggs laid per day: 16</span></p><p id="u6fb87d07" class="ne-p"><span class="ne-text">Eggs eaten for breakfast: 3</span></p><p id="u2d130b66" class="ne-p"><span class="ne-text">Eggs used for baking: 4</span></p><p id="u1c29857b" class="ne-p"><span class="ne-text"></span></p><p id="uaf24f703" class="ne-p"><span class="ne-text">Remaining eggs: 16 - 3 - 4 = 9</span></p><p id="u4c8c4836" class="ne-p"><span class="ne-text"></span></p><p id="u62bd6c7c" class="ne-p"><span class="ne-text">Now, we can calculate how much money Lily makes by selling the remaining eggs:</span></p><p id="u7d5e3842" class="ne-p"><span class="ne-text"></span></p><p id="ua5f6df04" class="ne-p"><span class="ne-text">Number of eggs sold: 9</span></p><p id="u98ea2af4" class="ne-p"><span class="ne-text">Price per egg: $2</span></p><p id="u8e0c48a7" class="ne-p"><span class="ne-text"></span></p><p id="u0a85808b" class="ne-p"><span class="ne-text">Total money made: 9 * 2 = $18</span></p><p id="u9f0cbfe6" class="ne-p"><span class="ne-text"></span></p><p id="u800eaa0f" class="ne-p"><span class="ne-text">Therefore, Lily makes $18 each day by selling the remaining eggs.</span></p></details>


<details class="lake-collapse"><summary id="u940156c4"><span class="ne-text">思维链-英文-2：</span></summary><p id="u11ec69f8" class="ne-p"><span class="ne-text">To calculate how much money Lily makes each day, we first need to determine how many eggs she has left after using some for breakfast and baking muffins.</span></p><p id="u4d844b7d" class="ne-p"><span class="ne-text"></span></p><p id="u476e996e" class="ne-p"><span class="ne-text">If the duck lays 16 eggs a day and Lily uses 3 for breakfast and 4 for baking muffins, she uses a total of 3 + 4 = 7 eggs.</span></p><p id="ue4b651cd" class="ne-p"><span class="ne-text"></span></p><p id="ub9ce1776" class="ne-p"><span class="ne-text">So, the number of eggs remaining for sale is 16 - 7 = 9 eggs.</span></p><p id="u8b0de29b" class="ne-p"><span class="ne-text">...</span></p><p id="uf2b9d1ff" class="ne-p"><span class="ne-text">Multiplying the number of eggs sold by the price per egg gives us the daily earnings: 9 * $2 = $18.</span></p><p id="u238431e6" class="ne-p"><span class="ne-text"></span></p><p id="uac07c7e1" class="ne-p"><span class="ne-text">Therefore, Lily makes $18 each day selling duck eggs.</span></p><p id="u0667cf66" class="ne-p"><span class="ne-text"></span></p><p id="u7d3aeff8" class="ne-p"><span class="ne-text">Output is truncated. View as a scrollable element or open in a text editor. Adjust cell output settings...</span></p></details>
