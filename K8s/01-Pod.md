Pod 这个概念，提供的是一种编排思想，而不是具体的技术方案。

类比：云计算系统的操作系统是 K8s，容器相当于进程，而 Pod 则是进程组。

传统基于虚拟机的容器到微服务架构的迁移：分析应用组成（组件、进程），将其拆分成松耦合的容器，利用 init container 来解决顺序和依赖关系。

# Pod 实现原理

**Pod 只是一个逻辑概念，指的是一组共享了某些资源的容器，Kubernetes 真正处理的还是宿主机上 Linux 容器的 Namespace 和 Cgroups**，并不存在所谓的 Pod 的边界或隔离环境。

**Pod 中的所有容器共享的是同一个 NetworkNamespace，并且可以声明共享同一个 Volume**。

如果只是容器共享网络和 Volume，直接通过 docker 命令也可以实现：

```shell
docker run --net=B --volumes-from=B --name=A image-A ...
```

但是这样的话，容器 B 就必须比容器 A 先启动，这样一个 Pod 中的多个容器就不是对等关系而是拓扑关系了。

在 Kubernetes 中，**Pod 的实现需要使用一个中间容器——Infra 容器。每个 Pod 中的 Infra 容器永远是第一个被创建的容器，其他用户定义的容器则通过 Join Network Namespace 的方式与 Infra 容器关联在一起**。

注：Kubernetes 项目中 Infra 容器一定要占用极少的资源，所以使用了一个特殊的镜像—— k8s.gcr.io/pause，该镜像使用汇编语言编写，永远处于“暂停”状态，解压后大小为 100~200KB 左右。

用户容器会加入到 Infra 容器的 Network Namespace 中，所以同一个 Pod 中的 A、B 容器可以直接使用 localhost 通信；看到的网络设备与 Infra 容器看到的完全一样；一个 Pod 只有一个 IP 地址，即该 Pod 的 Network Namespace 对应的 IP 地址；其他所有的网络资源都是一个 Pod 一份，且被该 Pod 内所有容器共享；Pod 的生命周期只和 Infra 容器一致，而与 A、B 无关。

对于同一个 Pod 中的所有用户容器而言，其进出流量也可以认为都是通过 Infra 容器完成的。

注：为 Kubernetes 开发网络插件时，应重点考虑如何配置 Pod 的 Network Namespace，而不是每个用户容器如何使用你的网络配置；如果网络插件需要在容器中安装某些包或配置才能完成的话，这是不可取的，因为 Infra 容器镜像的 rootfs 中几乎什么都没有，应该只关注如何配置 Pod，即 Infra 容器的 Network Namespace 即可。

