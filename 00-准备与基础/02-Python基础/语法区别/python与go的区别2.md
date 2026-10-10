#  变量与类型系统
## 多变量声明
```python
# Python
x, y, z = 10, 3.14, "text"
```

```go
// Go
var (
    a int = 10         // 显式类型
    b     = 3.14       // 类型推断
    c     = "text"
)

// 短声明（只能在函数内）
x, y := 42, "hello"   // 同时声明多个不同类型变量
```

## 零值机制
```python
# Python没有零值概念，未赋值的变量会报错
# 但类属性可以有默认值
```

```go
// Go所有类型都有零值：
var (
    num int     // 0
    str string  // ""
    ptr *int    // nil
    ok  bool    // false
)
```

## 类型转换
```python
# Python自动类型转换
result = 3 + 5.2  # 自动转为float
```

```go
// Go必须显式转换
var a int = 42
var b float64 = float64(a)  // 强制类型转换
var c int32 = int32(a)
```

# 作用域与可见性
## Python
```python
# 通过命名约定区分
_private_var = "内部使用"  # 单下划线约定私有
public_var = "公开"      # 模块级作用域
```

## Go
```go
// 通过首字母大小写控制可见性
var privateVar string = "包内可见"  // 小写字母开头
var PublicVar string = "可导出"    // 大写字母开头

// 块级作用域
func demo() {
    if tmp := 10; tmp > 5 {  // tmp只在if块内可见
        fmt.Println(tmp)
    }
}
```

# 函数参数处理
## 默认参数
```python
# Python支持默认参数
def greet(name="Guest"):
    print(f"Hello {name}")
```

```go
// Go不支持默认参数，需通过结构体参数实现
type GreetOptions struct {
    Name string
}

func Greet(opts GreetOptions) {
    if opts.Name == "" {
        opts.Name = "Guest"
    }
    fmt.Printf("Hello %s", opts.Name)
}

// 调用
Greet(GreetOptions{Name: "Alice"})
```

## 可变参数
```python
# Python
def sum(*nums):
    return sum(nums)
```

```go
// Go
func Sum(nums ...int) int {  // ...语法
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}
```

# 结构体 vs 类
## Python类
```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def magnitude(self):
        return (self.x**2 + self.y**2)**0.5

v = Vector(3, 4)
print(v.magnitude())
```

## Go结构体
```go
type Vector struct {
    X, Y float64  // 字段首字母大写才可导出
}

// 方法定义
func (v Vector) Magnitude() float64 {  // 值接收者
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// 使用
v := Vector{X: 3, Y: 4}
fmt.Println(v.Magnitude())

// 指针接收者（修改结构体内容）
func (v *Vector) Scale(f float64) {
    v.X *= f
    v.Y *= f
}
```

# 接口实现
## Python鸭子类型
```python
class Writer:
    def write(self, data):
        pass

class FileWriter(Writer):
    def write(self, data):
        # 实现写入文件

        class NetworkWriter:  # 非显式继承
            def write(self, data):
# 实现网络传输
```

## Go接口
```go
type Writer interface {
    Write([]byte) (int, error)
}

type FileWriter struct{}

func (f FileWriter) Write(data []byte) (int, error) {
    // 实现文件写入
    return len(data), nil
}

type NetworkWriter struct{}

func (n NetworkWriter) Write(data []byte) (int, error) {
    // 实现网络传输
    return len(data), nil
}

// 隐式实现接口，无需显式声明
```

# 包与导入
## Python模块
```python
# utils.py
def helper():
    print("Help!")

# main.py
from utils import helper
import math as mth
```

## Go包
```go
// utils/utils.go
package utils

import "fmt"

// 首字母大写才能被外部访问
func Helper() {
    fmt.Println("Help!")
}

// main.go
package main

import (
    "math"           // 标准库
    "project/utils"  // 自定义包
    fmt "fmt"        // 别名
)

func main() {
    utils.Helper()
    fmt.Println(math.Sqrt(4))
}
```

# 特殊语法结构
## defer 语句
```go
func fileOperation() {
    f, _ := os.Open("file.txt")
    defer f.Close()  // 确保函数返回前执行

    // 处理文件内容...
    // 即使中间有return或panic，defer仍会执行
}
```

## 指针操作
```go
func pointerDemo() {
    a := 10
    ptr := &a        // 获取地址
    *ptr = 20        // 通过指针修改值
    fmt.Println(a)   // 输出20
}
```

## 类型断言
```go
func typeAssertion(val interface{}) {
    if s, ok := val.(string); ok {
        fmt.Println("字符串长度:", len(s))
    } else {
        fmt.Println("不是字符串")
    }
}
```

# 深度语法对比表
| **语法特性** | **Python** | **Go** |
| :--- | :--- | :--- |
| 空值表示 | `None` | `nil`
（指针/引用类型的零值） |
| 循环结构 | `for...in`
, `while` | 只有 `for`
（三种形式） |
| 错误处理 | `try...except` | `error`
返回值 + `panic/recover` |
| 模块/包 | 文件即模块 | 必须声明package + 严格目录结构 |
| 继承 | 显式类继承 | 通过结构体嵌入实现组合 |
| 多态 | 鸭子类型 | 接口隐式实现 |
| 内存管理 | 自动GC | 自动GC + 手动控制指针 |
| 代码格式化 | PEP8规范 | `gofmt`
强制统一格式 |
| 异步编程 | `async/await` | `goroutine`
 + `channel` |
| 元编程 | 装饰器、元类 | 有限反射（reflect包） |
| 泛型 | 鸭子类型/Type Hints | 1.18+ 加入泛型 |
| 依赖管理 | `pip`
 + `requirements.txt` | `go.mod`
 + `go.sum` |

# 关键差异总结
1. **类型系统**
Go的严格静态类型要求显式声明，而Python的动态类型允许灵活的类型变化
2. **错误哲学**
Go强调显式错误处理（`**if err != nil**`模式），Python推崇异常机制
3. **并发模型**
Go的`**goroutine**`和`**channel**`提供原生的CSP并发支持，Python依赖线程/进程+异步IO
4. **代码组织**
Go强制使用包和明确的代码结构，Python的模块系统更加灵活
5. **运行时效率**
Go编译为本地机器码，Python通过解释器执行字节码
6. **元编程能力**
Python在运行时修改类/对象的行为更灵活，Go需要依赖反射（有限支持）

建议根据项目需求选择：

+ **选择Python**：快速原型开发、数据科学、脚本任务
+ **选择Go**：高性能网络服务、系统编程、需要严格类型检查的场景
