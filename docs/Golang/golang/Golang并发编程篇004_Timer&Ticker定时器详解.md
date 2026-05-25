Golang定时器基于通道机制，与goroutine配合使用，提供了简介而强大的时间控制功能。

## 1. Golang中的定时器

Golang中有两种类型的定时器：
* Timer：单次触发的定时器，在指定时间后执行一次。
> Timer 用于延迟执行某个任务
* Ticker：周期性触发的定时器，按照固定间隔重复执行。
> Ticker 用于间隔执行某个任务。

不罗嗦了，我们直接开始看这两种定时器的具体定义和使用！
## 2. Timer 单次定时器
### 2.1. 关于Timer
Timer结构体源码：
```go
type Timer struct {
	C <-chan Time
	r runtimeTimer
}
```
* Timer代表了单个时间：当到达Timer指定的时间后，当前时间会被发送到`C <-chan Time`通道上，除非Timer是由`AfterFunc`方法创建的。（后文会介绍）
* Timer必须由`NewTimer`或`AfterFunc`方法创建。
### 2.2. 初始化

Timer必须由`NewTimer`或`AfterFunc`方法创建。

#### 2.2.1. NewTimer 创建定时器

方法源码：

```go
func NewTimer(d Duration) *Timer {
	c := make(chan Time, 1)
	t := &Timer{
		C: c,
		r: runtimeTimer{
			when: when(d),
			f:    sendTime,
			arg:  c,
		},
	}
	startTimer(&t.r)
	return t
}
```

* `NewTimer`返回一个Timer，在经过指定的时间间隔`d Duration`后，Timer会将当前时间发送到其C通道上。



示例：

```go
log.Printf("starting...\n")

t := time.NewTimer(time.Second * 2) // 创建两秒定时器
<-t.C // 等待定时器触发（两秒）

log.Printf("after 2 sencond...\n")

// 输出
2025/11/21 10:58:11 starting...
2025/11/21 10:58:13 after 2 sencond...
```



#### 2.2.2. AfterFunc 指定时间间隔后执行

方法源码：

```go
func AfterFunc(d Duration, f func()) *Timer {
	t := &Timer{
		r: runtimeTimer{
			when: when(d),
			f:    goFunc,
			arg:  f,
		},
	}
	startTimer(&t.r)
	return t
}
```

* `AfterFunc`方法在指定的时间间隔`d Duration`后，在自己的goroutine中自动调用指定的方法`f func()`。
* `AfterFunc`方法返回的Timer可以用于取消调用（使用Stop方法）。
* `AfterFunc`方法返回的Timer的C通道是nil的，禁止使用。



示例：

```go
log.Printf("starting...\n")

time.AfterFunc(time.Second*2, func() { // 两秒后打印
  log.Printf("after 2 senconds...")
})

time.Sleep(time.Second * 3) // 主协程等待3秒
log.Printf("done")

// 输出
2025/11/21 11:06:11 starting...
2025/11/21 11:06:13 after 2 senconds...
2025/11/21 11:06:14 done
```



#### 2.2.3. After 创建定时器只返回通道

除了`NewTimer`和`AfterFunc`方法外，`After`方法可以创建定时器，但是After方法实际上是通过调用`NewTimer`方法创建的。看源码：

```go
func After(d Duration) <-chan Time {
	return NewTimer(d).C
}
```

从源码可以看到，`After`方法调用`NewTime`r方法创建了一个Timer，**但是只返回了Timer的C通道**。

> 这意味着在指定时间到达之前，调用者没有无法通过调用`Timer.Stop`方法来停止计时，垃圾收集器不会回收底层定时器。

> 所以如果担心效率，请改用 NewTimer，并在不再需要定时器时调用 Timer.Stop



示例：

```go
log.Printf("starting...\n")

timerChan := time.After(time.Second * 2) // 创建2秒定时器（只返回通道）
<-timerChan // 等待定时器触发

fmt.Println("after 2 seconds...")
```



