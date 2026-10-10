<!-- type: glossary -->
# GLM 系列（智谱 AI / 清华大学）

> 清华大学与智谱 AI 合作研发的**国产大语言模型系列**，以独特的双向注意力架构和多模态能力著称，是国内学术+产业结合的标杆。

## 基本情况

- **开发者**：智谱 AI（Zhipu AI）+ 清华大学 KEG 实验室
- **首次发布**：ChatGLM（2023.03）→ ChatGLM2（2023.06）→ ChatGLM3（2023.10）→ GLM-4（2024.01）
- **使用地址**：[chatglm.cn](https://chatglm.cn/)
- **开源地址**：[github.com/THUDM/GLM-4](https://github.com/THUDM/GLM-4)

## 主要模型

| 模型 | 定位 | 亮点 |
| --- | --- | --- |
| GLM-4-9B-Chat | 对话模型（主打） | 128K 上下文、网页浏览、代码执行、Function Call |
| GLM-4-9B | 基础模型 | 适合微调 |
| GLM-4-9B-Chat-1M | 超长上下文 | **支持 1M（100 万）token 上下文** |
| GLM-4V-9B | 多模态 | 图文理解 |
| CogView | 文生图 | 类似 DALL-E |

## 特点

- **开源矩阵对标 OpenAI**：对话、长文本、多模态、代码执行——产品矩阵完整
- **1M 超长上下文**：GLM-4-9B-Chat-1M 是当时开源模型中上下文最长的
- **学术基因**：来自清华大学，在 NLP 顶会（ACL/EMNLP）有大量相关论文
- **Function Calling 支持**：国内较早支持原生 Function Calling 能力的模型

## 相关笔记

- [[通义千问 Qwen]] — 阿里的 LLM 产品线
- [[Baichuan 系列]] — 百川智能的 LLM
- [[模型选型指南](../../../07-LLM应用开发/01-LLM应用开发/对话型应用的设计/第一步_模型选型.md)]

## 参考

- [智谱 AI 开放平台](https://open.bigmodel.cn/)
- [GLM-4 GitHub](https://github.com/THUDM/GLM-4)