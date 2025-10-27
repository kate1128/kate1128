> + **格式化推荐使用 f-string**，它既简洁又高效，尤其适合 Python 3.6 及以上版本。
>

[06-输出.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1753081582872-c38584c1-eefc-4e2d-8b5b-8ce55d89910d.pdf)

Python 的 `print()` 函数是一个内置函数，用于将指定的内容输出到标准输出设备（通常是控制台）。这是 Python 中最常用的函数之一。以下是 `print()` 的详细功能、参数和用法。

#  基本用法
`print()` 将内容输出到控制台。

```python
print("Hello, World!")
# 输出：Hello, World!
```

# 参数详解
## `value(s)`
+ 可以是单个值，也可以是多个值，用逗号分隔。
+ 每个值都会被转换为字符串后输出。

```python
print("Hello", "Python", 3.9)
# 输出：Hello Python 3.9
```

##  `sep`
+ 用于指定多个值之间的分隔符，默认为一个空格 `" "`。

```python
print("apple", "banana", "cherry", sep=", ")
# 输出：apple, banana, cherry
```

## `end`
+ 用于指定打印语句的结尾内容，默认为换行符 `"\n"`。

```python
print("Hello", end=" ")
print("World!")
# 输出：Hello World!
```

##  `file`
+ 指定输出的目标，默认为 `sys.stdout`（标准输出设备）。
+ 可以设置为文件对象，将输出内容写入文件。

```python
with open("output.txt", "w") as f:
    print("Hello, File!", file=f)
# 内容被写入 output.txt 文件
```

## `flush`
+ 指定是否立即刷新输出缓冲区，默认值为 `False`。
+ 设置为 `True` 时会强制刷新，用于需要实时输出的场景。

```python
import time
print("Loading", end="", flush=True)
time.sleep(1)
print(". Done!")
# 输出：Loading. Done!
```

# 输出格式化
## 字符串拼接
通过 `+` 或 `f-string` 拼接字符串。

```python
name = "Alice"
age = 25
print("Name: " + name + ", Age: " + str(age))  # 使用 +
print(f"Name: {name}, Age: {age}")            # 使用 f-string
# 输出：Name: Alice, Age: 25
```

## 格式化字符串
使用 `str.format()` 方法进行占位符替换。

```python
print("Name: {}, Age: {}".format("Bob", 30))
print("Name: {0}, Age: {1}".format("Bob", 30))  # 位置参数
print("Name: {name}, Age: {age}".format(name="Bob", age=30))  # 关键字参数
# 输出：Name: Bob, Age: 30
```

# 常见用法
## 输出变量
```python
x = 42
print("The answer is", x)
# 输出：The answer is 42
```

## 输出带换行的多行内容
```python
print("Line 1\nLine 2\nLine 3")
# 输出：
# Line 1
# Line 2
# Line 3
```

## 输出到文件
```python
with open("log.txt", "a") as f:
    print("This is a log entry.", file=f)
# 内容被追加到 log.txt 文件中
```

## 实时进度显示
```python
import time
for i in range(5):
    print(f"\rProgress: {i+1}/5", end="", flush=True)
    time.sleep(1)
# 输出：实时更新进度条，显示 Progress: 1/5 到 Progress: 5/5
```

## 使用自定义分隔符
```python
items = ["apple", "banana", "cherry"]
print(*items, sep=" | ")
# 输出：apple | banana | cherry
```

# 注意事项
1. 如果输出包含非字符串内容（如整数或对象），`print()` 会自动调用 `str()` 将其转换为字符串。

```python
print([1, 2, 3])  # 输出：[1, 2, 3]
```

2. 如果需要打印原始格式（如转义字符），可以使用 `repr()`：

```python
print(repr("Hello\nWorld"))
# 输出：'Hello\nWorld'
```

# 练习
要打印三行，每行之后加一个空行，可以使用 Python 中的 `print()` 函数并结合 `end` 参数或使用 `\n` 来实现。这里是几种不同的方式：

## 方法 1：使用 `print()` 每行后自动加空行
```python
print("Line 1")
print()  # 打印空行
print("Line 2")
print()  # 打印空行
print("Line 3")
print()  # 打印空行
```

**输出：**

```python
Line 1

Line 2

Line 3

```

## 方法 2：使用 `end` 参数指定换行
你也可以通过 `end` 参数指定每行后加一个换行符 `"\n"`，即便 `print()` 默认会自动加换行。

```python
print("Line 1", end="\n\n")  # 每行后面加两个换行符
print("Line 2", end="\n\n")
print("Line 3", end="\n\n")
```

**输出：**

```python
Line 1

Line 2

Line 3

```

## 方法 3：使用格式化字符串（`f-string`）
通过 `f-string` 来格式化输出时，也可以在每行后添加一个空行。

```python
print(f"Line 1\n")
print(f"Line 2\n")
print(f"Line 3\n")
```

**输出：**

```python
Line 1

Line 2

Line 3

```

## 方法 4：使用循环和 `sep` 参数
如果你有多个行要打印，并且希望更灵活地管理输出格式，可以使用循环，结合 `sep` 和 `end`：

