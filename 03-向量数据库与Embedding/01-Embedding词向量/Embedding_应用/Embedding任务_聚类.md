**聚类的意思是把彼此相近的样本聚集在一起，本质也是在使用一种表示和相似度衡量来处理文本**。比如有大量的未分类文本，如果事先能知道有几种类别，就可以用聚类的方法先将样本大致分一下。

使用Kaggle的DBPedia数据集：[DBPedia Classes](https://www.kaggle.com/datasets/danofer/dbpedia-classes?select=DBPEDIA_val.csv)。

这个数据集对一段文本会给出三个不同层次级别的分类标签，这里用第一层的类别。

[DBPEDIA_val.csv](https://www.yuque.com/attachments/yuque/0/2025/csv/2639475/1735969196883-d1eac843-8d38-4fbb-8907-cb3eda95bf56.csv)

```python
import pandas as pd
```

```python
df = pd.read_csv("./dataset/DBPEDIA_val.csv")
df.shape
```

<details class="lake-collapse"><summary id="u42fba397"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="cC3n7" class="ne-codeblock language-json"><code>(36003, 4)</code></pre></details>
```python
df.head()
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735627539697-29ce33af-72d5-4a9e-b4b6-31f300863814.png)

查看一下类别数量：

```python
df.l1.value_counts()
```

<details class="lake-collapse"><summary id="u591c6063"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="k8Wwy" class="ne-codeblock language-json"><code>l1
Agent             18647
Place              6855
Species            3210
Work               3141
Event              2854
SportsSeason        879
UnitOfWork          263
TopicalConcept      117
Device               37
Name: count, dtype: int64</code></pre></details>
数量有点多，随机采样200条：

```python
sdf = df.sample(200)
sdf.l1.value_counts()
```

<details class="lake-collapse"><summary id="ud7a9e487"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="kc3ph" class="ne-codeblock language-json"><code>l1
Agent             107
Place              36
Species            21
Work               14
Event              11
SportsSeason        8
TopicalConcept      2
UnitOfWork          1
Name: count, dtype: int64</code></pre></details>
为了便于观察，只保留3个数量差不多的类别：`Place`、`Work`和`Species`。

```python
cdf = sdf[
    (sdf.l1 == "Place") | (sdf.l1 == "Work") | (sdf.l1 == "Species")
]
cdf.shape
```

<details class="lake-collapse"><summary id="u4e36db29"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="fEy4c" class="ne-codeblock language-json"><code>(71, 4)</code></pre></details>
接下来先把文本变成向量：

```python
# 智谱AI更换为get_embedding_zhipu(x)
cdf["embedding"] = cdf.text.apply(lambda x: get_embedding(x))
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735627629061-0c13c754-1757-493f-a8c0-987319df4944.png)

接下来用PCA（主成分分析）给降维，将原来的向量从1536维降到3维，便于显示。

```python
from sklearn.decomposition import PCA
arr = np.array(cdf.embedding.to_list())
pca = PCA(n_components=3)
vis_dims = pca.fit_transform(arr)
cdf["embed_vis"] = vis_dims.tolist()
arr.shape, vis_dims.shape
```

<details class="lake-collapse"><summary id="u30851911"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="KMX4f" class="ne-codeblock language-json"><code>/var/folders/xt/mpcnl_9151dbq43hptdlv2jr0000gn/T/ipykernel_69847/3198218742.py:5: SettingWithCopyWarning: 
A value is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead

See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
  cdf[&quot;embed_vis&quot;] = vis_dims.tolist()</code></pre></details>
<details class="lake-collapse"><summary id="u9806757c"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="uvzA2" class="ne-codeblock language-json"><code>((71, 1536), (71, 3))</code></pre></details>
```python
%matplotlib inline
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(subplot_kw={"projection": "3d"}, figsize=(8, 8))
cmap = plt.get_cmap("tab20")
categories = sorted(cdf.l1.unique())

# 分别绘制每个类别
for i, cat in enumerate(categories):
    sub_matrix = np.array(cdf[cdf.l1 == cat]["embed_vis"].to_list())
    x=sub_matrix[:, 0]
    y=sub_matrix[:, 1]
    z=sub_matrix[:, 2]
    colors = [cmap(i/len(categories))] * len(sub_matrix)
    ax.scatter(x, y, z, c=colors, label=cat)

ax.legend(bbox_to_anchor=(1.2, 1))
plt.show();
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735615896417-f91034f5-0490-4969-8a2c-b9c68881c4c9.png)

可以比较明显的看出，三个不同类型的数据分别在不同的位置。