共享 Volume：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:
  restartPolicy: Never
  volumes:       # 在 Pod 层级定义 Volume
  - name: shared-data
    hostPath:      
      path: /data
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts: # 在 Pod 的容器中声明挂载上面定义的 Volume，所以 nginx-container 可以从 /usr/share/nginx/html 目录中读取到 debian-container生成的 index.html 文件
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: debian-container
    image: debian
    volumeMounts:  # 在 Pod 的容器中声明挂载上面定义的 Volume
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]
```



# 为什么需要 Pod

## 调度上的考虑

**Gang Scheduling** 是在并发系统中将多个相关联的进程调度到不同处理器上同时运行的策略，其最主要的原则是保证所有相关联的进程能够同时启动，防止部分进程的异常，导致整个关联进程组的阻塞。

示例说明：

Linux 系统中的日志处理 rsyslogd 由三个进程组成：imklog 模块、imuxsock 模块、rsyslogd 的 main 函数主进程，这三个进程一定要运行在同一台机器上，否则它们之间基于 Socket 的通信和文件交换会出问题。

现在将 rsyslogd 容器化，而受限于容器的“单进程模型”，其三个模块必须分别制作为三个不同的容器，如果三个容器运行时的内存配额都是 1GB。

假设 Kubernetes 集群由两个节点：node1 有 3GB 可用内存，node2 有 2.5GB可用内存。为了能让三个容器运行在同一台机器上，就必须在 main 主进程外的另外两个容器上设置 affinity=main（与 main 容器有亲密性）约束，即他们两个必须和 main 容器运行在同一台机器上。

而在调度时，main、imklog 容器都先后被调度到了 node2 上，当 imuxsock 容器被调度时就有问题了，根据 affinity=main 约束，imuxsock 容器只能运行在 node2 上，但是 node2 上的可用内存只剩下 0.5GB 了。

针对上述问题，有很多可供选择的解决方案：

- Mesos 中的资源囤积(resource hoarding)机制
  - 所有设置了 affinity 约束的任务都到达时才开始对它们统一进行调度
  - 会带来调度效率损失和死锁的可能性
- Google Omega 论文中提出了乐观调度处理冲突的方法
  - 先不管冲突，而是通过精心设计的回滚机制在出现冲突后解决问题
  - 过于复杂
- Kubernetes 项目中
  - **Pod 是 Kubernetes 中的原子调度单位，所以 Kubernetes 的调度器是统一按照 Pod 而非容器的资源需求进行计算的**

上面示例的三个容器会组成一个 Pod，Kubernetes 调度时会选择可用内存等于 3GB 的 node1 节点绑定，而不会考虑 node2。


## 容器设计模式

[容器设计模式](https://www.usenix.org/conference/hotcloud16/workshop-program/presentation/burns)

当用户想在一个容器中跑多个功能并不相关的应用时，应优先考虑它们是不是更应该被描述成一个 Pod 里的多个容器，而一个 Pod 里的多个容器通常具有以下关系：**多个容器互相之间会发生直接的文件交换、使用 localhost 或 Socket 文本进行本地通信、会发生频繁的远程调用、需要共享某些 Linux Namespace 等**。

并不是所有有关系的容器都属于同一个 Pod，如 Web 应用容器和 MySQL 虽然会繁盛访问关系，但没必要也不应部署在同一台机器上，更适合做成两个 Pod。

### 示例一

war 包与 Web 服务器：Java Web 应用的 war 包需要被放在 Tomcat 的 webapp 目录下运行，如果使用 Docker 该如何处理这个组合关系呢？

方法一：war 包直接放在 Tomcat 镜像的 webapp 目录下，做成一个新镜像运行。但是如果要更新 war 包内容或升级 Tomcat 镜像，就要重新制作一个新的发布镜像；

方法二：永远只发布一个 Tomcat 镜像，不过这个容器的 webapp 目录必须声明一个 hostPath 类型的 Volume，从而把宿主机上的 war 包挂载到 Tomcat 容器中运行。不过，要如何让每一台宿主机都预先准备好这个存储有 war 包的目录呢？还需要独立维护一套分布式存储系统。

有了 pod 后，可以把 war 包和 Tomcat 分别做成镜像，然后把它们作为一个 Pod 里的两个容器组合在一起：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: javaweb-2
spec:
	# spec.initContainers 定义的容器都比 spec.containers 定义的用户容器先启动
	# initContainers 容器会按顺序逐一启动，直到它们都启动且退出了，用户容器才会启动
  initContainers:  
  - image: geektime/sample:v2
    name: war
    command: ["cp", "/sample.war", "/app"]
    volumeMounts:
    - mountPath: /app
      name: app-volume
  containers:
  - image: geektime/tomcat:7.0
    name: tomcat
    command: ["sh","-c","/root/apache-tomcat-7.0.42-v2/bin/start.sh"]
    volumeMounts:
    - mountPath: /root/apache-tomcat-7.0.42-v2/webapps
      name: app-volume
    ports:
    - containerPort: 8080
      hostPort: 8001 
  volumes:
  - name: app-volume
    emptyDir: {}
```

initContainers 类型的 war 包容器启动后，将其 cp 到 /app 目录下然后退出，然后这个 /app 目录挂载了一个名为 app-volume 的 Volume，而 Tomcat 容器同样声明挂载了 app-volume 到自己的 webapps 目录下，这样，等 Tomcat 容器启动时，它的 webapps 目录下一定会有 war 包文件，因为那个 Volume 是被两个容器共享的。

