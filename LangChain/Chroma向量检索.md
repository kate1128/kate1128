> 两种相似度检索算法：
>
> consine， 余弦距离：`vectordb.similarity_search(question,k=3)`
>
> MMR，最大边际相关性：`vectordb.max_marginal_relevance_search(question,k=3)`
>
> 看不出来这俩效果上有什么不一样的，Chroma的相似度搜索使用的是余弦距离
>

## 前提
跟着[搭建向量数据库](https://www.yuque.com/qiaokate/su87gb/dkg8d18dh04a6nw2#O7jcR)已经搭建好向量数据库了

## 相似度检索
Chroma的相似度搜索使用的是余弦距离，即：

$ similarity = cos(A, B) = \frac{A \cdot B}{\parallel A \parallel \parallel B \parallel} = \frac{\sum_1^n a_i b_i}{\sqrt{\sum_1^n a_i^2}\sqrt{\sum_1^n b_i^2}} $

其中$ a_i $、$ b_i $分别是向量$ A $、$ B $的分量。

需要数据库返回严谨的按余弦相似度排序的结果时可以使用`similarity_search`函数。

```python
question="什么是大语言模型"
```

```python
sim_docs = vectordb.similarity_search(question,k=3)
print(f"检索到的内容数：{len(sim_docs)}")
```

<details class="lake-collapse"><summary id="u4f5fc977"><span class="ne-text">output：</span></summary><pre data-language="python" id="F3CVI" class="ne-codeblock language-python"><code>检索到的内容数：3</code></pre></details>
```python
for i, sim_doc in enumerate(sim_docs):
    print(f"检索到的第{i}个内容: \n{sim_doc.page_content[:200]}", end="\n--------------\n")
```

<details class="lake-collapse"><summary id="ud232ebcf"><span class="ne-text">output：</span></summary><pre data-language="python" id="zeRv7" class="ne-codeblock language-python"><code>检索到的第0个内容: 
3
1.4.1
式(1.1) 和式(1.2) 的解释. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
4
第2 章模型评估与选择
5
2.1
经验误差与过拟合
. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
5
--------------
检索到的第1个内容: 
为主线，遇到自己推导不出来或者看不懂的公式时再来查阅南瓜书；
• 对于初学机器学习的小白，西瓜书第1 章和第2 章的公式强烈不建议深究，简单过一下即可，等你学得
有点飘的时候再回来啃都来得及；
• 每个公式的解析和推导我们都力(zhi) 争(neng) 以本科数学基础的视角进行讲解，所以超纲的数学知识
我们通常都会以附录和参考文献的形式给出，感兴趣的同学可以继续沿着我们给的资料进行深入学习；
• 
--------------
检索到的第2个内容: 
→_→
欢迎去各大电商平台选购纸质版南瓜书《机器学习公式详解》
←_←
目录
第1 章绪论
1
1.1
引言. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
1
1.2
基本术语
. . . . . . . . . . . . . . . . . . . .
--------------</code></pre></details>
## MMR检索
如果只考虑检索出内容的相关性会导致内容过于单一，可能丢失重要信息。

最大边际相关性 (`MMR, Maximum marginal relevance`) 可以帮助我们在保持相关性的同时，增加内容的丰富度。

核心思想是在已经选择了一个相关性高的文档之后，再选择一个与已选文档相关性较低但是信息丰富的文档。这样可以在保持相关性的同时，增加内容的多样性，避免过于单一的结果。

```python
mmr_docs = vectordb.max_marginal_relevance_search(question,k=3)
```

```python
for i, sim_doc in enumerate(mmr_docs):
    print(f"MMR 检索到的第{i}个内容: \n{sim_doc.page_content[:200]}", end="\n--------------\n")
```

<details class="lake-collapse"><summary id="uf6aea06d"><span class="ne-text">output：</span></summary><pre data-language="python" id="X6TJ9" class="ne-codeblock language-python"><code>MMR 检索到的第0个内容: 
3
1.4.1
式(1.1) 和式(1.2) 的解释. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
4
第2 章模型评估与选择
5
2.1
经验误差与过拟合
. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
5
--------------
MMR 检索到的第1个内容: 
23
→_→
配套视频教程：https://www.bilibili.com/video/BV1Mh411e7VU
←_←
--------------
MMR 检索到的第2个内容: 
在线阅读地址：https://datawhalechina.github.io/pumpkin-book（仅供第1 版）
最新版PDF 获取地址：https://github.com/datawhalechina/pumpkin-book/releases
编委会
主编：Sm1les、archwalker、jbb0523
编委：juxiao、Majingmin、MrBigFan、shanry、Ye
--------------</code></pre></details>
## 参考
本文对应源代码

[llm-universe/notebook/C3 搭建知识库/4.搭建并使用向量数据库.ipynb at main · datawhalechina/llm-universe](https://github.com/datawhalechina/llm-universe/blob/main/notebook/C3%20%E6%90%AD%E5%BB%BA%E7%9F%A5%E8%AF%86%E5%BA%93/4.%E6%90%AD%E5%BB%BA%E5%B9%B6%E4%BD%BF%E7%94%A8%E5%90%91%E9%87%8F%E6%95%B0%E6%8D%AE%E5%BA%93.ipynb)

