> 使用Embedding Adaptors(嵌入适配器)进行查询结果的增强
>
> 其实就是查完把信息搞成数据集告诉模型，增强检索能力
>
> <br/>color2
遍历生成的查询和每个查询检索到的文档，计算查询和文档的嵌入向量。
>
+ 使用`evaluate_results`函数评估了每对查询和文档的相关性。
+ `adapter_query_embeddings`存储查询的嵌入向量
+ `adapter_doc_embeddings`存储文档的嵌入向量
+ `adapter_labels`存储对应的标签
>
<br/>


## Embedding Adaptors原理及准备工作
![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Advanced%20Retrieval%20for%20AI%20with%20Chroma/images/Embedding%20Adapter.png)

通过图片了解 `Embedding Adaptors`（嵌入适配器）的工作原理：

在一个系统中，用户通过应用程序(App)提交一个查询（例如“Revenue Growth”），查询会被发送到大型语言模型（LLM）中。而在检索部分，嵌入模型会根据查询生成嵌入向量，嵌入适配器会对这些嵌入进行进一步处理或转换。最后，系统会基于嵌入向量检索相关文档（例如“Annual Income...”），最终，LLM会使用这些信息来生成答案。

<br/>color2
`Embedding Adaptors`（嵌入适配器）是一种改变查询的嵌入，直接为了产生更好的检索结果。

在检索系统中插入一个额外的阶段，称为嵌入适配器。这发生在嵌入模型之后，但在得到最相关的结果之前。

<br/>

首先，加载需要的辅助函数，为了有效地训练模型，这次需要使用`torch`。

```python
from helper_utils import load_chroma, word_wrap, project_embeddings
from chromadb.utils.embedding_functions import SentenceTransformerEmbeddingFunction
import numpy as np
import umap
from tqdm import tqdm

import torch
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>D:\Anaconda3\Lib\site-packages\umap\distances.py:1063: NumbaDeprecationWarning: The 'nopython' keyword argument was not supplied to the 'numba.jit' decorator. The implicit default value for this argument is currently False, but it will be changed to True in Numba 0.59.0. See https://numba.readthedocs.io/en/stable/reference/deprecation.html#deprecation-of-object-mode-fall-back-behaviour-when-using-jit for details.
  @numba.jit()
D:\Anaconda3\Lib\site-packages\umap\distances.py:1071: NumbaDeprecationWarning: The 'nopython' keyword argument was not supplied to the 'numba.jit' decorator. The implicit default value for this argument is currently False, but it will be changed to True in Numba 0.59.0. See https://numba.readthedocs.io/en/stable/reference/deprecation.html#deprecation-of-object-mode-fall-back-behaviour-when-using-jit for details.
  @numba.jit()
D:\Anaconda3\Lib\site-packages\umap\distances.py:1086: NumbaDeprecationWarning: The 'nopython' keyword argument was not supplied to the 'numba.jit' decorator. The implicit default value for this argument is currently False, but it will be changed to True in Numba 0.59.0. See https://numba.readthedocs.io/en/stable/reference/deprecation.html#deprecation-of-object-mode-fall-back-behaviour-when-using-jit for details.
  @numba.jit()
D:\Anaconda3\Lib\site-packages\umap\umap_.py:660: NumbaDeprecationWarning: The 'nopython' keyword argument was not supplied to the 'numba.jit' decorator. The implicit default value for this argument is currently False, but it will be changed to True in Numba 0.59.0. See https://numba.readthedocs.io/en/stable/reference/deprecation.html#deprecation-of-object-mode-fall-back-behaviour-when-using-jit for details.</code></pre></details>
生成的嵌入函数并将所有内容加载到`chroma_collection`集合中。

```python
embedding_function = SentenceTransformerEmbeddingFunction()

chroma_collection = load_chroma(filename='./data/2024年北京市政府工作报告.pdf', \
                                collection_name='beijing_annual_report_2024', \
                                embedding_function=embedding_function,
                                langcode='zh')  # 注意中文文档将langcode改为'zh'
chroma_collection.count()
```

<details class="lake-collapse"><summary id="u83a3abe4"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="MZuKd" class="ne-codeblock language-json"><code>1028</code></pre></details>
投射的嵌入数据。

```python
embeddings = chroma_collection.get(include=['embeddings'])['embeddings']
umap_transform = umap.UMAP(random_state=0, transform_seed=0).fit(embeddings)
projected_dataset_embeddings = project_embeddings(embeddings, umap_transform)
```

<details class="lake-collapse"><summary id="u2ff99bf6"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="LfBHw" class="ne-codeblock language-json"><code>100%|██████████| 1028/1028 [21:21&lt;00:00,  1.25s/it]</code></pre></details>
与以前相同，配置`OPENAI_API_KEY`。

```python
import os
import openai
from openai import OpenAI

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) 
openai.api_key = os.environ['OPENAI_API_KEY']

