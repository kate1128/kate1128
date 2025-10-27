> 学习 ReRank 算法，以及语义检索的评价指标。
>
> Rerank 让大型语言模型按照与**查询相关性对搜索结果从高到低排序**。这种方法可以利用语义信息来更准确地评估文档和查询之间的相关性，从而提高搜索结果的质量。
>

[Weaviate关键词检索](https://www.yuque.com/qiaokate/su87gb/xtiz6hz8i6kz7hby)和[Weaviate稠密检索](https://www.yuque.com/qiaokate/su87gb/hfyifau7o3t44ge1)的主要任务是寻找相关的结果，它们返回的分数只是根据某种度量找到的相关结果，并不能完全反映结果和查询之间的真实相关性。

重排（Rerank）是一种优化关键词检索和稠密检索结果的方法。它是语义检索中除了稠密检索外的重要组成部分。Rerank 让大型语言模型按照与查询相关性对搜索结果从高到低排序。这种方法可以利用语义信息来更准确地评估文档和查询之间的相关性，从而提高搜索结果的质量。

##  配置环境
安装包：

```python
!pip install cohere 
!pip install weaviate-client
```

```python
import os
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # 读取本地 .env 文件
```

```python
import cohere
co = cohere.Client(os.environ['COHERE_API_KEY'])
```

创建连接存储所有维基百科条目数据库的客户端。

```python
import weaviate

# 连接到包含 10M 维基百科的用于网络演示的向量数据库
# 使用一个公共的拥有只读权限的API键
auth_config = weaviate.auth.AuthApiKey(
    api_key=os.environ['WEAVIATE_API_KEY']) # "76320a90-53d8-42bc-b41d-678647c6672e"
```

```python
client = weaviate.Client(
    url=os.environ['WEAVIATE_API_URL'],
    auth_client_secret=auth_config,
    additional_headers={
        "X-Cohere-Api-Key": os.environ['COHERE_API_KEY'],
    }
)
```

##  稠密检索
### 稠密检索的不足
首先我们调用上节课的 `dense_retrieval` 函数，查看稠密检索的结果

```python
from utils import dense_retrieval
```

```python
from utils import print_result
```

```python
query_1 = "What is the capital of Canada?"
```

```python
dense_retrieval_results = dense_retrieval(query_1, client)
```

```python
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>item 0
_additional:{'distance': -150.8031}

lang:en

text:The governor general of the province had designated Kingston as the capital in 1841. However, the major population centres of Toronto and Montreal, as well as the former capital of Lower Canada, Quebec City, all had legislators dissatisfied with Kingston. Anglophone merchants in Quebec were the main group supportive of the Kingston arrangement. In 1842, a vote rejected Kingston as the capital, and study of potential candidates included the then-named Bytown, but that option proved less popular than Toronto or Montreal. In 1843, a report of the Executive Council recommended Montreal as the capital as a more fortifiable location and commercial centre, however, the Governor General refused to execute a move without a parliamentary vote. In 1844, the Queen's acceptance of a parliamentary vote moved the capital to Montreal.

title:Ottawa

url:https://en.wikipedia.org/wiki?curid=22219

views:2000


item 1
_additional:{'distance': -150.28354}

lang:en

text:For brief periods, Toronto was twice the capital of the united Province of Canada: first from 1849 to 1852, following unrest in Montreal, and later 1856–1858. After this date, Quebec was designated as the capital until 1866 (one year before Canadian Confederation). Since then, the capital of Canada has remained Ottawa, Ontario.

title:Toronto

url:https://en.wikipedia.org/wiki?curid=64646

views:3000


item 2
_additional:{'distance': -150.02524}

lang:en

text:Selection of Ottawa as the capital of Canada predates the Confederation of Canada. The selection was contentious and not straightforward, with the parliament of the United Province of Canada holding more than 200 votes over several decades to attempt to settle on a legislative solution to the location of the capital.

    title:Ottawa

url:https://en.wikipedia.org/wiki?curid=22219

views:2000


item 3
_additional:{'distance': -149.92365}

lang:en

text:Until the late 18th century Québec was the most populous city in present-day Canada. As of the census of 1790, Montreal surpassed it with 18,000 inhabitants, but Quebec (pop. 14,000) remained the administrative capital of New France. It was then made the capital of Lower Canada by the Constitutional Act of 1791. From 1841 to 1867, the capital of the Province of Canada rotated between Kingston, Montreal, Toronto, Ottawa and Quebec City (from 1852 to 1856 and from 1859 to 1866).

title:Quebec City

url:https://en.wikipedia.org/wiki?curid=100727

views:2000


item 4
_additional:{'distance': -149.71033}

lang:en

text:The Quebec Conference on Canadian Confederation was held in the city in 1864. In 1867, Queen Victoria chose Ottawa as the definite capital of the Dominion of Canada, while Quebec City was confirmed as the capital of the newly created province of Quebec.

title:Quebec City

url:https://en.wikipedia.org/wiki?curid=100727

views:2000</code></pre></details>
注：经过测试，发现当前数据库中文预料可能较少，对中文检索比较简单，所以对查询（query）进行了简化。(例如只保留关键词，类似主语)

```python
query_1 = "加拿大首都"
```

```python
dense_retrieval_results = dense_retrieval(query_1, client, 'zh')
```

```python
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="u768989d2"><span class="ne-text">output：</span></summary><pre data-language="json" id="z8wRn" class="ne-codeblock language-json"><code>item 0
_additional:{'distance': -152.25616}

lang:zh

text:18世纪晚期之前，魁北克城一直是加拿大人口最多的城市。在1790年的普查期间，蒙特利尔以18,000居民超过了魁北克，但魁北克（14,000人口）依然保住了新法兰西行政首府的地位。在1791年宪法中，魁北克城成为下加拿大的首府。从1841年到1867年，加拿大省的首府在几个城市之间轮替，包括金士顿，蒙特利尔，多伦多，渥太华和魁北克（1852年到1856年，1859年到1866年）。

title:魁北克市

url:https://zh.wikipedia.org/wiki?curid=117192

views:600


item 1
_additional:{'distance': -150.94444}

lang:zh

text:渥太華（）是加拿大的聯邦首都，全國第四大城市，市區人口是934,243人，首都圈地區是1,323,783人（根據2016年人口普查），面積2,779平方公里，位於安大略省東南部，渥太華河南岸，多倫多以東400公里，蒙特利爾以西190公里。與美國、澳大利亞等國不同，渥太華不是聯邦直轄的行政區，但是渥太華土地管理和城市規劃是由國家首都委員會（National Capital Commission）負責。

title:渥太華

url:https://zh.wikipedia.org/wiki?curid=70236

views:800


item 2
_additional:{'distance': -150.90271}

lang:zh

text:1857年12月31日，维多利亚女王选择渥太华为加拿大省的首都（包括现在的安大略和魁北克）。虽然现代的渥太华是加拿大第四大城市，但在当年，她仅仅是一个木材贸易通道中的内陆小镇，并且距离殖民地的几个主要城市（东部的蒙特利尔和魁北克城；西部的多伦多和京士頓）路途遥远。女王的顾问们建议渥太华成为首都之选有两大重要理由：首先，渥太华是唯一具有一定规模、并且位于加拿大省东西部边界地（现安大略与魁北克边界）的城市，定都于此是平衡两个殖民地及其英裔、法裔居民的聪明妥协之举；其次，1812年战争表明，其他主要城市容易受到美国人的攻击，因为过于靠近美加边界。渥太华位于腹地，易于防守，渥太华河及丽都运河使之与加拿大东西部之间交通极为便利。另外两个方面的考虑是：渥太华正好介于多伦多和魁北克城之间（距离这两个城市都是500公里），并且城市规模较小，因而不容易受到大规模的暴徒袭击，因为政治动机，以往的首都城市都受到过这种攻击。

title:渥太華

url:https://zh.wikipedia.org/wiki?curid=70236

views:800


item 3
_additional:{'distance': -150.68478}

lang:zh

text:多伦多（，），是北美洲国家加拿大安大略省首府，加拿大的最大城市。多伦多坐落在安大略湖西北岸的南安大略地区。根据2021年的加拿大人口普查，多伦多市人口达2,794,356人，为加拿大最大城市。多伦多市是大多伦多地区的心脏地区，也是安大略省南部人口稠密区（称作“金馬蹄地區”）的一部分。都會区有6,202,225名居民，而覆蓋範圍較廣的大多倫多地區則有9,765,188名居民。作為加拿大的经济中心，多伦多是一個世界级城市，也是世界上最大的金融中心之一。多伦多在经济上的领先地位在于金融、商业服务、电信、航太、交通运输、媒体、艺术、电影、电视製作、出版、软件、医药研究、教育、旅游、体育等产业。多伦多证券交易所是世界第七大交易所，总部设于市内，有多数加拿大公司在这里上市。

title:多伦多

url:https://zh.wikipedia.org/wiki?curid=3132

views:1000


item 4
_additional:{'distance': -150.47894}

lang:zh

text:蒙特婁曾经是加拿大经济首都，拥有最多的人口及最发达的经济，但是在1976年蒙特婁奧運會后被安大略省的多伦多超过。今天蒙特利尔仍然是加拿大最重要的经济中心之一，人工智慧、航空工业、金融、设计、电影工业等行业发达。蒙特婁被认为是世界最佳宜居城市，并被联合国教育、科学及文化组织认定为设计之城。1999年第35屆國際技能競賽在這裡舉行。

title:蒙特利尔

url:https://zh.wikipedia.org/wiki?curid=43791

views:1000</code></pre></details>
查看检索结果，结果中第二个是正确的，是渥太华。有一些不再是正确答案的结果。多伦多不是加拿大的首都。然后，我们还有魁北克市，这是错误的答案。为什么会发生这种情况呢？通过一个小例子来帮助理解这个概念。虽然和当前的搜索结果有点不同，但有助于我们理解这个情况。

假设查询的问题是“加拿大的首都是什么？”，可能的回答有以下五个：

+ 加拿大的首都是渥太华：这是正确的。
+ 多伦多位于加拿大：这也是正确的，但与问题无关。
+ 法国的首都是巴黎：这也是正确的，但不是问题的答案。
+ 加拿大的首都是悉尼：这是不正确的。
+ 安大略的省会是多伦多：这是正确的，但同样未能回答问题。

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/5-1.png)

