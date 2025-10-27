[第1章 初识python笔记.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1753093414047-b5821b44-9ea7-4d4c-857d-9a722b07a9b3.pdf)

#  第1节 python介绍
## 计算机
按层次来划分计算机系统的话，我们可以划分成七个层次。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/665aa2e0310114de3f18bd9c0fe20dcac8fe4d19eba629540ecd2bc13162ba92.jpg)

1. 硬件逻辑层：主要由门电路、触发器等逻辑电路组成，属于电子工程的领域，这里就不展开介绍了。
2. 微程序机器层：编程语言主要是微指令集，微指令所组成的微程序直接交由硬件执行，主要是由生产硬件的公司的程序员来编写的。
3. 传统机器层：编程语言主要是CPU指令集（机器指令），和硬件是直接相关的，程序员所用机器指令编写的程序可以交由微程序直接进行解析，而这里提到的指令集，存储在CPU内部，对CPU的运算进行指导和优化，拥有指令集，CPU就可以有效地运行。我们知道，CPU的制造商分为AMD和Intel两大阵营，那么这两大厂商生产的CPU最大的区别是——指令集不同，Intel的CPU所使用的指令集不适合AMD的CPU，同样的，AMD的CPU所使用的指令集也不适合Intel。除了不同厂商以外，同一个厂商也可以生产不同指令集的CPU，即不同架构的CPU使用不同的CPU指令集。
4. 操作系统层：操作系统，一方面，向上提供了简易的操作界面，使得用户能够容易地操作计算机；同时，向下对接了指令系统，管理硬件资源。操作系统对用户程序所使用机器的各种资源进行管理和分配，包括CPU、存储器等等，比如说，当一个用户程序需要运行的时候，首先由操作系统将其加载到内存中，这就需要操作系统首先为其分配内存空间来进行存储。再比如说，某一个程序需要使用某一个输出设备进行结果输出的时候，需要操作系统为其提供该设备的控制权。由此可见，操作系统是在软件和硬件之间的适配层。
5. 汇编语言层：编程语言是汇编语言，汇编语言可以翻译成可直接执行的机器语言，完成这个翻译过程的程序就是汇编器。从这一层开始，它们所使用的编程语言就是人类比较容易理解的语言了。
6. 高级语言层：编程语言就是为广大程序员所接受的高级语言，种类非常多，有几百种，常见的编程语言有Python、Java、C/C++、Golang等。
7. 应用层：计算机针对某种用途而设计的应用，像Word、Excel等。

## 程序翻译与程序解释
程序翻译与程序解释计算机是无法直接理解人类语言的，它只认识01010101...这样的比特位，因此，我们需要进行程序翻译或程序解析，把人类语言翻译或解析成计算机所能理解的语言。

## 程序
为了完成某种特定功能，以某种程序设计语言编写的有序指令的集合。

程序是指挥cpu工作的“工作手册”。计算机只能执行二进制代码，程序设计语言一般类似英文，想要让计算机理解你写的程序，必须把程序代码“翻译”成计算机能理解的二进制代码，根据翻译形式的不同，可以分为：

+ 编译：将程序代码翻译成计算机能理解的二进制目标代码，会生成特定的可执行代码（在window上是exe文件），可执行代码是二进制的，无法看到源代码。然后执行可执行代码就可以得到想要的结果
    - C/C++、oc 等，<font style="color:#DF2A3F;">go 也是</font>
+ 解释：将程序代码一句一句翻译为计算机可以执行的指令，立即执行，不会生成可执行文件
    - python、php、JavaScript等

## 语言的区别
### 解释型语言
在运行的时候将程序翻译成机器语言，所以运行速度相对于编译型语言要慢。比如PHP、Python

+ 优点：可移植性较好，只要有解释环境，可在不同的操作系统上运行
+ 缺点：运行需要解释环境，运行起来比编译的要慢，占用资源也要多一些，代码效率低，代码修改后就可运行，不需要编译过程

### 编译型语言
在程序执行之前，有一个单独的编译过程，将程序翻译成机器语言，以后执行这个程序的时候，就不用再进行翻译了。比如C、C++、Java、<font style="color:#DF2A3F;">go</font>

+ 优点：运行速度快，代码效率高，编译后的程序不可修改，保密性较好
+ 缺点：代码需要经过编译方可运行，可移植性差，只能在兼容的操作系统上运行

## python排名
python排名TIOBE排行榜中C和Java一直占据着前两位，近20年来没有哪个语言可以撼动它们两的地位，直到这几年Python发展越来越快，市场占有率一直在提升，在2021年时排名第一。

