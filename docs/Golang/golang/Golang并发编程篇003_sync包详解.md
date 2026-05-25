> sync包是Go标准库中的一个重要包，提供了基本的同步原语，用于协调goroutine的执行。包括Mutex互斥锁、WaitGroup、Once、Cond、Pool、Atomic原子操作等主要组件。
## 1. atomic原子操作包

### 1.1. 关于atomic

`atomic`是`sync`下的子包，提供了低级原子内存原语，对实现同步算法非常有用。

> 使用这些方法需要非常小心，除了特殊的低级应用程序外，最好还是用通道和`sync`包下的其他组件来实现同步：Share memory by communicating; don't communicate by sharing memory.



用Go内存模型的术语来说：如果原子操作A的效果能够被原子操作B观察到，那么A会在B之前同步。

这意味着在并发环境中，A的操作结果对B是可见的，并且保证了执行顺序

> if the effect of an atomic operation A is observed by atomic operation B, then A "synchronized before" B.



程序中所有的原子操作，都会按照某种顺序一致的方式执行。这种语义与C++的顺序一致原子操作以及Java中的volatile变量具有相同的语义。

> All the atomic operations executed in a program behave as through executed in some sequentially consistent order. 
>
> This definition provides the same semantics as C++'s sequentially consistent atomics and Java's volatile variables



**总结：**aotomic确保并发访问共享数据时的数据一致性，同时避免竞态条件。这些规则定义了原子操作之间的内存可见性和执行顺序关系，使开发者能够在不需要显式锁的情况下实现安全的并发控制。



### 1.2. atomic原子操作

`atomic`包提供的常用原子操作如下：

* SwapT：存储新值并返回旧值

  ```go
  // 类似于
  old = *addr
  *addr = new
  return old
  ```

* CompareAndSwapT：比较并交换

  ```go
  // 类似于
  if *addr == old {
  	*addr = new
  	return true
  }
  return false
  ```

* AddT：加

  ```go
  // 类似于
  *addr += delta
  return *addr
  ```

* LoadT：加载
  ```go
  // 类似于
  return *addr
  ```
  
* StoreT：存储
  ```go
  // 类似于
  *addr = val
  ```



Go提供了两种方式来调用上述的原子操作：

* 直接使用atomic原子操作方法。

  ```go
  // 示例
  intVal := new(int32)
  fmt.Println(atomic.LoadInt32(intVal)) // 0
  
  atomic.StoreInt32(intVal, 10)
  fmt.Println(atomic.LoadInt32(intVal)) // 10
  
  fmt.Println(atomic.SwapInt32(intVal, 20)) // 10
  fmt.Println(atomic.LoadInt32(intVal)) // 20
  
  fmt.Println(atomic.CompareAndSwapInt32(intVal, 20, 30)) // true
  fmt.Println(atomic.LoadInt32(intVal)) // 30
  
  fmt.Println(atomic.AddInt32(intVal, 10)) // 40 
  fmt.Println(atomic.LoadInt32(intVal)) // 40
  ```

* 使用atomic原子类的方法。

  ```go
  // 示例
  intVal := atomic.Int32{}
  
  fmt.Println(intVal.Load()) // 0
  
  intVal.Store(10)
  fmt.Println(intVal.Load()) // 10
  
  fmt.Println(intVal.Swap(20)) // 10
  fmt.Println(intVal.Load()) // 20
  
  fmt.Println(intVal.CompareAndSwap(20, 30)) // true
  fmt.Println(intVal.Load()) // 30
  
  fmt.Println(intVal.Add(10)) // 40
  fmt.Println(intVal.Load()) // 40
  ```



### 1.3. 支持的类型

**原子操作方法支持的类型：**

* Int32
  * `func SwapInt32(addr *int32, new int32) (old int32)`
  * `func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)`
  * `func AddInt32(addr *int32, delta int32) (new int32)`
  * `func AndInt32(addr *int32, mask int32) (old int32)`
  * `func OrInt32(addr *int32, mask int32) (old int32)`
  * `func LoadInt32(addr *int32) (val int32)`
  * `func StoreInt32(addr *int32, val int32)`
* UInt32
  * `func SwapUint32(addr *uint32, new uint32) (old uint32)`
  * `func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)`
  * `func AddUint32(addr *uint32, delta uint32) (new uint32)`
  * `func AndUint32(addr *uint32, mask uint32) (old uint32)`
  * `func OrUint32(addr *uint32, mask uint32) (old uint32)`
  * `func LoadUint32(addr *uint32) (val uint32)`
  * `func StoreUint32(addr *uint32, val uint32)`
* Uintptr
  * `func SwapUintptr(addr *upintptr, new upintptr) (old upintptr)`
  * `func CompareAndSwapUintptr(addr *upintptr, old, new upintptr) (swapped bool)`
  * `func AddUintptr(addr *upintptr, delta upintptr) (new upintptr)`
  * `func AndUintptr(addr *upintptr, mask upintptr) (old upintptr)`
  * `func OrUintptr(addr *upintptr, mask upintptr) (old upintptr)`
  * `func LoadUintptr(addr *upintptr) (val upintptr)`
  * `func StoreUintptr(addr *upintptr, val upintptr)`
* Pointer
  * `func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)`
  * `func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)`
  * `func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)`
  * `func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)`

**原子类型：**

* atomic.Bool
  * `func (x *Bool) Load() bool`
  * `func (x *Bool) Store(val bool)`
  * `func (x *Bool) Swap(new bool) (old bool)`
  * `func (x *Bool) CompareAndSwap(old, new bool) (swapped bool)`