进行稠密检索时会发生什么呢？

假设五个句子在 embedding 空间的分布如图所示。稠密检索的原理是将查询 生成 embedding，然后返回与之最接近的内容，即“安大略的首都是多伦多”。稠密检索看重**语义相似性**，因此它返回与问题最相似的内容。但这可能不是正确的答案，甚至可能不是真实的陈述，它只是一个在语义上与问题接近的句子。因此，稠密检索有可能返回的并非答案。我们如何修复这个问题呢？这就是重排起作用的地方。

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/5-2.png)

假设查询是“加拿大的首都是什么”，此时有10个可能的答案，其中一些与问题相关，而另一些则不相关。因此，当我们使用稠密检索时，它会给我们与查询最相似的五个答案，也就是与查询最相似的五个内容。假设返回内容就是绿色的这些句子。现在我们有五个与查询非常接近的句子，但我们不知道哪一个才是正确答案。这就是 Rerank 发挥作用的地方。

重排模型为每个查询结果对打一个相关得分，告诉您答案相对于查询的相关程度。这 5 个句子最高相关性为 0.9，对应于“加拿大的首都是渥太华”，这就是正确的答案。这就是重排的作用。

### 重排模型的训练方式
![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/5-3.png)

重排模型的训练需要大量的高质量样本，这些样本包括与查询高度相关的响应或文档。训练的目标是使模型能够给出高相关性的得分。同时，我们也需要提供一些错误的查询响应作为样本，这些响应可能与查询不完全匹配，可能是接近但不符合的情况，或者是一个可能与查询不匹配的文档。通过训练模型对优质的查询响应给出高分，对不理想的查询响应给出低分，以此获得一个能够分配相关性的重排模型。当查询和响应高度相关时，该模型将给出高得分。

