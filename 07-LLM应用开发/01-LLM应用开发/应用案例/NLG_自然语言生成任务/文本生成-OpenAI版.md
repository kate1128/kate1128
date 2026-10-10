# 引言
[什么是 NLG](https://www.yuque.com/qiaokate/su87gb/av05bind4f1otkhb)

任何知识都能被描述为文本或者说语言，从而先人的智慧才被记录在书卷上代代相传。

因此，绝大多数的自然语言处理任务都可以描述为自然语言生成任务，甚至是文本生成任务，将文本作为输入并将新的文本作为输出，这也是近两年大火的T5模型（`Text-to-Text Transfer Transformer`）的初衷。

举例来说，文本分类任务可以理解为输出类别名，如猫/狗、是/否；文本纠错任务可以理解为输入有错误的文本并理解，输出正确的文本描述；智能问答可以理解为根据背景知识及问句进行推理，输出相应的回答……

可以说，文本生成类任务的应用相当之广，本篇将介绍一些常见的文本生成任务，其中也包含一些曾经并不属于文本生成类任务，但如今也能使用`NLG`技术进行解决的任务。

```python
# # 安装一些必要的包
# !pip install openai
# # torch install 命令 https://pytorch.org/get-started/locally/
# !pip install torch==2.0.0+cpu torchvision==0.15.1+cpu --extra-index-url https://download.pytorch.org/whl/cpu
# !pip install tokenizers==0.13.2
# !pip install transformers==4.27.4
# !pip install --no-binary=protobuf protobuf==3.20.1
# !pip install sentencepiece==0.1.97
# !pip install redlines
# !pip install tenacity==8.2.2
```

> 暂时没有 OPENAI_API_KEY，用别的模型测
>

```python
# 配置openai api key
import openai
OPENAI_API_KEY = "输入你的key"  # TODO
openai.api_key = OPENAI_API_KEY
```

```python
# 查看API支持的模型
models = openai.Model.list()
print([x.id for x in models.data])
```

<details class="lake-collapse"><summary id="ud90c2766"><span class="ne-text">output：</span></summary><pre data-language="json" id="iOXbm" class="ne-codeblock language-json"><code>['babbage', 'davinci', 'text-davinci-edit-001', 'babbage-code-search-code', 'text-similarity-babbage-001', 'code-davinci-edit-001', 'text-davinci-001', 'ada', 'babbage-code-search-text', 'babbage-similarity', 'code-search-babbage-text-001', 'text-curie-001', 'code-search-babbage-code-001', 'text-ada-001', 'text-embedding-ada-002', 'text-similarity-ada-001', 'curie-instruct-beta', 'ada-code-search-code', 'ada-similarity', 'code-search-ada-text-001', 'text-search-ada-query-001', 'davinci-search-document', 'ada-code-search-text', 'text-search-ada-doc-001', 'davinci-instruct-beta', 'gpt-3.5-turbo', 'text-similarity-curie-001', 'code-search-ada-code-001', 'ada-search-query', 'text-search-davinci-query-001', 'curie-search-query', 'gpt-3.5-turbo-0301', 'davinci-search-query', 'babbage-search-document', 'ada-search-document', 'text-search-curie-query-001', 'whisper-1', 'text-search-babbage-doc-001', 'curie-search-document', 'text-davinci-003', 'text-search-curie-doc-001', 'babbage-search-query', 'text-babbage-001', 'text-search-davinci-doc-001', 'text-search-babbage-query-001', 'curie-similarity', 'curie', 'text-similarity-davinci-001', 'text-davinci-002', 'davinci-similarity', 'cushman:2020-05-03', 'ada:2020-05-03', 'babbage:2020-05-03', 'curie:2020-05-03', 'davinci:2020-05-03', 'if-davinci-v2', 'if-curie-v2', 'if-davinci:3.0.0', 'davinci-if:3.0.0', 'davinci-instruct-beta:2.0.0', 'text-ada:001', 'text-davinci:001', 'text-curie:001', 'text-babbage:001', 'ada:ft-personal-2023-05-07-07-50-50', 'ada:ft-personal-2023-04-15-13-19-25', 'ada:ft-personal-2023-04-15-13-29-50']</code></pre></details>
模型列表中包含内置的60多个可用模型，以及自己`fine tune`的模型，`fine tune`模型以"`ft-personal`"开头。

### `openai.Model.delete`
如需删除自己`fine tune`的模型，可以使用`openai.Model.delete`命令。

```python
# 如需删除自己fine tune的模型，可以使用openai.Model.delete命令。
openai.Model.delete('ada:ft-personal-2023-04-15-12-54-03')
```

<details class="lake-collapse"><summary id="u71d0d9de"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="oTKnP" class="ne-codeblock language-json"><code>&lt;Model model id=ada:ft-personal-2023-04-15-12-54-03 at 0x20506421090&gt; JSON: {
    &quot;deleted&quot;: true,
    &quot;id&quot;: &quot;ada:ft-personal-2023-04-15-12-54-03&quot;,
    &quot;object&quot;: &quot;model&quot;
}</code></pre></details>
### `openai.Model.list`
```python
models = openai.Model.list()
print([x.id for x in models.data if x.id.find('ft-personal') != -1])
```

<details class="lake-collapse"><summary id="u8edbbc10"><span class="ne-text" style="color: var(--jp-cell-prompt-not-active-font-color)">output：</span></summary><pre data-language="json" id="bSDny" class="ne-codeblock language-json"><code>['ada:ft-personal-2023-04-15-13-19-25', 'ada:ft-personal-2023-04-15-13-29-50']</code></pre></details>
# 文本摘要任务
[文本摘要任务](https://www.yuque.com/qiaokate/su87gb/el9t8anaouxw8y25)

# 文本纠错任务
[文本纠错任务](https://www.yuque.com/qiaokate/su87gb/txlms8kh3zhuemiv)

# 机器翻译任务
[机器翻译任务](https://www.yuque.com/qiaokate/su87gb/py009hgsla7vu1w2)

# 相关文献
+ 【1】[自然语言生成介绍](https://easyai.tech/ai-definition/nlg/)
+ 【2】[fine tune tutorial](https://platform.openai.com/docs/guides/fine-tuning/preparing-your-dataset)
+ 【3】[文本摘要任务介绍](https://zhuanlan.zhihu.com/p/83596443)
+ 【4】[中文文本纠错算法整体介绍](https://zhuanlan.zhihu.com/p/583590525)
+ 【5】[中文文本纠错任务简介](https://zhuanlan.zhihu.com/p/545776580)
+ 【6】[pycorrector介绍](https://github.com/shibing624/pycorrector)
+ 【7】[长文本英翻中tutorial](https://github.com/openai/openai-cookbook/blob/main/examples/book_translation/translate_latex_book.ipynb)

