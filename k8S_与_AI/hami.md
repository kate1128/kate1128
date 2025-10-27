1. 中文文档：

[HAMi/README_cn.md at master · Project-HAMi/HAMi](https://github.com/Project-HAMi/HAMi/blob/master/README_cn.md)

2. webUI：

[HAMi-WebUI/README_ZH.md at main · Project-HAMi/HAMi-WebUI](https://github.com/Project-HAMi/HAMi-WebUI/blob/main/README_ZH.md)

3. hami文档：

[What is HAMi? | HAMi](https://project-hami.io/docs/)

[GitCode - 全球开发者的开源社区,开源代码托管平台](https://gitcode.com/gh_mirrors/ha/HAMi/overview?utm_source=artical_gitcode&index=top&type=card&webUrl&isLogin=1)

4. hami代码：

[GitHub - Project-HAMi/HAMi: Heterogeneous AI Computing Virtualization Middleware](https://github.com/Project-HAMi/HAMi)

5. nvidia：

[Installing the NVIDIA Container Toolkit — NVIDIA Container Toolkit 1.17.3 documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

HAMi（High-performance Adaptive Memory Interface）是一个针对异构计算环境的内存管理框架，它通过虚拟化GPU内存资源，使得多个应用程序可以在共享同一物理GPU的情况下运行，从而提高资源利用率和系统性能。HAMi的主要目标是优化GPU资源的分配和利用，尤其是在深度学习、科学计算和图形渲染等需要大量计算资源的应用场景中。

虚拟内存是一种内存管理技术，它允许操作系统将物理内存（RAM）和硬盘上的空间结合起来使用，为程序提供比物理内存更大的内存空间。当程序试图访问超过物理内存限制的内存时，操作系统会将这些数据从硬盘上的交换文件（swap file）加载到物理内存中。这样，程序就可以认为它有无限的内存可用，而实际上，操作系统在后台管理着物理内存和硬盘之间的数据交换。

HAMi和虚拟内存的概念在某些方面是相似的，因为它们都涉及到通过某种形式的抽象来扩展内存资源。然而，它们的关注点和实现方式不同。HAMi专注于GPU资源的虚拟化，允许多个应用程序共享物理GPU资源，就像它们各自拥有独立的GPU一样。这种虚拟化技术可以提高GPU资源的利用率，减少资源浪费，并支持更灵活的资源分配。

虚拟内存则是针对所有类型数据的内存管理，它是一种全系统范围的内存扩展技术，用于解决物理内存不足的问题。它涉及到的抽象层次更高，涵盖了整个操作系统的内存管理，而不是专注于某一类硬件资源。

总结来说，HAMi可以被看作是针对GPU资源的虚拟化技术，它提供了对GPU内存资源的细粒度控制和管理，类似于虚拟内存提供了对系统级内存资源的控制，但它们的应用场景和优化目标不同。



---

<font style="color:rgb(28, 30, 33);">异构 AI 计算虚拟化中间件 (HAMi)，原名</font>`<font style="color:rgb(28, 30, 33);"> k8s-vGPU-scheduler</font>`<font style="color:rgb(28, 30, 33);">，是一个“一体化”图表，旨在管理 k8s 集群中的</font>**<font style="color:rgb(28, 30, 33);">异构 </font>**<font style="color:rgb(28, 30, 33);">AI 计算设备。它可以提供在任务之间共享</font>**<font style="color:rgb(28, 30, 33);">异构 </font>**<font style="color:rgb(28, 30, 33);">AI 设备的能力。</font>

![画板](https://cdn.nlark.com/yuque/0/2024/jpeg/2639475/1728976159467-22fa14d2-6701-440f-8ec2-eb13d8d59a88.jpeg)

![画板](https://cdn.nlark.com/yuque/0/2024/jpeg/2639475/1728976390896-fd4f0cb6-f379-4d2b-bb12-03fda8d1bd73.jpeg)

> 所以是做了一个类似hami的调度中间层，为了解决异构调度？？
>
> 应用：![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1728976750300-484bb62d-79e8-44a2-8c93-adec674b5bd7.png)？？？
>
> 但是没有任何device资源，也没有resources的限制？？？
>