### 2.3. Timer的结构体方法
#### 2.3.1. Stop  停止定时器
源码：
```go
func (t *Timer) Stop() bool {
	if t.r.f == nil {
		panic("time: Stop called on uninitialized Timer")
	}
	return stopTimer(&t.r)
}
```
Stop方法用于停止定时器计时，从而防止定时器触发。
* 如果调用Stop方法成功停止了定时器，方法返回true。如果定时器已经到期（触发过了）或者已经被Stop过了，返回false。
* Stop方法不会关闭定时器的通道，以防止从通道错误地读取成功。
* 为了确保调用Stop后定时器的通道是空的，可以检查Stop方法的返回值并排空通道：
	```go
	if !t.Stop() { // 如果Stop失败
		<-t.C // 排空通道
	}
	```
* 对于使用` AfterFunc(d, f) `创建的定时器，如果` t.Stop` 返回 false，则定时器已经到期，并且函数 f 已在其自己的 goroutine 中启动； Stop 不会等待 f 完成才返回。如果调用者需要知道 f 是否完成，它必须显式地与 f 协调。



#### 2.2.2. Reset 重置定时器
源码
```go
func (t *Timer) Reset(d Duration) bool {
	if t.r.f == nil {
		panic("time: Reset called on uninitialized Timer")
	}
	w := when(d)
	return resetTimer(&t.r, w)
}
```
Reset方法将定时器的过期时间设置为新的，即当前时间过`d Duration`时间后.
* 如果定时器已激活（had been active），将返回true。如果定时器已经到期或者已经被停止，方法返回false。注意这里返回的true和false并不代表是否Reset成功！！！看示例：
	```go
	func main() {
		timer := time.NewTimer(time.Second * 5) // 计时5秒
	
		time.Sleep(time.Second)
		log.Println(timer.Stop()) // 一秒后停止计时 ：true
	
		time.Sleep(time.Second)
		log.Println(timer.Reset(time.Second * 5)) // 再过一秒后Reset成5秒：false
	
		log.Println(<-timer.C) // 输出的是重新Reset，5秒后的时间。
	}
	```
	可以看到，Reset时由于定时器已经被Stop，返回了false。实际上还是设置了新的过期时间为5秒。
	> 请注意，不可能正确使用 Reset 的返回值，因为在耗尽通道和新定时器到期之间存在竞争条件。如上所述，应始终在已停止或过期的通道上调用重置。返回值的存在是为了保持与现有程序的兼容性。
* 对于`NewTimer`方法创建的Timer，调用Reset方法时，应该确保Timer是一个通道中没有值的已过期或已停止的定时器。如果通道不为空，接收方可能会收到两个结果：一个原本的，一个新的。
* 如果程序已经从 t.C 接收到一个值，则知道定时器已到期并且通道已耗尽，因此可以直接使用 t.Reset。但是，如果程序尚未从 t.C 接收到值，则必须停止定时器，并且如果 Stop 报告定时器在停止之前已过期，则通道需要显式耗尽：
	```go
	if !t.Stop() {
		<-t.C
	}
	t.Reset(d)
	```
