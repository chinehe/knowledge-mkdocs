Go语言中没有“类”的概念，也不支持“类”的继承等面向对象的概念。

GO使用结构体来自定义数据类型,通过结构体的内嵌再配合接口比面向对象具有更高的扩展性和灵活性。

## 1. 结构体的概念

Go语言中的基础数据类型可以表示一些事物的基本属性，但是当我们想表达一个事物的全部或部分属性时，这时候再用单一的基本数据类型明显就无法满足需求了，Go语言提供了一种自定义数据类型，可以封装多个基本数据类型，这种数据类型叫结构体，英文名称struct。 也就是我们可以通过struct来定义自己的类型了。
> Go语言中通过struct来实现类似面向对象的效果。

结构体是一种用户自定义的数据类型，它允许我们将不同类型的数据组合在一起。结构体由一系列字段组成，每个字段都有自己的名称和类型。

## 2. 声明结构体

### 2.1. 声明全新的结构体

声明一个结构体的基本语法如下：

```go
type 类型名 struct{
    属性1 属性类型1
    属性2 属性类型2
}
```

- 类型名：标识自定义结构体的名称，在同一个包内不能重复。
- 属性名：
  - 结构体中字段大写开头表示可公开访问，小写表示私有（仅在定义当前结构体的包中可访问）。
  - 结构体中的属性名必须唯一。
- 属性类型：表示结构体属性的具体类型。

示例：

```go
type person struct {
    name, city string // 相同类型的属性可以写在一起
    age        int8
}
```

### 2.2. 基于现有类型定义新类型

#### 2.2.1. 使用方式

除了声明全新的结构体外，还可以基于现有的类型（不一定是结构体）来定义新类型：

```go
type 新类型名称 现有类型名称 
```

- 新类型具有和现有类型相同的属性。
- 新类型与现有类型是两个不同的类型，(不能直接使用现有类型的方法，也不能把新类型存入现有类型的数组…);可以通过强转成现有类型后使用。

示例：

```go
type MyInt int // 定义int的新类型

type MyFunc func(int) int // 定义func的新类型

type User struct {
    a int
    b string
}

type MyUser User // 定义结构体的新类型

type I interface {
    get() string
}

type MyInterface I // 定义接口的新类型
```

#### 2.2.2. 基础类型

基于基础类型定义的新类型具有如下特性：

1. 由于新类型与基本类型是不同的类型，因此不能直接相互比较或直接运算，需要强转后才能相互比较和运算。
2. 相同的新类型变量可以直接相互比较和直接运算。
3. 可以为新类型定义新方法。

```go
type MyInt int // int的新类型

type MyString string // string的新类型

func main() {
    i := 1
    MyI := MyInt(1)
    MyII := MyInt(1)
    fmt.Println(i == MyI) // 原类型与新类型变量不能直接比较，此处报错
    fmt.Println(MyInt(i) == MyI)  // 强转后可以比较
    fmt.Println(MyI == MyII) // 相同新类型可以比较

    s := "Hello World"
    myS := MyString("Hello World")
    mySS := MyString("Hello World")
    fmt.Println(s == myS) // 原类型与新类型变量不能直接比较，此处报错
    fmt.Println(MyString(s) == myS) // 强转后可以比较
    fmt.Println(myS == mySS)// 相同新类型可以比较
    fmt.Println(strings.ToUpper(myS)) // 也不能通用，此处报错
    fmt.Println(strings.ToUpper(string(myS))) // 需要强转
}
```

#### 2.2.3. 函数

由于Go语言中函数也是一种类型，因此我们可以基于一种函数创建一种新类型。

```go
type 新类型名称 func(参数列表)返回列表
```

Go语言中，具有相同参数列表和相同返回列表的函数被视为相同的类型。因此当基于一种函数类型定义新类型后，所有具有相同参数列表和返回值列表的函数都是该新类型。