通过组合的方式解决了 war 包与 Tomcat 容器间耦合关系的问题，不用每次代码变更时都对 Tomcat 容器重新 build，在镜像间实现了解耦。这种组合的操作在容器设计模式中叫做 sidecar，即在一个 Pod 中，启动一个辅助容器（war 包容器），来完成一些独立于主进程（主容器，Tomcat 容器）之外的工作。

### 示例二

假设有一个应用，需要不断把日志文件输出到容器的 /var/log 目录。

做法：把一个 Pod 里的 Volume 挂载到应用容器的 /var/log 目录上，然后在这个 Pod 里同时运行一个 sidecar 容器，它也声明挂载了同一个 Volume 到自己的 /var/log 目录，而 sidear 容器只需要做一件事，那就是不断从自己的 /var/log 目录读取日志文件，转发到 MongoDB 或 ElasticSearch 中存储起来，这样，一个最基本的日志收集工作就完成了。

**Pod 中所有容器都共享同一个 Network Namespace，所以很多与 Pod 网络相关的配置和管理，也可以交给 sidecar 完成，而完全无需干涉用户容器**。典型例子就是 Istio 这个微服务治理项目。

# Pod 对象

## Pod 级别的属性

Pod 可以看作是传统环境里的机器，而容器则是运行在这个"机器"中的用户程序。所以，**凡是调度、网络、存储、安全相关的属性，基本都是 Pod 级别的**。这些属性都描述了“机器”这个整体，而不是里面运行的“程序”。

- NodeSelector 是一个供用户将 Pod 与 Node 进行绑定的字段

```yaml
apiVersion: v1
kind: Pod
...
spec:
 nodeSelector:
   disktype: ssd  # 这个 Pod 永远只能运行在携带了“disktype: ssd”标签(Label)的节点上！
```

- NodeName 一旦 Pod 的这个字段被赋值，K8s 就会认为这个 Pod 已经经过了调度，调度的结果就是负值的节点名字。所以这个字段一般由调度器负责设置，再测试或调试的时候，用户也可以设置它来“骗”过调度器。
- HostAliases 定义了 Pod 的 hosts 文件(如 /etc/hosts) 里的内容
    - 在 K8s 中，如果要设置 hosts 文件中的内容，一定要通过这种方法，否则，如果直接修改 hosts 文件的话，在 Pod 被删除重建后，kubectl 会自动覆盖掉被修改的内容！

```yaml
apiVersion: v1
kind: Pod
...
spec:
  hostAliases:
  - ip: "10.1.2.3"
    hostnames:
    - "foo.remote"
    - "bar.remote"
...
```

在 Pod 启动后，上面的配置会在 /etc/hosts 文件中增加以下内容：

```
10.1.2.3 foo.remote
10.1.2.3 bar.remote
```

---

**凡是跟容器的 Linux Namespace 相关的属性，也一定是 Pod 级别的**。是为了让它里面的容器尽可能多的共享 Linux Namespace，仅保留必要的隔离和限制能力。示例：

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  shareProcessNamespace: true  # 这个 Pod 里的容器要共享 PID Namespace
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    stdin: true
    tty: true
```

所以在启动 Pod 后，nginx 容器和 shell 容器（开启了 tty、stdin，相当于设置了 `docker run -it`）会共享 PID Namespace，然后就可以使用 `kubectl attach -it nginx -c shell` 连接到 shell 容器的 tty 上，通过 `ps ax` 指令不仅可以看到 shell 容器本身的 `ps ax`  指令，还能看到 nginx 容器的进程以及 Infra 容器的 /pause 进程。

这就意味着，整个 Pod 里的每个容器进程，对于所有容器而言都是可见的。

---

**凡是 Pod 中的容器要共享宿主机的 Namespace，也一定是 Pod 级别的定义**。示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  # 共享宿主机的 Network、IPC、PID Namespace
  hostNetwork: true
  hostIPC: true
  hostPID: true
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    stdin: true
    tty: true
```

示例的 Pod 中所有容器会直接使用宿主机的网络、直接与宿主机进行 IPC 通信、看到宿主机中正在运行的所有进程。

## containers 字段