* atomic.Pointer
  * `func (x *Pointer[T]) Load() *T`
  * `func (x *Pointer[T]) Store(val *T)`
  * `func (x *Pointer[T]) Swap(new *T) (old *T)`
  * `func (x *Pointer[T]) CompareAndSwap(old, new *T) (swapped bool)`
* atomic.Int32、atomic.Int64、atomic.Uint32、atomic.Uint64、atomic.Uintptr
  * Load
  * Store
  * Swap
  * CompareAndSwap
  * Add
  * And
  * Or
* atomic.Value
  * `func (v *Value) Load() (val any)`
  * `func (v *Value) Store(val any)`
  * `func (v *Value) Swap(new any) (old any)`
  * `func (v *Value) CompareAndSwap(old, new any) (swapped bool)`



### 1.4. 使用示例

int示例：

```go
var a int

func add() {
	a = a + 1
}

func main() {
	for i := 0; i < 500; i++ {
		go func() {
			add()
		}()
	}
	time.Sleep(time.Second)
	println(a)
}

// 输出
// 每次输出不一样，非并发安全
```



atomic.Int32示例：

```go

var a atomic.Int32

func add() {
	a.Add(1)
}

func main() {
	for i := 0; i < 500; i++ {
		go func() {
			add()
		}()
	}
	time.Sleep(time.Second)
	println(a.Load())
}

// 输出
// 每次输出都为500，并发安全
```



## 2. Cond 条件变量
### 2.1. 关于Cond
Cond实现类一个条件变量，用于gorountines等待或者通知事件发生的集合点。
```go
type Cond struct {
	noCopy noCopy // 标识不能被复制

	// L is held while observing or changing the condition
	L Locker // 每个Cond都有一个关联的Locker L（通常是Mutex或RWMutex），在改变条件和调用Wait方法时必须保持它。

	notify  notifyList // 通知列表
	checker copyChecker //复制检查器
}
```
### 2.2. 结构体方法
#### 2.2.1. NewCond构造函数
用于构造一个Cond结构体对象，无需多言~
```go
// NewCond returns a new Cond with Locker l.
func NewCond(l Locker) *Cond {
	return &Cond{L: l}
}
```

#### 2.2.2. Wait等待
Wait方法会以原子地方式解锁`c.L`并暂停调用者goroutine的执行（进入等待状态）；当后续恢复执行时，Wait方法会在返回前锁住`c.L`。
* 除非被Signal或Broadcast方法唤醒，否则等待的方法不会恢复执行。
```go
func (c *Cond) Wait() {
	c.checker.check() // 检查Cond是否被复制
	t := runtime_notifyListAdd(&c.notify) // 添加通知对象
	c.L.Unlock() // 解锁
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock() // 加锁
}
```
需要特别注意的是：由于等待时，c.L是没有上锁的，通常情况下，调用者不能假定当Wait方法返回时，等待的条件是满足的，因此调用者应该在循环中等待。
```go
c.L.Lock()
for !condition() {
	    c.Wait()
}
... make use of condition ...
c.L.Unlock()
```
如果在调用 sync.Cond 类型的 Wait 方法时不在循环中检查条件，可能会导致一些问题，主要是因为条件变量的特性和工作原理：在调用 Wait 方法时，条件变量会释放关联的互斥锁，并阻塞当前协程，直到另一个协程调用 Signal 或 Broadcast 方法来通知条件变量。在通知后，等待的协程会被唤醒，并尝试重新获取互斥锁。如果不在循环中检查条件，等待的协程可能会在条件不满足的情况下被唤醒，然后继续执行后续逻辑，这可能导致程序行为不符合预期。可能出现的问题：

* 竞态条件：如果在等待协程被唤醒后不再检查条件就继续执行后续逻辑，可能会导致竞态条件的发生，从而导致程序出现不确定的行为。

* 条件不满足：如果在唤醒后直接执行后续逻辑而不检查条件，可能会导致程序在条件不满足的情况下继续执行，从而产生错误的结果。

* 协程阻塞：如果不在循环中调用 Wait 方法，等待的协程可能会在条件不满足的情况下被唤醒，但由于条件仍未满足，协程会继续等待，导致协程长时间阻塞。

