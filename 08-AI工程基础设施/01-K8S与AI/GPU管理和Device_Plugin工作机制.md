<font style="color:rgb(38, 38, 38);">本文将主要分享以下几个方面的内容：</font>

1. <font style="color:rgb(38, 38, 38);">需求来源</font>
2. <font style="color:rgb(38, 38, 38);">GPU 的容器化</font>
3. <font style="color:rgb(38, 38, 38);">Kubernetes 的 GPU 管理</font>
4. <font style="color:rgb(38, 38, 38);">工作原理</font>
5. <font style="color:rgb(38, 38, 38);">课后思考与实践</font>

# <font style="color:rgb(24, 24, 24);"> 需求来源</font>
<font style="color:rgb(38, 38, 38);">2016 年，随着 AlphaGo 的走红和 TensorFlow 项目的异军突起，一场名为 AI 的技术革命迅速从学术圈蔓延到了工业界，所谓 AI 革命从此拉开了帷幕。</font>

<font style="color:rgb(38, 38, 38);">经过三年的发展，AI 有了许许多多的落地场景，包括智能客服、人脸识别、机器翻译、以图搜图等功能。其实机器学习或者说是人工智能，并不是什么新鲜的概念。而这次热潮的背后，云计算的普及以及算力的巨大提升，才是真正将人工智能从象牙塔带到工业界的一个重要推手。</font>

<font style="color:rgb(38, 38, 38);">与之相对应的，从 2016 年开始，Kubernetes 社区就不断收到来自不同渠道的大量诉求。希望能在 Kubernetes 集群上运行 TensorFlow 等机器学习框架。这些诉求中，除了前面课程所介绍的，像 Job 这些离线任务的管理之外，还有一个巨大的挑战：深度学习所依赖的异构设备及英伟达的 GPU 支持。</font>

<font style="color:rgb(38, 38, 38);">我们不禁好奇起来：Kubernetes 管理 GPU 能带来什么好处呢？本质上是成本和效率的考虑。由于相对 CPU 来说，GPU 的成本偏高。在云上单 CPU 通常是一小时几毛钱，而 GPU 的花费则是从单 GPU 每小时 10 元 ~ 30 元不等，这就要想方设法的提高 GPU 的使用率。</font>

<br/>color2
<font style="color:rgb(38, 38, 38);">为什么要用 Kubernetes 管理以 GPU 为代表的异构资源？</font>

<font style="color:rgb(38, 38, 38);">具体来说是三个方面：</font>

+ **<font style="color:rgb(38, 38, 38);">加速部署</font>**<font style="color:rgb(38, 38, 38);">：通过容器构想避免重复部署机器学习复杂环境；</font>

<font style="color:rgb(38, 38, 38);">首先是加速部署，避免把时间浪费在环境准备的环节中。通过容器镜像技术，将整个部署过程进行固化和复用，可以借此提升 GPU 的使用效率。</font>

+ **<font style="color:rgb(38, 38, 38);">提升集群资源使用率</font>**<font style="color:rgb(38, 38, 38);">：统一调度和分配集群资源；</font>

<font style="color:rgb(38, 38, 38);">通过分时复用，来提升 GPU 的使用效率。当 GPU 的卡数达到一定数量后，就需要用到 Kubernetes 的统一调度能力，使得资源使用方能够做到用即申请、完即释放，从而盘活整个 GPU 的资源池。</font>

+ **<font style="color:rgb(38, 38, 38);">保障资源独享</font>**<font style="color:rgb(38, 38, 38);">：利用容器隔离异构设备，避免互相影响。</font>

<font style="color:rgb(38, 38, 38);">而此时还需要通过 Docker 自带的设备隔离能力，避免不同应用的进程运行同一个设备上，造成互相影响。在高效低成本的同时，也保障了系统的稳定性。</font>

<br/>

