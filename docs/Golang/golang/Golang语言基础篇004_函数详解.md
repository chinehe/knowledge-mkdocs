函数是Go语言中的基本构建块，用于封装可重用的代码逻辑。理解函数的各种特性和使用方法对于编写高质量的Go程序至关重要。

## 1. 函数的基本概念

函数是一段可重复使用的代码块，接受输入参数并返回输出结果。在Go语言中，函数具有以下特点：

* **名称唯一**：同一个包中的函数名称不能重复，即使参数列表不一样。

  > * Go语言中不支持函数重载。
  >
  > * 同变量一样，函数名称的首字母的大小写会决定能否在包外访问。

- **函数也是一种类型**：函数也是一种类型，一个函数可以赋值给变量，也可以作为函数参数传递。还可以定义函数类型！

## 2. 函数的声明和定义

### 2.1. 基本语法

Go语言中声明函数的基本格式如下：

```go
func 函数名称(参数列表)(返回值列表){
    // 函数
}
```

### 2.2. 函数名称

- Go语言中不支持函数重载，同一个包中函数名称不能重复，即使参数列表不一样。
- 同变量一样，函数名称的首字母的大小写会决定能否在包外访问。

### 2.3. 参数列表

* 参数列表可以为空，也可以为多个。

* 当两个或多个连续的函数命名参数是同一类型，则除了最后一个类型之外，其他都可以省略。

  ```go
  // 标准写法
  func add(a int, b int) int {
      return a + b
  }
  
  // 简写形式
  func add(a, b int) int {
      return a + b
  }
  ```

* 支持不定参数

  ```go
  func test(s string, args ...int) {}
  ```

* 参数列表中的参数名称可以忽略（必须同时忽略所有参数名称）

  ```go
  func maxInt(int, int) int {
  	return math.MaxInt
  }
  ```

### 2.4. 返回值列表

* 函数可以返回任意数量的返回值。如果返回值为0或1个，括号可以省略。

  ```go
  // 返回单个值
  func add(a, b int) int {
      return b + a
  }
  
  // 返回多个值
  func swap(a, b string) (string, string) {
      return b, a
  }
  ```

* 返回值列表中，可以对返回值进行命名。命名后返回值变量将被初始化为对应类型的零值。

  ```go
  // 命名返回值
  func divideWithRemainder(dividend, divisor int) (quotient, remainder int) {
      quotient = dividend / divisor
      remainder = dividend % divisor
      return  // 自动返回命名的返回值
  }
  ```

* 返回值要么全都命名，要么全都不命名，不能混用。

* 当两个或多个连续的函数命名参数是同一类型，则除了最后一个类型之外，其他都可以省略。

## 3. 函数的调用

### 3.1. 基本调用

```go
func main() {
    // 调用无返回值函数
    sayHello()

    // 调用有返回值函数
    result := add(3, 5)
    fmt.Println(result)  // 8

    // 调用多返回值函数
    quotient, err := divide(10, 3)
    if err != nil {
        fmt.Println("错误:", err)
    } else {
        fmt.Printf("商: %d\n", quotient)
    }

    // 调用命名返回值函数
    q, r := divideWithRemainder(10, 3)
    fmt.Printf("商: %d, 余数: %d\n", q, r)
}
```

### 3.2. 忽略返回值

```go
// 忽略所有返回值
divide(10, 2)

// 忽略部分返回值（使用下划线）
_, err := divide(10, 0)
if err != nil {
    fmt.Println("除法运算出错:", err)
}
```

### 3.3. 可变参数函数

Go语言支持可变参数函数，可以接受不定数量的参数：

```go
// 可变参数函数
func sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

// 调用可变参数函数
func main() {
    fmt.Println(sum(1, 2, 3))        // 6
    fmt.Println(sum(1, 2, 3, 4, 5))  // 15

    // 传递切片给可变参数函数
    nums := []int{1, 2, 3, 4}
    fmt.Println(sum(nums...))  // 10
}
```

**注意：**

- 一个函数中，最多只能设置一个不定参数。

- 不定参数必须在参数列表中的最后位置。

- 调用时，不定参数传值的个数为0-n，传值的类型为定义的不定参数的类型。

- Golang 可变参数本质上就是 slice。

  ```go
  func test(args ...string) {
  	fmt.Printf("%T", args)
  }
  func main() {
  	a := []string{"A", "B", "C"}
  	test(a...) // []string
  	test() // []string
  }
  ```

  

### 3.4. 参数传递

#### 3.4.1 形参与实参

- 形参

  函数定义时的参数，可称为函数的形参。形参就像定义在函数体内的局部变量。

- 实参

  调用函数，传递过来的变量就是函数的实参.

