Go 语言拥有一些不需要进行导入操作就可以使用的内置函数，本文将详细梳理Golang中各个内置函数的作用和使用方式。



Go语言提供了丰富的内置函数，这些函数为我们提供了对语言基本类型和结构的底层操作能力：

1. **内存管理函数**：new、make用于不同类型内存分配
2. **容器操作函数**：len、cap、append、copy、delete用于操作各种容器类型
3. **并发控制函数**：close用于通道操作
4. **错误处理函数**：panic、recover用于异常处理
5. **调试函数**：print、println用于简单输出（不推荐在生产环境使用）
6. **复数函数**：complex、real、imag用于复数操作

### 1. 内存管理函数

内存管理函数用于分配和初始化内存。

#### 1.1. new

```go
func new(Type) *Type
```

new内置函数用于分配内存。第一个参数是类型，而不是值，返回的值是指向该类型新分配的零值的指针。

使用示例：

```go
package main

import "fmt"

func main() {
    // 使用new分配内存
    p := new(int)  // *int类型，指向int的零值0
    fmt.Println(*p) // 输出: 0
    
    // 修改值
    *p = 5
    fmt.Println(*p) // 输出: 5
    
    // 对比直接声明变量
    var i int      // int类型，值为0
    fmt.Println(i) // 输出: 0
    
    // 对比make函数（用于slice, map, channel）
    s := make([]int, 5) // 创建长度为5的slice
    fmt.Println(s)      // 输出: [0 0 0 0 0]
    
    // 使用new创建结构体
    type Person struct {
        Name string
        Age  int
    }
    
    personPtr := new(Person)
    fmt.Printf("Name: %s, Age: %d\n", personPtr.Name, personPtr.Age) // 输出: Name: , Age: 0
}
```

注意事项：

* new返回的是指针类型
* new分配的内存会被初始化为类型的零值
* new与make不同，new用于值类型和结构体，make用于slice、map和channel

#### 1.2. make

```go
func make(t Type, size ...IntegerType) Type
```

make 内置函数分配并初始化类型为 slice、map 或 chan的对象。与 new 一样，第一个参数是类型，而不是值。与 new 不同，make 的返回类型与其参数的类型相同，而不是指向它的指针。结果的规格取决于类型。

* slice

参数size用于指定长度，切片的容量等于其长度。可以提供第二个整数参数来指定不同的容量（容量必须不小于长度）。

```go
make([]数据类型,切片长度,切片容量)
```

例如，`make([]int, 0, 10) `分配大小为 10 的基础数组，并返回由该基础数组支持的长度为 0、容量为 10 的切片。

*  Map

为空映射分配足够的空间来容纳指定数量的元素。该大小可以省略，在这种情况下会分配较小的起始大小。

```
make(map[键类型]值类型,初始容量)
```

* Channel

通道的缓冲区使用指定的缓冲区容量进行初始化。如果为零或省略大小，则通道是无缓冲的。

```go
make(chan 数据类型,缓冲区容量)
```

使用示例：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // 创建切片
    slice1 := make([]int, 5)        // 长度5，容量5
    slice2 := make([]int, 3, 10)    // 长度3，容量10
    fmt.Printf("slice1: len=%d cap=%d %v\n", len(slice1), cap(slice1), slice1)
    fmt.Printf("slice2: len=%d cap=%d %v\n", len(slice2), cap(slice2), slice2)
    
    // 创建映射
    map1 := make(map[string]int)           // 空映射
    map2 := make(map[string]int, 100)      // 初始容量100
    map1["key"] = 1
    map2["key"] = 2
    fmt.Printf("map1: %v\n", map1)
    fmt.Printf("map2: %v\n", map2)
    
    // 创建通道
    ch1 := make(chan int)        // 无缓冲通道
    ch2 := make(chan int, 5)     // 缓冲通道，容量5
    
    // 启动goroutine向缓冲通道发送数据
    go func() {
        for i := 1; i <= 3; i++ {
            ch2 <- i
            fmt.Printf("发送: %d\n", i)
        }
        close(ch2)
    }()
    
    // 从通道接收数据
    time.Sleep(time.Millisecond) // 等待goroutine执行
    for value := range ch2 {
        fmt.Printf("接收: %d\n", value)
    }
}
```

注意事项：

* make只能用于slice、map和channel
* make返回的是值类型，不是指针
* make会初始化元素为零值
* 对于slice，可以分别指定长度和容量

### 2. 容器操作函数

容器操作函数用于操作各种容器类型，如数组、切片、映射等。

#### 2.1. len

```go
func len(v Type) int
```

len 内置函数根据参数 v 的类型返回 v 的长度：

* 数组：返回数组的长度。
* 数组指针：返回指针所指向的数组的长度。
* 切片Slice：返回切片中元素的个数，如果切片为nil，返回0。
* 映射Map：返回映射中元素的个数，如果映射为nil，返回0。
* 字符串：返回字符串中字节的个数。
* 通道Channel：返回通道缓冲区中排队（未读）的元素数量；如果通道为nil，返回0。

使用示例：

```go
package main

