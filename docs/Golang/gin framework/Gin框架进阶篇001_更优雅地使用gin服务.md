## 1. 最简单的Gin服务

一个最简单的Gin服务如下所示：

```go
package main

import (
	"net/http"

	// 导入gin框架
	"github.com/gin-gonic/gin"
)

func main() {
	// 创建默认的gin路由
	router := gin.Default()
	
	// 定义一个简单的GET端点
	router.GET("/ping", func(c *gin.Context) {
		// 返回JSON数据
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})
	
	// 启动服务（默认端口8080）
	err := router.Run()
	if err != nil {
		return
	} 
}
```

## 2. 自定义HTTP配置

在最简单的示例中，我们使用`router.Run()`方法来启动gin服务，该方法的源码如下：

```go
func (engine *Engine) Run(addr ...string) (err error) {
	defer func() { debugPrintError(err) }()

	if engine.isUnsafeTrustedProxies() {
		debugPrint("[WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.\n" +
			"Please check https://github.com/gin-gonic/gin/blob/master/docs/doc.md#dont-trust-all-proxies for details.")
	}
	engine.updateRouteTrees()
	address := resolveAddress(addr)
	debugPrint("Listening and serving HTTP on %s\n", address)
	err = http.ListenAndServe(address, engine.Handler())
	return
}
```

从源码可以看到，该方法实际上是调用了`http.ListenAndServe(address, engine.Handler())`方法。

```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```

可以看到，`ListenAndServe`方法中使用的是最基础的`Server`配置。



我们可以通过如下的方式来自定义HTTP服务配置：

```go

func main() {
	// 创建默认的gin路由
	router := gin.Default()

	// 定义一个简单的GET端点
	router.GET("/ping", func(c *gin.Context) {
		// 返回JSON数据
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})

	// 启动服务（默认端口8080）
	server := &http.Server{
		Addr:           ":8080",
		Handler:        router,
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	server.ListenAndServe()
}

```

## 3. 优雅重启与停止

如果使用Go1.8或者更高的版本，可以使用`http.Server`内置的`Shutdown`方法来实现服务的优雅停止。

`Shutdown`方法会优雅地关闭服务器，不会中断任何活动连接。

* 首先关闭所有打开的监听器，然后关闭所有空闲连接，最后无限期地等待连接变为空闲并关闭。
* 如果提供的上下文在关闭完成之前过期，将返回上下文的错误，否则返回关闭服务器底层监听器时产生的任何错误。
* 调用Shutdown后，`Serve`、`ServeTLS`、`ListenAndServe]`和 `ListenAndServeTLS`等方法会立即返回 `ErrServerClosed`。一旦服务器调用了 Shutdown，就不能再次使用；后续调用 Serve 等方法将返回 ErrServerClosed。
* Shutdown 不会尝试关闭或等待被劫持的连接（如 WebSocket）。如果需要，Shutdown 的调用者应该单独通知这些长连接即将关闭，并等待它们关闭。

### 3.1. 优雅关闭 - 使用context

```go
// 创建默认的gin路由
router := gin.Default()

// 定义一个简单的GET端点
router.GET("/ping", func(c *gin.Context) {
  time.Sleep(10 * time.Second)
  // 返回JSON数据
  c.JSON(http.StatusOK, gin.H{
    "message": "pong",
  })
})

// 启动服务
server := &http.Server{
  Addr:    ":8080",
  Handler: router,
}
// 在单独的协程中启动服务
go func() {
  if err := server.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
    log.Fatalf("listen: %s\n", err)
  }
}()

// 优雅关闭
ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
defer stop()
// 等待信号
<-ctx.Done()

// 恢复中断信号的默认行为并通知用户
stop()
log.Println("shutting down gracefully, press Ctrl+C again to force")

// 等待五秒
timeoutCtx, cancelFunc := context.WithTimeout(context.Background(), 5*time.Second)
defer cancelFunc()
if err := server.Shutdown(timeoutCtx); err != nil {
  log.Fatalf("Server Shutdown: %v", err)
}

log.Println("Server exiting")
```

### 3.2. 优雅关闭 - 不使用context

```go
// 创建默认的gin路由
router := gin.Default()

// 定义一个简单的GET端点
router.GET("/ping", func(c *gin.Context) {
  time.Sleep(10 * time.Second)
  // 返回JSON数据
  c.JSON(http.StatusOK, gin.H{
    "message": "pong",
  })
})

// 启动服务
server := &http.Server{
  Addr:    ":8080",
  Handler: router,
}
// 在单独的协程中启动服务
go func() {
  if err := server.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
    log.Fatalf("listen: %s\n", err)
  }
}()

// 优雅关闭
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)

// 等待信号
<-quit

// 恢复中断信号的默认行为并通知用户
signal.Stop(quit)
log.Println("Shutdown Server ...")

// 等待5秒
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
if err := server.Shutdown(ctx); err != nil {
  log.Fatal("Server Shutdown:", err)
}
log.Println("Server exiting")
```