```go
// 函数定义时的参数a，b为形参
func add(a, b int) int {
	return a + b
}

func main() {
  // 调用函数时，传递的变量为函数的实参
	c := add(1, 2)
	println(c)
}
```

#### 3.4.2. 值传递与引用传递

函数可以通过两种方式来传递参数：

- 值传递

  指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。

- 引用传递

  是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

>  无论是值传递，还是引用传递，传递给函数的都是变量的副本，不过，值传递是值的拷贝。引用传递是地址的拷贝，一般来说，地址拷贝更为高效。而值拷贝取决于拷贝的对象大小，对象越大，则性能越低。

> map、slice、chan、指针、interface默认以引用的方式传递。

示例：

```go
// 变更数组内容
func changeArr(arr [5]int) {
	for i := 0; i < 5; i++ {
		arr[i] = i
	}
}

// 变更切片内容
func changeSlice(slice []int) {
	for i := 0; i < 5; i++ {
		slice[i] = i
	}
}

func main() {
  // 数组测试
	arr := [5]int{0, 0, 0, 0, 0}
	fmt.Println(arr) // [0 0 0 0 0]
	changeArr(arr) // 数组为值传递
  // change函数修改数组内容后，原数组内容不变
	fmt.Println(arr) // [0 0 0 0 0]

  // 切片测试
	slice := []int{0, 0, 0, 0, 0}
	fmt.Println(slice) // [0 0 0 0 0]
	changeSlice(slice) // 切片为引用传递
  // change函数修改切片内容后，原数组内容变更
	fmt.Println(slice) // [0 1 2 3 4]
}
```

## 4. 函数返回return

### 4.1. 常规return

使用return关键字进行函数的返回操作：

```go
func test()(x,y int){
    // 某些操作
    return x,y // 函数返回
}
```

return 关键字后面返回的值类型和数量必须与返回值列表相同。

### 4.2. 无return

如果一个函数没有返回值列表，则函数体内，可以使用return，而自动结束：

```go
func changeSlice(slice []int) {
	for i := 0; i < 5; i++ {
		slice[i] = i
	}
  // 自动结束
}
```

如果需要中途结束，还是需要显式return：

```go
func changeSlice(slice []int) {
  if len(slice) == 0 || len(slice) > 100{
    return // 提前结束
  }
	for i := 0; i < 5; i++ {
		slice[i] = i
	}
}
```

### 4.3. 无参数return

没有参数的 return 语句被称作“裸”返回。这只能用于以下两种场景：

- 函数没有返回值列表：此时return表示函数执行结束。

  ```go
  func changeSlice(slice []int) {
    if len(slice) == 0 || len(slice) > 100{
      return // 提前结束
    }
  	for i := 0; i < 5; i++ {
  		slice[i] = i
  	}
  }
  ```

- 函数返回值都命名了：此时return返回值列表中各个变量的当前值

  ```go
  func divide(a, b int) (res, mod int) {
  	res = a / b
  	mod = a % b
  	return
  }
  ```

  

命名返回参数可被同名局部变量遮蔽，此时需要显式返回。

```go
func add(x, y int) (z int) {
    { // 不能在一个级别，引发 "z redeclared in this block" 错误。
        var z = x + y
        // return   // Error: z is shadowed during return
        return z // 必须显式返回。
    }
}
```



## 5. 函数也是一种类型

### 5.1. 函数赋值给变量

可以定义函数类型的变量，将函数赋值给变量：

```go
func add(a, b int) int {
    return a + b
}

func subtract(a, b int) int {
    return a - b
}

func main() {
    // 函数赋值给变量
    var operation func(int, int) int
    operation = add
    fmt.Println(operation(5, 3))  // 8

    // 短变量声明
    op := subtract
    fmt.Println(op(5, 3))  // 2
}
```

### 5.2. 函数作为参数

函数的参数可以为函数类型，调用时将具体的函数传入：

```go
// 函数作为参数
func calculate(a, b int, op func(int, int) int) int {
    return op(a, b)
}

func multiply(a, b int) int {
    return a * b
}

func main() {
    result := calculate(5, 3, multiply)
    fmt.Println(result)  // 15

    // 使用匿名函数
    result2 := calculate(5, 3, func(a, b int) int {
        return a / b
    })
    fmt.Println(result2)  // 1
}
```

### 5.3. 函数作为返回值

函数的返回值可以为函数类型，返回时将具体的函数作为结果返回：

```go
// 函数作为返回值
func getOperation(op string) func(int, int) int {
    switch op {
    case "add":
        return func(a, b int) int { return a + b }
    case "subtract":
        return func(a, b int) int { return a - b }
    case "multiply":
        return func(a, b int) int { return a * b }
    default:
        return func(a, b int) int { return 0 }
    }
}

func main() {
    addOp := getOperation("add")
    fmt.Println(addOp(5, 3))  // 8

    mulOp := getOperation("multiply")
    fmt.Println(mulOp(5, 3))  // 15
}
```