openai_client = OpenAI()
```

## Creating a dataset（创建数据集）
与以前相同（增加不同的系统指示）。

```python
def generate_queries(model="gpt-3.5-turbo"):
    messages = [
        {
            "role": "system",
            # 系统角色消息，它指示模型扮演一个有帮助的、专业的政府年度报告研究助手角色。
            # 提供的内容是一个明确的任务描述，要求模型提出10到15个短问题，
            # 这些问题在分析年度报告时非常重要。
            # 这个指示还特别要求模型不要输出复合问题，即包含多个句子或连接词的问题。
            # 同时要求每个问题单独一行，行与行之间用换行符分隔。
            "content": "You are a helpful and expert financial research assistant. You help users analyze government work reports to better understand the government. "
            "Suggest 10 to 15 short questions that are important to ask when analyzing an annual report. "
            "Do not output any compound questions (questions with multiple sentences or conjunctions)."
            "Output each question on a separate line divided by a newline."
            "Use Chinese"
        },
    ]

    response = openai_client.chat.completions.create(
        model=model,
        messages=messages,
    )
    content = response.choices[0].message.content
    content = content.split("\n")
    return content
```

<font style="color:rgba(0, 0, 0, 0.87);">使用LLM进行生成的15个问题。</font>

```python
generated_queries = generate_queries()
for query in generated_queries:
    print(query)
```

<details class="lake-collapse"><summary id="u66466f5d"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="Pp8Xz" class="ne-codeblock language-json"><code>1. 政府在过去一年中主要实施了哪些经济政策？
2. 政府在年度报告中提及的财政支出重点是什么？
3. 政府在年度报告中提及的经济增长率是多少？
4. 政府在年度报告中是否提及了通货膨胀率？
5. 政府在年度报告中提及了哪些重点投资项目？
6. 政府在年度报告中是否提及了失业率和就业情况？
7. 政府在年度报告中是否提及了社会福利改革和提升？
8. 政府在年度报告中是否提及了环境保护和可持续发展？
9. 政府在年度报告中是否提及了减少贫困和改善生活水平的措施？
10. 政府在年度报告中提及的税收政策有哪些调整？
11. 政府在年度报告中是否提及了外贸政策和国际经济合作？
12. 政府在年度报告中提及了哪些金融改革和监管举措？
13. 政府在年度报告中是否提及了农业发展和农村改革？
14. 政府在年度报告中提及的基础设施建设情况如何？
15. 政府在年度报告中是否提及了科技创新和人才培养计划？</code></pre></details>
指定返回每个查询的前10个结果，并且要求结果包括相关文档和嵌入向量。

```python
results = chroma_collection.query(query_texts=generated_queries, n_results=10, include=['documents', 'embeddings'])
retrieved_documents = results['documents']
```

执行这段代码后，`retrieved_documents`中将包含每个查询问题检索到的文档列表。提示模型系统（给定的陈述是否与给定的相关查询）。

```python
def evaluate_results(query, statement, model="gpt-3.5-turbo"):
    messages = [
    {
        "role": "system",
        # 指示模型扮演一个有用的财务研究助手角色，
        # 并要求模型评估给定查询与声明的相关性，只输出"yes"或"no"。
        "content": "You are a helpful expert financial research assistant. You help users analyze financial statements to better understand companies. "
        "For the given query, evaluate whether the following satement is relevant."
        "Output only 'yes' or 'no'."
    },
    {
        "role": "user",
        "content": f"Query: {query}, Statement: {statement}"
    }
    ]
    # 使用OpenAI的chat API发送请求，传递模型名称、消息列表和最大令牌数。
    # 最大令牌数设置为1，因为期望的输出只有"yes"或"no"。
    response = openai_client.chat.completions.create(
        model=model,
        messages=messages,
        max_tokens=1
    )
    # 从响应中提取内容，即模型的输出。
    content = response.choices[0].message.content
    # 如果内容为"yes"，返回1，表示声明与查询相关；
    # 否则返回-1，表示声明与查询不相关。
    if content == "yes":
        return 1
    return -1
     
