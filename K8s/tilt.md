[Tilt](https://github.com/tilt-dev/tilt) 主要是为了针对 K8s 上的微服务做本地开发。[Tilt 官方文档](https://docs.tilt.dev/index.html)。



# Hello World

示例，所有文件均在 learn_tilt 目录下

main.go：

```go
func main() {
	http.HandleFunc("/", NewExampleRouter)
	log.Println("Serving on port 8000")
	err := http.ListenAndServe(":8000", nil)
	if err != nil {
		log.Fatalf("Server exited with: %v", err)
	}
}

func NewExampleRouter(resp http.ResponseWriter, req *http.Request) {
	fmt.Println(req.Method, req.URL)
}
```

构建程序：`CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o .` 得到了 learn_tilt 的可执行文件（Linux 系统下的）

Dockerfile：

```dockerfile
FROM scratch
COPY ./learn_tilt /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["./learn_tilt"]
```

deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: learn-tilt
  labels:
    app: learn-tilt
spec:
  selector:
    matchLabels:
      app: learn-tilt
  template:
    metadata:
      labels:
        app: learn-tilt
    spec:
      containers:
        - name: tilt-test
          image: learn-tilt # 要和下面 Tilefile 第一行指定的镜像名一致！
          ports:
            - containerPort: 8000
```

Tiltfile：

```shell
# 基于当前路径下的 Dockerfile 在当前路径下构建一个名为 learn-tilt 的镜像
docker_build('learn-tilt', '.', dockerfile='./Dockerfile')
# 告诉 tilt 去加载的 K8s 配置文件，注意，配置文件中使用的镜像名要和上面指定的镜像名一致
k8s_yaml('./deployment.yaml')
# 配置了端口转发，注意，第一个参数必须和 K8s 配置文件中的 metadata.name 一致
k8s_resource('learn-tilt', port_forwards=8000)
```

在 Tiltfile 所在的目录下，执行 `tult up`，tilt 会在浏览器中打开一个新的页面，里面包含了应用的状态和终端日志。

在 Tiltfile 所在的目录下，执行 `tilt down`，tilt 会删除该 TIltfile 相关的资源，docker 镜像还是会保留！

# Tiltfile

## 应用的依赖

编写一个程序启动前的程序文件，record_start.py：

```python
import time

contents = """package main
import "time"
var StartTime = time.Unix(0, %d)
""" % int(time.time() * 1000 * 1000 * 1000)

# 写模式打开 start.go 文件，并向里面写入 contents
f = open("start.go", "w")
f.write(contents)
f.close()
```

修改 Tiltfile：

```shell
# 声明**本地**已有的脚本或命令，这里是执行了一个命令，对应的程序名为 deploy
local_resource('start', 'python record_time.py')

docker_build('example-go-image', '.', dockerfile='deployments/Dockerfile')
k8s_yaml('deployments/kubernetes.yaml')
# resource_deps 指定要执行的依赖或脚本
k8s_resource('learn-tilt', port_forwards=8000, resource_deps=['start'])
```

注意，如果是在 Windows 下的话，需要将 python 命令配置到 PATH 路径下。

以上面为例，必须在 start 执行完成后才会打包镜像、部署应用！

微服务部署中，通常会有很多需要关注的基准，如构建镜像的时间、调度应用的时间、服务端准备好提供流量的时间等。

Tilt 提供了一些默认基准，以及捕获你自己的工具。

说明：这里只是对 local_resource 和 resource_deps 使用的简单示例，所以没有修改 main.go 的代码。

有了  local_resource 和 resource_deps，就可以将程序的构建也放进这里了。

```shell
local_resource('start', 'python record_time.py')

# 这里是测试将程序编译过程也放进来，Linux 下可以
# Windows 下需要使用 set 设置临时环境变量，但似乎无法在一行中完成？
# compile_cmd = 'CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o ./learn_tilt .'
# local_resource('go-compile', compile_cmd, deps=['./main.go', './start.go'], resource_deps=['start'])

# 这里打印了 go 的版本来模拟多层依赖
# deps 定义了依赖的文件，这些文件被更新时会触发再次执行该命令！！resource_deps 定义了前置依赖
local_resource('go-compile', 'go version', deps=['./main.go', './start.go'], resource_deps=['start'])
# only 指定了在构建 docker 镜像时仅需要的源文件，这样就可以确保其他文件变更时不会触发构建
docker_build('learn-tilt', '.', dockerfile='./Dockerfile', only=['learn_tilt'])
k8s_yaml('./deployment.yaml')
k8s_resource('learn-tilt', port_forwards=8000, resource_deps=['start', 'go-compile'])
```

可以看到构建顺序：

```
Tilt started on http://localhost:10350/
v0.27.1, built 2022-04-08

Initial Build
Loading Tiltfile at: E:\mod\src\learn\learn_tilt\Tiltfile
Successfully loaded Tiltfile (12.9269ms)
        start │
        start │ Initial Build
        start │ Running cmd: python record_time.py
   go-compile │
   go-compile │ Initial Build
   go-compile │ Running cmd: go version
   go-compile │ go version go1.17.6 windows/amd64
   learn-tilt │
   learn-tilt │ Initial Build
   learn-tilt │ STEP 1/3 — Building Dockerfile: [learn-tilt]
   learn-tilt │ Building Dockerfile for platform linux/amd64:
   .......
   learn-tilt │ STEP 2/3 — Pushing learn-tilt:tilt-c32bd7357ba7aa0a
   learn-tilt │      Skipping push: building on cluster's container runtime
   learn-tilt │
   learn-tilt │ STEP 3/3 — Deploying
   learn-tilt │      Applying YAML to cluster
   learn-tilt │      Objects applied to cluster:
   learn-tilt │        → learn-tilt:deployment
   .......
```



## 检查

`tilt ci --file ./Tiltfile` 可以检查步骤是否正常，注意，该命令也会构建镜像部署等！

`tilt down --file ./Tiltfile` 删除指定 Tiltfile 在检查过程中生成的内容。

可以通过一个脚本来检查微服务的多个应用的 Tiltfile，如：

```shell
#!/bin/bash
# -e 代表如果脚本中某个命令报错就退出
# -x 打印执行的命令
set -ex
# cd to the root of the repo.
cd $(dirname $(dirname $0))

echo "Testing 0-base"
tilt ci --file 0-base/Tiltfile
tilt down --file 0-base/Tiltfile

echo "Testing 1-measured"
tilt ci --file 1-measured/Tiltfile
tilt down --file 1-measured/Tiltfile
```



