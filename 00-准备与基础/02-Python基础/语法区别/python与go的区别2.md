# <font style="color:rgb(0, 0, 0);"> 变量与类型系统</font>
## <font style="color:rgb(0, 0, 0);">多变量声明</font>
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

## <font style="color:rgb(0, 0, 0);">零值机制</font>
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

## <font style="color:rgb(0, 0, 0);">类型转换</font>
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

# <font style="color:rgb(0, 0, 0);">作用域与可见性</font>
## <font style="color:rgb(0, 0, 0);">Python</font>
```python
# 通过命名约定区分
_private_var = "内部使用"  # 单下划线约定私有
public_var = "公开"      # 模块级作用域
```

## <font style="color:rgb(0, 0, 0);">Go</font>
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

# <font style="color:rgb(0, 0, 0);">函数参数处理</font>
## <font style="color:rgb(0, 0, 0);">默认参数</font>
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

## <font style="color:rgb(0, 0, 0);">可变参数</font>
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

# <font style="color:rgb(0, 0, 0);">结构体 vs 类</font>
## <font style="color:rgb(0, 0, 0);">Python类</font>
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

## <font style="color:rgb(0, 0, 0);">Go结构体</font>
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

# <font style="color:rgb(0, 0, 0);">接口实现</font>
## <font style="color:rgb(0, 0, 0);">Python鸭子类型</font>
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

## <font style="color:rgb(0, 0, 0);">Go接口</font>
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

# <font style="color:rgb(0, 0, 0);">包与导入</font>
## <font style="color:rgb(0, 0, 0);">Python模块</font>
```python
# utils.py
def helper():
    print("Help!")

# main.py
from utils import helper
import math as mth
```

## <font style="color:rgb(0, 0, 0);">Go包</font>
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

# <font style="color:rgb(0, 0, 0);">特殊语法结构</font>
## <font style="color:rgb(0, 0, 0);">defer 语句</font>
```go
func fileOperation() {
    f, _ := os.Open("file.txt")
    defer f.Close()  // 确保函数返回前执行

    // 处理文件内容...
    // 即使中间有return或panic，defer仍会执行
}
```

## <font style="color:rgb(0, 0, 0);">指针操作</font>
```go
func pointerDemo() {
    a := 10
    ptr := &a        // 获取地址
    *ptr = 20        // 通过指针修改值
    fmt.Println(a)   // 输出20
}
```

## <font style="color:rgb(0, 0, 0);">类型断言</font>
```go
func typeAssertion(val interface{}) {
    if s, ok := val.(string); ok {
        fmt.Println("字符串长度:", len(s))
    } else {
        fmt.Println("不是字符串")
    }
}
```

