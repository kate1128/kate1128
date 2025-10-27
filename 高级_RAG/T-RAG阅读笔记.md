论文地址：[https://arxiv.org/abs/2402.07483](https://arxiv.org/abs/2402.07483)

[2402.07483v2.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1737690458167-2cbd8948-1f4a-425e-b3ff-772194e41998.pdf)

<br/>color2
**<font style="color:rgb(36, 36, 36);">T-RAG = RAG + 微调 + 实体检测</font>**

<br/>

<font style="color:rgb(107, 107, 107);">T-RAG 方法的前提是将 RAG 架构与开源微调 LLM 和实体树向量数据库相结合。重点是上下文检索。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737698399245-a38c8d67-365a-4df3-9b4f-4424ce464ea1.png)

![画板](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1737703480843-20c0a979-2c93-4556-845c-890c05effddd.jpeg)

# <font style="color:rgb(36, 36, 36);"> 介绍</font>
<font style="color:rgb(36, 36, 36);">大型语言模型 (LLM) 越来越多地被应用于各个领域，包括</font><font style="color:rgb(36, 36, 36);">针对</font>**<font style="color:rgb(36, 36, 36);">私有企业文档的</font>****<font style="color:rgb(36, 36, 36);">问答</font>**<font style="color:rgb(36, 36, 36);">，其中</font>**<font style="color:rgb(36, 36, 36);">数据安全性</font>**<font style="color:rgb(36, 36, 36);">和</font>**<font style="color:rgb(36, 36, 36);">稳健性</font>**<font style="color:rgb(36, 36, 36);">至关重要。</font>

<font style="color:rgb(36, 36, 36);">检索增强生成 (</font><font style="color:rgb(36, 36, 36);"> </font>**<font style="color:rgb(36, 36, 36);">RAG</font>**<font style="color:rgb(36, 36, 36);"> </font><font style="color:rgb(36, 36, 36);">) 是构建此类应用程序的突出框架，但确保其稳健性需要大量定制。</font>

<font style="color:rgb(36, 36, 36);">本研究分享了部署 LLM 应用程序对私人组织文档进行问答的经验，使用名为</font>**<font style="color:rgb(36, 36, 36);">Tree-RAG（T-RAG）</font>**<font style="color:rgb(36, 36, 36);">的系统，该系统结合了</font>**<font style="color:rgb(36, 36, 36);">实体层次结构</font>**<font style="color:rgb(36, 36, 36);">以提高性能。</font>

<font style="color:rgb(36, 36, 36);">评估证明了这种方法的有效性，为现实世界的 LLM 应用提供了宝贵的见解。</font>

# <font style="color:rgb(36, 36, 36);">敏感资料</font>
<font style="color:rgb(36, 36, 36);">由于这些文件的敏感性，安全风险是主要关注点，因此在公共 API 上使用</font>**<font style="color:rgb(36, 36, 36);">专有 LLM 模型</font>**<font style="color:rgb(36, 36, 36);">是不切实际的，以避免数据泄露风险。</font>

<font style="color:rgb(36, 36, 36);">这需要使用</font><font style="color:rgb(36, 36, 36);">可以在现场部署的</font>**<font style="color:rgb(36, 36, 36);">开源模型。</font>**

<font style="color:rgb(36, 36, 36);">此外，有限的计算资源和基于可用文档的较小的训练数据集也带来了挑战。</font>

<font style="color:rgb(36, 36, 36);">此外，确保对用户查询做出可靠和准确的响应增加了复杂性，需要在这样的环境中部署强大的应用程序时进行大量的定制和决策。</font>

# <font style="color:rgb(36, 36, 36);">总结</font>
<font style="color:rgb(36, 36, 36);">这项研究让我感兴趣的是，研究人员开发了一个应用程序，</font>**<font style="color:rgb(36, 36, 36);">将检索增强生成 (RAG)</font>**<font style="color:rgb(36, 36, 36);">与</font>**<font style="color:rgb(36, 36, 36);">经过微调的开源大型语言模型 (LLM)</font>**<font style="color:rgb(36, 36, 36);">相结合，以生成响应。该模型使用来自组织文档的指令数据集进行训练。</font>