containers 与之前(容器设计模式-示例一)提到的 initContainers 字段都是 Pod 对容器的定义，内容也完全相同，只是 **initContainers 的生命周期会先于所有的 containers，且严格按定义的顺序执行**。

其中 image(镜像)、command(启动命令)、workingDir(容器工作目录)、ports(容器要开放的端口)、volumeMounts(容器要挂载的 Volume) 都是 containers 的主要属性。其他还有：

- imagePullPolicy 镜像拉取策略
    - 默认值为 Always，即每次创建 Pod 都重新拉取一次镜像
    - 值为 Never，Pod 永远不会主动拉取镜像
    - 值为 IfNotPresent，只在宿主机上不存在这个镜像时才拉取
- lifecycle，Container Lifecycle Hooks，即容器状态发生变化时触发的一系列“钩子”

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart: # 容器启动后立刻执行一个指定操作。在 postStart 启动时 Docker 容器 ENTRYPOINT 可能未结束
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
      preStop: # 容器被杀死前执行一个指定操作。会阻塞当前容器杀死流程，这个 Hook 定义操作完成后，才允许容器被杀死
        exec:
          command: ["/usr/sbin/nginx","-s","quit"] # 这里调用了 nginx 退出指令
```



## 生命周期

Pod 生命周期的变化主要体现在 Pod API 对象的 Status 部分。pod.status.phase 就是 Pod 的当前状态：

- Pending：Pod 的 yaml 文件已提交给 K8s，API 对象已被创建并保存在 etcd 中。但这个 Pod 中部分容器因某种原因而不能被顺利创建。
- Runing：Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中。
- Succeeded：Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见。
- Failed：Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志。
- Unknown：异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kube-apiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题。

Pod 对象的 status 字段还可以细分出一组 conditions，这些细分状态的值包括：PodScheduled、Ready、Initialized、Unscheduled，用于描述造成当前 status 的具体原因。

其中 Ready 这个细分状态意味着 Pod 不仅正常启动了（Running 状态），而且已经可以对外提供服务了。所以，存在 Pod 的状态是 Running，但是应用其实已经停止服务了的情况。

## 容器健康检查和恢复机制

### restartPolicy

可以为 Pod 中的容器定义一个健康检查“探针”(Probe)，kubelet 会根据这个 Probe 返回值决定这个容器的状态，而不是直接以容器镜像是否运行（Docker 返回的信息）作为依据。

这种机制是生产环境中保证应用健康存活的重要手段。

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: test-liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:  # 定义一个健康检查。类型是 ecec，会在容器启动后在容器中执行一条指定的命令
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5  # 容器启动 5s 后开始执行
      periodSeconds: 5  # 每 5s 执行一次
```

如果 /tmp/healthy 文件存在，健康检查锁执行的命令返回值就是 0，Pod 就认为这个容器不经已启动，而且是健康的。

```yaml
kubectl create -f test-liveness-exec.yaml
kubectl get pod test-liveness-exec  # 查看 Pod 状态，Pod 已通过健康检查，进入了 Running 状态
# 30s 后查看 Pod 的 Events 发现报告了一个异常，因为 /tmp/healthy 已经不存在了
kubectl describe pod test-liveness-exec
# 再次查看 Pod 状态，没有进入 Failed 状态，而是保持 Running 状态
# RESTARTS 字段从 0 变为 1，说明这个异常的容器已经被 K8s 重启了
kubectl get pod test-liveness-exec
```

注意：K8s 中并没有 Docker 的 Stop 语义，虽然是 Restart，但实际却是重新创建了容器。这就是 K8s 的 **Pod 恢复机制**，**也叫 restartPolicy**，是 Pod 的 spec 部分的一个标准字段(pod.spec.restartPolicy)

- 默认值为 Always，即任何情况下，只要容器不在运行状态，一定会被重启
    - Deployment 的该策略仅支持 Always，为了维持一定的副本数！
- Onfailure，只在容器异常时才自动重启容器
- Never，从不重启容器
    - 如果关心容器退出后的上下文环境，如容器退出后的日志、文件、目录，可以设置为 Never，因为一旦容器被自动重新创建，这些内动就可能丢失了（被垃圾回收）

