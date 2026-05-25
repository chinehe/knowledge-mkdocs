流程控制是任何编程语言的核心组成部分，它决定了程序的执行路径。Go语言提供了丰富的流程控制语句，包括条件语句、循环语句和跳转语句，帮助开发者精确控制程序的执行流程。

## 1. 条件语句

### 1.1. if语句

Go语言中if条件语句的格式如下：

```go
if 布尔表达式1 {
    // 布尔表达式1结果为true时执行
} else if 布尔表达式2 {
    // 布尔表达式1结果为false且布尔表达式2结果为true时执行
}else{
    // 前面的布尔表达式都为false时执行
}
```

- `else if `子语句可能没有，也可能又多个。
- `else `子语句可能没有，但不能有多个。
- `if` 、 `else if` 、 `else `子语句执行体内部都可以嵌套if条件语句

> Go语言中不支持三目运算符!

示例：

```go
// 基本if语句
if x > 0 {
    fmt.Println("x是正数")
}

// if-else语句
if x > 0 {
    fmt.Println("x是正数")
} else {
    fmt.Println("x不是正数")
}

// if-else if-else语句
if x > 0 {
    fmt.Println("x是正数")
} else if x < 0 {
    fmt.Println("x是负数")
} else {
    fmt.Println("x是零")
}
```



Go语言的if语句支持在条件判断前执行初始化语句：

```go
// 在if语句中声明和初始化变量
if x := computeValue(); x > 10 {
    fmt.Printf("计算结果%d大于10\n", x)
} else {
    fmt.Printf("计算结果%d不大于10\n", x)
}

// 注意：x的作用域仅限于if语句块内
// fmt.Println(x)  // 错误：未定义

// 文件操作示例
if file, err := os.Open("test.txt"); err != nil {
    fmt.Println("打开文件失败:", err)
} else {
    defer file.Close()
    // 处理文件...
}
```

### 1.2. 基本switch语句

switch 语句用于基于不同条件执行不同动作，每一个 case 分支都是唯一的，从上直下逐一测试，直到匹配为止：

格式：

```go
switch 外层表达式 {
    case 内层表达式1:
            // 内层执行体1
    case 内层表达式2:
            // 内层执行体2
            fallthrough
    case 内层表达式3,内层表达式4:
            // 内层执行体3
    default:
            // 默认执行
}
```

* 外层表达式和内层表达式都可以是任意类型，但需要保证两者的结果是同一种类型。

* 当外层表达式和内层表达式的结果一致时，执行对应的结构体。

* 同一个case后面可以有多个内层表达式，用,分隔。这表示当后面的任意表达式满足时，执行对应的内层执行体。

  > 上面的内层表达式3和内层表达式4，他们任意一个满足时，都会执行内层执行体3

* 外层表达式可以忽略（等于switch true{...}），此时内层表达式只能为布尔表达式。当内层布尔表达式为true时执行对应的执行体。

* 与Java不同，Go语言中的某个内层执行体执行完成后，默认不执行下一个内层执行体，如果需要执行下一个执行体，需要添加fallthrough关键字。

  > 上面的内层表达式1满足后，只会执行内层执行体1.
  >
  > 内层表达式2满足后，会执行内层执行体2和内层执行体3.

* default下的默认执行体，在上面的所有case条件都不成立的时候执行。

  >  default可以省略。

* Go语言的switch语句支持在switch关键字后执行初始化语句。



**示例：简单switch语句**

```go
// 基本switch语句
grade := "B"
switch grade {
case "A":
    fmt.Println("优秀")
case "B":
    fmt.Println("良好")
case "C":
    fmt.Println("一般")
default:
    fmt.Println("不及格")
}
```



**示例：表达式switch**

```go
// case后面可以跟表达式
x := 5
switch {
case x < 0:
    fmt.Println("x是负数")
case x > 0 && x < 10:
    fmt.Println("x是个位正数")
case x >= 10:
    fmt.Println("x是两位或以上的正数")
}

// switch后面可以跟表达式
x := 10
y := 0
switch x + y {
case 0:
  fmt.Println("0")
case 10:
  fmt.Println("10")
default:
  fmt.Println("default")
}

// switch后面甚至可以初始化变量
x := 10
y := 0
switch r := x + y; r >= 0 {
case true:
  fmt.Println("true")
case false:
  fmt.Println("false")
default:
  fmt.Println("default")
}
```



**示例：多条件匹配**

```go
// 一个case可以匹配多个值
ch := 'a'
switch ch {
case 'a', 'e', 'i', 'o', 'u':
    fmt.Println("元音字母")
case 'b', 'c', 'd', 'f', 'g':
    fmt.Println("辅音字母")
default:
    fmt.Println("其他字符")
}
```



**示例：fallthrough关键字**

```go
// fallthrough会执行下一个case，即使条件不满足
x := 10
switch x {
case 10:
    fmt.Println("等于10")
    fallthrough  // 继续执行下一个case
case 20:
    fmt.Println("等于20或执行了fallthrough")
case 30:
    fmt.Println("等于30或执行了fallthrough")
default:
    fmt.Println("其他值")
}
// 输出：
// 等于10
// 等于20或执行了fallthrough
```

