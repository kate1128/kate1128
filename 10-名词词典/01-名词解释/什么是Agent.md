<!-- type: glossary -->
# AI Agent（智能体）

> 一种能够自主感知环境、做出决策、采取行动以完成目标的 AI 系统。与传统的"一问一答"式 LLM 不同，Agent 会主动规划和执行多步操作。

## 是什么

如果把 LLM 比作"大脑"，Agent 就是给大脑装上了**眼睛（感知）、手（工具调用）和计划表（规划与记忆）**。

- **感知（Perceive）**：从环境接收信息——可以是传感器数据、API 返回结果、用户输入
- **思考（Reason & Plan）**：分析当前状态，制定多步行动计划
- **行动（Act）**：调用工具执行——搜索网页、查询数据库、发邮件、运行代码

Agent 的四大核心特性：**自主性**（独立决策）、**适应性**（从反馈中学习）、**交互性**（与人/环境互动）、**功能性**（在特定领域执行任务）。

## 为什么重要

Agent 将 LLM 从"对话工具"升级为"任务执行者"：

- **能力跃迁**：从回答问题 → 完成复杂任务（如"帮我调研这三家公司的营收，做成表格发邮件给团队"）
- **自动化前沿**：2024-2025 年，Agent 被认为是 AI 应用的下一个趋势——OpenAI 的 Operator、Anthropic 的 Computer Use、以及各类 Coding Agent 都在探索 Agent 的边界
- **技术栈整合**：Agent 需要整合 LLM、工具调用、记忆系统、规划能力——它是 AI 工程化的集大成者

## 关键要点

- **核心架构**：LLM（大脑）+ Tools（手）+ Memory（记忆）+ Planning（计划）
- **决策模式**：ReAct（推理-行动循环）是主流范式；高级变体包括 Plan-and-Execute、Multi-Agent 协作
- **常见陷阱**：目标偏差（追错目标）、无限循环（重复无效行动）、工具滥用（调用不该调的工具）——需要 Guardrails 和 Human-in-the-loop
- **框架**：LangChain Agent、AutoGPT、CrewAI（多智能体）、Coze/Dify（低代码 Agent）

## 相关概念

- [[Function Calling]] — Agent 调用工具的基础能力，LLM 输出结构化 JSON 触发外部函数
- [[ReAct]] — 推理（Reasoning）+ 行动（Action）交替进行的 Agent 决策框架
- [[多智能体协作]] — 多个 Agent 分工协作解决复杂任务
- [[MCP 模型上下文协议]] — Anthropic 提出的 Agent-工具标准化协议

## 参考

- [Lilian Weng - LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)
- [Anthropic - Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)