import "fmt"

func main() {
    // 数组的len
    arr := [5]int{1, 2, 3, 4, 5}
    fmt.Printf("数组 len: %d\n", len(arr)) // 输出: 5
    
    // 切片的len
    slice1 := make([]int, 3, 10) // 长度为3，容量为10
    fmt.Printf("切片1 len: %d\n", len(slice1)) // 输出: 3
    
    slice2 := arr[1:3] // 从数组创建切片
    fmt.Printf("切片2 len: %d\n", len(slice2)) // 输出: 2
    
    // nil切片的len
    var nilSlice []int
    fmt.Printf("nil切片 len: %d\n", len(nilSlice)) // 输出: 0
    
    // 映射的len
    m := make(map[string]int)
    m["apple"] = 5
    m["banana"] = 3
    fmt.Printf("映射 len: %d\n", len(m)) // 输出: 2
    
    // nil映射的len
    var nilMap map[string]int
    fmt.Printf("nil映射 len: %d\n", len(nilMap)) // 输出: 0
    
    // 字符串的len
    str := "Hello世界"
    fmt.Printf("字符串 len: %d\n", len(str)) // 输出: 11 (英文1字节，中文3字节)
    
    // 通道的len
    ch := make(chan int, 5)
    ch <- 1
    ch <- 2
    fmt.Printf("通道 len: %d\n", len(ch)) // 输出: 2
    
    // nil通道的len
    var nilCh chan int
    fmt.Printf("nil通道 len: %d\n", len(nilCh)) // 输出: 0
}
```

注意事项：

* len返回的是当前元素的数量，而不是容量
* 对于字符串，len返回的是字节数，而不是字符数
* len操作的时间复杂度是O(1)

#### 2.2. cap

```go
func cap(v Type) int
```

cap 内置函数根据参数 v 的类型返回 v 的容量：

* 数组：返回数组的长度（与 len(v) 相同）。

* 数组指针：返回指针所指向的数组的长度（与 len(v) 相同）。

* Slice：返回重新切片时切片所能达到的最大长度；如果切片为nil，返回0。

* Channel：返回通道缓冲区容量，以元素为单位；如果通道为nil，返回0。

使用示例：

```go
package main

import "fmt"

func main() {
    // 数组的cap
    arr := [5]int{1, 2, 3, 4, 5}
    fmt.Printf("数组 cap: %d\n", cap(arr)) // 输出: 5
    
    // 切片的cap
    slice1 := make([]int, 3, 10) // 长度为3，容量为10
    fmt.Printf("切片1 cap: %d\n", cap(slice1)) // 输出: 10
    
    slice2 := arr[1:3] // 从数组创建切片
    fmt.Printf("切片2 cap: %d\n", cap(slice2)) // 输出: 4 (从索引1到数组末尾)
    
    // nil切片的cap
    var nilSlice []int
    fmt.Printf("nil切片 cap: %d\n", cap(nilSlice)) // 输出: 0
    
    // 通道的cap
    ch1 := make(chan int, 5) // 缓冲通道，容量为5
    fmt.Printf("缓冲通道 cap: %d\n", cap(ch1)) // 输出: 5
    
    ch2 := make(chan int) // 无缓冲通道
    fmt.Printf("无缓冲通道 cap: %d\n", cap(ch2)) // 输出: 0
    
    // nil通道的cap
    var nilCh chan int
    fmt.Printf("nil通道 cap: %d\n", cap(nilCh)) // 输出: 0
}
```

注意事项：

* 对于切片，cap表示底层数组的容量
* 切片的长度可以小于或等于容量
* 通过cap可以了解切片在不重新分配内存的情况下可以增长到多大

#### 2.3. append

```go
func append(slice []Type, elems ...Type) []Type
```

append 内置函数将元素追加到切片的末尾。

* 如果目的地有足够的容量，则会对目的地进行重新切片以容纳新元素。
* 如果没有，将分配一个新的底层数组。追加返回更新后的切片。因此，有必要存储追加的结果，通常存储在保存切片本身的变量中。
* 作为一种特殊情况，可以合法的将字符串附加到字节切片

使用示例：

```go
package main