##  使用重排改进关键词检索
将导入之前在第一课中使用的关键词检索函数。再次问它，“加拿大的首都是什么”

```python
from utils import keyword_search
```

```python
query_1 = "What is the capital of Canada?"
results = keyword_search(query_1,
                         client,
                         properties=["text", "title", "url", "views", "lang", "_additional {distance}"],
                         num_results=3
                        )

for i, result in enumerate(results):
    print(f"i:{i}")
    print(result.get('title'))
    print(result.get('text'))
```

```python
i:0
Monarchy of Canada
In his 1990 book, "Continental Divide: the Values and Institutions of the United States and Canada," Seymour Martin Lipset argues that the presence of the monarchy in Canada helps distinguish Canadian identity from American identity. Since at least the 1930s, supporters of the Crown have held the opinion that the Canadian monarch is also one of the rare unified elements of Canadian society, focusing both "the historic consciousness of the nation" and various forms of patriotism and national love "[on] the point around which coheres the nation's sense of a continuing personality". Former Governor General Vincent Massey articulated in 1967 that the monarchy "is part of ourselves. It is linked in a very special way with our national life. It stands for qualities and institutions which mean Canada to every one of us and which for all our differences and all our variety have kept Canada Canadian." But, according to Arthur Bousfield and Gary Toffoli, Canadians were, through the late 1960s to the 2000s, encouraged by the federal government to "neglect, ignore, forget, reject, debase, suppress, even hate, and certainly treat as foreign what their parents and grandparents, whether spiritual or blood, regarded as the basis of Canadian nationhood, autonomy, and history", including the monarchy. Former Governor General Roland Michener said in 1970 that anti-monarchists claimed the Canadian Crown is foreign and incompatible with Canada's multicultural society, which the government promoted as a Canadian identifier, and Lawrence Martin called in 2007 for Canada to become a republic in order to "re-brand the nation". However, Michener also stated, "[the monarchy] is our own by inheritance and choice, and contributes much to our distinctive Canadian identity and our chances of independent survival amongst the republics of North and South America." Journalist Christina Blizzard emphasized in 2009 that the monarchy "made [Canada] a haven of peace and justice for immigrants from around the world", while Michael Valpy contended in 2009 that the Crown's nature permitted non-conformity amongst its subjects, thereby opening the door to multiculturalism and pluralism.
i:1
Early modern period
North America outside the zone of Spanish settlement was a contested area in the 17th century. Spain had founded small settlements in Florida and Georgia but nowhere near the size of those in New Spain or the Caribbean islands. France, The Netherlands, and Great Britain held several colonies in North America and the West Indies from the 17th century, 100 years after the Spanish and Portuguese established permanent colonies. The British colonies in North America were founded between 1607 (Virginia) and 1733 (Georgia). The Dutch explored the east coast of North America and began founding settlements in what they called New Netherland (now New York State.). France colonized what is now Eastern Canada, founding Quebec City in 1608. France's loss in the Seven Years' War resulted in the transfer of New France to Great Britain. The Thirteen Colonies, in lower British North America, rebelled against British rule in 1775, largely due to the taxation that Great Britain was imposing on the colonies. The British colonies in Canada remained loyal to the crown, and a provisional government formed by the Thirteen Colonies proclaimed their independence on July 4, 1776 and subsequently became the original 13 United States of America. With the 1783 Treaty of Paris ending the American Revolutionary War, Britain recognised the former Thirteen Colonies' independence.
i:2
Flag of Canada
By the Second World War, the Red Ensign was viewed as Canada's "de facto" national flag. A joint committee of the Senate and House of Commons was appointed on November 8, 1945, to recommend a national flag to officially adopt. It received 2,409 designs from the public and was addressed by the director of the Historical Section of the Canadian Army, Fortescue Duguid, who pointed out that red and white were Canada's official colours and there was already an emblem representing the country: three joined maple leaves seen on the escutcheon of the Canadian coat of arms. By May 9 the following year, the committee reported back with a recommendation "that the national flag of Canada should be the Canadian red ensign with a maple leaf in autumn golden colours in a bordered background of white". The Legislative Assembly of Quebec had urged the committee to not include any of what it deemed as "foreign symbols", including the Union Flag, and Mackenzie King, then still prime minister, declined to act on the report; fearing it may lead to political instability. As a result, the Union Flag was kept as a national flag, and the order to fly the Canadian Red Ensign at government buildings was maintained.
```



```python
query_2_zh = "加拿大 首都"
results_zh = keyword_search(query_2_zh,
                         client,
                         results_lang='zh',
                         properties=["text", "title", "url", "views", "lang", "_additional {distance}"],
                         num_results=3
                        )

for i, result in enumerate(results_zh):
    print(f"i:{i}")
    print(result.get('title'))
    print(result.get('text'))
```

