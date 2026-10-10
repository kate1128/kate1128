## 参考
1. 官网地址：[https://pytorch.org/](https://pytorch.org/)
2. 中文网：[https://pytorch.p2hp.com/](https://pytorch.p2hp.com/)
3. 学习资料：[https://pytorch.org/resources/](https://pytorch.org/resources/)
4. 文档：[https://pytorch.org/docs/stable/index.html](https://pytorch.org/docs/stable/index.html)
5. 中文教程：[https://pytorch.apachecn.org/2.0/tutorials/recipes/recipes_index/](https://pytorch.apachecn.org/2.0/tutorials/recipes/recipes_index/)
6. github：[https://github.com/pytorch/pytorch](https://github.com/pytorch/pytorch)

<font style="color:rgb(0, 0, 0);">PyTorch 是一种开源深度学习框架，以出色的灵活性和易用性著称。这在一定程度上是因为与机器学习开发者和数据科学家所青睐的热门 Python 高级编程语言兼容。</font>

## <font style="color:rgb(0, 0, 0);"> 什么是 PyTorch？</font>
<font style="color:rgb(0, 0, 0);">PyTorch 是一种用于构建深度学习模型的功能完备框架，是一种通常用于图像识别和语言处理等应用程序的机器学习。</font>

<font style="color:rgb(0, 0, 0);">使用 Python 编写，因此对于大多数机器学习开发者而言，学习和使用起来相对简单。PyTorch 的独特之处在于，它完全支持 GPU，并且使用</font>[A simple explanation of reverse-mode automatic differentiation](https://justindomke.wordpress.com/2009/03/24/a-simple-explanation-of-reverse-mode-automatic-differentiation/)<font style="color:rgb(0, 0, 0);">技术，因此可以动态修改计算图形。这使其成为快速实验和原型设计的常用选择。</font>

## <font style="color:rgb(0, 0, 0);">为何选择 PyTorch？</font>
<font style="color:rgb(0, 0, 0);">PyTorch 是 Facebook AI Research 和其他几个实验室的开发者的工作成果。该框架将</font><font style="color:rgb(0, 0, 0);"> </font>[<font style="color:rgb(0, 0, 0);">Torch</font>](http://torch.ch/)<font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">中高效而灵活的 GPU 加速后端库与直观的 Python 前端相结合，后者专注于快速原型设计、可读代码，并支持尽可能广泛的深度学习模型。</font>

<font style="color:rgb(0, 0, 0);">Pytorch 支持开发者使用熟悉的命令式编程方法，但仍可以输出到图形。它于 2017 年以开源形式发布，其 Python 根源使其深受机器学习开发者的喜爱。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736936517924-4c7d61ad-d0a3-4fce-aaf6-70d143fea056.png)

<font style="color:rgb(0, 0, 0);">图像引用</font><font style="color:rgb(0, 0, 0);"> </font>[<font style="color:rgb(0, 0, 0);">https://pytorch.org/features/</font>](https://pytorch.org/features/)

<font style="color:rgb(0, 0, 0);">值得注意的是，PyTorch 采用了</font><font style="color:rgb(0, 0, 0);"> </font>[<font style="color:rgb(0, 0, 0);">Chainer</font>](https://chainer.org/)<font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">创新技术，称为</font>[<font style="color:rgb(0, 0, 0);">反向模式自动微分</font>](https://justindomke.wordpress.com/2009/03/24/a-simple-explanation-of-reverse-mode-automatic-differentiation/)<font style="color:rgb(0, 0, 0);">。从本质上讲，它就像一台磁带录音机，录制完成的操作，然后回放，计算梯度。这使得 PyTorch 的调试相对简单，并且能够很好地适应某些应用程序，例如动态神经网络。由于每次迭代可能都不相同，因此非常适用于原型设计。</font>

<font style="color:rgb(0, 0, 0);">PyTorch 在 Python 开发者中特别受欢迎，因为它使用 Python 编写，并使用该语言的命令式、运行时定义即时执行模式，在这种模式下，从 Python 调用运算时执行运算。随着 Python 编程语言的广泛采用，一项</font>[<font style="color:rgb(0, 0, 0);">调查显示</font>](https://www.datanami.com/2021/01/26/python-popularity-persists-ai-drives-pytorch/)<font style="color:rgb(0, 0, 0);">，AI 和机器学习任务受到越来越多的关注，并且相关 PyTorch 的采用也随之提升。这使得 PyTorch 对于刚接触深度学习的 Python 开发者来说是一个很好的选择，而且越来越多的深度学习课程基于 PyTorch。从早期版本开始，API 一直保持一致，这意味着代码对于经验丰富的 Python 开发者来说相对容易理解。</font>

<font style="color:rgb(0, 0, 0);">PyTorch 的独特优势是快速原型设计和小型项目。其易用性和灵活性也使其深受学术和研究界的喜爱。</font>

<font style="color:rgb(0, 0, 0);">Facebook 开发者一直努力改进 PyTorch 的高效应用。新版本已提供增强功能，例如支持谷歌的 TensorBoard 可视化工具以及即时编译。此外，还扩展了对</font><font style="color:rgb(0, 0, 0);"> </font>[<font style="color:rgb(0, 0, 0);">ONNX</font>](https://onnx.ai/)<font style="color:rgb(0, 0, 0);">（开放神经网络交换）的支持，使开发者能够匹配适合其应用程序的深度学习框架或运行时。</font>

## <font style="color:rgb(0, 0, 0);">PyTorch 的主要优势</font>
<font style="color:rgb(0, 0, 0);">PyTorch 的一些重要特性包括：</font>

+ [<font style="color:rgb(0, 0, 0);">PyTorch.org</font>](https://pytorch.org/)<font style="color:rgb(0, 0, 0);"> 有一个充满活力的大型社区，具有优秀的文档和教程。论坛十分活跃，并能给予帮助和支持。</font>
+ <font style="color:rgb(0, 0, 0);">采用 Python 编写，并集成了热门的 Python 库，例如用于科学计算的</font><font style="color:rgb(0, 0, 0);"> </font>[<font style="color:rgb(0, 0, 0);">NumPy</font>](https://numpy.org/)<font style="color:rgb(0, 0, 0);">、SciPy 和用于将 Python 编译为 C 以提高性能的 Cython。由于 PyTorch 的语法和用法类似于 Python，因此对于 Python 开发者来说，学习起来相对容易。</font>
+ <font style="color:rgb(0, 0, 0);">受主要云平台的有力支持。</font>
+ <font style="color:rgb(0, 0, 0);">脚本语言（称为 TorchScript）在即时模式下易于使用且灵活。这是一种快速启动执行模式，从 Python 调用运算时立即执行运算，但也可以在 C++ 运行时环境中转换为图形模型，以提高速度和实现优化。</font>
+ <font style="color:rgb(0, 0, 0);">它支持 CPU、GPU、并行处理以及分布式训练。这意味着计算工作可以在多个 CPU 和 GPU 核心之间分配，并且可以在多台机器上的多个 GPU 上进行训练。</font>
+ <font style="color:rgb(0, 0, 0);">PyTorch 支持动态计算图形，能够在运行时更改网络行为。与大多数机器学习框架相比，提供了更大的灵活性优势，因为大多数机器学习框架要求在运行时之前将神经网络定义为静态对象。</font>
+ [<font style="color:rgb(0, 0, 0);">PyTorch Hub</font>](https://pytorch.org/hub/)<font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">是一个预训练模型库，在某些情况下只需使用一行代码就可以调用。</font>
+ <font style="color:rgb(0, 0, 0);">新自定义组件可创建为标准 Python 类的子类，可以通过 TensorBoard 等外部工具包轻松共享参数，并且可以轻松导入和内联使用库。</font>
+ <font style="color:rgb(0, 0, 0);">PyTorch 拥有一组备受好评的 API，可用于扩展核心功能。</font>
+ <font style="color:rgb(0, 0, 0);">既支持用于实验的“即时模式”，也支持用于高性能执行的“图形模式”。</font>
+ <font style="color:rgb(0, 0, 0);">拥有从计算机视觉到增强学习等领域的大量工具和库。</font>
+ <font style="color:rgb(0, 0, 0);">支持 Python 程序员熟悉的纯 C++ 前端接口，可用于构建高性能 C++ 应用程序。</font>

### PyTorch 平台的优势
1. **灵活性和易用性**：
    - 由于 PyTorch 是一个动态计算图框架，它在构建和调试模型时更加直观和灵活。
    - 动态计算图允许在训练过程中修改模型的结构，这对于研究和实验尤其重要。
2. **深度学习与科学计算的集成**：
    - PyTorch 与 **NumPy** 紧密集成，可以轻松进行张量运算，而 PyTorch 中的 `torch.Tensor` 对象几乎可以做到和 `numpy.ndarray` 一样的功能，并且支持 GPU 加速。
3. **强大的社区支持**：
    - PyTorch 拥有一个活跃的开源社区和广泛的第三方库支持，许多最新的研究和预训练模型都首先在 PyTorch 上实现。
4. **跨平台支持**：
    - PyTorch 可以在 Windows、Linux 和 macOS 上运行，并且支持各种硬件（包括 CPU 和 GPU）。
5. **高效的 GPU 支持**：
    - PyTorch 通过 `torch.cuda` 可以非常方便地将模型和数据迁移到 GPU 上进行加速。

