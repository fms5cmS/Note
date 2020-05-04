以 SQLite 3 为数据库举例说明：

Windows 下安装 GCC：在 [链接](http://tdm-gcc.tdragon.net/download) 中下载后安装，然后将安装目录/bin 添加到环境变量中

数据库：[SQLite3 下载](https://www.sqlite.org/download.html) 解压后将目录添加到环境变量中

安装：`go get github.com/mattn/go-sqlite3`

下载 mysql 驱动：`go get -t -v github.com/go-sql-driver/mysql/...`

# Model

```go
package models

import (
	"github.com/Unknwon/com"
	"github.com/astaxie/beego/orm"
	_ "github.com/mattn/go-sqlite3"  // 需要导入驱动包
	"os"
	"path"
	"time"
)

const (
	_DB_NAME = "data/beeblog.db"
	_SQLITE3_DRIVER = "sqlite3"
)

type Category struct {
	Id          int64 //如果字段名为 Id，且类型为 int、int32 等 int 类型 则 ORM 会默认这是主键
	Title       string //默认长度为 255 字节
	Created     time.Time `orm:"index"` //创建时间，后面的内容表示在建字段时建索引
	Views       int64     `orm:"index"` //浏览次数
	TopicTime   time.Time `orm:"index"`
	TopicCount  int64 //统计当前分类（主题）下有多少文章
	TopicUserId int64 //记录最后一个操作的用户 Id
}

type Topic struct {
	Id              int64
	Uid             int64
	Title           string
	Author          string
	Content         string `orm:"size(5000)"` //设置内容的容量为 5000 字节
	Attachment      string                    //附件
	Created         time.Time `orm:"index"`
	Updated         time.Time `orm:"index"`
	Views           int64     `orm:"index"`
	ReplyTime       time.Time `orm:"index"` //回复时间
	ReplyCount      int64                   //评论个数
	ReplyLastUserId int64                   //最后一条评论的用户 Id
}

func init() {
	// 检查数据库文件是否存在
	if !com.IsExist(_DB_NAME){
		// Dir() 会取出目录路径（不包括文件）
		os.MkdirAll(path.Dir(_DB_NAME), os.ModePerm)
		// 创建数据库文件
		os.Create(_DB_NAME)
	}

	//注册定义的 model
	orm.RegisterModel(new(Category),new(Topic))
	// 注册驱动，SQLITE3 是自动注册的，所以不写下面一行
	//orm.RegisterDriver(_SQLITE3_DRIVER,orm.DRSqlite)
	// 注册数据库，强制要求必须有一个叫做 default 的数据库，可以有多个数据库
	// 最后一个参数是最大连接数
	orm.RegisterDataBase("default",_SQLITE3_DRIVER,_DB_NAME,10)
}
```

```go
func main() {
	//将所有的 orm 信息打印出来以方便调试
	orm.Debug = true
	//自动建表：不强制（如果已存在表则不再重建），打印相关信息
	orm.RunSyncdb("default",false,true)
	//注册路由
	beego.Router("/",&controllers.HomeController{})
	beego.Router("/login",&controllers.LoginController{})

	//启动
	beego.Run()
}
```

