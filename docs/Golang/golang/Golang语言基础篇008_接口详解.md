接口（Interface）是Go语言中实现抽象和多态的核心机制，它定义了一组方法签名但不包含具体实现。通过接口，我们可以编写更加灵活和可扩展的代码，实现面向接口编程的设计原则。
Go语言的接口设计独特，采用隐式实现的方式，类型只要实现了接口定义的所有方法就自动被认为实现了该接口，这种设计使得接口的使用更加灵活和简洁。本文将深入介绍接口的概念、声明方式、实现方法以及最佳实践，帮助读者掌握Go语言中这一重要特性。

## 1. 声明接口

接口（interface）是Go语言中一种非常重要的特性，它定义了一组方法的签名，但不包含具体实现。接口提供了一种抽象的方式来定义对象的行为。

> 接口（interface）定义了一个对象的行为规范，只定义规范不实现，由具体的对象来实现规范的细节。

声明一个接口的语法格式如下：

```go
type 接口名称 interface{
    方法名称1(参数列表1) 返回值列表1
    方法名称2(参数列表2) 返回值列表2
    ...
}
```

* 接口名：自定义的接口类型名。

  >  Go语言的接口在命名时，一般会在单词后面添加er，如有写操作的接口叫Writer，有字符串功能的接口叫Stringer等。
  >
  > 接口名最好要能突出该接口的类型含义。

* 方法名：当方法名首字母是大写且这个接口类型名首字母也是大写时，这个方法可以被接口所在的包（package）之外的代码访问。

* 参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以省略。

* 接口中没有数据属性。

* 接口中只有方法声明，没有方法实现。

* 接口中可以内嵌别的接口，表示拥有内嵌接口的全部方法。

* 接口中可以嵌套匿名结构体，这是一种特殊情况。


示例：
```go
// 声明一个简单的接口
type Writer interface {
    Write([]byte) (int, error)
}
```

```go
// 声明一个包含多个方法的接口
type ReadWriter interface {
    Read([]byte) (int, error)
    Write([]byte) (int, error)
}
```



## 2. 实现接口

Go语言的接口设计理念与其他面向对象语言有所不同，它更加灵活和隐式。在Go中，如果一个类型实现了接口定义的所有方法，那么就认为该类型实现了这个接口，无需显式声明。

> 一个结构体可以同时实现多个接口，一个接口也可以被多个结构体实现。

示例：

```go
// 这是接口
type Sayer interface {
	say()
}
 
// dog结构体
type dog struct {
}
 
// dog结构体有Sayer接口的所有方法，则认为实现了Sayer接口
func (d dog) say() {
	fmt.Println("汪汪汪")
}
 
// cat结构体
type cat struct {
}
 
// cat结构体有Sayer接口的所有方法，则认为实现了Sayer接口
func (c cat) say() {
	fmt.Println("喵喵喵")
}
 
// 测试方法，传入一个Sayer接口类型
func Say(sayer Sayer) {
	sayer.say()
}
 
func main() {
	Say(cat{}) // 喵喵喵
	Say(dog{}) // 汪汪汪
}
```



## 3. 空接口

空接口是指没有定义任何方法的接口。

可以直接用`interface{}`和`any`来表示空接口：

```go
var a interface{} // 空接口类型的对象

var b any // Go 1.18及以后版本推荐使用any关键字
```



当然，我们也可以自己写一个空接口类型：

```go
type Everything interface{}
```



空接口没有定义任何方法。而上面提到，在Go中，如果一个类型实现了接口定义的所有方法，那么就认为该类型实现了这个接口，无需显式声明。**因此任何类型都实现了空接口，空接口类型的变量可以存储任意类型的变量。**



## 4. 使用接口

接口类型变量能够存储所有实现了该接口的类型的实例。

```Go
// 假设结构体A、B都实现了接口I,接口中有一个方法say()
var i I // 接口变量
a := A{}
b := B{}
i = a
i.say()
i = b
i.say()
```

