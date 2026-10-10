## Ollama 简介
> "Get up and running with large language models locally."
>

[Ollama](https://github.com/ollama/ollama) 仓库创建于 2023年6月26日，截至2024年8月，Ollama 经过了一年多的发展。相信在不久的未来，将会在越来越多的边缘端产品中看到大模型的身影。

什么是 Ollama ？正如 Ollama 官方仓库所说：本地启动并运行大型语言模型。

Ollama 是一个开源的大型语言模型服务工具，旨在帮助用户快速在本地运行大模型。通过简单的安装指令，用户可以通过一条命令轻松启动和运行开源的大型语言模型。 它提供了一个简洁易用的命令行界面和服务器，专为构建大型语言模型应用而设计。用户可以轻松下载、运行和管理各种开源 LLM。与传统 LLM 需要复杂配置和强大硬件不同，Ollama 能够让用户在消费级的 PC 上体验 LLM 的强大功能。

Ollama 会自动监测本地计算资源，如有 GPU 的条件，会优先使用 GPU 的资源，同时模型的推理速度也更快。如果没有 GPU 条件，直接使用 CPU 资源。

Ollama 极大地简化了在 Docker 容器中部署和管理大型语言模型的过程，使用户能够迅速在本地启动和运行这些模型。

## Ollama 特点
+ **开源免费**：Ollama 及其支持的模型完全开源且免费，用户可以随时访问和使用这些资源，而无需支付任何费用。
+ **简单易用**：Ollama 无需复杂的配置和安装过程，只需几条简单的命令即可启动和运行，为用户节省了大量时间和精力。
+ **支持多平台**：Ollama 提供了多种安装方式，支持 Mac、Linux 和 Windows 平台，并提供 Docker 镜像，满足不同用户的需求。
+ **模型丰富**：Ollama 支持包括 Llama3.1、Gemma2、Qwen2 在内的众多热门开源 LLM，用户可以轻松一键下载和切换模型，享受丰富的选择。
+ **功能齐全**：Ollama 将模型权重、配置和数据捆绑成一个包，定义为 Modelfile，使得模型管理更加简便和高效。
+ **支持工具调用**：Ollama 支持使用 Llama 3.1 等模型进行工具调用。这使模型能够使用它所知道的工具来响应给定的提示，从而使模型能够执行更复杂的任务。
+ **资源占用低**：Ollama 优化了设置和配置细节，包括 GPU 使用情况，从而提高了模型运行的效率，确保在资源有限的环境下也能顺畅运行。
+ **隐私保护**：Ollama 所有数据处理都在本地机器上完成，可以保护用户的隐私。
+ **社区活跃**：Ollama 拥有一个庞大且活跃的社区，用户可以轻松获取帮助、分享经验，并积极参与到模型的开发和改进中，共同推动项目的发展。

## 支持的模型
Ollama 支持的模型库列表 [https://ollama.com/library](https://ollama.com/library)。

下面是一些受欢迎的模型：

| Model | Tag | Parameters | Size | Download |
| :---: | :---: | :---: | :---: | --- |
| llama3 | - | 70b | 40GB | `ollama run llama3:70b` |
| llama3 | - | 8b | 4.7GB | `ollama run llama3:8b` |
| gemma | - | 7b | 5.0GB | `ollama run gemma:7b` |
| gemma | - | 2b | 1.7GB | `ollama run gemma:2b` |
| mistral | - | 7b | 4.1GB | `ollama run mistral:7b` |
| qwen | - | 110b | 63GB | `ollama run qwen:110b` |
| phi3 | - | 3.8b | 2.2GB | `ollama run phi3:3.8b` |
| llama2 | - | 7b | 3.8GB | `ollama run llama2:7b` |
| codellama | Code | 70b | 39GB | `ollama run codellama:70b` |
| llama3.1 | - | 405b | 231GB | `ollama run llama3.1:405b` |
| gemma2 | - | 27b | 16GB | `ollama run gemma2:27b` |
| qwen2 | - | 72b | 41GB | `ollama run qwen2:72b` |
| llava | Vision | 7b | 4.7GB | `ollama run llava:7b` |
| nomic-embed-text | Embedding | v1.5 | 274MB | `ollama pull nomic-embed-text:v1.5` |


> 还有好多其他的模型自己看一下，注意：运行 7B 模型至少需要 8GB 内存，运行 13B 模型至少需要 16GB 内存，运行 33B 模型至少需要 32GB 内存。
>

## Ollama 常用命令
终端输入 `Ollama`，输出如下：

```bash
Usage:
  ollama [flags]
  ollama [command]

Available Commands:
  serve       Start ollama
  create      Create a model from a Modelfile
  show        Show information for a model
  run         Run a model
  pull        Pull a model from a registry
  push        Push a model to a registry
  list        List models
  ps          List running models
  cp          Copy a model
  rm          Remove a model
  help        Help about any command

Flags:
  -h, --help      help for ollama
  -v, --version   Show version information
```

总结如下：

| 命令 | 描述 |
| --- | :--- |
| `ollama serve` | 启动 Ollama |
| `ollama create` | 从 Modelfile 创建模型 |
| `ollama show` | 显示模型信息 |
| `ollama run` | 运行模型 |
| `ollama pull` | 从注册表中拉取模型 |
| `ollama push` | 将模型推送到注册表 |
| `ollama list` | 列出所有模型 |
| `ollama ps` | 列出正在运行的模型 |
| `ollama cp` | 复制模型 |
| `ollama rm` | 删除模型 |
| `ollama help` | 显示任意命令的帮助信息 |


![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1738748584369-b38dadcb-561e-487a-b771-14861d05f27f.png)

| 标志 | 描述 |
| --- | --- |
| `-h, --help` | 显示 Ollama 的帮助信息 |
| `-v, --version` | 显示版本信息 |


终止 Ollama 模型推理服务，可以使用 `/bye`。

![](../images/C1-1-1.png)

注意：Ollama 进程会一直运行，如果需要终止 Ollama 所有相关进程，可以使用以下命令：

```bash
Get-Process | Where-Object {$_.ProcessName -like '*ollama*'} | Stop-Process
```

## 参考
参考：[https://github.com/datawhalechina/handy-ollama/blob/main/docs/C1/1.%20Ollama%20%E4%BB%8B%E7%BB%8D.md](https://github.com/datawhalechina/handy-ollama/blob/main/docs/C1/1.%20Ollama%20%E4%BB%8B%E7%BB%8D.md)

