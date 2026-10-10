智能对话，有时候也叫智能客服、对话机器人、聊天机器人等等。总之就是和用户通过聊天窗口进行交互的一种技术。传统的对话机器人（下面都这么叫了）一般包括三个大的模块：

+ `NLU`：负责对用户输入进行理解。我们在本章一开始已经提到了，主要就是意图分类+实体识别这两种技术。实际中还可能有实体关系抽取、情感识别等组件。
+ `DM`：Dialogue Management，对话管理。就是在拿到NLU的结果后，如何确定机器人的回复，也就是对话方向的控制。
+ `NLG`：Natural Language Generation，自然语言生成。就是生成最终要回复给用户的输出。

##  分类
<br/>color2
对话机器人一般包括三种，不同类型的技术方案侧重有所不同。常见的类型如下：

+ 任务型机器人
+ 问答型机器人
+ 闲聊型机器人

<br/>

### 任务型机器人
任务型机器人主要用来完成特定的任务，比如订机票、订餐等，这一类机器人最关键的是要获取完成任务所需的各种信息（专业术语叫：槽位）。整个对话过程其实可以看作是一个填槽过程，通过与用户不断对话获取到需要的槽位信息。比如订餐这个任务，就餐人数、就餐时间、联系人电话等就是基本信息，机器人就要想办法获取到这些信息。这里NLU就是重头，DM一般使用两种方法：模型控制或流程图控制。前者通过模型自动学习来实现流转，后者则根据意图类型进行流转控制。

### 问答型机器人
问答型机器人主要用来回复用户问题，和上一章介绍的QA有点类似，平时我们常见的客服机器人往往是这种类型。它们更重要的是Question的匹配，DM相对弱一些。

### 问答型机器人
闲聊机器人就是和客户瞎扯淡的机器人，没啥实际作用。

## 主动发起/被动接受
<br/>color2
以上是大致的分类，但真实场景中的对话机器人往往是多种功能的结合体。更加适合从主动发起/被动接受这个角度来划分。

+ 主动发起对话的机器人
+ 被动接受对话的机器人

<br/>

### 主动发起对话的机器人
前者一般是以外呼的方式进行，营销、催款、通知等都是常见的场景。这种对话机器人一般不闲聊，电话费不允许。它们基本都是带着特定任务或目的走流程，流程走完就挂断结束。与用户的互动更多是以QA的形式完成，因为主动权在机器人手里，所以流程一遍都是固定控制的，甚至QA的数量、回答次数也会控制。

### 被动接受对话的机器人
后者一般是以网页或客户端的形式存在，绝大部分时候都是用户找上门来了，比如大部分公司首页都有个「智能客服」，就是类似功能。它们以QA为主，辅以闲聊。稍微复杂点的是上面提到的任务型机器人，需要不断收集槽位信息。

##  智能对话机器人会有什么新变化吗
ChatGPT时代，智能对话机器人会有什么新变化吗？接下来，我们探讨一下这方面内容。

首先，可以肯定的是ChatGPT极大的扩展了对话机器人的边界，在此之前其实有不少端到端的方案，感兴趣的读者可以略读一下【相关文献13】。之前的方案现在看来是有点复杂、繁琐，效果还不好。所以之前除了闲聊，很少有真正使用端到端方案的对话机器人。不过ChatGPT的强大In-Context能力不仅让使用更加简单（我们只需把历史对话分角色放进去就好了），而且效果也更好，除了闲聊，问答型机器人它也可以很擅长，交互更加humanable。

我们具体来展开说说它可以做的，我们尽量聚焦当下能做到的，不做过多未来畅想。