因此，为了避免这些问题，通常建议在调用 sync.Cond 类型的 Wait 方法时使用循环来检查条件。这样可以确保条件满足时才继续执行后续逻辑，避免竞态条件和不确定行为的发生。
#### 2.2.3. Signal通知
Signal方法会唤醒一个等待的goroutine（如果有的话）。
* 在调用Signal方法时，允许调用者持有c.L锁，但不是必须持有。
```go
func (c *Cond) Signal() {
	c.checker.check()
	runtime_notifyListNotifyOne(&c.notify)
}
```
> 在 Go 语言中，sync.Cond 类型的 Signal 方法会唤醒一个等待在条件变量上的 goroutine，但它并不会影响 goroutine 的调度优先级。这意味着使用 Signal 方法唤醒的 goroutine 和其他正常运行的 goroutine 之间不会有优先级上的差别。
> 此外，当调用 Signal 方法唤醒一个 goroutine 时，并不保证该 goroutine 会立即获取到条件变量关联的互斥锁。其他 goroutine 可能在此时尝试锁定互斥锁，并且它们可能会在被唤醒的 goroutine 之前获取到锁。因此，被唤醒的 goroutine 可能需要等待其他 goroutine 释放锁之后才能继续执行。
> 这种情况可能会导致一些竞态条件的发生，因为被唤醒的 goroutine 并不会立即执行，而是需要等待互斥锁。其他 goroutine 可能会在此期间修改共享状态，从而影响被唤醒的 goroutine 的行为。
> 因此，在使用 sync.Cond 类型时，需要特别注意在调用 Signal 方法后被唤醒的 goroutine 和其他竞争锁的 goroutine 之间可能存在的竞态条件。正确的处理方式应该是在唤醒的 goroutine 中重新检查条件并获取互斥锁，以确保在正确的条件下执行后续逻辑。
#### 2.2.4. Broadcast广播
Broadcast会唤醒所有在 c 上等待的 goroutine。
* 允许但不要求调用者在调用过程中持有 c.L。
```go
func (c *Cond) Broadcast() {
	c.checker.check()
	runtime_notifyListNotifyAll(&c.notify)
}
```
### 2.3. 示例
#### 2.3.1. 实现生产者-消费者模式
可以用Cond来实现生产者、消费者模式。（实际不会这么写，此处只是为了演示Cond的用法）
```go
var (
	mux   = sync.Mutex{} // 互斥锁
	cond  = sync.NewCond(&mux) // 条件变量
	queue []int // 消息队列
)

// producer 生产者
func producer(count int) {
	for i := 0; i < count; i++ {
		mux.Lock() // 获取锁
		queue = append(queue, i+1) // 生产消息
		fmt.Printf("produce %d.\n", i+1)
		cond.Signal() // 通知消费者
		mux.Unlock() // 解锁
	}
}

// consumer 消费者
func consumer() {
	for {
		mux.Lock() // 获取锁
		for len(queue) == 0 { 
			cond.Wait() // 如果没有消息就等待
		}
		fmt.Printf("consume %d.\n", queue[0])
		queue = queue[1:]
		mux.Unlock() // 解锁
	}
}

func main() {
	go producer(10)
	go consumer()

	time.Sleep(time.Second)
}

```
#### 2.3.2. 多协程等待任务完成
可以使用 sync.Cond 让多个协程等待某个任务的完成。任务完成后通过 Broadcast 方法通知所有等待的协程。
```go
var (
	mu    sync.Mutex
	cond  = sync.NewCond(&mu)
	done  bool
)

func worker(id int) {
	mu.Lock()
	for !done {
		cond.Wait()
	}
	fmt.Println("Worker", id, "completed")
	mu.Unlock()
}

func main() {
	for i := 0; i < 5; i++ {
		go worker(i)
	}

	// 模拟任务完成
	mu.Lock()
	done = true
	cond.Broadcast()
	mu.Unlock()

	// 主程序等待一段时间，让协程完成输出
	// 这里只是为了示例，实际情况可能需要更复杂的逻辑
	select {}
}
```

## 3. Map 并发安全的map

### 3.1. 关于sync.Map

`sync.Map`就像普通的`map[any]any`，但是他是并发安全的。另外，加载、存储和删除操作都具有常数时间复杂度。



需要注意的是，大部分代码中，都应该使用普通的Go map，可以配合锁机制来避免并发安全问题。这样可以有更好的类型安全性，且更加容易维护与map内容弄相关的其他不变量。

> **不变量(Invariants)示例:**
> 在实际应用中，除了 map 本身的键值对关系外，可能还需要维护其他约束条件：
>
> * 缓存大小限制
> * 数据一致性约束
> * 业务逻辑相关的状态约束

> **为什么普通 map 更容易维护不变量?**
>
> * 显式控制：可以精确控制何时加锁、解锁，在锁保护下执行复杂的业务逻辑
> * 灵活性：可以根据具体需求选择合适的锁类型（Mutex、RWMutex等）
> * 可组合性：更容易与其他数据结构或业务逻辑集成
> * 类型安全：普通 map 支持泛型，提供编译时类型检查



`sync.Map`比较适用于以下两种场景：

* 给定键只会写入一次，但会读取多次的场景中，比如在只会增长的缓存中。
* 多个goroutine读取、写入和覆盖不相交键集的场景中。

在这两种场景中，与配合锁机制的普通map相比，`sync.Map`可以显著地减少锁竞争。

### 3.2. 内存模型语义

sync.Map 确保写操作"同步发生于"(synchronizes before)任何观察到该写操作效果的读操作之前，这是 Go 内存模型中的一个重要概念。
读写操作分类

读操作 (Read Operations)：

* Map.Load
* Map.LoadAndDelete
* Map.LoadOrStore
* Map.Swap
* Map.CompareAndSwap
* Map.CompareAndDelete



写操作 (Write Operations)：

* Map.Delete
* Map.LoadAndDelete
* Map.Store
* Map.Swap



条件性写操作：

* Map.LoadOrStore: 当返回的 loaded 为 false 时是写操作（即存储了新值）
* Map.CompareAndSwap: 当返回的 swapped 为 true 时是写操作（即成功交换了值）
* Map.CompareAndDelete: 当返回的 deleted 为 true 时是写操作（即成功删除了值）

### 3.3. 初始化

`sync.Map`类型的零值是一个可以直接使用的空map，首次使用后就禁止被复制。

```go
var syncMap sync.Map
syncMap.Store("key", "value")
fmt.Println(syncMap.Load("key"))

// 输出
value true
```



### 3.4. 结构体方法

