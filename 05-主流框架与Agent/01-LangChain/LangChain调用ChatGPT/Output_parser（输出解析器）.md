`OutputParsers` 将语言模型的原始输出转换为可以在下游使用的格式。 OutputParsers 有几种主要类型，包括：

+ 将 LLM 文本转换为结构化信息（例如 JSON） 
+ 将 ChatMessage 转换为字符串 
+ 将除消息之外的调用返回的额外信息（如 OpenAI 函数调用）转换为字符串

最后，将模型输出传递给 `output_parser`，它是一个 `BaseOutputParser`，这意味着它接受**字符串或 BaseMessage 作为输入**。 StrOutputParser 特别简单地将任何输入转换为字符串。

```python
from langchain_core.output_parsers import StrOutputParser

output_parser = StrOutputParser()
output_parser.invoke(output)
```

<details class="lake-collapse"><summary id="uf2536328"><span class="ne-text">output：</span></summary><pre data-language="python" id="L4oO6" class="ne-codeblock language-python"><code>'I carried luggage heavier than my body and dived into the bottom of the Nile River. After passing through several flashes of lightning, I saw a pile of halos, not sure if this is the place.'</code></pre></details>
从上面结果可以看到，通过输出解析器成功将 `ChatMessage` 类型的输出解析为了`字符串`

或者：

```python
from langchain.output_parsers import ResponseSchema
from langchain.output_parsers import StructuredOutputParser

gift_schema = ResponseSchema(name="gift",
                             description="Was the item purchased\
                             as a gift for someone else? \
                             Answer True if yes,\
                             False if not or unknown.")

delivery_days_schema = ResponseSchema(name="delivery_days",
                                      description="How many days\
                                      did it take for the product\
                                      to arrive? If this \
                                      information is not found,\
                                      output -1.")

price_value_schema = ResponseSchema(name="price_value",
                                    description="Extract any\
                                    sentences about the value or \
                                    price, and output them as a \
                                    comma separated Python list.")


response_schemas = [gift_schema, 
                    delivery_days_schema,
                    price_value_schema]

output_parser = StructuredOutputParser.from_response_schemas(response_schemas)

format_instructions = output_parser.get_format_instructions()

print(format_instructions)
```