### 1.3. 类型switch

switch 语句还可以被用于 type-switch 来判断某个 interface 变量中实际存储的变量类型。

> typw-switch语句算是switch语句的一个变种

语法格式：

```go
switch 接口变量.(type){
    case 类型1:
        // 执行体1
    case 类型2:
        // 执行体2
    default:
        // 默认执行体
}
```



示例：

```go
// 类型switch用于判断接口变量的具体类型
func processValue(v interface{}) {
    switch val := v.(type) {
    case int:
        fmt.Printf("整数: %d\n", val)
    case string:
        fmt.Printf("字符串: %s\n", val)
    case bool:
        fmt.Printf("布尔值: %t\n", val)
    case nil:
        fmt.Println("空值")
    default:
        fmt.Printf("未知类型: %T\n", val)
    }
}

func main() {
    processValue(42)
    processValue("Hello")
    processValue(true)
    processValue(nil)
}
```

### 1.4. select语句

select 是Go中的一个控制结构，类似于用于通信的switch语句。每个case必须是一个通信操作，要么是发送要么是接收。 select 随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。一个默认的子句应该总是可运行的。

语法格式：

```go
select {
    case communication clause  :
       // statement(s)      
    case communication clause  :
       // statement(s)
    /* 你可以定义任意数量的 case */
    default : /* 可选 */
       statement(s)
}
```

* 每个case都必须是一个通信操作.
* 如果任意某个通信操作可以进行，它就执行；其他被忽略。
* 如果有多个case都可以运行，Select会随机公平地选出一个执行。其他不会执行。
* 当没有任何通信操作可以执行时，如果有default子句，则执行该语句。如果没有default字句，select将阻塞，直到某个通信可以运行。

>  在一个select语句中，Go会按顺序从头到尾评估每一个发送和接收的语句。
>
> 如果其中的任意一个语句可以继续执行（即没有被阻塞），那么就从那些可以执行的语句中任意选择一条来使用。 如果没有任意一条语句可以执行（即所有的通道都被阻塞），那么有两种可能的情况：
>
>  ①如果给出了default语句，那么就会执行default的流程，同时程序的执行会从select语句后的语句中恢复。
>
>  ②如果没有default语句，那么select语句将被阻塞，直到至少有一个case可以进行下去。

示例：

```go
// 基本select语句
ch1 := make(chan string)
ch2 := make(chan string)

go func() {
    time.Sleep(1 * time.Second)
    ch1 <- "来自通道1的消息"
}()

go func() {
    time.Sleep(2 * time.Second)
    ch2 <- "来自通道2的消息"
}()

// select会选择第一个准备好的case执行
select {
case msg1 := <-ch1:
    fmt.Println("收到:", msg1)
case msg2 := <-ch2:
    fmt.Println("收到:", msg2)
case <-time.After(3 * time.Second):
    fmt.Println("超时")
}
```

## 2. 循环语句

### 2.1. for语句

for循环是一个循环控制结构，可以循环循环执行指定次数或不限次数。

语法格式如下：

```go
for 初始化子句;条件子句;后置子句{
    // 执行体
}
```

执行流程：

1. 执行初始化子句
2. 判断条件子句结果，如果为true继续执行，否则退出循环。
3. 执行执行体
4. 执行后置子句
5. 自动跳转到第2步。



注意事项：

- 初始化子句、条件子句、后置子句都可以为空。当初始化子句和后置子句都为空时，可以省略两个分号。
  - 初始化子句为空表示不执行初始化操作。
  - 后置子句为空表示不执行后置操作。
  - 条件子句为空默认为true。
- for循环可以嵌套使用。



**示例：基本for循环**

```go
// 传统的for循环
for i := 0; i < 5; i++ {
    fmt.Printf("循环次数: %d\n", i)
}
```



**示例：while循环形式**

```go
// Go语言没有while关键字，但for可以实现相同效果
i := 0
for i < 5 {
    fmt.Printf("循环次数: %d\n", i)
    i++
}
```



**示例：无限循环**

```go
// 无限循环
for {
    fmt.Println("无限循环")
    // 需要break或其他方式跳出循环
    break  // 或者return、goto等
}
```

### 2.2. for-range循环

range关键字用于遍历数组、切片、映射、字符串和通道：

* 遍历数组/切片

  ```go
  // 遍历数组或切片
  numbers := []int{1, 2, 3, 4, 5}
  for index, value := range numbers {
      fmt.Printf("索引%d: 值%d\n", index, value)
  }
  
  // 只获取索引
  for i := range numbers {
      fmt.Printf("索引: %d\n", i)
  }
  
  // 只获取值
  for _, value := range numbers {
      fmt.Printf("值: %d\n", value)
  }
  ```

* 遍历映射

  ```go
  // 遍历映射
  scores := map[string]int{
      "数学": 95,
      "语文": 87,
      "英语": 92,
  }
  
  for subject, score := range scores {
      fmt.Printf("%s: %d分\n", subject, score)
  }
  ```

