> 这节就是用到 Weaviate 和 Cohere 的 API key，要自己申请一下
>
> 然后就是使用关键词检索（Keyword Search）并利用Weaviate数据库回答问题。
>

#  环境配置
1. 安装包

```python
!pip install cohere
!pip install weaviate-client
!pip install python-dotenv
```

2. 打开本文件目录下的`.env `文件，替换为自己的 API Key

```python
WEAVIATE_API_KEY="your_weaviate_api_key"
WEAVIATE_API_URL="your_weaviate_api_url"
COHERE_API_KEY="your_cohere_api_key"
```

3. 导入本地 `.env` 文件

```python
import os
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # 读取本地 .env 文件
```

# Weaviate 数据库
Weaviate 是一个开源数据库。它具有关键词检索功能，也具有依赖于语言模型的向量检索功能。

## 加载配置
```python
import weaviate
auth_config = weaviate.auth.AuthApiKey(
    api_key=os.environ['WEAVIATE_API_KEY'])  # 获取环境变量中的 Weaviate API 密钥，进行身份验证。
```

## 连接数据库
<br/>color2
字段解释：

+ `weaviate.Client() `：`Weaviate` 客户端对象。
+ `url` ：`Weaviate` 客户端的 URL 属性。这个 URL 指定了与 `Weaviate` 服务进行通信的位置。
+ `auth_client_secret` ：`Weaviate`  客户端的身份验证密钥属性
+ `additional_headers` ：额外的请求头信息。

<br/>

```python
client = weaviate.Client(
    url=os.environ['WEAVIATE_API_URL'],  
    auth_client_secret=auth_config,
    additional_headers={
        "X-Cohere-Api-Key": os.environ['COHERE_API_KEY'],  # 这里添加了一个名为 X-Cohere-Api-Key 的请求头，其值为环境变量中的 Cohere API 密钥。
    }
)
```

运行下面这行代码后，确保客户端已经准备好并连接上了。如果返回 True，那就意味着本地 Weaviate 客户端能够连接到远程 Weaviate 数据库。

```python
print(client.is_ready())
```

# 关键词检索
假设有一个 query，eg：“草是什么颜色？”，在一个非常小的文档集中进行搜索，其中包含了以下五个句子：“明天是星期六”，“草是绿色的”，“加拿大的首都是渥太华”，“天空是蓝色的”，“鲸鱼是哺乳动物”。

关键词检索的工作原理是比较 **query** 和**文档**之间有多少共同的单词。如果我们比较 query 和第一句话之间有多少共同的单词，可以发现它们只有一个共同的词：“是”。 可以统计这个文档集中每个句子的词语计数情况。然后可以发现第二个句子与 query 有最多的共同词，因此关键词检索可能会将其作为答案返回。

## 构建关键词检索函数`keyword_search`
创建查询数据库的函数"`keyword_search`"

```python
def keyword_search(query,
                   results_lang='en',
                   properties=["title", "url", "text"],
                   num_results=3):
    """
    关键词搜索函数

    参数：
    query：要搜索的关键词
    results_lang：搜索结果的语言，默认为英文（'en'）
    properties：要返回的属性列表，默认为 title（标题）、url 和 text（文本）
    num_results：要返回的结果数量，默认为 3 个

    返回：
    搜索结果列表
    """

    # 构建过滤器，限制搜索结果的语言
    where_filter = {
        "path": ["lang"],
        "operator": "Equal",
        "valueString": results_lang
    }

    # 发送查询请求，获取搜索结果
    response = (
        client.query.get("Articles", properties)
        # 使用了 BM25 算法 ，来计算 query 和文档之间的相关性得分。
        .with_bm25(
            query=query
        )
        .with_where(where_filter)
        .with_limit(num_results)
        .do()
    )

    # 提取搜索结果
    result = response['data']['Get']['Articles']
    return result
```

<br/>color2
这个函数接受四个参数：

+ `query`（要搜索的关键词）
+ `results_lang`（搜索结果的语言，默认为英文）
+ `properties`（要返回的属性列表，默认为标题、URL 和文本）
+ `num_results`（要返回的结果数量，默认为 3 个）

