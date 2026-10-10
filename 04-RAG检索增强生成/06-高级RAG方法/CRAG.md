地址：[https://arxiv.org/pdf/2406.04744](https://arxiv.org/pdf/2406.04744)

[2406.04744v2.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1737701521127-ae1f2b4b-bbe8-412d-af92-c1135596afd8.pdf)

参考：

[前沿重器[43] | 谷歌中科院新文：CRAG-可矫正的检索增强生成](https://mp.weixin.qq.com/s/91F_sxXNL1_h9Wg_aZDP2w)

<font style="color:rgb(0, 0, 0);">前面有聊过self-RAG通过大模型决策，提升了RAG系统的效果，本期给大家介绍一个更为简便、优秀的插件式决策模块CRAG。论文和有关资料在这里：</font>

> + <font style="color:rgb(1, 1, 1);">论文：Corrective Retrieval Augmented Generation，https://arxiv.org/abs/2401.15884。首次发表是1月29，第二版2月16，挺新的。</font>
> + <font style="color:rgb(1, 1, 1);">讲解：https://zhuanlan.zhihu.com/p/681186898</font>
> + <font style="color:rgb(1, 1, 1);">github：https://github.com/HuskyInSalt/CRAG，不过只找到了推理和eval的代码，train的没开源。</font>
>
> <font style="color:rgb(0, 0, 0);">为避免赘述，放一批前情提要，RAG这块我已经写了挺多文章了，从基础到迭代优化再到进阶，都有，有些基础知识可以通过这里的文章查到。</font>
>
> + [<font style="color:rgb(0, 0, 0);">前沿重器[40] | 高级RAG技术——博客阅读</font>](http://mp.weixin.qq.com/s?__biz=MzIzMzYwNzY2NQ==&mid=2247489357&idx=1&sn=4193a9e92c7749e4ecdacbce7cf6b9db&chksm=e8824fd3dff5c6c5b6c04be81624bf075c96fef23fc40f8cd165a600ecaa02d434a5fd21da41&scene=21#wechat_redirect)
> + [<font style="color:rgb(0, 0, 0);">前沿重器[41] | 综述-面向大模型的检索增强生成（RAG）</font>](http://mp.weixin.qq.com/s?__biz=MzIzMzYwNzY2NQ==&mid=2247489372&idx=1&sn=8fc154edac26943a447d596a4e697ede&chksm=e8824fc2dff5c6d44d8e7a1be4cc1b472e8cdd8e8484bf02d00f78ab708e6340f6f90243cb0c&scene=21#wechat_redirect)
> + [<font style="color:rgb(0, 0, 0);">前沿重器[42] | self-RAG-大模型决策的典型案例探究</font>](http://mp.weixin.qq.com/s?__biz=MzIzMzYwNzY2NQ==&mid=2247489393&idx=1&sn=1917808697d84cfd17e628d58b3bfd89&chksm=e8824fefdff5c6f93a322772245c5d4c5c1b22f0f05a45f60d096a99b07bfdb12f54042d5aaf&scene=21#wechat_redirect)<font style="color:rgb(0, 0, 0);">（尤其这篇，今天我会经常提到）</font>
> + [<font style="color:rgb(0, 0, 0);">心法利器[104] | 基础RAG-向量检索模块（含代码）</font>](http://mp.weixin.qq.com/s?__biz=MzIzMzYwNzY2NQ==&mid=2247489330&idx=1&sn=23cf88a96441f93f78461ff92be6a03c&chksm=e8824facdff5c6ba95ea2e5822e1ac40037c0312b774b028fb545117911dfbfe38e1deef11f2&scene=21#wechat_redirect)
> + [<font style="color:rgb(0, 0, 0);">心法利器[105] | 基础RAG-大模型和中控模块代码（含代码）</font>](http://mp.weixin.qq.com/s?__biz=MzIzMzYwNzY2NQ==&mid=2247489343&idx=1&sn=44b5f90ec117a5f4f70881247483f008&chksm=e8824fa1dff5c6b73c7c41ae36ae110e22d2d5f87b7b34796f0fd68e472bd8af1919bdae58c4&scene=21#wechat_redirect)
> + [<font style="color:rgb(0, 0, 0);">心法利器[106]  基础RAG-调优方案</font>](http://mp.weixin.qq.com/s?__biz=MzIzMzYwNzY2NQ==&mid=2247489349&idx=1&sn=e68ad131621a0cf94d5532f5da2d0199&chksm=e8824fdbdff5c6cdcb8e683b67b07d996af3cf51107099529e4c12b859e8192be7d05d07f479&scene=21#wechat_redirect)<font style="color:rgb(0, 0, 0);">  
</font>
>

<font style="color:rgb(0, 0, 0);">下面来听我讨论一下CRAG这篇论文吧。</font>

<font style="color:rgb(0, 0, 0);">目录：</font>

+ <font style="color:rgb(1, 1, 1);">核心方案</font>
+ <font style="color:rgb(1, 1, 1);">实验</font>
+ <font style="color:rgb(1, 1, 1);">补充讨论</font>

## **<font style="color:rgb(0, 0, 0);"> 核心方案</font>**
<font style="color:rgb(0, 0, 0);">整个方案的架构，可以用论文里的这张图来总结：</font>

![](https://cdn.nlark.com/yuque/0/2025/webp/2639475/1737704596312-b9be2780-f238-4858-9489-f61992f59298.webp)

<font style="color:rgb(136, 136, 136);">image-20240224165934617</font>

<font style="color:rgb(0, 0, 0);">说白了，CRAG就是在原本的RAG基础上，增加了一个做简单的知识正确性决策和纠正的中间模块。该模块使用一个决策模型retrieval evaluator来判断检索的材料和query之间的关系，如果相关，则可以直接用于进行答案生成，如果不相关，则去网上进行检索，使用网络搜索的结果，中间还有个模糊状态，会同时使用自己的检索结果和网络的检索结果。</font>

<font style="color:rgb(0, 0, 0);">而在这里，拆解一下，相比RAG，这里多做了这几件事：</font>

+ <font style="color:rgb(1, 1, 1);">判别模块建模。</font>
+ <font style="color:rgb(1, 1, 1);">判断后处理。</font>
+ <font style="color:rgb(1, 1, 1);">具体实验和其他执行过程中的细节。</font>

### <font style="color:rgb(0, 0, 0);">判别模块建模</font>
<font style="color:rgb(0, 0, 0);">这个判别模块，是用于评估query和检索得到的doc之间的关系的，最终是会分为3档，分别是correct、ambiguous和incorrect，即正确、模糊和不正确，这个档位是根据模型预测得到的，此处的模型使用的是T5-large（唉，T5-large已经被说是lightwight了）。</font>

<font style="color:rgb(0, 0, 0);">不过比较尴尬的是，无论是论文还是github都没找到具体的训练细节，如数据、策略等，从推理的代码来看，就是直接用的“T5ForSequenceClassification”进行推理的，这个模型的输出就是一个概率了。另外一个线索是在附录A里有找到用于进行相关性判断的prompt，这个按照文章的说法是用来做5.5节实验的，但好像也可以用来做这个模型训练的预标注。（如果有朋友找到了，欢迎补充一下）</font>

<font style="color:rgb(0, 0, 0);">这里可以大概推断，某种程度上说，就是一个打分的分类模型了。在搜索领域，有点像排序的打分模型，用来判断query和doc的相关性，这让我们在实际应用中，可以有更多操作空间，甚至可以结合实际加入更多额外特征，例如字面相似度、用户画像、意图等。</font>

### <font style="color:rgb(0, 0, 0);">判别阈值的设置</font>
<font style="color:rgb(0, 0, 0);">模型的预测用的是分类模型，输出的就是一个打分，而前文提及判别模块会分3档，说明此处需要2个阈值才能分，在附录里面作者提及在不同的实验数据下，阈值的设置是存在些许不同的：</font>

> <font style="color:rgb(0, 0, 0);">The two confidence thresholds for triggering one of the three actions were set empirically. Specifically, they were set as (0.59, -0.99) in PopQA, (0.5, -0.91) in PubQA and ArcChallenge, as well as (0.95, -0.91) in Biography.</font>
>

<font style="color:rgb(0, 0, 0);">这里有两个信息：</font>

+ <font style="color:rgb(1, 1, 1);">不同数据下，阈值的变化波动是比较大的，我们需要根据实际问题和需求来调整。</font>
+ <font style="color:rgb(1, 1, 1);">有关ambiguous和incorrect之间的阈值，似乎会卡的比较死，基本都在-0.9左右，说明大部分知识多少还是有些关系的，完全放弃可能不是一个很好的选择，这个可以放在实际情况里参考。</font>

### <font style="color:rgb(0, 0, 0);">判断后处理</font>
<font style="color:rgb(0, 0, 0);">判断的后处理还是比较简单的。首先还是先定义这个模块的任务，根据判别模块得到的结果“correct、ambiguous和incorrect”，执行额外的操作，为最后的生成准备原料。此处分为3种情况进行处理。</font>

<font style="color:rgb(0, 0, 0);">首先是correct的情况，query和doc之间的是正确的，说明doc中包含了query所需要的正确内容，所以不需要进行处理，用query和doc直接交给大模型做后续生成即可。</font>

<font style="color:rgb(0, 0, 0);">然后先说incorrect，这说明query和doc之间的是不相关的，那我们就不能把doc交给大模型来让大模型回答query问题，毕竟根本不靠谱。但问题总要回答的（此处可以说是一个假设，在文章后面的讨论我会专门说一下这个问题），所以就开启了网络检索模式，该模式是会拿着用户query去网上进行搜索，文章用的搜索引擎是谷歌搜索。</font>

<font style="color:rgb(0, 0, 0);">比较复杂的是ambiguous情况，说明模棱两可，换言之可能有比较接近的结果但是不足以回答，此时也需要通过网络检索进行补充。</font>

<font style="color:rgb(0, 0, 0);">这里的网络检索，文章中并非直接用用户query来搜，而是做了转化，这个转化的方法是构造了一个prompt，用few-shot的方式交给GPT-3.5 Turbo得到的，而最终的网络搜索用的是谷歌搜索引擎API。</font>

## <font style="color:rgb(0, 0, 0);">实验</font>
<font style="color:rgb(0, 0, 0);">文中做的实验还是很多的，而且有挺多挺有价值的发现。</font>

### <font style="color:rgb(0, 0, 0);">方案有效性</font>
![](https://cdn.nlark.com/yuque/0/2025/webp/2639475/1737704752571-cd1cc335-1880-49b4-b347-31956f86a7f5.webp)

<font style="color:rgb(136, 136, 136);">方案有效性</font>

<font style="color:rgb(0, 0, 0);">首先就是在论文提出的方案有效性上，确实有不少的提升的，再者因为这个4个数据是跨领域的，所以也一定程度体现了跨领域的能力。</font>

<font style="color:rgb(0, 0, 0);">第二是其插件能力，最下面的两块是尝试将CRAG作为插件插入到RAG和self-RAG里面，结果发现相比没有插入都有一定的提升。</font>

### <font style="color:rgb(0, 0, 0);">判别模型效果的准确率</font>
<font style="color:rgb(0, 0, 0);">这个判别器本身的效果对整体方案肯定是有影响的，因此出于严谨，确实要评估好这个判别模型的准确率。这一节作者是放在消融实验里，但个人感觉因为这里缺少一个实验，就是这不同的准确率的判别方案对最终效果的影响，而单单只是分析判别器效果，所以放上面可能还比较连贯，所以放这里。</font>

<font style="color:rgb(0, 0, 0);">文章最终使用的是一个基于T5-large微调的模型，与之对比的是未经过模型微调的chatgpt，但是也考虑了few-shot和COT的方案，结果如下：</font>

![](https://cdn.nlark.com/yuque/0/2025/webp/2639475/1737704766002-7562a9d5-aa9d-4bb5-91f8-f1298a468585.webp)

<font style="color:rgb(136, 136, 136);">判别模型效果的准确率</font>

<font style="color:rgb(0, 0, 0);">结果可以发现，T5的效果要远比chatgpt要好很多很多，说明在特定任务下，小模型经过微调也能得到不错的效果，根本的贡献还是业务数据所蕴含的边界和场景信息。当然出于严谨，这里可以加入一下微调大模型后的效果，以控制好大模型和微调这两个变量的关系，可能可以说明更多问题。另外这里也一定程度说明了，小模型微调仍具有很强的效果，这个实验虽然简单，但是弥补了之前我在self-RAG里面提到的拓展点吧。我把上一篇文章提及这部分问题的内容摘录下来：</font>

**<font style="color:rgb(0, 0, 0);">评论模型的模型方案选择</font>**<font style="color:rgb(0, 0, 0);">。最开始其实我并不能理解为什么作者会使用大模型作为标签预测的基础模型，但在对问题的逐步了解，也能够明白，有关“IsREL标签”之类的标签，背后很大程度依赖“世界知识”，例如问题和query的匹配，这个问题抽象出来其实就是一个不对称的句子对分类问题，这个任务固然小模型也能做，但小模型一般只能学习到训练数据相关的“知识”，而并非推理的相关本身，且对没学到的，依旧是不会，因此使用大模型作为基础模型是合理的。但话锋一转，尽管我们有预期，小模型效果可能一般，但从实验角度，小模型差多少，是否就不可用，这里是一个值得探索的问题，另一个角度，用更大的模型，是否还有收益，我们不得而知。</font>

### <font style="color:rgb(0, 0, 0);">消融实验</font>
<font style="color:rgb(0, 0, 0);">首先的第一个实验是探索相关性的3级拆分的有效性，然而是否需要3级的拆分，只用两级拆分是否可以，下面的图做了实验，结果发现相比分成3级，都会有一定的下降，其中把correct删除的影响是最大的。我猜是因为，本身知识库具有更专业针对性的领域知识，在正确的情况下可靠性很高，然而网络检索的结果并不一定正确，这些信息的加入反而会影响最终结果的准确性。</font>

![](https://cdn.nlark.com/yuque/0/2025/webp/2639475/1737704775547-57f6d052-db71-4f16-a267-4f74e82e26ef.webp)

<font style="color:rgb(136, 136, 136);">类目切分消融实验</font>

<font style="color:rgb(0, 0, 0);">下一个实验是探索各个知识利用组件对最终效果的影响，这里包括知识细化（我理解就是切片）、检索query改写、和外部知识的使用，结果发现，直接3个都很大程度会影响最终结果。这里对两个方案，不同模块的重要性差异较大。</font>

![](https://cdn.nlark.com/yuque/0/2025/webp/2639475/1737704786230-660274bb-0df0-4e91-8634-4ee8b2a3b115.webp)

<font style="color:rgb(136, 136, 136);">知识组件消融实验</font>

<font style="color:rgb(0, 0, 0);">紧跟着的实验是验证检索效果对最终效果的影响。</font>

![](https://cdn.nlark.com/yuque/0/2025/webp/2639475/1737704802488-d120c9d0-3e03-4828-b5ed-8bbc3f09b7d2.webp)

<font style="color:rgb(136, 136, 136);">检索效果消融实验</font>

<font style="color:rgb(0, 0, 0);">首先可以发现，在准确率不断下降的情况下，RAG的效果都会有下降，而self-CRAG的下降会没那么明显。这里的原因除了本人提及的CRAG增加的判别模块之外，还有一个原因是增加了web-search，网络检索其实为整体结果都提供了一个兜底，所以下限是会有提升的，而我理解self-RAG是没有的，而且可能这方面原因占比还会不小。</font>

<font style="color:rgb(0, 0, 0);">第二点是，两者相比没有检索，都要好很多。这个可以作为后续支撑RAG推进项目的一个重要依据了，很多场景下有知识支持的场景，可能RAG是一个重要路径了。</font>

<font style="color:rgb(0, 0, 0);">第三点从横坐标看，多少有点震撼。检索准确率即使减到了10%，相比没检索的baseline效果，都要好不少。当然这个跟这个数据集有关了，来看看这个数据集的介绍：</font>

> <font style="color:rgb(0, 0, 0);">PopQA (Mallen et al., 2023) is a short-form generation task.  Generally, only one entity of factual knowledge is expected to be answered for each single question.  In our experiments, we exactly followed the setting in Self-RAG (Asai et al., 2023) which evaluated methods on a </font>**<font style="color:rgb(0, 0, 0);">long-tail subset</font>**<font style="color:rgb(0, 0, 0);"> consisting of 1,399 rare entity queries whose </font>**<font style="color:rgb(0, 0, 0);">monthly Wikipedia page views are less than 100</font>**<font style="color:rgb(0, 0, 0);">.  Accuracy was adopted as the evaluation metric.</font>
>

<font style="color:rgb(0, 0, 0);">说白了就是比较偏的知识点了，这个对于大模型而言确实不擅长，正好是RAG的擅长的领域了，知识都给了，不回答对真说不过去。所以这个结论有一定的局限，但因为其严谨性，在垂直领域仍有很大的参考性。</font>

## <font style="color:rgb(0, 0, 0);">补充讨论</font>
### <font style="color:rgb(0, 0, 0);">self-RAG和CRAG的对比</font>
![](https://cdn.nlark.com/yuque/0/2025/webp/2639475/1737704812755-5a8fd0e3-cd25-475a-9281-d0b94d650a0d.webp)

<font style="color:rgb(136, 136, 136);">self-RAG和CRAG的对比</font>

+ <font style="color:rgb(1, 1, 1);">从结构上，self-RAG是比CRAG要复杂不少的，多了很多判别器，而且判别器还用了很高成本的微调大模型。</font>
+ <font style="color:rgb(1, 1, 1);">但是效果上，CRAG在更多情况是比同情况的self-RAG要优秀的。</font>
+ <font style="color:rgb(1, 1, 1);">不过这个对比可能并不严谨就是了，这里挺多印象因素的。</font>
    - <font style="color:rgb(1, 1, 1);">对CRAG有利的点：LLaMA2-hf-7b下self-RAG的实验是CRAG作者做的（带*）。</font>
    - <font style="color:rgb(1, 1, 1);">对self-RAG有利的点：selfRAG-LLaMA2-hf-7b实验中的这个大模型本身是经过self-RAG自己的微调的。</font>
+ <font style="color:rgb(1, 1, 1);">self-RAG的发布时间还在CRAG之前，从self-RAG的结构和实验其实可以感受到作者在背后的努力尝试了。</font>

<font style="color:rgb(0, 0, 0);">叠个甲，这里并非拉踩，只是引发的思考。在做RAG内部的自动调优，是否需要很多的判断条件，亦或者是让模型自动纠正的方式，还有待进一步探索。</font>

### <font style="color:rgb(0, 0, 0);">判别模型的提升和使用</font>
<font style="color:rgb(0, 0, 0);">尽管文章没有对判别模型的训练给出更多细节，但是就T5这个的效果来看，小一些的模型似乎也能够得到不错的效果，这给了应用场景的使用很大的支持。在RAG的应用中，内部是可以做很多业务定制和优化的，类似论文中的判别器，在应用场景中可能就是精排模型或者是判别模型。</font>

<font style="color:rgb(0, 0, 0);">在</font>[<font style="color:rgb(0, 0, 0);">心法利器[106]  基础RAG-调优方案</font>](http://mp.weixin.qq.com/s?__biz=MzIzMzYwNzY2NQ==&mid=2247489349&idx=1&sn=e68ad131621a0cf94d5532f5da2d0199&chksm=e8824fdbdff5c6cdcb8e683b67b07d996af3cf51107099529e4c12b859e8192be7d05d07f479&scene=21#wechat_redirect)<font style="color:rgb(0, 0, 0);">中，我就有提到其中一个优化方向是精排模型，对已有的知识进行排序和筛选，同时也可以做过滤，以尽可能为大模型的提供精准可靠的知识，这点在本文中得到了体现。这里的方案设计，更多还是依赖于对数据、问题的理解，现有资源配置，以及个人技术方案的储备：</font>

+ <font style="color:rgb(1, 1, 1);">不同场景的数据具有很多不同的特点，例如有些场景口语化严重，有些场景专业名词多等，前者用泛化模型收益会比较高，后者用词典做挖掘则更关键。</font>
+ <font style="color:rgb(1, 1, 1);">对问题的理解来源于对数据的理解，只有看过足够的数据才知道问题的特性，并根据特定采取不一样的方案策略。</font>
+ <font style="color:rgb(1, 1, 1);">self-RAG下使用了微调大模型用于内部的判别决策，并得到了效果，这里有一个前提是真的有资源去微调和部署。</font>
+ <font style="color:rgb(1, 1, 1);">厚积薄发，只有积累足够的知识才能融会贯通，在应用中想到并且实践。</font>

