<!-- type: glossary -->
# GPTs（ChatGPT 自定义版本）

> OpenAI 推出的一项功能，允许 ChatGPT Plus/Team/Enterprise 用户**无需编写代码**，通过自然语言对话和上传资料，创建定制化的 AI 助手（GPT）。

## 是什么

GPTs 的本质是"预配置的 ChatGPT"——你把 System Prompt、知识文档、可调用的工具打包在一起，得到一个专属于特定场景的 AI 助手。

创建过程：
1. 在 ChatGPT 界面点击"创建 GPT"
2. 用自然语言描述你想要的角色和行为（如"你是一个 Python 代码审查专家，用中文回复，给出改进建议"）
3. 上传知识文档（PDF、Markdown 等）作为参考库
4. 配置能力：网页浏览、DALL-E 图片生成、代码解释器
5. 发布——可以私有、分享链接、或上架 GPT Store

## 为什么重要

- **个性化 AI 的民主化**：不需要会编程就能创建 AI 助手——这是 OpenAI 的战略方向，把 ChatGPT 从"产品"变成"平台"
- **GPT Store 生态**：OpenAI 推出了类似 App Store 的 GPT Store，优秀的 GPT 可以产生收入
- **知识工作者的增强工具**：HR 可以创建面试 GPT，法务可以创建合规问答 GPT，教师可以创建学科辅导 GPT
- **与 RAG 的对比**：GPTs 本质上是 OpenAI 做了 RAG 的全托管——用户上传文档，OpenAI 负责 Embedding、检索、生成的全流程

## 关键要点

- **需要 ChatGPT Plus 订阅**（$20/月），免费用户不能创建
- **Actions 能力**：高级 GPT 可以通过 Actions 调用外部 API——这是 Function Calling 的 GUI 版
- **局限**：文档有大小限制、不能精确控制检索策略、无法自定义 Embedding 模型——适合轻量级场景，生产级 RAG 还是需要自建
- **API 调试技巧**：先跑通 GPTs 验证效果，再通过 API 实现。GPTs 本质上是用 `gpt-4` + `retrieval` tool + 自定义 instructions 组成的

## 相关概念

- [[RAG 检索增强生成]] — GPTs 的知识库功能是托管版 RAG
- [[System Prompt]] — GPTs 的 instructions 就是 System Prompt
- [[Function Calling]] — GPTs 的 Actions 底层使用 Function Calling
- [[Coze / Dify]] — 类似的"零代码 AI 助手构建平台"，功能比 GPTs 更灵活

## 参考

- [OpenAI - Introducing GPTs](https://openai.com/blog/introducing-gpts)
- [OpenAI GPT Store](https://chatgpt.com/gpts)