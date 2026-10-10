很多`from llama import BasicModelRunner`这样的代码，不是要安装`llama`库，**要安装的是**`**lamini**`**库**，`llama`只是`lamini`库的一个子集。以下为`lamini`库的安装和使用说明。

##  安装
`lamini`库的安装非常简单，只需要执行以下命令即可：

```python
pip install lamini
```

## 注册
> 打不开
>

接下来需要去[ lamini官网](https://www.lamini.ai/)注册一个账号获取api key，才能使用`lamini`库的全部功能。

账号注册可以使用谷歌邮箱（默认）或者其他邮箱。注册完成后点击官网左上角的`Account`即可看到自己的 api key 以及剩余额度。

## 使用
### 默认方式
`lamini`默认需要在你的用户目录下创建一个配置文件 `~/.powerml/configure_llama.yaml`，然后按如下方式写入配置信息：

```yaml
production:
    key: "<YOUR-KEY-HERE>"
```

用户目录在Windows系统一般为`C:\Users\Administrator`， 在 Linux/maxOS 一般为`~/`。

### 简便方式
需要用到`llama`的`LLMEngine`或`BasicModelRunner`类时，直接将`production.key`写入类的参数`config`中即可，比如：

```python
llm = LLMEngine(
    id="example_llm",
    config={"production.key": "<YOUR-KEY-HERE>"}
)
```

或者：

```python
non_finetuned = BasicModelRunner("meta-llama/Llama-2-7b-hf", 
                                 config={"production.key": "<YOUR-KEY-HERE>"})

```

只需将`<YOUR-KEY-HERE>`替换为在`lamini`官网的api key即可。如果担心代码泄露api key的话，可以仿照使用ChatGPT的方式，用一个配置文件来存储`production.key`

