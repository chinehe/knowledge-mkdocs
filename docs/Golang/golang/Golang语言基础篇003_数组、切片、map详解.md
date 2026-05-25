数组、切片和映射（map）是Go语言中最常用的三种数据结构，它们各自有不同的特点和适用场景。掌握这些数据结构的使用方法对于编写高效的Go程序至关重要。

## 1. 数组(Array)

### 11. 数组的概念和特性

数组是Go语言中的一种基本数据结构，它是**固定长度**的**同类型**元素序列。数组的特点包括：

- **固定长度**：数组一旦声明，其长度就不能改变。

  > 数组长度是数组类型的一部分，因此`var a [10]int`与`var b [5] int`是不同的数据类型。
- **同质性**：数组中的所有元素必须是相同类型。
- **连续内存**：数组元素在内存中连续存储。
- **值类型**：数组是值类型，赋值和传参时会复制整个数组。因此改变数组副本的值，不会影响原数组。
- **可比较**：支持 “==”、”!=” 操作符，因为内存总是被初始化过的，这里比较的是数组的内容。

### 1.2. 数组的声明和初始化

在Go语言中，数组有多种声明和初始化方式：

#### 1.2.1. 声明数组

声明数组的方式如下：

```go
// 声明一个长度为5的整数数组（零值初始化）
var arr1 [5]int

// 声明并初始化数组
var arr2 [3]string = [3]string{"apple", "banana", "orange"}

// 使用短变量声明
arr3 := [5]int{1, 2, 3, 4, 5}

// 让编译器自动计算数组长度
arr4 := [...]int{1, 2, 3, 4, 5}  // 长度为5

// 指定索引初始化
arr5 := [5]int{0: 10, 4: 50}  // 索引0为10，索引4为50，其余为0
```

**注意**：声明数组是，数组长度必须是常量，使用变量会报错。

```go
length := 3 // length一个变量
c := [length]int{1, 2, 3} // 报错：declared and not used: length ； invalid array length length

const length = 3 // length是一个常量
c := [length]int{1, 2, 3} // 正确
```



#### 1.2.2. 多维数组

多维数组的基本使用方式如下：

```go
// 声明二维数组
var matrix [3][3]int

// 初始化二维数组
matrix2 := [2][3]int{
    {1, 2, 3},
    {4, 5, 6},
}

// 使用索引号初始化元素
var arr2 = [...][2]int{
    0: {1: 2},
    1: {1: 4},
}

// 访问二维数组元素
value := matrix2[0][1]  // 第一行第二列的值
```

需要特别注意的是，多维数组声明时，只有第一个维度支持编译器自动计算长度：

```go
// 第一个维度支持编译器自动计算长度
matrix2 := [...][3]int{
    {1, 2, 3},
    {4, 5, 6},
}

// 其他维度不支持编译器自动计算长度
matrix2 := [2][...]int{
    {1, 2, 3},
    {4, 5, 6},
} // 报错：invalid use of [...] array (outside a composite literal)
```



### 1.3. 数组的操作

#### 1.3.1. 访问和修改数组元素

```go
arr := [5]int{1, 2, 3, 4, 5}

// 访问元素
first := arr[0]  // 获取第一个元素
last := arr[4]   // 获取最后一个元素

// 修改元素
arr[0] = 10  // 修改第一个元素

// 获取数组长度
length := len(arr)  // 返回5
```

#### 1.3.2. 数组遍历

```go
arr := [5]int{1, 2, 3, 4, 5}

// 使用索引遍历
for i := 0; i < len(arr); i++ {
    fmt.Printf("索引%d: %d\n", i, arr[i])
}

// 使用range遍历（推荐）
for index, value := range arr {
    fmt.Printf("索引%d: %d\n", index, value)
}

// 只获取值（忽略索引）
for _, value := range arr {
    fmt.Printf("值: %d\n", value)
}
```

#### 1.3.3. 数组比较和复制

```go
arr1 := [3]int{1, 2, 3}
arr2 := [3]int{1, 2, 3}
arr3 := [3]int{1, 2, 4}

// 数组可以直接比较（长度和类型必须相同）
fmt.Println(arr1 == arr2)  // true
fmt.Println(arr1 == arr3)  // false

// 数组赋值会复制整个数组
arr4 := arr1  // 复制arr1到arr4
arr4[0] = 10  // 修改arr4不会影响arr1
fmt.Println(arr1[0])  // 仍然是1
```