<br/>

代码解释：

1. 构建过滤器： 在函数内部，首先构建了一个过滤器 `where_filter` ，用于限制搜索结果的语言。这里限制搜索结果的语言与 `results_lang` 参数相同。
2. 发送查询请求和获取搜索结果： 使用 `Weaviate` 客户端对象 `client` ，向数据集中“`Articles`”类型的数据进行查询操作。查询结果存储在 response 变量中。查询操作包括以下几部分：
    1. 使用 `properties` 参数指定的属性列表，来确定我们的搜索结果中需要包含的内容
    2. 调用` .with_bm25(query=query) `将关键词查询 query 添加到查询请求中，它将使用 BM25 算法对 query 和文章内容的相关性进行加权计算，以提高搜索结果的相关性。
    3. 调用`.with_where(where_filter) `来将语言过滤器添加到查询请求中，以限制搜索结果的语言，
    4. 调用`.with_limit(num_results)` 将结果数量限制添加到查询请求中，以指定要返回的搜索结果数量
    5. 最后调用` .do() `来执行查询操作。
3. 提取搜索结果并返回： 从 `response` 中提取搜索结果，具体提取了 `response['data']['Get']['Articles']` 部分，存储在 `result` 变量中。

## BM25 算法
上面代码块 29 行可以发现，关键词检索函数中使用了 **BM25 算法** ，来计算 query 和文档之间的相关性得分。

首先介绍一下` TF-IDF `算法，它是一种常见的文本分析技术，它通过将词频（TF）与逆文档频率（IDF）相乘，来衡量文档中词语的重要性。词频是每个词项（term）在该文档中出现的频率。逆向文件频率是一种表征词项重要性的度量。然而，TF-IDF 算法在计算文档与查询之间的相关性时存在一些不足，例如未考虑文档长度和文档频率对权重的影响。

而 BM25 算法是一种经典的搜索算法，它是对传统的 TF-IDF 算法的改进。BM25 通过考虑词频、文档长度和词项的逆文档频率来计算文档与查询之间的相关性得分，从而更准确地评估文档的相关性。这种改进使得 BM25 在信息检索任务中表现更出色，能够更好地满足实际应用的需求。

### BM25 算法的关键要点
#### 文档与 query 之间的相关性计算
BM25 算法通过计算文档中的每个词项与 `query` 中的词项之间的相关性得分来衡量文档的相关性。这一相关性得分是基于每个词项在文档中的频率（TF）以及在整个文档集合中的逆文档频率（`IDF`）计算得出的。

#### TF（词频）因子
BM25 考虑了词频的影响，但相比于 TF-IDF ，它使用了一种更平滑的方式来处理词频。具体来说，BM25使用了如下的词频计算公式：

`TF(t,d)=f(t,d)×(k1+1)f(t,d)+k1×(1−b+b×|d|avgdl)`

其中：

    - `f(t,d)`表示词项
    - `d`表示文档 `d`的长度。
    - `avgdl` 表示平均文档长度。
    - `k1` 和`b`是调节参数，用于平衡 TF 和文档长度的影响。

#### IDF（逆文档频率）因子
与 TF-IDF 类似，BM25 也考虑了词项的逆文档频率。IDF 的计算方式可以是传统的 `log⁡(N−n+0.5n+0.5)`，也可以是其他变种形式。 N表示文档集合中的文档总数。

    - N 表示文档集合中的文档总数。
    - n 表示包含词项 t 的文档数。
4. 调节参数 k1 和 b 是 BM25 中的两个调节参数，用于平衡 TF 和文档长度的影响。通常情况下，它们的选择取决于具体的应用和数据集。
5. 最终得分计算

最终的 BM25 得分是对每个查询词项的 TF 和 IDF 加权求和，然后将所有查询词项的得分相加得到文档的总得分。

## 使用关键词检索函数
现在让我们使用这个关键词检索函数，并传递一个 query 给它。 假设想搜索`"What is the most viewed televised event?"`，即“收视率最高的电视节目是什么？”将 query 传递给函数，然后将其打印出来

