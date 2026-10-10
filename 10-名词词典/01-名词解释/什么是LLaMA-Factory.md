<!-- type: glossary -->
# LLaMA-Factory

> 一个开源的**LLM 微调框架**，支持对 100+ 种大模型进行高效微调（全参数/LoRA/QLoRA），提供 Web UI 和命令行两种操作方式。是目前中文社区最流行的 LLM 微调工具之一。

## 是什么

LLaMA-Factory 解决了一个核心痛点：**把 LLM 微调从"需要深入理解分布式训练和 CUDA"降级到"选个模型、选个数据集、点一下训练"**。

核心能力：
- **多模型支持**：LLaMA、Mistral、Falcon、ChatGLM、Qwen、Baichuan、DeepSeek 等 100+ 模型
- **多微调方法**：Full Fine-tuning（全参数）、LoRA/QLoRA（参数高效）、Freeze（冻结部分层）
- **训练特性**：混合精度训练、梯度累积、Flash Attention、DeepSpeed 加速、多卡分布式训练
- **Web UI**：通过 Gradio 提供可视化界面，不写代码也能完成微调

## 为什么重要

- **微调民主化**：以前微调 LLM 需要深厚工程能力，现在 LLaMA-Factory 让中级开发者也能做
- **生产力工具**：大大缩短"数据 → 微调 → 评估"的迭代周期
- **集成生态**：与 Hugging Face Transformers、PEFT、DeepSpeed 等无缝对接

## 关键要点

- **GitHub Stars 快速增长**（截至 2024 年已超 20k），中文社区活跃度很高
- **核心工作流**：准备数据（JSON/CSV）→ 选择模型和方法 → 配置参数 → 训练 → 评估 → 导出/部署
- **LoRA 是推荐起点**：全参数微调需要大量显存（如 LLaMA-7B 全参数约需 60GB+），LoRA 在消费级 GPU（24GB）上就能跑
- **与 Hugging Face 的关系**：底层使用 Transformers 和 PEFT 库，LLaMA-Factory 做的是"包装和易用化"

## 相关概念

- [[Fine-tuning 微调]] — LLaMA-Factory 解决的核心问题
- [[PEFT 参数高效微调]] — LoRA/QLoRA 等高效方法的统称
- [[LoRA 低秩适应]] — LLaMA-Factory 最推荐的微调方法
- [[HuggingFace]] — LLaMA-Factory 底层依赖的模型生态

## 参考

- [LLaMA-Factory GitHub](https://github.com/hiyouga/LLaMA-Factory)
- 本知识库: [06-模型微调与部署/03-LlamaFactory微调框架](../../06-模型微调与部署/03-LlamaFactory微调框架/)