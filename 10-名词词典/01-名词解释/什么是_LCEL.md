<!-- type: glossary -->
# LCEL（LangChain Expression Language）

> LangChain 提供的一种**声明式管道语法**，用 `|` 运算符将 Prompt、LLM、输出解析器等组件串联成可执行的链（Chain）。相比原来的 Chain 类代码量大幅减少。

## 是什么

在 LCEL 出现之前，用 LangChain 串联多个组件非常繁琐——你需要创建各种 Chain 子类、手动管理中间数据的传递。LCEL 引入了类似 Unix 管道的语法：

```python
# 使用 LCEL 创建链
chain = prompt | llm | output_parser

# 相当于：
# 1. 把用户输入填进 prompt 模板
# 2. 把填好的 prompt 发送给 LLM
# 3. 把 LLM 的输出送进 output_parser 解析
result = chain.invoke({"topic": "Python"})
```

每个 `|` 左边的输出会作为下一个组件的主要输入。这种语法简洁、可读，而且原生支持异步、流式和批处理。

## 为什么重要

- **声明式 > 命令式**：代码写"你想做什么"而不是"怎么一步步做"——减少样板代码、降低出错概率
- **自动获得高级能力**：LCEL 链自带 `.invoke()`、`.stream()`、`.batch()`、`.ainvoke()` 等方法，不需要额外配置
- **LangChain 的战略方向**：2024 年起 LCEL 已经是 LangChain 推荐的标准用法，旧的 Chain 类正在逐步被取代

## 关键要点

- **核心语法**：`chain = component1 | component2 | component3`
- **Runnable 协议**：LangChain 的所有组件都实现了相同的 Runnable 接口——这意味着你可以自由组合 Prompt、ChatModel、Retriever、OutputParser 等任何组件
- **常见模式**：
  - `prompt | llm | str_parser`：最简单的链
  - `{"context": retriever, "question": input} | prompt | llm`：带检索的 RAG 链
  - `chain1 | chain2`：链的嵌套组合

## 相关概念

- [[LangChain]] — LCEL 所属的框架
- [[Prompt 工程]] — Prompt 组件是 LCEL 链条中的一环
- [[Runnable 协议]] — LCEL 的组件接口标准

## 参考

- [LangChain LCEL 文档](https://python.langchain.com/docs/expression_language/)
- 本知识库: [05-主流框架与Agent/01-LangChain](../../05-主流框架与Agent/01-LangChain/)