### 5.4. 定义函数类型

还可以将一个函数声明定义为一个类型：

```go
type OperateFunc func(a, b int)int
```

定义函数类型后，拥有相同参数列表和返回值列表的函数，都被视为该类型：

```go
// 定义函数类型
type OperateFunc func(a, b int) int

func add(a, b int) int {
	return a + b
}

func sub(a, b int) int {
	return a - b
}

func multiply(a, b int) int {
	return a * b
}

func divide(a, b int) int {
	return a / b
}

// 该方法传入一个OperateFunc类型
func operate(a, b int, operate OperateFunc) int {
	return operate(a, b)
}

func main() {
  // 调用时，可以将所有拥有相同参数列表和返回值的函数当成OperateFunc类型传入
	fmt.Println(operate(1, 2, add))
	fmt.Println(operate(1, 2, sub))
	fmt.Println(operate(1, 2, multiply))
	fmt.Println(operate(1, 2, divide))
}
```



## 6. 匿名函数和闭包

### 6.1. 匿名函数

匿名函数是指不需要定义函数名的一种函数实现方式。1958年LISP首先采用匿名函数。Go语言支持随时在代码里定义匿名函数。

匿名函数由一个不带函数名的函数声明和函数体组成。匿名函数的优越性在于可以直接使用函数内的变量，不必申明：

```go
func main() {
    // 定义并立即执行匿名函数
    result := func(a, b int) int {
        return a + b
    }(3, 5)
    fmt.Println(result)  // 8

    // 将匿名函数赋值给变量
    square := func(x int) int {
        return x * x
    }
    fmt.Println(square(4))  // 16
}
```

### 6.2. 闭包

闭包是由函数及其相关引用环境组合而成的实体(即：闭包=函数+引用环境)。

> 官方定义：一个拥有许多变量和绑定了这些变量的环境的表达式（通常是一个函数），因而这些变量也是该表达式的一部分。

闭包是匿名函数与其外部环境的组合，可以访问外层函数的变量：

```go
// 闭包示例
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

func main() {
    // 创建计数器
    c1 := counter()
    fmt.Println(c1())  // 1
    fmt.Println(c1())  // 2
    fmt.Println(c1())  // 3

    // 创建另一个独立的计数器
    c2 := counter()
    fmt.Println(c2())  // 1
    fmt.Println(c1())  // 4
}
```

在该例子中，变量count就是“被绑定的环境变量”，返回的函数就是“绑定变量的函数”。



除了闭包中定义的环境变量外，还可以使用参数传递环境变量：

```go
// 闭包捕获变量
func createMultiplier(factor int) func(int) int {
    return func(value int) int {
        return value * factor
    }
}

func main() {
    double := createMultiplier(2)
    triple := createMultiplier(3)

    fmt.Println(double(5))   // 10
    fmt.Println(triple(5))   // 15
}
```

## 7. defer语句

defer语句用于延迟函数的执行，直到包含defer语句的函数即将返回时才执行（常用来做资源清理工作）：

```go
func readFile(filename string) {
    file, err := os.Open(filename)
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()  // 确保文件在函数结束时关闭

    // 处理文件...
    // 即使发生错误，file.Close()也会被执行
}

func main() {
    // defer的执行顺序是后进先出(LIFO)
    defer fmt.Println("最后执行")
    defer fmt.Println("第二执行")
    defer fmt.Println("最先执行")

    fmt.Println("函数主体")
    // 输出顺序：
    // 函数主体
    // 最先执行
    // 第二执行
    // 最后执行
}
```

**特性：**

- 延迟调用直到return前才会被执行（常用来做资源清理工作）。

- 多个defer语句，按照先进后出的方式执行

- defer语句中的变量，在defer声明时就决定了。

  ```go
  func main() {
  	a := 1
  	fmt.Printf("%s-%d\n", "start", a)
  	a++
  	defer func(a int) {
  		fmt.Printf("%s-%d\n", "defer", a)
  	}(a)
  	a++
  	fmt.Printf("%s-%d\n", "end", a)
  }
  // 输出
  start-1
  end-3  
  defer-2
  ```

  

## 8. main函数与init函数

在Go语言中，main函数和init函数是两个特殊的函数，它们在程序执行过程中扮演着重要角色。main函数是程序的入口点，而init函数用于包的初始化。

### 8.1. main函数

main函数是每个可执行Go程序的入口点，程序从main函数开始执行。

#### 8.1.1. main函数的特点

