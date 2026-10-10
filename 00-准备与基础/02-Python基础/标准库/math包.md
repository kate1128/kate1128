在 Python 中，`math` 是一个内置模块，提供了许多数学相关的函数和常量，涵盖了基本的数学运算、三角函数、对数运算、常见的数学常量等。要使用 `math` 模块中的功能，需要先导入该模块：

```python
import math
```

下面是 `math` 模块的一些常用功能和常量：

### 1. **常见的数学常量**
+ `math.pi`：圆周率（π），大约为 `3.141592653589793`
+ `math.e`：自然对数的底数（e），大约为 `2.718281828459045`

```python
import math
print(math.pi)  # 输出: 3.141592653589793
print(math.e)   # 输出: 2.718281828459045
```

### 2. **常见的数学函数**
#### (1) **指数和对数**
+ `math.exp(x)`：返回 `e` 的 `x` 次方。
+ `math.log(x, base)`：返回 `x` 在指定 `base` 下的对数。如果没有提供 `base`，则默认是自然对数（以 `e` 为底）。
+ `math.log10(x)`：返回以 10 为底的对数。
+ `math.log2(x)`：返回以 2 为底的对数。

```python
import math
print(math.exp(1))    # 输出: 2.718281828459045
print(math.log(10))   # 输出: 2.302585092994046（自然对数）
print(math.log10(100)) # 输出: 2.0
print(math.log2(8))   # 输出: 3.0
```

#### (2) **三角函数**
+ `math.sin(x)`：返回 `x` 的正弦值，`x` 以弧度为单位。
+ `math.cos(x)`：返回 `x` 的余弦值，`x` 以弧度为单位。
+ `math.tan(x)`：返回 `x` 的正切值，`x` 以弧度为单位。
+ `math.asin(x)`：返回 `x` 的反正弦值，返回值以弧度表示。
+ `math.acos(x)`：返回 `x` 的反余弦值，返回值以弧度表示。
+ `math.atan(x)`：返回 `x` 的反正切值，返回值以弧度表示。

```python
import math
print(math.sin(math.pi / 2))  # 输出: 1.0（π/2 的正弦值）
print(math.cos(0))           # 输出: 1.0（0 的余弦值）
print(math.tan(math.pi / 4)) # 输出: 1.0（π/4 的正切值）
```

#### (3) **其他数学运算**
+ `math.factorial(x)`：返回 `x` 的阶乘，`x` 必须是非负整数。
+ `math.fabs(x)`：返回 `x` 的绝对值。
+ `math.floor(x)`：返回小于或等于 `x` 的最大整数（向下取整）。
+ `math.ceil(x)`：返回大于或等于 `x` 的最小整数（向上取整）。
+ `math.sqrt(x)`：返回 `x` 的平方根。

```python
import math
print(math.factorial(5))   # 输出: 120
print(math.fabs(-3.14))    # 输出: 3.14
print(math.floor(3.7))     # 输出: 3
print(math.ceil(3.2))      # 输出: 4
print(math.sqrt(16))       # 输出: 4.0
```

#### (4) **圆周相关函数**
+ `math.hypot(x, y)`：返回 `(x^2 + y^2)` 的平方根，等同于计算点 `(x, y)` 到原点的距离。

```python
import math
print(math.hypot(3, 4))  # 输出: 5.0（3-4 直角三角形的斜边长度）
```

#### (5) **角度与弧度转换**
+ `math.degrees(x)`：将弧度转换为角度。
+ `math.radians(x)`：将角度转换为弧度。

```python
import math
print(math.degrees(math.pi))   # 输出: 180.0
print(math.radians(180))       # 输出: 3.141592653589793
```

### 3. **其他有用的数学函数**
+ `math.isqrt(x)`：返回整数 `x` 的整数平方根（即不进行浮点运算）。
+ `math.prod(iterable)`：返回可迭代对象中所有元素的乘积。
+ `math.gcd(a, b)`：返回 `a` 和 `b` 的最大公约数。
+ `math.comb(n, k)`：返回从 `n` 中选取 `k` 的组合数。
+ `math.perm(n, k)`：返回从 `n` 中选取 `k` 的排列数。

```python
import math
print(math.isqrt(16))   # 输出: 4
print(math.prod([1, 2, 3, 4]))  # 输出: 24
print(math.gcd(56, 98))  # 输出: 14
print(math.comb(5, 3))   # 输出: 10
print(math.perm(5, 3))   # 输出: 60
```

### 4. **总结**
+ `math` 模块提供了大量的数学函数，适用于从基础的数学运算到更复杂的数学公式计算。
+ 该模块的函数大多数都以 **弧度** 为单位处理角度。
+ 常见的功能包括三角函数、指数对数运算、阶乘、取整、平方根等。
+ `math` 模块专注于数学运算，如果需要更高级的数值计算，可能会涉及到 `numpy` 等其他库。

通过了解并使用这些数学函数，你可以在 Python 中轻松实现各种数学计算。

