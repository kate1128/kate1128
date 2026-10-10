<!-- type: glossary -->
# PyTorch

> 由 Meta（Facebook）开发的**开源深度学习框架**，以动态计算图和 Pythonic API 著称。目前是 AI 研究和 LLM 开发领域使用最广泛的框架。

## 是什么

PyTorch 和 TensorFlow 是深度学习领域的两大框架。PyTorch 的核心优势在于：

- **动态计算图（Define-by-Run）**：计算图随代码执行动态构建，可以像写普通 Python 一样写模型，调试时直接加 `pdb` 断点
- **Python 原生风格**：API 设计与 NumPy 高度一致，学习曲线平滑
- **自动微分引擎（Autograd）**：自动计算梯度，反向传播全自动化

简单的 PyTorch 训练循环：
```python
for epoch in range(epochs):
    # 前向计算
    output = model(input_data)
    loss = loss_fn(output, target)
    # 反向传播
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

## 为什么重要

- **LLM 的土壤**：Transformer、Llama、Mistral——几乎所有现代 LLM 的模型代码都是 PyTorch 实现的
- **学术标准**：90% 以上的 AI/ML 顶会论文用 PyTorch 实现
- **生态完善**：Hugging Face Transformers、PyTorch Lightning、TorchServe、TorchVision/Audio/Text 等生态工具覆盖全流程

## 关键要点

- **PyTorch vs TensorFlow**：2024 年，PyTorch 在研究和开源领域占绝对主导。TensorFlow 仍存在于部分生产环境和 Google 内部
- **PyTorch Lightning**：PyTorch 的高级封装，减少训练循环样板代码——适合快速实验
- **TorchScript / TorchDynamo**：PyTorch 的静态图/编译方案，用于生产部署加速
- **与 LLM 的关系**：你用 Hugging Face 加载一个 LLaMA 模型，底层跑的就是 PyTorch

## 相关概念

- [[深度学习]] — PyTorch 所服务的领域
- [[HuggingFace]] — 基于 PyTorch 的模型生态，LLM 使用的主要入口
- [[TensorFlow]] — PyTorch 的主要竞品
- [[LoRA 微调]] — 常见的 LLM 微调方法，通过 PEFT 库在 PyTorch 上实现

## 参考

- [PyTorch 官方文档](https://pytorch.org/docs/)
- [PyTorch 入门教程](https://pytorch.org/tutorials/)