* 遍历字符串

  ```go
  s := "Hello World"
  for i, c := range s {
    fmt.Printf("%d: %c\n", i, c)
  }
  for i := range s {
    fmt.Printf("%d: %c\n", i, s[i])
  }
  ```

* 遍历通道

  ```go
  // 定义通道
  c := make(chan int)
  
  // 启动一个单独的协程
  go func() {
  // 重复十次
  for i := 0; i < 10; i++ {
    // 睡眠1秒
    time.Sleep(time.Second)
    // 往通道写入数据
    c <- i
  }
  // 关闭通道
  close(c)
  }()
  
  // 遍历读取通道数据
  for i := range c {
  fmt.Println(i)
  }
  ```

  

## 3. 跳转语句

### 3.1. break

break关键字用于跳出当前循环语句或者switch语句。

#### 3.1.1. 循环中使用

break关键字在循环体中使用，表示不执行后续操作，直接退出当前循环体。

```go
循环体{
  // 前面的执行语句
  break
  // 后面的执行语句
}
```

- break关键字执行后，关键字后面的执行语句不会被执行。
- break关键字执行后，退出当前循环。
- 如果break关键字在多重循环内，将退出的是break关键字所在的最内层循环。



示例：

```go
func main() {
  a := []int{1, 2, 3, 4, 5}
  for i := 0; i < 5; i++ {
      for j, v := range a {
          fmt.Print(v)
          if j == i {
              break
          }
      }
      fmt.Println()
  }
}
// 结果:
1
12   
123  
1234 
12345
```



break关键字还可以搭配标签使用，表示退出标签后面紧跟的循环。

语法格式：

```go
标签名字:    
循环1 {
    // 前置语句1
    循环2 {
        // 前置语句2
        break 标签名字
        // 后置语句2
    }
    // 后置语句1
}
```

- 标签后面应该紧跟循环，不能再有其他语句。

- break关键字执行后，关键字后面的执行语句不会被执行。

  > 如后置语句2、后置语句1

- break关键字执行后，退出标签后面紧跟的循环。



#### 3.1.2. switch中使用

break关键字在switch中使用，表示不执行后续操作，直接退出当前switch体。

语法格式：

```go
switch语句{
  case子句{
      // 前面的执行语句
      break
      // 后面的执行语句
  }
}
```

- break关键字执行后，关键字后面的执行语句不会被执行。
- break关键字执行后，退出当前switch。
- 如果break关键字在多重switch内，将退出的是break关键字所在的最内层switch。

示例：

```go
func main() {
  var a = 1
  var b = 2
  switch a {
  case 1:
      fmt.Println(11)
      switch b {
      case 2:
          fmt.Println(21)
          if b > a {
              break
          }
          fmt.Println(22)
      }
      fmt.Println(12)
  }
}
// 结果
11
21
12
```



break关键字还可以搭配标签使用，表示退出标签后面紧跟的switch。

```go
func main() {
  var a = 1
  var b = 2
label:
  switch a {
  case 1:
      fmt.Println(11)
      switch b {
      case 2:
          fmt.Println(21)
          if b > a {
              break label
          }
          fmt.Println(22)
      }
      fmt.Println(12)
  }
}
// 结果
11
21
```

- 标签后面应该紧跟switch，不能再有其他语句。
- break关键字执行后，关键字后面的执行语句不会被执行。
- break关键字执行后，退出标签后面紧跟的switch。

### 3.2. continue语句

continue关键字用于停止当前循环体的执行，转而执行下一次循环。

语法格式：

```go
for 初始化子句;条件子句;后置子句{
  // 执行体1
  continue
  // 执行体2
}
```

- continue关键字只能在循环中使用。

- continue关键字执行后，关键字后面的执行体不会执行。

  > - 执行体2 不会被执行。

- 如果continue关键字在多重循环体内，将作用在关键字所在的最内层循环。



示例：

```go
func main() {
  a := []int{1, 2, 3}
  for i := range a {
      fmt.Print(i, "->")
      if i == 1 {
          fmt.Println("XXX")
          continue
      }
      fmt.Println(a[i])
  }
}
// 结果
0->1  
1->XXX
2->3  
```



continue关键字还可以搭配标签使用，表示作用于标签后面紧跟的循环。

语法格式：

```go
标签名字:    
  循环1 {
      // 前置语句1
      循环2 {
          // 前置语句2
          continue 标签名字
          // 后置语句2
      }
      // 后置语句1
  }
```

- 标签后面应该紧跟循环，不能再有其他语句。

- continue关键字执行后，关键字后面的执行语句不会被执行。

  > 如后置语句2、后置语句1

-  continue关键字执行后，继续执行标签后面紧跟的循环。



### 3.3. goto语句

Go语言保留了goto语句，但建议谨慎使用。

goto必须搭配标签使用，表示跳转到标签所在的位置，并开始执行标签后的语句。



示例：

```go
func main() {
    a := []int{1, 2, 3}
    count := 0
tag:
    for i := 0; i < len(a); i++ {
        fmt.Print(a[i])
    }
    fmt.Println()
    count++
    if count < 3 {
        goto tag
    }
}
// 结果
123
123
123
```