import "fmt"

func main() {
    // 追加单个元素
    slice := make([]int, 0, 5)
    fmt.Printf("初始切片: len=%d cap=%d %v\n", len(slice), cap(slice), slice)
    
    slice = append(slice, 1)
    fmt.Printf("追加1后: len=%d cap=%d %v\n", len(slice), cap(slice), slice)
    
    // 追加多个元素
    slice = append(slice, 2, 3, 4)
    fmt.Printf("追加2,3,4后: len=%d cap=%d %v\n", len(slice), cap(slice), slice)
    
    // 追加另一个切片
    more := []int{5, 6, 7}
    slice = append(slice, more...)
    fmt.Printf("追加切片后: len=%d cap=%d %v\n", len(slice), cap(slice), slice)
    
    // 追加字符串到字节切片
    byteSlice := []byte("Hello")
    byteSlice = append(byteSlice, " World"...)
    fmt.Printf("追加字符串后: %s\n", byteSlice)
    
    // 容量增长示例
    s := make([]int, 0)
    for i := 1; i <= 10; i++ {
        s = append(s, i)
        fmt.Printf("长度: %d 容量: %d %v\n", len(s), cap(s), s)
    }
}
```

注意事项

* append返回一个新的切片，必须保存返回值
* 当容量不足时，append会创建新的底层数组
* 切片容量增长策略：
  * 小容量时翻倍增长：确保小切片能快速适应数据增长
  * 大容量时渐进增长：避免大容量切片过度分配内存，节省内存空间
  * 非严格固定倍数：实际扩容还会考虑内存对齐等因素，所以不是严格的数学倍数关系

* 使用`...`操作符可以展开切片作为参数传递

#### 2.4. copy

```go
func copy(dst, src []Type) int
```

copy 内置函数将源切片中的元素复制到目标切片中。（作为一种特殊情况，它还将字节从字符串复制到字节切片。）源和目标可能重叠。Copy 返回复制的元素数量，该数量将是 len(src) 和 len(dst) 中的最小值。

* dst 参数用于指定目标切片，其类型只能为切片，不能为数组/字符串。
* src 参数用于指定源切片，其类型只能为切片或字符串。
* dst 与 src 的类型必须相同。当src为字符串时，dst必须为[]byte。

使用示例：

```go
package main

import "fmt"

func main() {
    // 基本复制
    src := []int{1, 2, 3, 4, 5}
    dst := make([]int, 5)
    n := copy(dst, src)
    fmt.Printf("复制了 %d 个元素: %v\n", n, dst)
    
    // 目标切片较短
    src1 := []int{1, 2, 3, 4, 5}
    dst1 := make([]int, 3)
    n1 := copy(dst1, src1)
    fmt.Printf("复制了 %d 个元素: %v\n", n1, dst1) // 只复制前3个元素
    
    // 源切片较短
    src2 := []int{1, 2, 3}
    dst2 := make([]int, 5)
    n2 := copy(dst2, src2)
    fmt.Printf("复制了 %d 个元素: %v\n", n2, dst2) // 复制3个元素，其余为零值
    
    // 字符串复制到字节切片
    str := "Hello, 世界"
    byteSlice := make([]byte, len(str))
    n3 := copy(byteSlice, str)
    fmt.Printf("复制了 %d 个字节: %s\n", n3, byteSlice)
    
    // 重叠切片复制
    slice := []int{1, 2, 3, 4, 5}
    n4 := copy(slice[1:], slice) // 向右复制
    fmt.Printf("重叠复制后: %v\n", slice)
}
```

注意事项：

* copy返回实际复制的元素数量
* 复制数量是源和目标切片长度的最小值
* copy可以处理重叠的切片
* 当源是字符串时，目标必须是[]byte类型

#### 2.5. delete

```go
func delete(m map[Type]Type1, key Type)
```

delete 内置函数用于从映射中删除具有指定键 (m[key]) 的元素:

- 如果 m 为 nil 或者不存在这样的元素，则删除是无操作。

使用示例：

```go
package main