```

从检索结果中提取嵌入向量，然后为生成的查询计算嵌入向量。

```python
retrieved_embeddings = results['embeddings']
query_embeddings = embedding_function(generated_queries)
```

<br/>color2
`adapter_query_embeddings`、`adapter_doc_embeddings`和`adapter_labels`三个空列表将被用于存储适配器查询的嵌入向量、文档的嵌入向量，以及对应的标签。

<br/>

```python
adapter_query_embeddings = []
adapter_doc_embeddings = []
adapter_labels = []
```

遍历生成的查询和每个查询检索到的文档，计算查询和文档的嵌入向量，并使用`evaluate_results`函数评估了每对查询和文档的相关性。对于每个查询和文档的组合，将查询的嵌入向量、文档的嵌入向量和评估结果（作为标签）分别添加到`adapter_query_embeddings`、`adapter_doc_embeddings`和`adapter_labels`列表中。

```python
import time
from tqdm import tqdm

for q, query in enumerate(tqdm(generated_queries)):
    for d, document in enumerate(retrieved_documents[q]):
        adapter_query_embeddings.append(query_embeddings[q])
        adapter_doc_embeddings.append(retrieved_embeddings[q][d])
        adapter_labels.append(evaluate_results(query, document))

        # 因为使用免费的openAIAPI会出现调用限制等，
        # 这里使用time.sleep()进行减缓请求速度，从而保证因请求过快而被API服务限流。
        # 这里也可以使用除openAI API以外的API服务。
        time.sleep(30)
```

<details class="lake-collapse"><summary id="u8a1ce534"><span class="ne-text">output：</span></summary><pre data-language="json" id="wUUaN" class="ne-codeblock language-json"><code>100%|██████████| 15/15 [2:05:28&lt;00:00, 501.89s/it]</code></pre></details>
查看新的数据集的长度。

```python
len(adapter_labels)
```

<details class="lake-collapse"><summary id="u87b11dbf"><span class="ne-text">output：</span></summary><pre data-language="json" id="aG6y5" class="ne-codeblock language-json"><code>150</code></pre></details>
得到150个，这是15个查询和10个结果，并且每一个都标有相关性。使用的`torch`进行训练的嵌入适配器。将的数据集转换成`torch tensor`类型数据集，并全部装进`torch`。

使用这些`Tensor`类型数据训练一个分类模型，该模型预测给定查询和文档对是否相关（即标签为 1 表示相关，−1 表示不相关）。

```python
adapter_query_embeddings = torch.Tensor(np.array(adapter_query_embeddings))
adapter_doc_embeddings = torch.Tensor(np.array(adapter_doc_embeddings))
adapter_labels = torch.Tensor(np.expand_dims(np.array(adapter_labels),1))
```

创建了一个**数据集**，数据集封装**查询嵌入向量、文档嵌入向量和标签的张量**。

```python
dataset = torch.utils.data.TensorDataset(adapter_query_embeddings, adapter_doc_embeddings, adapter_labels)
```

## Setting up the model（设置模型）
设置的嵌入适配器模型。使用适配器矩阵来更新查询嵌入向量，并计算更新后的查询嵌入向量与文档嵌入向量之间的余弦相似度。

```python
def model(query_embedding, document_embedding, adaptor_matrix):
    updated_query_embedding = torch.matmul(adaptor_matrix, query_embedding)
    return torch.cosine_similarity(updated_query_embedding, document_embedding, dim=0)

```

使用均方误差损失来评估模型输出与真实标签之间差异。

```python
def mse_loss(query_embedding, document_embedding, adaptor_matrix, label):
    return torch.nn.MSELoss()(model(query_embedding, document_embedding, adaptor_matrix), label)
```

初始化一个适配器矩阵，这个矩阵将用于更新查询嵌入向量或者在模型中执行其他转换任务。

```python
mat_size = len(adapter_query_embeddings[0])
adapter_matrix = torch.randn(mat_size, mat_size, requires_grad=True)
```

建立训练循环，通过最小化损失函数来优化适配器矩阵。

```python
min_loss = float('inf')
best_matrix = None

for epoch in tqdm(range(100)):
    for query_embedding, document_embedding, label in dataset:
        loss = mse_loss(query_embedding, document_embedding, adapter_matrix, label)

        if loss < min_loss:
            min_loss = loss
            best_matrix = adapter_matrix.clone().detach().numpy()

        loss.backward()
        with torch.no_grad():
            adapter_matrix -= 0.01 * adapter_matrix.grad
            adapter_matrix.grad.zero_()
        
