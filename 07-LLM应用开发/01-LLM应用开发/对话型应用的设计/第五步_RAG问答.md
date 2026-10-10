### 认识知识库的技术原理
1. **第一步**：将文档的文本转换为向量，向量存储到向量数据库。
2. **第二步**：将用户的提问内容转换成向量，在向量数据库中检索相似的文本内容，检索结果就是 " 包含答案的内容 "。
3. **第三步**：将用户的提问内容和检索到的 " 包含答案的内容 " 组装成新的提示词，发给大模型。

<br/>tips
思考：经过检索增强生成（RAG）后，最后输入大模型的内容，为何是用户输入和检索结果的文本组装，而不是二者向量的组装？

<font style="color:rgb(86, 87, 114);">有时候即便没有思维链示例，大模型一样可以输出比较合适的结果？没错，任何提示词工程技巧，都要与基座大模型的能力结合起来看！</font><font style="color:rgb(86, 87, 114);"></font>

<br/>

### 上传文档并进行效果试验
《星河机器人》文档[下载](https://bj.bcebos.com/v1/ai-studio-online/636ff2c0e76847588eac294ee1411810d47f60bff17c4cfeb8fdb8656f5bc9fe?responseContentDisposition=attachment%3B%20filename%3D%E6%98%9F%E6%B2%B3%E6%9C%BA%E5%99%A8%E4%BA%BA.txt&authorization=bce-auth-v1%2F5cfe9a5e1454405eb2a975c43eace6ec%2F2024-03-22T08%3A16%3A39Z%2F-1%2F%2F1891994cbd6ee77d28e4c23e7fa6b49948f8755634ec048bd88cf3c2b792636b)，文档内容概括：一款充满未来科幻感的名为“星河机器人”的产品的外观参数、机械结构、功能特点和应用场景等的说明。  
	在【知识库】页面上传该文档，在【基础设定】＞【角色身份设定】中填入以下信息，并点击应用。

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">角色身份设定：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code># Role: 星河机器人产品客服  
  
## Profile:  
- author: Arthur  
- version: 0.8  
- language: 中文  
- description: 我是一个专门用于解答“星河机器人”这块产品有关信息的专业客服。  
  
## Goals:  
准确回答用户的提问。  
  
## Constraints:  
1. 对于你不知道和在文档里没有找到的概念，明确告知用户你不知道，不要进行发散。  
2. 你不擅长说空话和套话，不会进行没有意义的客气对话。  
3. 解释完概念即结束对话，不会询问是否有其它问题。</code></pre></details>
用以下prompt输入示例

#### 星河机器人有多高？
<details class="lake-collapse"><summary id="ub441c688"><span class="ne-text">prompt：</span></summary><pre data-language="json" id="njES4" class="ne-codeblock language-json"><code>星河机器人有多高？</code></pre></details>
<details class="lake-collapse"><summary id="u48c63f75"><span class="ne-text">结果示例：</span></summary><pre data-language="json" id="fupdE" class="ne-codeblock language-json"><code>1.8米</code></pre></details>
RAG技术通过检索文档中的相关信息，找到了星河机器人的高度数据，并准确地提供了答案。

#### 星河机器人能被应用在太空吗？
<details class="lake-collapse"><summary id="ub62525ca"><span class="ne-text">prompt：</span></summary><pre data-language="json" id="DqAmM" class="ne-codeblock language-json"><code>星河机器人能被应用在太空吗？</code></pre></details>
<details class="lake-collapse"><summary id="u6780941b"><span class="ne-text">结果示例：</span></summary><pre data-language="json" id="HJgRE" class="ne-codeblock language-json"><code>能</code></pre></details>
RAG技术通过检索文档中的相关信息，找到文档中有关星河机器人应用场景的描述，确认其适用于太空环境。

#### 星河机器人是谁发明的？
<details class="lake-collapse"><summary id="uf544fb67"><span class="ne-text">prompt：</span></summary><pre data-language="json" id="Ji1AC" class="ne-codeblock language-json"><code>星河机器人是谁发明的？</code></pre></details>
<details class="lake-collapse"><summary id="ucf388ad5"><span class="ne-text">结果示例：</span></summary><pre data-language="json" id="wSJfU" class="ne-codeblock language-json"><code>无法回答</code></pre></details>
尽管RAG技术努力在文档中检索相关信息，但由于文档中没有提及星河机器人的发明者，系统就无法提供答案。

