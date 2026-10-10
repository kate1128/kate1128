先看 token 是什么以及文本 token 化产生的问题：

[什么是 token](https://www.yuque.com/qiaokate/su87gb/ozsbnrrkz5awc15r)

## `Embedding`的主要思想
<br/>color2
1. **把特征固定在某一个维度D**，比如256、300、768等等，总之不再是词表那么大的数字。这就**避免了维度过高的问题**。

<br/>

> 比如说 3000 维映射成 30 维，以某种规则去映射
>

<br/>color2
2. **利用自然语言文本的上下文关系学习一个稠密表示**。

<br/>

> 原来全是 1 和 0，现在全变成小数了，这句话的向量是根据上下文动态变化的
>

<br/>color2
也就是说，每个Token的表示不再是预先算好的了，而是在过程中学习到的，元素也不再是很多个0，而是每个位置都有一个小数，这D个小数构成了一个Token表示。至于D个特征到底是什么，不知道，也不重要。只需要知道这D个小数就表示这个Token。

<br/>

还是继续以前面的例子来说明，这时候词表的表示变成下面这样了：

> 我 0.xxx0, 0.yyy0, 0.zzz0, ... D个小数  
们 0.xxx1, 0.yyy1, 0.zzz1, ... D个小数  
相 0.xxx2, 0.yyy2, 0.zzz2, ... D个小数  
信 0.xxx3, 0.yyy3, 0.zzz3, ... D个小数  
……下面省略
>


⚠️这些小数怎么来的？

随机来的。

<br/>

就像下面这样：

### 把特征固定在某一个维度D
```python
import numpy as np
```

[什么是 numpy](https://www.yuque.com/qiaokate/su87gb/pubs5insfa4pgnu4)

```python
rng = np.random.default_rng(42)
# 词表大小N=16，维度D=256
table = rng.uniform(size=(16, 256))
table.shape
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>(16, 256)</code></pre></details>
打印 `table`：

```python
table
```

<details class="lake-collapse"><summary id="u94182f67"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="z80wh" class="ne-codeblock language-json"><code>array([[0.77395605, 0.43887844, 0.85859792, ..., 0.24783956, 0.23666236,
        0.74601428],
       [0.81656876, 0.10527808, 0.06655886, ..., 0.11585672, 0.07205915,
        0.84199321],
       [0.05556792, 0.28061144, 0.33413004, ..., 0.00925978, 0.18832197,
        0.03128351],
       ...,
       [0.50647331, 0.22303613, 0.94414565, ..., 0.79202324, 0.40169878,
        0.72247782],
       [0.9151384 , 0.80071297, 0.39044651, ..., 0.03994193, 0.79502741,
        0.28297954],
       [0.68255979, 0.64272531, 0.65262805, ..., 0.18645529, 0.21927175,
        0.32320729]])</code></pre></details>
### 利用自然语言文本的上下文关系学习一个稠密表示
在模型训练过程中，会**根据不同的上下文不断地更新这个参数**，最后模型训练完后得到的这个矩阵就是`Token`的表示。完全可以把它当成一个黑盒子，输入一个`X`，根据标签`Y`不断更新参数，最终就得到一组参数，这些参数的名字就叫「**模型**」。

这种表示方法在深度学习早期（2013-2015年左右）比较流行，不过由于这个**矩阵训练好后就固定不变**了，这在有些时候就**不合适**。

**句子才是语义的最小单位**，因此相比`Token`，更加应该关注和需要句子的表示，期望可以**根据不同上下文动态地获得句子表示**。这中间当然经历了比较多的探索，一直到如今的大模型时代，对模型输入任意一句话，它都能返回一个非常不错的表示，而且依然是固定长度的向量。

> 如果对这方面感兴趣，可以进一步阅读
>
> + [句子表征综述 | Yam](https://yam.gift/2022/03/27/NLP/2022-03-27-Sentence-Representation-Summarization/)
> + [NLP 表征的历史与未来 | Yam](https://yam.gift/2020/12/12/NLP/2020-12-12-NLP-Representation-History-Future/)
>

## 总结
<br/>color2
Embedding本质就是**一组稠密向量**，用来表示一段文本（可以是字、词、句、段等），获取到这个表示后，就可以进一步做一些任务。

<br/>



