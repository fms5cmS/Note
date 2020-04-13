# 本地使用

本地玩耍注意事项：可以配置域名转发，但一定要配置 hosts，且使 hosts 生效后才行，设置完成后重启浏览器。(注意：修改的是浏览器所在机器的 host 文件)

- Linux 下：
  1. `vim /etc/hosts`
  2. 添加好对应的域名及 IP，保存退出即可
- Windows 下：
  1. 进入 C:\Windows\System32\drivers\etc
  2. 记事本打开 hosts 文件，添加好对应的域名及 IP，保存退出即可

# 常用命令

都是在安装目录 /usr/local/nginx/sbin 下执行的，如果将该路径写入了 PATH ，则在任意路径下都可以执行。

```shell
nginx -?            # 帮助，也可使用 -h
nginx -s stop       # 快速关闭 Nginx，可能不保存相关信息，并迅速终止web服务。
nginx -s quit       # 平稳关闭 Nginx，保存相关信息，有安排的结束web服务。
nginx -s reload     # 因改变了 Nginx 相关配置，需要重新加载配置而重载。在 Nginx 不停止对客户服务的情况下使用修改的配置项
nginx -s reopen     # 打开一个新的日志文件，重新开始记录日志文件。
nginx -c filename   # 为 Nginx 指定一个配置文件，来代替缺省的。
nginx -g xxx	    # 指定配置指令，可以覆盖配置文件中的指令
nginx -p            # 指定运行目录
nginx -t            # 不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。也可使用 -T
nginx -v            # 显示 Nginx 的版本。
nginx -V            # 显示 Nginx 的版本，编译器版本和配置参数。
```

启动命令：`./nginx`

平滑重启：`kill -HUP pid号`（pid 号可使用`ps -ef|grep nginx`查询得到 Nginx 的主进程号）

# 热部署

在启动 Nginx 的情况下，想要更换 Nginx 的版本：

- 下载新的 Nginx，并编译
- 把现有的 Nginx 的二进制文件备份
  - 在安装目录/sbin 内：`cp nginx nginx.old`
- 将刚编译好的最新版的 Nginx 的二进制文件复制到 sbin 下，替换到正在运行的所使用的二进制文件
  - `cd 解压目录/objs/`
  - `cp -r nginx /home/fms5cms/nginx/sbin -f`
- 向正在运行的 Nginx 的 master 进程发送信号要进行热部署做一次版本升级
  - `kill -USR2 pid号`（pid 号可使用`ps -ef|grep nginx`查询得到 Nginx 的主进程号）
  - 旧的 master 进程会新起一个 master 的主进程，新的 master 进程会使用新的 nginx 二进制文件来启动；
  - 旧的 master 进程会平滑的把所有请求过渡到新的 worker 进程
- 向旧的 master 进程发送信号：优雅的关闭所有的 worker 进程（旧的 master 进程还在）
  - `kill -WINCH pid号`，此时所有的请求已经切换到新的 worker 进程中了
    - 有可能发现问题，需要将新版本退回到旧版本，此时还可以向旧的 master 进程发 reload 信号重新启动 worker 进程，再把新版本关掉，所以旧的 master 进程不会自动退出，而是保留下来允许我们做版本回退

# 日志切割

- 将原本的日志修改名字
- `./nginx -s reopen`

上面的方法不好，通常定期执行一次日志切割，可以将内容写成脚本

# Nginx 的请求方式处理

Nginx 是一个高性能的 Web 服务器，能够同时处理大量的并发请求。它结合多进程机制和异步机制 ，异步机制使用的是异步非阻塞方式 ，接下来就给大家介绍一下 Nginx 的多线程机制和异步非阻塞机制 。

- 多进程机制：
  - 服务器每当收到一个客户端时，就由服务器主进程 （ master process ）生成一个 子进程（ worker process ）来和客户端建立连接进行交互，直到连接断开，该子进程就结束了。
  - 优点：
    - 进程之间相互独立，不用加锁，减少了使用锁对性能造成影响，同时降低编程的复杂度，降低开发成本；
    - 采用独立的进程，可以让进程互相之间不会影响 ，如果一个进程发生异常退出时，其它进程正常工作， master 进程则很快启动新的 worker 进程，确保服务不会中断，从而将风险降到最低
  - 操作系统生成一个子进程需要进行 内存复制等操作，在资源和时间上会产生一定的开销。当有大量请求时，会导致系统性能下降 。
- 异步非阻塞机制：
  - 每个工作进程 使用 异步非阻塞方式 ，可以处理多个客户端请求。
  - 当某个工作进程收到客户端的请求后，调用 IO 进行处理，如果不能立即得到结果，就去处理其他请求（即为 非阻塞 ）；而 客户端 在此期间也 无需等待响应 ，可以去处理其他事情（即为 异步 ）。当 IO 返回时，就会通知此 工作进程 ；该进程得到通知，暂时挂起当前处理的事务去 响应客户端请求 。

可参考：[芋道源码的“浅谈 Nginx 服务器的内部核心架构设计”](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486590&idx=2&sn=1cdacaf1fa27bdb90efab5b9aa2f6efc&chksm=fa4973cfcd3efad935da7bc4ce7314b861640ce477f528b377def9ad02f341fe4ab0ecbd77c4&mpshare=1&scene=23&srcid=0327lSlW7wrJdBlrwwqPVCrk#rd)