```python
i:0
首都
每一個國家通常只設立一個首都，因為政府通常會將其重要機關集中在首都地區，以方便政府高層行政和管理，但亦有例外。一些國家有多個首都，一些甚至沒有。有時候，實際的首都和法定的首都由於某些原因並不在同一個城市。譬如，一個稱為「首都」的城市，實際上並非中央政府所在地。反之，所謂的正式「首都」雖然是中央政府的所在地，但可能不是政治決策的地理中心。故此，「行政首都」一般被認定為是該國的「國家首都」。
i:1
幻想戰記
遊戲中存在六個大陸，各個大陸的地圖之間沒有物理連接，地圖與地圖之間的移動方式為點選各個大陸上稱為「戰場」或者「首都」的據點。其中各個「戰場」是可以宣戰的地圖，而「首都」則不能被進攻（也就是說就算任何一個國家的本土被侵佔完畢該國也不會滅國）。
i:2
首都
首都，作為國家政治、經濟、文化的會聚並不是永恆不變的。在古代，國家一般採取中央集權政策，地方勢力有限；首都一旦淪陷，就意味著朝代的覆亡。中國三國時代，蜀漢、吳因失去各自的首都──成都和建業（今南京）而亡國。
```

+ 英文答案输出的前三个答案并不理想。它们涉及加拿大的君主制、早期现代时期和加拿大国旗。
+ 中文答案输出的前三个答案相关性更差。它们只考虑了首都，没有关于加拿大的信息。

为什么它们会这样呢？因为关键词检索仅仅是在查找与查询有许多共同单词的文档，但无法真正判断出是否这些文档确实在回答问题。所有这些文章都包含与查询有很多共同单词，但它们并非答案。

让我们扩大下检索规模，要求它返回 500 个结果。为了便于观测，这里不打印文本，只打印标题。

```python
query_1 = "What is the capital of Canada?"
results = keyword_search(query_1,
                         client,
                         properties=["text", "title", "url", "views", "lang", "_additional {distance}"],
                         num_results=500
                        )

for i, result in enumerate(results[:10]): # 您可以自行调整输出的标题数量
    print(f"i:{i}")
    print(result.get('title'))
    #print(result.get('text'))
```

```python
i:0
Monarchy of Canada
i:1
Early modern period
i:2
Flag of Canada
i:3
Flag of Canada
i:4
Prime Minister of Canada
i:5
Hamilton, Ontario
i:6
Liberal Party of Canada
i:7
Stephen Harper
i:8
Monarchy of Canada
i:9
Flag of Canada
```



```python
query_1_zh = "加拿大 首都"
results_zh = keyword_search(query_1_zh,
                         client,
                         results_lang='zh',
                         properties=["text", "title", "url", "views", "lang", "_additional {distance}"],
                         num_results=500
                        )

for i, result in enumerate(results_zh[:10]): # 您可以自行调整输出的标题数量
    print(f"i:{i}")
    print(result.get('title'))
    #print(result.get('text'))
```

```python
i:0
首都
i:1
幻想戰記
i:2
首都
i:3
首府
i:4
首都 (香港)
i:5
中華民國首都
i:6
西安市
i:7
首都
i:8
首都
i:9
首都
```

这里有打分最高的前 500个结果。我们如何才能确定这些结果中是否包含答案呢？这就是重排的作用所在。下面这个函数对响应进行重排，并输出打分最高的 10 个。

```python
import cohere
def rerank_responses(query, responses, num_responses=10, results_lang='en'):
    """
    根据给定的查询，使用指定的模型对响应列表进行重排序。

    Args:
        query (str): 查询。
        responses (list): 响应的列表。
        num_responses (int, optional): 返回的响应数量，默认为10。
        results_lang (str, optional): 指定的语言模型版本，默认为英文（官方只提供英文和多语言两个版本）。

    Returns:
        list: 重排序后的响应列表。
    """
    
    model_name = 'rerank-english-v2.0' if results_lang=='en' else 'rerank-multilingual-v2.0'
    
    reranked_responses = co.rerank(
        model=model_name,
        query=query,
        documents=responses,
        top_n=num_responses,
    )
    return reranked_responses
```

现在，让我们将答案的文本上进行重排。

```python
texts = [result.get('text') for result in results] # 只提取结果中的文本
reranked_text = rerank_responses(query_1, texts)
```

```python
for i, rerank_result in enumerate(reranked_text):
    print(f"i:{i}")
    print(f"{rerank_result}")
    print()
```