```go
type IntOperateFunc = func(int, int) int // 定义一种新类型

func Add(a, b int) int {
	return a + b
}

func Sub(a, b int) int {
	return a - b
}

func Multiply(a, b int) int {
	return a * b
}

func Divide(a, b int) int {
	return a / b
}

func Operate(a, b int, operate IntOperateFunc) int { // 这个方法入参中包含新类型
	return operate(a, b)
}

func main() {
	Operate(1, 2, Add) // 具有相同参数列表和返回值列表的函数可以被直接当成新类型使用
	Operate(1, 2, Sub) // 具有相同参数列表和返回值列表的函数可以被直接当成新类型使用
	Operate(1, 2, Multiply) // 具有相同参数列表和返回值列表的函数可以被直接当成新类型使用
	Operate(1, 2, Divide) // 具有相同参数列表和返回值列表的函数可以被直接当成新类型使用
}

```

#### 2.2.4. 结构体

基于现有结构体定义的新结构体具备如下特性：

* 新结构体具备与原始结构体相同的属性字段。
* 新结构体不具备原始结构体的方法。
* 新结构体与原始结构体之间可以强转。

示例：

```go
// 原始结构体
type Person struct {
  Name string
  Age  int
}

// 原始结构体的方法
func (p Person) Print() {
  fmt.Printf("Name: %s, Age: %d\n", p.Name, p.Age)
}

// 基于原始结构体定义新结构体
type NewPerson Person

// 新结构体的方法
func (np NewPerson) NewPrint() {
  fmt.Printf("New Name: %s, New Age: %d\n", np.Name, np.Age)
}

func main() {
  // 原始结构体属性
  p := Person{Name: "John", Age: 30}
  
  // 新结构体具有和原始结构体相同的属性
  newP := NewPerson{Name: "John", Age: 30}
  
  // 原始结构体的方法
  p.Print() // Name: John, Age: 30
  newP.Print() // 新结构体不能直接使用原始结构体的方法，此处报错
  Person(newP).Print() // 新结构体强转成原始结构体：Name: John, Age: 30

  // 新结构体的方法
  p.NewPrint() // 原始结构体不能直接使用新结构体的方法，此处报错
  newP.NewPrint() // New Name: John, New Age: 30
  NewPerson(p).NewPrint() // 原始结构体强转成新结构体：New Name: John, New Age: 30
}
```

#### 2.2.5. 接口

也可以为基于接口创建新类型：

```go
// Printer 接口
type Printer interface {
	Print()
}

type NewPrinter  Printer
```

新接口类型的特性：

* Go接口没有属性，因此不存在新旧接口属性相同的说法。

* 新接口类型自动具备与原始接口完全相同的方法。

  > 如果我们把接口方法理解成接口的“属性”，是不是会好理解一点呢。

* 基于接口创建的新类型也是一种接口类型，不能为其定义新的方法。

* 新老接口类型不需要强转也能直接通用。



有没有发现一个问题，新老接口具备完全相同的方法集！一个结构体实现了其中一个接口，就自然而然实现了另一个接口。

> 所以这两个类型除了名字不一样，其他看起来是一模一样。
>
> 我也还没发现这种做法有什么作用！



示例：

```go

// 旧接口
type Printer interface {
	Print()
}

// 新接口类型
type NewPrinter Printer

// 因为新类型也是一个接口类型，不能为新接口类型定义新方法，此处报错
func (p NewPrinter)Do()  {
}

// 	定义一个结构体，实现接口，会自动实现这两个接口
type Person struct {
	Name string
	Age  int
}

func (p Person) Print() {
	fmt.Println("Name:", p.Name, "Age:", p.Age)
}

// 此处测试用，入参为原始接口类型
func Do(myInterface Printer) {
	myInterface.Print()
}

func main() {
  // 原始接口类型
	var p Printer
  // 新接口类型
	var newP NewPrinter
	p = Person{"Alice", 18}
	newP = Person{"Bob", 20}
	Do(p) // 传入原始接口类型
	Do(newP) // 传入新接口类型，不需要强转也可以
}
```



