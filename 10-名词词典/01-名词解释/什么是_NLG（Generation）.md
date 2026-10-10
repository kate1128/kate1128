<!-- type: glossary -->
# NLG（自然语言生成，Natural Language Generation）

> NLP 的两大核心任务之一，目标是让计算机**自动生成**流畅、连贯、符合语境的人类语言文本。LLM 的本质就是 NLG。

## 是什么

NLG 关心的是"怎么把信息漂亮地说出来"：

- **数据到文本**：给定一组天气数据（温度 28°C, 湿度 65%, 晴），生成"今天天气晴朗，气温 28°C，湿度适中"
- **摘要生成**：输入一篇长文章，输出 3 句话的精简摘要
- **对话生成**：用户说"你好"，模型回复"你好！有什么可以帮助你的？"
- **代码生成**：输入自然语言描述，输出 Python/JS/SQL 代码

NLG 经历了从**模板填充** → **统计方法** → **神经网络** → **LLM（预训练+生成）** 的演进。现代 LLM（如 GPT 系列）将 NLG 能力推到了前所未有的高度——不仅能生成流畅文本，还能推理、编程、角色扮演。

## 为什么重要

- **LLM 的"输出层"**：ChatGPT、Claude 的每一次回复都是 NLG
- **生产力倍增器**：自动写邮件、写报告、写代码、翻译——这些 NLG 应用直接影响工作效率
- **创造力工具**：AIGC（AI Generated Content）本质上是 NLG + 图像生成 + 视频生成，覆盖内容生产全链路

## 关键要点

- **生成策略**：贪心解码（选最可能的 token）、Beam Search（保留多条候选）、采样（Temperature/Top-p 控制随机性）
- **Temperature 是核心参数**：低 Temperature（0-0.3）→ 更确定、适合事实型任务；高 Temperature（0.7-1.0）→ 更有创意、适合创意写作
- **NLG 的核心挑战**：事实准确性（不编造）、一致性（不前后矛盾）、多样性（不重复模板化）
- **评估困难**：BLEU/ROUGE 等自动化指标只能参考，人工评估仍是金标准

## 相关概念

- [[NLU 自然语言理解]] — NLU 负责"读懂"输入，NLG 负责"生成"输出
- [[LLM 大型语言模型]] — 当前最强大的 NLG 系统
- [[幻觉]] — NLG 的核心缺陷：生成看似合理但事实错误的内容
- [[Temperature]] — 控制 NLG 随机性的关键参数

## 参考

- [Hugging Face - Text Generation](https://huggingface.co/tasks/text-generation)
- [OpenAI - Text Generation Guide](https://platform.openai.com/docs/guides/text-generation)