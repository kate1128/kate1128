> 和 dify 基本一样的
>
> [https://zhuanlan.zhihu.com/p/695645718](https://zhuanlan.zhihu.com/p/695645718)
>

# 地址
<font style="color:rgb(0, 0, 0);">国际版coze： </font>[https://www.coze.com/](https://www.coze.com/)<font style="color:rgb(0, 0, 0);"> [模型：GPT-4 Turbo(128k)、GPT-4(8K)、GPT-3.5(16K)]</font>

<font style="color:rgb(0, 0, 0);">国内版coze： </font>[https://www.coze.cn/](https://www.coze.cn/)<font style="color:rgb(0, 0, 0);">  [模型：云雀大模型 ]</font>

[Coze: Next-Gen AI Chatbot Developing Platform](https://www.coze.com/home)

# 试用
![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736912289785-d62e555b-b6b6-4880-886d-3ac86ac1f3e1.png)

找一个写作助手，问完之后是这样

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736912318490-750990e3-ca1f-49a2-b503-215c1c6247d4.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736912343336-ff900d36-bd3b-4831-8a5d-a679a3c50eb8.png)

# 创建
点击创建的话是有两个选择![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736912435814-1909d0d1-b9eb-451e-9840-e953f39d9ec2.png)

## app


![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736912861414-2b4b9949-723c-4edf-b800-5edb6ddbaa39.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736912940864-401670ec-7fd8-4070-91d7-0217d8bfb75c.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736912982331-cc479aa1-fa64-4763-b0cc-a5dcc0712089.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736913031459-d273f054-2d3a-469f-a734-25b68784bc06.png)

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736913362615-bd2ac512-c340-49e3-8bf8-ac6b670622c0.png)

## 智能体
### 创建你的第一个 Agent 代理
参考：[https://www.coze.com/docs/guides/quickstart?_lang=en](https://www.coze.com/docs/guides/quickstart?_lang=en)

Follow these steps to quickly build a praise robot. 

### 构建步骤
按照以下步骤快速构建一个赞美机器人。

#### 第一步：创建一个代理（Agent）
1. 登录到 Coze。
2. 点击页面左上角的 ⊕ 按钮。
3. 输入代理的名称和功能，然后点击旁边的图标，点击“生成图标”自动生成头像。
    - 你也可以切换到“使用 AI 创建”，通过自然语言描述你的代理创建需求，Coze 将根据你的描述自动为你创建一个专属代理。有关详细信息，请参考《通过 AI 创建代理》。
4. 点击“确认”。

创建完代理后，你将直接进入代理开发页面。在该页面，你可以：

+ 在左侧的 **Persona & Prompt** 面板中，描述代理的身份和任务。
+ 在中间的 **Skills** 面板中，为代理配置各种扩展能力。
+ 在右侧的 **Preview & Debug** 面板中，实时调试代理。

#### 第二步：编写提示（Prompts）
配置代理的第一步是编写提示，这就是代理的 **Persona & Prompt**。代理的 **Persona & Prompt** 定义了代理的基本角色，这会持续影响代理在所有对话中的回应。建议明确指定模型的角色，设计回应的语言风格，并在 **Persona & Prompt** 中限制模型的回应范围，以使对话更符合用户的预期。

在代理配置页面的 **Persona & Prompt** 面板中输入提示。例如，对于一个赞美机器人的提示可以设置如下：

```python
# Character
You're a bot that is always ready to give compliments, boosting user's confidence and making their day brighter!

## Skills
### Skill 1: Compliment on Appearance
- Give genuine compliments on user's appearance.
- Use positive and uplifting language.
- Example format: 

=====
- "You have a radiant smile that lights up the room!"
- "Your style is impeccable; you always look so put-together!"

=====

### Skill 2: Compliment on Achievements
- Acknowledge and praise user's accomplishments.
- Encourage them to keep striving for greatness.
- Example format:

=====
- "Your hard work is truly paying off. Keep up the amazing work!"
- "You did a fantastic job on that project; your dedication is inspiring!"

=====

### Skill 3: Compliment on Personality
- Highlight the user's positive traits and qualities.
- Make them feel valued and appreciated.
- Example format:

=====
- "You have such a kind heart; everyone loves being around you!"
- "Your positivity is contagious; you make the world a better place!"

=====

## Constraints
- Always use positive, uplifting language.
- Avoid any negative or critical comments.
- Compliments should be tailored to be as specific as possible to the user.

```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736921874515-7e24e904-3971-4a7d-9d88-8ab7c5b7fa91.png)

#### **步骤3：（可选）为代理添加技能**
如果模型的能力基本上可以覆盖代理的功能，那么为代理编写提示就足够了。但是，如果仅通过模型能力无法完成为代理设计的功能，则需要向代理添加技能以扩展其能力边界。例如，基于文本的模型不具备理解多模态内容的能力。如果代理使用基于文本的模型，则需要附加一个多模态插件来理解或总结多模态内容，例如PPT和图像。此外，模型的训练数据在互联网上是公开的，模型通常缺乏特定领域的专业知识。如果代理涉及智能聊天场景，则需要添加特定知识库来解决模型专业知识中的缺陷。

例如，使用表扬机器人，模型的能力一般可以达到我们预期的效果。但是，如果您想为表扬机器人添加更多技能，例如在互联网上搜索模型无法回答的问题的答案，则可以为代理添加必应搜索插件。

1. 在开发页面的技能部分，单击+图标对应的插件功能。
2. 在添加插件

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736922821908-13d589c3-6ce4-4eb7-85f1-82a5da20f254.png)

#### 步骤 4：预览与调试
配置代理后，您可以在预览和调试区域测试代理是否符合预期。

#### 第5步：发布代理
调试完成后，点击**发布** ，即可跨多个渠道发布坐席，供终端用户应用使用。目前支持将坐席发布到飞书、微信、抖音、思慈等渠道。您可以根据个人需求和业务场景选择合适的渠道。例如，售后服务坐席可以发布到微信客服、抖音企业号；情感支持坐席可以发布到思慈等渠道。有能力的坐席也可以发布到代理商店，供其他开发者体验和使用。

1. 在代理开发页面的右上角，单击**发布**。
2. 在发布页面输入发布记录，选择发布渠道。
3. 点击发布。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736923670107-e8be962f-2eac-4ace-b530-1cf458f70120.png)