#### 1.3.4. 数组的长度

内置函数 len 和 cap 都返回数组长度 (元素数量)。

```go
// 声明并初始化数组
a := [...][2]int{
  {1, 2},
  {3, 4},
  {5, 6},
}
fmt.Println(len(a)) // 3
fmt.Println(len(a[0])) // 2
fmt.Println(cap(a)) // 3
fmt.Println(cap(a[0])) // 2
```

#### 1.3.5. 数组截取

使用`arr[m:n]`的语法，可以截取数组。

```go
// 声明一维数组
a := [6]int{1,2,3,4,5,6}
// 截取
fmt.Println(a[0:1]) // [1]
fmt.Println(a[3:]) // [4 5 6]
fmt.Println(a[:5]) // [1 2 3 4 5]
```

该方式同样适用于多维数组：

```go
// 声明多维数组
a := [...][2]int{
  {1, 2},
  {3, 4},
  {5, 6},
}
// 截取
fmt.Println(a[0:2][0:1]) // [[1 2]]
```

**特别注意：** 截取数组得到的结果，是一个切片Slice，而非一个数组：

```go
// 定义数组
a := [5]int{1, 2, 3, 4, 5}
// 数组截取
b := a[2:4]
fmt.Println(a) // [1 2 3 4 5]
fmt.Println(b) // [3 4]
fmt.Printf("%T\n", a) // [5]int
fmt.Printf("%T\n", b) // []int
```

> []int是一个切片Slice，而非一个数组。关于切片后面会说...

## 2. 切片(Slice)

由于数组的长度固定，这在很多场景下不够灵活。例如：

```go
// 这样的代码在实际开发中很少见
func processScores(scores [100]int) {
    // 只能处理恰好100个分数
}
```

为了解决这个问题，Go语言引入了切片（Slice）。

### 2.1. 切片的概念和特性

切片是对数组的封装，提供了动态数组的功能。切片的特点包括：

> slice 并不是数组或数组指针。它通过内部指针和相关属性引用数组片段，以实现变长方案。

- **动态长度**：可以根据需要增长或缩小
- **引用类型**：切片是引用类型，指向底层数组
- **灵活性**：支持追加、截取等操作



### 2.2. 切片的声明和初始化

#### 2.2.1. 声明切片

切片的基本声明方式如下：

```go
// 声明一个整数切片（零值为nil）
var slice1 []int

// 使用make创建切片(虽然元素仍为零值，但切片已经初始化，不为nil)
slice2 := make([]int, 5)      // 长度为5，容量为5
slice3 := make([]int, 3, 5)   // 长度为3，容量为5
```

与数组一样，切片也可以在声明时初始化：

```go
// 直接初始化
slice8 := []int{1, 2, 3, 4, 5}
```

#### 2.2.2. 通过截取来创建切片

可以通过截取数组的方式来初始化切片：

```go
// 定义数组
a := [5]int{1, 2, 3, 4, 5}
// 数组截取,以初始化切片
b := a[2:4]
fmt.Println(a) // [1 2 3 4 5]
fmt.Println(b) // [3 4]
fmt.Printf("%T\n", a) // [5]int
fmt.Printf("%T\n", b) // []int
```

切片也支持截取操作，可以通过截取切片的方式来初始化另一个切片：

```go
// 声明并初始化切片
a := []int{1, 2, 3, 4, 5}
// 切片截取,以初始化切片
b := a[2:4]
fmt.Println(a) // [1 2 3 4 5]
fmt.Println(b) // [3 4]
fmt.Printf("%T\n", a) // []int
fmt.Printf("%T\n", b) // []int
```

#### 2.2.3. 理解切片的内部结构

切片的底层是数组，切片包含三个字段：
- 指向底层数组的指针
- 长度（len）
- 容量（cap）

```go
slice := make([]int, 3, 5)
fmt.Println(len(slice))  // 3
fmt.Println(cap(slice))  // 5
```

>  由于长度和容量是切片的属性，因此获取长度和容量时不需要遍历底层数组，性能约等于访问一个普通变量。

### 2.3. 切片的操作

#### 2.3.1. append追加元素

