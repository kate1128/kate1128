[07-输入.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1753081557160-228dd464-d3d9-4557-a698-4d1f6511f81a.pdf)

在 Python 中，`input()` 是一个用于从用户获取输入的内置函数。它会等待用户输入并返回输入的内容，通常作为字符串返回。下面是关于 `input()` 函数的详细介绍以及使用方法。

# `input()` 的基本用法
```python
user_input = input("请输入一些内容: ")
print(f"你输入的是: {user_input}")
```

## 解释：
1. `input()` 函数的参数是一个可选的提示信息。当程序调用 `input()` 时，会在控制台显示该提示信息，提示用户输入内容。
2. `input()` 会暂停程序的执行，直到用户输入内容并按下 **回车** 键。
3. 用户输入的内容会作为字符串返回，无论输入的内容是数字、字母还是其他符号，返回的都是字符串类型。

## 示例：
```python
name = input("请输入你的名字: ")
print(f"你好，{name}！")
```

**输出：**

```python
请输入你的名字: Alice
你好，Alice！
```

# `input()` 返回类型
默认情况下，`input()` 返回一个 **字符串**。即使用户输入的是数字或其他数据类型，返回的也会是字符串。如果需要将输入的字符串转换成其他类型（如整数或浮点数），可以使用相应的类型转换函数。例如：

```python
age = input("请输入你的年龄: ")
age = int(age)  # 将输入的字符串转换为整数
print(f"你输入的年龄是: {age}")
```

## 示例：
```python
# 获取用户的年龄，并将其转化为整数
age = int(input("请输入你的年龄: "))
print(f"你的年龄是: {age}")
```

如果用户输入了非数字内容，调用 `int()` 会抛出错误（`ValueError`），因此可以使用异常处理来避免程序崩溃。

## 异常处理示例：
```python
try:
    age = int(input("请输入你的年龄: "))
    print(f"你输入的年龄是: {age}")
except ValueError:
    print("请输入一个有效的数字！")
```

# 获取多个输入
如果需要一次获取多个输入，可以使用 `input()` 和 `split()` 函数配合来处理。例如，如果你想让用户输入多个数字，并将其拆分成一个列表，可以这样做：

```python
numbers = input("请输入几个数字，用空格隔开: ").split()
numbers = [int(num) for num in numbers]
print(f"你输入的数字是: {numbers}")
```

## 解释：
+ `split()` 方法将输入的字符串按空格拆分成多个子字符串（列表）。
+ 使用列表推导式将每个子字符串转换为整数。

---

# 处理数字输入
如果用户输入的是数字，并且你希望直接进行计算，可以将 `input()` 获取的内容转换为 `int` 或 `float` 类型。

```python
# 获取两个数字并进行加法运算
num1 = float(input("请输入第一个数字: "))
num2 = float(input("请输入第二个数字: "))
sum_result = num1 + num2
print(f"这两个数字的和是: {sum_result}")
```

## 总结：
+ `input()` 函数用于从用户获取输入，返回类型为字符串。
+ 可以使用 `int()` 或 `float()` 等函数将字符串转换为其他数据类型。
+ 提示信息是可选的，可以提供给用户明确的输入指导。
+ `input()` 还可以与 `split()` 配合使用，以便从一行输入中获取多个值。

# 常见用法总结
```python
# 基本输入
name = input("请输入你的名字: ")
print(f"你好，{name}！")

# 输入整数并进行计算
num1 = int(input("请输入第一个数字: "))
num2 = int(input("请输入第二个数字: "))
print(f"它们的和是: {num1 + num2}")

# 输入多个值并转换
values = input("请输入一些数字，用空格隔开: ").split()
numbers = [int(value) for value in values]
print(f"你输入的数字是: {numbers}")
```

`input()` 是非常基础但又强大的一个函数，它允许我们从控制台与用户交互，获取用户输入并根据需要进行处理。

