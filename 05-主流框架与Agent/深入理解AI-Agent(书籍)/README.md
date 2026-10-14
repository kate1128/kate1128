<!-- type: resource-index -->
# 《深入理解 AI Agent：设计原理与工程实践》

> **来源**：[bojieli/ai-agent-book](https://github.com/bojieli/ai-agent-book) — Apache 2.0 开源协议
> **核心公式**：Agent = LLM + 上下文 + 工具
> **本书定位**：从原理到工程实战，系统化学习 AI Agent 的完整指南

---

## 目录

| 章节 | 标题 | 核心内容 |
|:---:|------|------|
| 00 | [引言](00-引言.md) | 本书概览与学习方法 |
| 01 | [AI Agent 入门](01-AI-Agent入门.md) | Agent 的定义、核心公式、发展简史 |
| 02 | [上下文工程](02-上下文工程.md) | Prompt、System Message、上下文窗口管理 |
| 03 | [用户记忆和知识库](03-用户记忆和知识库.md) | Memory、RAG、长期记忆架构 |
| 04 | [工具](04-工具.md) | Function Calling、Tool Use、MCP 协议 |
| 05 | [Coding Agent 与通用 Agent](05-Coding-Agent与通用Agent.md) | 代码生成、多步执行、通用 Agent 架构 |
| 06 | [交互：观察与动作空间](06-交互：观察与动作空间的扩展.md) | 环境感知、动作执行、反馈循环 |
| 07 | [Agent 的评估](07-Agent的评估.md) | 评测指标、Benchmark、红队测试 |
| 08 | [模型后训练](08-模型后训练.md) | Fine-tuning、RLHF、对齐训练 |
| 09 | [Agent 的持续进化](09-Agent的持续进化.md) | 在线学习、反馈驱动改进 |
| 10 | [多 Agent 协作](10-多Agent协作.md) | 多智能体架构、协作模式、通信协议 |
| 11 | [后记](11-后记.md) | 总结与展望 |
| 12 | [思考题参考答案](12-思考题参考答案.md) | 各章节思考题的参考解答 |

---

## 配套资源

- 📖 **在线阅读**：[bojieli.github.io/ai-agent-book](https://bojieli.github.io/ai-agent-book/)
- 📥 **PDF/EPUB 下载**：[GitHub Releases](https://github.com/bojieli/ai-agent-book/releases)
- 🧪 **103 个配套实验**：见原仓库 `experiments/` 目录

## 与本知识库的关系

- 本书提供了 Agent 的**系统化理论基础**
- `04-Agent智能体/` 目录下的笔记提供了**具体框架的实践代码**（LangChain Agent、Function Calling 等）
- `06-多智能体协作/` 与第 10 章互补
- `07-MCP模型上下文协议/` 与第 4 章的工具部分互补

> 建议阅读顺序：先读本书建立系统性认知 → 再读散落的实践笔记补充代码细节。