append 内置函数将元素追加到切片的末尾:

* 如果目标切片有足够的容量，直接添加新元素。
* 如果目标切片没有足够的容量，会重新分配一个底层数组，然后返回新的切片。因此有必要保存append的结果（通常是保存到切片原本的变量中）。
* 作为一种特殊情况，将字符串append到字节切片是合法的。



追加单个元素：

```go
slice := []int{1, 2, 3}

// 追加单个元素
slice = append(slice, 4)

fmt.Println(slice)  // [1 2 3 4]
```

追加多个元素：

```go
slice := []int{1, 2, 3}

// 追加多个元素
slice = append(slice, 5, 6, 7)
fmt.Println(slice)  // [1 2 3 5 6 7]
```

还可以追加另一个切片的元素：

```go
slice := []int{1, 2, 3}

// 追加另一个切片
other := []int{8, 9, 10}
slice = append(slice, other...)

fmt.Println(slice)  // [1 2 3 8 9 10]
```



#### 2.3.2. 切片截取

与数组一样，使用`slice[m:n]`的语法，可以截取切片：

```go
// 声明一维切片
a := []int{1,2,3,4,5,6}
// 截取
fmt.Println(a[0:1]) // [1]
fmt.Println(a[3:]) // [4 5 6]
fmt.Println(a[:5]) // [1 2 3 4 5]
```

该方式同样适用于多维切片：

```go
// 声明多维切片
a := [][]int{
  {1, 2},
  {3, 4},
  {5, 6},
}
// 截取
fmt.Println(a[0:2][0:1]) // [[1 2]]
```

**特别注意：** 截取数组得到的结果，是一个切片Slice，而非一个数组：

```go
// 定义切片
a := []int{1, 2, 3, 4, 5}
// 切片截取
b := a[2:4]
fmt.Println(a) // [1 2 3 4 5]
fmt.Println(b) // [3 4]
fmt.Printf("%T\n", a) // []int
fmt.Printf("%T\n", b) // []int
```



另外，截取时还可以指定容量：

```go
// 语法格式
slice := array[low:high:capacity]
```

* low：起始索引（包含）
* high：结束索引（不包含）
* capacity：指定切片的容量

> 上述三个索引多不能超过原数组的长度。

新切片的长度与容量：

* 长度是 high - low
* 容量是 capacity - low

```go
slice := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

// 基本截取操作
part1 := slice[2:5]   // [3 4 5]
part2 := slice[:5]    // [1 2 3 4 5]
part3 := slice[5:]    // [6 7 8 9 10]
part4 := slice[:]     // 整个切片的副本

// 指定容量的截取
part5 := slice[2:5:7] // 长度为3，容量为5（7-2）
```



**特别注意：截取后得到的新切片，与原数组和切片共享底层数组内存，直到其中一个重新分配。**

```go
// 定义源切片
a := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
// 截取
b := a[:5]

// 从0索引开始截取，由于共享内存，因此两个切片起始地址相同
fmt.Printf("%p\n", a) // 0x14000016500
fmt.Printf("%p\n", b) // 0x14000016500

// 修改其中一个切片的元素，另一个切片的数组也会变（只针对共享内存区间的元素）
a[0] = 111
a[8] = 888
b[1] = 222
fmt.Println(a) // [111 222 3 4 5 6 7 8 888]
fmt.Println(b) // [111 222 3 4 5]

// 追加元素以使b切片重新分配
for i := 0; i < 10; i++ {
  b = append(b, i)
}

// 由于重新分配内存，b切片起始地址变更
fmt.Printf("%p\n", a) // 0x14000016500
fmt.Printf("%p\n", b) // 0x14000110000

// 再次修改元素，两者不再互相影响
a[0] = 100
b[1] = 200
fmt.Println(a) // [100 222 3 4 5 0 1 2 3]
fmt.Println(b) // [111 200 3 4 5 0 1 2 3 4 5 6 7 8 9]
```





#### 2.3.3. copy复制切片

内置函数copy可以用来复制切片：

```go
src := []int{1, 2, 3, 4, 5}
dst := make([]int, len(src))

// 使用copy函数复制
n := copy(dst, src)
fmt.Printf("复制了%d个元素\n", n)
fmt.Println(dst)  // [1 2 3 4 5]
```

