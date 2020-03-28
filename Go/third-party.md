Go 语言库常用的一个设计套路：一般来说，一个库实现某个功能，通过一个结构体+方法的形式来做。而为了使用方便，会在包级别定义和结构体方法一致的函数，使用结构体默认的实例。

# 配置文件相关

[viper](https://github.com/spf13/viper)，支持 JSON, TOML, YAML, HCL, envfile and Java properties 等多种格式的配置文件，但不支持 ini 格式。严格而言，viper 并不解析具体格式，解析的工作根据实际的格式交给对应的库。它最大的好处是为所有支持的格式提供统一的 API，这让使用者不必关心具体是什么格式，同时换配置文件格式代码几乎不用修改。

```go
type config struct {
	language string
	people   person
}

type person struct {
	name   string
	age    int
	height int
	weight int
}

func TestReadConfig(t *testing.T) {
	// 设置配置文件名，不需要带扩展名，便于在不修改代码的情况下替换配置文件的类型
	viper.SetConfigName("configfile")
	// 设置配置文件类型(可选，viper 会自动判断)
	viper.SetConfigType("toml")
	// 可以添加多个配置文件的路径
	viper.AddConfigPath("../")
	// 读取配置文件，会根据不同的文件类型调用不同的解析库进行解析
	if err := viper.ReadInConfig(); err != nil {
		if _, ok := err.(viper.ConfigFileNotFoundError); ok {
			panic(fmt.Errorf("Can not find config file"))
		} else {
			panic(fmt.Errorf("Fatal error config file: %s", err))
		}
	}
	var cfg config
	// 将从配置文件中的读取到的配置信息填充到结构体中
	cfg.language = viper.GetString("language")
	cfg.people.age = viper.GetInt("person.age")
	t.Logf("the config is %v", cfg)
	some := viper.GetStringMap("person")
	for key, value := range some {
		t.Logf("%v: %v", key, value)
	}
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

# 日志相关

[日志记录指导原则](../通用/Log.md)

Go 语言标准库自带一个日志库：log，但这个库没有日志级别的概念，同时不支持结构化的日志。主流的三个日志库：

[Logrus](https://github.com/sirupsen/logrus)使用简单，且兼容标准库 log，但性能一般，其并发安全是通过 mutex 实现的，如果确定不需要锁，可以使用 `logrus.SetNoLock()` 禁用。

[uber-go/zap](https://github.com/uber-go/zap) 是 Uber 出品的快速、结构化、支持级别的日志库。[从 Go 高性能日志库 zap 看如何实现高性能 Go 组件](https://studygolang.com/articles/14220)

[rs/zerolog](https://github.com/rs/zerolog) 独特的链式 API 允许 Zerolog 避免内存分配和反射来编写 JSON（或 CBOR）日志事件。

# ORM

对象关系映射(ORM)。[GORM](https://gorm.io/) 和 Xorm 是 Go 中最流行的两个 ORM 库，前者更常用。

```shell
# 下载 gorm
go get -u github.com/jinzhu/gorm
# 下载 mysql 驱动
go get -t -v github.com/go-sql-driver/mysql/...
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
