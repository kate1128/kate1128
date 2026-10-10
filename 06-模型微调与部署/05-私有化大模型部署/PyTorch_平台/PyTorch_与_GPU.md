## PyTorch 的工作原理
PyTorch 和 TensorFlow 的相似之处在于，两者的核心组件都是张量和图形。

### 张量
[张量](https://pytorch.org/tutorials/beginner/blitz/tensor_tutorial.html#sphx-glr-beginner-blitz-tensor-tutorial-py)是一种核心 PyTorch 数据类型，类似于多维数组，用于存储和操作模型的输入和输出以及模型的参数。张量与 NumPy 的 ndarray 类似，只是张量可以在 GPU 上运行以加速计算。

### 图形
神经网络将一系列嵌套函数应用于输入参数，以转换输入数据。深度学习的目标是通过计算相对损失指标的偏导数（梯度），优化这些参数（包括权重和偏差，在 PyTorch 中以张量的形式存储）。在前向传播中，神经网络接受输入参数，并向下一层的节点输出置信度分数，直至到达输出层，在该层计算分数误差。在一个称为梯度下降的过程中，通过反向传播，误差会再次通过网络发送回来，并调整权重，从而改进模型。

[图形](https://developer.nvidia.com/discover/graph-analytics)是由已连接节点（称为顶点）和边缘组成的数据结构。每个现代深度学习框架都基于图形的概念，其中神经网络表示为计算的图形结构。PyTorch 在由函数对象组成的[有向无环图](https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html#sphx-glr-beginner-blitz-autograd-tutorial-py) (DAG) 中保存张量和执行操作的记录。在以下 DAG 中，叶是输入张量，根是输出张量。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736936517864-5e53f504-e551-4397-a81f-accc48e77cd6.png)

在许多热门框架（包括 TensorFlow）中，计算图形是一个静态对象。PyTorch 基于[动态计算图形](https://developer.nvidia.com/blog/recursive-neural-networks-pytorch/)，即，在运行时构建和重建计算图形，并使用与执行前向传递的计算相同的代码，同时还创建反向传播所需的数据结构。PyTorch 是首个运行时定义深度学习框架，与 TensorFlow 等静态图形框架的功能和性能相匹配，非常适合从标准卷积网络到时间递归神经网络等所有网络。

## PyTorch 的主要组件
1. **Tensor**
    - PyTorch 中的基本数据结构是 **Tensor**，类似于 NumPy 的数组，但其具有 GPU 加速能力。Tensors 是多维数组，可以在 CPU 或 GPU 上进行操作。
    - 示例：

```python
import torch

# 创建一个张量
tensor = torch.tensor([[1, 2], [3, 4]])
print(tensor)
```

2. **Autograd**
    - **Autograd** 自动计算梯度，是 PyTorch 中实现反向传播的关键。
    - 示例：

```python
import torch

x = torch.ones(2, 2, requires_grad=True)
y = x + 2
z = y * y * 3
out = z.mean()

# 反向传播
out.backward()

# 输出梯度
print(x.grad)
```

3. **模型（Model）**
    - PyTorch 中的模型通常是继承自 `torch.nn.Module` 的类，并且模型的所有层和操作都需要在 `forward()` 方法中定义。
    - 示例：

```python
import torch
import torch.nn as nn

class SimpleModel(nn.Module):
    def __init__(self):
        super(SimpleModel, self).__init__()
        self.fc1 = nn.Linear(2, 2)

    def forward(self, x):
        return self.fc1(x)

model = SimpleModel()
print(model)
```

4. **优化器（Optimizer）**
    - PyTorch 提供了各种优化器，如 **SGD**、**Adam** 等，用于优化模型的权重。
    - 示例：

```python
import torch.optim as optim

optimizer = optim.SGD(model.parameters(), lr=0.01)
```

5. **损失函数（Loss Function）**
    - PyTorch 提供了许多常见的损失函数，如交叉熵损失（`CrossEntropyLoss`）、均方误差（`MSELoss`）等。
    - 示例：

```python
criterion = nn.CrossEntropyLoss()
```

6. **训练循环（Training Loop）**
    - 在 PyTorch 中，训练模型通常包括前向传播、损失计算、反向传播和优化步骤。
    - 示例：

```python
for epoch in range(num_epochs):
    model.train()
    for inputs, labels in dataloader:
        optimizer.zero_grad()
        outputs = model(inputs)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
```

## PyTorch 用例
众所周知，PyTorch 框架十分便捷且灵活，增强学习、图像分类和自然语言处理等示例是比较常见的[用例](https://ngc.nvidia.com/catalog/collections?orderBy=scoreDESC&pageNumber=0&query=pytorch&quickFilter=collections&filters=)。

### PyTorch 发展趋势与应用
1. **研究领域**：
    - PyTorch 是深度学习研究中的首选框架，许多学术论文和新技术都是基于 PyTorch 实现的。尤其在自然语言处理（NLP）、计算机视觉（CV）等领域，PyTorch 凭借其易用性和灵活性成为了首选框架。
2. **工业应用**：
    - PyTorch 也在工业界得到了广泛应用，尤其在大规模生产环境中，结合 **TorchServe** 等工具，可以将 PyTorch 模型快速地部署到生产系统。
3. **支持量化与压缩**：
    - PyTorch 近年来加强了对模型量化、剪枝等技术的支持，尤其在模型部署到边缘设备或资源受限的环境中，能够显著提升模型的效率和响应速度。
4. **集成与服务**：
    - 随着 **PyTorch Lightning**、**TensorRT**、**ONNX** 等工具的出现，PyTorch 在生产环境中的集成能力逐渐增强。

### 商业、研究和教育示例
**自然语言处理 (NLP)：**从 Siri 到 Google Translate，深度[神经网络](https://developer.nvidia.com/discover/artificialneuralnetwork)在机器理解自然语言方面取得了突破性进展。其中大多数模型都将语言视为单词或字符的平面序列，并使用一种称为[时间递归神经网络](https://developer.nvidia.com/discover/recurrentneuralnetwork) (RNN) 的模型处理该序列。但是，许多语言学家认为，语言极易理解为一个由短语组成的层次树，因此，大量研究已经进入称为[递归神经网络](https://developer.nvidia.com/blog/recursive-neural-networks-pytorch/)的[深度学习](https://developer.nvidia.com/deep-learning)模型，该模型将这种结构考虑在内。虽然这些模型难以实现且运行效率低下，但 [PyTorch](http://pytorch.org/) 使这些模型和其他复杂自然语言处理变得容易得多。Salesforce 正在[使用 PyTorch](https://pytorch.org/) 进行 NLP 和多任务学习。

+ **研究：**PyTorch 具有易用性、灵活性和快速原型设计，是研究的首选。斯坦福大学正在[利用 PyTorch](https://pytorch.org/)的灵活性高效研究新的算法方法。
+ **教育：**[Udacity](https://www.udacity.com/course/deep-learning-pytorch--ud188) 正在使用 PyTorch 培养 AI 创新者。

**PyTorch 的重要意义……**

+ **数据科学家**
对于熟悉 Python 的程序员而言，PyTorch 学习起来相对容易。它提供了简单的调试、简单的 API，并且兼容各种内置 Python 的扩展。其动态执行模型也非常适合原型设计，尽管会产生一些性能开销。
+ **软件开发者**
PyTorch 支持各种功能，可以快速轻松地部署 AI 模型。它还具有丰富的库生态系统，如 Captum（用于模型可解释性）、skorch（scikit-learn 兼容性）等，以支持开发。PyTorch 具有出色的加速器生态系统，例如，[Glow](https://github.com/pytorch/glow)（用于训练）和 [NVIDIA®](https://developer.nvidia.com/tensorrt)[TensorRT™](https://developer.nvidia.com/tensorrt)（用于推理）。

## GPU：深度学习的关键
在架构方面，CPU 仅由几个具有大缓存内存的核心组成，一次只可以处理几个软件线程。相比之下，[GPU 由数百个核心组成](https://blogs.nvidia.com/blog/2009/12/16/whats-the-difference-between-a-cpu-and-a-gpu/)，可以同时处理数千个线程。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736936517913-32eca237-f107-432e-a0e8-729722192b18.png)

先进的深度学习神经网络可能有数百万乃至十亿以上的参数需要通过反向传播进行调整。由于神经网络由大量相同的神经元构建而成，因此本质上具有高度并行性。这种并行性会自然地映射到 NGC (NVIDIA GPU Cloud)，用户可以在其中提取容器，这些容器具有可用于各种任务（例如计算机视觉、自然语言处理等）的预训练模型，且所有依赖项和框架位于一个容器中。借助 NVIDA 的 [TensorRT](https://developer.nvidia.com/tensorrt)，使用 NVIDIA GPU 时，可以在 PyTorch 上显著提高推理性能。

## 面向开发者的 NVIDIA 深度学习
GPU 加速深度学习框架能够为设计和训练自定义深度神经网络带来灵活性，并为 Python 和 C/C++ 等常用编程语言提供编程接口。MXNet、PyTorch、TensorFlow 等广泛使用的[深度学习框架](https://developer.nvidia.com/deep-learning-frameworks)依赖于 NVIDIA GPU 加速库，能够提供高性能的多 GPU 加速训练。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736936518410-69349e14-e741-483b-8831-02a17ca9ee2b.png)

## NVIDIA GPU 加速的端到端数据科学
基于 [CUDA-X AI](https://developer.nvidia.com/machine-learning) 创建的 NVIDIA [RAPIDS™](https://developer.nvidia.com/rapids) 开源软件库套件使您完全能够在 GPU 上执行端到端数据科学和分析流程。此套件依靠 NVIDIA CUDA 基元进行低级别计算优化，但通过用户友好型 Python 接口实现了 GPU 并行化和高带宽显存速度。

借助 RAPIDS GPU DataFrame，数据可以通过一个类似 Pandas 的接口加载到 GPU 上，然后用于各种连接的机器学习和图形分析算法，而无需离开 GPU。这种级别的互操作性是通过 Apache Arrow 这样的库实现的。这可加速端到端流程（从数据准备到机器学习，再到深度学习）。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736936518073-47cc087a-b901-4017-b8fd-2e140f180e19.png)

RAPIDS 支持在许多热门数据科学库之间共享设备内存。这样可将数据保留在 GPU 上，并省去了来回复制主机内存的高昂成本。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736936518312-ec13d34d-bace-42f8-9441-315333ad71fb.png)

## 后续步骤
**请阅读以下博客：**

+ 与 PyTorch 结合使用 RAPIDS。使用 GPU 进行 ETL 和预处理…… | 作者：Even Oldridge | RAPIDS AI
+ [使用 PyTorch 的递归神经网络](https://developer.nvidia.com/blog/recursive-neural-networks-pytorch/)
+ PyTorch 深度学习框架：速度 + 易用性 | 作者：Synced | SyncedReview
+ 使用 Rapids、FastAI 和 Pytorch 将深度学习推荐系统加速 15 倍

**了解：**

+ 开发者、研究人员和数据科学家可以通过 [PyTorch 示例](https://ngc.nvidia.com/catalog/collections?orderBy=scoreDESC&pageNumber=0&query=pytorch&quickFilter=collections&filters=)轻松访问 NVIDIA 优化深度学习框架容器化，这些示例针对 NVIDIA GPU 进行了性能调整和测试。这能够消除对软件包和依赖项的管理需要，或根据源头构建深度学习框架的需要。请访问 [NVIDIA NGC](https://www.nvidia.cn/gpu-cloud/) 了解详情并开始使用。
+ [NVIDIA NeMo](https://developer.nvidia.com/nvidia-nemo) 是一个开源工具包，具有 PyTorch 后端，可进一步推动抽象概念。借助 NeMo，您可以使用三行代码快速编写和训练复杂的神经网络架构。NeMo 还附带有适用于 [ASR](https://catalog.ngc.nvidia.com/orgs/nvidia/models/nemospeechmodels#cid=dl13_partn_en-us)、[NLP](https://catalog.ngc.nvidia.com/orgs/nvidia/models/nemonlpmodels#cid=dl13_partn_en-us) 和 [TTS](https://catalog.ngc.nvidia.com/orgs/nvidia/models/nemottsmodels#cid=dl13_partn_en-us) 的可扩展模型集合。这些集合提供了轻松构建先进网络架构（例如 QuartzNet、BERT、Tacotron 2 和 WaveGlow）的方法。借助 NeMo，您还可以通过随时可用的 API 自动从 [NVIDIA NGC](https://catalog.ngc.nvidia.com/) 下载和实例化这些模型，在自定义数据集上微调这些模型。
+ NVIDIA 提供经过优化的软件堆栈，可加速深度学习工作流程的训练和推理阶段。如需详细了解相关信息，请访问 [NVIDIA 深度学习主页](https://developer.nvidia.com/deep-learning)。
+ NVIDIA Volta<sup>™</sup> 和 Turing<sup>™</sup> GPU 上的 [Tensor Core](https://developer.nvidia.com/tensor_cores) 专门为深度学习而设计，能够显著提高训练和推理性能。了解有关获取[参考实现](https://developer.nvidia.com/deep-learning-examples)的更多内容。
+ [NVIDIA 深度学习培训中心](https://www.nvidia.cn/deep-learning-ai/education/?ncid=so-dis-dldlwsd1-72342) (DLI) 能够为开发者、数据科学家和研究人员提供有关 AI 和加速计算的实操培训。
+ 请查看[深度学习简介](https://developer.nvidia.com/blog/deep-learning-nutshell-core-concepts/)，详细了解有关深度学习的更多深入技术探讨。
+ 如需了解开发者资讯和资源，请访问 NVIDIA [开发者网站](https://developer.nvidia.com/)。

## 参考
[https://www.nvidia.cn/glossary/pytorch/](https://www.nvidia.cn/glossary/pytorch/)
