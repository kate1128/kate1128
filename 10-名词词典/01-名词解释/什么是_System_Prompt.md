System Prompt 是随着 ChatGPT API 开放并逐步得到大量使用的一个新兴概念，事实上，**它并不在大模型本身训练中得到体现，而是大模型服务方为提升用户体验所设置的一种策略**。

<br/>color2
具体来说，在使用 ChatGPT API 时，可以设置两种 Prompt：

+ 一种是 System Prompt，该种 Prompt 内容会在整个会话过程中持久地影响模型的回复，且相比于普通 Prompt 具有更高的重要性；
+ 另一种是 User Prompt，这更偏向于平时提到的 Prompt，即需要模型做出回复的输入。

<br/>

一般设置 System Prompt 来对模型进行一些初始化设定，例如，可以在 System Prompt 中给模型设定希望它具备的人设如一个个人知识库助手等。System Prompt 一般在一个会话中仅有一个。在通过 System Prompt 设定好模型的人设或是初始设置后，可以通过 User Prompt 给出模型需要遵循的指令。

<details class="lake-collapse"><summary id="u0f0cd95f"><span class="ne-text">eg： 当需要一个幽默风趣的个人知识库助手，并向这个助手提问我今天有什么事时，可以构造如下的 Prompt：</span></summary><pre data-language="properties" id="HLSI7" class="ne-codeblock language-properties"><code>{
    &quot;system prompt&quot;: &quot;你是一个幽默风趣的个人知识库助手，可以根据给定的知识库内容回答用户的提问，注意，的回答风格应是幽默风趣的&quot;,
    &quot;user prompt&quot;: &quot;我今天有什么事务？&quot;
}</code></pre></details>
通过如上 Prompt 的构造，可以让模型以幽默风趣的风格回答用户提出的问题。

