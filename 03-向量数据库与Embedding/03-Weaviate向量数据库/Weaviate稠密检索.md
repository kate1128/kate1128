> 稠密检索是干啥的？
>
> 1. 先把 query 映射到 Embeddings 空间
> 2. 找到映射空间内与 query_embed 最邻近的 text
> 3. 生成结果
>
> 其次，还用 Annoy 库构建了一个向量索引
>

##  环境配置
安装必须的 Python 库

+ 安装 `Cohere` 获取 `Embedding`
+ `Annoy` ：进行近似最近邻搜索
+ `dotenv` 库：用来检查环境变量的
+ `COHERE_API_KEY`[获取COHERE_API_KEY](https://www.yuque.com/qiaokate/su87gb/rbmqi8vkx8vu3pgg)
+ `WEAVIATE_API_URL`

```python
# !pip install cohere
# !pip install weaviate-client
# !pip install python-dotenv
# !pip install Annoy
```

```python
import os
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # 读取本地的 .env 文件
```

```python
import cohere
co = cohere.Client(os.environ['COHERE_API_KEY']) #获取 cohere 服务
```

```python
import weaviate
auth_config = weaviate.auth.AuthApiKey(
    api_key=os.environ['WEAVIATE_API_KEY'])
```

```python
client = weaviate.Client(
    url=os.environ['WEAVIATE_API_URL'],
    auth_client_secret=auth_config,
    additional_headers={
        "X-Cohere-Api-Key": os.environ['COHERE_API_KEY'],
    }
)
print(client.is_ready()) #检查服务是否已经连接
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>True</code></pre></details>
##  面向语义检索的向量数据库
![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/4-1.png)

如图所示，查询：“加拿大的首都是什么？” 假设此时有五个可能的回答或句子。可以像[文本映射到 Embeddings 空间](https://www.yuque.com/qiaokate/su87gb/zuwe3220zyhwacpy)中那样绘制它们。可以看到在图中，意义相似的句子会彼此靠近。因此，如果绘制这五个句子，会看到有关加拿大和法国首都的句子彼此靠近，而有关颜色的句子聚在右上方。

如果使用一个为搜索进行优化的 Embeddings 模型将 query 投影到相同的空间中，query 将最接近他所查询的答案。当询问“加拿大的首都是什么”时，它将最接近“加拿大的首都是渥太华”这个句子。

以上就是如何将 Embeddings 中学到的相似性和距离的特性应用于搜索。

```python
def dense_retrieval(query,
                    results_lang='en',
                    properties = ["text", "title", "url", "views", "lang", "_additional {distance}"],
                    num_results=5):
    """
    执行基于给定查询的稠密检索。

    参数：
    - query (str): 用作判断距离和相似性标准的查询。
    - results_lang (str): 过滤结果的语言（默认为英语，'en'）。
    - properties (list): 每个结果要检索的属性列表（默认包括"text"、"title"、"url"、"views"、"lang"和"_additional {distance}"）。
    - num_results (int): 要检索的结果数量。

    返回：
    - result (list): 基于稠密检索的检索到的文章列表。
    """
    nearText = {"concepts": [query]} #这里使用 query 作为评判距离和相似性的标准

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
```

这个函数对结果进行排版，使展示出来的结果更易读

```python
from utils import print_result #这个函数对结果进行排版，使展示出来的结果更易读
```

```python
def print_result(result):
    """ Print results with colorful formatting """
    for i,item in enumerate(result):
        print(f'item {i}')
        for key in item.keys():
            print(f"{key}:{item.get(key)}")
            print()
        print()
```

### 基础难度的查询
在此对输出结果中的部分内容进行解释：`_additional:{'distance': -154.75615}` 结果中的 distance 代表 query 和 result 之间的距离。

```python
query = "谁写了哈姆雷特？"
dense_retrieval_results = dense_retrieval(query,results_lang='zh')
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="u890fccdf"><span class="ne-text" style="color: var(--jp-content-font-color1)">output：</span></summary><pre data-language="json" id="eXTIE" class="ne-codeblock language-json"><code>item 0
_additional:{'distance': -150.74162}

lang:zh

text:《哈姆雷特》（）又名《王子復仇记》，是莎士比亞于1599年至1602年间的一部悲劇作品，是他最負盛名和被人引用最多的劇本。習慣上將本劇與《馬克白》、《李爾王》和《奧賽羅》一起，並稱為莎士比亞的“四大悲劇”。

title:哈姆雷特

url:https://zh.wikipedia.org/wiki?curid=72611

views:1000


item 1
_additional:{'distance': -150.65308}

lang:zh

text:威廉·霍尔特·耶茨（William Holt Yates）在1843年写道穆罕默德·阿里帕夏治下的总督阿卜杜拉·拉赫曼·贝伊（Abd-ur-Rahman Bey），据说特别残暴和贪婪。他是一个变节的科普特人，滥用职权敛财。人们甚至认为他曾将人锯成了两半。耶茨还增加了一些细节：“传闻这个家伙后来被暗杀了，是政府批准的。”

title:锯刑

url:https://zh.wikipedia.org/wiki?curid=7157095

views:100


item 2
_additional:{'distance': -149.99219}

lang:zh

text:《哈姆雷特》反映了法国文艺复兴时期人文主义者蒙泰涅的怀疑主义思想。在此之前，人文主义者米蘭多拉認為人是上帝最伟大的造物，具有上帝的形象，并可以选择自己的本性；然而这种观点在蒙泰涅的《隨筆集》中被反驳。哈姆雷特的“人类是一件多么了不得的杰作”与蒙泰涅的思想相呼应，但学者无法确认莎士比亚直接引用了蒙泰涅的作品，还是俩人一同对时代的气息做出了类似的反应。

title:哈姆雷特

url:https://zh.wikipedia.org/wiki?curid=72611

views:1000


item 3
_additional:{'distance': -149.72849}

lang:zh

text:1598年，弗朗西斯·梅洛斯出版了他的《智慧宝库》，涵盖了从乔叟到当时的英国文学，包括了莎士比亚的十二篇戏剧。然而，《哈姆雷特》不在其中，暗示戏剧在当时还没写完。《哈姆雷特》十分出名，《新天鹅》系列编辑伯纳德·罗德认为“梅洛斯不可能对如此重要的作品此视而不见。”

title:哈姆雷特

url:https://zh.wikipedia.org/wiki?curid=72611

views:1000


item 4
_additional:{'distance': -149.68834}

lang:zh

text:James Alfred Wight，筆名為詹姆士·赫利奧特(James Herriot)是英國一位知名的獸醫作家，以描寫獸醫為一系列的故事聞名。他出生於1916年10月3日出生於英國東北部的達拉謨郡（County Durham）。父James Wight（1890–1960）白天是一名造船鋼板工，夜晚則在當地一家戲院擔任鋼琴手、母Hannah Bell（1890–1980）為一名縫紉師兼歌手。詹姆士·赫利奧特從小在爸爸的影響下培養出了踢足球的興趣，終其一生都致力於Sunderland Association Football Club。原本他是以足球為寫作題材，卻因反應不佳而改寫獸醫生活，沒想到大受歡迎。

title:吉米·哈利

url:https://zh.wikipedia.org/wiki?curid=5928362

views:100</code></pre></details>
### 中等难度的查询功能
从运行结果中可以看出两种方式的差异，其中 `dense_retrieval` 识别的结果更加准确。

1. 山西在哪里？

```python
query = "山西在哪里？"
dense_retrieval_results = dense_retrieval(query,results_lang='zh')
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="u0d47233a"><span class="ne-text">output：</span></summary><pre data-language="json" id="Qd55k" class="ne-codeblock language-json"><code>item 0
_additional:{'distance': -154.62271}

lang:zh

text:山西处于中纬度地区。山西坐落于黃土高原東部，有太行山和呂梁山兩座大山，省內最高峰是五臺山，3058米。境内由东北到西南依次分布着大同盆地、忻州盆地、太原盆地、临汾盆地、运城盆地、长治盆地、晋城盆地，阳泉盆地、寿阳盆地、襄垣盆地、黎城盆地等盆地。接邻省区：河北、陕西、河南、内蒙古。

title:山西省

url:https://zh.wikipedia.org/wiki?curid=445

views:1000


item 1
_additional:{'distance': -152.94821}

lang:zh

text:山西疆域轮廓呈东北斜向西南的平行四边形，是典型的为黄土广泛覆盖的山地高原，地势东北高西南低。高原内部起伏不平，河谷纵横，地貌类型复杂多样，有山地、丘陵、台地、平原，山多川少，山地、丘陵面积占全省总面积的80.1%，平川、河谷面积占总面积的19.9%。全省大部分地区海拔在1500米以上，最高点为五台山主峰叶斗峰，海拔3061.1米，为华北最高峰。

title:山西省

url:https://zh.wikipedia.org/wiki?curid=445

views:1000


item 2
_additional:{'distance': -152.31085}

lang:zh

text:山西大学（，縮寫：），簡稱山大，创建于1902年5月8日，前身为山西大学堂，坐落于中华人民共和国山西省太原市小店区，是一所公立高等院校。現為山西省人民政府和教育部共同建設的部省合建大学、第二轮“双一流”建设高校、国家中西部高校综合实力提升工程（一省一校工程）重点建设的14所院校之一，同时还是中西部高校联盟成员。

title:山西大学

url:https://zh.wikipedia.org/wiki?curid=51821

views:100


item 3
_additional:{'distance': -152.27774}

lang:zh

text:山西省，简称晋，中華人民共和國的一個省份，地处黄土高原东翼。山西表里山河，南临黄河，西邻吕梁山，东靠太行山。因在太行山以西，故称山西。省会太原市。省境內春秋時為晉國之地，故簡稱晉。山西地區有獨特的语言、風俗，省內土地豐足、矿产資源豐富，位處汾河沿岸一帶的晋中盆地一直被稱作华北的「漁米之鄉」。山西历史悠久，临汾一带曾是尧都、大同曾是北魏首都。省会太原曾是赵国、前秦、东魏、北齐、北晋、后唐、后晋、后汉、北汉的都城。

title:山西省

url:https://zh.wikipedia.org/wiki?curid=445

views:1000


item 4
_additional:{'distance': -151.57559}

lang:zh

text:山南市（）是中华人民共和国西藏自治区东南部的地級市。山南市位于雅鲁藏布江干流中下游地区，在冈底斯山至念青唐古拉山以南，北接拉萨市，西连日喀则市，东接林芝市，南与印度、不丹两国接壤，部份地区与兩國有争议。

title:山南市

url:https://zh.wikipedia.org/wiki?curid=4020

views:200</code></pre></details>
2. 山西

```python
from utils import keyword_search

query = "山西"
keyword_search_results = keyword_search(query, client,results_lang='zh')
print_result(keyword_search_results)
```

<details class="lake-collapse"><summary id="ud1c10159"><span class="ne-text">output：</span></summary><pre data-language="json" id="XgWL6" class="ne-codeblock language-json"><code>item 0
_additional:None

lang:zh

text:嘉靖三十四年农历腊月十二（1556年1月23日），山西、陕西和河南同时发生地震。这次地震分布在陕西、山西、河南、甘肃等地，地震波及大半个中国，有感范围远达福建、两广等地。百姓民众因压砸、焚溺与饥疫而死者无法估计。死亡人口之多幾乎達到當時中國人口的百分之一，也是古今中外地震史上僅有的案例。这次大地震使陕西、山西、河南等省97州受灾，101个县受害，灾区面积大约28万平方公里。地震有感范围为5省227个县。“余震月动三五次者半年，未止息者三载，五年渐轻方止”。

title:嘉靖大地震

url:https://zh.wikipedia.org/wiki?curid=65483

views:900


item 1
_additional:None

lang:zh

text:嘉靖三十四年农历腊月十二（1556年1月23日），山西、陕西和河南同时发生地震。这次地震分布在陕西、山西、河南、甘肃等地，地震波及大半个中国大陸，有感范围远达福建、两广等地。百姓民众因压砸、焚溺、与饥疫而死者无法估计，其奏报有名者便达83万有多，不知名者不可胜数。死亡人口之多幾達當時人口的百分之一，也是古今中外地震史上僅有的案例。这次大地震致使陕西、山西、河南等省97州受灾，101个县受害，灾区面积大约28万平方公里。地震有感范围为5省227个县。“余震月动三五次者半年，未止息者三载，五年渐轻方止”。由于明代后期吏治腐败，国库空虚。地震发生后明朝从国库调拨大量资金用于救灾，导致明朝国库连续两年亏空，加上地震引发的自然灾害和瘟疫导致明朝政府税收减少，对明朝的国力和财政状况亦造成不同程度的影响。

title:明世宗

url:https://zh.wikipedia.org/wiki?curid=10973

views:1000


item 2
_additional:None

lang:zh

text:嘉靖三十四年农历腊月十二（1556年1月23日），山西、陕西和河南同时发生地震。这次地震分布在陕西、山西、河南、甘肃等地，地震波及大半个中国，有感范围远达福建、两广等地。百姓民众因压砸、焚溺、与饥疫而死者无法估计，其奏报有名者便达83万有多，不知名者不可胜数。死亡人口之多幾達當時中國人口的百分之一，也是古今中外地震史上僅有的案例。这次大地震致使陕西、山西、河南等省97州受灾，101个县受害，灾区面积大约28万平方公里。地震有感范围为5省227个县。“余震月动三五次者半年，未止息者三载，五年渐轻方止”。由于明代后期吏治腐败，国库空虚。地震发生后明朝从国库调拨大量资金用于救灾，导致明朝国库连续两年亏空，加上地震引发的自然灾害和瘟疫导致明朝政府税收减少，对明朝的国力和财政状况亦造成不同程度的影响。

title:明朝

url:https://zh.wikipedia.org/wiki?curid=428010

views:2000</code></pre></details>
### 更复杂的查询功能
问一些更难回答的问题，看看两种检索算法的差异。

1. 历史 最高 人

```python
from utils import keyword_search

query = "历史 最高 人"
keyword_search_results = keyword_search(query, client,results_lang='zh')
print_result(keyword_search_results)
```

<details class="lake-collapse"><summary id="u710e34be"><span class="ne-text">output：</span></summary><pre data-language="json" id="VKsmR" class="ne-codeblock language-json"><code>item 0
_additional:None

lang:zh

text:高雄市 106 年底現住人口數計 2,776,912 人；其中男性 1,375,515 人，女性 1,401,397 人；各分局轄區人口數以鳳山分局 359,120 人最多，三民二分局 263,988 人次之，六龜分局 22,311 人最少。民國 106 年底高雄市人口密度為 941 人；各分局轄區人口密度以三民二分局 29,573 人最高，六龜分局 14 人最低。民國 106 年底高雄市性比例為 98.15；各分局轄區性比例以六龜分局 112.10 最高，新興分局 92.11 最低。

title:高雄市政府警察局

url:https://zh.wikipedia.org/wiki?curid=1826158

views:300


item 1
_additional:None

lang:zh

text:昆仑山脉（；）西起帕米尔高原东部，东到柴达木河上游谷地，于东经97°至99°处与巴颜喀拉山脉和阿尼玛卿山（积石山）相接，北邻塔里木盆地与柴达木盆地。山脉全长2,500余公里，宽130至200公里，平均海拔5,500至6,000米，西窄东宽，总面积达50多万平方公里。一般认为最高峰是位於新疆维吾尔自治区和西藏自治区交界处的昆仑女神峰（7167米）最高。如将东帕米尔高原视为喀喇昆仑山的一部分，则公格尔峰（7649米）最高。昆仑火山群是周邊山區、高原上罕見的火山山區，有超過70座火山坐落於此，其中木吉火山海拔高達5808公尺（），但因其位於高原上，因此實際上火山錐實體僅高於周邊高原約300公尺（僅海拔的5％），但若僅考量絕對高度（海拔）而不考量相對高度而言，該山為亞洲、中國第一高、東半球第二高的火山（僅次於吉力馬扎羅山），也是之一，與伊朗的德馬峰並論，最後一次爆發時間為1951年5月27日。

title:昆仑山脉

url:https://zh.wikipedia.org/wiki?curid=58008

views:800


item 2
_additional:None

lang:zh

text:初中开设的课程为语文、数学、外语（通常为英语，某些地区有日语、俄语等课程）、道德与法治、历史、地理、物理、生物、化学、信息技术、美术、音乐、体育与健康、综合实践等，也有地区将物理、化学合并为科学课程，历史、地理合并为历史与社会课程。一般而言，语文，数学，英语，历史，道德与法治等科目将贯穿初中三年；地理，生物会在七年级和八年级开设，物理会在八年级和九年级开设，化学会在九年级开设。

title:初级中学

url:https://zh.wikipedia.org/wiki?curid=219440

views:200</code></pre></details>
2. 历史上最高的人是谁？

```python
query = "历史上最高的人是谁？"
dense_retrieval_results = dense_retrieval(query, results_lang='zh')
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="ubc6cec44"><span class="ne-text">output：</span></summary><pre data-language="json" id="XLOk6" class="ne-codeblock language-json"><code>item 0
_additional:{'distance': -150.69139}

lang:zh

text:鲍喜顺在2005年、2008年两度獲得吉尼斯世界纪录認證被承認是地球上因自然原因而長得最高的活人（其他擁有世界之最頭銜的長人不是已逝，如罗伯特·瓦德罗，就是因特殊病變而过度发育稱為巨人症，如列昂尼德·斯塔德尼克）。实际上，这一纪录在2007年即被乌克兰人列昂尼德·斯塔德尼克（Leonid Stadnyk）取代，但因其后来拒绝接受吉尼斯世界纪录组织测量身高而并未被该组织继续认可，故鲍喜顺重新成为世界最高人，直至2009年该纪录持有者被土耳其人蘇丹·科塞（Sultan Kösen）取代。

title:鮑喜順

url:https://zh.wikipedia.org/wiki?curid=180624

views:100


item 1
_additional:{'distance': -150.47076}

lang:zh

text:蘇丹·科塞（；），出生於土耳其马尔丁，是自2009年起被確認為全世界最高的人，被列入金氏世界紀錄大全，其雙手和腳掌亦打破金氏世界紀錄大全，腳掌長達40公分。2009年時，科塞高247公分，到了2012年，他高了4公分，達到251公分。

title:蘇丹·科塞

url:https://zh.wikipedia.org/wiki?curid=2840966

views:200


item 2
_additional:{'distance': -149.28877}

lang:zh

text:瑪麗蓮·沃斯·莎凡特（Marilyn vos Savant，）曾經被記載為吉尼斯世界記錄所認定擁有最高智商的人類及女性 (1984 to 1989)。她於1946年出生於美國密苏里州的圣路易斯，瑪麗蓮在剛滿10歲的1956年9月時初次接受史丹福-比奈智力測驗 （心智年齡比例智商），測得智商高達228，並登上世界紀錄。然而，智商的判定與比較方式後來遭到爭議， 隨後吉尼斯世界記錄在1990年移除了“智商最高的人”這個項目。

title:瑪麗蓮·沃斯·莎凡特

url:https://zh.wikipedia.org/wiki?curid=792357

views:100


item 3
_additional:{'distance': -147.5022}

lang:zh

text:萊茵霍爾德·梅斯納爾（，），來自南蒂羅爾的義大利登山家兼探險家，常被人稱為有史以來最偉大的登山運動員。最令他名聲遠揚的壯舉包括人類史上首次不用氧氣補給獨立登頂珠峰成功；他亦是第一個登頂世上所有十四座八千米山峰的人。

title:萊茵霍爾德·梅斯納爾

url:https://zh.wikipedia.org/wiki?curid=1000939

views:100


item 4
_additional:{'distance': -147.4777}

lang:zh

text:艾德蒙·珀西瓦尔·希拉里爵士，KG，ONZ，KBE（Edmund Percival Hillary，），是紐西蘭登山家和探險家，在和雪巴人嚮導丹增·诺盖的合作之下，他和丹增·诺盖成了可證明的記錄中最早成功攀登珠穆朗瑪峰峰頂的人。

title:艾德蒙·希拉里

url:https://zh.wikipedia.org/wiki?curid=164956

views:200</code></pre></details>
接下来，使用不同语言询问相同的问题，均用英文来回答。理论上不同语言被 Embeddings 映射后应该在临近的空间内，识别结果应该是相似的。

```python
query = "أطول رجل في التاريخ" #历史上最高的人是谁？
dense_retrieval_results = dense_retrieval(query)
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="ue825d46e"><span class="ne-text">output：</span></summary><pre data-language="json" id="Qw401" class="ne-codeblock language-json"><code>item 0
_additional:{'distance': -147.44199}

lang:en

text:Robert Pershing Wadlow (February 22, 1918 July 15, 1940), also known as the Alton Giant and the Giant of Illinois, was a man who was the tallest person in recorded history for whom there is irrefutable evidence. He was born and raised in Alton, Illinois, a small city near St. Louis, Missouri.

title:Robert Wadlow

url:https://en.wikipedia.org/wiki?curid=359117

views:3000


item 1
_additional:{'distance': -147.09518}

lang:en

text:Kösen turned 40 years old on 10 December 2022. He celebrated his birthday a few days early by visiting the Ripley's Believe It or Not! museum in Orlando, Florida, USA and posing next to a life-sized statue of Robert Wadlow, the tallest man ever at 272 cm (8 ft 11.1 in).

title:Sultan Kösen

url:https://en.wikipedia.org/wiki?curid=8445237

views:2000


item 2
_additional:{'distance': -146.9144}

lang:en

text:Bol and Gheorghe Mureșan are the two tallest players in the history of the National Basketball Association. Official NBA publications have listed Bol at either or tall. He was measured by the Guinness Book of World Records at 7 ft 6  in tall. Complementing his great height, Bol had exceptionally long limbs (inseam ) and large hands and feet (size 16 ). His arm span, at , is (as of 2013) the longest in NBA history, and his upward reach was . He was extremely slender, limiting his offensive capability.

title:Manute Bol

url:https://en.wikipedia.org/wiki?curid=283871

views:2000


item 3
_additional:{'distance': -146.62762}

lang:en

text:Sultan Kösen (born 10 December 1982) is a Turkish farmer who holds the Guinness World Record for tallest living male at . Of Kurdish ethnicity, he is the seventh tallest man in history.

title:Sultan Kösen

url:https://en.wikipedia.org/wiki?curid=8445237

views:2000


item 4
_additional:{'distance': -146.54651}

lang:en

text:Jonah Adam (Cardeli) Falcon (born July 29, 1970) is an American actor and television presenter. He came to international attention in 1999 because of his claim that he has the largest penis in the world, which he claims is long when erect; Falcon has not authorized or permitted independent verification of this figure.

title:Jonah Falcon

url:https://en.wikipedia.org/wiki?curid=22044645

views:2000</code></pre></details>
用日语提问，中文回答

```python
query = "歴史上、最も背が高かった人物は誰ですか？"
dense_retrieval_results = dense_retrieval(query,  results_lang='zh')
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="uc633d88b"><span class="ne-text">output：</span></summary><pre data-language="json" id="xTloi" class="ne-codeblock language-json"><code>item 0
_additional:{'distance': -149.38632}

lang:zh

text:蘇丹·科塞（；），出生於土耳其马尔丁，是自2009年起被確認為全世界最高的人，被列入金氏世界紀錄大全，其雙手和腳掌亦打破金氏世界紀錄大全，腳掌長達40公分。2009年時，科塞高247公分，到了2012年，他高了4公分，達到251公分。

title:蘇丹·科塞

url:https://zh.wikipedia.org/wiki?curid=2840966

views:200


item 1
_additional:{'distance': -147.42696}

lang:zh

text:鲍喜顺在2005年、2008年两度獲得吉尼斯世界纪录認證被承認是地球上因自然原因而長得最高的活人（其他擁有世界之最頭銜的長人不是已逝，如罗伯特·瓦德罗，就是因特殊病變而过度发育稱為巨人症，如列昂尼德·斯塔德尼克）。实际上，这一纪录在2007年即被乌克兰人列昂尼德·斯塔德尼克（Leonid Stadnyk）取代，但因其后来拒绝接受吉尼斯世界纪录组织测量身高而并未被该组织继续认可，故鲍喜顺重新成为世界最高人，直至2009年该纪录持有者被土耳其人蘇丹·科塞（Sultan Kösen）取代。

title:鮑喜順

url:https://zh.wikipedia.org/wiki?curid=180624

views:100


item 2
_additional:{'distance': -146.97598}

lang:zh

text:特斯拉身高1.88米，体重64公斤，从1888年到1926年左右，体重几乎没有变化。他的外表被报纸编辑者描述为“几乎是最高的，几乎是最瘦的，当然也是经常去的最严肃的人。”他在纽约市是一个优雅、时尚的人物，对自己的仪容、服装一丝不苟，日常活动有条不紊，他保持这种外表是为了促进他的商业关系。他还被描述为有一双浅色眼睛，“很大的手”和“极其大的拇指”。

title:尼古拉·特斯拉

url:https://zh.wikipedia.org/wiki?curid=349732

views:1000


item 3
_additional:{'distance': -146.52371}

lang:zh

text:19世纪学者-{zh-hans:弗朗茨·博厄斯; zh-hant:法蘭茲·鮑亞士;}-曾为芝加哥哥伦布纪念博览会搜集了全球各种族的身高数据，当代学者分析该数据得出平原印第安人是19世纪晚期平均身高最高的人群。这对于人体测量学的发展尤为重要，因为在学界的传统观念中，平均身高的增长往往要和生活水平和卫生水平的提高挂钩。

title:平原印第安人

url:https://zh.wikipedia.org/wiki?curid=7589221

views:100


item 4
_additional:{'distance': -145.81403}

lang:zh

text:波爾以7呎7吋(約231公分)的登錄身高和格奧爾基·穆瑞森並列NBA史上最高的球員，兩人曾在1994年一同效力於華盛頓子彈(現在的華盛頓巫師前身)。波爾在NBA打了十個球季，先後效力過華盛頓子彈、金州勇士、費城76人、邁阿密熱火等隊。

title:马努特·波尔

url:https://zh.wikipedia.org/wiki?curid=3351637

views:300</code></pre></details>
##  从头构建语义向量库
构建语义向量库需要先对数据进行预处理，之后将处理好的文本映射成 Embeddings ，并用 Annoy 库为 Embeddings 构建索引。这样就构建了一个完整的语义向量库。

### 文本预处理
```python
from annoy import AnnoyIndex
import numpy as np
import pandas as pd
import re
```

```python
text_zh = """

大革命时期(1919年—1927年).1919年爆发的五四运动成为新民主主义革命的开端,并直接促成1921年7月23日中国共产党第一次全国代表大会召开,宣告中国共产党的成立.此后,中国共产党不断发起工人暴动,反抗北洋政府统治.1924年国民党“一大”后,国民党与共产党实现第一次合作,促成1926年开始的北伐战争的胜利进行.1927年,国民党右派接连发动“四一二”反革命政变和“七一五”反革命政变,第一次国共合作破裂,国民大革命宣告失败.

土地革命战争时期(1927年—1937年).1927年8月1日,中国共产党领导南昌起义,开始武装反抗国民党的反动统治.1927年9月秋收起义后,确定了“农村包围城市,武装夺取政权”的革命道路,开辟了以井冈山为代表的无数农村革命根据地,并成功粉碎国民党的四次“围剿”.1933年—1934年,由于王明"左倾"错误路线影响,第五次反围剿失败.1934年10月开始中国工农红军被迫进行长征.1936年10月三大主力会师甘肃会宁,标志着长征的胜利结束.其间,日本侵占中国东北并不断向南推进.中共主张停止内战,一致抗日；而国民党采取“攘外必先安内”的不抵抗政策,导致国土沦丧.1936年西安事变和平解决后,国共第二次合作初步形成.

抗日战争时期(1931年—1945年).以1931年“九一八”事变为起点,中国人民进入了艰苦卓绝的14年抗战时期.以国共第二次合作为标志,抗日民族统一战线形成,全国人民团结一心,最终打败了日本侵略者,维护了国家的主权独立,极大地提高了国际地位.中国在此后成为联合国安理会五大常任理事国之一.

解放战争时期(1945年—1949年).1945年抗战胜利后,国共进行重庆谈判,签订关于和平建国问题的协定(即《双十协定》).1946年,蒋介石撕毁《双十协定》,发动内战.在中共的英明领导和人民群众的大力支持下,解放战争最终获得胜利.1949年10月1日,中华人民共和国中央人民政府成立,标志着新民主主义革命的基本结束和社会主义革命的开始.

"""
```

### 切块 chunking
#### 为什么要切块？
1. GPT 模型大多具有数十亿甚至上百亿的参数。进行一次前向传播需要大量的计算能力和内存。但是，大多数硬件设备（如 GPU ）都有内存限制，文档切块使模型能够在这些限制内工作。
2. GPT 模型有一个固定的最大序列长度，例如 1024 个 token。这意味着模型一次只能处理这么多 token。对于超过这个长度的文档，需要进行切块才能被模型处理。
3. 通过在多个文档块上进行训练，模型可以更好地学习和泛化到各种不同的文本样式和结构。
4. 对文档切块可以为训练数据提供更多的样本。例如，一个长文档可以被切块成多个部分，并分别作为单独的训练样本。

#### 本节使用两种方式进行切块
1. **按照句子进行切块**。生成的 Embedding 关注句子级别的信息，基本不会出现超过模型上下文长度的情况。但生成的向量集中在句子的特定含义上，缺失了很多上下文的信息。这也意味着检索时可能会错过在段落或文档中找到的更广泛的上下文信息。
2. **按照段落进行切块**。当生成一个完整段落或文档的 Embedding 时，既考虑了整个上下文，也可以考虑到文本中句子和短语之间的关系。这可以产生更全面的向量表示，从而捕获文本的更广泛的含义和主题。但是，较大的输入文本大小可能会引入噪声或淡化单个句子或短语的重要性，从而在查询时难以找到精确的匹配。而且较长的段落依旧可能超出上下文限制。

![](https://raw.githubusercontent.com/datawhalechina/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/content/%E9%80%89%E4%BF%AE-Large%20Language%20Models%20with%20Semantic%20Search/images/4-2.png)

如图所示，切块后将 query 以及搜索内容都映射到 Embeddings 空间，对比 query 和搜索内容的相似性以及距离，将会寻找到最符合的答案。

##### 句子切块
```python
# 对句子进行拆分
texts_zh = text_zh.split('.')

# 清除回车和分行
texts_zh = np.array([t.strip(' \n') for t in texts_zh])

print(texts_zh)
```

```python
['大革命时期(1919年—1927年)'
 '1919年爆发的五四运动成为新民主主义革命的开端,并直接促成1921年7月23日中国共产党第一次全国代表大会召开,宣告中国共产党的成立'
 '此后,中国共产党不断发起工人暴动,反抗北洋政府统治'
 '1924年国民党“一大”后,国民党与共产党实现第一次合作,促成1926年开始的北伐战争的胜利进行'
 '1927年,国民党右派接连发动“四一二”反革命政变和“七一五”反革命政变,第一次国共合作破裂,国民大革命宣告失败'
 '土地革命战争时期(1927年—1937年)' '1927年8月1日,中国共产党领导南昌起义,开始武装反抗国民党的反动统治'
 '1927年9月秋收起义后,确定了“农村包围城市,武装夺取政权”的革命道路,开辟了以井冈山为代表的无数农村革命根据地,并成功粉碎国民党的四次“围剿”'
 '1933年—1934年,由于王明"左倾"错误路线影响,第五次反围剿失败' '1934年10月开始中国工农红军被迫进行长征'
 '1936年10月三大主力会师甘肃会宁,标志着长征的胜利结束' '其间,日本侵占中国东北并不断向南推进'
 '中共主张停止内战,一致抗日；而国民党采取“攘外必先安内”的不抵抗政策,导致国土沦丧' '1936年西安事变和平解决后,国共第二次合作初步形成'
 '抗日战争时期(1931年—1945年)' '以1931年“九一八”事变为起点,中国人民进入了艰苦卓绝的14年抗战时期'
 '以国共第二次合作为标志,抗日民族统一战线形成,全国人民团结一心,最终打败了日本侵略者,维护了国家的主权独立,极大地提高了国际地位'
 '中国在此后成为联合国安理会五大常任理事国之一' '解放战争时期(1945年—1949年)'
 '1945年抗战胜利后,国共进行重庆谈判,签订关于和平建国问题的协定(即《双十协定》)' '1946年,蒋介石撕毁《双十协定》,发动内战'
 '在中共的英明领导和人民群众的大力支持下,解放战争最终获得胜利'
 '1949年10月1日,中华人民共和国中央人民政府成立,标志着新民主主义革命的基本结束和社会主义革命的开始' '']
```

##### 段落切块
```python
# 对段落进行拆分
texts_zh = text_zh.split('\n\n')

# 删除空格和新行
texts_zh = np.array([t.strip(' \n') for t in texts_zh])

print(texts_zh)
```

```python
[''
 '大革命时期(1919年—1927年).1919年爆发的五四运动成为新民主主义革命的开端,并直接促成1921年7月23日中国共产党第一次全国代表大会召开,宣告中国共产党的成立.此后,中国共产党不断发起工人暴动,反抗北洋政府统治.1924年国民党“一大”后,国民党与共产党实现第一次合作,促成1926年开始的北伐战争的胜利进行.1927年,国民党右派接连发动“四一二”反革命政变和“七一五”反革命政变,第一次国共合作破裂,国民大革命宣告失败.'
 '土地革命战争时期(1927年—1937年).1927年8月1日,中国共产党领导南昌起义,开始武装反抗国民党的反动统治.1927年9月秋收起义后,确定了“农村包围城市,武装夺取政权”的革命道路,开辟了以井冈山为代表的无数农村革命根据地,并成功粉碎国民党的四次“围剿”.1933年—1934年,由于王明"左倾"错误路线影响,第五次反围剿失败.1934年10月开始中国工农红军被迫进行长征.1936年10月三大主力会师甘肃会宁,标志着长征的胜利结束.其间,日本侵占中国东北并不断向南推进.中共主张停止内战,一致抗日；而国民党采取“攘外必先安内”的不抵抗政策,导致国土沦丧.1936年西安事变和平解决后,国共第二次合作初步形成.'
 '抗日战争时期(1931年—1945年).以1931年“九一八”事变为起点,中国人民进入了艰苦卓绝的14年抗战时期.以国共第二次合作为标志,抗日民族统一战线形成,全国人民团结一心,最终打败了日本侵略者,维护了国家的主权独立,极大地提高了国际地位.中国在此后成为联合国安理会五大常任理事国之一.'
 '解放战争时期(1945年—1949年).1945年抗战胜利后,国共进行重庆谈判,签订关于和平建国问题的协定(即《双十协定》).1946年,蒋介石撕毁《双十协定》,发动内战.在中共的英明领导和人民群众的大力支持下,解放战争最终获得胜利.1949年10月1日,中华人民共和国中央人民政府成立,标志着新民主主义革命的基本结束和社会主义革命的开始.'
 '']
```

##### 句子拆分
```python
# 对句子进行拆分
texts_zh = text_zh.split('.')

# 清楚空格和分行
texts_zh = np.array([t.strip(' \n') for t in texts_zh])
```

##### 为每个句子前添加整段文字的主题内容
为了增加每个分句对模型的可读性，为每个句子前添加整段文字的主题内容

```python
title_zh = '新民主主义革命4个发展阶段'

texts_zh = np.array([f"{title_zh} {t}" for t in texts_zh])
```

```python
print(texts_zh)
```

<details class="lake-collapse"><summary id="u6d849586"><span class="ne-text">output：</span></summary><pre data-language="json" id="PhzAd" class="ne-codeblock language-json"><code>['新民主主义革命4个发展阶段 大革命时期(1919年—1927年)'
 '新民主主义革命4个发展阶段 1919年爆发的五四运动成为新民主主义革命的开端,并直接促成1921年7月23日中国共产党第一次全国代表大会召开,宣告中国共产党的成立'
 '新民主主义革命4个发展阶段 此后,中国共产党不断发起工人暴动,反抗北洋政府统治'
 '新民主主义革命4个发展阶段 1924年国民党“一大”后,国民党与共产党实现第一次合作,促成1926年开始的北伐战争的胜利进行'
 '新民主主义革命4个发展阶段 1927年,国民党右派接连发动“四一二”反革命政变和“七一五”反革命政变,第一次国共合作破裂,国民大革命宣告失败'
 '新民主主义革命4个发展阶段 土地革命战争时期(1927年—1937年)'
 '新民主主义革命4个发展阶段 1927年8月1日,中国共产党领导南昌起义,开始武装反抗国民党的反动统治'
 '新民主主义革命4个发展阶段 1927年9月秋收起义后,确定了“农村包围城市,武装夺取政权”的革命道路,开辟了以井冈山为代表的无数农村革命根据地,并成功粉碎国民党的四次“围剿”'
 '新民主主义革命4个发展阶段 1933年—1934年,由于王明&quot;左倾&quot;错误路线影响,第五次反围剿失败'
 '新民主主义革命4个发展阶段 1934年10月开始中国工农红军被迫进行长征'
 '新民主主义革命4个发展阶段 1936年10月三大主力会师甘肃会宁,标志着长征的胜利结束'
 '新民主主义革命4个发展阶段 其间,日本侵占中国东北并不断向南推进'
 '新民主主义革命4个发展阶段 中共主张停止内战,一致抗日；而国民党采取“攘外必先安内”的不抵抗政策,导致国土沦丧'
 '新民主主义革命4个发展阶段 1936年西安事变和平解决后,国共第二次合作初步形成'
 '新民主主义革命4个发展阶段 抗日战争时期(1931年—1945年)'
 '新民主主义革命4个发展阶段 以1931年“九一八”事变为起点,中国人民进入了艰苦卓绝的14年抗战时期'
 '新民主主义革命4个发展阶段 以国共第二次合作为标志,抗日民族统一战线形成,全国人民团结一心,最终打败了日本侵略者,维护了国家的主权独立,极大地提高了国际地位'
 '新民主主义革命4个发展阶段 中国在此后成为联合国安理会五大常任理事国之一'
 '新民主主义革命4个发展阶段 解放战争时期(1945年—1949年)'
 '新民主主义革命4个发展阶段 1945年抗战胜利后,国共进行重庆谈判,签订关于和平建国问题的协定(即《双十协定》)'
 '新民主主义革命4个发展阶段 1946年,蒋介石撕毁《双十协定》,发动内战'
 '新民主主义革命4个发展阶段 在中共的英明领导和人民群众的大力支持下,解放战争最终获得胜利'
 '新民主主义革命4个发展阶段 1949年10月1日,中华人民共和国中央人民政府成立,标志着新民主主义革命的基本结束和社会主义革命的开始'
 '新民主主义革命4个发展阶段 ']</code></pre></details>
### 生成 Embeddings
将处理后的 texts 传入 cohere 的 embed 函数中生成 Embeddings

```python
response_zh = co.embed(
    texts=texts_zh.tolist(),
    model="embed-multilingual-v2.0"
).embeddings #用 text 生成 Embeddings
```

查看一下生成的 embeds 的形状，可以看到不同长度的句子都被映射成相同维度的 Embeddings

```python
embeds_zh = np.array(response_zh)
print(embeds_zh.shape)
```

<details class="lake-collapse"><summary id="ude2d7ff0"><span class="ne-text">output：</span></summary><pre data-language="json" id="NOsxX" class="ne-codeblock language-json"><code>(24, 768)</code></pre></details>
使用的模型不同，所以 Embedding 维度会有所不同

### 创建搜索索引
在这一小节中会用到 Annoy 库构建索引。

[什么是 Annoy](https://www.yuque.com/qiaokate/su87gb/igvzvhhgdk62r9pm)

```python
search_index_zh = AnnoyIndex(embeds_zh.shape[1], 'angular')# 这里使用 Annoy 算法构建与 embeds 相同个数的索引。

# 将所有的向量添加到搜索索引中
for i in range(len(embeds_zh)):
    search_index_zh.add_item(i, embeds_zh[i])

search_index_zh.build(10) # 10 棵树
search_index_zh.save('test_zh.ann')
```

<details class="lake-collapse"><summary id="uf7942574"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="KyyO5" class="ne-codeblock language-json"><code>True</code></pre></details>
构建 search 函数的主要操作为：

1. 把 query 映射到 Embeddings 空间
2. 采用最近邻算法搜寻 search_index 中与 query 距离最相近的向量
3. 生成结果

```python
def search_zh(query):

  # 先把 query 映射到 Embeddings 空间
  query_embed = co.embed(texts=[query], model="embed-multilingual-v2.0").embeddings

  # 找到映射空间内与 query_embed 最邻近的 text
  similar_item_ids = search_index_zh.get_nns_by_vector(query_embed[0],
                                                    3,
                                                  include_distances=True)
  # 生成结果
  results = pd.DataFrame(data={'texts': texts_zh[similar_item_ids[0]],
                              'distance': similar_item_ids[1]})

  print(texts_zh[similar_item_ids[0]])

  return results
```

调用`search_zh`

```python
query = "大革命时期是哪几年？"
search_zh(query)
```

<details class="lake-collapse"><summary id="u085117cf"><span class="ne-text">output：</span></summary><pre data-language="json" id="PA2TH" class="ne-codeblock language-json"><code>['新民主主义革命4个发展阶段 大革命时期(1919年—1927年)' '新民主主义革命4个发展阶段 土地革命战争时期(1927年—1937年)'
 '新民主主义革命4个发展阶段 解放战争时期(1945年—1949年)']</code></pre></details>
将结果存储为 pandas 的 Dataframe 格式，可以让更好的分析结果

|  | **texts** | **distance** |
| :--- | :--- | :--- |
| **0** | 新民主主义革命4个发展阶段 大革命时期(1919年—1927年) | 0.370540 |
| **1** | 新民主主义革命4个发展阶段 土地革命战争时期(1927年—1937年) | 0.425366 |
| **2** | 新民主主义革命4个发展阶段 解放战争时期(1945年—1949年) | 0.426011 |


## 英文版本
### 基础难度
```python
query = "Who wrote Hamlet?"
dense_retrieval_results = dense_retrieval(query)
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="uae04bc0a"><span class="ne-text">output：</span></summary><pre data-language="json" id="nx9wG" class="ne-codeblock language-json"><code>item 0
_additional:{'distance': -154.75615}

lang:en

text:There are many works that have been pointed to as possible sources for Shakespeare's play—from ancient Greek tragedies to Elizabethan plays. The editors of the Arden Shakespeare question the idea of &quot;source hunting&quot;, pointing out that it presupposes that authors always require ideas from other works for their own, and suggests that no author can have an original idea or be an originator. When Shakespeare wrote there were many stories about sons avenging the murder of their fathers, and many about clever avenging sons pretending to be foolish in order to outsmart their foes. This would include the story of the ancient Roman, Lucius Junius Brutus, which Shakespeare apparently knew, as well as the story of Amleth, which was preserved in Latin by 13th-century chronicler Saxo Grammaticus in his &quot;Gesta Danorum&quot;, and printed in Paris in 1514. The Amleth story was subsequently adapted and then published in French in 1570 by the 16th-century scholar François de Belleforest. It has a number of plot elements and major characters in common with Shakespeare's &quot;Hamlet&quot;, and lacks others that are found in Shakespeare. Belleforest's story was first published in English in 1608, after &quot;Hamlet&quot; had been written, though it's possible that Shakespeare had encountered it in the French-language version.

title:Hamlet

url:https://en.wikipedia.org/wiki?curid=13554

views:3000


item 1
_additional:{'distance': -154.56943}

lang:en

text:English poet John Milton was an early admirer of Shakespeare and took evident inspiration from his work. As John Kerrigan discusses, Milton originally considered writing his epic poem &quot;Paradise Lost&quot; (1667) as a tragedy. While Milton did not ultimately go that route, the poem still shows distinct echoes of Shakespearean revenge tragedy, and of &quot;Hamlet&quot; in particular. As scholar Christopher N. Warren argues, &quot;Paradise Lost&quot;s Satan &quot;undergoes a transformation in the poem from a Hamlet-like avenger into a Claudius-like usurper,&quot; a plot device that supports Milton's larger Republican internationalist project. The poem also reworks theatrical language from &quot;Hamlet&quot;, especially around the idea of &quot;putting on&quot; certain dispositions, as when Hamlet puts on &quot;an antic disposition,&quot; similarly to the Son in &quot;Paradise Lost&quot; who &quot;can put on / [God's] terrors.&quot;

title:Hamlet

url:https://en.wikipedia.org/wiki?curid=13554

views:3000


item 2
_additional:{'distance': -154.2872}

lang:en

text:The Tragedy of Hamlet, Prince of Denmark, often shortened to Hamlet (), is a tragedy written by William Shakespeare sometime between 1599 and 1601. It is Shakespeare's longest play, with 29,551 words. Set in Denmark, the play depicts Prince Hamlet and his attempts to exact revenge against his uncle, Claudius, who has murdered Hamlet's father in order to seize his throne and marry Hamlet's mother.

title:Hamlet

url:https://en.wikipedia.org/wiki?curid=13554

views:3000


item 3
_additional:{'distance': -153.54633}

lang:en

text:In 1598, Francis Meres published his &quot;Palladis Tamia&quot;, a survey of English literature from Chaucer to its present day, within which twelve of Shakespeare's plays are named. &quot;Hamlet&quot; is not among them, suggesting that it had not yet been written. As &quot;Hamlet&quot; was very popular, Bernard Lott, the series editor of &quot;New Swan&quot;, believes it &quot;unlikely that he [Meres] would have overlooked ... so significant a piece&quot;.

title:Hamlet

url:https://en.wikipedia.org/wiki?curid=13554

views:3000


item 4
_additional:{'distance': -153.43037}

lang:en

text:Shakespeare almost certainly wrote the role of Hamlet for Richard Burbage. He was the chief tragedian of the Lord Chamberlain's Men, with a capacious memory for lines and a wide emotional range. Judging by the number of reprints, &quot;Hamlet&quot; appears to have been Shakespeare's fourth most popular play during his lifetime—only &quot;Henry IV Part 1&quot;, &quot;Richard III&quot; and &quot;Pericles&quot; eclipsed it. Shakespeare provides no clear indication of when his play is set; however, as Elizabethan actors performed at the Globe in contemporary dress on minimal sets, this would not have affected the staging.

title:Hamlet

url:https://en.wikipedia.org/wiki?curid=13554

views:3000</code></pre></details>
### 中等难度
```python
query = "What is the capital of Canada?"
dense_retrieval_results = dense_retrieval(query)
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="uc45d38a6"><span class="ne-text">output：</span></summary><pre data-language="json" id="Vrnb6" class="ne-codeblock language-json"><code>item 0
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
### 复杂难度
```python
from utils import keyword_search

query = "Tallest person in history?"
keyword_search_results = keyword_search(query, client)
print_result(keyword_search_results)
```

<details class="lake-collapse"><summary id="uf8f1c818"><span class="ne-text">output：</span></summary><pre data-language="json" id="cUaRd" class="ne-codeblock language-json"><code>item 0
_additional:None

lang:en

text:The population of Japan peaked at 128,083,960 in 2008. It had decreased by 2,373,960 by December 2020. In 2011, the economy of China became the world's second largest. Japan's economy descended to third largest by nominal GDP. Despite Japan's economic difficulties, this period also saw Japanese popular culture, including video games, anime, and manga, expanding worldwide, especially among young people. In March 2011, the Tokyo Skytree became the tallest tower in the world at , displacing the Canton Tower. It is the second tallest structure in the world after the Burj Khalifa ().

title:History of Japan

url:https://en.wikipedia.org/wiki?curid=25890428

views:2000


item 1
_additional:None

lang:en

text:Alpinism author Jon Krakauer (1997) wrote in &quot;Into Thin Air&quot; that it would be a bigger challenge to climb the second-highest peak of each continent, known as the Seven Second Summits – a feat that was not accomplished until January 2013. This discussion had previously been published in an article titled &quot;The Second Seven Summits&quot; in Rock &amp; Ice Magazine (#77) authored by the mountaineer and Seven Summits completer David Keaton. This is especially true for Asia, as K2 (8,611 m) demands greater technical climbing skills than Everest (8,848 m), while altitude-related factors such as the thinness of the atmosphere, high winds and low temperatures remain much the same. Some of those completing the seven ascents are aware of the magnitude of the challenge. In 2000, in a foreword to Steve Bell et al., &quot;Seven Summits&quot;, Morrow opined &quot;[t]he only reason Reinhold [Messner] wasn’t the first person to complete the seven was that he was too busy gambolling up the 14 tallest mountains in the world.&quot;

title:Seven Summits

url:https://en.wikipedia.org/wiki?curid=220861

views:2000


item 2
_additional:None

lang:en

text:Hamad was able to focus on turning Qatar from a small desert backwater into a major world power by continuing to exploit the country’s vast oil fields and discovering and tapping the world’s third largest gas reserves. By 2010 liquefied natural gas production had reached 77 million tons, making Qatar the richest country in the world. With fewer than two million inhabitants, the average income in the country shot to a staggering $86,440 per year per person. Qatar expert Olivier Da Lage said: &quot;When he came to power in 1995, Sheikh Hamad had a goal to place Qatar on the world map by exploiting the gas resources which his father did not develop for fear it would change the emirate's society. Eighteen years on, he has finished the job – Qatar has acquired the financial clout to command respect from neighboring countries and Western governments alike&quot;.In 2005, under the direction of Hamad and the former prime minister of Qatar Sheikh Hamad bin Jassim bin Jaber Al Thani, the Qatar Investment Authority was established, a sovereign wealth fund to manage the country's oil and natural gas surpluses. The Qatar Investment Authority and its subsidiaries have acquired many businesses abroad, including London's iconic department store Harrods from entrepreneur Mohammed Al-Fayed, Paris-based department store Printemps, French football club Paris Saint-Germain F.C., a former 10% stake in Porsche, a 75% stake in film studio Miramax Films which they acquired from Disney, a 2% stake in media conglomerate and Universal Music Group parent company Vivendi, a $100 million USD investment in Chernin Group – whose founder Peter Chernin was COO of News Corp and President of Fox, a 1% stake in luxury goods manufacturer Louis Vuitton Moët Hennessy, a 6% stake in Credit Suisse, a 12.6% stake in Barclays and several other major companies. They also backed Glencore's $31 billion takeover bid for Xstrata. Qatar is the largest property owner in London with their holdings including the United Kingdom's tallest building The Shard, the London Olympic Village and the InterContinental London Park Lane hotel. They also own several hotels in Cannes including the Majestic Hotel, Grand Hyatt Cannes Hôtel Martinez and the Carlton Hotel, Cannes. QIA was considered to have one of the leading bids in the sales of both Anschutz Entertainment Group and Hulu. As of May 2013, it was reported the Investment Authority was in talks to purchase Neiman Marcus and Bergdorf Goodman.

title:Hamad bin Khalifa Al Thani

url:https://en.wikipedia.org/wiki?curid=295929

views:2000</code></pre></details>


```python
query = "Tallest person in history"
dense_retrieval_results = dense_retrieval(query)
print_result(dense_retrieval_results)
```

<details class="lake-collapse"><summary id="u97f4a6a1"><span class="ne-text">output：</span></summary><pre data-language="json" id="uVhXe" class="ne-codeblock language-json"><code>item 0
_additional:{'distance': -148.98888}

lang:en

text:Robert Pershing Wadlow (February 22, 1918 July 15, 1940), also known as the Alton Giant and the Giant of Illinois, was a man who was the tallest person in recorded history for whom there is irrefutable evidence. He was born and raised in Alton, Illinois, a small city near St. Louis, Missouri.

title:Robert Wadlow

url:https://en.wikipedia.org/wiki?curid=359117

views:3000


item 1
_additional:{'distance': -148.09317}

lang:en

text:Bol came from a family of extraordinarily tall men and women. He said: &quot;My mother was , my father , and my sister is . And my great-grandfather was even taller—.&quot; His ethnic group, the Dinka, and the Nilotic people of which they are a part, are among the tallest populations in the world. Bol's hometown, Turalei, is the origin of other exceptionally tall people, including basketball player Ring Ayuel. &quot;I was born in a village, where you cannot measure yourself,&quot; Bol reflected. &quot;I learned I was 7 foot 7 in 1979, when I was grown. I was about 18 or 19.&quot;

title:Manute Bol

url:https://en.wikipedia.org/wiki?curid=283871

views:2000


item 2
_additional:{'distance': -146.75945}

lang:en

text:One year before his death, Wadlow passed John Rogan as the tallest person ever. On June 27, 1940 (18 days before his death), he was measured by doctors at .

title:Robert Wadlow

url:https://en.wikipedia.org/wiki?curid=359117

views:3000


item 3
_additional:{'distance': -146.63979}

lang:en

text:Kösen turned 40 years old on 10 December 2022. He celebrated his birthday a few days early by visiting the Ripley's Believe It or Not! museum in Orlando, Florida, USA and posing next to a life-sized statue of Robert Wadlow, the tallest man ever at 272 cm (8 ft 11.1 in).

title:Sultan Kösen

url:https://en.wikipedia.org/wiki?curid=8445237

views:2000


item 4
_additional:{'distance': -146.57892}

lang:en

text:Sultan Kösen (born 10 December 1982) is a Turkish farmer who holds the Guinness World Record for tallest living male at . Of Kurdish ethnicity, he is the seventh tallest man in history.

title:Sultan Kösen

url:https://en.wikipedia.org/wiki?curid=8445237

views:2000</code></pre></details>
### 文本预处理
```python
from annoy import AnnoyIndex
import numpy as np
import pandas as pd
import re
```

```python
text = """
Interstellar is a 2014 epic science fiction film co-written, directed, and produced by Christopher Nolan.
It stars Matthew McConaughey, Anne Hathaway, Jessica Chastain, Bill Irwin, Ellen Burstyn, Matt Damon, and Michael Caine.
Set in a dystopian future where humanity is struggling to survive, the film follows a group of astronauts who travel through a wormhole near Saturn in search of a new home for mankind.

Brothers Christopher and Jonathan Nolan wrote the screenplay, which had its origins in a script Jonathan developed in 2007.
Caltech theoretical physicist and 2017 Nobel laureate in Physics[4] Kip Thorne was an executive producer, acted as a scientific consultant, and wrote a tie-in book, The Science of Interstellar.
Cinematographer Hoyte van Hoytema shot it on 35 mm movie film in the Panavision anamorphic format and IMAX 70 mm.
Principal photography began in late 2013 and took place in Alberta, Iceland, and Los Angeles.
Interstellar uses extensive practical and miniature effects and the company Double Negative created additional digital effects.

Interstellar premiered on October 26, 2014, in Los Angeles.
In the United States, it was first released on film stock, expanding to venues using digital projectors.
The film had a worldwide gross over $677 million (and $773 million with subsequent re-releases), making it the tenth-highest grossing film of 2014.
It received acclaim for its performances, direction, screenplay, musical score, visual effects, ambition, themes, and emotional weight.
It has also received praise from many astronomers for its scientific accuracy and portrayal of theoretical astrophysics. Since its premiere, Interstellar gained a cult following,[5] and now is regarded by many sci-fi experts as one of the best science-fiction films of all time.
Interstellar was nominated for five awards at the 87th Academy Awards, winning Best Visual Effects, and received numerous other accolades"""
```

### 句子切块
```python
# 对句子进行拆分
texts = text.split('.')

# 清除回车和分行
texts = np.array([t.strip(' \n') for t in texts])

print(texts)
```

```python
['Interstellar is a 2014 epic science fiction film co-written, directed, and produced by Christopher Nolan'
 'It stars Matthew McConaughey, Anne Hathaway, Jessica Chastain, Bill Irwin, Ellen Burstyn, Matt Damon, and Michael Caine'
 'Set in a dystopian future where humanity is struggling to survive, the film follows a group of astronauts who travel through a wormhole near Saturn in search of a new home for mankind'
 'Brothers Christopher and Jonathan Nolan wrote the screenplay, which had its origins in a script Jonathan developed in 2007'
 'Caltech theoretical physicist and 2017 Nobel laureate in Physics[4] Kip Thorne was an executive producer, acted as a scientific consultant, and wrote a tie-in book, The Science of Interstellar'
 'Cinematographer Hoyte van Hoytema shot it on 35 mm movie film in the Panavision anamorphic format and IMAX 70 mm'
 'Principal photography began in late 2013 and took place in Alberta, Iceland, and Los Angeles'
 'Interstellar uses extensive practical and miniature effects and the company Double Negative created additional digital effects'
 'Interstellar premiered on October 26, 2014, in Los Angeles'
 'In the United States, it was first released on film stock, expanding to venues using digital projectors'
 'The film had a worldwide gross over $677 million (and $773 million with subsequent re-releases), making it the tenth-highest grossing film of 2014'
 'It received acclaim for its performances, direction, screenplay, musical score, visual effects, ambition, themes, and emotional weight'
 'It has also received praise from many astronomers for its scientific accuracy and portrayal of theoretical astrophysics'
 'Since its premiere, Interstellar gained a cult following,[5] and now is regarded by many sci-fi experts as one of the best science-fiction films of all time'
 'Interstellar was nominated for five awards at the 87th Academy Awards, winning Best Visual Effects, and received numerous other accolades']
```

### 段落切块
```python
# 对段落进行拆分
texts = text.split('\n\n')

# 删除空格和新行
texts = np.array([t.strip(' \n') for t in texts])

print(texts)
```

```python
['Interstellar is a 2014 epic science fiction film co-written, directed, and produced by Christopher Nolan.\nIt stars Matthew McConaughey, Anne Hathaway, Jessica Chastain, Bill Irwin, Ellen Burstyn, Matt Damon, and Michael Caine.\nSet in a dystopian future where humanity is struggling to survive, the film follows a group of astronauts who travel through a wormhole near Saturn in search of a new home for mankind.'
 'Brothers Christopher and Jonathan Nolan wrote the screenplay, which had its origins in a script Jonathan developed in 2007.\nCaltech theoretical physicist and 2017 Nobel laureate in Physics[4] Kip Thorne was an executive producer, acted as a scientific consultant, and wrote a tie-in book, The Science of Interstellar.\nCinematographer Hoyte van Hoytema shot it on 35 mm movie film in the Panavision anamorphic format and IMAX 70 mm.\nPrincipal photography began in late 2013 and took place in Alberta, Iceland, and Los Angeles.\nInterstellar uses extensive practical and miniature effects and the company Double Negative created additional digital effects.'
 'Interstellar premiered on October 26, 2014, in Los Angeles.\nIn the United States, it was first released on film stock, expanding to venues using digital projectors.\nThe film had a worldwide gross over $677 million (and $773 million with subsequent re-releases), making it the tenth-highest grossing film of 2014.\nIt received acclaim for its performances, direction, screenplay, musical score, visual effects, ambition, themes, and emotional weight.\nIt has also received praise from many astronomers for its scientific accuracy and portrayal of theoretical astrophysics. Since its premiere, Interstellar gained a cult following,[5] and now is regarded by many sci-fi experts as one of the best science-fiction films of all time.\nInterstellar was nominated for five awards at the 87th Academy Awards, winning Best Visual Effects, and received numerous other accolades']
```

### 句子拆分
```python
# 对句子进行拆分
texts = text.split('.')

# 清楚空格和分行
texts = np.array([t.strip(' \n') for t in texts])
```

### 为每个句子前添加整段文字的主题内容
为了增加每个分句对模型的可读性，为每个句子前添加整段文字的主题内容

```python
title = 'Interstellar (film)'

texts = np.array([f"{title} {t}" for t in texts])
```

```python
print(texts)
```

```python
['Interstellar (film) Interstellar is a 2014 epic science fiction film co-written, directed, and produced by Christopher Nolan'
 'Interstellar (film) It stars Matthew McConaughey, Anne Hathaway, Jessica Chastain, Bill Irwin, Ellen Burstyn, Matt Damon, and Michael Caine'
 'Interstellar (film) Set in a dystopian future where humanity is struggling to survive, the film follows a group of astronauts who travel through a wormhole near Saturn in search of a new home for mankind'
 'Interstellar (film) Brothers Christopher and Jonathan Nolan wrote the screenplay, which had its origins in a script Jonathan developed in 2007'
 'Interstellar (film) Caltech theoretical physicist and 2017 Nobel laureate in Physics[4] Kip Thorne was an executive producer, acted as a scientific consultant, and wrote a tie-in book, The Science of Interstellar'
 'Interstellar (film) Cinematographer Hoyte van Hoytema shot it on 35 mm movie film in the Panavision anamorphic format and IMAX 70 mm'
 'Interstellar (film) Principal photography began in late 2013 and took place in Alberta, Iceland, and Los Angeles'
 'Interstellar (film) Interstellar uses extensive practical and miniature effects and the company Double Negative created additional digital effects'
 'Interstellar (film) Interstellar premiered on October 26, 2014, in Los Angeles'
 'Interstellar (film) In the United States, it was first released on film stock, expanding to venues using digital projectors'
 'Interstellar (film) The film had a worldwide gross over $677 million (and $773 million with subsequent re-releases), making it the tenth-highest grossing film of 2014'
 'Interstellar (film) It received acclaim for its performances, direction, screenplay, musical score, visual effects, ambition, themes, and emotional weight'
 'Interstellar (film) It has also received praise from many astronomers for its scientific accuracy and portrayal of theoretical astrophysics'
 'Interstellar (film) Since its premiere, Interstellar gained a cult following,[5] and now is regarded by many sci-fi experts as one of the best science-fiction films of all time'
 'Interstellar (film) Interstellar was nominated for five awards at the 87th Academy Awards, winning Best Visual Effects, and received numerous other accolades']
```

### 生成 Embeddings
将处理后的 texts 传入 cohere 的 embed 函数中生成 Embeddings

```python
response = co.embed(
    texts=texts.tolist()
).embeddings #用 text 生成 Embeddings
```

```python
default model on embed will be deprecated in the future, please specify a model in the request.
```

查看一下生成的 embeds 的形状，可以看到不同长度的句子都被映射成相同维度的 Embeddings

```python
embeds = np.array(response)
print(embeds.shape)
```

```python
(15, 4096)
```

### 创建搜索索引
```python
search_index = AnnoyIndex(embeds.shape[1], 'angular')# 这里使用 Annoy 算法构建与 embeds 相同个数的索引。

# 将所有的向量添加到搜索索引中
for i in range(len(embeds)):
    search_index.add_item(i, embeds[i])

search_index.build(10) # 10 棵树
search_index.save('test.ann')
```

<details class="lake-collapse"><summary id="u10cd0dc4"><span class="ne-text" style="color: rgba(0, 0, 0, 0.87)">output：</span></summary><pre data-language="json" id="nPQp9" class="ne-codeblock language-json"><code>True</code></pre></details>


```python
pd.set_option('display.max_colwidth', None)

def search(query):

  # 先把 query 映射到 Embeddings 空间
  query_embed = co.embed(texts=[query]).embeddings

  # 找到映射空间内与 query_embed 最邻近的 text
  similar_item_ids = search_index.get_nns_by_vector(query_embed[0],
                                                    3,
                                                  include_distances=True)
  # 生成结果
  results = pd.DataFrame(data={'texts': texts[similar_item_ids[0]],
                              'distance': similar_item_ids[1]})

  print(texts[similar_item_ids[0]])

  return results
```



```python
query = "How much did the film make?"
search(query)
```

```python
default model on embed will be deprecated in the future, please specify a model in the request.
```

```python
['Interstellar (film) The film had a worldwide gross over $677 million (and $773 million with subsequent re-releases), making it the tenth-highest grossing film of 2014'
 'Interstellar (film) Interstellar premiered on October 26, 2014, in Los Angeles'
 'Interstellar (film) In the United States, it was first released on film stock, expanding to venues using digital projectors']
```

|  | **texts** | **distance** |
| :--- | :--- | :--- |
| **0** | Interstellar (film) The film had a worldwide gross over 677million(and773 million with subsequent re-releases), making it the tenth-highest grossing film of 2014 | 1.018955 |
| **1** | Interstellar (film) Interstellar premiered on October 26, 2014, in Los Angeles | 1.145063 |
| **2** | Interstellar (film) In the United States, it was first released on film stock, expanding to venues using digital projectors | 1.167268 |