<font style="color:rgb(36, 36, 36);">他们引入了一种新的评估指标，称为</font>**<font style="color:rgb(36, 36, 36);">“正确-详细”</font>**<font style="color:rgb(36, 36, 36);">，旨在评估生成的响应的质量。该指标根据响应的正确性进行评估，同时还考虑包含超出原始问题范围的其他相关信息。</font>

# <font style="color:rgb(36, 36, 36);">T-RAG</font>
<font style="color:rgb(36, 36, 36);">以下是 Tree-RAG（T-RAG）的工作流程：</font>

<font style="color:rgb(36, 36, 36);">对于给定的用户查询，在向量数据库中搜索相关的文档块，该块作为 LLM 上下文学习的上下文参考。</font>

<font style="color:rgb(36, 36, 36);">如果查询中提到任何与组织相关的实体，则从实体树中提取有关实体的信息并将其添加到</font>_**<font style="color:rgb(36, 36, 36);">上下文中</font>**_<font style="color:rgb(36, 36, 36);">。经过微调的 Llama-2 7B 模型会根据呈现的数据生成响应。</font>

![画板](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1737701449408-2018df20-6f39-463b-b8fd-71afd34e1dd1.jpeg)

<br/>color2
<font style="color:rgb(107, 107, 107);">T-RAG 的一个特点是除了矢量数据库之外还包含一个实体树，用于上下文检索。</font>

<br/>

# <font style="color:rgb(36, 36, 36);">实体树</font>
<font style="color:rgb(36, 36, 36);">实体树存储了有关组织实体及其层次结构的详细信息。此树中的每个节点代表一个实体，父节点表示其各自的组成员身份。</font>

<br/>color2
<font style="color:rgb(36, 36, 36);">在检索过程中，框架利用</font>**<font style="color:rgb(36, 36, 36);">实体树</font>**<font style="color:rgb(36, 36, 36);">来增强从向量数据库检索到的上下文。</font>

<br/>

_<font style="color:rgb(36, 36, 36);">实体树搜索和上下文生成的过程如下：</font>_

1. <font style="color:rgb(36, 36, 36);">首先，解析器模块扫描用户查询以查找与组织内的实体名称相对应的关键字。</font>
2. <font style="color:rgb(36, 36, 36);">一旦识别出一个或多个匹配项，就会从树中提取有关每个匹配实体的详细信息。</font>
3. <font style="color:rgb(36, 36, 36);">这些细节被转换成文本陈述，提供有关实体及其在组织层次结构中的位置的信息。</font>
4. <font style="background-color:rgb(232, 243, 232);">随后，将该信息与从向量数据库检索到的文档块合并以构建上下文。</font>
5. <font style="color:rgb(36, 36, 36);">通过采用这种方法，当用户查询时，模型可以获取有关实体及其在组织内的层次定位的相关信息。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1737698399299-8be1f3bd-54ff-417f-992e-bace7a480f24.png)

![画板](https://cdn.nlark.com/yuque/0/2025/jpeg/2639475/1737702746293-a9cdaa7e-a43b-478c-ac12-db3fff4fa662.jpeg)

<font style="color:rgb(36, 36, 36);">考虑上图，上下文生成的检索过程涉及利用组织结构图中的说明性示例来演示如何执行树搜索和检索。</font>

<font style="color:rgb(36, 36, 36);">除了获取上下文文档之外，spaCy 库还使用自定义规则来识别组织内的命名实体。</font>

<font style="color:rgb(36, 36, 36);">如果查询包含一个或多个这样的实体，则从树中提取有关实体层次位置的相关信息并将其转换为文本语句。然后将这些语句与检索到的文档一起合并到上下文中。</font>

<font style="color:rgb(36, 36, 36);">但是，如果用户的查询没有提及任何实体，则会省略树搜索，并且仅利用检索到的文档的上下文。</font>

# <font style="color:rgb(36, 36, 36);">综上所述</font>
<font style="color:rgb(36, 36, 36);">我发现这项研究非常有趣，因为它结合了 RAG 和微调。它利用本地托管的开源模型来解决数据隐私问题，同时解决推理延迟、令牌使用成本以及区域和地理可用性问题。</font>

<font style="color:rgb(36, 36, 36);">实体如何通过 spaCy 框架用于实体搜索和上下文生成也很有趣。事实上，这不仅仅是一个研究成果，而是基于构建用于实际用途的 LLM 应用程序的经验所获得的经验教训。</font>