## python应用
+ web后端开发
+ 网络爬虫
+ 人工智能
+ 自动化运维
+ 网络编程
+ 国内：豆瓣、百度、阿里、新浪等；
+ 国外：Google、FaceBook、Twitter

## python起源
第一个公开发行版发行于1991年。

设计目标：一门简单直观的语言并与主要竞争者一样强大开源，以便任何人都可以为它做贡献代码像纯英语那样容易理解适用于短期开发的日常任务

这些想法中的基本都已经成为现实，Python已经成为一门流行的编程语言。

## python之禅
+ Python 开发者的哲学是：用一种方法，最好是只有一种方法来做一件事
+ 如果面临多种选择，Python 开发者一般会拒绝花俏的语法，而选择明确没有或者很少有歧义的语法
+ 优美胜于丑陋（Python 以编写优美的代码为目标）
+ 明了胜于晦涩（优美的代码应当是明了的，命名规范，风格相似）
+ 简洁胜于复杂（优美的代码应当是简洁的，不要有复杂的内部实现）
+ 复杂胜于凌乱（如果复杂不可避免，那代码间也不能有难懂的关系，要保持接口简洁）
+ 扁平胜于嵌套（优美的代码应当是扁平的，不能有太多的嵌套）
+ 间隔胜于紧凑（优美的代码有适当的间隔，不要奢望一行代码解决问题）
+ 可读性很重要（优美的代码是可读的）
+ 即便假借特例的实用性之名，也不可违背这些规则（这些规则至高无上）
+ 不要包容所有错误，除非你确定需要这样做（精准地捕获异常，不写 except:pass 风格的代码）
+ 当存在多种可能，不要尝试去猜测，而是尽量找一种，最好是唯一一种明显的解决方案（如果不确定，就用穷举法）
+ 虽然这并不容易，因为你不是 Python 之父（这里的 Dutch 是指 Guido）
+ 做也许好过不做，但不假思索就动手还不如不做（动手之前要细思量）
+ 如果你无法向人描述你的方案，那肯定不是一个好方案；反之亦然（方案测评标准）
+ 命名空间是一种绝妙的理念，我们应当多加利用（倡导与号召）

## 为什么选择 python?
+ 代码量少同一样问题，用不同的语言解决，代码量差距还是很多的，一般情况下 Python 是 Java 的 1/5

## python的特点
+ 是跨平台语言【**可以运行在不同的操作系统上**】
+ python 是一种**解释型**语言【**开发过程中没有了编译的环节**】
+ 开发过程中没有了编译这个环节，类似于PHP和Perl语言
+ python 是交互式语言
+ 可以在一个Python提示符，直接互动执行程序
+ python 是面向对象语言
+ python支持面向对象的风格或代码封装在对象的编程技术
+ python 是完全面向对象的语言    
    - 函数、模块、数字、字符串都是**对象**，在 **Python 中一切皆对象**    
    - 完全支持继承、重载、多重继承    
    - 支持重载运算符，也支持泛型设计
+ python 是初学者的语言
+ python 对初级程序员而言，是一种伟大的语言，它支持广泛的应用程序开发，从简单的文字处理到 WWW 浏览器再到游戏

### 优点
+ 易于学习：python 有相对较少的关键字，结构简单，和一个明确定义的语法，学习起来更加简单
+ 易于阅读：python 代码定义的更清晰
+ 易于维护：python 的成功在于它的源代码是相当容易维护的
+ **一个广泛的标准库**：python的最大的优势之一是丰富的库，跨平台的，在UNIX，Windows和Macintosh兼容很好。python拥有一个强大的标准库，Python语言的核心只包含数字、字符串、列表、字典、文件等常见类型和函数，而由Python标准库提供了系统管理、网络通信、文本处理、数据库接口、图形系统、XML处理等额外的功能
+ python社区提供了大量的**第三方模块，使用方式与标准库类似**。它们的功能覆盖**科学计算、人工智能、机器学习、Web开发、数据库接口、图形系统**多个领域
+ 互动模式：互动模式的支持，您可以从终端输入执行代码并获得结果的语言，互动的测试和调试代码片断
+ 可移植：基于其开放源代码的特性，Python已经被移植（也就是使其工作）到许多平台
+ 可扩展：如果需要一段运行很快的关键代码，或者是想要编写一些不愿开放的算法，你可以使用C或C++完成那部分程序，然后从你的Python程序中调用
+ 数据库：python提供所有主要的商业数据库的接口
+ GUI编程：python支持GUI可以创建和移植到许多系统调用
+ 可嵌入：你可以将python嵌入到C/C++程序，让你的程序的用户获得"脚本化"的能力
+ 免费、开源
+ 面向对象