Go语言中所有类型都默认实现了空接口，所以空接口可以作为任何类型数据的容器 。

```Go
a := 1
var i interface{} // 空接口变量i
i = a
```



示例：

```go
// 这是接口
type Sayer interface {
	say()
}
 
// dog结构体
type dog struct {
}
 
// dog结构体有Sayer接口的所有方法，则认为实现了Sayer接口
func (d dog) say() {
	fmt.Println("汪汪汪")
}
 
// cat结构体
type cat struct {
}
 
// cat结构体有Sayer接口的所有方法，则认为实现了Sayer接口
func (c cat) say() {
	fmt.Println("喵喵喵")
}
 
// 使用接口：传入一个Sayer接口类型
func Say(sayer Sayer) {
	sayer.say()
}
 
func main() {
	Say(cat{}) // 喵喵喵
	Say(dog{}) // 汪汪汪
}
```

>  面向接口编程可以实现更加灵活和具备可扩展性的代码，这只是冰山一角～

## 5. 值/指针接收者的区别

在实现接口时，既可以使用值类型接收者，也可以使用指针类型接收者！

> 甚至还可以混用，即一部分方法用值接收者方式实现、一部分方法用指针接收者方式实现！

```go
// 接口
type Sayer interface {
	Say() string
	Hello() string
}
```

```go
type Cat struct {
}

// 使用值类型接收者实现接口
func (c Cat) Say() string {
	return "meow"
}
```

```go
type Cat struct {
}

// 使用指针类型接收者实现接口
func (c *Cat) Say() string {
	return "meow"
}
```



这两种方式的区别在于：

* 值接收者方式实现接口：既可以使用结构体对象，也可以使用结构体对象指针，来赋值给该接口类型的变量。

  >  因为Go语言中有对指针类型变量求值的语法糖

  ```go
  type Sayer interface {
  	Say() string
  }
  
  type Cat struct {
  }
  
  // 值类型接收者
  func (c Cat) Say() string {
  	return "meow"
  }
  
  func main() {
  	var c Sayer
  	c = &Cat{}
  	println(c.Say()) // 正确
  
  	c = Cat{}
  	println(c.Say()) // 正确
  }
  ```

  

* 指针接收者方式实现接口：结构体对象指针可以赋值给该接口类型的变量。

  ```go
  type Sayer interface {
  	Say() string
  }
  
  type Cat struct {
  }
  
  // 指针类型接收者
  func (c *Cat) Say() string {
  	return "meow"
  }
  
  func main() {
  	var c Sayer
  	c = &Cat{}
  	println(c.Say()) // 正确
  
  	c = Cat{} // 编译报错
  	println(c.Say())
  }
  ```

  

* 两种方式混用：只有结构体对象指针可以赋值给该接口类型的变量。

  ```go
  
  type Sayer interface {
  	Say() string
  	Hello() string
  }
  
  type Cat struct {
  }
  
  // 混用：值类型接收者
  func (c Cat) Say() string {
  	return "meow"
  }
  
  // 混用：指针类型接收者
  func (c *Cat) Hello() string {
  	return "hello"
  }
  
  func main() {
  	var c Sayer
  	c = &Cat{}
  	println(c.Say()) // 正确
  	println(c.Hello()) // 正确
  
  	c = Cat{} // 编译报错
  	println(c.Say())
  	println(c.Hello())
  }
  ```

  

## 6. 内嵌（组合）接口

接口和结构体声明时，都可以内嵌（组合）另一个接口：

* **结构体内嵌接口：**结构体中可以内嵌匿名接口字段，表示实现了该接口。此时结构体可以不去显式实现接口的所有方法，而只需要显式实现结构体需要使用的方法即可。
* **接口内嵌另一个接口：**表示组合另一个接口的功能（可以简单理解成复制了其内嵌接口的方法）。



**结构体内嵌接口：**