> 使用内置函数copy，比手动循环copy更加高效。
>
> 这里就不贴测试性能的代码了，读者可以自己去测试！

#### 2.3.4. 切片遍历

切片的遍历方式与数组遍历一样：

* 使用索引遍历

```go
slice := []int{1, 2, 3, 4, 5}

// 使用索引遍历
for i := 0; i < len(slice); i++ {
    fmt.Printf("索引%d: %d\n", i, slice[i])
}
```

* 使用range遍历

```go
slice := []string{"apple", "banana", "orange"}

// 使用range遍历
for index, value := range slice {
    fmt.Printf("索引%d: %s\n", index, value)
}

// 只获取值
for _, value := range slice {
    fmt.Printf("水果: %s\n", value)
}
```

#### 2.3.5. 删除元素

Go语言并没有对删除切片元素提供专用的语法或者接口，需要使用切片本身的特性来删除元素。



* 可以使用截取操作来删除前置元素和后置元素。

```go
// 删除前置元素
slice := []int{1, 2, 3, 4, 5}
slice := slice[2:] // [3, 4, 5]

// 删除后置元素
slice := []int{1, 2, 3, 4, 5}
slice := slice[:2] // [1,2]
```

* 可以使用append内置函数+截取操作来删除元素

```go
// 删除前置元素(不常用)
a = []int{1, 2, 3,4,5}
a = append(a[:0], a[2:]...) // [3, 4, 5]

// 删除后置元素（不常用）
a = []int{1, 2, 3,4,5}
a = append(a[:0], a[:2]...) // [1,2]

// 删除中间元素（常用）
a = []int{1, 2, 3,4,5}
a = append(a[:2], a[3:]...) // [1,2,4,5]
```

* 可以使用copy内置函数+截取操作来删除元素

```go
// 删除前置元素
a = []int{1, 2, 3,4,5}
a = a[:copy(a, a[2:])]// [3, 4, 5]

// 删除后置元素
a = []int{1, 2, 3,4,5}
a = a[:copy(a, a[:2])] // [1,2]

// 删除中间元素
a = []int{1, 2, 3,4,5}
a = a[:2+copy(a[2:], a[3:])] // [1,2,4,5]
```



**最佳实践：**

* 使用截取操作来删除前置元素和后置元素。
* 使用append内置函数+截取操作来删除中间元素

### 2.4. 切片的扩容机制

当切片容量不足时，append操作会触发扩容：

```go
slice := make([]int, 0, 2)  // 长度0，容量2
slice = append(slice, 1)    // [1]
slice = append(slice, 2)    // [1 2]
slice = append(slice, 3)    // 触发扩容，容量变为4或更多
```

扩容规则：
- 当所需容量超过当前容量时，会创建新的底层数组
- 新容量通常是原容量的2倍（具体规则较为复杂）
- 扩容后会将原数据复制到新数组



使用go1.25.0版本，从容量0开始填充元素，触发扩容的时机及结果如下：

```go
func main() {
	slice := make([]int, 0, 0)
	fmt.Printf("len(%d) cap(%d)\n", len(slice), cap(slice))
	oldCap := cap(slice)
	for i := 0; i < 1000; i++ {
		slice = append(slice, i)
		if cap(slice) != oldCap {
			fmt.Printf("len(%d) cap(%d)\n", len(slice), cap(slice))
			oldCap = cap(slice)
		}
	}
}

// len(0) cap(0)
// len(1) cap(4)
// len(5) cap(8)
// len(9) cap(16)
// len(17) cap(32)
// len(33) cap(64)
// len(65) cap(128)
// len(129) cap(256)
// len(257) cap(512)
// len(513) cap(848)
// len(849) cap(1280)
```

### 2.5. 切片的陷阱和最佳实践

#### 2.5.1. 共享底层数组的问题

```go
original := []int{1, 2, 3, 4, 5}
sub := original[1:3]  // [2 3]

// 修改子切片会影响原切片
sub[0] = 20
fmt.Println(original)  // [1 20 3 4 5]
```

#### 2.5.2. 避免切片污染