#### 3.4.1. Load
Load方法返回存储在Map中的指定Key的值。
* 如果Key在Map中不存在，返回nil
* 第二个返回值ok，用于指定Map是否存储了该Key
```go
func (m *Map) Load(key any) (value any, ok bool) {
...
}
```
#### 3.4.2. Store
Store方法项map中设置指定的键值对
* 实际调用的时Swap方法
```go
func (m *Map) Store(key, value any) {
	_, _ = m.Swap(key, value)
}
```
#### 3.4.3. LoadOrStore
LoadOrStore方法用于检查指定Key是否存在并返回Key对应的值。
* 如果指定Key在Map中存在，就返回原来的值。
* 如果指定Key在Map中不存在，就将传入的键值对保存到Map中，并返回传入的值。
* 第二个返回值用于标识原来是否有值（即是否load了原来的值）
```go
func (m *Map) LoadOrStore(key, value any) (actual any, loaded bool) {
...
}
```
#### 3.4.4. LoadAndDelete
LoadAndDelete方法用于加载并删除指定的Key。
* 如果Key在Map中存在，就返回原来的值，并将该键值对从Map中删除。
* 如果Key在Map中不存在，就返回nil。
* 第二个返回值用于标识原来是否有值。
```go
func (m *Map) LoadAndDelete(key any) (value any, loaded bool) {
...
}
```
#### 3.4.5. Delete
删除Map中指定的Key。
* 实际上调用的是LoadAndDelete方法。
```go
func (m *Map) Delete(key any) {
	m.LoadAndDelete(key)
}
```
#### 3.4.6. Swap
Swap方法将指定的键值对存储到Map中，并返回原先存储的值。
* 如果原先无值，就返回nil。
* 第二个返回值用于标识原先是否有值。
```go
func (m *Map) Swap(key, value any) (previous any, loaded bool) {
...
}
```
#### 3.4.7. CompareAndSwap
CompareAndSwap方法比较Map中指定Key的值是否等于old，如果等于old就设置新的值。
* old的值必须是可比较的类型。
* 返回值用于标识是否执行了Swap。
```go
func (m *Map) CompareAndSwap(key, old, new any) bool {
...
}
```
#### 3.4.8. CompareAndDelete
CompareAndDelete方法比较Map中指定Key的值是否等于old，如果等于old就删除指定的键值对。
* old必须是可以比较的类型。
* 返回值用于标识是否执行了删除操作。
* 如果当前Map中没有指定的Key，返回始终返回false，即使传入的old为nil。
```go
func (m *Map) CompareAndDelete(key, old any) (deleted bool) {
...
}
```
#### 3.4.9. Range
Range 为映射中存在的每个键和值依次调用 f 。如果 f 返回 false，则 range 停止迭代。
> Range 不一定对应于 Map 内容的任何一致快照：不会多次访问任何键，但如果同时存储或删除任何键的值（包括通过 f），Range 可能会反映该键的任何映射Range 调用期间的任意点。 Range 不会阻塞接收器上的其他方法；甚至 f 本身也可以调用 m 上的任何方法。
```go
func (m *Map) Range(f func(key, value any) bool) {
...
}
```
### 3.5. 示例
#### 3.5.1. 并发读取和写入
```go
func main() {
	var m sync.Map
	var wg sync.WaitGroup
	numRoutines := 5

	// 并发写入
	for i := 0; i < numRoutines; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			m.Store(i, fmt.Sprintf("value%d", i))
		}(i)
	}

	// 等待所有写入操作完成
	wg.Wait()

	// 并发读取
	for i := 0; i < numRoutines; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			if value, ok := m.Load(i); ok {
				fmt.Printf("Key: %v, Value: %v\n", i, value)
			}
		}(i)
	}

	// 等待所有读取操作完成
	wg.Wait()
}
```

#### 3.5.3. 并发删除
```go
func main() {
	var m sync.Map
	var wg sync.WaitGroup
	numKeys := 5

	// 初始化 sync.Map
	for i := 0; i < numKeys; i++ {
		m.Store(i, fmt.Sprintf("value%d", i))
	}

	// 并发删除
	for i := 0; i < numKeys; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			m.Delete(i)
			fmt.Printf("Deleted key: %v\n", i)
		}(i)
	}

	// 等待所有删除操作完成
	wg.Wait()
}
```


## 4. Mutex互斥锁

### 4.1. Locker锁接口

Locker是所有“锁”的接口，如Mutex、RWMutex等。
> A Locker represents an object that can be locked and unlocked.
```go
type Locker interface {
	Lock() // 获取锁
	Unlock() // 释放锁
}
```

### 4.2. 关于Mutex
Mutex是一个互斥锁。
* Mutex的零值是一个未锁定的互斥锁。
 > 特别注意：Mutex是不可重入的互斥锁！！！
```go
type Mutex struct {
	state int32
	sema  uint32
}
```
### 4.3. 初始化

Mutex的零值是有个可以直接使用并且unlokced的互斥锁，首次使用后禁止复制。

```go
var lock sync.Mutex
lock.Lock()
defer lock.Unlock()
```

### 4.4. 结构体方法

