下载：`go get -u github.com/gin-gonic/gin`

# Hello World

```go
import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
  // 返回 Gin 的 Engine，包含了 RouterGroup，相当于创建一个路由 Handlers，可以后期绑定各类路由规则和函数、中间件等
  router := gin.Default()
	router.GET("/test", func(c *gin.Context) {
    // gin.H 就是一个 map[string]interface{}
		c.JSON(http.StatusOK, gin.H{"message": "test"})
	})
	s := &http.Server{
		Addr:     ":1321",
		Handler:  router,
	}
	s.ListenAndServe()
}
```

注意：查看 `gin.Default()` 源码，会在程序启动时打印 Warning 信息，且会添加 `Logger()`、`Recovery()` 中间件。

中间件使用见[02-中间件](02-中间件.md)。

# 路由

## 注册路由

```go
func InitRouter() *gin.Engine {
	r := gin.New()
	r.Use(gin.Logger())
	gin.SetMode("debug")
	apiv1 := r.Group("/api/v1")
	{
		// 获取标签列表，后面是对应的业务逻辑
		apiv1.GET("/tags", v1.GetTags)
		// 新建标签
		apiv1.POST("/tags", v1.AddTag)
	}
	return r
}
```

## GET

### 查询参数

访问路径：`http://localhost:1321/test?first_name=b&last_name=x`

```go
r.GET("/test", func(c *gin.Context) {
  // 获取 URL 中携带的查询参数
  firstName := c.Query("first_name")
  // 获取 URL 中携带的查询参数，如果没有，则使用默认值
  lastName := c.DefaultQuery("last_name", "default_last_name")
  c.String(http.StatusOK, "%s_%s", firstName, lastName)
})
```

### 路径变量

访问路径：`http://localhost:8080/zzk/24`

```go
r.GET("/user/:name/:id", func(c *gin.Context) {
  c.JSON(200,gin.H{
    "name": c.Param("name"),
    "id":   c.Param("id"),
  })
})
```

## POST

### 表单参数

```go
r.POST("/body", func(c *gin.Context) {
  firstName := c.PostForm("first_name")
  lastName := c.DefaultPostForm("last_name", "default_last_name")
  c.String(http.StatusOK, "%s %s", firstName, lastName)
})
```

# 上传文件

```go
func (c *gin.Context) {
	// 获取上传的文件
  file, err := c.FormFile("file")
  if (err != nil) {
    c.JSON(http.StatusInternalServerError, gin.H{
      "msg": err.Error(),
    })
    return
	}
	// todo 检查文件相关信息(格式、大小、权限等)
	dst := fmt.Sprintf("/uploads/%s", file.Filename)
	// 保存文件到指定的 dst 目录
  c.SavaeUpLoadedFile(file, dst)
  c.JSON(http.StatusOK, gin.H{
    "msg":"ok",
  })
}
```

# 静态资源

```go
  // 设置静态文件夹绑定，有以下两种写法
  r.Static("assets", "static/assets")
  r.StaticFS("/static",http.Dir("static/template"))
  // 设置静态文件
  r.StaticFile("/favicon.ico","static/favicon.ico")
```