```go
// 错误示例：可能导致意外的内存占用
func processLargeData() []byte {
    large := make([]byte, 1024*1024)  // 1MB数据
    // 处理数据...
    return large[100:200]  // 返回小切片，但仍引用整个1MB数组
}

// 正确做法：复制数据
func processLargeDataCorrectly() []byte {
    large := make([]byte, 1024*1024)
    // 处理数据...
    result := make([]byte, 100)
    copy(result, large[100:200])
    return result
}
```

## 3. 映射(Map)

### 3.1. 映射的概念和特性

映射是一种无序的key-value键值对数据结构，类似于其他语言中的哈希表或字典。映射的特点包括：

- **无序性**：映射中的元素是无序的
- **动态性**：可以根据需要添加或删除键值对
- **引用类型**：映射是引用类型，零值为nil，必须初始化才能使用。
- **键唯一性**：每个键只能对应一个值

### 3.2 map的键类型限制

映射的键必须是可比较的类型，包括：

- 布尔类型
- 数字类型
- 字符串类型
- 指针类型
- 通道类型
- 接口类型
- 结构体类型（所有字段都是可比较的）
- 数组类型（元素类型是可比较的）

不可作为键的类型：

- 切片
- 映射
- 函数

### 3.3. map的操作

#### 3.3.1 map的声明和初始化

声明一个map的方式如下：

```go
// 声明一个空映射（零值为nil）
var map1 map[string]int

// 使用make创建映射
map2 := make(map[string]int)

// 直接初始化
map3 := map[string]int{
    "apple":  5,
    "banana": 3,
    "orange": 8,
}

// 声明并初始化
var map4 = map[string]string{
    "name": "张三",
    "city": "北京",
}
```

> map可以嵌套，如`map[string]map[string]int`

#### 3.3.1. 添加和修改键值对

使用`map[key]=value`的方式，可以添加或者修改键值对。

* 如果指定的key在map中不存在，则向map中添加键值对。
* 如果指定的key在map中存在，则修改map中指定key对应的值。

```go
// 创建映射
ages := make(map[string]int)

// 添加元素
ages["张三"] = 25
ages["李四"] = 30

// 修改元素
ages["张三"] = 26

fmt.Println(ages)  // map[张三:26 李四:30]
```

#### 3.3.2. 访问键值

使用`v = map[key]`的方式可以访问map中指定key的值。

* 如果map中存在指定key，返回key对应的值。
* 如果map中不存在指定的key，返回零值（值对应类型的零值）。

```go
ages := map[string]int{
    "张三": 25,
    "李四": 30,
}

// 获取存在的键
age := ages["张三"]
fmt.Println(age)  // 25

// 获取不存在的键（返回零值）
age = ages["王五"]
fmt.Println(age)  // 0
```



使用`v,ok = map[key]`的方式，可以访问map中指定key的值，并且返回该值是否存在的标识。

* 如果map中存在指定key，返回key对应的值，标识符为true。
* 如果map中不存在指定的key，返回零值（值对应类型的零值），标识符为false。

```go
ages := map[string]int{
    "张三": 25,
    "李四": 30,
}

// 检查键是否存在
age, exists := ages["张三"]
if exists {
    fmt.Printf("张三的年龄是%d\n", age)
} else {
    fmt.Println("找不到张三的信息")
}

// 简写形式
if age, ok := ages["李四"]; ok {
    fmt.Printf("李四的年龄是%d\n", age)
}
```



#### 3.3.3. 删除元素

使用delete内置函数，可以删除map中指定的key。

> 删除一个不存在的key，并不会报错哦～

```go
ages := map[string]int{
    "张三": 25,
    "李四": 30,
    "王五": 35,
}

// 删除键值对
delete(ages, "王五")

// 删除不存在的键不会报错
delete(ages, "赵六")

fmt.Println(ages)  // map[张三:25 李四:30]
```

#### 3.3.4. 遍历映射

遍历map有两种方式：

```go
// 只遍历键
for 键 := range 映射名称{

}
// 遍历键和值
for 键,值 := range 映射名称{

}
```

示例：

```go
scores := map[string]int{
    "数学": 95,
    "语文": 87,
    "英语": 92,
}

// 遍历映射（顺序不确定）
for subject, score := range scores {
    fmt.Printf("%s: %d分\n", subject, score)
}

// 只获取键
for subject := range scores {
    fmt.Println("科目:", subject)
}

// 只获取值
for _, score := range scores {
    fmt.Println("分数:", score)
}
```
