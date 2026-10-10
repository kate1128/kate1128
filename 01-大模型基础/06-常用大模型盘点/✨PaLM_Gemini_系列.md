<!-- type: glossary -->
# PaLM / Gemini 系列（Google）

> Google 开发的**大语言模型系列**，PaLM 是前代，Gemini 是当前主力。Gemini 以**多模态能力**和**原生 1M+ 上下文窗口**为核心差异化。

## 基本情况

- **开发者**：Google DeepMind
- **演进路线**：PaLM（2022.04）→ PaLM 2（2023.05）→ Gemini 1.0（2023.12）→ Gemini 1.5（2024.02）→ Gemini 2.0（2024.12+）
- **使用地址**：[gemini.google.com](https://gemini.google.com/app)

## Gemini 模型矩阵

| 版本 | 定位 | 典型场景 |
| --- | --- | --- |
| Gemini Ultra | 最强推理 | 复杂科学推理、数学证明、专业编程 |
| Gemini Pro | 平衡（主力） | 日常对话、内容创作、代码辅助 |
| Gemini Nano | 端侧 | Pixel 手机本地运行 |

## 核心差异化

- **原生多模态**：不是"文本模型 + 外挂视觉模块"，而是从一开始就用文本、图像、音频、视频联合训练
- **超长上下文**：Gemini 1.5 Pro 支持 **100 万 token** 上下文（约 75 万词），可直接分析整本书或长视频
- **Google 生态集成**：与 Google Search、Gmail、Google Docs 等原生集成
- **TPU 训练**：用 Google 自研 TPU 而非 NVIDIA GPU 训练

## 使用限制

- **国内访问需要科学上网**
- **Gemini API 免费额度较慷慨**（适合个人开发者）
- **中文能力弱于文心/通义/DeepSeek**——它是全球化产品，不是针对中文优化的

## 相关笔记

- [[GPT 系列]] — Gemini 的主要竞品
- [[多模态大模型](../../05-多模态大模型/多模态大模型.md)] — 多模态技术原理
- [[模型选型指南](../../../07-LLM应用开发/01-LLM应用开发/对话型应用的设计/第一步_模型选型.md)]

## 参考

- [Google Gemini](https://deepmind.google/technologies/gemini/)
- [Gemini 1.5 Technical Report](https://arxiv.org/abs/2403.05530)