```go
// 接口有两个方法
type Sayer interface {
	SayHello()
	SayBye()
}

// 结构体内嵌接口
type Person struct {
	Sayer
	Name string
}

// 只实现其中一个接口方法
func (t *Person) SayHello() {
	fmt.Println("hello")
}

func main() {
	var p Sayer = &Person{Name: "tom"} // 即使没有实现全部接口方法，由于内嵌接口，也是实现了接口
	p.SayHello() // 能够正常调用
	p.SayBye() // 调用没有实现的方法，编译通过，执行panic
}
```

从上例可以看到：

* 内嵌接口后，即使没有实现接口的全部方法，结构体可以当成内嵌的接口类型变量（即实现了接口）。
* **内嵌接口的方式实现接口后，只能调用实现了的房啊，否则会panic。**



**接口内嵌另一个接口：**

```go
// 定义基础接口
type Reader interface {
    Read([]byte) (int, error)
}

type Writer interface {
    Write([]byte) (int, error)
}

type Closer interface {
    Close() error
}

// 组合接口
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

// 定义结构体
type File struct {
    name string
}

// 实现Reader接口
func (f *File) Read(data []byte) (int, error) {
    // 简化实现
    copy(data, []byte("文件内容"))
    return len(data), nil
}

// 实现Writer接口
func (f *File) Write(data []byte) (int, error) {
    fmt.Printf("向文件 %s 写入数据: %s\n", f.name, string(data))
    return len(data), nil
}

// 实现Closer接口
func (f *File) Close() error {
    fmt.Printf("关闭文件 %s\n", f.name)
    return nil
}

// File自动实现了ReadWriter和ReadWriteCloser接口
func main() {
    file := &File{name: "test.txt"}
    
    var rw ReadWriter = file
    rw.Write([]byte("Hello"))
    
    var rwc ReadWriteCloser = file
    rwc.Close()
}
```

从上例可以看到：

* 接口组合另一个接口后，视同与接口定义了被组合接口的所有方法。



## 7. 类型断言

接口类型变量能够存储所有实现了该接口的类型的实例，那么我们应该如何知道这个变量存储的变量的具体类型呢？

答案是使用类型断言，类型断言可以将接口类型的变量转换成其实际存储的类型。

格式：

```go
// 安全的类型断言
具体类型变量,成功标识符 := 接口类型变量.(具体类型)

// 直接类型断言
具体类型变量 := 接口类型变量.(具体类型)
```

* 安全的类型断言
  * 如果实际类型与括号中的具体类型相同，则将接口类型变量转换成具体类型，并且成功标识符为true。
  * 如果实际类型与括号中的具体类型不相同，则转换失败，具体类型变量被赋予零值，成功标识符为false。
* 直接类型断言
  * 如果实际类型与括号中的具体类型相同，则将接口类型变量转换成具体类型。
  * 如果实际类型与括号中的具体类型不相同，则panic。

示例：

```go
// 	接口类型变量
var v interface{}
v = 1000 // 具体类型为int

// 安全的类型断言 - 成功场景
i, ok := v.(int)
if ok {
  fmt.Printf("类型断言成功,i=%d\n", i)
} else {
  fmt.Printf("类型断言失败,i=%d\n", i)
}

// 安全的类型断言 - 失败场景
s, ok := v.(string)
if ok {
  fmt.Printf("类型断言成功,s=%s\n", s)
} else {
  fmt.Printf("类型断言失败,s=%s\n", s)
}

// 直接类型断言 - 成功场景
ii := v.(int)
fmt.Printf("类型断言成功,ii=%d\n", ii)

// 直接类型断言 - 失败场景
ss := v.(string)
fmt.Printf("类型断言成功,ss=%s\n", ss)
```

输出：

```go
// 实际输出：
类型断言成功,i=1000
类型断言失败,s=
类型断言成功,ii=1000
panic: interface conversion: interface {} is int, not string
```



## 8. 类型选择

使用类型断言，可以将接口类型变量中存储的值转换成具体类型的值。