#### 4.4.1. Lock获取锁
Lock方法用于获取互斥锁，如果互斥锁已经被占用，调用者将被阻塞，直到互斥锁可用。
```go
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	// Slow path (outlined so that the fast path can be inlined)
	m.lockSlow()
}
```
#### 4.4.2. TryLock尝试获取锁
TryLock方法会获取互斥锁，并返回是否获取互斥锁成功。
> 请注意，虽然 TryLock 的正确使用确实存在，但很少见，并且 TryLock 的使用通常表明互斥体的特定使用中存在更深层次的问题。
```go
func (m *Mutex) TryLock() bool {
	old := m.state
	if old&(mutexLocked|mutexStarving) != 0 {
		return false
	}

	// There may be a goroutine waiting for the mutex, but we are
	// running now and can try to grab the mutex before that
	// goroutine wakes up.
	if !atomic.CompareAndSwapInt32(&m.state, old, old|mutexLocked) {
		return false
	}

	if race.Enabled {
		race.Acquire(unsafe.Pointer(m))
	}
	return true
}
```
#### 4.4.3. 释放锁
Unlock方法将释放互斥锁（解锁）。
* 如果调用Unlock方法时，互斥锁并没有被锁定，将会产生允许时错误。
* 锁定的互斥锁不与特定的 goroutine 关联。允许一个 Goroutine 锁定一个 Mutex，然后安排另一个 Goroutine 解锁它。
```go
func (m *Mutex) Unlock() {
	if race.Enabled {
		_ = m.state
		race.Release(unsafe.Pointer(m))
	}

	// Fast path: drop lock bit.
	new := atomic.AddInt32(&m.state, -mutexLocked)
	if new != 0 {
		// Outlined slow path to allow inlining the fast path.
		// To hide unlockSlow during tracing we skip one extra frame when tracing GoUnblock.
		m.unlockSlow(new)
	}
}
```
### 4.5. Mutex的互斥公平性

Mutex有两种操作模式：

* normal正常模式
  * 等待获取锁的 goroutine 会按照 FIFO（先进先出）的顺序排队。
  * 当一个被唤醒的等待者尝试获取锁时，它并不拥有锁，而是与新到达的 goroutine 竞争锁的所有权。
  * 新到达的 goroutine 有优势，因为它们已经在 CPU 上运行，并且可能存在大量新到达的 goroutine，因此被唤醒的等待者很可能会失败并重新排队在等待队列的最前面。
  * 如果一个等待者连续尝试获取锁超过1毫秒，那么锁会切换到饥饿模式。
* starvation饥饿模式
  * 锁的所有权会直接从解锁的 goroutine 转移到等待队列最前面的等待者。
  * 新到达的 goroutine 不会尝试获取锁，即使锁看起来是解锁的，也不会自旋等待，而是排队在等待队列的尾部。
  * 如果一个等待者成功获取锁并发现自己是等待队列中的最后一个等待者，或者等待时间少于1毫秒，那么锁会切换回正常操作模式。

> 正常模式的性能要优于饥饿模式，因为在正常模式下，一个 goroutine 可以连续多次获取锁，即使存在被阻塞的等待者。饥饿模式的作用是防止尾部延迟的极端情况。
> Go 标准库中的 sync.Mutex 实现了这样的模式切换机制，以兼顾性能和避免饥饿情况。这种设计可以确保在大多数情况下，正常模式下的性能表现良好，同时避免极端情况下的饥饿问题。

### 4.6. 示例

使用`sync.Mutex`来实现一个简单的并发安全的Map：

```go
type SafeMap struct {
	m map[int]int
	sync.Mutex
}

func (m *SafeMap) Get(key int) int {
	m.Lock()
	defer m.Unlock()
	return m.m[key]
}

func (m *SafeMap) Set(key int, value int) {
	m.Lock()
	defer m.Unlock()
	m.m[key] = value
}
```



## 5. RWMutex读写锁
### 5.1. 关于RWMutex
RWMutex是一个读写互斥锁，该锁可以被任意数量的读锁或者单个写锁持有。

当由一个或多个goroutine持有一个RWMutex的读锁时，任意goroutine调用Lock方法，都会导致后续调用RLock方法的调用者被阻塞，直到写入者获取并释放锁，以确保锁最终可供写入者使用。

```go
type RWMutex struct {
	w           Mutex        // held if there are pending writers
	writerSem   uint32       // semaphore for writers to wait for completing readers
	readerSem   uint32       // semaphore for readers to wait for completing writers
	readerCount atomic.Int32 // number of pending readers
	readerWait  atomic.Int32 // number of departing readers
}
```
### 5.2. 限制说明

**禁止递归读锁定：**

* RWMutex 不支持递归读锁定（recursive read locking）
* 同一个 goroutine 多次调用 RLock **可能**导致死锁或其他未定义行为

**不支持锁升级/降级：**

* RLock 无法升级为 Lock：持有读锁后不能直接获取写锁，必须先释放所有读锁，再获取写锁。
* Lock 无法降级为 RLock：持有写锁后不能直接降级为读锁，必须先释放写锁，再获取读锁。

**设计考量：**
这些限制是为了防止死锁和提高性能：

* 避免复杂的锁状态转换
* 确保写操作能够及时获得锁
* 简化内部实现逻辑

### 5.3. 初始化

RWMutex的零值是一个未锁的互斥锁，首次使用后禁止复制。

```go
rwLock := sync.RWMutex{}
rwLock.RLock()
defer rwLock.RUnlock()
```



### 5.4. 结构体方法

#### 5.4.1. RLock获取读锁
RLock 锁定 rw 以进行读取。它不应该用于递归读锁定；阻塞的 Lock 调用会阻止新的读取者获取锁。
* 如果当前有goroutine持有写锁，则阻塞。
* 如果当前有goroutine持有读锁：
	* 如果有goroutine在等待写锁，则阻塞。
	* 如果无goroutine在等待写锁，则持有读锁。
