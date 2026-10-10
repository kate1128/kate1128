<!-- type: glossary -->
# HAMI (Heterogeneous AI Computing Virtualization Middleware)

> 一个开源的 **Kubernetes GPU 虚拟化与共享中间件**，核心能力是将一张物理 GPU 细粒度切分给多个容器同时使用，解决"GPU 贵、独占浪费"的问题。

## 是什么

在 K8s 集群中，NVIDIA 原生的 GPU 调度只能整卡分配——一张 A100 只能给一个 Pod。实际场景中，推理任务可能只需要 2-3GB 显存，独占一整张 80GB 的卡是巨大浪费。

HAMi 的解决思路：在 K8s 和 GPU 驱动之间插入一层虚拟化中间件，实现：
- **显存隔离**：限制每个容器可用的 GPU 显存（如分配 8GB/80GB）
- **算力隔离**：限制每个容器可用的 GPU 核心利用率
- **多容器共享**：多个 Pod 可以同时使用同一张物理 GPU

类似产品：NVIDIA MIG（硬件级，仅 A100/H100 支持）、MPS（软件级，兼容性有限）、HAMi（软件级，兼容性好）

## 为什么重要

- **GPU 利用率提升**：很多团队的 GPU 利用率不到 30%，HAMi 可以让一张卡跑多个推理服务
- **降低成本**：多个小模型推理可以共享 GPU，不用为每个模型各买一张卡
- **K8s 原生化**：以 Device Plugin 形式运行，对业务 Pod 透明——不需要改代码
- **国产 GPU 兼容**：支持华为 Ascend、寒武纪等国产加速卡（这是 HAMi 相比 NVIDIA MIG 的独特优势）

## 关键要点

- **工作原理**：通过劫持 CUDA API 调用，在用户态实现显存和算力的限制——容器内看到的是"虚拟 GPU"
- **核心组件**：HAMi-core（CUDA 拦截层）+ HAMi-scheduler（K8s 调度器扩展）+ Device Plugin
- **与 MIG 的区别**：MIG 是硬件切分（颗粒度固定、安全性最高），HAMi 是软件切分（颗粒度灵活、兼容性更好）
- **典型场景**：模型推理共享（多个小模型共用一张 GPU）、开发环境多租户共享、GPU 池化管理

## 相关概念

- [[Kubernetes Device Plugin]] — HAMi 在 K8s 中的集成方式
- [[GPU 虚拟化]] — HAMi 所属的技术大类
- [[NVIDIA MIG]] — HAMi 的竞品/互补方案
- [[vLLM]] — 推理框架，常与 HAMi 配合使用（Hami 管 GPU 分配，vLLM 管推理加速）

## 参考

- [HAMi GitHub](https://github.com/Project-HAMi/HAMi)
- [开源 vGPU 方案 HAMi 介绍](https://mp.weixin.qq.com/s?__biz=Mzk0NzE5OTQyOQ==&mid=2247487108&idx=1&sn=6d0dbed2ba87faa39f02c980d5718839)
- [HAMI-VGPU 思维导图](https://github.com/Project-HAMi/HAMi/blob/master/docs/mind-map/HAMI-VGPU-mind-map-Chinese.xmind)