# <font style="color:rgb(24, 24, 24);">GPU 的容器化</font>
<font style="color:rgb(38, 38, 38);">上面了解到了通过 Kubernetes 运行 GPU 应用的好处，通过前面的学习也知道，Kubernetes 是容器调度平台，而其中的调度单元是容器，所以在学习如何使用 Kubernetes 之前，先了解一下如何在容器环境内运行 GPU 应用。</font>

## <font style="color:rgb(24, 24, 24);">如何利用容器运行 GPU 程序</font>
<font style="color:rgb(38, 38, 38);">在容器环境下使用 GPU 应用，实际上不复杂。主要分为两步：</font>

+ **<font style="color:rgb(38, 38, 38);">构建支持 GPU 的容器镜像</font>**<font style="color:rgb(38, 38, 38);">；</font>
+ **<font style="color:rgb(38, 38, 38);">利用 Docker 将该镜像运行起来</font>**<font style="color:rgb(38, 38, 38);">，并且把 </font>**<font style="color:rgb(38, 38, 38);">GPU 设备</font>**<font style="color:rgb(38, 38, 38);">和</font>**<font style="color:rgb(38, 38, 38);">依赖库</font>**<font style="color:rgb(38, 38, 38);">映射到容器中。</font>

## <font style="color:rgb(24, 24, 24);">如何准备 GPU 容器镜像</font>
<font style="color:rgb(38, 38, 38);">有两个方法准备：</font>

### <font style="color:rgb(38, 38, 38);">直接使用官方深度学习容器镜像</font>
<font style="color:rgb(38, 38, 38);">比如直接从 </font>[https://hub.docker.com/](https://hub.docker.com/)<font style="color:rgb(38, 38, 38);"> 或者阿里云镜像服务中寻找官方的 GPU 镜像，包括像 TensorFlow、Caffe、PyTorch 等流行的机器学习框架，都有提供标准的镜像。这样的好处是简单便捷，而且安全可靠。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735894908212-dea5ba02-ab9c-4de0-ab22-d5b6e9f54221.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735894918630-7ec02cb1-a7d0-4d72-8816-d1287e798622.png)

### <font style="color:rgb(38, 38, 38);">基于 Nvidia 的 CUDA 镜像基础构建</font>
<font style="color:rgb(38, 38, 38);">如果官方镜像无法满足需求时，对 TensorFlow 框架进行了定制修改，就需要重新编译构建自己的 TensorFlow 镜像。这种情况下，我们的最佳实践是：依托于 Nvidia 官方镜像继续构建，而不要从头开始。</font>

<font style="color:rgb(38, 38, 38);">如下图中的 TensorFlow 例子所示，这个就是以 Cuda 镜像为基础，开始构建自己的 GPU 镜像。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735894943318-26d44c01-5217-4163-8536-a54e262f1459.png)

## <font style="color:rgb(24, 24, 24);">GPU 容器镜像原理</font>
<font style="color:rgb(38, 38, 38);">要了解如何构建 GPU 容器镜像，先要知道如何要在宿主机上安装 GPU 应用。</font>

<font style="color:rgb(38, 38, 38);">如下图左边所示，最底层是先安装 Nvidia 硬件驱动；再到上面是通用的 Cuda 工具库；最上层是 PyTorch、TensorFlow 这类的机器学习框架。上两层的 CUDA 工具库和应用的耦合度较高，应用版本变动后，对应的 CUDA 版本大概率也要更新；而最下层的 Nvidia 驱动，通常情况下是比较稳定的，它不会像 CUDA 和应用一样，经常更新。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735894986738-a5e73c4c-dcfe-4e9c-a20e-aace87cc3028.png)

<font style="color:rgb(38, 38, 38);">同时 Nvidia 驱动需要内核源码编译，如上图右侧所示，英伟达的 GPU 容器方案是：在宿主机上安装 Nvidia 驱动，而在 CUDA 以上的软件交给容器镜像来做。同时把 Nvidia 驱动里面的链接以 Mount Bind 的方式映射到容器中。这样的一个好处是：当你安装了一个新的 Nvidia 驱动之后，你就可以在同一个机器节点上运行不同版本的 CUDA 镜像了。</font>