main函数具有以下特点：

- main函数必须存在于main包中
- main函数不接受任何参数
- main函数不返回任何值
- 每个可执行程序有且仅有一个main函数
- main函数在所有init函数执行完毕后才会执行

```go
package main

import "fmt"

func main() {
    fmt.Println("程序开始执行")
    // 程序的主要逻辑写在这里
    fmt.Println("程序执行结束")
}
```

#### 8.1.2. main函数的作用

main函数是程序执行的起点，所有的业务逻辑通常从这里开始展开。在main函数中，我们可以：

- 初始化应用程序
- 调用其他函数处理业务逻辑
- 启动服务器
- 处理程序的主循环等

```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func main() {
    // 初始化配置
    fmt.Println("初始化应用程序...")
    
    // 启动Web服务器
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, World!")
    })
    
    fmt.Println("服务器启动在 :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### 8.2. init函数

init函数是Go语言中用于包级别初始化的特殊函数。它在程序启动时自动执行，用于执行包的初始化操作。

#### 8.2.1. init函数的特点

init函数是用于程序执行前做包初始化的函数，比如可以用来初始化包内的变量。init函数具有以下等特性：

* 定义init函数不能有参数和返回值。
* 每个包/源文件里面可以包含多个init函数。

* init函数在main函数执行之前，自动被调用，不能被其他函数调用。

```go
package main

import "fmt"

// 全局变量
var globalVar string

// init函数在main函数之前执行
func init() {
    globalVar = "初始化完成"
    fmt.Println("init函数执行")
}

func main() {
    fmt.Println(globalVar)
}
```

#### 8.2.2. Go初始化执行顺序

Go程序的初始化过程遵循特定的顺序：

1. 初始化导入的包（按依赖顺序）
2. 初始化全局变量
3. 执行init函数
4. 执行main函数

```go
// utils.go
package main

import "fmt"

func init() {
    fmt.Println("utils包的init函数")
}

// main.go
package main

import "fmt"

func init() {
    fmt.Println("main包的init函数")
}

func main() {
    fmt.Println("main函数执行")
}
```

输出结果：
```
utils包的init函数
main包的init函数
main函数执行
```

#### 8.2.3. init函数的用途

init函数通常用于以下场景：

- 初始化全局变量
- 检查程序状态
- 注册资源
- 执行一次性设置任务

```go
package database

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq" // PostgreSQL驱动
)

var db *sql.DB

func init() {
    var err error
    db, err = sql.Open("postgres", "user=username dbname=mydb sslmode=disable")
    if err != nil {
        panic(fmt.Sprintf("数据库连接失败: %v", err))
    }
    
    // 测试连接
    if err = db.Ping(); err != nil {
        panic(fmt.Sprintf("数据库连接测试失败: %v", err))
    }
    
    fmt.Println("数据库连接成功")
}

func GetDB() *sql.DB {
    return db
}
```

#### 8.2.4. 多个init函数的执行顺序

* 同文件中定义的多个init函数，根据函数声明的位置顺序从上到下执行。

* 同包中不同文件定义的多个init函数，根据源文件名从小到大的顺序执行。

* 不同包中的init函数，如果不互相依赖的话，根据main包中import的顺序执行。

* 不同包中的init函数，如果存在依赖的话，则先调用最早被依赖的package中的init()，最后调用main函数。

> 如果init函数中使用了println()或者print()你会发现在执行过程中这两个不会按照你想象中的顺序执行。这两个函数官方只推荐在测试环境中使用，对于正式环境不要使用。
> 

### 8.3. main函数与init函数的协作

main函数和init函数在程序启动过程中协同工作，各自承担不同的职责：

```go
// config.go
package main

import "fmt"

var config map[string]string

func init() {
    fmt.Println("初始化配置...")
    config = make(map[string]string)
    config["app_name"] = "MyApp"
    config["version"] = "1.0.0"
    fmt.Println("配置初始化完成")
}

// logger.go
package main

import "fmt"

func init() {
    fmt.Println("初始化日志系统...")
    // 日志系统初始化代码
    fmt.Println("日志系统初始化完成")
}

// main.go
package main

import "fmt"

func init() {
    fmt.Println("main包初始化...")
}

func main() {
    fmt.Printf("启动应用: %s 版本: %s\n", config["app_name"], config["version"])
    fmt.Println("应用程序主逻辑执行")
}
```

输出结果：
```shell
初始化配置...
配置初始化完成
初始化日志系统...
日志系统初始化完成
main包初始化...
启动应用: MyApp 版本: 1.0.0
应用程序主逻辑执行
```

通过合理使用main函数和init函数，可以让Go程序具有清晰的启动流程和良好的初始化机制。
