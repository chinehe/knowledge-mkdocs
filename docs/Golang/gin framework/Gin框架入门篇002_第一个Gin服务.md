## 1. 准备工作

* 安装Go：安装合适版本的Go。

## 2. 第一个gin服务

1. 初始化项目

   1. 创建项目文件夹

      ```shell
      # 创建并进入项目文件夹
      mkdir gin-quickstart && cd gin-quickstart
      ```

   2. 初始化项目

      ```shell
      # 初始化项目
      go mod init gin-quickstart
      ```

2. 导入gin框架

   ```shell
   # 引入gin依赖
   go get -u github.com/gin-gonic/gin
   ```

3. 编写第一个gin服务

   1. 创建`main.go`文件

   2. 编写服务代码

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

4. 运行

   ```shell
   # 启动gin服务
   go run main.go
   ```

5. 测试

   1. 打开浏览器，访问[`http://localhost:8080/ping`](http://localhost:8080/ping)地址。
   2. 预期结果`{"message":"pong"}`