### 2.3. 类型别名

>  类型别名是Go1.9版本添加的功能。

在上一个章节中，我们介绍了一种基于现有类型定义新类型的方式。

现在，我们将介绍一种“差不多的”概念 — 类型别名！

#### 2.3.1. 定义方式

定义类型别名的方式和上一章节“基于现有类型定义新类型”及其类似：

```go
// 基于现有类型定义新类型
type 新类型名称 现有类型名称 

// 类型别名
type 类型别名 = 现有类型名称
```

LOOK！！！

类型别名定义时，就多了一个`=`号。



#### 2.3.2. 特性

类型别名只是Type的别名，本质上TypeAlias与Type是同一个类型：

* 类型别名与现有类型是相同的类型，(能直接使用现有类型的方法，也能把类型别名的变量存入现有类型的数组…)。

* 类型别名和“基于现有类型定义新类型”不同，前者本质上就是现有类型；后者是基于现有类型创建了一个新的类型。

```go
type OldType struct{}    // 现有类型
type TypeAlias = OldType // 类型别名
type NewType OldType     // 基于现有类型创建新类型
 
func main() {
    fmt.Printf("%T\n", OldType{})   // main.OldType
    fmt.Printf("%T\n", TypeAlias{}) // main.OldType
    fmt.Printf("%T\n", NewType{})   // main.NewType
}
```



```go
// 方式1：零值初始化
var person1 Person

// 方式2：显式初始化
person2 := Person{Name: "张三", Age: 25, Sex: "男"}

// 方式3：按顺序初始化（不推荐，易出错）
person3 := Person{"李四", 30, "女"}

// 方式4：使用new关键字
person4 := new(Person)  // 返回指向结构体的指针

// 方式5：匿名结构体
address := struct {
    City  string
    State string
}{
    City:  "北京",
    State: "北京市",
}
```

## 3. 实例化、初始化

* 这里的实例化指为结构体对象分配内存。
* 这里的初始化指为结构体对象的属性赋值。

### 3.1 声明实例化

结构体本身也是一种类型，我们可以像声明内置类型一样使用var关键字声明结构体类型：

```go
var 变量名称 结构体类型
```

声明结构体类型的变量时，就已经为变量分配了内存，并完成了结构体属性的默认赋值。

> 此时所有结构体字段被赋予其类型的零值。



示例：

```go
type Person struct {
	Age  int
	Name string
}

func main() {
	var p Person
	fmt.Printf("%+v", p) // {Age:0 Name:}
}
```



### 3.2 new内置函数实例化

还可以使用`new`内置函数来实例化一个结构体：

```go
变量名 := new(结构体名)
```

`new`内置函数作用：

* 实例化结构体，所有结构体字段被赋予其类型的零值。
* 返回对象的指针。



示例：

```go
type Person struct {
	Age  int
	Name string
}

func main() {
	p := new(Person)
	fmt.Printf("%+v", p) // &{Age:0 Name:}
}
```

### 3.3 使用属性键值对初始化

格式：

```go
变量名 := 结构体名{
  属性1:值1,
  属性2:值2,
}
```

通过上述的方式，可以在声明结构体对象变量的同时，对结构体属性进行初始化：

* 键对应结构体的属性，值对应该字段的初始值。
* 键值对可以没有，也可以是所有结构体属性。
* 对于没有指定键值对的属性，其值初始化为对应类型的零值。



示例：

```go
type Person struct {
	Age  int
	Name string
}

func main() {
	p := Person{
		Name: "张三",
	}
	fmt.Printf("%+v", p) // {Age:0 Name:张三}
}
```



### 3.4 使用属性值列表初始化

初始化结构体的时候可以简写，也就是初始化的时候不写键，直接写值。

这种方式需要注意：

- 必须初始化结构体的所有字段
- 初始值的填充顺序必须与字段在结构体中的声明顺序一致。
- 该方式不能与键值对初始化方式混用。