```python
lines = ["Line 1", "Line 2", "Line 3"]
for line in lines:
    print(line, end="\n\n")
```

**输出：**

```python
Line 1

Line 2

Line 3

```

# 格式化输出
这四种方式都可以帮助你打印出每行之后带有空行的输出，具体使用哪一种可以根据个人喜好或项目需要进行选择。在 Python 中，`print()` 的格式化输出非常灵活，支持多种方式来控制输出内容的格式。主要有三种常用的格式化输出方法：

1. **使用 **`%`** 操作符**（老式格式化方式）
2. **使用 **`str.format()`** 方法**（较为通用的格式化方法）
3. **使用 f-string（格式化字符串字面量）**（Python 3.6+ 推荐使用）

接下来详细解释这三种方法及其使用示例

## 使用 `%` 操作符（老式格式化）
> <font style="color:#DF2A3F;">看起来和 go 一样，语法不一样，逗号变成百分号</font>
>

这种方法是早期 Python 中的字符串格式化方式，类似于 C 语言中的 `printf` 函数。它通过 `%` 操作符来指定格式化字符串，并通过元组或字典提供值。

#### **基本用法**：
```python
name = "Alice"
age = 25
print("Name: %s, Age: %d" % (name, age))
# 输出：Name: Alice, Age: 25
```

+ `%s` 用于格式化字符串
+ `%d` 用于格式化整数
+ `%f` 用于格式化浮动数值（浮点数）
+ `%%` 输出 %

#### **格式化浮动数值**：
```python
pi = 3.14159
print("Pi value: %.2f" % pi)
# 输出：Pi value: 3.14  （保留两位小数）
```

## 使用 `str.format()` 方法（较为通用的格式化方法）
`str.format()` 是一种更加现代和灵活的格式化方式，它允许使用大括号 `{}` 占位符，并通过 `format()` 方法将值插入占位符中。

#### **基本用法**：
```python
name = "Alice"
age = 25
print("Name: {}, Age: {}".format(name, age))
# 输出：Name: Alice, Age: 25
```

#### **位置和关键字参数**：
可以通过位置参数或者关键字参数来控制插入的值：

```python
# 位置参数
print("Name: {0}, Age: {1}".format("Alice", 25))
# 输出：Name: Alice, Age: 25

# 关键字参数
print("Name: {name}, Age: {age}".format(name="Alice", age=25))
# 输出：Name: Alice, Age: 25
```

#### **格式控制**：
`str.format()` 还可以用于控制数字的格式，例如浮动数值、填充宽度等。

```python
# 控制浮点数的显示精度
pi = 3.1415926535
print("Pi value: {:.2f}".format(pi))  # 输出：Pi value: 3.14

# 对齐和填充
print("{:<10} | {:>10}".format("left", "right"))
# 输出：
# left       |      right

# 通过指定宽度和填充字符
print("{:*^10}".format("Python"))  # 输出：**Python**
```

## 使用 f-string（格式化字符串字面量）（Python 3.6+）
f-string（格式化字符串字面量）是 Python 3.6 中引入的一种方式，它提供了简洁且高效的格式化方法。只需在字符串前加上 `f` 或 `F`，然后在字符串中使用大括号 `{}` 插入表达式。

#### **基本用法**：
```python
name = "Alice"
age = 25
print(f"Name: {name}, Age: {age}")
# 输出：Name: Alice, Age: 25
```

#### **表达式支持**：
f-string 允许在 `{}` 中直接插入任何表达式，并自动进行求值：

```python
x = 10
y = 20
print(f"Sum of {x} and {y} is {x + y}")
# 输出：Sum of 10 and 20 is 30
```

#### **格式控制**：
f-string 也支持对数字进行格式化，例如控制浮动数值的精度或对齐：

```python
# 控制浮动数值的精度
pi = 3.1415926535
print(f"Pi value: {pi:.2f}")
# 输出：Pi value: 3.14

# 对齐和填充
print(f"{'left':<10} | {'right':>10}")
# 输出：left       |      right

# 填充字符
print(f"{'Python':*^10}")
# 输出：**Python**
```

## 比较总结
| **特性** | `%`** 操作符** | `str.format()` | **f-string** |
| --- | --- | --- | --- |
| **可读性** | 中等 | 高 | 非常高 |
| **灵活性** | 低 | 中等 | 高 |
| **格式化复杂性** | 简单 | 较复杂 | 简单且直观 |
| **性能** | 低 | 中等 | 高（推荐使用） |
| **支持版本** | Python 2 和 3 | Python 2.7 和 3.x | Python 3.6 及以上 |



+ **推荐使用 f-string**，它既简洁又高效，尤其适合 Python 3.6 及以上版本。

<br/>

+ 如果使用的是 Python 3.5 或更早的版本，`str.format()` 是一个非常好的选择。
+ `%` 操作符虽然被遗弃，但仍然被广泛使用，适用于一些简单的格式化场景，特别是对旧代码的兼容性要求较高时。

# 总结
`print()` 的格式化输出使得在 Python 中处理字符串更加灵活和高效。根据不同的需求和 Python 版本，可以选择最适合的格式化方法。对于较新的 Python 版本，f-string 是首选，因为它提供了最简洁且高效的语法。

