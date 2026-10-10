<!-- type: glossary -->
# Claude 系列（Anthropic）

> Anthropic 公司开发的闭源大语言模型系列，以**安全性、诚实性和长上下文处理**著称，是 GPT-4 最重要的竞品。

## 基本情况

- **开发者**：Anthropic（由前 OpenAI 员工 Dario Amodei 和 Daniela Amodei 于 2021 年创立）
- **核心理念**：Constitutional AI（宪法 AI）——通过预设的"宪法"规则来约束模型行为，而非纯 RLHF
- **使用地址**：[claude.ai](https://claude.ai)

## 主要模型

| 模型 | 定位 | 上下文 | 输入价格 ($/1M tokens) | 输出价格 |
| :--- | :--- | :--- | :--- | :--- |
| Claude 3.5 Haiku | 最快、最便宜 | 200K | 0.25 | 1.25 |
| Claude 3.5 Sonnet | 平衡（推荐） | 200K | 3 | 15 |
| Claude 3.5 Opus | 最强推理 | 200K | 15 | 75 |

## 关键区别点

- **长上下文是标配**：全系列 200K token（约 15 万词），比 GPT-4 标准版（128K）更长
- **代码和推理**：Claude 3.5 Sonnet 在编程任务上表现突出，是 Cursor/Copilot 等编程助手的常用后端
- **Artifacts 功能**：对话中直接生成可预览的代码/文档/React 组件
- **Constitutional AI**：更注重安全对齐，拒绝回答有害请求的准确性更高

## 相关笔记

- [[GPT 系列]] — Claude 的主要竞品
- [[System Prompt]] — Claude 对 System Prompt 的遵循度很高
- [[模型选型指南](../../../07-LLM应用开发/01-LLM应用开发/对话型应用的设计/第一步_模型选型.md)] — 何时选 Claude

## 参考

- [Anthropic 官网](https://www.anthropic.com/)
- [Claude API 文档](https://docs.anthropic.com/)