```go
func (rw *RWMutex) RLock() {
	...
}
```
#### 5.4.2 TryRLock尝试获取读锁
TryRLock尝试获取读锁并返回是否获取成功。
```go
// Note that while correct uses of TryRLock do exist, they are rare,
// and use of TryRLock is often a sign of a deeper problem
// in a particular use of mutexes.
func (rw *RWMutex) TryRLock() bool {
	...
}
```
#### 5.4.3. RUnlock 释放读锁
RUnlock释放单个读锁，对剩余的读者不影响。
* 如果调用时，没有goroutine占有读锁，将会导致一个运行时错误。
```go
func (rw *RWMutex) RUnlock() {
	...
}
```
#### 5.4.4. RLocker获取读锁器
RLocker 返回一个 Locker 接口，通过调用 rw.RLock 和 rw.RUnlock 实现 Lock 和 Unlock 方法。
```go
func (rw *RWMutex) RLocker() Locker {
	return (*rlocker)(rw)
}

type rlocker RWMutex

func (r *rlocker) Lock()   { (*RWMutex)(r).RLock() }
func (r *rlocker) Unlock() { (*RWMutex)(r).RUnlock() }
```
#### 5.4.5. Lock获取写锁
Lock 锁定 rw 以进行写入。如果该锁已被锁定以进行读取或写入，则 Lock 会阻塞，直到该锁可用为止。
```go
func (rw *RWMutex) Lock() {
...
}
```
#### 5.4.6. TryLock尝试获取写锁
TryLock 尝试锁定 rw 进行写入并报告是否成功。
```go
// Note that while correct uses of TryLock do exist, they are rare,
// and use of TryLock is often a sign of a deeper problem
// in a particular use of mutexes.
func (rw *RWMutex) TryLock() bool {
...
}
```
#### 5.4.7. Unlock释放写锁
Unlock 释放写锁。
* 如果 rw 被持有写锁，则会出现运行时错误。
* 与互斥体一样，锁定的 RWMutex 不与特定的 goroutine 关联。一个 goroutine 可以 RLock（Lock）一个 RWMutex，然后安排另一个 goroutine 对其进行 RUnlock（Unlock）
```go
func (rw *RWMutex) Unlock() {
	...
}
```
### 5.5. 示例

使用`sync.RWMutex`实现并发安全的Map：

```go
type SafeMap struct {
	m map[int]int
	sync.RWMutex
}

func NewSafeMap() *SafeMap {
	return &SafeMap{
		m: map[int]int{},
	}
}

func (m *SafeMap) Get(key int) int {
	m.RLock()
	defer m.RUnlock()
	return m.m[key]
}

func (m *SafeMap) Set(key int, value int) {
	m.Lock()
	defer m.Unlock()
	m.m[key] = value
}
```

## 6. Once 单次执行

### 6.1. 关于Once
sync.Once 可以确保在程序运行过程中某个操作只会执行一次，即使被多个 goroutine 同时调用。一旦操作执行完成，后续的调用会立即返回而不再执行。
```go
type Once struct {
	done atomic.Uint32 // 指示动作是否被执行过
	m    Mutex // 互斥锁
}
```
### 6.2. 初始化

`sync.Once`的零值是一个未执行的可以直接使用的值，初次使用后禁止复制。

```go
once := sync.Once{}
once.Do(func() {
  fmt.Println("init")
})
once.Do(func() {
  fmt.Println("init")
})

// 输出
init
```

### 6.3. once.Do()

`sync.Once`只有一个方法`func (o *Once) Do(f func()) `,其入参为一个无参函数。

```go
func (o *Once) Do(f func()) {
	if o.done.Load() == 0 {
		// Outlined slow-path to allow inlining of the fast-path.
		o.doSlow(f)
	}
}
```

对于一个Once，只有在第一次调用Once的Do方法时，才会执行传入的方法。
* 对于一个Once对象，如果调用多次Do方法（不论是否在同一个协程中调用），都只会在第一次调用时，才会执行传入的方法。即使每次传入的参数不同。

**特别注意：**

* 因为**在对 f 的一次调用返回之前，对 Do 的调用不会返回**，所以如果在f中调用Do再次调用Do方法，就会出现死锁。在下面的示例中，外层传入的方法中，又一次调用了once.Do方法，这将导致死锁！

```go
func main() {
	once := sync.Once{}

	once.Do(func() {
		once.Do(func() { 
			fmt.Println("HELLO WORLD")
		})
	})
}
// 输出
fatal error: all goroutines are asleep - deadlock!
```
### 6.4. Once与函数字面量
Do 方法用于需要确保只运行一次的初始化操作。由于被执行的函数 f 是无参数的，因此有时候可能需要使用函数字面量（function literal）来捕获函数的参数，以便在 Do 方法中调用。

>函数字面量是匿名函数的一种形式，可以在声明时直接定义函数体。通过函数字面量，我们可以将需要传递给函数的参数捕获到闭包中，然后在 Do 方法中调用这个函数。
```go
func main() {
	var once sync.Once

	// 准备一个带参数的函数
	initFunc := func(message string) {
		fmt.Println("Initializing with message:", message)
	}

	// 使用函数字面量来捕获参数并传递给初始化函数
	message := "Hello, World!"
	once.Do(func() {
		initFunc(message)
	})

	// 再次调用 Do 方法，函数将不会再次执行
	once.Do(func() {
		fmt.Println("This won't be executed again")
	})
}
```
### 6.5. 使用示例

使用`sync.Once`来控制数据库初始化示例：