### 缺点
+ 运行速度慢
+ 和C程序相比非常慢，因为Python是解释型语言，代码在执行时会一行一行地翻译成CPU能理解的机器码，这个翻译过程非常耗时，所以很慢。而C程序是运行前直接编译成CPU能执行的机器码，所以非常快
+ 代码不能加密
+ 如果要发布Python程序，实际上就是发布源代码，这一点跟C语言不同，C语言不用发布源代码，只需要把编译后的机器码（也就是在Windows上常见的xxx.exe文件）发布出去。要从机器码反推出C代码是不可能的，所以，凡是编译型的语言，都没有这个问题，而解释型的语言，则必须把源码发布出去

## python版本
Python有2个版本，Python2和Python3

+ 新的Python程序建议使用Python3.0版本的语法
+ Python 2.x 是过去的版本（官方已停止维护）
    - 解释器名称是 python
+ Python 3.x 是现在和未来主流的版本
    - 解释器名称是 python3
    - 相对于 Python 的早期版本，这是一个较大的升级
    - 为了不带入过多的累赘，Python 3.0 在设计的时候没有考虑向下兼容
        * 许多早期Python版本设计的程序都无法在Python3.0上正常执行
    - Python3.0发布于2008年
    - 到目前为止，Python3.0的稳定版本已经有很多年了
        * Python3.3发布于2012
        * Python3.4发布于2014
        * Python3.5发布于2015
        * Python3.6发布于2016
1. python3使用越来越广泛，大部分新的项目开始使用python3
2. 大部分三方库已经支持Python3.x
3. python3.x起始比python2.x效率低，但是python3.x有极大的优化空间，效率正在追赶
4. 使用python3，完全可以看得懂且维护Python2.x开发的项目
5. Python3.x已经成为趋势

## python解释器
+ cpython 官方默认的解释器，使用最广泛
+ jpython 运行于 java 平台上的解释器
+ ironpython 运行于 .net 平台上的解释器
+ pypy 使用 Python 编写的解释器，支持 JIT 技术（即时编译）

# 第2节 软件安装
## python解释器
Python的解释器是一种可以解释、执行Python代码的软件程序。

Python官方提供了多个解释器，包括CPython、Jython、IronPython、PyPy等。其中，CPython是最常用的一个，也是官方默认的解释器。

## 集成开发环境 (IDE)
集成开发环境（IDE），Integrated Development Environment）——集成了开发软件需要的所有工具，一般包括以下工具：

+ 图形用户界面
+ 代码编辑器 (支持代码补全／自动缩进)
+ 编译器/解释器
+ 调试器 (断点/单步执行)

### Pycharm
推荐初学者使用

官网：[https://www.jetbrains.com/pycharm/](https://www.jetbrains.com/pycharm/)

中文版：[https://www.jetbrains.com.cn/pycharm/](https://www.jetbrains.com.cn/pycharm/)

#### 版本：
专业版涉及所有高级功能，免费试用30天，购买许可密钥才能在试用期之后激活。

|  | PyCharm Professional Edition | PyCharm Community Edition |
| --- | --- | --- |
| 智能Python编辑器 | ✓ | ✓ |
| 图形调试器和测试运行器 | ✓ | ✓ |
| 导航和重构 | ✓ | ✓ |
| 代码检查 | ✓ | ✓ |
| VCS支持 | ✓ | ✓ |
| 科学工具 | ✓ |  |
| Web开发 | ✓ |  |
| Python Web框架 | ✓ |  |
| Python分析器 | ✓ |  |
| 远程开发能力 | ✓ |  |
| 支持数据库和SQL | ✓ |  |


学生可以申请免费使用：[https://www.jetbrains.com/shop/eform/students社区版是免费的，包括所需的所有基本功能。建议使用社区版即可。](https://www.jetbrains.com/shop/eform/students社区版是免费的，包括所需的所有基本功能。建议使用社区版即可。)

#### 特点：
PyCharm 是 Python 的一款非常优秀的集成开发环境

PyCharm 除了具有一般 IDE 所必备功能外，还可以在 Windows、Linux、macOS 下使用

PyCharm 适合开发大型项目

+ 一个项目通常会包含很多源文件
+ 每个源文件的代码行数是有限的，通常在几百行之内
+ 每个源文件各司其职，共同完成复杂的业务功能

### jupyter
免费使用，集成在anaconda中，或者使用pip单独安装。推荐中级玩家使用，记录自己的学习笔记、代码案例，非常方便。

### anaconda
官网：https://www.anaconda.com/products/individual

使用个人版即可，team和enterprise版太贵了，买不起。推荐高级玩家使用，集成了很多第3方库及开发工具，常用在数据分析相关的工作。

### IDLE
安装python解释器时自带的解释器，适用于简单脚本的编写。太简陋了，一般不推荐使用。

## 安装python3
官方网址：[https://www.python.org/](https://www.python.org/)

点击Downloads

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/e29ab3ea908f31d6eeee584cb4e64a40ba25af65fdb8dfbbdb6a46d0ecd29ee6.jpg)

选择windows

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/87b0788e72c5c2080fd24c08368ec19d9c40cdff1afb2616775f7ed5b9409ade.jpg)

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/0593e1e4a2f3c08e6b5b2b262a23b3e92849c2d1e4f068df37c4271f374545e7.jpg)

双击安装

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/7a30135f7f123ab6844ec47b8202e5b2d19c04fe7ffa80a7aeb87c24381c176e.jpg)