```python
i:0
RerankResult<document['text']: Selection of Ottawa as the capital of Canada predates the Confederation of Canada. The selection was contentious and not straightforward, with the parliament of the United Province of Canada holding more than 200 votes over several decades to attempt to settle on a legislative solution to the location of the capital., index: 407, relevance_score: 0.9875684>

i:1
RerankResult<document['text']: Montreal was the capital of the Province of Canada from 1844 to 1849, but lost its status when a Tory mob burnt down the Parliament building to protest the passage of the Rebellion Losses Bill. Thereafter, the capital rotated between Quebec City and Toronto until in 1857, Queen Victoria herself established Ottawa as the capital due to strategic reasons. The reasons were twofold. First, because it was located more in the interior of the Province of Canada, it was less susceptible to attack from the United States. Second, and perhaps more importantly, because it lay on the border between French and English Canada, Ottawa was seen as a compromise between Montreal, Toronto, Kingston and Quebec City, which were all vying to become the young nation's official capital. Ottawa retained the status as capital of Canada when the Province of Canada joined with Nova Scotia and New Brunswick to form the Dominion of Canada in 1867., index: 100, relevance_score: 0.9795897>

i:2
RerankResult<document['text']: Ottawa is the political centre of Canada and headquarters to the federal government. The city houses numerous foreign embassies, key buildings, organizations, and institutions of Canada's government, including the Parliament of Canada, the Supreme Court, the residence of Canada's viceroy, and Office of the Prime Minister., index: 202, relevance_score: 0.9753901>

i:3
RerankResult<document['text']: Until the late 18th century Québec was the most populous city in present-day Canada. As of the census of 1790, Montreal surpassed it with 18,000 inhabitants, but Quebec (pop. 14,000) remained the administrative capital of New France. It was then made the capital of Lower Canada by the Constitutional Act of 1791. From 1841 to 1867, the capital of the Province of Canada rotated between Kingston, Montreal, Toronto, Ottawa and Quebec City (from 1852 to 1856 and from 1859 to 1866)., index: 496, relevance_score: 0.9711838>

i:4
RerankResult<document['text']: Ottawa was chosen as the capital for two primary reasons. First, Ottawa's isolated location, surrounded by dense forest far from the Canada–US border and situated on a cliff face, would make it more defensible from attack. Second, Ottawa was approximately midway between Toronto and Kingston (in Canada West) and Montreal and Quebec City (in Canada East) making the selection an important political compromise., index: 479, relevance_score: 0.96653706>

i:5
RerankResult<document['text']: Canada is a country in North America. Its ten provinces and three territories extend from the Atlantic Ocean to the Pacific Ocean and northward into the Arctic Ocean, covering over , making it the world's second-largest country by total area. Its southern and western border with the United States, stretching , is the world's longest binational land border. Canada's capital is Ottawa, and its three largest metropolitan areas are Toronto, Montreal, and Vancouver., index: 481, relevance_score: 0.9421884>

i:6
RerankResult<document['text']: Although both rebellions were put down in short order, the British government sent Lord Durham to investigate the causes. He recommended self-government be granted and Lower and Upper Canada be re-joined in an attempt to assimilate the French Canadians. Accordingly, the two colonies were merged into the Province of Canada by the "Act of Union 1840", with the capital at Kingston, and Upper Canada becoming known as Canada West. Parliamentary self-government was granted in 1848. There were heavy waves of immigration in the 1840s, and the population of Canada West more than doubled by 1851 over the previous decade. As a result, for the first time, the English-speaking population of Canada West surpassed the French-speaking population of Canada East, tilting the representative balance of power., index: 68, relevance_score: 0.86567897>

i:7
RerankResult<document['text']: Ottawa is headquarters to numerous major medical organizations and institutions such as Canadian Red Cross, Canadian Blood Services, Health Canada, Canadian Medical Association, Royal College of Physicians and Surgeons of Canada, Canadian Nurses Association, and the Medical Council of Canada., index: 394, relevance_score: 0.86153823>

i:8
RerankResult<document['text']: Ontario ( ; ) is one of the thirteen provinces and territories of Canada. Located in Central Canada, it is Canada's most populous province, with 38.3 percent of the country's population, and is the second-largest province by total area (after Quebec). Ontario is Canada's fourth-largest jurisdiction in total area when the territories of the Northwest Territories and Nunavut are included. It is home to the nation's capital city, Ottawa, and the nation's most populous city, Toronto, which is Ontario's provincial capital., index: 228, relevance_score: 0.4989891>

i:9
RerankResult<document['text']: With sixty percent of Canada's steel produced in Hamilton by Stelco and Dofasco, the city has become known as the Steel Capital of Canada. After nearly declaring bankruptcy, Stelco returned to profitability in 2004. On August 26, 2007 United States Steel Corporation acquired Stelco for C$38.50 in cash per share, owning more than 76 percent of Stelco's outstanding shares. On September 17, 2014, US Steel Canada announced it was applying for bankruptcy protection and it would close its Hamilton operations., index: 5, relevance_score: 0.49455282>
```

```python
texts_zh = [result.get('text') for result in results_zh]
reranked_text_zh = rerank_responses_zh(query_1_zh, texts_zh)
```

```python
for i, rerank_result in enumerate(reranked_text_zh):
    print(f"i:{i}")
    print(f"{rerank_result}")
    print()
```