示例：

```go
type person struct {
    name, city string // 相同类型的属性可以写在一起
    age        int8
}
 
func main() {
    var p = person{
        "chinehe",
        "WuXi",
        1,
    }
}
```

### 3.5 自定义构造函数

Go语言的结构体没有构造函数，我们可以自己实现。

```Go
func newPerson(name, city string, age int8) *person {
    return &person{
        name: name,
        city: city,
        age:  age,
    }
}
```

调用构造函数：

```Go
p9 := newPerson("pprof.cn", "测试", 90)
fmt.Printf("%#v\n", p9)
```

> 因为struct是值类型，如果结构体比较复杂的话，值拷贝性能开销会比较大，所以该构造函数返回的是结构体指针类型。

## 4. 访问结构体属性

其实上文中的示例代码中，已经无数次展示了如何方法结构体的属性～

即使用`.`操作符：

```go
type Person struct {
    Name string
    Age  int
}

func main() {
    // 创建结构体实例
    person := Person{Name: "张三", Age: 25}

    // 访问字段
    fmt.Printf("姓名: %s\n", person.Name) // 姓名: 张三
    fmt.Printf("年龄: %d\n", person.Age) // 年龄: 25

    // 修改字段
    person.Age = 26
    fmt.Printf("新年龄: %d\n", person.Age) // 新年龄：26

    // 指针访问字段
    ptr := &person
    ptr.Name = "李四"
   // 通过指针修改会影响原值
    fmt.Printf("新姓名: %s\n", person.Name)  // 新姓名: 李四
}
```

## 5. 结构体方法

### 5.1. 概念

Go语言中的方法（Method）是一种作用于特定类型变量的函数。这种特定类型变量叫做接收者（Receiver）。接收者的概念就类似于其他语言中的this或者 self。

> 方法与函数的区别是，函数不属于任何类型，方法属于特定的类型。

### 5.2. 定义方式

结构体方法的定义格式如下：

```go
func (接收者变量 接收者类型)方法名称(参数列表)(返回参数){
    方法体
}
```

* 接收者中的参数变量名在命名时，官方建议使用接收者类型名的第一个小写字母，而不是self、this之类的命名。例如，Person类型的接收者变量应该命名为 p，Connector类型的接收者变量应该命名为c等。
* 接收者类型：接收者类型和参数类似，可以是指针类型和非指针类型。
  * 同一个类型，不能定义两个名称相同的方法，即使接收者类型分别为指针类型和非指针类型。
  * 同一个类型的方法，推荐保持接收者类型一致（非强制），要么都是指针类型，要么都是非指针类型。
  * 方法名、参数列表、返回参数：具体格式与函数定义相同。

>  注意事项：非本地类型不能定义方法，也就是说我们不能给别的包的类型定义方法。

### 5.3 指针接收者与非指针接收者

* 指针类型的接收者由一个结构体的指针组成，由于指针的特性，调用方法时修改接收者指针的任意成员变量，在方法结束后，修改都是有效的。这种方式就十分接近于其他语言中面向对象中的this或者self。
* 当方法作用于值类型接收者时，Go语言会在代码运行时将接收者的值复制一份。在值类型接收者的方法中可以获取接收者的成员值，但修改操作只是针对副本，无法修改接收者变量本身。

示例：

```go

type Person struct {
	Name string
	Age  int
}

// 非指针接收者
func (p Person) PrintAndReset() {
	fmt.Printf("Name: %s, Age: %d\n", p.Name, p.Age)
	p.Name = ""
	p.Age = 0
}

// 指针接收者
// 这里演示用，其实不推荐混用
func (p *Person) PrintAndReset2() {
	fmt.Printf("Name: %s, Age: %d\n", p.Name, p.Age)
	p.Name = ""
	p.Age = 0
}

func main() {
	p1 := Person{Name: "Alice", Age: 30}
	p1.PrintAndReset()
	fmt.Printf("Name: %s, Age: %d\n", p1.Name, p1.Age) // Name: Alice, Age: 30

	p2 := &Person{Name: "Bob", Age: 25}
	p2.PrintAndReset2()
	fmt.Printf("Name: %s, Age: %d\n", p2.Name, p2.Age) // Name: , Age: 0
}
```