进行安装

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/a9f4a52098fe32941829b78e245cef7004ee97fb43aeb1bdd81c7fe96f78824b.jpg)

## 等待安装
安装成功点击close

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/84c39b94435a19f54517180093b2be8219abf0ff01a3f344b6a9901e0f97dc3d.jpg)

打开windwos终端验证

出现下图中的三行文字，则表示安装成功。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/510f518bae729e24ee834286fa4defad5e5592ce8fa97c6d347e03b13d84db29.jpg)

卸载：

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/e93f187683a42d316e0da593385e0422492f6056b4de06647fdb23fc6aa230a0.jpg)

## 安装pycharm
去官网下载pycharm

[https://www.jetbrains.com.cn/pycharm/](https://www.jetbrains.com.cn/pycharm/)

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/bc47123c0b0aa04be85dfb92b653ab509bb67aa3bebeb63f5c867be2aca74b57.jpg)

点击下载

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/9ce4f0beb152aaa8674e46104bef42a1262944e54d321b77680c437ab368ac9e.jpg)

根据自己的电脑配置，选择合适的版本

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/d2622a03fcf658afa8f423d17e5faffdcd9d367a6521c4ffc726a4c10151e1a4.jpg)  
免费，开源构建

点击“下一步”

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/64ea0427ecc0cd65c5d5d8fede2327460c3bca7757f7d9a4d9979e6ba5282762.jpg)

选择“下一步”

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/074b1b566324af6c51cb3981901839c51c43a17e3a2082a34cb9df7584a7fa19.jpg)

## 进行勾选选择“下一步”
![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/7479bb6919b27ec11beaa4909c73239007e8f8b35f6103b5cae316396496f10b.jpg)

进行安装

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/506bafc425ee1f4f2015eda88c367d7638ad78b560619de161c10878ece34f1b.jpg)

等待

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/179b56cb27f39cc5e12ca300d6bb41dad83e1920ddd7593d50327cb0380bfade.jpg)

点击

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/a96daea7da2883edfee1aa6cf9a2964246066f75a00639bee81f5ac7629d0793.jpg)

+ 启动- 点击继续- 汉化方法  
+ 点击Plugins，搜索 chinese  
+ 选择中文语言包  
+ 点击install  
+ 点击restart IDE，重启ide

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/9c49fa2fb64a56142968943a2e17b6de3f1f0db75ce81173247a6a391dba60a9.jpg)

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/68b16b5ccb84bab0aa97e64ed49d90282872a03a6396b96f3586e7c4b8eed7ca.jpg)

打开项目后，可以选择File  $ \rightarrow $  settings  $ \rightarrow $  Plugins进行安装。不想使用时，可以找到插件，选择禁用。

> Chinese (Simplified) Language Pack / 中文语言包
>

## 选择风格
![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/30df46bea33cf7b940719daa98835687ad2e4215d061c5b8d0ccd8dae2d4a043.jpg)

## 运行代码的方式
### Python 源代码
1. Python 源代码就是一个特殊格式的文本文件，可以使用任意文本编辑软件做 Python 的开发  
2. Python 程序的文件扩展名是 .py

### 交互式
+ 直接在终端中运行解释器，而不输入要执行的文件名
+ 在 Python 的 Shell 中直接输入 Python 的代码，会立即看到程序执行结果开始->windows系统  $ \rightarrow $  运行（或者直接搜索）输入cmd，然后在命令行模式输入python，回车。

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/4bb0f4b0854bc81ebf08198bce1b0491a6beb70671a990d864aff7f84ac9474a.jpg)

+ 优点
    - 适合于学习/验证 Python 语法或者局部代码
+ 缺点
    - 代码不能保存
    - 不适合运行太大的程序