**Pod 的恢复过程，永远都是发生在当前节点上，而不会跑到别的节点上去**。事实上，一旦一个 Pod 与一个节点（Node）绑定，除非这个绑定发生了变化（pod.spec.node 字段被修改），否则它永远都不会离开这个节点。这也就意味着，如果这个宿主机宕机了，这个 Pod 也不会主动迁移到其他节点上去。

如果你想让 Pod 出现在其他的可用节点上，就必须使用 Deployment 这样的“控制器”来管理 Pod，哪怕你只需要一个 Pod 副本。

**实际使用 Kubernetes 中，相比编写一个单独的 Pod 的 yaml 文件，更推荐使用一个 replicas=1 的 Deployment。后者可以在 Pod 所在节点故障时将其调度到健康的节点上，而单独的 Pod 只能在节点健康的情况下由 kubelet 保证 Pod 的健康状态。**

- **只要 Pod 的 restartPolicy 指定的策略允许重启异常的容器（如：Always），那么这个 Pod 就会保持 Running 状态，并进行容器重启**。否则，Pod 就会进入 Failed 状态 
- **对于包含多个容器的 Pod，只有它里面所有的容器都进入异常状态后，Pod 才会进入 Failed 状态。在此之前，Pod 都是 Running 状态。此时，Pod 的 READY 字段会显示正常容器的个数**



### livenessProbe

除了上面示例的 exec 类型，livenessProbe 还可以定义为发起 HTTP 或 TCP 请求的方式：

```yaml
...
    livenessProbe:
         httpGet:
           path: /healthz
           port: 8080
           httpHeaders:
           - name: X-Custom-Header
             value: Awesome
           initialDelaySeconds: 3
           periodSeconds: 3
```

```yaml
    ...
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

所以，Pod 可以暴露一个健康检查 URL(如 /healthz)，或者直接让健康检查去检测应用的监听端口。

在 Kubernetes 的 Pod 中，还有一个叫 readinessProbe 的字段。虽然用法与 livenessProbe 类似，但作用却大不一样。readinessProbe 检查结果的成功与否，决定的这个 Pod 是不是能被通过 Service 的方式访问到，而并不影响 Pod 的生命周期。见后续 Service 部分。

# Projected Volume

在 K8s 中有几种特殊的 Volume，它们不是为了存放容器中的数据、也不是用于进行容器和宿主机间的数据交换，而是**为容器提供预先定义好的数据**。从容器的角度看，这些 Volume 中的信息就像是被 K8s “投射”(Project) 进容器中的。目先 K8s 支持的 Projected Volume 有四种。

Secret、ConfigMap、DownwardAPI 三种 Projected Volume 定义的信息，大多可以通过环境变量的方式出现在容器中，但不具备自动更新的能力，建议使用 Volume 文件的方式获取这些信息。

ServiceAccountToken 只是一种特殊的 Secret。

## Secret

Secret：把 Pod 要访问的加密数据存放到 etcd 中，然后就可以通过在 Pod 容器中挂载 Volume 的方式访问这些 Secret 里保存的信息了。典型应用场景：存放数据库的 Credential 信息。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-projected-volume 
spec:
  containers:
  - name: test-secret-volume
    image: busybox
    args:
    - sleep
    - "86400"
    volumeMounts:
    - name: mysql-cred
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: mysql-cred
    projected:  # 声明挂载的 Volume 是 protected 类型
      sources:  # 指定这个 Volume 的数据来源分别是名为 user、pass 的 Secret 对象
      - secret:
          name: user
      - secret:
          name: pass
```

如何生成 Secret 对象呢？

```shell
cat ./username.txt # admin
cat ./password.txt # c1oudc0w!
# 生成 Secret 对象，并指定名字分别为 user、pass
kubectl create secret generic user --from-file=./username.txt
kubectl create secret generic pass --from-file=./password.txt
# 查看 Secret 对象
kubectl get secrets
```

也可以通过 yaml 文件创建 Secret 对象 test-projected-volume.yaml：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  user: YWRtaW4=      # echo -n 'admin' | base64
  pass: MWYyZDFlMmU2N2Rm # echo -n '1f2d1e2e67df' | base64
