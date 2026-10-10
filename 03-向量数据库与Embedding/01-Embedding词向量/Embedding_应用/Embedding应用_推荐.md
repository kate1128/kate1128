在很多APP或网站上都能看到推荐功能。比如在购物网站，每当选购一件商品后，系统就会推荐一些相关的产品。在这一小节中，就来做一个类似的应用，不过推荐的不是商品，而是文本，比如帖子、文章、新闻等。

<details class="lake-collapse"><summary id="ub371a8d9"><span class="ne-text" style="color: var(--jp-content-font-color1)">以新闻为例，先说一下基本逻辑：</span></summary><ol class="ne-ol"><li id="u1a318873" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">首先要有一个基础的文章库，可能包括标题、内容、标签等。</span></li><li id="u7ae43dc5" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">计算已有文章的</span><code class="ne-code"><span class="ne-text" style="color: var(--jp-content-font-color1)">Embedding</span></code><span class="ne-text" style="color: var(--jp-content-font-color1)">并存储。</span></li><li id="u488ad5b4" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">根据用户浏览记录，推荐和浏览记录最相似的文章。</span></li></ol></details>
看起来好像和前面的QA差不多，事实也的确如此，因为它们本质上都是相似匹配问题。只不过QA使用的是用户的Question去匹配已有知识库，而推荐是使用用户的浏览记录去匹配。

<details class="lake-collapse"><summary id="u290e9942"><span class="ne-text" style="color: var(--jp-content-font-color1)">但是很明显，推荐相比QA要更复杂一些，主要包括以下几个方面：</span></summary><ol class="ne-ol"><li id="u77aa364d" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">刚开始用户没有记录时的推荐（一般行业称为冷启动问题）。</span></li><li id="uf81ae2ad" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">除了相似还有其他要考虑的因素：比如热门内容、新内容、内容多样性、随时间变化的兴趣变化等等。</span></li><li id="u36767a45" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">编码（Embedding输入）问题：应该取标题呢，还是文章，还是简要描述或者摘要，还是都要计算。</span></li><li id="uec14d3ef" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">规模问题：推荐面临的量级一般会远超QA，除了横向扩展机器，是否能从流程和算法设计上提升效率。</span></li><li id="ub72e5eed" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">用户反馈对推荐系统的影响问题：用户反感或喜欢与文章本身并没有直接关系，比如用户喜欢体育新闻但讨厌中国足球。</span></li><li id="u8e755d92" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">线上实时更新问题。</span></li></ol></details>
当然，一个完整的线上系统要考虑的因素可能更多。列出这些只是希望设计一个方案时能够充分调研和考虑，同时结合实际情况进行。反过来说，可能并不需要考虑上面的每个因素。所以，在实际操作时一定要活学活用，充分理解需求后再动手实施。

<details class="lake-collapse"><summary id="u3a6261e9"><span class="ne-text" style="color: var(--jp-content-font-color1)">这里综合考虑上面的因素给一个比较简单的方案，但务必注意，其中每个模块的方案都不是唯一的。整体设计如下：</span></summary><ol class="ne-ol"><li id="u555dfc18" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">用户注册登录时，让其选择感兴趣的类型（如体育、音乐、时尚等），通过这一步将用户框在一个大的范围内，同时用来顺道解决冷启动问题。</span></li><li id="ud13ae731" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">给用户推荐内容时，在知道类别（用户注册时选择+浏览记录）后，应依次考虑时效性、热门程度、多样性等。</span></li><li id="u33474e27" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">考虑到性能问题，可以编码「标题+摘要」。</span></li><li id="u30d9b231" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">对大类别进一步细分，只在细分类别里进行相似度计算。</span></li><li id="u45aa4e5c" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">记录用户实时行为（如浏览Item、浏览时长、评论、收藏、点赞、转发等）。</span></li><li id="ucba5aec3" data-lake-index-type="0" style="text-align: left"><span class="ne-text" style="color: var(--jp-content-font-color1)">动态更新内容库，更新用户行为库。</span></li></ol></details>
<br/>color2
在具体实施时，使用最常用的流程线方案：**召回+排序**。

