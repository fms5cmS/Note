Go 语言库常用的一个设计套路：一般来说，一个库实现某个功能，通过一个结构体+方法的形式来做。而为了使用方便，会在包级别定义和结构体方法一致的函数，使用结构体默认的实例。

# 配置文件相关

[viper](https://github.com/spf13/viper)，支持 JSON, TOML, YAML, HCL, envfile and Java properties 等多种格式的配置文件，但不支持 ini 格式。严格而言，viper 并不解析具体格式，解析的工作根据实际的格式交给对应的库。为所有支持的格式提供统一的 API，这让使用者不必关心具体是什么格式，同时换配置文件格式代码几乎不用修改。

configfile.toml：

```toml
language = "Go"
host = "localhost"
port = 10086

# 个人信息
[person]
name = "zzk"
age = 17
h = 170

[family]
address = "sx"
```

读取配置：

```go
type config struct {
	Language string
	Host     string
	Port     int
	People   Person `mapstructure:"Person"`  // 指定内嵌的结构体类型
}

type Person struct {
	Name   string
	Age    int
	Height int `mapstructure:"h"` // 字段名与配置文件中的 key 不一致时，指定配置文件中的 key
}

type Family struct {
	Address string
}

var v = viper.New()


func TestReadConfig(t *testing.T) {
	// 设置配置文件名，不需要带扩展名，便于在不修改代码的情况下替换配置文件的类型
	v.SetConfigName("configfile")
	// 设置配置文件类型(可选，viper 会自动判断)
	v.SetConfigType("toml")
	// 可以添加多个配置文件的路径
	v.AddConfigPath("../")
	v.WatchConfig()  // 监控配置文件
	v.OnConfigChange(func(e fsnotify.Event) {
		fmt.Println("config file changed: ", e.Name)
		fill()
	})
	fill()
}

func fill() {
	// 读取配置文件，会根据不同的文件类型调用不同的解析库进行解析
	if err := v.ReadInConfig(); err != nil {
		if _, ok := err.(viper.ConfigFileNotFoundError); ok {
			panic(fmt.Errorf("Can not find config file"))
		} else {
			panic(fmt.Errorf("Fatal error config file: %s", err))
		}
	}
	language := v.GetString("language")
	age := v.GetInt("person.age") // 读取配置文件中的嵌入字段
	fmt.Printf("language = %d, language = %s\n", age, language)
	// 将从配置文件中的读取到的配置信息填充到结构体中(含内嵌的结构体！)
	var conf config
	if err := v.Unmarshal(&conf); err != nil {
		panic("error 1")
	}
	fmt.Printf("%v\n", conf)
	var family Family
	if err := v.UnmarshalKey("family", &family); err != nil {
		panic("error 2")
	}
	fmt.Printf("%v\n", family)
}
```

- 注意

当同一个 Key 有多种方式设置值时，其优先顺序为(以下为降序)：

1. 显示设置，即 `viper.Set` 方式硬编码（也叫 overrides）
2. flag 方式，即通过 BindFlag 相关函数设置
3. 环境变量，即通过 BindEnv 或 AutomaticEnv 等函数设置
4. 配置文件
5. key/value 远程存储，支持 ectd 和 consul
6. 默认值，即通过 SetDefault 设置

---

[go-ini](https://ini.unknwon.io/)，从 ini 格式的文件中读取配置。

# ORM

对象关系映射(ORM)。[GORM](https://gorm.io/) 和 Xorm 是 Go 中最流行的两个 ORM 库，前者更常用。

```shell
# 下载 gorm
go get -u github.com/jinzhu/gorm
# 下载 mysql 驱动
go get -t -v github.com/go-sql-driver/mysql/...
```

## 全局 DB

注：代码中需导入数据库驱动！以 MySQL 为例如：

```go
import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/jinzhu/gorm"
	"github.com/sirupsen/logrus"
	"github.com/spf13/viper"
	"sync"
)

var (
	db   *gorm.DB
	once = new(sync.Once)
)

func init() {
	once.Do(func() {
		var (
			host, user, password, dbname, url string
			err error
		)
		viper.SetConfigName("conf")
		viper.AddConfigPath("./")
		if err := viper.ReadInConfig(); err != nil {
			logrus.Errorf("load config error: %v", err)
		}
		host = viper.GetString("mysql.host")
		user = viper.GetString("mysql.user")
		password = viper.GetString("mysql.password")
		dbname = viper.GetString("mysql.dbname")
		url = fmt.Sprintf("%s:%s@(%s)/%s?charset=utf8&parseTime=True&loc=Local", user, password, host, dbname)
		db, err = gorm.Open("mysql", url)
		if err != nil {
			panic("failed to connect database")
		}
		// 开启日志
		db.LogMode(true)
		// 全局禁用表名复数,如果设置为 true, `User` 的默认表名为 `user`,否则表名就是 `users`
		db.SingularTable(true)
	})
}
```

## struct

- 用于保存查询结果

```go
type Hero struct {
	ID          uint64
	Name        string
	RoleMain    string
  // 这里的标签用于指定该字段对应表中哪个字段
  // 默认情况示例：RoleMain 对应表中的 role_main
	Birthday    time.Time `gorm:"column:birthdate"`
	HpMax       float32
	MpMax       float32
	AttackMax   float32
	DefenseMax  float32
	AttackSpeed float32 `gorm:"column:attack_speed_max"`
	AttackRange string
}
```

- 用于记录查询条件

```go
type heroArgs struct {
	ID       uint64
	Name     string
	RoleMain string
}
```

## 填充查询结果

填充时只能向 struct 和 slice 中填充

- `First()` 仅查询符合条件的第一条记录
- `Last()` 进查询符合条件的最后一条记录

由于创建返回结构体时使用 `new()` 来创建的，所以填充时不用取地址，否则要使用 `&hero`！

```go
func (h *heroArgs) QueryFirst() (*Hero, error) {
	hero := new(Hero)
	// SELECT * FROM `heros` WHERE (id >= 1009) ORDER BY `heros`.`id` ASC LIMIT 1
	err := db.Table("heros").Where("id >= ?", h.ID).First(hero).Error
	return hero, err
}

func (h *heroArgs) QueryLast() (*Hero, error) {
	hero := new(Hero)
	// SELECT * FROM `heros`  WHERE (role_main = '法师') ORDER BY `heros`.`id` DESC LIMIT 1
	err := db.Table("heros").Where("role_main = ?", h.RoleMain).Last(hero).Error
	return hero,err
}
```

- `Find()` 查询所有符合条件的记录，注意是 `Find(&heroes)`

```go
func (h *heroArgs) QueryFind() ([]*Hero, error) {
	heroes := make([]*Hero, 0)
	// SELECT * FROM `heros`  WHERE (role_main = '战士' AND id > 10029)
	err := db.Table("heros").Where("role_main = ? AND id > ?", 
                                 h.RoleMain, h.ID).Find(&heroes).Error
	return heroes, err
}
```

## 查询条件

- `Where()` 中指定查询条件

```go
db.Where("name IN (?)", []string{"jinzhu", "jinzhu 2"}).Find(&users)
db.Where("name LIKE ?", "%jin%").Find(&users)
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// 也可以传入结构体或 map
// 注意：传入结构体时，gorm 仅使用字段值为**非零值**的字段作为查询条件！！
// SELECT * FROM users WHERE name = "jinzhu";
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;
db.Where(map[string]interface{}{"name": "jinzhu", "age": 0}).Find(&users)
```

- `NOT()`、`Or()` 不常使用，见[官网](https://v1.gorm.io/docs/query.html)

## 其他

- `Order()` 排序，注意：该方法必须是在填充结果之前！

```go
// SELECT * FROM users ORDER BY age desc, name;
db.Order("age desc, name").Find(&users)
// 重排序
// users1：SELECT * FROM users ORDER BY age desc;
// users2：SELECT * FROM users ORDER BY age;
db.Order("age desc").Find(&users1).Order("age", true).Find(&users2)
```

- `Group()`、`Having()` 分组

```go
func (h *heroArgs) QueryGroupByRole() ([]int, []string) {
	count, role := make([]int, 0), make([]string, 0)
	// SELECT COUNT(1) num, role_main FROM heros GROUPBY role_main HAVING (num > 10)
	rows, _ := db.Table("heros").Select("COUNT(1) num, role_main").Group("role_main").Having("num > ?", 10).Rows()
	for rows.Next() {
		c, r := 0, ""
		rows.Scan(&c, &r)
		count = append(count, c)
		role = append(role, r)
	}
	return count, role
}

type Result struct {
	Num  int
	Role string
}

func (h *heroArgs) QueryGroup() ([]Result, error) {
	results := make([]Result, 0)
	// SELECT count(1) num, role_main role FROM `heros`   GROUP BY role
  // 以下两种写法都可以
	// err := db.Table("heros").Select("count(1) num, role_main role").Group("role").Scan(&results).Error
	err := db.Table("heros").Select("count(1) num, role_main role").Group("role").Find(&results).Error
	return results, err
}
```

- `Limit()` 指定查询多少条记录

```go
// SELECT * FROM users LIMIT 3;
db.Limit(3).Find(&users)
```

- `Offset()` 指定跳过多少条记录

```go
// SELECT * FROM users OFFSET 3;
db.Offset(3).Find(&users)
```

- `SubQuery()` 子查询

```go
func (h *heroArgs) QuerySub() ([]*Hero, error) {
	heroes := make([]*Hero, 0)
  // SELECT * FROM `heros`  WHERE (id > (SELECT AVG(id) FROM `heros`)) 
	err := db.Table("heros").Where("id > ?",db.Table("heros").Select("AVG(id)").SubQuery()).
  	Find(&heroes).Error
	return heroes, err
}
```

- `Count()` 计数

```go
func (h *heroArgs) QueryManyAndCount() ([]*Hero, int, error) {
	heroes := make([]*Hero, 0)
	count := 0
  // SELECT * FROM `heros`  WHERE (id > 10029)
  // SELECT count(*) FROM `heros`  WHERE (id > 10029)
	err := db.Table("heros").Where("id > ?", h.ID).Find(&heroes).Count(&count).Error
	return heroes, count, err
}
```

# 日志相关

[日志记录指导原则](../通用/Log.md)

Go 语言标准库自带一个日志库：log，但这个库没有日志级别的概念，同时不支持结构化的日志。主流的三个日志库：

[Logrus](https://github.com/sirupsen/logrus)使用简单，且兼容标准库 log，但性能一般，其并发安全是通过 mutex 实现的，如果确定不需要锁，可以使用 `logrus.SetNoLock()` 禁用。

[uber-go/zap](https://github.com/uber-go/zap) 是 Uber 出品的快速、结构化、支持级别的日志库。[从 Go 高性能日志库 zap 看如何实现高性能 Go 组件](https://studygolang.com/articles/14220)。

[rs/zerolog](https://github.com/rs/zerolog) 独特的链式 API 允许 Zerolog 避免内存分配和反射来编写 JSON（或 CBOR）日志事件。

[lumberjack](https://github.com/natefinch/lumberjack) 是将日志写入滚动文件中，可以设置单日志文件的最大占用空间、最大生存周期、可保留的最多旧文件数等。如果有超出设置项的情况，则对日志文件进行滚动处理。

# 验证

[官网](https://pkg.go.dev/gopkg.in/go-playground/validator.v9?tab=doc)

```shell
go get github.com/go-playground/validator/v10
```

## 约束

借助 `validate` Tag 来对结构体的字段进行约束，如：

```go
type User struct {
	Name string `validate:"min=6,max=10"`
	Age  int    `validate:"min=1,max=100"`
}
```

### 范围约束

- 数值：约束其值
- 字符串：约束其长度
- 切片、数组、map：约束其长度

```go
type User struct {
	Name string `validate:"ne=admin"` // ne 不等于
	Age  int    `validate:"gte=18"`   // gte  大于等于
	// oneof 只能是列举出的值其中一个，这些值必须是数值或字符串，以空格分隔，如果字符串中有空格，将字符串用单引号包围
	Sex string `validate:"oneof=male female"`
	// 注意如果字段类型是time.Time，使用 gt/gte/lt/lte 等约束时不用指定参数值，默认与当前的 UTC 时间比较
	RegTime time.Time `validate:"lte"` // lte 小于等于
}

func TestValidator(t *testing.T) {
	// 创建验证器，这个验证器可以指定选项、添加自定义约束
	validate := validator.New()
	// 使用 Struct() 验证各种结构对象的字段是否符合定义的约束
	u := User{Name: "admin", Age: 15, Sex: "none", RegTime: time.Now().UTC().Add(1 * time.Hour)}
	err = validate.Struct(u)
	if err != nil {
		// 四个字段的值都验证失败了
		fmt.Println("User validate error: \n",err)
	}
}
```

### 跨字段约束

可以定义跨字段的约束，即该字段与其他字段之间的关系，以下两种：

1. 参数字段就是同一个结构中的平级字段；
2. 参数字段为结构中其他字段的字段

语法就是在之前的约束后面加上 `field`，如 `eqfield` 就是字段间的相等约束。如果是更深层次的字段，在 `field` 前面还要加上 `cs`，如 `eqcsfield`

```go
eqfield=ConfirmPassword
eqcsfield=InnerStructField.Field
```

```go

type RegisterForm struct {
	Name     string `validate:"min=2"`
	Age      int    `validate:"min=18"`
	Password string `validate:"min=10"`
	// 该字段的值要和 Password 字段的值相等
	Pwd      string `validate:"eqfield=Password"`
}

func TestCrossFieldConstraint(t *testing.T) {
	validate := validator.New()
	f := RegisterForm{
		Name:     "dj",
		Age:      18,
		Password: "1234567890",
		Pwd:      "123",
	}
	err = validate.Struct(f)
	if err != nil {
		// Key: 'RegisterForm.Pwd' Error:Field validation for 'Pwd' failed on the 'eqfield' tag
		fmt.Println("f validate error: \n", err)
	}
}
```

### 字符串

字符串相关的约束：

```
contains=：   包含参数子串，例如contains=email；
containsany： 包含参数中任意的 UNICODE 字符，例如containsany=abcd；
containsrune：包含参数表示的 rune 字符，例如containsrune=☻；
excludes：    不包含参数子串，例如excludes=email；
excludesall： 不包含参数中任意的 UNICODE 字符，例如excludesall=abcd；
excludesrune：不包含参数表示的 rune 字符，excludesrune=☻；
startswith：  以参数子串为前缀，例如startswith=hello；
endswith：    以参数子串为后缀，例如endswith=bye。
```

### 唯一性

使用 `unqiue` 来指定唯一性约束，对不同类型的处理如下：

- 对于数组和切片，`unique` 约束没有重复的元素；
- 对于 map，`unique` 约束没有重复的值；
- 对于元素类型为结构体的切片，`unique` 约束结构体对象的某个字段不重复，通过 `unqiue=field` 指定这个字段名。

```go
type User struct {
  Name    string   `validate:"min=2"`
  Age     int      `validate:"min=18"`
  Hobbies []string `validate:"unique"`
  Friends []User   `validate:"unique=Name"`
}
```

### 邮件

通过 `email` 限制字段必须是邮件格式：

```go
type User struct {
  Name  string `validate:"min=2"`
  Age   int    `validate:"min=18"`
  Email string `validate:"email"`
}
```

### 特殊

有一些比较特殊的约束：

```
-：        跳过该字段，不检验；
|：        使用多个约束，只需要满足其中一个，例如 rgb|rgba；
required： 字段必须设置，不能为默认值；
omitempty：如果字段未设置，则忽略它。
```

## VarWithValue()

在一些简单的场景中，可能只需要对两个变量进行比较，如果每次都定义结构体和 Tag 就过于麻烦，Validator 提供了 `VarWithValue()` 方法，只要传入两个变量及约束即可：

```go
func TestVarWithValue(t *testing.T) {
	str1 := "my name is fms5cms"
	str2 := "ms5cms"
	validate := validator.New()
	t.Log(validate.VarWithValue(str1,str2,"eqfield"))
	t.Log(validate.VarWithValue(str1,str2,"nefield"))
}
```

## 自定义约束

```go
ormation struct {
	Info string `validate:"startwithfS"`
}

func CheckStartWithfS(fl validator.FieldLevel) bool {
	value := fl.Field().String()
	return strings.HasPrefix(value,"fS")
}

func TestCustomize(t *testing.T) {
	validate := validator.New()
	// 注册校验规则
	validate.RegisterValidation("startwithfS",CheckStartWithfS)
	question := Information{"fS?"}
	answer := Information{"my favorite group"}
	t.Log(validate.Struct(question))
	t.Log(validate.Struct(answer))
}
```

## 错误处理

validator 返回的错误实际上只有两种：

- 参数错误时，返回 `InvalidValidationError` 类型；
- 校验错误时返回 `ValidationErrors`，这是一个切片，保存了每个字段违反的每个约束信息。

所以 validator 校验返回只有三种情况：

1. `nil` 没有错误
2. `InvalidValidationError` 输入参数错误
3. `ValidationErrors` 字段违反约束

可以在程序判断 `err != nil` 后，依次将 err 转换为 `InvalidValidationError` 和 `ValidationErrors` 以获取更详细的信息。

# 会话追踪

[gorilla/sessions](http://www.gorillatoolkit.org/pkg/sessions)为自定义session后端提供cookie和文件系统session以及基础结构。

主要功能是：

- 简单的API：将其用作设置签名（以及可选的加密）cookie的简便方法。
- 内置的后端可将 session 存储在 cookie 或文件系统中。
- Flash 消息：一直持续读取的 session值。
- 切换 session 持久性（又称“记住我”）和设置其他属性的便捷方法。
- 旋转身份验证和加密密钥的机制。
- 每个请求有多个 session，即使使用不同的后端也是如此。
- 自定义 sessio n后端的接口和基础结构：可以使用通用 API 检索并批量保存来自不同商店的 session。

```go
package main

import (
  "fmt"
  "net/http"
  "github.com/gorilla/sessions"
)

// 初始化一个cookie存储对象
// something-very-secret应该是一个自己的密匙，只要不被别人知道就行
var store = sessions.NewCookieStore([]byte("something-very-secret"))

func main() {
  http.HandleFunc("/save", SaveSession)
  http.HandleFunc("/get", GetSession)
  err := http.ListenAndServe(":8080", nil)
  if err != nil {
    fmt.Println("HTTP server failed,err:", err)
    return
  }
}

func SaveSession(w http.ResponseWriter, r *http.Request) {
  //　获取一个session对象，session-name是session的名字
  session, err := store.Get(r, "session-name")
  if err != nil {
    http.Error(w, err.Error(), http.StatusInternalServerError)
    return
  }

  // 在session中存储值
  session.Values["foo"] = "bar"
  session.Values[42] = 43
  // 保存更改
  session.Save(r, w)
}
func GetSession(w http.ResponseWriter, r *http.Request) {
  session, err := store.Get(r, "session-name")
  if err != nil {
    http.Error(w, err.Error(), http.StatusInternalServerError)
    return
  }
  foo := session.Values["foo"]
  fmt.Println(foo)
}
```

删除 session：

```go
session.Options.MaxAge = -1
session.Save(r, w)
```

# JWT

[JWT](../Base/02-HTTP/HTTP.md#验证-token)作为令牌系统而不是在每次请求时都发送用户名和密码，因此比其他方法（如基本身份验证）具有固有的优势。

```go
var jwtSecret = []byte("fms5cmS")

// Claims 用于存储数据
type Claims struct {
	Username string `json:"username"`
	Password string `jon:"password"`
	jwt.StandardClaims
}
// Header 部分：alg=HS256、typ=JWT
// Payload(也叫 Claims)部分：
//   Registered 部分这里未设置
//   Public 部分需要额外注册？
//   Private 部分可以随意定义，这里的两个字段都是 Private 的
// Signature 部分，防止数据被篡改
//   注意，该库的 SignedString 方法接收一个 interface{} 类型，但必须是 []byte，否则运行时报错
func GenerateToken(username, password string) (string, error) {
	nowTime := time.Now()
	expireTime := nowTime.Add(3 * time.Hour)
	claims := Claims{
		Username: username,
		Password: password,
		StandardClaims: jwt.StandardClaims{
			ExpiresAt: expireTime.Unix(),
			Issuer:    "fms5cmS",
		},
	}
	tokenClaims := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return tokenClaims.SignedString(jwtSecret)
}

func ParseToken(token string) (*Claims, error) {
	tokenClaims, err := jwt.ParseWithClaims(token, &Claims{}, func(token *jwt.Token) (interface{}, error) {
		// 对 alg 即签名算法校验
		if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
			return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
		}
		return jwtSecret, nil
	})
	if tokenClaims != nil {
		if claims, ok := tokenClaims.Claims.(*Claims); ok && tokenClaims.Valid { // 校验有效性
			return claims, nil
		}
	}
	return nil, err
}
```

# 生成 API 文档

[swag](https://github.com/swaggo/swag) 可以自动生成 Restful 风格的 API 文档。支持 Gin、echo、buffalo 框架以及 Go 自带的 net/http 包。



# 定时任务

[robfig/cron](https://github.com/robfig/cron) 实现了 cron 规范解析器和任务运行器，简单来讲就是包含了定时任务所需的功能。

Cron 表达式：`秒 分 时 日 月 周`

| 字段名       | 允许的值        | 允许的特殊字符      |
| ------------ | --------------- | ------------------- |
| Seconds      | 0-59            | `,` `-` `*` `/`     |
| Minutes      | 0-59            | `,` `-` `*` `/`     |
| Hours        | 0-23            | `,` `-` `*` `/`     |
| Day-of-Month | 1-31            | `,` `-` `*` `?` `/` |
| Month        | 1-12 或 JAN-DEC | `,` `-` `*` `/`     |
| Day-of-Week  | 1-7 或 SUN-SAT  | `,` `-` `*` `?` `/` |

这里的 Cron 比 Linux 中的 Crontab 多了秒，特殊字符：

| 符号  | 含义                                       |
| ----- | ------------------------------------------ |
| `*`   | 匹配所有值                                 |
| `，`  | 指定可选值                                 |
| `-`   | 指定一个范围                               |
| `*/n` | 指定增量                                   |
| `?`   | 不指定值，用于替代 `*`，类似于 Go 中的 `_` |

# 邮箱

```go
go get -u gopkg.in/gomail.v2
```

gomail 是一个用于发送电子邮件得第三方开源库，目前只支持 SMTP 服务器发送电子邮件。

SMTP Server 的 HOST 端口号为 465，还有一种常用的 HOST 端口号为 25，不建议使用。25 端口在云服务厂商上是一个经常被默认封禁的端口，且不可解封，如果使用 25 端口，可能部署到云服务环境后邮件无法正常发送。



# 令牌桶

```go
go get -u github.com/juju/ratelimit@sv1.0.1
```

ratelimit 提供了一个简单高效的令牌桶实现，可以用于实现限流器的逻辑



# 分布式链路追踪

Jaeger 是 uber 开源的分布式链路追踪系统，它提供了分布式上下文传播、分布式交易监控、原因分析、服务依赖性分析、性能/延迟优化分析等核心功能。

通过 Docker 的方式安装并启动：

```shell
docker run -d --name jaeger \
	-e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
	-p 5775:5775/udp \
	-p 6831:6831/udp \
	-p 6832:6832/udp \
	-p 5778:5778 \
	-p 16686:16686 \
	-p 14268:14268 \
	-p 9411:9411 \
	jaegertracing/all-in-one:1.16
```

启动后，打开浏览器访问 `http://localhost:16686` 即可看到 Jaeger 的 Web UI 界面。

```go
go get -u github.com/opentracing/opentracing-go
go get -u github.com/uber/jaeger-client-go/
```



# 热更新

- 配置热更新

要对配置文件热更新，就需要对配置文件进行监听。开源库 fsnotify 是一个跨平台文件系统监听事件库，常用于文件监听。它是基于 golang.org/x/sys 实现的，并非 syscall 标准库，所以安装的同时需要更新 golang.org/x/sys 的版本

```go
go get -u golang.org/x/sys/...
go get -u github.com/fsnotify/fsnotify
```