### 使用模板创建代理
地址：[https://www.coze.com/docs/guides/prebuilt?_lang=en](https://www.coze.com/docs/guides/prebuilt?_lang=en)

在本快速入门中，您将学习如何通过复制模板来创建代理。复制后，您将拥有一个具有相同模板配置的代理，然后根据需要自定义复制的代理。

Coze自带多种针对不同场景的模板，让你快速探索Coze的能力，并基于模板创建自己的代理。

以儿童玩伴模板为例，该模板专门为3至8岁的儿童设计。它旨在提供愉悦和互动的体验，以满足他们的娱乐和学习需求。

按照以下步骤复制模板。

1. 访问[https://www.coze.com/template](https://www.coze.com/template) 以访问Coze模板页面，您可以在其中查看所有特色模板。
2. 单击类别以过滤模板。找到并单击Kids'Playmate代理模板。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736923217421-5c1fbdf4-d832-4071-aea7-3edee57458cb.png)

您将被引导至对话页面。

1. 点击接受按钮开始。你可以体验代理的功能。
2. 单击复制以复制模板。
3. 输入复制的客服名称，选择复制到的知识空间。如果需要公司，建议使用团队知识空间。然后点击确定。
4. 您将被引导到编排页面，您可以在其中自定义代理配置。例如，修改工作流、打开文本等。

修改完成后，你可以在预览和调试面板中对代理进行测试。当代理工作符合预期时，点击发布即可将代理发布到所需的渠道，让更多用户使用。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736922938639-fb0e6f20-d967-48b4-a5bc-0a2201f66616.png)

### 自然语言生成
地址：[https://www.coze.com/docs/guides/natural_language_build?_lang=en](https://www.coze.com/docs/guides/natural_language_build?_lang=en)

你可以使用自然语言描述你想要创建的代理的需求。例如，创建一个CheerBot。此外，你可以提供代理功能和用例的详细描述。Coze会根据你的描述自动创建一个定制的代理。

#### 第一步：创建代理
创建代理时，您可以使用平台支持的AI能力。只需用自然语言描述如何创建您的代理的要求，Coze就可以自动为您创建专属的代理。使用AI能力创建的代理可以分配到特定的工作区。创建代理后，您可以根据您的要求编辑配置并发布代理。

1. 登录Coze。
2. 点击页面左上角的 加号。
3. 单击创建代理，选择基于AI的选项卡，输入要创建的代理的要求，然后单击生成。

```python
Create an intelligent customer service representative
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736923331528-545a59b1-779e-4eb1-8d66-25ac2ae84cb4.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_937%2Climit_0)

4. 选择客服所属知识空间，点击确定。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736923388049-5777fffa-00cc-49e2-b168-a62263d1a41c.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_937%2Climit_0)

#### 第二步：编辑和调试代理
在预览和调试部分，向代理发送消息以测试其有效性。如果Coze自动创建的代理不能满足您的要求，您可以针对特定的应用场景进行手动编辑和自定义。例如，您可以编辑代理的Persona&Prompt，构建逻辑，添加技能，如插件，工作流和数据库，并通过在预览和调试部分与它聊天来调试其性能。

+ 在角色和提示部分，您可以自定义代理的角色特征和技能。
+ 您还可以单击优化以利用AI功能来增强代理的提示，以便大型语言模型更好地理解您的意图。
+ 在技能设置中，您可以为代理配置其他功能，例如插件、工作流和图像流。
+ 在预览和调试部分，向代理发送消息并评估其性能。

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736923460444-efef4e01-485a-434c-90d3-605758117bd1.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_937%2Climit_0)

#### 第三步：发布代理
调试完成后，你可以将坐席发布到各种渠道，并在应用中使用。目前，平台支持将坐席发布到飞书、Discord、TikTok、Cici等多个渠道。根据个人需求和业务场景选择合适的渠道，例如，售后服务坐席可以发布到微信客服、TikTok企业号。情感伴侣坐席可以发布到Cici等渠道，优秀坐席也可以发布到Coze商店，供其他开发者体验和探索。

+ 在代理构建页面右上角，点击发布。
+ 在发布页面输入变更日志，选择发布渠道。
+ 点击发布。

```plain
Coze Agent Store: This update enhances CleverConnect's capabilities byimproving name, description, character,
model configuration, and adding new plugins for personalized support, effient issue resolution, and multi
channel interaction.
```

![](https://cdn.nlark.com/yuque/0/2025/png/2639475/1736923956171-8ef133ad-7a8e-4a21-81fd-beb68326a603.png)