+ **召回**：通过各种不同属性或特征（如用户偏好、热点、行为等）先找到一批要推荐列表。
+ **排序**：根据多样性、时效性、用户反馈、热门程度等属性对结果进行排序。

<br/>

### 导包
```python
from dataclasses import dataclass
import pandas as pd
```

### 导入数据集
[AG_News.csv](https://www.yuque.com/attachments/yuque/0/2025/csv/2639475/1735872794589-b56ba1fd-ed33-47c6-9841-1ec1aedc5882.csv)

使用如下的数据集：[AG News Classification Dataset | Kaggle](https://www.kaggle.com/datasets/amananandrai/ag-news-classification-dataset?select=train.csv)

```python
df = pd.read_csv("./dataset/AG_News.csv")
df.shape
```

<details class="lake-collapse"><summary id="ueb2c5c4a"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="kn6Nz" class="ne-codeblock language-json"><code>(120000, 3)</code></pre></details>
```python
df.head()
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735627944609-8d2f189e-f4c2-47cb-834d-e3b4894fdd71.png)

```python
df["Class Index"].value_counts()
```

<details class="lake-collapse"><summary id="u32dcb1ca"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="SOWnB" class="ne-codeblock language-json"><code>Class Index
3    30000
4    30000
2    30000
1    30000
Name: count, dtype: int64</code></pre></details>
根据数据集介绍（[AG News Classification Dataset | Kaggle](https://www.kaggle.com/datasets/amananandrai/ag-news-classification-dataset?select=train.csv)），四个类型分别是：`1-World, 2-Sports, 3-Business, 4-Sci/Tech`，每个类型有30000条数据，共12万条。

###  做一个简单的流水线系统
接下来，将使用上面已经介绍的知识来做一个简单的流水线系统。

为了便于运行，依然取100条`sample`作为示例：

```python
# 指定种子，不然得不到下面一样的结果
sdf = df.sample(100, random_state=26187)

sdf["Class Index"].value_counts()
```

<details class="lake-collapse"><summary id="ube7b2a9e"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="DiDet" class="ne-codeblock language-json"><code>Class Index
3    27
2    26
1    24
4    23
Name: count, dtype: int64</code></pre></details>
```python
sdf.iloc[1]
```

<details class="lake-collapse"><summary id="u3e7bd062"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="dHNUU" class="ne-codeblock language-json"><code>Class Index                                                    2
Title                 Swimming: Shibata Joins Japanese Gold Rush
Description     ATHENS (Reuters) - Ai Shibata wore down Frenc...
Name: 5303, dtype: object</code></pre></details>
#### 首先维护一个用户偏好和行为记录
```python
from typing import List
@dataclass
class User:
    
    user_name: str

@dataclass
class UserPrefer:
    
    user_name: str
    prefers: List[int]


@dataclass
class Item:
    
    item_id: str
    item_props: dict


@dataclass
class Action:
    
    action_type: str
    action_props: dict


@dataclass
class UserAction:
    
    user: User
    item: Item
    action: Action
    action_time: str
```

行为记录：

```python
u1 = User("u1")
up1 = UserPrefer("u1", [1, 2])

# sdf.iloc[1] 正好是sport（类别为2）
i1 = Item("i1", {
    "id": 1, 
    "catetory": "sport",
    "title": "Swimming: Shibata Joins Japanese Gold Rush", 
    "description": "\
    ATHENS (Reuters) - Ai Shibata wore down French teen-ager  Laure Manaudou to win the women's 800 meters \
    freestyle gold  medal at the Athens Olympics Friday and provide Japan with  their first female swimming \
    champion in 12 years.", 
    "content": "content"
})

a1 = Action("浏览", {
    "open_time": "2023-04-01 12:00:00", 
    "leave_time": "2023-04-01 14:00:00",
    "type": "close",
    "duration": "2hour"
})

ua1 = UserAction(u1, i1, a1, "2023-04-01 12:00:00")
```

#### 计算所有文本的Embedding
```python
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np
```

```python
sdf["embedding"] = sdf.apply(lambda x: get_embedding(x.Title + x.Description), axis=1)
```

```python
# 智谱AI更改为 get_embedding_zhipu
sdf["embedding"] = sdf.apply(lambda x: get_embedding_zhipu(x.Title + x.Description), axis=1)
```

#### 处理一下召回
```python
import random
class Recall:
    
    def __init__(self, df: pd.DataFrame):
        self.data = df
    
    def user_prefer_recall(self, user, n):
        up = self.get_user_prefers(user)
        idx = random.randrange(0, len(up.prefers))
        return self.pick_by_idx(idx, n)
    
    def hot_recall(self, n):
        # 随机进行示例
        df = self.data.sample(n)
        return df
    
    def user_action_recall(self, user, n):
        actions = self.get_user_actions(user)
        interest = self.get_most_interested_item(actions)
        recoms = self.recommend_by_interest(interest, n)
        return recoms
    
    def get_most_interested_item(self, user_action):
        """
        可以选近一段时间内用户交互时间、次数、评论（相关属性）过的Item
        """
        # 就是sdf的第2行，idx为1的那条作为最喜欢（假设）
        # 是一条游泳相关的Item
        idx = user_action.item.item_props["id"]
        im = self.data.iloc[idx]
        return im
    
    def recommend_by_interest(self, interest, n):
        cate_id = interest["Class Index"]
        q_emb = interest["embedding"]
        # 确定类别
        base = self.data[self.data["Class Index"] == cate_id]
        # 此处可以复用QA那一段代码，用给定embedding计算base中embedding的相似度
        base_arr = np.array(
            [v.embedding for v in base.itertuples()]
        )
        q_arr = np.expand_dims(q_emb, 0)
        sims = cosine_similarity(base_arr, q_arr)
        # 排除掉自己
        idxes = sims.argsort(0).squeeze()[-(n+1):-1]
        return base.iloc[reversed(idxes.tolist())]
    
    def pick_by_idx(self, category, n):
        df = self.data[self.data["Class Index"] == category]
        return df.sample(n)
    
    def get_user_actions(self, user):
        dct = {"u1": ua1}
        return dct[user.user_name]
    
    def get_user_prefers(self, user):
        dct = {"u1": up1}
        return dct[user.user_name]
    
    def run(self, user):
        ur = self.user_action_recall(user, 5)
        if len(ur) == 0:
            ur = self.user_prefer_recall(user, 5)
        hr = self.hot_recall(3)
        return pd.concat([ur, hr], axis=0)
```

```python
r = Recall(sdf)
```

```python
rd = r.run(u1)
```

```python
# 共8个，5个用户行为推荐、3个热门
rd
```

![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735628097858-fa5a1f98-9144-4076-8410-8c5e415846a1.png)

<br/>color2
需要再次说明的是，上面的只是一个大致的流程，实际中有很多细节或优化点需要注意，比如：

1. 建数据库表（上面的`get_`其实都是查表）
2. 将`Item`、`User`和`Action`也进行`Embedding`，全部使用`Embedding`后再做召回
3. 对『感兴趣`get_most_interested_item`』更多的优化，考虑更多行为和反馈，召回更多不同类型条目
4. 性能和自动更新数据的考虑
5. 线上评测，A/B等

<br/>

可以发现，虽然只做了召回一步，但其中涉及到的内容已经远远不止之前QA那一点了，QA用到的东西可能只是其中一小部分。不过事无绝对，即便是QA任务也可能根据实际情况不同需要做很多优化，比如召回+排序。但总体来说，类似推荐这样比较综合的系统相对来说会更加复杂一些。

后面就是排序了，这一步需要区分不同的应用场景，可以做或不做，做的话（就是对刚刚得到的列表进行排序）也可以简单或复杂。比如简单地按发布时间，复杂的综合考虑多样性、时效性、用户反馈、热门程度等多种属性。具体操作时，可以直接按相关属性排序，也可以用模型排序。后面扩展