### 退出方法
+ 直接输入 exit()

### 使用热键退出
在python解释器中，按热键`ctrl+d`可以退出解释器

## 使用pycharm
### 项目
+ 项目- 开发项目 就是开发一个专门解决一个复杂业务功能的软件
+ 通常每一个项目 就具有一个独立专属的目录，用于保存所有和项目相关的文件。
+ 一个项目 通常会包含 很多源文件

### 新建项目
通过欢迎界面或者菜单File/NewProject可以新建项目

打开pycharm软件

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/f0f386495dfe2e28aa19e8faca34da484ca2b5c4c6aed85fbb607319264c41db.jpg)

+ 新建Python学习 项目
+ 选择创建工程位置
+ 选择解释器
+ 点击创建

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/fe45a07540a67e878e168e86f138b3fc489de00360f4c103aca8e0460b463850.jpg)

### 创建成功
![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/12cbc2f27aae718ad4e2b781f4672ed0f8e3ba09e5662da3e50298df415d6a42.jpg)

### 新建python源代码
在Python学习目录下新建测试`.py`文件

在文件夹名称上点击鼠标右键，新建  $ \rightarrow $  python文件

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/13698a906bf4b4da87919c8a8324ed429aad26cf5ff1a223f66c02d7b35380f9.jpg)

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/c5fddccc41ac7d524bf5a08780680b325b50a598a9cc34d74af138065f5a3ffc.jpg)

### 编辑python源代码
编辑测试.py并且输入以下内容：

```python
print("hello python")  
print("hello world")
```

+ print是python中我们学习的第一个函数  
+ print函数的作用，可以把""内部的内容，输出到屏幕上

### 运行python源代码
运行python源代码- 在pycharm中，鼠标在代码中，鼠标右键选择运行- 在pycharm中，鼠标放在要执行的文件名上，鼠标右键选择运行- 在pycharm中，点击三角运行符号

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/12846ed16a80045542e8b6844eb1e64ff4576bcdd506084fee8df70d46723bab.jpg)

### 打开Python项目
在开始界面，直接选择上次创建的项目

![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/2acc9be412059b10d7e4705beba8d806d346f2a038b8800a6df21880c253f463.jpg)

+ 或者在进入项目后，点击文件  $ \rightarrow $  打开，可以切换到其他项目。
+ 设置项目使用的解释器版本
    - 打开的目录如果不是由PyCharm建立的项目目录，有的时候使用的解释器版本是Python2.x的，需要单独设置解释器的版本
    - 通过File/Settings...可以打开设置窗口
    - 文件导航区域能够浏览/定位/打开项目文件
    - 文件编辑区域能够编辑当前打开的文件控制台区域能够：
        * 输出程序执行内容
        * 跟踪调试代码的执行
        * 右上角的工具栏能够执行(SHIFT+F10)/调试(SHIFT+F9)代码
        * 通过控制台上方的单步执行按钮(F8)，可以单步执行代码
    - 代码执行顺序从上到下，从左至右

### 注意事项
+ python文件后缀以py结尾
+ 一行一个语句
    - 文件名尽量不用使用中文，不要包含空格
    - 不要混合使用tab键和空格缩进，缩进用于区分代码块
    - 除了在引号里（单引号、双引号）中，其它地方不要使用中文，要用英文半角
    - python编码规范遵循PEP8（[https://www.python.org/dev/peps/pep-0008/）](https://www.python.org/dev/peps/pep-0008/）)
+ 终端中运行 python xxx.py

# 知识总结及练习
## 知识总结
![](https://cdn-mineru.openxlab.org.cn/result/2025-07-21/cdcedad3-efda-431d-9d7b-f64b30198bbd/cf76f3b4110105c13c98e8f9ff2d7ad78d464940dfe9c182467149a73c72b989.jpg)

## 命令总结
| 类别 | 函数 | 说明 | 函数定义 |
| --- | --- | --- | --- |
| 基本的输入输出 | print() | 输出 | `print(*objects, sep=',' end='\n', file=None, flush=False)` |
| |  |  |  |


## 单词总结
| 单词 | 释义 |
| :---: | :---: |
| print | 打印、输出 |
| input | 输入 |
| error | 错误 |
| IDE | 集成开发环境 |
| IDLE | python解释器自带的代码编辑器 |
| Pycharm | 写代码的软件名 |
| plugins | 插件 |
| restart | 重启 |
| install | 安装 |
| uninstall | 卸载 |
| IPO | Input-Process-Output（输入-处理-输出） |
| PEP | Python Enhancement Proposals Python改进提案 |