观察搜索结果。它是一大段文本，不太方便我们浏览。但可以看到它是一个字典列表。所以定义一个函数，遍历键值对，以更好的方式打印它。

```python
def print_result(result):
    """
    打印搜索结果。

    参数:
    result: 搜索结果列表，每个元素是一个字典，包含要打印的键值对。

    返回:
    无返回值，仅打印结果。

    """
    for i, item in enumerate(result):
        print(f'item {i}')  # 打印索引
        for key in item.keys():
            print(f"{key}:{item.get(key)}")  # 打印键值对
            print()  # 打印空行，增加可读性
        print()  # 打印空行，用于分隔不同的字典
```

调用这个函数，查看搜索结果

```python
print_result(keyword_search_results)
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>item 0 text:The most active Gamergate supporters or &quot;Gamergaters&quot; said that Gamergate was a movement for ethics in games journalism, for protecting the &quot;gamer&quot; identity, and for opposing &quot;political correctness&quot; in video games and that any harassment of women was done by others not affiliated with Gamergate. They argued that the close relationships between journalists and developers demonstrated a conspiracy among reviewers to focus on progressive social issues. Some supporters pointed to what they considered disproportionate praise for games such as &quot;Depression Quest&quot; and &quot;Gone Home&quot;, which feature unconventional gameplay and stories with social implications, while they viewed traditional AAA games as downplayed. False claims of the &quot;ethics in game journalism&quot; had started as early as 2012, when Geoff Keighley was accused of such unethical behavior when he was presenting information about &quot;Halo 4&quot; among advertisements for Mountain Dew and Doritos, an event called &quot;Doritosgate&quot; in the gamer culture.
title:Gamergate (harassment campaign)
url:https://en.wikipedia.org/wiki?curid=43758363

item 1 text:&quot;Rolling Stone&quot; stated Jackson's Super Bowl performance &quot;is far and away the most famous moment in the history of the Super Bowl halftime show&quot;. &quot;PopCrush&quot; called the performance &quot;one of the most shocking moments in pop culture&quot; as well as a &quot;totally unexpected and unforgettable moment&quot;. &quot;Gawker&quot; ranked the performance among the most recent of the &quot;10 Shows that Advanced Sex on Television&quot;, commenting the set &quot;had all the elements of a huge story&quot; and &quot;within seconds the world searched furtively for pictures&quot;, concluding &quot;it remains so ubiquitous, it's impossible to look at a starburst nipple shield without thinking &quot;Janet Jackson&quot;&quot;. &quot;E! Online&quot; ranked it among the top ten most shocking celebrity moments of the prior two decades. A study of television's most impactful moments of the last 50 years conducted by Sony Electronics and the Nielsen Television Research Company ranked Jackson's Super Bowl performance at #26. The incident was the only Super Bowl event on the list and the highest music and entertainment event aside from the death of Whitney Houston. TV Guide Network ranked it at #2 in a 2010 special listing the &quot;25 Biggest TV Blunders&quot;. &quot;Complex&quot; stated &quot;It's the Citizen Kane of televised nip-slips—so unexpected, and on such a large stage, that nothing else will ever come close. If Beyoncé were to whip out both breasts and put on a puppet show with them when she performs this year in New Orleans, it would rate as just the second most shocking Super Boob display. Janet's strangely ornamented right nipple is a living legend, and so is Justin Timberlake's terrified reaction.&quot; Music channel Fuse listed it as the most controversial Super Bowl halftime show, saying the &quot;revealing performance remains (and will forever remain) the craziest thing to ever happen at a halftime show. Almost immediately after the incident, the FCC received a flood of complaints from parents who just wanted their children to enjoy a nice, wholesome three hours of grown men inflicting damaging and long-lasting pain on each other for sport. Halftime shows would never be the same.&quot; Patrick Gipson of &quot;Bleacher Report&quot; ranked it as #1 in its list of the most &quot;Jaw Dropping Moments of the Last Decade&quot;, stating Janet &quot;changed the landscape of live television forever&quot;. Gipson explained &quot;It prompted a million mothers to cover their eyes, fathers and sons to jump out of their seats in shock and numerous sanctions by the Federal Communications Commission, including a US$550,000 fine against CBS. Talk about a halftime show that will be hard to top.&quot; The incident was also declared &quot;the most memorable Super Bowl halftime show in history&quot;, as well as &quot;the most controversial&quot;, adding &quot;you can't talk about this halftime show, or any subsequent halftime show from here to eternity, without mentioning the wardrobe malfunction&quot;.
title:Super Bowl XXXVIII halftime show controversy
url:https://en.wikipedia.org/wiki?curid=498971

item 2 text:West Germany (established in May 1949) was not eligible for the 1950 World Cup (the first after the war), and so all preparations were made with a view toward the 1954 matches in Bern, Switzerland. By that time Adidas's football boots were considerably lighter than the ones made before the war, based on English designs. At the World Cup Adi had a secret weapon, which he revealed when West Germany made the finals against the overwhelmingly favored Hungarian team, which was undefeated since May 1950 and had defeated West Germany 8–3 in group play. Despite this defeat, West Germany made the knock-out rounds by twice defeating Turkey handily. The team defeated Yugoslavia and Austria to reach the final (a remarkable achievement), where the hope of many German fans was simply that the team &quot;avoid another humiliating defeat&quot; at the hands of the Hungarians. The day of the final began with light rain, which brightened the prospects of the West German team who called it &quot;&quot;Fritz Walter-Wetter&quot;&quot; because the team's best player excelled in muddy conditions. Dassler informed Herberger before the match of his latest innovation—&quot;screw in studs.&quot; Unlike the traditional boot which had fixed leather spike studs, Dassler's shoe allowed spikes of various lengths to be affixed depending on the state of the pitch. As the playing field at Wankdorf Stadium drastically deteriorated, Herberger famously announced, &quot;Adi, screw them on.&quot; The longer spikes improved the footing of West German players compared to the Hungarians whose mud-caked boots were also much heavier. The West Germans staged a come from behind upset, winning 3-2, in what became known as the &quot;Miracle in Bern.&quot; Herberger publicly praised Dassler as a key contributor to the win, and Adidas's fame rose both in West Germany, where the win was considered a key post-war event in restoring German self-esteem and abroad, where in the first televised World Cup final viewers were introduced to &quot;the ultimate breakthrough.&quot;
title:Adolf Dassler
url:https://en.wikipedia.org/wiki?curid=2373164</code></pre></details>
得到的第一个结果是一段文本。由正文（text）、标题（title）、地址（url）组成。想要查找的是收视率最高的电视节目，这个结果看起来并不完全正确，但包含了许多关键词。 第二个结果是关于“超级碗”的文章，它可能是一个收视率很高的电视节目。 然后在这里还有第三个结果，它提到了“世界杯”。 可以看到每篇文章的 URL。

再举一个中文的例子，搜索“中国”。

```python
query = "中国"
keyword_search_results = keyword_search(query，results_lang='zh')  # 中文用“zh”
print_result(keyword_search_results)
```

<details class="lake-collapse"><summary id="u6ca332db"><span class="ne-text">output：</span></summary><pre data-language="json" id="AWXlB" class="ne-codeblock language-json"><code>item 0 text:古时“中国”含义不一：或指天子所在的京师为“中国”。《》：“惠此中国，以绥四方。”毛传：“中国，京师也。”《》：“夫而后之中国，践天子位焉。”《集解》：“刘熙曰；‘帝王所都为中，故曰中国’。”或指华夏族、汉族地区为中国（以其在四夷之中）。《》：“《小雅》尽废，则四夷交侵，中国微矣。”又《》：“是以声名洋溢乎中国，施及蛮貊。”而华夏族多建都于黄河南、北，因称其地为“中国”，与“中土”、“中原”、“中州”、“中夏”、“中华”含义相同，初时本指今河南省北部、山西省南部和陕西省南部及附近地区，后来中原王朝的活动范围扩大，黄河中下游一带，也被称为“中国”。或指统辖中原之国，《》：“盂达于是连吴固蜀，潜图中国。”，也把所统辖的地区，包括不属于黄河流域的地方，也全部称为“中国”。《》：“其后秦遂以兵灭六国，并中国。”在统一的情况下，中央王朝常自称为“中国”；而分裂时期，“中国”也能指稱黄河中下游地区（即中原）或延續正統的王朝。《晉書·載記第十四》苻堅對其弟苻融言“劉禪可非漢之遺祚；然終為中國之所並”。此處「中国」指三國時期于華北地區的魏国，原因是魏繼承漢的正統。此外，古時「中國」一詞也具有單獨代指漢民族的用法。
title:中國的稱號
url:https://zh.wikipedia.org/wiki?curid=527278

item 1 text:瑞士银行（中国）有限公司为瑞银子公司，其前身是2004年成立的瑞士银行有限公司北京分行。2012年3月，中国银监会发布《中国银监会关于由瑞士银行有限公司在中国境内分支机构改制的瑞士银行（中国）有限公司开业的批复》，批准瑞士银行（中国）有限公司，英文名为UBS (China) Limited 作为由瑞银单独出资的外商独资银行开业，北京市西城区金融大街7号英蓝国际金融中心1217－1230单元为注册营业地址；注册资本为20亿元人民币，近85%由瑞士银行有限公司拨付，其余部分由原瑞士银行有限公司在中国境内分行的营运资金划转；核准PETER ERIC WALSHE瑞士银行（中国）有限公司董事长的任职资格、金纪湘（SIMON JIXIANG JIN）瑞士银行（中国）有限公司行长的任职资格。允许其经营对各类客户的外汇业务及对除中国境内公民以外客户的人民币业务。 2012年7月，位于北京西城区的瑞士银行（中国）有限公司正式开业。
title:瑞银集团
url:https://zh.wikipedia.org/wiki?curid=556866

item 2 text:独特的外型更引起了时尚界的注意，使得她在Puma的Suede 50活动，Levi’s的新年TVC，以及路易威登2018年的展览和活动中悉数登场；并出现在多个亚洲尖端时尚生活杂志的版面，包括Vogue me（中国）、时尚芭莎（中国）、Nylon（中国/日本）、Ellemen睿士、大都市Numéro（中国）、红秀GRAZIA。（凤凰网音乐）title:刘柏辛
url:https://zh.wikipedia.org/wiki?curid=6070776</code></pre></details>
总共得到了三个结果，每一个结果都是一段文本。由正文（text）、标题（title）、地址（url）组成。想要寻找的是跟中国有关的信息，可以看到三个结果里都多次出现了“中国”这个关键词。

可以尝试修改查询，以查看数据集中还有什么内容。 在这里，还可以尝试查看属性。下面是构建该数据集时使用的属性列表，这些属性都存储在数据库中

```python
properties = ["text", "title", "url", "views", "lang"]
# 其他可以尝试的语言：en, de, fr, es, it, ja, ar, zh, ko, hi
```

可以通过查看属性 views 来查看维基百科页面收到的观看次数，并且用它来进行筛选或排序。还可以使用其他语言进行筛选。可以尝试的其他语言包括英语、德语、法语、西班牙语、意大利语、日语、阿拉伯语、中文、韩语和印地语。只需输入其中一种语言，并将其传递给关键词检索，它将以该语言提供结果。但是，在选择语言时，请注意所选语言的文档与 query 之间必须具有共同出现的关键词。这样才能获得相关的结果。

BM25 只需要有一个共同出现的关键词，就可以将其评分为某种程度上相关。而且 query 和文档共享的词越多，文档中重复的次数越多，得分就越高。

以上是一些高级示例。它展示了用 query 查询数据库，并随后查看结果的过程。

# 关键词检索的更深理解
<br/>color2
接下来从更高的层次上回顾一下**搜索**。如下图所示，搜索的主要组成部分包括 **Query** 、**搜索系统（Search System）** 和 **搜索系统可以访问的之前处理过的数据库（Document Archive）**。搜索系统会按照数据库中的数据与 query 的相关性从高到低的顺序，给出一系列搜索结果作为响应。

<br/>

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/2-1.png)

<br/>color2
可以将 **搜索系统（Search System）** 视为具有两个阶段。

+ 第一阶段通常是 `**Retrieval**`**（检索或搜索）** 阶段

`Retrieval`**（检索或搜索）** 会根据某种排序算法（比如 TF-IDF、BM25 等）得出初始排序结果，但可能并不总是最符合用户意图的顺序。

+ 之后还有一个称为 **Reranking（重排）** 的阶段

而 `Reranking` 指的是在 `Retrieval` 返回初始排序结果后，对这些结果进行进一步排序的过程。`Reranking` 可以基于各种因素，例如语义相关性、用户偏好、领域特定信息等。`Reranking` 通常是必需的，因为希望包含或者引入除了文本相关性之外的其他信息。

<br/>

此外，第一阶段 Retrieval 的实现通常需要 `**Inverted Index（倒排索引）**` 。`Inverted Index`是信息检索领域中一种常用的数据结构，用于快速查找包含特定词项（term）的文档。它的基本思想是将文档集合中每个词项与包含该词项的文档列表建立关联，从而实现通过词项快速定位包含该词项的文档。

在下图中，可以看到 Inverted Index 具有两列，一列是关键词，另一列是包含该关键词的文档 ID 。这样的结构使得搜索引擎能够快速地定位包含用户查询词的文档，从而支持高效的信息检索。在实际的应用场景中， Inverted Index 还会记录该词项在文档中的位置信息和出现的频率等等。

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/2-2.png)

输入 `query = "What color is the sky?"` 时，可以看到在 `Inverted Index` 中，单词`"color"`对应文档 804 ，而单词`"sky"`也对应文档 804 。因此，804 将在第一阶段检索的结果中获得很高的评分。

# 关键词检索的限制
如下图所示，假设查询`"Strong pain in the side of the head"`（译为“头部侧面非常痛”）时，如果我们在文档库（Document Archive）中搜索到一个文档，这个文档中有句子可以准确地回答它，比如`"Sharp temple headache"`（译为“太阳穴剧烈疼痛”）。但由于这个答案使用了不同的关键词，关键词检索就会无法检索到这个文档。

而语言模型（Laguange Model）可以解决这个问题，因为语言模型不仅能够关注关键词，还可以考虑到文档中句子的含义，能够为 query 检索到这样的文档。

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/2-3.png)

<br/>color2
Language Model 可以改进搜索的两个阶段（Retrieval 和 Reranking），语言模型通过 Embedding（嵌入）来改进。Embedding 将是下一节。看一下 Reranking 是如何进行的，以及 Laguange Model 如何改进它。

<br/>

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/2-4.png)

# 其他
```python