+ 作为问答类产品，比如知识问答、情感咨询、心理咨询等等，完全称得上诸事不决ChatGPT。举几个简单例子，比如问它编程概念（如闭包的作用），问它如何追一个心仪的女孩子，问它怎么避免焦虑等等。它的大部分回答绝对能让你眼前一亮。这可是之前只能闲聊的机器人完全够不着的高度。
+ 作为智能客服，通过与企业知识库结合（In-Context方式和微调方式①）完全可以胜任客服工作，而且相比之前的QA类的客服，它回答更加个性化，效果也更好（如果不记得了，回忆一下前面“文档问答”的内容）。
+ 作为智能营销机器人，智能客服更加偏向被动、为用户答疑解惑的方向，营销机器人则更加主动一些，它会根据已存储的用户信息，主动向用户推荐相关产品，根据预设的目标（可以理解为槽位，长期要收集的信息）向用户发起对话。它还可以同时负责维护客户关系。
+ 作为NPC（Non-Player Character）、陪聊机器人等休闲娱乐类产品。
+ 作为教育、培训的导师，可以进行一对一教学，尤其适合语言、编程类学习。

这些都是它确定可以做的，为什么能做？归根结底还是其大规模参数所学到的知识和具备的理解力。尤其是后者，应该是决定性的（只有知识就是Google搜索引擎）。

当然，并不是什么都要ChatGPT，我们千万要避免「手里有锤子，到处找钉子」的思维方式。某位哲人说过，一项新技术的出现，短期内总是被高估，长期内总是被低估。ChatGPT绝对是划时代的，但也不意味着你什么都要ChatGPT一下。比如，某些分类和实体抽取任务，之前的方法已经能达到非常好的效果，这时候就完全不需要替换。我们知道很多实际任务它并不会随着技术发展有太多变化，比如分类任务，难道你出来个新技术，分类任务就不是分类任务了吗。技术的更新会让我们的效率得到提升，也就是说做同样的任务更加简单和高效了，可以做更难的任务了，但不等于任务也会发生变化。所以，一定要理清楚这里面的关键。

不过如果你是新开发，或者不了解这方面的专业知识，那就另当别论了，使用LLM的API反而可能是更好的策略。但即便如此，实际上线前还是应该考虑清楚各种细节，比如服务不可用怎么办，并发大概多少，时延要求多少，用户规模大概多少等等。我们技术方案的选型是和公司或自己的需求息息相关的，没有绝对好的方案，只有当下是否适合的方案。同时，要尽可能多考虑几步，但也不用太多（时刻谨记：「过度优化是原罪」），比如你用户只有不到1万，上来就搞个分布式的设计方案就有点坑了。但这并不妨碍你在代码和架构设计时考虑扩展性，比如数据库，我们可能用SQLite，但你代码里可不能直接和它耦合死，可以使用能同时支持其他数据库，甚至分布式数据库的ORM工具。这样虽然写起来稍微麻烦了一点点（真的是一点点），但你的代码更加清晰，而且和可能会变化的东西解耦了。这样如果日后规模上去了，数据库可以随便换，代码基本不用动。

## ChatGPT的一些局限
最后，我们也应该了解ChatGPT的一些局限，除了它本身的局限（这块内容可以参考后面专门讲缺陷的章节），在工程上始终应该关注下面几个话题：

+ 响应时间和稳定性
+ 并发和横向可扩展性
+ 可维护性和迭代
+ 成本

只有当这些都能满足你的期望时，才应该选择。始终记住，人才是关键，不要被任何工具绑架。

<br/>color2
注：关于In-Context方式和微调方式的通俗解释