```go
import (
	"fmt"
	"sync"
)

// 数据库连接模拟
type Database struct {
	host string
}

func (db *Database) Query(sql string) string {
	return fmt.Sprintf("Executing %s on %s", sql, db.host)
}

var (
	dbInstance *Database
	once       sync.Once
)

// GetDatabaseInstance 获取数据库实例，保证全局只有一个实例
func GetDatabaseInstance() *Database {
	once.Do(func() {
		fmt.Println("Initializing database connection...")
		// 模拟耗时的数据库连接初始化过程
		dbInstance = &Database{
			host: "localhost:5432",
		}
		// 这里的初始化操作只会执行一次
	})
	return dbInstance
}

func main() {
	// 多个goroutine同时请求数据库实例
	var wg sync.WaitGroup

	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func(id int) {
			defer wg.Done()
			db := GetDatabaseInstance()
			result := db.Query(fmt.Sprintf("SELECT * FROM users WHERE id=%d", id))
			fmt.Println(result)
		}(i)
	}

	wg.Wait()
}

// 输出
Initializing database connection...
Executing SELECT * FROM users WHERE id=4 on localhost:5432
Executing SELECT * FROM users WHERE id=2 on localhost:5432
Executing SELECT * FROM users WHERE id=3 on localhost:5432
Executing SELECT * FROM users WHERE id=0 on localhost:5432
Executing SELECT * FROM users WHERE id=1 on localhost:5432
```



### 6.6. Once相关

#### 6.6.1. OnceFunc
OnceFunc 返回一个仅调用 f 一次的函数。返回的函数可以被并发调用。如果 f 发生恐慌，则返回的函数将在每次调用时以相同的值发生恐慌。
```go
func OnceFunc(f func()) func() {
	...
}
```
```go
func main() {
	onceFunc := sync.OnceFunc(func() {
		fmt.Println("Hello World")
	})
	go onceFunc() 
	go onceFunc()
	go onceFunc()
	go onceFunc()
	time.Sleep(time.Second)
}
// 只会打印一次Hello World
```
#### 6.6.2. OnceValue
OnceValue 返回一个仅调用 f 一次的函数，并返回 f 返回的值。返回的函数可以被并发调用。如果 f 发生恐慌，则返回的函数将在每次调用时以相同的值发生恐慌。
* 如果第一次执行没有panic，后续调用会直接返回第一次执行的结果。
```go
func OnceValue[T any](f func() T) func() T {
	···
}
```
```go
func main() {
	onceValue := sync.OnceValue[string](func() string {
		return time.Now().Format("20060102150405")
	})

	print := func() {
		fmt.Println(onceValue())
	}

	go print()
	go print()
	go print()
	time.Sleep(time.Second)
}
// 程序将打印三次同一个结果
```

#### 6.6.3. OnceValues
OnceValues 返回一个仅调用 f 一次的函数，并返回 f 返回的值。返回的函数可以被并发调用。如果 f 发生恐慌，则返回的函数将在每次调用时以相同的值发生恐慌。
*  如果第一次执行没有panic，后续调用会直接返回第一次执行的结果。
```go
func OnceValues[T1, T2 any](f func() (T1, T2)) func() (T1, T2) {
	...
}
```
> 该方法与OnceValue方法差不多，只是返回值的数量为两个而已。

## 7. WaitGroup等待组
### 7.1. 关于WaitGroup
WaitGroup用于等待一组goroutine执行结束。
> 要等待的主线程，通过调用Add方法来设定需要等待的goroutine数量。每个被等待的gorotine在执行完成时调用Done方法。等待的协程将阻塞，直到所有被等待的goroutine执行完成。
```go
type WaitGroup struct {
	noCopy noCopy

  // Bits (high to low):
	//   bits[0:32]  counter
	//   bits[32]    flag: synctest bubble membership
	//   bits[33:64] wait count
	state atomic.Uint64
	sema  uint32
}
```


### 7.2. 初始化

`sync.WaitGroup`的零值是一个可以直接使用的计数器为0的值，初次使用后禁止复制。

```go
func main() {
	wg := sync.WaitGroup{}
	wg.Add(1)
	go func() {
		defer wg.Done()
		fmt.Println("Hello World!")
	}()
	wg.Wait()
}
```

### 7.3. 结构体方法

#### 7.3.1. Add添加等待数量
Add方法向WaitGroup的计数器添加delta个等待数量。
* delta可能为负数。
* 如果WaitGroup的计数器变成0，所有在Wait该WaitGroup都将被唤醒。
* 如果WaitGroup的计数器变成了负数，Add方法将panic。
```go
func (wg *WaitGroup) Add(delta int) {
...
}
```
> 当计数器为0时，Add一个正数的操作应该在Wait操作之前。反之，Wait操作将不会阻塞。
> 当计数器大于0时，Add一个正数或负数可能发生在任何时候。
> 通常，这意味着对 Add 的调用应该在创建 goroutine 或其他要等待的事件的语句之前执行。如果重复使用 WaitGroup 来等待多个独立的事件集，则必须在所有先前的 Wait 调用返回后发生新的 Add 调用。
#### 7.3.2. Done减少一个等待数量
被等待的goroutine需要在执行完成后，调用Done方法来使WaitGroup的计数器减一。从源码可以看出，Done实际上是调用了Add方法。
```go
func (wg *WaitGroup) Done() {
	wg.Add(-1)
}
```
#### 7.3.3. Wait等待
调用Wait方法的goroutine将被阻塞，直到WaitGroup的计数器变成0。
```go
func (wg *WaitGroup) Wait() {
...
}
```
#### 7.3.4. Go

Go 在新的 goroutine 中调用 f 并将该任务添加到 WaitGroup]中。当 f 返回时，该任务将从 WaitGroup 中删除。

```go
func (wg *WaitGroup) Go(f func()) {
	wg.Add(1)
	go func() {
		defer wg.Done()
		f()
	}()
}
```



这个方法可以在我们使用无参函数时，为我们提供便利：