```python
i:0
RerankResult<document['text']: 國會山莊工程完結之前，英屬北美當中三個殖民地－加拿大省（包括現安大略和魁北克）、新伯倫瑞克和新斯科舍－於1867年7月1日締結成加拿大聯邦，而渥太華亦成為加拿大聯邦的首都。往後四年内曼尼托巴、卑詩、愛德華王子島和西北地區（包括現艾伯塔、萨斯喀彻温、育空地區、西北地區和努纳武特）相繼加入加拿大聯邦，令聯邦政府公務員數目遞增，議會亦需容納來自曼尼托巴、卑詩和愛德華王子島的新議員，國會山莊建築群的辦公空間因此已不敷應用。, index: 195, relevance_score: 0.99914074>

i:1
RerankResult<document['text']: 加拿大（英语、法语：Canada，IPA读音：（英）（法））是一个位于北美洲最北部的国家，属于西半球及北半球，其领土西临太平洋，东濒大西洋，北接北冰洋，有部分领土位于北极圈内。加拿大东北方与丹麦领地格陵兰相望，並以漢斯島為界接壤，南方及西北方与美国本土及阿拉斯加州接壤，法国属地圣皮埃尔和密克隆位于其东部的岛屿之中。加拿大的领土面积达998万4670平方公里，为全球面积第二大国家，亦是发达国家之中的领土面积最大者。该国首都为渥太华，以2016年都会区人口排序，前五大城市为多伦多、蒙特利尔、温哥华、卡尔加里、埃德蒙顿。十个省和三个地区组成加拿大全國。加拿大经济发达，环境优美，被《福布斯》列於2020年退休宜居國的名單中。, index: 87, relevance_score: 0.9919067>

i:2
RerankResult<document['text']: 1793年，上加拿大副總督閃高（）選擇此處為該殖民地的首府，並打算將此處命名為「佐治拿」（）以紀念國王佐治三世，但建議被總督多徹斯特否決。此處的聚落待到1826年才設立村級行政區劃，並以英國首都之名取名為「倫敦」。倫敦沒有像閃高預期般成為上加拿大的首府，卻成為第一大城約克（即現多倫多）以西地域的行政中心。湯瑪士·塔爾博特上校（）獲任為該地區的首席殖民官，主管了該帶的土地勘探並籌建了西安大略半島地區的政府辦公建築群，倫敦村和西南安大略的其他聚落因此得益。, index: 211, relevance_score: 0.9716179>

i:3
RerankResult<document['text']: 伦敦（）是位于加拿大安大略省西南部的一座城市，座落魁北克市-温莎走廊之上，大概位於多伦多與温莎之間的半途位置。倫敦市在行政事務上是獨立於其周邊的米德爾塞克斯縣，但該縣的行政中心仍設於倫敦市內。根據加拿大2011年人口普查統計，倫敦市共有人口36萬6151，在全國排名第15位；都會區人口則達474,786人，在全國排名第11位。, index: 190, relevance_score: 0.8489722>

i:4
RerankResult<document['text']: 1858年，維多利亞女王將加拿大省的首府定於拜敦；由於軍營山地理上淩駕於拜敦鎮址和渥太華河之上，加上該片土地的業權已歸政府所有，當局因此決定將加拿大省的議會建築設置於此。加拿大省工務局於同年5月7日對外徵求議會建築設計，總共收取298份方案。得選方案於1859年8月29日公佈，議會中央大樓（）、東西兩側的辦公大樓和總督官邸各由不同建築師設計。中央大樓預算造價為30萬元，而東西兩側辦公大樓的預算造價則各為12萬元。, index: 177, relevance_score: 0.799129>

i:5
RerankResult<document['text']: 西北地區（，缩写为NWT；，因紐特語：ᓄᓇᑦᓯᐊᖅ）是加拿大一級行政區裡面的三個「-{zh-hans:地区;zh-hk:地區;zh-tw:地方;}-/领地」（Territory）之一，面積1,171,918平方公里，但在1999年努那福特自西北地方分離而出之前，面積曾高達3,439,296平方公里，佔了加拿大領土的三分之一。2011年普查時人口為41,462人。首府為耶洛奈夫。, index: 192, relevance_score: 0.3897207>

i:6
RerankResult<document['text']: 1837年上加拿大起義期間，倫敦是托里派（，即支持政府的一方）的主要陣營之一。但在查理斯·登科姆（）率領的短暫騷亂過後，英國政府於1838年決定在此駐兵。隨著軍人及其家眷遷入，他們對該帶服務業的需求亦隨之上升，並吸引大批平民遷居此處。隨著人口上升，倫敦亦於1840年改設鎮。, index: 218, relevance_score: 0.270095>

i:7
RerankResult<document['text']: 在鐵路交通方面，倫敦分別座落加拿大國家鐵路介乎多倫多與芝加哥的主綫（並設一條通往温莎的支綫）和加拿大太平洋鐵路介乎多倫多與底特律的主綫。城際客運鐵路服務則由維亞鐵路提供；維亞的倫敦火車站是加拿大第四繁忙的客運鐵路站。安大略高速鐵路将于十年后将伦敦、多伦多的车程控制在一个半小时内。, index: 173, relevance_score: 0.18892181>

i:8
RerankResult<document['text']: 國會山莊所在的山丘由石灰石構成，表面的原始森林則充斥著櫸木和鐵杉，為渥太華河上一座主要路標，數百年來為原住民、歐裔皮毛交易員和探險家指點了通往北美洲内陸之路。現渥太華的前身拜敦聚落（）形成後，麗都運河開通至此，而這座山丘則成為一個軍事基地所在，並取名為軍營山（）。當局原計劃在此設立一座要塞，但最終沒有成事。, index: 170, relevance_score: 0.16613355>

i:9
RerankResult<document['text']: 首都由5 幢樓高52至56 層的樓宇（電梯層數至70 樓，不設4 樓、13 樓等）物業組成，設於樓高5 層的大型平台上，提供2,096 個住宅單位。一半單位為3房設計，三成單位為4房設計，兩者面積均為900平方呎以上。兩房設計佔一成多，面積由683餘至700餘平方呎。, index: 22, relevance_score: 0.15974103>
```

在输入查询和结果之后，让我们打印出重排的前 10 个结果。

请注意，其中获得了正确答案。它确定渥太华作为加拿大的首都，并且相关分数非常高，接近 1，达到 0.98。值得注意的是，排名第二的文章也相当不错，它涉及加拿大历史上不同的首都，其相关分数为 0.97。第三个也很出色。重排从关键词检索出的 10 个答案中挑选出相关性最高的 10。

##  使用重排改进稠密检索
### 进一步理解重排
我将再次使用稠密检索函数，尝试解决一个稍微困难的问题。我们询问："谁是历史上最高的人？" 对于关键词检索来说，这将是一个有挑战性的问题，因为它更关注包含“历史”或“人物”的文章，并不能捕捉到问题的真实意义。我们希望稠密检索可以做得更好。因此，我们将调用稠密检索函数来获取更准确的结果。

```python
from utils import dense_retrieval
```

```python
query_2 = "Who is the tallest person in history?"
```

```python
results = dense_retrieval(query_2, client)
```

```python
for i, result in enumerate(results):
    print(f"i:{i}")
    print(result.get('title'))
    print(result.get('text'))
    print()
```