In-Context主要是利用ChatGPT的理解能力，把它当做超级大脑，我们把相关上下文给它，让它根据上下文回答问题，就类似前面的[NLU应用 文档问答](https://www.yuque.com/qiaokate/su87gb/zueypw7kgd30hmf0)。对于不确定的问题，还可以设计兜底话术。  
	微调方式则是直接将自定义数据喂给ChatGPT的微调接口（现在没有开放，但理论上可行），让它学习这些自定义内容，之后直接问就好了，就像我们现在直接问它「中国的首都是哪里」，它可以正确回答一样。

<br/>

## 实现一个对话机器人
下面，我们一起使用ChatGPT来实现一个对话机器人。设计阶段首先至少需要考虑以下一些因素（这并不包括上面提到的那些）：

+ 使用目的
+ 如何使用
+ 消息查询、存储
+ 消息解析
+ 实时干预
+ 更新策略

首先，需要明确使用目的是什么，如上所言，不同的用途要考虑的因素也不一样。简单（但很实际）起见，我们以一个「订餐机器人」为例，简单的开场白后获取用户联系方式、订餐人数、用餐时间三个信息。

使用也比较简单，主要利用ChatGPT的多轮对话能力即可，这里的重点是控制上下文。不过由于任务简单，我们不用对历史记录做召回再进行对话，直接在每一轮时把已经获取的信息告诉它，同时让它继续获取其他信息，直到所有信息获取完毕为止。另外，我们可以限制一下输出Token的数量（输出文本的长度）。

对于用户的消息（以及机器人的回复），实际中往往需要存储起来，用来做每一轮回复的历史消息召回。而且这个日后还可能有其他用途，比如使用对话记录对用户进行画像，或者用来当做训练数据等等。存储可以直接放到数据库，或传到类似ElasticSearch这样的内部搜索引擎中。

消息的解析可以实时进行（并不一定要用ChatGPT）或离线进行，本案例我们需要实时在线解析。这个过程我们可以让ChatGPT在生成回复时顺便做掉。

实时干预是应该要关注的，或者需要设计这样的模块。一方面是回复内容有时候即便做了限制，依然有可能被某些问法问到不太合适的答复；另一方面也不能排除部分恶意用户对机器人进行攻击，因此最好有干预机制的设计。这里，我们设计一个简单策略：检测用户是否提问敏感类问题，如果发现此类问题直接返回设定好的文本，不需要调用ChatGPT进行对话回复。

更新策略主要是对企业知识库的更新，这里由于我们使用的是In-Context能力，所以并不需要调整ChatGPT，可能需要调整Embedding接口（目前openai不支持）。此案例暂不涉及。

综上，我们需要先对用户输入进行敏感性检查，没问题后开始对话。同时应存储用户消息，并在每轮对话时将用户历史消息传递给接口。

```python
from openai import OpenAI
client = OpenAI(api_key="YOUR OPENAI KEY")

from zhipuai import ZhipuAI
zhipu_client = ZhipuAI(api_key="YOUR ZHIPU KEY")
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>/usr/local/lib/python3.8/site-packages/requests/__init__.py:89: RequestsDependencyWarning: urllib3 (1.26.15) or chardet (3.0.4) doesn't match a supported version!
  warnings.warn(&quot;urllib3 ({}) or chardet ({}) doesn't match a supported &quot;</code></pre></details>
### 敏感性检查
先看一下敏感性检查，这个接口比较多，openai提供了一个相关的接口，国内几大厂商也有相关API。这个本身是和对话无关的。我们以openai接口为例。

```python
def check_risk(inp: str):
    # 这个接口的参数`input`支持Batch（传入List），会返回多个结果
    response = client.moderations.create(input=inp)
    return data["results"][0]["flagged"]
```

```python
check_risk("good")
```

<details class="lake-collapse"><summary id="uf2154ff4"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="GtXNP" class="ne-codeblock language-json"><code>False</code></pre></details>
### 构造接口的输入
接下来我们考虑如何构造接口的输入，这里有两个事情要做：

1. 查询历史对话记录作为上下文，简单起见我们可以只考虑上一轮。
2. 计算输入的Token数，根据模型能接受最大Token长度和想输出的最大长度，反推上下文的最大长度，并对历史对话进行处理（如截断）。

```python
from dataclasses import dataclass, asdict
from typing import List, Dict
from datetime import datetime
import uuid
import json
import re
```

```python
@dataclass
class User:
    
    user_id: str
    user_name: str
```

```python
@dataclass
class ChatSession:
    
    user_id: str
    session_id: str
    cellphone: str
    people_number: int
    meal_time: str
    chat_at: datetime


@dataclass
class ChatRecord:
    
    user_id: str
    session_id: str
    user_input: str
    bot_output: str
    chat_at: datetime
```

上面我们首先设计了两个简单的数据结构，一个是聊天信息，一个是聊天记录，前者记录聊天基本信息，后者记录聊天记录。其中，session_id主要用来区分每一次对话，当用户点击产品页面的「开始对话」之类的按钮后，就生成一个session_id；在下次对话时再生成一个新的。

### 处理核心对话逻辑
接下来，我们处理核心对话逻辑，这一块主要是利用ChatGPT的能力，明确要求，把每一轮对话都喂给它。给出响应。

```python
def ask(msg):
    response = openai_client.chat.completions.create(
        model="gpt-3.5-turbo", 
        temperature=0.2,
        max_tokens=100,
        top_p=1,
        frequency_penalty=0,
        presence_penalty=0,
        messages=msg
    )
    ans = response.choices[0].message.content
    return ans


def ask_zhipu(msg):
    response = zhipu_client.chat.completions.create(
        model="glm-3-turbo", 
        temperature=0.1,
        max_tokens=100,
        top_p=0.9,
        messages=msg,
    )

    ans = response.choices[0].message.content
    return ans
```

```python
!pip install sqlalchemy
```

```python
from sqlalchemy import insert

class Chatbot:
    
    def __init__(self, api: str="openai"):
        self.system_inp = """现在你是一个订餐机器人（角色是assistant），你的目的是向用户获取手机号码、用餐人数量和用餐时间三个信息。你可以自由回复用户消息，但牢记你的目的。每一轮你需要输出给用户的回复，以及获取到的信息，信息应该以JSON方式存储，包括三个key：cellphone表示手机号码，people_number表示用餐人数，meal_time表示用餐时间储。

回复格式：
给用户的回复：{回复给用户的话}
获取到的信息：{"cellphone": null, "people_number": null, "meal_time": null}
"""
        self.max_round = 10
        self.slot_labels = ["meal_time", "people_number", "cellphone"]
        self.reg_msg = re.compile(r"\n+")
        self.api = api
        if self.api == "openai":
            self.ask_func = ask
        elif self.api == "zhipu":
            self.ask_func = ask_zhipu
        else:
            raise ValueError(f"Invalid api: {api}")


    def check_over(self, slot_dict: dict):
        for label in self.slot_labels:
            if slot_dict.get(label) is None:
                return False
        return True
    
    def send_msg(self, msg: str):
        print(f"机器人：{msg}")
    
    def chat(self, user_id: str):
        sess_id = uuid.uuid4().hex
        chat_at = datetime.now()
        msg = [
            {"role": "user", "content": self.system_inp},
        ]
        n_round = 0
        
        history = []
        slot = {"cellphone": None, "people_number": None, "meal_time": None}
        while True:
            if n_round > self.max_round:
                bot_msg = "非常感谢您对我们的支持，再见。"
                self.send_msg(bot_msg)
                break
            
            try:
                # 智谱AI改成ask_zhipu
                bot_inp = self.ask_func(msg)
            except Exception as e:
                print(f"Error: {e}")
                bot_msg = "机器人出错，稍后将由人工与您联系，谢谢。"
                self.send_msg(bot_msg)
                break
            
            tmp = self.reg_msg.split(bot_inp)
            bot_msg = tmp[0].strip("给用户的回复：")
            self.send_msg(bot_msg)
            if len(tmp) > 1:
                slot_str = tmp[1].strip("获取到的信息：")
                slot = json.loads(slot_str)
                print(f"\tslot: {slot}")
            n_round += 1
            
            if self.check_over(slot):
                break

            user_inp = input()
            
            msg += [
                {"role": "assistant", "content": bot_inp},
                {"role": "user", "content": user_inp},
            ]
            
            record = ChatRecord(user_id, sess_id, bot_inp, user_inp, datetime.now())
            history.append(record)
            
            if check_risk(user_inp):
                break
        
        chat_sess = ChatSession(user_id, sess_id, **slot, chat_at=chat_at)
        self.store(history, chat_sess)
    
    
    def store(self, history: List[ChatRecord], chat: ChatSession):
        with SessionLocal.begin() as sess:
            q = insert(
                chat_record_table
            ).values(
                [asdict(v) for v in history]
            )
            sess.execute(q)
        with SessionLocal.begin() as sess:
            q = insert(
                chat_session_table
            ).values(
                [asdict(chat)]
            )
            sess.execute(q)
```

### 把两张表建好
接下来，我们把两张表建好（User表这里就不建了）。**注意：建表只要一次。**

```python
from sqlalchemy import Table, Column, Integer, String, DateTime, Text, MetaData, SmallInteger
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import os


db_file = "chatbot.db"

if os.path.exists(db_file):
    os.remove(db_file)

engine = create_engine(f"sqlite:///{db_file}")
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

metadata_obj = MetaData()

chat_record_table = Table(
    "chat_record_table",
    metadata_obj,
    Column("id", Integer, primary_key=True),
    Column("user_id", String(64), index=True),
    Column("session_id", String(64), index=True),
    Column("user_input", Text),
    Column("bot_output", Text),
    Column("chat_at", DateTime),
)

chat_session_table = Table(
    "chat_session_table",
    metadata_obj,
    Column("id", Integer, primary_key=True),
    Column("user_id", String(64), index=True),
    Column("session_id", String(64), index=True),
    Column("cellphone", String(16)),
    Column("people_number", SmallInteger),
    Column("meal_time", String(32)),
    Column("chat_at", DateTime),
)

metadata_obj.create_all(engine, checkfirst=True)
```

```python
!ls -la ./chatbot.db
```

```python
-rw-r--r--  1 Yam  staff  28672 May 25 21:18 ./chatbot.db
```

### 进行简单的尝试
现在我们进行简单的尝试：

```python
!pip install pnlp
```

```python
import pnlp
nick = "长琴"
user = User(pnlp.generate_uuid(nick), nick)
chatbot = Chatbot("zhipu")
chatbot.chat(user.user_id)
```

<details class="lake-collapse"><summary id="u847f9ff9"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="QbIRD" class="ne-codeblock language-json"><code>机器人：您好！非常感谢您选择我们的订餐服务。为了更好地为您服务，请问您能提供一下您的手机号码吗？这样我们才能及时通知您订单的状态。
	slot: {'cellphone': None, 'people_number': None, 'meal_time': None}
是13712345678
机器人：好的，您的手机号码已经记录为13712345678。接下来，请问您需要预订几人份的餐品呢？
	slot: {'cellphone': '13712345678', 'people_number': None, 'meal_time': None}
我们一起5个人
机器人：明白了，您需要预订的餐品份数为5人。请问您预计什么时候用餐呢？
	slot: {'cellphone': '13712345678', 'people_number': 5, 'meal_time': None}
今天晚上8点
机器人：好的，您的用餐时间为今天晚上8点。我们已经为您记录了预订信息，正在为您准备餐品。如有需要，我们会通过手机号码13712345678与您联系。祝您用餐愉快！
	slot: {'cellphone': '13712345678', 'people_number': 5, 'meal_time': '今天晚上8点'}</code></pre></details>


```python
import pnlp
nick = "长琴"
user = User(pnlp.generate_uuid(nick), nick)
chatbot = Chatbot("openai")
chatbot.chat(user.user_id)
```

<details class="lake-collapse"><summary id="u31453434"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="L0xU6" class="ne-codeblock language-json"><code>机器人：请问您的手机号码是多少呢？
	slot: {'cellphone': None, 'people_number': None, 'meal_time': None}
我的手机是13788889999
机器人：好的，您的手机号码是13788889999，请问用餐人数是几位呢？
	slot: {'cellphone': '13788889999', 'people_number': None, 'meal_time': None}
我们一共五个人
机器人：好的，您们一共五个人，最后，请问您们的用餐时间是什么时候呢？
	slot: {'cellphone': '13788889999', 'people_number': 5, 'meal_time': None}
稍等我问一下啊
机器人：好的，没问题，我等您的消息。
好了，明天下午7点，谢谢
机器人：好的，您们的用餐时间是明天下午7点，我们已经为您记录好了，请问还有其他需要帮助的吗？
	slot: {'cellphone': '13788889999', 'people_number': 5, 'meal_time': '明天下午7点'}</code></pre></details>


查表看看刚刚的记录：

```python
import sqlite3
```

```python
def query_table(table: str):
    con = sqlite3.connect("chatbot.db")
    cur = con.cursor()
    q = cur.execute(f"SELECT * FROM {table}")
    return q.fetchall()
```

```python
query_table("chat_session_table")
```

<details class="lake-collapse"><summary id="uca8f7f0a"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="ANHQZ" class="ne-codeblock language-json"><code>[(1,
  'dc3be3b3516555d3b0b6a77a1d9c7e82',
  '05a88a8e3db8490eacf14b8bb9800fcc',
  '13788889999',
  5,
  '明天下午7点',
  '2023-04-08 00:00:34.618232')]</code></pre></details>
上面我们实现了一个非常简易的任务机器人，虽然简易，但我们其实很容易就能发现，NLU、DM和NLG三个模块已经完全不需要了。唯一的不足可能是接口反应有点慢，从对话来看其实并没有太多问题。

## 强调
另外，需要再次对几个问题进行强调，以便大家可以更好地构建应用。

第一点，当要支持的对话轮次非常多时（比如培训、面试这样的场景），则需要**实时将每一轮的对话索引起来**，每一轮先召回所有历史对话中相关的topN轮作为上下文（正如我们在文档问答中那样）。然后让ChatGPT根据这些上下文对用户进行回复。这样理论上我们是可以支持无限轮的。召回的过程其实就是一个回忆的过程，这里可以优化的点或者说想象的空间很大。

第二点，在传递message参数给ChatGPT时，由于有长度限制，有时候上下文中遇到特别长回复那种轮次，可能会导致只能传几轮（甚至一轮就耗光长度了）。根据ChatGPT自己的说法：当历史记录非常长时，我们确实可能只能利用其中的一小部分来生成回答。为了应对这种情况，通常会使用一些技术来选择最相关的历史记录，以便在生成回答时使用。例如，可能会使用一些**关键词提取技术**，识别出历史记录中最相关的信息，并将其与当前的输入一起使用。

此外，还可能会使用一些**摘要技术来对历史记录进行压缩和精简**，以便在生成回答时只使用最重要的信息。另外，还可以使用一些记忆机制，例如注意力机制，以便在历史记录中选择最相关的信息。虽然这些技术可以帮助模型在历史记录很长时选择最相关的信息，但在某些情况下，历史记录仍然可能过于复杂，导致模型难以正确理解和处理。在这种情况下，可能需要使用其他技术来限制历史记录的长度或提供其他方面的辅助信息，以便模型可以更好地理解和回答用户的问题。

另外，根据ChatGPT的说法，在生成回复时，它也会使用一些技术来限制输出长度，例如**截断输出或者使用一些策略来生成更加简洁的回答**。当然，用户也可以使用特定的输入限制或规则来帮助缩短回答。总之，尽可能地在输出长度和回答质量之间进行平衡。

第三点，充分考虑安全性，**根据实际情况合理设计架构**（但不要过度设计）。

最后，值得一提的是，上面只是利用了ChatGPT的一丢丢功能，大家完全可以结合自己的业务，或者大开脑洞，开发更多有用、有趣的产品和应用。