```

<details class="lake-collapse"><summary id="ubba84524"><span class="ne-text">output：</span></summary><pre data-language="json" id="RrNPT" class="ne-codeblock language-json"><code>0%|          | 0/100 [00:00&lt;?, ?it/s]D:\Anaconda3\Lib\site-packages\torch\nn\modules\loss.py:535: UserWarning: Using a target size (torch.Size([1])) that is different to the input size (torch.Size([])). This will likely lead to incorrect results due to broadcasting. Please ensure they have the same size.
    return F.mse_loss(input, target, reduction=self.reduction)
    100%|██████████| 100/100 [00:05&lt;00:00, 16.91it/s]</code></pre></details>
打印最佳损失值。

```python
print(f"Best loss: {min_loss.detach().numpy()}")
```

<details class="lake-collapse"><summary id="ub9cba46b"><span class="ne-text">output：</span></summary><pre data-language="json" id="qM8ua" class="ne-codeblock language-json"><code>Best loss: 0.3013603091239929</code></pre></details>
最佳损失值约为0.30，意味有了很大的进步。

接下来，看看适配器矩阵是如何影响的查询向量的。

使用`NumPy`的矩阵乘法函数计算`best_matrix`（一个`NumPy`数组）和`test_vector`（一个`PyTorch`张量）之间的矩阵乘法将结果转换为`NumPy`数组。

```python
test_vector = torch.ones((mat_size,1))
scaled_vector = np.matmul(best_matrix, test_vector).numpy()
```

将上一步保存在`scaled_vector`数组进行绘制条形图。

```python
import matplotlib.pyplot as plt
plt.bar(range(len(scaled_vector)), scaled_vector.flatten())
plt.show()
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736144928813-c9718888-d397-41bf-8f9e-21d430cae0a7.png)

在这里可以看到的每个维度仅由1组成的测试向量被拉伸和挤压，有些已经拉长了很多，而另一些则几乎为零。所以这意味着的嵌入适配器基本上已经确定与这些维度相关性。

使用`embedding_function`函数为生成的查询计算嵌入向量。使用之前获得的最佳适配器矩阵（`best_matrix`）来调整查询嵌入向量，产生`adapted_query_embeddings`。使用`umap`通过`project_embeddings`函数将原始查询嵌入向量和调整后的查询嵌入向量投影。

```python
query_embeddings = embedding_function(generated_queries)
adapted_query_embeddings = np.matmul(best_matrix, np.array(query_embeddings).T).T

projected_query_embeddings = project_embeddings(query_embeddings, umap_transform)
projected_adapted_query_embeddings = project_embeddings(adapted_query_embeddings, umap_transform)
```

<details class="lake-collapse"><summary id="u69d59bdd"><span class="ne-text">output：</span></summary><pre data-language="json" id="xlj3c" class="ne-codeblock language-json"><code>100%|██████████| 15/15 [00:09&lt;00:00,  1.64it/s]
100%|██████████| 15/15 [00:08&lt;00:00,  1.71it/s]</code></pre></details>
绘制原始查询嵌入向量、调整后的查询嵌入向量以及数据集嵌入向量的分布图。

```python
import matplotlib.pyplot as plt

plt.figure()
plt.rcParams['font.sans-serif'] = ['Microsoft YaHei']  # 设置全局字体为微软雅黑，显示中文
plt.scatter(projected_dataset_embeddings[:, 0], projected_dataset_embeddings[:, 1], s=10, color='gray')
plt.scatter(projected_query_embeddings[:, 0], projected_query_embeddings[:, 1], s=150, marker='X', color='r', label="original")
plt.scatter(projected_adapted_query_embeddings[:, 0], projected_adapted_query_embeddings[:, 1], s=150, marker='X', color='green', label="adapted")

plt.gca().set_aspect('equal', 'datalim')
plt.title("Adapted Queries")
plt.axis('off')
plt.legend()
```

<details class="lake-collapse"><summary id="ue9e8d689"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="KwTLB" class="ne-codeblock language-json"><code>&lt;matplotlib.legend.Legend at 0x2cebec6d0d0&gt;</code></pre></details>
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736144928822-78f6fe38-135b-477c-895d-e8106d1b4b16.png)

红色`X`标记表示原始查询嵌入向量的分布，绿色`X`标记表示调整后的查询嵌入向量的分布。

可以直接获得关于适配器矩阵调整效果的直观感受，的原始查询非常分散，但是的调整后的查询更接近某些特定的文档嵌入向量。