```python
i:0
Robert Wadlow
Robert Pershing Wadlow (February 22, 1918 July 15, 1940), also known as the Alton Giant and the Giant of Illinois, was a man who was the tallest person in recorded history for whom there is irrefutable evidence. He was born and raised in Alton, Illinois, a small city near St. Louis, Missouri.

i:1
Manute Bol
Bol came from a family of extraordinarily tall men and women. He said: "My mother was , my father , and my sister is . And my great-grandfather was even taller—." His ethnic group, the Dinka, and the Nilotic people of which they are a part, are among the tallest populations in the world. Bol's hometown, Turalei, is the origin of other exceptionally tall people, including basketball player Ring Ayuel. "I was born in a village, where you cannot measure yourself," Bol reflected. "I learned I was 7 foot 7 in 1979, when I was grown. I was about 18 or 19."

i:2
Sultan Kösen
Sultan Kösen (born 10 December 1982) is a Turkish farmer who holds the Guinness World Record for tallest living male at . Of Kurdish ethnicity, he is the seventh tallest man in history.

i:3
Sultan Kösen
Kösen turned 40 years old on 10 December 2022. He celebrated his birthday a few days early by visiting the Ripley's Believe It or Not! museum in Orlando, Florida, USA and posing next to a life-sized statue of Robert Wadlow, the tallest man ever at 272 cm (8 ft 11.1 in).

i:4
Netherlands
The Dutch are the tallest people in the world, by nationality, with an average height of for adult males and for adult females in 2009. The average height of young males in the Netherlands increased from 5 feet, 4 inches to approximately 6 feet between the 1850s until the early 2000s. People in the south are on average about shorter than those in the north.
```

```python
query_2_zh = "历史上最高的人?"
```

```python
for i, result in enumerate(results_zh):
    print(f"i:{i}")
    print(result.get('title'))
    print(result.get('text'))
    print()
```

```python
i:0
鮑喜順
鲍喜顺在2005年、2008年两度獲得吉尼斯世界纪录認證被承認是地球上因自然原因而長得最高的活人（其他擁有世界之最頭銜的長人不是已逝，如罗伯特·瓦德罗，就是因特殊病變而过度发育稱為巨人症，如列昂尼德·斯塔德尼克）。实际上，这一纪录在2007年即被乌克兰人列昂尼德·斯塔德尼克（Leonid Stadnyk）取代，但因其后来拒绝接受吉尼斯世界纪录组织测量身高而并未被该组织继续认可，故鲍喜顺重新成为世界最高人，直至2009年该纪录持有者被土耳其人蘇丹·科塞（Sultan Kösen）取代。

i:1
蘇丹·科塞
蘇丹·科塞（；），出生於土耳其马尔丁，是自2009年起被確認為全世界最高的人，被列入金氏世界紀錄大全，其雙手和腳掌亦打破金氏世界紀錄大全，腳掌長達40公分。2009年時，科塞高247公分，到了2012年，他高了4公分，達到251公分。

i:2
瑪麗蓮·沃斯·莎凡特
瑪麗蓮·沃斯·莎凡特（Marilyn vos Savant，）曾經被記載為吉尼斯世界記錄所認定擁有最高智商的人類及女性 (1984 to 1989)。她於1946年出生於美國密苏里州的圣路易斯，瑪麗蓮在剛滿10歲的1956年9月時初次接受史丹福-比奈智力測驗 （心智年齡比例智商），測得智商高達228，並登上世界紀錄。然而，智商的判定與比較方式後來遭到爭議， 隨後吉尼斯世界記錄在1990年移除了“智商最高的人”這個項目。

i:3
艾德蒙·希拉里
艾德蒙·珀西瓦尔·希拉里爵士，KG，ONZ，KBE（Edmund Percival Hillary，），是紐西蘭登山家和探險家，在和雪巴人嚮導丹增·诺盖的合作之下，他和丹增·诺盖成了可證明的記錄中最早成功攀登珠穆朗瑪峰峰頂的人。

i:4
艾瑪·莫拉諾
艾瑪·馬丁娜·露易吉亞·莫拉諾（，），生於意大利奇維亞斯科，超級人瑞，曾是世界最年長者（世界紀錄第5名）和1890年代最後1位去世的人。除此之外，她也是歐洲的第3年長者（享嵩壽117歲又137天），僅次於雅娜·卡爾曼特（享嵩壽122歲164天）和露西爾·朗東（生於1904年2月11日）。
```

我们发现这里已经获到了正确的答案：历史上最高的人是罗伯特·沃德洛。而且还查询到了其他文件。

不过，我们仍然可以使用重排来帮助我们。

当我们对这些结果应用重排时会发生什么呢？让我们再次调用重新排名函数，它将会给出查询文本的相关性并对结果进行重新排序。

```python
texts = [result.get('text') for result in results]
reranked_text = rerank_responses(query_2, texts)
```

```python
for i, rerank_result in enumerate(reranked_text):
    print(f"i:{i}")
    print(f"{rerank_result}")
    print()
```