def keyword_search(query, 
                   client,
                   results_lang='en', 
                   properties = ["text", "title", "url", "views", "lang", "_additional {distance}"],
                   num_results=3):

    where_filter = {
    "path": ["lang"],
    "operator": "Equal",
    "valueString": results_lang
    }

    response = (
        client.query.get("Articles", properties)
        .with_bm25(
          query=query
        )
        .with_where(where_filter)
        .with_limit(num_results)
        .do()
        )
    result = response['data']['Get']['Articles']
    return result


def dense_retrieval(query, 
                    client,
                    results_lang='en', 
                    properties = ["text", "title", "url", "views", "lang", "_additional {distance}"],
                    num_results=5):

    nearText = {"concepts": [query]}
    
    # To filter by language
    where_filter = {
    "path": ["lang"],
    "operator": "Equal",
    "valueString": results_lang
    }
    response = (
        client.query
        .get("Articles", properties)
        .with_near_text(nearText)
        .with_where(where_filter)
        .with_limit(num_results)
        .do()
    )

    result = response['data']['Get']['Articles']

    return result


def print_result(result):
    """ Print results with colorful formatting """
    for i,item in enumerate(result):
        print(f'item {i}')
        for key in item.keys():
            print(f"{key}:{item.get(key)}")
            print()
        print()
```