## 4. 启动多个Gin服务

有时，我们需要在一个go应用中启动多个Web服务。

下面的示例中演示了如何在一个应用中启动多个Gin服务（包含优雅停止）：

```go
package main

import (
	"context"
	"errors"
	"log"
	"net/http"
	"os/signal"
	"sync"
	"syscall"
	"time"

	"github.com/gin-gonic/gin"
)

func main() {
	// 启动平台服务
	platServer := StartPlatServer()

	// 启动静态服务
	staticServer := StartStaticServer()

	// 优雅关闭
	ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer stop()

	// 等待退出信号
	<-ctx.Done()

	// 恢复中断信号的默认行为并通知用户
	stop()
	log.Println("shutting down gracefully, press Ctrl+C again to force")

	// 等待五秒
	timeoutCtx, cancelFunc := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancelFunc()

	// 关闭服务
	wg := sync.WaitGroup{}
	wg.Add(2)
	go func() {
		if err := platServer.Shutdown(timeoutCtx); err != nil {
			log.Fatalf("plat server shutdown error: %s\n", err)
		}
		log.Println("plat server shutdown")
		wg.Done()
	}()
	go func() {
		if err := staticServer.Shutdown(timeoutCtx); err != nil {
			log.Fatalf("static server shutdown error: %s\n", err)
		}
		log.Println("static server shutdown")
		wg.Done()
	}()

	wg.Wait()
	log.Println("Server exiting")
}

// StartPlatServer 启动平台服务
func StartPlatServer() *http.Server {
	// 创建一个默认的路由
	platRouter := gin.Default()
	// 添加路由
	platRouter.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "Hello Plat",
		})
	})

	// 启动服务
	platServer := &http.Server{
		Addr:    ":8080",
		Handler: platRouter,
	}
	go func() {
		if err := platServer.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
			log.Fatalf("plat server error: %s\n", err)
		}
	}()

	return platServer
}

// StartStaticServer 启动静态服务
func StartStaticServer() *http.Server {
	// 创建一个默认的路由
	staticRouter := gin.Default()
	// 添加路由
	staticRouter.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "Hello Static",
		})
	})

	// 启动服务
	staticServer := &http.Server{
		Addr:    ":8081",
		Handler: staticRouter,
	}
	go func() {
		if err := staticServer.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
			log.Fatalf("static server error: %s\n", err)
		}
	}()

	return staticServer
}

```

## 5. Gin服务的模式

Gin框架提供了几种不同的运行模式，主要区别在于日志记录和恢复行为。

### 5.1. Gin的三种模式

Gin服务的主要模式

* DebugMode（调试模式）
  * 默认模式，提供详细的日志输出，包含完整的错误堆栈跟踪。
* ReleaseMode（发布模式）
  * 生产环境中推荐使用的模式、最小化日志输出、提高性能。
* TestMode（测试模式）
  * 主要用于测试场景、不会输出日志、方便进行自动化测试。

### 5.2. 模式切换

**默认使用Debug模式：**

gin默认使用debug模式运行。

```go
func main() {
	fmt.Println(gin.Mode()) // debug
}
```



**`SetMode`方法切换运行模式：**

使用gin.SetMode方法可以切换运行模式，模式设置必须在创建任何 gin.Engine 实例之前完成。

```go
import "github.com/gin-gonic/gin"

func main() {
    // 在创建任何 gin 实例之前设置
    gin.SetMode(gin.ReleaseMode)
    
    // 后续创建的路由器将使用 ReleaseMode
    router := gin.New()
    // 或者
    router := gin.Default()
}

```



**通过环境变量设置：**

还可以通过设置环境变量`GIN_MODE`的值来切换运行模式。

```shell
## 设置环境变量
export GIN_MODE=release

## 然后运行应用程序
go run main.go
```

```shell
## 直接运行时设置环境变量
GIN_MODE=release go run main.go

## 或者在构建后运行
GIN_MODE=release ./your-app
```

### 5.3. 三种模式的区别



| 特性/模式 | Debug模式                    | Release模式        | Test模式           |
| --------- | ---------------------------- | ------------------ | ------------------ |
| 日志输出  | 详细日志，包含路由匹配等信息 | 最小化日志输出     | 几乎无日志输出     |
| 错误处理  | 显示详细的错误堆栈信息       | 只记录关键错误信息 | 错误信息被抑制     |
| 性能影响  | 性能较低                     | 性能最优           | 性能较好           |
| 使用场景  | 开发调试阶段                 | 生产环境部署       | 单元和集成测试阶段 |