**特别注意：**

- 不论接收者是指针还是非指针类型，都可以使用类型对象变量和类型对象指针变量来调用
- 如果不使用变量，而直接在初始化语句后调用结构体方法，则不能用类型对象变量来调用指针类型接收者方法。

```go
type Person struct {
	name string
}
 
func (p Person) Hello(prefix string) string {
	return fmt.Sprintf(prefix, p.name)
}
 
func (p *Person) Hello2(prefix string) string {
	return fmt.Sprintf(prefix, p.name)
}
 
func main() {
	p := &Person{"chinehe"}
	p2 := Person{"chinehe"}
	fmt.Println(p.Hello("hello,i am %s"))   // 正确
	fmt.Println(p.Hello2("hello,i am %s"))  // 正确
	fmt.Println(p2.Hello("hello,i am %s"))  // 正确
	fmt.Println(p2.Hello2("hello,i am %s")) // 正确
 
	fmt.Println((&Person{"chinehe"}).Hello("hello,i am %s"))  // 正确
	fmt.Println((&Person{"chinehe"}).Hello2("hello,i am %s")) // 正确
	fmt.Println((Person{"chinehe"}).Hello("hello,i am %s"))   // 正确
	fmt.Println((Person{"chinehe"}).Hello2("hello,i am %s"))  // 错误
}
```



在Go语言中，我们可以为结构体定义方法：

```go
type Rectangle struct {
    Width, Height float64
}

// 为Rectangle定义方法（值接收者）
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// 为Rectangle定义方法（指针接收者）
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}

    // 调用值接收者方法
    fmt.Printf("面积: %.2f\n", rect.Area())  // 50.00

    // 调用指针接收者方法
    rect.Scale(2)
    fmt.Printf("缩放后面积: %.2f\n", rect.Area())  // 200.00
}
```

### 5.4 调用结构体的方法

根据调用者不同，分为两种表现形式:

- 结构体对象直接调用

  ```go
  结构体对象.方法名称(参数列表)
  ```

- 结构体对象作为参数调用

  ```go
  结构体类型或类型指针.方法名称(结构体对象,参数列表)
  ```

  > 这种方式下，结构体对象的调用规则应该遵从参数的规则，而非接收者的规则



示例：

```go
type Person struct {
	name string
}
 
func (p Person) Hello(prefix string) string {
	return fmt.Sprintf(prefix, p.name)
}
 
func (p *Person) Hello2(prefix string) string {
	return fmt.Sprintf(prefix, p.name)
}
 
func main() {
	p := Person{"chinehe"}
	p2 := &Person{"chinehe"}
	fmt.Println(Person.Hello(p, "hello,i am %s"))
	fmt.Println(Person.Hello2(p2, "hello,i am %s")) // 错误
	fmt.Println((*Person).Hello(p, "hello,i am %s")) // 错误
	fmt.Println((*Person).Hello(p2, "hello,i am %s"))
}
```



## 6. 匿名结构体

在定义一些临时数据结构等场景下还可以使用匿名结构体，即没有名字的结构体：

```go
var 变量名称 struct{
    属性1 属性类型1
    属性2 属性类型2
}
```

示例：

```go
var person  = struct {
    name, city string // 相同类型的属性可以写在一起
    age        int8
}{
    age: 18,
    name: "chinehe",
}
```



## 7. 结构体匿名字段

结构体允许其成员字段在声明时没有字段名而只有类型，这种没有名字的字段就称为匿名字段。

示例：

```go
type person struct {
    string
    age int
}
```

注意：