```python
i:0
RerankResult<document['text']: Robert Pershing Wadlow (February 22, 1918 July 15, 1940), also known as the Alton Giant and the Giant of Illinois, was a man who was the tallest person in recorded history for whom there is irrefutable evidence. He was born and raised in Alton, Illinois, a small city near St. Louis, Missouri., index: 0, relevance_score: 0.9734939>

i:1
RerankResult<document['text']: Sultan Kösen (born 10 December 1982) is a Turkish farmer who holds the Guinness World Record for tallest living male at . Of Kurdish ethnicity, he is the seventh tallest man in history., index: 2, relevance_score: 0.8664718>

i:2
RerankResult<document['text']: The Dutch are the tallest people in the world, by nationality, with an average height of for adult males and for adult females in 2009. The average height of young males in the Netherlands increased from 5 feet, 4 inches to approximately 6 feet between the 1850s until the early 2000s. People in the south are on average about shorter than those in the north., index: 4, relevance_score: 0.80162543>

i:3
RerankResult<document['text']: Kösen turned 40 years old on 10 December 2022. He celebrated his birthday a few days early by visiting the Ripley's Believe It or Not! museum in Orlando, Florida, USA and posing next to a life-sized statue of Robert Wadlow, the tallest man ever at 272 cm (8 ft 11.1 in)., index: 3, relevance_score: 0.6874202>

i:4
RerankResult<document['text']: Bol came from a family of extraordinarily tall men and women. He said: "My mother was , my father , and my sister is . And my great-grandfather was even taller—." His ethnic group, the Dinka, and the Nilotic people of which they are a part, are among the tallest populations in the world. Bol's hometown, Turalei, is the origin of other exceptionally tall people, including basketball player Ring Ayuel. "I was born in a village, where you cannot measure yourself," Bol reflected. "I learned I was 7 foot 7 in 1979, when I was grown. I was about 18 or 19.", index: 1, relevance_score: 0.6396235>
```

```python
results_zh = dense_retrieval(query_2_zh, client, results_lang="zh")
```

```python
texts_zh = [result.get('text') for result in results_zh]
reranked_text_zh = rerank_responses_zh(query_2_zh, texts_zh)
```

```python
for i, rerank_result in enumerate(reranked_text_zh):
    print(f"i:{i}")
    print(f"{rerank_result}")
    print()
```

```python
i:0
RerankResult<document['text']: 蘇丹·科塞（；），出生於土耳其马尔丁，是自2009年起被確認為全世界最高的人，被列入金氏世界紀錄大全，其雙手和腳掌亦打破金氏世界紀錄大全，腳掌長達40公分。2009年時，科塞高247公分，到了2012年，他高了4公分，達到251公分。, index: 1, relevance_score: 0.9980808>

i:1
RerankResult<document['text']: 鲍喜顺在2005年、2008年两度獲得吉尼斯世界纪录認證被承認是地球上因自然原因而長得最高的活人（其他擁有世界之最頭銜的長人不是已逝，如罗伯特·瓦德罗，就是因特殊病變而过度发育稱為巨人症，如列昂尼德·斯塔德尼克）。实际上，这一纪录在2007年即被乌克兰人列昂尼德·斯塔德尼克（Leonid Stadnyk）取代，但因其后来拒绝接受吉尼斯世界纪录组织测量身高而并未被该组织继续认可，故鲍喜顺重新成为世界最高人，直至2009年该纪录持有者被土耳其人蘇丹·科塞（Sultan Kösen）取代。, index: 0, relevance_score: 0.97534317>

i:2
RerankResult<document['text']: 瑪麗蓮·沃斯·莎凡特（Marilyn vos Savant，）曾經被記載為吉尼斯世界記錄所認定擁有最高智商的人類及女性 (1984 to 1989)。她於1946年出生於美國密苏里州的圣路易斯，瑪麗蓮在剛滿10歲的1956年9月時初次接受史丹福-比奈智力測驗 （心智年齡比例智商），測得智商高達228，並登上世界紀錄。然而，智商的判定與比較方式後來遭到爭議， 隨後吉尼斯世界記錄在1990年移除了“智商最高的人”這個項目。, index: 2, relevance_score: 0.81032884>

i:3
RerankResult<document['text']: 艾瑪·馬丁娜·露易吉亞·莫拉諾（，），生於意大利奇維亞斯科，超級人瑞，曾是世界最年長者（世界紀錄第5名）和1890年代最後1位去世的人。除此之外，她也是歐洲的第3年長者（享嵩壽117歲又137天），僅次於雅娜·卡爾曼特（享嵩壽122歲164天）和露西爾·朗東（生於1904年2月11日）。, index: 4, relevance_score: 0.78858316>

i:4
RerankResult<document['text']: 艾德蒙·珀西瓦尔·希拉里爵士，KG，ONZ，KBE（Edmund Percival Hillary，），是紐西蘭登山家和探險家，在和雪巴人嚮導丹增·诺盖的合作之下，他和丹增·诺盖成了可證明的記錄中最早成功攀登珠穆朗瑪峰峰頂的人。, index: 3, relevance_score: 0.6488002>
```

我们发现重排得到的结果中确实是与罗伯特·沃德洛是相关性最高的那个，为 0.97。对于其他文章，它给出的相关性并不高。

+ PS: 中文支持确实不太行

重新排名帮助我们确定在稠密检索出现的答案中哪一个才是正确的答案。现在，我们鼓励您在这里暂停，尝试自己的例子。用自己的查询，找到搜索结果，然后使用重排找到正确的答案。

### 搜索系统的评估
![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/5-4.png)

既然我们有了所有这些搜索系统，您可能想知道如何评估它们。有多种[评估方法](https://zhuanlan.zhihu.com/p/351986117)，如:

+ 平均精度（MAP）
+ 平均倒数排名（MRR）
+ 归一化折减累积增益（NDCG）

那么，如何创建一个测试集来评估这些模型呢？一个优质的测试集应该包含查询和正确的响应。然后，您可以将这些正确的响应与模型给出的响应进行比较，就像评估分类模型的准确性、精确度或召回率一样。如果您想了解更多关于评估搜索系统的信息，我们将会在资源中提供一些文章链接供您参考。

现在，您已经学会使用搜索和重排来检索包含特定问题答案的文档。下节将学习如何结合搜索系统和生成模型，以便以人类的方式输出查询的答案。