* 对于使用` AfterFunc(d, f)` 创建的定时器，Reset 要么重新安排 f 运行的时间（在这种情况下 Reset 返回 true），要么安排 f 再次运行（在这种情况下返回 false）。当 Reset 返回 false 时，Reset 既不会等待前一个 f 完成才返回，也不保证后续运行 f 的 goroutine 不会与前一个 goroutine 并发运行。如果调用者需要知道 f 的先前执行是否完成，它必须显式地与 f 协调。
### 2.4. 使用示例
#### 2.4.1. 使用Timer实现延时执行
场景：在指定时间间隔后，开始执行指定任务。
```go
func main() {
	// 创建一个5秒的定时器
	timer := time.NewTimer(time.Second * 5)
	
	// 起一个协程,5秒后执行domSomeThing方法
	go func() {
		<-timer.C
		doSomeThing()
	}()
	
	// 主线程继续执行其他的
	// 这里睡眠等待子协程输出
	time.Sleep(time.Second * 6)
}

func doSomeThing() {
	log.Println("DO")
}
```
上面的示例也可以将`NewTimer`替换成使用`After`方法：
```go
func main() {
	// 创建一个5秒的定时器
	c := time.After(time.Second * 5)

	// 起一个协程,5秒后执行domSomeThing方法
	go func() {
		<-c
		doSomeThing()
	}()

	// 主线程继续执行其他的
	// 这里睡眠等待子协程输出
	time.Sleep(time.Second * 6)
}

func doSomeThing() {
	log.Println("DO")
}
```
当然，这里使用`AfterFunc方法`好像更加优雅
```go
func main() {
	// AfterFunc方法将在5秒后执行doSomeThing方法
	_ = time.AfterFunc(time.Second*5, doSomeThing)

	// 主线程继续执行其他的
	// 这里睡眠等待子协程输出
	time.Sleep(time.Second * 6)
}

func doSomeThing() {
	log.Println("DO")
}
````
#### 2.4.2使用Timer实现超时控制
场景：执行一个比较耗时的操作doSomeThing，如果执行耗时超过5秒，就打印超时日志。
```go
func main() {
	// 定时器设置超时时间为4秒
	timer := time.NewTimer(time.Second * 4)

	// resultChan 用以接收返回结果
	resultChan := make(chan int, 1)

	// 起一个协程去执行耗时操作
	go func() {
		resultChan <- doSomeThing() // 执行结果放入结果通道
	}()

	// 监听两个通道
	select {
	case <-timer.C:
		log.Println("doSomeThing timeout.")
	case r := <-resultChan:
		log.Printf("doSomeThing got result:%d\n", r)
	}

	// 这里等2秒,等待子协程输出
	time.Sleep(2 * time.Second)
}

func doSomeThing() int {
	log.Println("doSomeThing start.")
	time.Sleep(5 * time.Second) // 模拟耗时操作
	log.Println("doSomeThing end.")
	return 100
}

```
输出结果：
```shell
2024/07/24 09:49:07 doSomeThing start.
2024/07/24 09:49:11 doSomeThing timeout.
2024/07/24 09:49:12 doSomeThing end.
```
## 3. Ticker 周期定时器
### 3.1. 关于Ticker
Ticker源码：
```go
type Ticker struct {
	C <-chan Time // The channel on which the ticks are delivered.
	r runtimeTimer
}
```
* Ticker也持有一个C通道，每经过指定的时间间隔，Ticker就会向该通道上发送当前时间（时钟滴答声）。
* 如果C通道的接收者不能及时取走通道上的数据（即接收者在新的时间到达时，还没有从通道取走上一次的时间），Ticker将调整时间间隔，或者废弃部分时间（时钟滴答声）：
	```go
	// sendTime does a non-blocking send of the current time on c.
	func sendTime(c any, seq uintptr) {
		select {
		case c.(chan Time) <- Now():
		default:
		}
	}
	```
### 3.2. 创建Tiker

创建Ticker只有`NewTicker`一个方法：

```go
func NewTicker(d Duration) *Ticker {
	if d <= 0 {
		panic("non-positive interval for NewTicker")
	}
	// Give the channel a 1-element time buffer.
	// If the client falls behind while reading, we drop ticks
	// on the floor until the client catches up.
	c := make(chan Time, 1)
	t := &Ticker{
		C: c,
		r: runtimeTimer{
			when:   when(d),
			period: int64(d),
			f:      sendTime,
			arg:    c,
		},
	}
	startTimer(&t.r)
	return t
}
```

* 时间间隔d必须大于0，否则将会panic。

示例：

```go
ticker := time.NewTicker(time.Second) // 创建周期为1秒的定时器
for t := range ticker.C {
  fmt.Printf("Tick at %s\n", t.Format("2006-01-02 15:04:05"))
}
```



### 3.3. Tiker的结构体方法

#### 3.3.1. Stop 停止定时器
```go
func (t *Ticker) Stop() {
	stopTimer(&t.r)
}
```
* Stop方法停止Ticker，Stop后，不会再往C通道上发送当前时间（时钟滴答声）。
* Stop方法不会关闭C通道，以防止接收者读取到错误的应答。



```go
log.Println("Start...")

