在Web开发中，模板渲染是将动态数据嵌入到HTML页面中的关键功能。Gin框架提供了强大且易用的HTML模板渲染功能，基于Go语言内置的`html/template`包实现。本文将详细介绍Gin框架的HTML模板渲染机制及其使用方法。

## 1. 模板加载

在使用Gin框架进行HTML模板渲染之前，我们需要先了解如何加载模板文件。

Gin框架提供了多种模板加载方式，以满足不同场景的需求:

* `LoadHTMLGlob`：加载符合`glob`模式（glob是入参）的HTML文件，并将结果与HTML渲染器关联。
* `LoadHTMLFiles`：加载一个或者多个HTML模板文件，并将结果与HTML渲染器关联。
* `LoadHTMLFS`：从指定的文件系统加载符合pattern`模式的HTML模板文件，并将结果与HTML渲染器关联。

### 1.1. LoadHTMLGlob 模式匹配加载多个模板

`LoadHTMLGlob`方法可以加载符合指定模式的模板文件，使用通配符进行匹配：



示例：

```go
func main() {
    r := gin.Default()
    
    // 加载templates目录下所有模板文件
    r.LoadHTMLGlob("templates/**/*")
    
    r.Run(":8080")
}
```

### 1.2. LoadHTMLFiles 加载单个/多个模板

使用`LoadHTMLFiles`可以加载一个或者多个HTML模板文件。



示例：

```go
func main() {
    r := gin.Default()
    
    // 加载指定模板文件
    r.LoadHTMLFiles(
        "./templates/user/profile.tmpl",
        "./templates/user/detail.tmpl",
        "./templates/post/index.tmpl",
    )
    
    r.Run(":8080")
}
```



### 1.3. LoadHTMLFS 从指定文件系统加载

使用`LoadHTMLFiles`可以从指定文件系统加载HTML模板文件。

源码：

```go
func (engine *Engine) LoadHTMLFS(fs http.FileSystem, patterns ...string) {
	if IsDebugging() {
		engine.HTMLRender = render.HTMLDebug{FileSystem: fs, Patterns: patterns, FuncMap: engine.FuncMap, Delims: engine.delims}
		return
	}

	templ := template.Must(template.New("").Delims(engine.delims.Left, engine.delims.Right).Funcs(engine.FuncMap).ParseFS(
		filesystem.FileSystem{FileSystem: fs}, patterns...))
	engine.SetHTMLTemplate(templ)
}
```





示例：

```go
func main() {
    r := gin.Default()
    
    // 从指定文件系统加载模板文件
    r.LoadHTMLFS(http.Dir("./templates"), "./user/*", "./post/*")
    
    r.Run(":8080")
}
```

## 2. 模板渲染

Gin框架提供了`c.HTML`方法来渲染HTML模板。该方法接收HTTP状态码、模板名称和传递给模板的数据。

> `c.HTML`方法是Gin框架渲染HTML页面的核心方法，它会自动设置Content-Type为text/html。



以下是一个基本的HTML模板渲染示例：

* 工程目录

  * templates

    * index.tmpl

      ```html
      <!DOCTYPE html>
      <html>
      <head>
          <title>{{.title}}</title>
      </head>
      <body>
          <h1>{{.content}}</h1>
          <p>当前时间：{{.time}}</p>
      </body>
      </html>
      ```

  * main.go

    ```go
    package main
    
    import (
        "github.com/gin-gonic/gin"
        "net/http"
        "time"
    )
    
    func main() {
        r := gin.Default()
        
        // 加载模板文件
        r.LoadHTMLGlob("templates/*")
        
        r.GET("/index", func(c *gin.Context) {
            // 渲染模板
            c.HTML(http.StatusOK, "index.tmpl", gin.H{
                "title": "首页",
                "content": "欢迎使用Gin框架",
                "time": time.Now(),
            })
        })
        
        r.Run(":8080")
    }
    ```



项目启动后，浏览器访问`http://127.0.0.1:8080/index`,将看到渲染后的HTML页面。

## 3. 进阶使用

### 3.1. 自定义模板渲染器

gin提供了`SetHTMLTemplate`方法供我们自定义模板渲染器。

你可以使用自定义的html模板渲染。



示例：

```go
import "html/template"

func main() {
  router := gin.Default()
  html := template.Must(template.ParseFiles("file1", "file2"))
  router.SetHTMLTemplate(html)
  router.Run(":8080")
}
```



### 3.2. 自定模板分隔符

gin提供了`Delims`方法供我们设置模板的左右分隔符。默认左分隔符为`{{`，默认右分隔符为`}}`。



示例：

```go
  router := gin.Default()
  router.Delims("{[{", "}]}")
  router.LoadHTMLGlob("/path/to/templates")
```



### 3.3. 自定义模板功能（模板函数）

gin提供了`SetFuncMap`方法来允许我们自定义模板函数。



示例：

* 工程目录

  * templates

    * raw.tmpl

      ```html
      Date: {[{.now | formatAsDate}]}
      ```

  * main.go

    ```go
    package main
    
    import (
    	"fmt"
    	"html/template"
    	"net/http"
    	"time"
    
    	"github.com/gin-gonic/gin"
    )
    
    func formatAsDate(t time.Time) string {
    	year, month, day := t.Date()
    	return fmt.Sprintf("%d%02d/%02d", year, month, day)
    }
    
    func main() {
    	router := gin.Default()
    	router.Delims("{[{", "}]}")
    	router.SetFuncMap(template.FuncMap{
    		"formatAsDate": formatAsDate,
    	})
    	router.LoadHTMLFiles("./templates/raw.tmpl")
    
    	router.GET("/raw", func(c *gin.Context) {
    		c.HTML(http.StatusOK, "raw.tmpl", gin.H{
    			"now": time.Date(2017, 0o7, 0o1, 0, 0, 0, 0, time.UTC),
    		})
    	})
    
    	_ = router.Run(":8080")
    }
    ```

    