错误处理是编写健壮程序的重要组成部分。Go语言采用了不同于传统try-catch机制的错误处理方式，通过显式的错误返回和检查来处理异常情况。这种方式虽然看起来繁琐，但能促使开发者认真对待每一个可能出错的地方。

## 1. 异常机制概述



**错误处理哲学：**

Go语言奉行"错误即是值"的哲学，函数通过返回错误值来表示异常情况，调用者必须显式地检查和处理这些错误。这种设计有几个优点：

1. **显式性**：错误处理代码清晰可见
2. **一致性**：统一的错误处理模式
3. **性能**：避免了异常抛出的开销
4. **可控性**：开发者可以精确控制错误处理流程



**错误类型：**

Go语言中包含以下两种类型的错误（异常）：

* **error**：表示一些可控制的的错误，还可返回 error 类型错误对象来表示函数调用状态。
* **panic**：一般指导致关键流程出现不可修复性错误的错误

## 2. error

### 2.1. error简述

error表示一种可控制的错误，一般由代码显式生成和返回，调用者根据业务需求对error进行处理。



例如go自带的`strconv.ParseBool`方法：

```go
// ParseBool 解析布尔值的字符串
func ParseBool(str string) (bool, error) {
	switch str {
  // 如果为"1", "t", "T", "true", "TRUE", "True":返回true和空错误
	case "1", "t", "T", "true", "TRUE", "True":
		return true, nil
  // 如果为"0", "f", "F", "false", "FALSE", "False":返回false和空错误
	case "0", "f", "F", "false", "FALSE", "False":
		return false, nil
	}
  // 如果不为上述值，则返回false和一个解析错误
	return false, syntaxError("ParseBool", str)
}
```

在`strconv.ParseBool`方法中，如果传入的字符串不合法，会返回一个error。我们在调用时，需要对该error进行处理：

```go
s := "Hello"
b, err := strconv.ParseBool(s)
if err != nil {
  fmt.Printf("Error: %v", err) // Error: strconv.ParseBool: parsing "Hello": invalid syntaxfalse
} else {
  fmt.Println(b)
}

s = "true"
b, err = strconv.ParseBool(s)
if err != nil {
  fmt.Printf("Error: %v", err)
} else {
  fmt.Println(b) // true
}
```

在上面的示例中，我们根据返回的error是否为空来控制业务逻辑：

* 如果解析字符串成功（error为nil）：打印解析结果。
* 如果解析字符串失败（error不为nil）：打印错误信息



### 2.2. error类型

error本质上是go语言内置的一个接口，其源码如下：

```go
type error interface {
	Error() string
}
```

可以看到，error其实就是只有一个`Error() string`方法的接口，任何实现了该接口的类型都是一个error类型。

> 所以在go语言中，error就是一个有特殊意义的普通接口！

### 2.3. 自定义error

上一个章节我们提到，error其实就是只有一个`Error() string`方法的接口，任何实现了该接口的类型都是一个error类型。

示例：

```go
// 定义自定义错误类型
type ValidationError struct {
    Field   string
    Message string
}

// 实现error接口
func (e ValidationError) Error() string {
    return fmt.Sprintf("验证错误 - 字段: %s, 信息: %s", e.Field, e.Message)
}
```

```go
// 使用自定义错误
func validateEmail(email string) error {
    if email == "" {
        return ValidationError{
            Field:   "email",
            Message: "邮箱不能为空",
        }
    }
    if !strings.Contains(email, "@") {
        return ValidationError{
            Field:   "email",
            Message: "邮箱格式不正确",
        }
    }
    return nil
}

func main() {
    err := validateEmail("invalid-email")
    if err != nil {
        // 类型断言检查具体错误类型
        if ve, ok := err.(ValidationError); ok {
            fmt.Printf("字段 %s 验证失败: %s\n", ve.Field, ve.Message)
        } else {
            fmt.Println("其他错误:", err)
        }
    }
}
```



### 2.4. 创建error

创建错误实际上就是创建一个实现了error接口的结构体的对象。

#### 2.4.1. 使用errors.New

`errors.New`方法源码如下：

```go
// 创建一个errorString对象
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```

从源码可以看到，`errors.New`方法实际上是创建了一个`errorString`结构体对象，该结构体实现了error接口，结构体中只包含一个字符串用于记录错误信息。



示例：

```go
import "errors"

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("除数不能为零")
    }
    return a / b, nil
}

func main() {
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("错误:", err.Error())
        return
    }
    fmt.Printf("结果: %.2f\n", result)
}
```

