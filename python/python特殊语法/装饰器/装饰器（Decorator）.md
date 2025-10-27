在 Python 中，**装饰器（Decorator）** 是一种强大的语法特性，它允许你在不修改原函数代码的前提下，动态地为函数或类添加额外功能。装饰器的核心思想是「包装函数」，通过 `@` 符号实现简洁的语法糖。

#  装饰器的本质
装饰器本质上是一个 **高阶函数**（接受函数作为参数并返回函数），它的核心作用：

+ **功能增强**：为函数添加日志、计时、权限校验等能力
+ **代码复用**：避免重复代码，实现横切关注点（Cross-cutting Concerns）
+ **动态修改**：运行时改变函数行为

# 基本用法：`@` 符号的作用
`@` 是 Python 的 **装饰器语法糖**，用于简化装饰器调用。以下两种写法完全等价：

```python
# 直接调用装饰器函数
def my_func():
    ...
my_func = decorator(my_func)
```

使用语法糖

```python
# 使用 @ 语法糖
@decorator
def my_func():
    ...
```

# 手把手示例：实现一个装饰器
## 基础装饰器：函数计时器
相当于 `timer（heavy_calculation(1)）`

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)  # 执行原函数
        elapsed = time.time() - start
        print(f"{func.__name__} 执行耗时: {elapsed:.2f}s")
        return result
    return wrapper

@timer
def heavy_calculation(n):
    time.sleep(n)
    return n * 2

heavy_calculation(1)  # 输出：heavy_calculation 执行耗时: 1.00s
```

**执行流程**：

1. `<font style="color:#DF2A3F;">@timer</font>`<font style="color:#DF2A3F;"> 等价于 </font>`<font style="color:#DF2A3F;">heavy_calculation = timer(heavy_calculation)</font>`
2. 调用 `heavy_calculation(1)` 实际执行的是 `wrapper(1)`
3. `wrapper` 包裹原函数，添加计时逻辑

## 保留元数据：`functools.wraps`
直接使用装饰器会导致原函数的 `__name__` 等元信息丢失：

```python
print(heavy_calculation.__name__)  # 输出：wrapper（错误！）
```

使用 `functools.wraps` 修正：

```python
from functools import wraps

def timer(func):
    @wraps(func)  # 保留原函数元数据
    def wrapper(*args, **kwargs):
        # ...同上...
    return wrapper

print(heavy_calculation.__name__)  # 输出：heavy_calculation（正确）
```

# 进阶用法：带参数的装饰器
若装饰器本身需要参数，需要 **嵌套三层函数**：

```python
def repeat(n_times):
    """执行函数 n 次的装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n_times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(n_times=3)
def say_hello():
    print("Hello!")

say_hello()
# 输出：
# Hello!
# Hello!
# Hello!
```

# 类装饰器：通过类实现
类装饰器通过实现 `__call__` 方法实现：

```python
class Logger:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print(f"调用函数: {self.func.__name__}")
        return self.func(*args, **kwargs)

@Logger
def add(a, b):
    return a + b

print(add(2, 3))  # 先打印日志，再输出结果 5
```

# 实际应用场景
| 场景 | 示例装饰器 |
| --- | --- |
| 权限验证 | `@login_required` |
| 缓存结果 | `@lru_cache` |
| 请求重试 | `@retry(times=3)` |
| 性能分析 | `@profile` |
| API 路由 | Flask 的 `@app.route("/")` |
| 输入验证 | `@validate_input(schema)` |


# 总结要点
1. `@`** 是语法糖**：`@decorator` 等价于 `func = decorator(func)`
2. **装饰器执行顺序**：从下往上应用（就近原则）

```python
@decorator1
@decorator2
def func(): ...
# 等价于 func = decorator1(decorator2(func))
```

3. **核心价值**：通过「包装函数」实现非侵入式功能增强
4. **最佳实践**：
    - 使用 `functools.wraps` 保留元数据
    - 优先选择无状态装饰器（避免副作用）
    - 复杂逻辑拆分为多个简单装饰器

装饰器是 Python 元编程的重要工具，掌握它能大幅提升代码的简洁性和可维护性。

