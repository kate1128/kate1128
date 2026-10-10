`LangChain`  提供了组件 链（chain） 可以将组件组合起来发挥 LLM 更强大的功能，但是语法非常复杂。`LCEL`  提供了一种管道语法，使从基本组件构建复杂链变得容易，可以通过 `LangChain`  完成下列组合：

```python
Chain = prompt | LLM |OutputParser 
```

链（Chains）通常将大语言模型（LLM）与提示（Prompt）结合在一起，基于此，可以对文本或数据进行一系列操作。

![](https://raw.githubusercontent.com/kate1128/llm-cookbook/54ad6728cde23add6e96b1f521ace605b5e60ca3/figures/LCEL.png)