## <font style="color:rgb(24, 24, 24);">如何利用容器运行 GPU 程序</font>
<font style="color:rgb(38, 38, 38);">有了前面的基础，我们就比较容易理解 GPU 容器的工作机制。下图是一个使用 Docker 运行 GPU 容器的例子。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735895011658-4392e0dd-fa6a-4e06-a2d6-1f12e0904525.png)

<font style="color:rgb(38, 38, 38);">我们可以观察到，在运行时刻一个 GPU 容器和普通容器之间的差别，仅仅在于需要将宿主机的设备和 Nvidia 驱动库映射到容器中。</font>

<font style="color:rgb(38, 38, 38);">上图右侧反映了 GPU 容器启动后，容器中的 GPU 配置。右上方展示的是设备映射的结果，右下方显示的是驱动库以 Bind 方式映射到容器后，可以看到的变化。</font>

<font style="color:rgb(38, 38, 38);">通常大家会使用 Nvidia-docker 来运行 GPU 容器，而 Nvidia-docker 的实际工作就是来自动化做这两个工作。其中挂载设备比较简单，而真正比较复杂的是 GPU 应用依赖的驱动库。对于深度学习，视频处理等不同场景，所使用的一些驱动库并不相同。这又需要依赖 Nvidia 的领域知识，而这些领域知识就被贯穿到了 Nvidia 的容器之中。</font>

# <font style="color:rgb(24, 24, 24);">Kubernetes 的 GPU 管理</font>
## <font style="color:rgb(24, 24, 24);">如何部署 GPU Kubernetes</font>
<font style="color:rgb(38, 38, 38);">首先看一下如何给一个 Kubernetes 节点增加 GPU 能力，我们以 CentOS 节点为例。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735895036220-0361452a-ce8b-44f2-8075-701e02918e11.png)

<font style="color:rgb(38, 38, 38);">如上图所示：</font>

### <font style="color:rgb(38, 38, 38);">首先安装 Nvidia 驱动；</font>
<font style="color:rgb(38, 38, 38);">由于 Nvidia 驱动需要内核编译，所以在安装 Nvidia 驱动之前需要安装 gcc 和内核源码。</font>

### <font style="color:rgb(38, 38, 38);">第二步通过 yum 源，安装 Nvidia Docker2；</font>
<font style="color:rgb(38, 38, 38);">安装完 Nvidia Docker2 需要重新加载 docker，可以检查 docker 的 daemon.json 里面默认启动引擎已经被替换成了 nvidia，也可以通过 docker info 命令查看运行时刻使用的 runC 是不是 Nvidia 的 runC。</font>

### <font style="color:rgb(38, 38, 38);">第三步是部署 Nvidia Device Plugin。</font>
<font style="color:rgb(38, 38, 38);">从 Nvidia 的 git repo 下去下载 Device Plugin 的部署声明文件，并且通过 kubectl create 命令进行部署。这里 Device Plugin 是以 deamonset 的方式进行部署的。这样我们就知道，如果需要排查一个 Kubernetes 节点无法调度 GPU 应用的问题，需要从这些模块开始入手，比如我要查看一下 Device Plugin 的日志，Nvidia 的 runC 是否配置为 docker 默认 runC 以及 Nvidia 驱动是否安装成功。</font>

## <font style="color:rgb(24, 24, 24);">验证部署 GPU Kubernetes 结果</font>
<font style="color:rgb(38, 38, 38);">当 GPU 节点部署成功后，我们可以从节点的状态信息中发现相关的 GPU 信息。</font>

### <font style="color:rgb(38, 38, 38);">一个是 GPU 的名称，这里是  nvidia.com/gpu；</font>
### <font style="color:rgb(38, 38, 38);">另一个是它对应的数量，如下图所示是 40 ，表示在该节点中含有 40 个 GPU。</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736131446226-ed928cf4-4c66-4b14-8d5b-2204fe89f596.png)