import "fmt"

func main() {
    // 创建一个映射
    m := make(map[string]int)
    m["apple"] = 5
    m["banana"] = 3
    m["orange"] = 7
    
    fmt.Println("删除前:", m)
    
    // 删除键为"banana"的元素
    delete(m, "banana")
    fmt.Println("删除后:", m)
    
    // 删除不存在的键（无操作）
    delete(m, "grape")
    fmt.Println("删除不存在的键:", m)
    
    // 对nil映射执行删除操作（无操作）
    var nilMap map[string]int
    delete(nilMap, "apple") // 不会产生错误
}
```

注意事项：

* delete操作是幂等的，多次删除同一个键不会产生错误
* 对nil映射执行delete操作是安全的，不会产生恐慌
* delete函数不返回任何值，无法确认删除操作是否实际删除了元素

### 3. 并发控制函数

并发控制函数主要用于goroutine和channel的操作。

#### 3.1. close

```go
func close(c chan<- Type)
```

close 内置函数用于关闭通道:

* 它只能由发送方执行（该通道必须是双向的或仅发送的），而不能由接收方执行，并且具有在收到最后发送的值后关闭通道的效果。
* 从关闭的通道 c 接收到最后一个值后，来自 c 的任何接收都将成功而不会阻塞，并返回通道元素的零值。
* 对于关闭的通道，形式 x, ok := <-c 也会将 ok 设置为 false。

使用示例：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int, 3)
    
    // 发送数据到通道
    ch <- 1
    ch <- 2
    ch <- 3
    
    // 关闭通道
    close(ch)
    
    // 从已关闭的通道接收数据
    for i := 0; i < 5; i++ {
        if value, ok := <-ch; ok {
            fmt.Println("接收到:", value)
        } else {
            fmt.Println("通道已关闭")
            break
        }
    }
}
```

注意事项：

* 对已关闭的通道执行关闭操作会产生恐慌（panic）
* 对已关闭的通道发送数据会产生恐慌（panic）
* 接收操作在通道关闭后仍然可以执行，但只会返回零值

### 4. 错误处理函数

错误处理函数用于处理程序中的异常情况。

#### 4.1. panic

```go
func panic(v any)
```

内置函数panic会停止当前goroutine的正常执行:

* 当函数 F 调用恐慌时，F 的正常执行立即停止。任何被 F 推迟执行的函数都会以正常的方式运行，然后 F 返回到它的调用者。
* 对于调用者 G 来说，F 的调用就像调用恐慌一样，终止 G 的执行并运行任何延迟的函数。这将继续下去，直到执行中的 goroutine 中的所有函数都以相反的顺序停止。此时，程序将以非零退出代码终止。这个终止序列称为恐慌，可以通过内置函数recover来控制。

使用示例：

```go
package main

import "fmt"

func main() {
    defer fmt.Println("main函数中的defer")
    
    fmt.Println("开始执行main函数")
    foo()
    fmt.Println("main函数执行结束") // 这行不会被执行
}

func foo() {
    defer fmt.Println("foo函数中的defer")
    
    fmt.Println("开始执行foo函数")
    panic("发生了一个错误")
    fmt.Println("foo函数执行结束") // 这行不会被执行
}
```

注意事项：

* panic应该只在真正严重错误时使用
* 在大多数情况下，应该优先使用错误返回机制而不是panic
* panic会终止当前goroutine的执行

#### 4.2. recover

```go
func recover() any
```

recover 内置函数允许程序管理恐慌 goroutine 的行为:

* 在延迟函数（但不是它调用的任何函数）内执行恢复调用可以通过恢复正常执行来停止恐慌序列，并检索传递给恐慌调用的错误值
* 如果在延迟函数之外调用恢复，它不会停止恐慌序列。在这种情况下，或者当 goroutine 没有恐慌时，或者如果提供给恐慌的参数为 nil，recover 将返回 nil。因此，recover 的返回值报告 goroutine 是否处于恐慌状态。

使用示例：