```

通过编写 yaml 文件创建出来的 Secret 对象只有一个，但它的 data 字段却以 key-value 的形式保存了两份 Secret 数据。注意：**Secret 对象要求这些数据必须是经过 Base64 转码的**，以免出现明文密码的安全隐患。

补充：生产环境中，还需要开启 Secret 的加密插件，增强数据的安全性。

```shell
kubectl create -f test-projected-volume.yaml
# 当 Pod 变成 Running 状态时，验证 Secret 对象是否已在容器中
kubectl exec -it test-projected-volume -- /bin/sh
ls /projected-volume/  # 会出现 user、pass 两个文件
# 保存在 etcd 中的用户名和密码信息，已经以文件的形式出现在了容器的 Volume 目录里
# 而这个文件的名字，就是 kubectl create secret 指定的 Key，或者说是 Secret 对象的 data 字段指定的 Key
cat /projected-volume/user
cat /projected-volume/pass
```

通过这种挂载方式进入到容器中的 Secret，一旦其对应的 etcd 里的数据被更新，这些 Volume 里的文件内容同样也会被更新。这是 kubelet 组件在定时维护这些 Volume。

注意：这个更新可能会有一定延时，所以在编写应用程序时，发起数据库连接的代码处要做好重试和超时的逻辑。

## ConfigMap

与 Secret 类似，不过 ConfigMap 保存的是不需要加密的、应用所需的配置信息。用法几乎与 Secret 完全相同：可以使用 `kubectl create configmap` 从文件或目录创建 ConfigMap，也可以编写 ConfigMap 对象的 yaml 文件。

```shell
# 从 .properties 文件创建 ConfigMap
kubectl create configmap ui-config --from-file=example/ui.properties

# 查看这个ConfigMap里保存的信息(data)
kubectl get configmaps ui-config -o yaml # 将指定的 Pod API 对象以 yaml 方式展示出来
# apiVersion: v1
# data: 
#	ui.properties: | 
#		color.good=purple 
#		color.bad=yellow 
#		allow.textmode=true 
#		how.nice.to.look=fairlyNice
# kind: ConfigMap
# metadata: 
#	name: ui-config 
#	...
```



## DownwardAPI

DownwardAPI 用于让 Pod 中的容器能直接获取到这个 Pod API 对象本身的信息。示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-downwardapi-volume
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
spec:
  containers:
    - name: client-container
      image: k8s.gcr.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
          readOnly: false
  volumes:
    - name: podinfo
      projected:
        sources:
        - downwardAPI: # projected 类型的 Volume 数据来源为 downwardAPI
            items:
              - path: "labels"
                fieldRef:
                  fieldPath: metadata.labels 
```

示例中，downwardApi Volume 声明要暴露 Pod 的 metadata.labels 给容器，这样，当前 Pod 的 labels 字段的值就会被 K8s 自动挂载成容器中的 /etc/podinfo/labels 文件。

而容器的启动命令则是不断打印 /etc/podinfo/labels 的内容。

```shell
kubectl create -f dapi-volume.yaml
# 查看 Pod 日志
kubectl logs test-downwardapi-volume
# cluster="test-cluster1"
# rack="rack-22"
# zone="us-est-coast"
```

DownwardApi 支持的字段：

```yaml
# 使用fieldRef可以声明使用:
spec.nodeName - 宿主机名字
status.hostIP - 宿主机IP
metadata.name - Pod的名字
metadata.namespace - Pod的Namespace
status.podIP - Pod的IP
spec.serviceAccountName - Pod的Service Account的名字
metadata.uid - Pod的UID
metadata.labels['<KEY>'] - 指定<KEY>的Label值
metadata.annotations['<KEY>'] - 指定<KEY>的Annotation值
metadata.labels - Pod的所有Label
metadata.annotations - Pod的所有Annotation

# 使用resourceFieldRef可以声明使用:
容器的CPU limit
容器的CPU request
容器的memory limit
容器的memory request
```