## <font style="color:rgb(24, 24, 24);">在 Kubernetes 中使用 GPU 的 yaml 样例</font>
<font style="color:rgb(38, 38, 38);">站在用户的角度，在 Kubernetes 中使用 GPU 容器还是非常简单的。只需要在 Pod 资源配置的 limit 字段中指定 nvidia.com/gpu 使用 GPU 的数量，如下图样例中我们设置的数量为 1；然后再通过 kubectl create 命令将 GPU 的 Pod 部署完成。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736131505721-14034bc4-9761-4656-a68a-3e0327476471.png)

## <font style="color:rgb(24, 24, 24);">查看运行结果</font>
<font style="color:rgb(38, 38, 38);">部署完成后可以登录到容器中执行 nvidia-smi 命令观察一下结果，可以看到在该容器中使用了一张 T4 的 GPU 卡。说明在该节点中的两张 GPU 卡其中一张已经能在该容器中使用了，但是节点的另外一张卡对于改容器来说是完全透明的，它是无法访问的，这里就体现了 GPU 的隔离性。</font>

```python
kubectl exec -it application-2412100938434071k3 -- nvidia-smi
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736131566047-f292f34e-7e78-4dd2-95ff-522d79e08c90.png)

# <font style="color:rgb(24, 24, 24);">工作原理</font>
## <font style="color:rgb(24, 24, 24);">通过扩展的方式管理 GPU 资源</font>
<font style="color:rgb(38, 38, 38);">Kubernetes 本身是通过插件扩展的机制来管理 GPU 资源的，具体来说这里有两个独立的内部机制。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735895122336-9dbe7602-1abd-4ae2-a821-47fc94e30b47.png)

### <font style="color:rgb(38, 38, 38);">Extend Resources</font>
<font style="color:rgb(38, 38, 38);">允许用户自定义资源名称。而该资源的度量是整数级别，这样做的目的在于通过一个通用的模式支持不同的异构设备，包括 RDMA、FPGA、AMD GPU 等等，而不仅仅是为 Nvidia GPU 设计的；</font>

### <font style="color:rgb(38, 38, 38);">Device Plugin Framework </font>
<font style="color:rgb(38, 38, 38);">允许第三方设备提供商以外置的方式对设备进行全生命周期的管理，而 Device Plugin Framework 建立 Kubernetes 和 Device Plugin 模块之间的桥梁。它一方面负责设备信息的上报到 Kubernetes，另一方面负责设备的调度选择。</font>

## <font style="color:rgb(24, 24, 24);">Extended Resource 的上报</font>
<font style="color:rgb(38, 38, 38);">Extend Resources 属于 Node-level 的 api，完全可以独立于 Device Plugin 使用。而上报 Extend Resources，只需要通过一个 PACTH API 对 Node 对象进行 status 部分更新即可，而这个 PACTH 操作可以通过一个简单的 curl 命令来完成。这样，在 Kubernetes 调度器中就能够记录这个节点的 GPU 类型，它所对应的资源数量是 1。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735895147122-62fd816f-5353-4836-a913-a97e2b0be2cb.png)

<font style="color:rgb(38, 38, 38);">	当然如果使用的是 Device Plugin，就不需要做这个 PACTH 操作，只需要遵从 Device Plugin 的编程模型，在设备上报的工作中 Device Plugin 就会完成这个操作。</font>

## <font style="color:rgb(24, 24, 24);">Device Plugin 工作机制</font>
<font style="color:rgb(38, 38, 38);">介绍一下 Device Plugin 的工作机制，整个 Device Plugin 的工作流程可以分成两个部分：</font>

+ **<font style="color:rgb(38, 38, 38);">一个是启动时刻的资源上报</font>**
+ **<font style="color:rgb(38, 38, 38);">另一个是用户使用时刻的调度和运行</font>**

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735895168534-d7ffa903-39d1-4f7e-acc5-bbd014125b5d.png)

<font style="color:rgb(38, 38, 38);">Device Plugin 的开发非常简单。主要包括最关注与最核心的两个事件方法：</font>

+ <font style="color:rgb(38, 38, 38);">其中 ListAndWatch 对应资源的上报，同时还提供健康检查的机制。当设备不健康的时候，可以上报给 Kubernetes 不健康设备的 ID，让 Device Plugin Framework 将这个设备从可调度设备中移除；</font>
+ <font style="color:rgb(38, 38, 38);">而 Allocate 会被 Device Plugin 在部署容器时调用，传入的参数核心就是容器会使用的设备 ID，返回的参数是容器启动时，需要的设备、数据卷以及环境变量。</font>

HAMI 就是用这种方式：

[HAMi vGPU 方案原理分析 Part1：hami-device-plugin-nvidia 实现](https://mp.weixin.qq.com/s/IO3dlhkF5Gu-pNqcS8cmuA)

## <font style="color:rgb(24, 24, 24);">资源上报和监控</font>
<font style="color:rgb(38, 38, 38);">对于每一个硬件设备，都需要它所对应的 Device Plugin 进行管理，这些 Device Plugin 以客户端的身份通过 GRPC 的方式对 kubelet 中的 Device Plugin Manager 进行连接，并且将自己监听的 Unis socket api 的版本号和设备名称比如 GPU，上报给 kubelet。</font>

<font style="color:rgb(38, 38, 38);">我们来看一下 Device Plugin 资源上报的整个流程。总的来说，整个过程分为四步，其中前三步都是发生在节点上，第四步是 kubelet 和 api-server 的交互。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735895193440-da518903-0b9c-46a2-b513-29a8a52cb644.png)

1. **<font style="color:rgb(38, 38, 38);">第一步是 Device Plugin 的注册</font>**<font style="color:rgb(38, 38, 38);">，</font>**<font style="color:rgb(38, 38, 38);">需要 Kubernetes 知道要跟哪个 Device Plugin 进行交互</font>**<font style="color:rgb(38, 38, 38);">。这是因为一个节点上可能有多个设备，需要 Device Plugin 以客户端的身份向 Kubelet 汇报三件事情。</font>
    1. <font style="color:rgb(38, 38, 38);">我是谁？就是 Device Plugin 所管理的设备名称，是 GPU 还是 RDMA；</font>
    2. <font style="color:rgb(38, 38, 38);">我在哪？就是插件自身监听的 unis socket 所在的文件位置，让 kubelet 能够调用自己；</font>
    3. <font style="color:rgb(38, 38, 38);">交互协议，即 API 的版本号。</font>
2. **<font style="color:rgb(38, 38, 38);">第二步是服务启动</font>**<font style="color:rgb(38, 38, 38);">，</font>**<font style="color:rgb(38, 38, 38);">Device Plugin 会启动一个 GRPC 的 server</font>**<font style="color:rgb(38, 38, 38);">。在此之后 Device Plugin 一直以这个服务器的身份提供服务让 kubelet 来访问，而监听地址和提供 API 的版本就已经在第一步完成了；</font>
3. **<font style="color:rgb(38, 38, 38);">第三步，当该 GRPC server 启动之后，kubelet 会建立一个到 Device Plugin 的 ListAndWatch 的长连接， 用来发现设备 ID 以及设备的健康状态</font>**<font style="color:rgb(38, 38, 38);">。当 Device Plugin 检测到某个设备不健康的时候，就会主动通知 kubelet。而此时如果这个设备处于空闲状态，kubelet 会将其移除可分配的列表。但是当这个设备已经被某个 Pod 所使用的时候，kubelet 就不会做任何事情，如果此时杀掉这个 Pod 是一个很危险的操作。</font>
4. **<font style="color:rgb(38, 38, 38);">第四步，kubelet 会将这些设备暴露到 Node 节点的状态中，把设备数量发送到 Kubernetes 的 api-server 中</font>**<font style="color:rgb(38, 38, 38);">。后续调度器可以根据这些信息进行调度。</font>

<font style="color:rgb(38, 38, 38);">需要注意的是 kubelet 在向 api-server 进行汇报的时候，只会汇报该 GPU 对应的数量。而 kubelet 自身的 Device Plugin Manager 会对这个 GPU 的 ID 列表进行保存，并用来具体的设备分配。而这个对于 Kubernetes 全局调度器来说，它不掌握这个 GPU 的 ID 列表，它只知道 GPU 的数量。这就意味着在现有的 Device Plugin 工作机制下，Kubernetes 的全局调度器无法进行更复杂的调度。比如说想做两个 GPU 的亲和性调度，同一个节点两个 GPU 可能需要进行通过 NVLINK 通讯而不是 PCIe 通讯，才能达到更好的数据传输效果。在这种需求下，目前的 Device Plugin 调度机制中是无法实现的。</font>

## <font style="color:rgb(24, 24, 24);">Pod 的调度和运行的过程</font>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1735895232920-91fc9d81-01ee-48a1-beb1-44dcce2b2a01.png)

<font style="color:rgb(38, 38, 38);">Pod 想使用一个 GPU 的时候，它只需要像之前的例子一样，在 Pod 的 Resource 下 limits 字段中声明 GPU 资源和对应的数量 (比如nvidia.com/gpu: 1)。Kubernetes 会找到满足数量条件的节点，然后将该节点的 GPU 数量减 1，并且完成 Pod 与 Node 的绑定。</font>

<font style="color:rgb(38, 38, 38);">绑定成功后，自然就会被对应节点的 kubelet 拿来创建容器。而当 kubelet 发现这个 Pod 的容器请求的资源是一个 GPU 的时候，kubelet 就会委托自己内部的 Device Plugin Manager 模块，从自己持有的 GPU 的 ID 列表中选择一个可用的 GPU 分配给该容器。此时 kubelet 就会向本机的 DeAvice Plugin 发起一个 Allocate 请求</font><font style="color:rgb(250, 140, 22);">，</font><font style="color:rgb(38, 38, 38);">这个请求所携带的参数，正是即将分配给该容器的设备 ID 列表。</font>

<font style="color:rgb(38, 38, 38);">Device Plugin 收到 AllocateRequest 请求之后，它就会根据 kubelet 传过来的设备 ID，去寻找这个设备 ID 对应的设备路径、驱动目录以及环境变量，并且以 AllocateResponse 的形式返还给 kubelet。</font>

<font style="color:rgb(38, 38, 38);">AllocateResponse 中所携带的设备路径和驱动目录信息，一旦返回给 kubelet 之后，kubelet 就会根据这些信息执行为容器分配 GPU 的操作，这样 Docker 会根据 kubelet 的指令去创建容器，而这个容器中就会出现 GPU 设备。并且把它所需要的驱动目录给挂载进来，至此 Kubernetes 为 Pod 分配一个 GPU 的流程就结束了。</font>

# <font style="color:rgb(24, 24, 24);">思考与实践</font>
## <font style="color:rgb(24, 24, 24);">总结</font>
<font style="color:rgb(38, 38, 38);">在本次课程中，我们一起学习了在 Docker 和 Kubernetes 上使用 GPU。</font>

<font style="color:rgb(38, 38, 38);">GPU 的容器化：</font>

+ <font style="color:rgb(38, 38, 38);">如何去构建一个 GPU 镜像</font>
+ <font style="color:rgb(38, 38, 38);">如何直接在 Docker 上运行 GPU 容器</font>

<font style="color:rgb(38, 38, 38);">利用 Kubernetes 管理 GPU 资源：</font>

+ <font style="color:rgb(38, 38, 38);">如何在 Kubernetes 支持 GPU 调度</font>
+ <font style="color:rgb(38, 38, 38);">如何验证 Kubernetes 下的 GPU 配置</font>
+ <font style="color:rgb(38, 38, 38);">调度 GPU 容器的方法</font>

<font style="color:rgb(38, 38, 38);">Device Plugin 的工作机制：</font>

+ <font style="color:rgb(38, 38, 38);">资源的上报和监控</font>
+ <font style="color:rgb(38, 38, 38);">Pod 的调度和运行</font>

<font style="color:rgb(38, 38, 38);">思考：</font>

+ <font style="color:rgb(38, 38, 38);">目前的缺陷</font>
+ <font style="color:rgb(38, 38, 38);">社区常见的 Device Plugin</font>

## <font style="color:rgb(24, 24, 24);">Device Plugin 机制的缺陷</font>
<font style="color:rgb(38, 38, 38);">最后我们来思考一个问题，现在的 Device Plugin 是否完美无缺？</font>

<font style="color:rgb(38, 38, 38);">需要指出的是 Device Plugin 整个工作机制和流程上，实际上跟学术界和工业界的真实场景有比较大的差异。这里最大的问题在于 GPU 资源的调度工作，实际上都是在 kubelet 上完成的。</font>

<font style="color:rgb(38, 38, 38);">而作为全局的调度器对这个参与是非常有限的，作为传统的 Kubernetes 调度器来说，它只能处理 GPU 数量。一旦你的设备是异构的，不能简单地使用数目去描述需求的时候，比如我的 Pod 想运行在两个有 nvlink 的 GPU 上，这个 Device Plugin 就完全不能处理。</font>

<font style="color:rgb(38, 38, 38);">更不用说在许多场景上，我们希望调度器进行调度的时候，是根据整个集群的设备进行全局调度，这种场景是目前的 Device Plugin 无法满足的。</font>

<font style="color:rgb(38, 38, 38);">更为棘手的是在 Device Plugin 的设计和实现中，像 Allocate 和 ListAndWatch 的 API 去增加可扩展的参数也是没有作用的。这就是当我们使用一些比较复杂的设备使用需求的时候，实际上是无法通过 Device Plugin 来扩展 API 实现的。</font>

<font style="color:rgb(38, 38, 38);">因此目前的 Device Plugin 设计涵盖的场景其实是非常单一的， 是一个可用但是不好用的状态。这就能解释为什么像 Nvidia 这些厂商都实现了一个基于 Kubernetes 上游代码进行 fork 了自己解决方案，也是不得已而为之。</font>

## <font style="color:rgb(24, 24, 24);">社区的异构资源调度方案</font>
+ <font style="color:rgb(38, 38, 38);">Nvidia GPU Device Plugin：</font>[https://github.com/NVIDIA/k8s-device-plugin](https://github.com/NVIDIA/k8s-device-plugin)
+ <font style="color:rgb(38, 38, 38);">GPU Share Device Plugin ：</font>[https://github.com/aliyunContainerService/gpushare-device-plugin](https://github.com/aliyunContainerService/gpushare-device-plugin)
+ <font style="color:rgb(38, 38, 38);">RDMA Device Plugin ：</font>[https://github.com/Mellanox/k8s-rdma-sriov-dev-plugin](https://github.com/Mellanox/k8s-rdma-sriov-dev-plugin)
+ <font style="color:rgb(38, 38, 38);">FPGA Device Plugin ：</font>[https://github.com/Xilinx/FPGA_as_a_Service/tree/master](https://github.com/Xilinx/FPGA_as_a_Service/tree/master)

<font style="color:rgb(38, 38, 38);">第一个是 Nvidia 贡献的调度方案，这是最常用的调度方案；</font>

<font style="color:rgb(38, 38, 38);">第二个是由阿里云服务团队贡献的 GPU 共享的调度方案，其目的在于解决用户共享 GPU 调度的需求，欢迎大家一起来使用和改进；</font>

<font style="color:rgb(38, 38, 38);">下面的两个 RDMA 和 FPGA 是由具体厂商提供的调度方案。</font>