而有时候，我们需要根据接口类型变量的具体类型，做不同的操作。如果全部使用类型断言去做，代码就会非常的难看且繁长。此时我们就可以使用类型选择这种语法来根据接口变量的具体类型执行不同的操作。



语法格式：

```go
// 需要具体类型变量的值
switch 具体类型变量 := 接口类型变量.(type) {
  case 具体类型1:
      // 业务逻辑
  case 具体类型2:
      // 业务逻辑
  case 具体类型3:
      // 业务逻辑
  default:
      // 业务逻辑
  }

// 不需要具体类型变量的值
switch 接口类型变量.(type) {
  case 具体类型1:
      // 业务逻辑
  case 具体类型2:
      // 业务逻辑
  case 具体类型3:
      // 业务逻辑
  default:
      // 业务逻辑
  }
```

* case中的所有具体类型，都必须是实现了该接口的类型，否则会编译报错。



示例：

```go
func Print(v any) {
	switch vv := v.(type) {
	case int:
		fmt.Printf("type: %s, value = %d\n", "int", vv)
	case string:
		fmt.Printf("type: %s, value = %s\n", "string", vv)
	case byte:
		fmt.Printf("type: %s, value = %v\n", "byte", vv)
	case bool:
		fmt.Printf("type: %s, value = %t\n", "bool", vv)
	default:
		fmt.Printf("type: %s, value = %v\n", "unknown", vv)
	}
}

func main() {
	Print(1) // type: int, value = 1
	Print("hello") // type: string, value = hello
	Print(true) // type: bool, value = true
	Print(byte(1)) // type: byte, value = 1
	Print(1.1) // type: unknown, value = 1.1
	Print(complex(1, 2)) // type: unknown, value = (1+2i)
}
```



## 9. 接口的最佳实践

### 9.1. 倾向于定义小接口

Go语言倾向于定义小而专注的接口，这样更容易实现和组合：

```go
// 好的做法：小而专注的接口
type Stringer interface {
    String() string
}

type Reader interface {
    Read([]byte) (int, error)
}

type Writer interface {
    Write([]byte) (int, error)
```

### 9.2. 接受接口，返回结构体

在设计函数时，倾向于接受接口参数，返回具体类型：

```go
// 接受接口作为参数
func ProcessData(r Reader) []byte {
    data := make([]byte, 1024)
    r.Read(data)
    return data
}

// 返回具体类型
func NewFile(name string) *File {
    return &File{name: name}
}
```

### 9.3. 接口作为函数参数实现解耦

```go
type Animal interface {
    Speak() string
}
```

```go
type Dog struct{}
func (d Dog) Speak() string {
    return "汪汪"
}

type Cat struct{}
func (c Cat) Speak() string {
    return "喵喵"
}

// 使用接口作为函数参数实现解耦
func MakeAnimalSpeak(a Animal) {
    fmt.Println(a.Speak())
}

func main() {
    dog := Dog{}
    cat := Cat{}
    MakeAnimalSpeak(dog) // 汪汪
    MakeAnimalSpeak(cat) // 喵喵
}
```



### 9.4. 接口与结构体的组合使用

接口和结构体经常一起使用来实现面向对象的设计模式：

```go
// 定义接口
type Shape interface {
    Area() float64
    Perimeter() float64
}
```

```go
// 定义结构体
type Rectangle struct {
    Width, Height float64
}

type Circle struct {
    Radius float64
}

// 为结构体实现接口
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

// 使用接口
func PrintShapeInfo(s Shape) {
    fmt.Printf("面积: %.2f, 周长: %.2f\n", s.Area(), s.Perimeter())
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    circle := Circle{Radius: 3}
    
    PrintShapeInfo(rect)
    PrintShapeInfo(circle)
}
```

## 10. 接口值的内部结构

一个接口的值（简称接口值）是由一个具体类型和具体类型的值两部分组成的。这两部分分别称为接口的动态类型和动态值。
1. 类型信息（动态类型）
2. 值信息（动态值）

示例：

```go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```

![image-20250925164925471](../../.public/image-20250925164925471.png)