注意：DownwardAPI 能获取到的信息，一定是 Pod 中容器进程启动之前就能确定下来的信息！如果要获取 Pod 容器运行后才出现的信息，如进程 PID，应该考虑在 Pod 中定义一个 sidecar 容器（见容器设计模式——示例一）。

## ServiceAccountToken

K8s 中有一种 Service Account 对象，它是 K8s 系统内置的一种“服务账户”，是 K8s 进行权限分配的对象。如 Service Account A 只被允许对 K8s API 进行 GET 操作，而 Service Account B 则可以有 K8s API 的所有操作权限。

而 Service Account 的授权信息和文件，实际保存在它所绑定的一个特殊的 Secret 对象中，这个特殊的 Secret 对象就是 ServiceAccountToken。

任何运行在 K8s 集群上的应用，都必须使用这个 ServiceAccountToken 里保存的授权信息，也就是 Token，才可以合法地访问 API Server。

为了方便使用，K8s 已经提供了一个默认“服务账户”（default Service Account）。并且，任何一个运行在 K8s 里的 Pod，都可以直接使用这个默认的 Service Account，而无需显式地声明挂载它。K8s 允许设置默认不为 Pod 里的容器自动挂载这个 Volume。

```shell
kubectl describe pod nginx-deployment-5c678cfb6d-lg9lw
# 会看到每个 Pod 都已经自动声明了一个类型为 Secret、名为 default-token-xxx 的 Volume
# 并自动挂载在每个容器的一个固定目录上。
# 而这个 Secret 类型的 Volume 就是默认 Service Account 对应的 ServiceAccountToken
```

所以，K8s 其实在每个 Pod 创建的时候，自动在它的 spec.volumes 部分添加上了默认 ServiceAccountToken 的定义，然后自动给每个容器加上了对应的 volumeMounts 字段。

一旦 Pod 创建完成，容器里的应用就可以直接从这个默认 ServiceAccountToken 的挂载目录里访问到授权信息和文件。这个容器内的路径在 Kubernetes 里是固定的，即：/var/run/secrets/kubernetes.io/serviceaccount。

应用程序只要直接加载这些授权文件，就可以访问并操作 K8s API 了。而且，如果你使用的是 Kubernetes 官方的 Client 包（k8s.io/client-go）的话，它还可以自动加载这个目录下的文件，你不需要做任何配置或者编码操作。

**这种把 K8s 客户端以容器方式运行在集群里，然后使用 default Service Account 自动授权的方式，被称为 InClusterConfig**。



# PodPreset

Pod 的字段很多，可以通过 PodPreset（K8s v1.11 版本及之后才支持） 来自动填充定义好的信息。

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: website
  labels:
    app: website
    role: frontend
spec:
  containers:
    - name: website
      image: nginx
      ports:
        - containerPort: 80
```

上面定义的 Pod 很简单，但是在生产环境中是无法使用的。所以，可以先定义一个 PodPreset 对象，这个对象里会预先定义好想要在上面 pod.yaml 中追加的字段：

```yaml
# preset.yaml
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: allow-database
spec:
  selector:  # 选择器，即后面追加的定义只会作用于 selector 做定义的带有“role: frontend”标签的 Pod 对象
    matchLabels:
      role: frontend
  env:
    - name: DB_PORT
      value: "6379"
  volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
```

运行：

```shell
kubectl create -f preset.yaml
kubectl create -f pod.yaml
# Pod 运行起来后，查看 Pod 的 API 对象，可以看到 Pod 里多除了 PorPreset 定义的内容
# 同时，Pod 还被自动加上了一个 annotation 标识这个 Pod 对象被 PodPreset 改动过
kubectl get pod website -o yaml
```

注意：

PodPreset 定义的内容，只会在 Pod API 对象被创建之前追加在这个对象本身上，而不会影响任何 Pod 的控制器的定义。

假设提交一个 Deployment 对象，那么这个 Deployment 对象永远不会被 PodPreset 改变，被修改的只是这个 Deployment 对象创建出来的所有 Pod！

如果定义了同时作用于一个 Pod 对象的多个 PodPreset，K8s 会 Merge 这两个 PodPreset 要做的修改，如果多个 PodPreset 的有冲突的话，这些冲突字段就不会被修改。