```go
package main

import "fmt"

func main() {
    fmt.Println("程序开始")
    
    // 使用recover捕获panic
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("捕获到panic:", r)
        }
    }()
    
    fmt.Println("执行一些操作")
    panic("出现错误")
    fmt.Println("这行不会被执行") // 这行不会被执行
}

func divide(a, b int) (result int) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("捕获到除零错误:", r)
            result = 0
        }
    }()
    
    result = a / b
    return result
}
```

注意事项：

* recover只能在defer函数中使用才能生效
* recover返回panic传递的值
* 如果没有panic发生，recover返回nil

### 5. 调试辅助函数

调试辅助函数主要用于程序调试阶段的简单输出。

#### 5.1. print / println

```go
func print(args ...Type)
func println(args ...Type)
```

print 内置函数以特定于实现的方式格式化其参数并将结果写入标准错误。print对于引导和调试很有用；不能保证它会保留在该语言中。

println 内置函数以特定于实现的方式格式化其参数并将结果写入标准错误。参数之间始终添加空格并附加换行符。println对于引导和调试很有用；不能保证它会保留在该语言中。

使用示例：

```go
package main

func main() {
    // 使用print
    print("Hello ")
    print("World!")
    print(42)
    
    // 使用println
    println() // 输出空行
    println("Hello World!")
    println("Number:", 42)
    println("Boolean:", true)
    println("String:", "Go语言")
}
```

注意事项：

* print和println函数主要用于调试，不应该在生产代码中使用
* 这些函数的输出格式是特定于实现的
* 官方不保证这些函数会在未来版本中保留

### 6. 复数处理函数

复数处理函数用于处理复数类型的数据。

#### 6.1. complex

```go
func complex(r, i FloatType) ComplexType
```

complex 内置函数根据两个浮点值构造一个复数值。实部和虚部必须具有相同的大小，可以是 float32 或 float64（或可分配给它们），并且返回值将是相应的复数类型（float32 为complex64，float64 为complex128）。

使用示例：

```go
package main

import (
    "fmt"
    "math/cmplx"
)

func main() {
    // 创建复数
    c1 := complex(float32(3.0), float32(4.0)) // complex64
    c2 := complex(3.0, 4.0)                   // complex128
    
    fmt.Printf("c1: %v, 类型: %T\n", c1, c1)
    fmt.Printf("c2: %v, 类型: %T\n", c2, c2)
    
    // 复数运算
    sum := c1 + c2
    fmt.Printf("和: %v\n", sum)
    
    // 使用复数函数
    fmt.Printf("实部: %v\n", real(c2))
    fmt.Printf("虚部: %v\n", imag(c2))
    fmt.Printf("模长: %v\n", cmplx.Abs(c2))
    fmt.Printf("共轭: %v\n", cmplx.Conj(c2))
}
```

注意事项：

* 实部和虚部必须是相同类型的浮点数
* complex64由两个float32组成
* complex128由两个float64组成

#### 6.2. real

```go
func real(c ComplexType) FloatType
```

real 内置函数返回复数 c 的实部。返回值将是与 c 类型对应的浮点类型。

使用示例：

```go
package main

import "fmt"

func main() {
    c1 := 3 + 4i         // complex128
    c2 := complex64(1 + 2i) // complex64
    
    fmt.Printf("c1的实部: %v\n", real(c1)) // 3
    fmt.Printf("c2的实部: %v\n", real(c2)) // 1
    
    // 实部类型与复数类型匹配
    fmt.Printf("c1实部类型: %T\n", real(c1)) // float64
    fmt.Printf("c2实部类型: %T\n", real(c2)) // float32
    
    // 使用实部和虚部重建复数
    realPart := real(c1)
    imagPart := imag(c1)
    reconstructed := complex(realPart, imagPart)
    fmt.Printf("重建的复数: %v\n", reconstructed)
}
```

#### 6.3. imag

```go
func imag(c ComplexType) FloatType
```

imag 内置函数返回复数 c 的虚部。返回值将是与 c 类型对应的浮点类型。

使用示例：

```go
package main

import "fmt"

func main() {
    c1 := 3 + 4i         // complex128
    c2 := complex64(1 + 2i) // complex64
    
    fmt.Printf("c1的虚部: %v\n", imag(c1)) // 4
    fmt.Printf("c2的虚部: %v\n", imag(c2)) // 2
    
    // 虚部类型与复数类型匹配
    fmt.Printf("c1虚部类型: %T\n", imag(c1)) // float64
    fmt.Printf("c2虚部类型: %T\n", imag(c2)) // float32
}
```