# <font style="color:rgb(0, 0, 0);">深度语法对比表</font>
| **<font style="color:rgb(0, 0, 0);">语法特性</font>** | **<font style="color:rgb(0, 0, 0);">Python</font>** | **<font style="color:rgb(0, 0, 0);">Go</font>** |
| :--- | :--- | :--- |
| <font style="color:rgb(0, 0, 0);">空值表示</font> | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">None</font>` | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">nil</font>`<br/><font style="color:rgb(0, 0, 0);">（指针/引用类型的零值）</font> |
| <font style="color:rgb(0, 0, 0);">循环结构</font> | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">for...in</font>`<br/><font style="color:rgb(0, 0, 0);">,</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">while</font>` | <font style="color:rgb(0, 0, 0);">只有</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">for</font>`<br/><font style="color:rgb(0, 0, 0);">（三种形式）</font> |
| <font style="color:rgb(0, 0, 0);">错误处理</font> | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">try...except</font>` | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">error</font>`<br/><font style="color:rgb(0, 0, 0);">返回值 +</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">panic/recover</font>` |
| <font style="color:rgb(0, 0, 0);">模块/包</font> | <font style="color:rgb(0, 0, 0);">文件即模块</font> | <font style="color:rgb(0, 0, 0);">必须声明package + 严格目录结构</font> |
| <font style="color:rgb(0, 0, 0);">继承</font> | <font style="color:rgb(0, 0, 0);">显式类继承</font> | <font style="color:rgb(0, 0, 0);">通过结构体嵌入实现组合</font> |
| <font style="color:rgb(0, 0, 0);">多态</font> | <font style="color:rgb(0, 0, 0);">鸭子类型</font> | <font style="color:rgb(0, 0, 0);">接口隐式实现</font> |
| <font style="color:rgb(0, 0, 0);">内存管理</font> | <font style="color:rgb(0, 0, 0);">自动GC</font> | <font style="color:rgb(0, 0, 0);">自动GC + 手动控制指针</font> |
| <font style="color:rgb(0, 0, 0);">代码格式化</font> | <font style="color:rgb(0, 0, 0);">PEP8规范</font> | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">gofmt</font>`<br/><font style="color:rgb(0, 0, 0);">强制统一格式</font> |
| <font style="color:rgb(0, 0, 0);">异步编程</font> | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">async/await</font>` | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">goroutine</font>`<br/><font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">+</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">channel</font>` |
| <font style="color:rgb(0, 0, 0);">元编程</font> | <font style="color:rgb(0, 0, 0);">装饰器、元类</font> | <font style="color:rgb(0, 0, 0);">有限反射（reflect包）</font> |
| <font style="color:rgb(0, 0, 0);">泛型</font> | <font style="color:rgb(0, 0, 0);">鸭子类型/Type Hints</font> | <font style="color:rgb(0, 0, 0);">1.18+ 加入泛型</font> |
| <font style="color:rgb(0, 0, 0);">依赖管理</font> | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">pip</font>`<br/><font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">+</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">requirements.txt</font>` | `<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">go.mod</font>`<br/><font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">+</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.05);">go.sum</font>` |


# <font style="color:rgb(0, 0, 0);">关键差异总结</font>
1. **<font style="color:rgb(0, 0, 0);">类型系统</font>**<font style="color:rgb(0, 0, 0);">  
</font><font style="color:rgb(0, 0, 0);">Go的严格静态类型要求显式声明，而Python的动态类型允许灵活的类型变化</font>
2. **<font style="color:rgb(0, 0, 0);">错误哲学</font>**<font style="color:rgb(0, 0, 0);">  
</font><font style="color:rgb(0, 0, 0);">Go强调显式错误处理（</font>`**<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.08);">if err != nil</font>**`<font style="color:rgb(0, 0, 0);">模式），Python推崇异常机制</font>
3. **<font style="color:rgb(0, 0, 0);">并发模型</font>**<font style="color:rgb(0, 0, 0);">  
</font><font style="color:rgb(0, 0, 0);">Go的</font>`**<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.08);">goroutine</font>**`<font style="color:rgb(0, 0, 0);">和</font>`**<font style="color:rgb(0, 0, 0);background-color:rgba(27, 31, 35, 0.08);">channel</font>**`<font style="color:rgb(0, 0, 0);">提供原生的CSP并发支持，Python依赖线程/进程+异步IO</font>
4. **<font style="color:rgb(0, 0, 0);">代码组织</font>**<font style="color:rgb(0, 0, 0);">  
</font><font style="color:rgb(0, 0, 0);">Go强制使用包和明确的代码结构，Python的模块系统更加灵活</font>
5. **<font style="color:rgb(0, 0, 0);">运行时效率</font>**<font style="color:rgb(0, 0, 0);">  
</font><font style="color:rgb(0, 0, 0);">Go编译为本地机器码，Python通过解释器执行字节码</font>
6. **<font style="color:rgb(0, 0, 0);">元编程能力</font>**<font style="color:rgb(0, 0, 0);">  
</font><font style="color:rgb(0, 0, 0);">Python在运行时修改类/对象的行为更灵活，Go需要依赖反射（有限支持）</font>

<font style="color:rgb(0, 0, 0);">建议根据项目需求选择：</font>

+ **<font style="color:rgb(0, 0, 0);">选择Python</font>**<font style="color:rgb(0, 0, 0);">：快速原型开发、数据科学、脚本任务</font>
+ **<font style="color:rgb(0, 0, 0);">选择Go</font>**<font style="color:rgb(0, 0, 0);">：高性能网络服务、系统编程、需要严格类型检查的场景</font>

