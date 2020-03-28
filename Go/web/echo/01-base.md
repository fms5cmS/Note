下载：`go get github.com/labstack/echo/v4`

在 go.mod 文件中加入 ：`replace golang.org/x/sys => github.com/golang/sys v0.0.0-20190830142957-1e83adbbebd0`

# Hello World

```go
import (
	"github.com/labstack/echo/v4"
	"net/http"
)

func main() {
	e := echo.New()
	e.GET("/", index)
	e.Logger.Fatal(e.Start(":1321"))
}

func index(c echo.Context) error {
	// 发送一个带有状态码的简单 HTML 响应
	return c.HTML(http.StatusOK,"<h1>Welcome Index Page!</h1>")
}m
```

# 路由

## GET

### 查询参数

访问路径：`http://localhost:1321/user?id=1&name=zzk`

```go
// 字段首字母必须大写
// 字段不是必须加 tag，如 `json: "id"`
type User struct {
	Id int
	Name string
}

func main() {
	e := echo.New()
	e.GET("/user", desc)
	e.Logger.Fatal(e.Start(":1321"))
}
func desc(c echo.Context) error {
	id := c.QueryParam("id")
	name := c.QueryParam("name")
	// 发送一个带状态码的 JSON 对象，会将 Golang 的对象转换成 JSON 字符串。
	return c.JSON(http.StatusOK, User{
		Id:   id,
		Name: name,
	})
}
```

### 路径参数

访问路径：`http://localhost:1321/users/2`

```go
func main() {
	e := echo.New()
	// 路径参数 http://localhost:1321/users/2
	e.GET("/users/:id", getUser)
	e.Logger.Fatal(e.Start(":1321"))
}
func getUser(c echo.Context) error {
	id := c.Param("id")
	// 发送一个带有状态码的纯文本响应
	return c.String(http.StatusOK,"Uid = "+id)
}
```

## POST

### 表单参数

```go
func main() {
	e := echo.New()
	e.POST("/user", save)
	e.Logger.Fatal(e.Start(":1321"))
}

func save(c echo.Context) error {
	name := c.FormValue("name")
	email := c.FormValue("email")
	return c.String(http.StatusOK, "name: "+name+"\nemail: "+email)
}
```

这里使用 Goland 自带的 Restful Client 来测试

```http
POST http://localhost:1321/user
Content-Type: application/x-www-form-urlencoded

name=bx&email="bx@163.com"

###
```
