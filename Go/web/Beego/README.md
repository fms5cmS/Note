[Beego](https://beego.me/)

安装 Beego：`go get -u github.com/astaxie/beego`

安装 bee 工具：`go get -u github.com/beego/bee`

[bee 工具](https://beego.me/docs/install/bee.md)具有热编译功能，每次对代码修改保存后会自动进行热编译。

- 初始化 Beego 项目： `bee new 项目名`
- 启动 Beego 项目： `bee run`，会监控 beego 项目，通过 fsnotify 监控文件系统
- 应用打包：`bee pack`，把项目打包成 zip 包
- 查看版本：`bee version`，会动态获取 bee、beego、Go 的版本，其中 beego 的版本是读取 `$GOPATH/src/github.com/astaxie/beego` 的文件进行的！
- 自动生成代码：`bee generate`
- 数据库迁移：`bee migrate`
- docker 化：`bee dockerize`





# Web项目目录结构

`bee new 项目名` 后会创建新项目，并生成目录结构：

```shell
project
├── conf              # 存放配置文件
│   └── app.conf      # beego 默认会解析该文件
├── controllers
│   └── default.go
├── main.go
├── models
├── routers
│   └── router.go
├── static
│   ├── css
│   ├── img
│   └── js
├── tests
│   └── default_test.go
└── views
    └── index.tpl
```



# 配置

- app.conf 文件，beego 默认会解析 conf/app.conf 文件
- 可通过 `beego.AppConfig.String("参数名")` 来获取配置信息

```go
func (c *MainController) Get() {
  c.Ctx.WriteString("appname:"+beego.AppConfig.String("appname")+
                    "\nhttpport:"+beego.AppConfig.String("httpport")+
                    "\nrunmode:"+beego.AppConfig.String("runmode")+"\n\n")
}
```
 - 也可通过默认参数来获取，默认参数都在 beego.BConfig 中，其中又分为不同类型：
     - App 配置放在 `beego.BConfig` 中
     - Web 配置放在 `beego.BConfig.WebConfig` 中
     - 监听配置放在 `beego.BConfig.Listen` 中
     - Session 配置放在 `beego.BConfig.WebConfig.Session` 中
     - 日志配置放在 `beego.BConfig.Log` 中
```go
func (c *MainController) Get() {
  port := strconv.Itoa(beego.BConfig.Listen.HTTPPort)

  c.Ctx.WriteString("appname:"+beego.BConfig.AppName+
                    "\nhttpport:"+port+
                    "\nrunmode:"+beego.BConfig.RunMode)
}
```



# 日志

Beego 使用 github.com/astaxie/beego/logs 包来完成日志功能。

- 日志级别（从上到下依次降低）：
    - LevelEmergency
   - LevelAlert
   - LevelCritical
   - LevelError
   - LevelWarning  ==  LevelWarn
   - LevelNotice
   - LevelInformational  ==  LevelInfo
   - LevelDebug  ==  LevelTrace

当设置了日志级别后，则只会打印该级别以下的日志。

```go
  logs.Trace("trace-1")
  logs.Info("info-1")
  //设置日志级别为 info，之后就只能打印出 info 级别以下的日志
  logs.SetLevel(logs.LevelInfo)
  logs.Trace("trace-2")
  logs.Info("info-2")
```