#### 2.4.2. 使用fmt.Errorf

`fmt.Errorf`方法源码如下：

```go
func Errorf(format string, a ...any) error {
    p := newPrinter()
    p.wrapErrs = true
    p.doPrintf(format, a)
    s := string(p.buf)
    var err error
    switch len(p.wrappedErrs) {
    case 0:
       err = errors.New(s)
    case 1:
       w := &wrapError{msg: s}
       w.err, _ = a[p.wrappedErrs[0]].(error)
       err = w
    default:
       if p.reordered {
          slices.Sort(p.wrappedErrs)
       }
       var errs []error
       for i, argNum := range p.wrappedErrs {
          if i > 0 && p.wrappedErrs[i-1] == argNum {
             continue
          }
          if e, ok := a[argNum].(error); ok {
             errs = append(errs, e)
          }
       }
       err = &wrapErrors{s, errs}
    }
    p.free()
    return err
}
```

* `fmt.Errorf`方法可以在创建错误时，格式化错误消息，相比于`errors.New`方法更加灵活。

* 另外从源码可以看到，该方法返回的对象类型有时为`*errors.errorString`，有时为`*fmt.wrapErrors`：

  > `*fmt.wrapErrors`涉及到错误包装的内容，我们后续讲解。



示例：

```go
import "fmt"

func processUser(id int) error {
    if id <= 0 {
        return fmt.Errorf("无效的用户ID: %d", id)
    }
    // 处理用户...
    return nil
}

func main() {
    err := processUser(-1)
    if err != nil {
        fmt.Println("处理用户时出错:", err)
    }
}
```

#### 2.4.3. 创建自定义错误类型

创建自定义错误类型实际上就是创建一个普通的结构体对象：

```go
// 定义自定义错误类型
type ValidationError struct {
    Field   string
    Message string
}

// 实现error接口
func (e ValidationError) Error() string {
    return fmt.Sprintf("验证错误 - 字段: %s, 信息: %s", e.Field, e.Message)
}

// 使用自定义错误
func validateEmail(email string) error {
    if email == "" {
        return ValidationError{
            Field:   "email",
            Message: "邮箱不能为空",
        }
    }
    if !strings.Contains(email, "@") {
        return ValidationError{
            Field:   "email",
            Message: "邮箱格式不正确",
        }
    }
    return nil
}

func main() {
    err := validateEmail("invalid-email")
    if err != nil {
        // 类型断言检查具体错误类型
        if ve, ok := err.(ValidationError); ok {
            fmt.Printf("字段 %s 验证失败: %s\n", ve.Field, ve.Message)
        } else {
            fmt.Println("其他错误:", err)
        }
    }
}
```

### 2.5. 基本error处理

一般来说，如果一个方法可能出现错误，我们通常会在该方法声明中增加一个error`类型的返回值。

```go
// 例如os.Open方法，最后一个返回值即为error类型
func Open(name string) (*File, error) {
	return OpenFile(name, O_RDONLY, 0)
}
```

> 这里所说的错误处理，即是调用者如何处理方法返回的错误。

处理错误最常见的方式是判断方法返回的错误是否为nil，以根据是否发生错误来决定业务流程（打印日志、中止处理等等）。

```go
// 基本的错误检查模式
file, err := os.Open("test.txt")
if err != nil {
    log.Fatal("打开文件失败:", err)
}
defer file.Close()

// 处理文件...
```



### 2.6 错误包装



## 4. panic和recover

虽然Go鼓励使用显式的错误处理，但在某些情况下可以使用panic和recover机制。

### 4.1. panic

panic用于引发运行时恐慌，会立即停止当前函数的执行：

```go
func divide(a, b int) int {
    if b == 0 {
        panic("除数不能为零")
    }
    return a / b
}

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("捕获到恐慌:", r)
        }
    }()

    result := divide(10, 0)
    fmt.Printf("结果: %d\n", result)

    fmt.Println("这行代码不会执行")
}
```

### 4.2. recover

recover用于捕获panic，只能在defer函数中使用：

```go
func riskyOperation() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("恢复自恐慌: %v\n", r)
            // 可以记录日志、清理资源等
        }
    }()

    // 可能引发恐慌的操作
    panic("出错了!")
}

func main() {
    riskyOperation()
    fmt.Println("程序继续执行")
}
```

### 4.3. panic/recover的最佳实践

```go
// 服务器中间件示例
func middleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                // 记录错误日志
                log.Printf("请求 %s 发生恐慌: %v", r.URL.Path, err)

                // 返回500错误
                http.Error(w, "内部服务器错误", http.StatusInternalServerError)
            }
        }()

        next.ServeHTTP(w, r)
    }
}