- 匿名字段默认采用类型名作为字段名。
- 匿名字段可以和非匿名字段混用。
- 结构体要求字段名称必须唯一，因此一个结构体中同种类型的匿名字段只能有一个。且不能有非匿名同种类型的字段使用类型名做字段名。

## 8. 结构体嵌套与继承

### 8.1. 嵌套结构体

一个结构体中可以嵌套包含另一个结构体或结构体指针。

```go
type 结构体名称 struct{
    嵌套结构体属性名称 嵌套结构体
    // 其他属性
}
```

- 嵌套结构体可以是结构体本身，也可以是结构体指针。
- 嵌套结构体属性和其他的属性并没有什么区别，只是属性类型为结构体。

示例：

```go
type address struct {
    pro  string
    city string
}
 
type person struct {
    name    string
    age     int
    firstAddress address // 结构体
    secondAddress *address // 结构体指针
}
```

### 8.2 匿名嵌套结构体

如果在嵌套结构体属性时，不指定属性名称，这就称为嵌套匿名结构体。

```go
type 结构体名称 struct{
    嵌套结构体
    // 其他属性
}
```

- 嵌套结构体可以是结构体本身，也可以是结构体指针。
- 该匿名嵌套结构体属性的默认名称为结构体名称。



**使用匿名结构体属性的子属性时，可以直接访问。**

* 当访问结构体成员时会先在结构体中查找该字段，找不到再去匿名结构体中查找。
  * 外层结构体属性与匿名嵌套结构体属性子属性重复时，默认使用外层结构体属性。如果要使用匿名嵌套结构体属性子属性，需要指定匿名结构体属性名称。
  * 多个匿名嵌套结构体属性的子属性重复时，为了避免歧义需要指定具体的匿名内嵌结构体属性，否则会报错。

* 初始化时，若要初始化匿名结构体属性的值，不能直接赋值子属性，需要指定名称。

```go
type address struct {
    pro  string
    city string
}
 
type person struct {
    name string
    age  int
    address
}
 
func main() {
    var person = person{
        name: "chinehe",
        age:  18,
        address: address{
            pro:  "JS",
            city: "WX",
        },
    }
    fmt.Println(person.name) // chinehe
    fmt.Println(person.age) // 18
    fmt.Println(person.pro) // JS
    fmt.Println(person.city) // WX
}
```

### 8.3 结构体“继承”

Go语言中使用嵌套匿名结构体的方式，也可以实现类似于继承的效果。

- 外层结构体可以直接使用嵌套匿名结构体属性的子属性。
- 外层结构体可以直接使用嵌套匿名结构体的方法。

```go
type person struct {
    name string
    age  int
}
 
func (p *person) hello() {
    fmt.Printf("hello,i am %v", p.name)
}
 
type student struct {
    person
    school string
}
 
func main() {
    s := student{
        person: person{
            name: "ChineHe",
            age:  18,
        },
        school: "JNU",
    }
    s.hello()
}
```



## 9. 结构体标签(Tag)

结构体标签是附加在字段上的元数据，常用于JSON序列化、数据库映射等场景，可以在运行的时候通过反射的机制读取出来。

Tag在结构体字段的后方定义，由一对反引号包裹起来，具体的格式如下：

```go
type 结构体名称 struct{
    属性名称 属性类型 `k1:"v1" k2:"v2"`
}
```

- 结构体标签由一个或多个键值对组成。
- 键与值使用冒号分隔，值用双引号括起来。
- 键值对之间使用一个空格分隔。



**注意事项：** 为结构体编写Tag时，必须严格遵守键值对的规则。结构体标签的解析代码的容错能力很差，一旦格式写错，编译和运行时都不会提示任何错误，通过反射也无法正确取值。



常用的标签选项：

- `json:"name"` - 指定JSON字段名
- `json:"name,omitempty"` - 如果字段为空值则忽略
- `json:"-"` - 忽略该字段
- `db:"column_name"` - 数据库字段名（ORM使用）