ticker := time.NewTicker(time.Second) // 创建周期为1秒的定时器
go func() { // 新协程，每次定时器触发时打印
  for t := range ticker.C {
    log.Printf("Tick at %s\n", t.Format("2006-01-02 15:04:05"))
  }
}()

<-time.After(time.Second * 5) // 主协程定时五秒
ticker.Stop() // 关闭周期定时器
log.Println("Ticker stopped")

time.Sleep(5 * time.Second) // 主协程等待五秒，观察程序行为
log.Println("End...")

// 输出
2025/11/21 11:24:27 Start...
2025/11/21 11:24:28 Tick at 2025-11-21 11:24:28
2025/11/21 11:24:29 Tick at 2025-11-21 11:24:29
2025/11/21 11:24:30 Tick at 2025-11-21 11:24:30
2025/11/21 11:24:31 Tick at 2025-11-21 11:24:31
2025/11/21 11:24:32 Tick at 2025-11-21 11:24:32
2025/11/21 11:24:32 Ticker stopped
2025/11/21 11:24:37 End...
```



#### 3.2.2. Reset 重置定时器
```go
func (t *Ticker) Reset(d Duration) {
	if d <= 0 {
		panic("non-positive interval for Ticker.Reset")
	}
	if t.r.f == nil {
		panic("time: Reset called on uninitialized Ticker")
	}
	modTimer(&t.r, when(d), int64(d), t.r.f, t.r.arg, t.r.seq)
}

```
* Reset方法stop Ticker，并将其时间间隔设置成新的时间间隔。
* 下一个（时钟滴答声）将在新的时间间隔后发送。
* 新的时间间隔必须大于0，否则会panic。



示例：

```go
log.Println("Start...")

ticker := time.NewTicker(time.Second) // 创建周期为一秒的周期定时器
go func() { // 新协程：每次定时器触发时打印
  for t := range ticker.C {
    log.Printf("Tick at %s\n", t.Format("2006-01-02 15:04:05"))
  }
}()

<-time.After(time.Second * 5) // 主协程定时五秒
ticker.Reset(time.Second * 2) // 重置周期定时器的周期为2秒
log.Println("Change to 2 seconds")

time.Sleep(5 * time.Second) // 主协程等待五秒，观察程序输出
log.Println("End...")

// 输出
2025/11/21 11:27:22 Start...
2025/11/21 11:27:23 Tick at 2025-11-21 11:27:23
2025/11/21 11:27:24 Tick at 2025-11-21 11:27:24
2025/11/21 11:27:25 Tick at 2025-11-21 11:27:25
2025/11/21 11:27:26 Tick at 2025-11-21 11:27:26
2025/11/21 11:27:27 Tick at 2025-11-21 11:27:27
2025/11/21 11:27:27 Change to 2 seconds
2025/11/21 11:27:29 Tick at 2025-11-21 11:27:29
2025/11/21 11:27:31 Tick at 2025-11-21 11:27:31
2025/11/21 11:27:32 End...
```



### 3.4. 使用示例
#### 3.4.1. 循环执行
场景：每过指定时间间隔，执行一次任务。
```go
func main() {
	// 指定时间间隔为5秒
	ticker := time.NewTicker(time.Second * 5)
	for {
		<-ticker.C
		// 每5秒执行一次
		doSomeThing()
	}
}

func doSomeThing() {
	log.Println("doSomeThing start.")
	time.Sleep(1 * time.Second) // 模拟耗时操作
	log.Println("doSomeThing end.")
}
```