// 数组安全访问
func safeGet(arr []int, index int) (int, error) {
    defer func() {
        if r := recover(); r != nil {
            log.Printf("数组访问越界: %v", r)
        }
    }()

    if index < 0 || index >= len(arr) {
        panic(fmt.Sprintf("索引%d超出范围[0,%d)", index, len(arr)))
    }

    return arr[index], nil
}
```

## 5. 错误处理最佳实践

### 5.1. 错误处理原则

1. **及时处理错误**：不要忽略错误返回值
2. **添加上下文信息**：使用fmt.Errorf添加有用的上下文
3. **不要吞掉错误**：至少要记录日志
4. **区分错误和异常**：预期的错误用error，真正的异常用panic

```go
// 不好的做法：忽略错误
file, _ := os.Open("test.txt")  // 错误：忽略了错误

// 好的做法：处理错误
file, err := os.Open("test.txt")
if err != nil {
    log.Printf("打开文件失败: %v", err)
    return
}
defer file.Close()
```

### 5.2. 错误日志记录

```go
import (
    "log"
    "go.uber.org/zap"  // 第三方日志库示例
)

func processOrder(orderID string) error {
    order, err := getOrder(orderID)
    if err != nil {
        // 记录结构化日志
        log.Printf("获取订单失败 order_id=%s error=%v", orderID, err)
        return fmt.Errorf("获取订单%s失败: %w", orderID, err)
    }

    if err := validateOrder(order); err != nil {
        log.Printf("订单验证失败 order_id=%s error=%v", orderID, err)
        return fmt.Errorf("订单%s验证失败: %w", orderID, err)
    }

    // 处理订单...
    return nil
}
```

### 5.3. 错误分类

```go
// 定义错误类型常量
type ErrorCode string

const (
    ErrInvalidInput   ErrorCode = "INVALID_INPUT"
    ErrNotFound       ErrorCode = "NOT_FOUND"
    ErrInternal       ErrorCode = "INTERNAL_ERROR"
    ErrUnauthorized   ErrorCode = "UNAUTHORIZED"
)

// 自定义业务错误
type BusinessError struct {
    Code    ErrorCode
    Message string
    Cause   error
}

func (e BusinessError) Error() string {
    return string(e.Code) + ": " + e.Message
}

func (e BusinessError) Unwrap() error {
    return e.Cause
}

// 使用示例
func getUser(id string) (*User, error) {
    if id == "" {
        return nil, BusinessError{
            Code:    ErrInvalidInput,
            Message: "用户ID不能为空",
        }
    }

    // 查询用户...
    return nil, BusinessError{
        Code:    ErrNotFound,
        Message: fmt.Sprintf("用户%s不存在", id),
    }
}

func main() {
    _, err := getUser("")
    if err != nil {
        var bizErr BusinessError
        if errors.As(err, &bizErr) {
            switch bizErr.Code {
            case ErrInvalidInput:
                fmt.Println("输入参数错误:", bizErr.Message)
            case ErrNotFound:
                fmt.Println("资源未找到:", bizErr.Message)
            default:
                fmt.Println("未知业务错误:", bizErr)
            }
        } else {
            fmt.Println("系统错误:", err)
        }
    }
}
```

### 5.4. 错误传播

```go
// 层级错误处理示例
func handleRequest(request Request) error {
    user, err := authenticate(request.Token)
    if err != nil {
        return fmt.Errorf("认证失败: %w", err)  // 包装认证错误
    }

    data, err := fetchData(user.ID)
    if err != nil {
        return fmt.Errorf("获取数据失败: %w", err)  // 包装数据获取错误
    }

    if err := processData(data); err != nil {
        return fmt.Errorf("处理数据失败: %w", err)  // 包装数据处理错误
    }

    return nil
}

// 在上层处理错误
func serveHTTP(w http.ResponseWriter, r *http.Request) {
    var req Request
    // 解析请求...

    if err := handleRequest(req); err != nil {
        // 根据错误类型返回不同的HTTP状态码
        if errors.Is(err, ErrUnauthorized) {
            http.Error(w, "未授权", http.StatusUnauthorized)
        } else if errors.Is(err, ErrNotFound) {
            http.Error(w, "资源未找到", http.StatusNotFound)
        } else {
            http.Error(w, "内部服务器错误", http.StatusInternalServerError)
        }

        // 记录详细错误日志
        log.Printf("处理请求失败: %v", err)
    }
}
```
