<!-- type: glossary -->
# Function Calling（函数调用）

> LLM 的一项核心能力：模型不再只输出自然语言文本，而是输出结构化的 JSON 对象，精确描述要调用哪个函数以及用什么参数。这是连接 LLM 与外部世界的桥梁。

## 是什么

Function Calling 改变了 LLM 的交互模式。传统模式是"人问 → 模型答"，Function Calling 变成了**"人问 → 模型决定调用什么工具 → 外部代码执行 → 结果返回模型 → 模型整合回答"**。

工作流程：
1. 开发者用 JSON Schema 定义可用的函数（函数名、参数、参数类型、描述）
2. 用户提问后，模型判断是否需要调用函数
3. 如需调用，模型输出一个包含函数名和参数的 JSON（而不是自然语言）
4. 开发者的代码真正执行这个函数（查数据库、调 API、发邮件等）
5. 执行结果返回给模型，模型整合后生成最终回复

> 注意：模型只"决定"调用什么、不"执行"函数——真正的执行是开发者代码做的。这保证了安全性。

## 为什么重要

- **Agent 的技术基础**：没有 Function Calling，Agent 就无法"做事"——搜索网页、查数据库、发邮件这些都需要模型先输出结构化的调用指令
- **确定性输出**：过去想从 LLM 获取结构化数据，只能靠 Prompt 祈祷它返回合法 JSON。Function Calling 让这个过程变得可靠
- **生态连接器**：Function Calling 让 LLM 能对接任何有 API 的系统——CRM、ERP、日历、代码解释器

## 关键要点

- **主流模型都支持**：OpenAI（GPT-3.5+）、Anthropic（Tool Use）、Google Gemini、以及多数国产模型
- **与 Tool Calling 的关系**：OpenAI 现在把更完整的工具调用能力称为 Tool Use / Tools API，Function Calling 是其中的基础机制
- **MCP 是下一个演进**：Anthropic 的 MCP（Model Context Protocol）试图把 Function Calling 标准化——定义一套"模型如何发现和调用工具"的通用协议
- **Parallel Function Calling**：GPT-4 起支持在单次响应中同时调用多个函数，例如"查北京和上海的天气"

## 相关概念

- [[Agent 智能体]] — Function Calling + 规划能力 = Agent
- [[MCP 模型上下文协议]] — Function Calling 的标准化演进方向
- [[结构化输出]] — Function Calling 在确定性输出方面的技术支撑

## 参考

- [OpenAI Function Calling Guide](https://platform.openai.com/docs/guides/function-calling)
- [Anthropic Tool Use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)