```go
// 原写法
func main() {
	myFunc := func() {
		fmt.Println("Hello, World!")
	}

	wg := sync.WaitGroup{}
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			myFunc()
		}()
	}

	wg.Wait()
}

// 便利写法
func main() {
	myFunc := func() {
		fmt.Println("Hello, World!")
	}

	wg := sync.WaitGroup{}
	for i := 0; i < 5; i++ {
		wg.Go(myFunc)
	}

	wg.Wait()
}
```



### 7.4. 示例

#### 7.4.1. 官方示例
```go
// This example fetches several URLs concurrently,
// using a WaitGroup to block until all the fetches are complete.
func ExampleWaitGroup() {
	var wg sync.WaitGroup
	var urls = []string{
		"http://www.golang.org/",
		"http://www.google.com/",
		"http://www.example.com/",
	}
	for _, url := range urls {
		// Increment the WaitGroup counter.
		wg.Add(1)
		// Launch a goroutine to fetch the URL.
		go func(url string) {
			// Decrement the counter when the goroutine completes.
			defer wg.Done()
			// Fetch the URL.
			http.Get(url)
		}(url)
	}
	// Wait for all HTTP fetches to complete.
	wg.Wait()
}
```

## 8. Pool池
### 8.1. 关于Pool
Pool是一组可以单独保存和检索的临时对象的集合。
* 任何存储在 sync.Pool 中的项都可能在任何时候被自动移除，而没有任何通知。如果在移除操作发生时，sync.Pool 是存储项的唯一引用，那么这个项可能会被释放或回收。
	> 在使用 sync.Pool 时，建议遵循以下最佳实践：
	> * 不依赖于池中对象的存在性，尽量避免在对象被移除后继续使用。
	> * 针对临时对象的重用和性能优化，而不是长期存储。
	> * 在池中存储的对象应该是无状态或者可重置状态的，以便于重复使用。

* 多个 goroutine 同时使用 Pool 是安全的。
* sync.Pool 的作用是缓存已分配但未使用的项，以供以后重复使用，从而减轻垃圾回收器的压力。换句话说，它可以轻松构建高效、线程安全的空闲列表（free list）。但并不适用于所有类型的空闲列表。
* 池的其中一个适当用途是用来管理一组临时项，这些项在并发独立的客户端之间静默共享，并且有可能被重复使用。sync.Pool 提供了一种方式来在许多客户端之间分摊分配开销。
* 在 fmt 包中，sync.Pool 被用来维护一个动态大小的临时输出缓冲区存储。这个存储会根据负载情况进行扩展（当有很多 goroutine 在活跃地进行打印操作时），并在空闲时收缩。
* 对于生命周期短暂的对象，维护一个空闲列表可能会增加额外的开销，不适合使用 sync.Pool。
在这种情况下，最好让对象自己实现空闲列表的管理，以更好地控制资源的分配和释放。
```go
type Pool struct {
	noCopy noCopy

	local     unsafe.Pointer // local fixed-size per-P pool, actual type is [P]poolLocal
	localSize uintptr        // size of the local array

	victim     unsafe.Pointer // local from previous cycle
	victimSize uintptr        // size of victims array

	// New optionally specifies a function to generate
	// a value when Get would otherwise return nil.
	// It may not be changed concurrently with calls to Get.
	New func() any
}
```
### 8.2. 结构体方法
#### 8.2.1. New生成对象
New用于指定一个函数，用于在 Get 方法返回 nil 时生成一个值。同时指出，New 方法不应该在并发调用 Get 方法的情况下被同时修改。
> 在使用 sync.Pool 时，可以通过 New 方法指定一个函数，在 Get 方法无法返回值时，用于生成一个新的值。这样可以避免返回 nil，并且可以根据需要动态地生成新的值。需要注意的是，在调用 Get 方法的同时，不应该并发地修改 New 方法，以避免出现竞争条件和不确定的行为。

> New 方法为 sync.Pool 提供了一种灵活的方式来生成值，从而确保在需要时能够获取到有效的值。
```go
type Pool struct {
	...

	// New optionally specifies a function to generate
	// a value when Get would otherwise return nil.
	// It may not be changed concurrently with calls to Get.
	New func() any
}
```
#### 8.2.2. Put
Put方法用于向池中添加以一个临时对象x。
```go
func (p *Pool) Put(x any) {
	...
}
```
#### 8.2.3. Get
Get 方法用于从池中获取一个项，并将其移除。需要注意的是，Get 方法可能会忽略池中的内容并返回一个新的项。此外，调用方不应该假设放入池中的值与 Get 返回的值之间存在任何关系，因为 Get 方法的行为是无法预测的。
```go
func (p *Pool) Get() any {
	...
}
```
### 8.3. 示例
#### 8.3.1. 官方示例
```go
var bufPool = sync.Pool{
	New: func() any {
		// The Pool's New function should generally only return pointer
		// types, since a pointer can be put into the return interface
		// value without an allocation:
		return new(bytes.Buffer)
	},
}

// timeNow is a fake version of time.Now for tests.
func timeNow() time.Time {
	return time.Unix(1136214245, 0)
}

func Log(w io.Writer, key, val string) {
	b := bufPool.Get().(*bytes.Buffer)
	b.Reset()
	// Replace this with time.Now() in a real logger.
	b.WriteString(timeNow().UTC().Format(time.RFC3339))
	b.WriteByte(' ')
	b.WriteString(key)
	b.WriteByte('=')
	b.WriteString(val)
	w.Write(b.Bytes())
	bufPool.Put(b)
}

func ExamplePool() {
	Log(os.Stdout, "path", "/search?q=flowers")
	// Output: 2006-01-02T15:04:05Z path=/search?q=flowers
}

```
