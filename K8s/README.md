极客时间[《深入剖析 Kubernetes》](https://time.geekbang.org/column/article/14642) 学习笔记。

容器可分为两部分：
- 一组联合挂载在 /var/lib/docker/overlay2 上的 rootfs，这一部分我们称为“容器镜像”（Container Image），是容器的静态视图；
- 一个由 Namespace+Cgroups 构成的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器的动态视图。

很多的集群管理项目(如 Yarn、Mesos，以及 Swarm)所擅长的，都是把一个容器，按照某种规则，放置在某个最佳节点上运行起来。这种功能，我们称为“调度”；

Kubernetes 项目擅长的是按照用户的意愿和整个系统的规则，完全自动化地处理好容器之间的各种关系。这种功能就是编排。

Kubernetes 是一个自动化的容器编排平台，核心功能：
- 服务发现与负载均衡
- 容器自动装箱
- 存储编排
- 自动容器恢复(节点健康检查功能会将运行在故障节点上的容器自动迁移到健康的节点上)
- 自动发布与回滚
- 配置与密文管理
- 批量执行
- 水平伸缩(业务负载检查会对 cpu 利用率过高或响应时间过长的业务自动扩容，将容器变为多份然后通过负载均衡提高性能)



# K8s 架构

Kubernetes 项目的架构由 Master 和 Node 两种节点组成，分别对应控制节点和计算节点。

控制节点(Master) 由三部分组成：

- 负责 API 服务的 kube-apiserver
  - 整个集群的持久化数据由 kube-apiserver 处理后保存到 Etcd(分布式存储系统) 中
  - 所有组件都会和 apiserver 进行连接，组件间一般没有独立的连接，都依赖于 apiserver 进行消息传送
  - 可以水平扩展
- 负责调度的 kube-scheduler
  - 把用户提交的 container 依据其对 cpu、内存请求大小来寻找合适的节点进行放置
  - 可以进行热备
- 负责容器编排的 kube-controller-manager
  - 完成对集群状态的管理
  - 可以进行热备

计算节点(Node)：

最核心的是 **kubelet 组件**，该组件**主要负责与容器运行时(如 Docker 项目)通过 CRI(Container Runtime Interface) 进行交互**，CRI 这个接口定义了容器运行时的各项核心操作，如启动一个容器所需的所有参数；

具体的容器运行时，如 Docker 项目一般通过 OCI 这个容器运行时规范同底层的 Linux 操作系统进行交互，即：把 CRI 请求翻译成对 Linux 操作系统的调用(操作 Linux Namespace 和 Cgroups 等)；

kubelet 还通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互，该插件是 Kubernetes 项目用来管理 GPU 等宿主机物理设备的主要组件；

kubelet 还会通过 CNI(Container Networking Interface) 调用网络插件(Networking)为容器配置网络；通过 CSI(Container Storage Interface) 调用存储插件(Volume Plugin)来持久化存储。



# K8s 核心概念

Kubernetes 项目最主要的设计思想是，从更宏观的角度，以统一的方式来定义任务之间的各种关系，并且为将来支持更多种类的关系留有余地。

Kubernetes 项目核心概念：

- Pod 是最小的调度及资源单元，由一个或多个容器组成
  - 用于定义容器运行的方式(Command、环境变量等)
  - 提供给容器共享的运行环境(网络、进程空间)，同一个 Pod 中的容器共享同一个 Network Namespace、同一组数据卷，从而达到高效率交换信息的目的
- Volume 用于管理存储
  - 声明在 Pod 中的容器可访问的文件目录
  - 可以被挂载在 Pod 中一个(或多个)容器的指定路径下
  - 支持多种后端存储的抽象，如本地、分布式、云存储...
- Deployment
  - 定义一组 Pod 的副本数目、版本等
  - 通过 Controller 维持 Pod 的数目，自动恢复失败的 Pod
  - 通过 Controller 指定的策略控制版本，如滚动升级、重新生成、回滚等
- Service
  - 提供访问一个或多个 Pod 实例的稳定访问地址
  - 支持多种方式实现，如 ClusterIP、NodePort、LoadBalancer
- Namespace
  - 一个集群内部的逻辑隔离机制(鉴权、资源额度)
  - 每个资源都属于一个 Namespace
  - 同一个 Namespace 中的资源命名唯一，不同 Namespace 中资源可重名

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    command:
      - redis-server
      - "/redis-master/redis.conf"
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: example-redis-config
        items:
        - key: redis-config
          path: redis.conf
```



---

- 应用与应用之间的关系：
  - 为了一次启动多个应用的实例，所以需要 Deployment 这个 Pod 的多实例管理器；
  - 有了这样一组相同的 Pod 后，需要通过一个固定的 IP 地址和端口以负载均衡的方式访问它，所以有了 Service 服务，该服务作为 Pod 的代理入口(Portal)，从而代替 Pod 对外暴露一个固定的网络地址；
  - 如果现在两个不同 Pod 之间不仅有“访问关系”，还要求在发起时加上授权信息，Kubernetes 项目提供了一种叫作 Secret 的对象(一个保存在 Etcd 里的键值对数据)，当指定的 Pod 启动时，自动把 Secret 里的数据以 Volume 的方式挂载到容器里
- 应用运行形态，Kubernetes 定义了新的、基于 Pod 改进后的对象：
  - Job 用来描述一次性运行的 Pod (如，大数据任务)
  - DaemonSet，用来描述每个宿主机上必须且只能运行一个副本的守护进程服务
  - CronJob，用于描述定时任务
  - 等等...



# 声明式 API

在 Kubernetes 项目中，推崇的使用方法是：

1. 通过一个“编排对象”，比如 Pod、Job、CronJob 等，来描述你试图管理的应用；
2. 为它定义一些“服务对象”，比如 Service、Secret、Horizontal Pod Autoscaler（自动水平扩展器）等。这些对象，会负责具体的平台级功能

这种使用方法，就是所谓的“声明式 API”。这种 API 对应的“编排对象”和“服务对象”，都是 Kubernetes 项目中的 API 对象（API Object）



# 安装

见[官网](https://kubernetes.io/docs/tasks/tools/install-kubectl/)，这里的步骤来自[Kubernetes 中文社区](https://www.kubernetes.org.cn/5846.html)，也可查看 http://www.iexhaustion.com/2019/03/13/kubeadm/ 中的教程。

修改 hostname：

```shell
# 修改 hostname
hostnamectl set-hostname zzk
# 查看修改结果
hostnamectl status
# 设置 hostname 解析
echo "127.0.0.1 $(hostname)" >> /etc/hosts
```

在 master 和 worker 节点执行以下命令：

```shell
curl -sSL https://kuboard.cn/install-script/v1.16.0/install-kubelet.sh | sh
```

kubeadm 是一个部署工具，kubeadm 选择**把 kubelet 直接运行在宿主机上，然后使用容器部署其他的 Kubernetes 组件**。

`kubeadm init --config=配置文件` 部署 Master 节点，可以不指定配置文件，其工作流程如下：

1. pre-flight checks，进行一系列检查工作；
2. 生成 Kubernetes 对外提供服务所需的各种证书和对应的目录(Kubernetes 对外提供服务时，默认时通过 HTTPS 才能访问 kube-apiserver)
3. 为其他组件生成访问 kube-apiserver 所需的配置文件；
4. 为 Master 组件生成 Pod 配置文件；
5. Master 容器启动后，kubeadm 会通过检查 localhost:6443/healthz 这个 Master 组件的健康检查 URL，等待 Master 组件完全运行起来；
6. kubeadm 会为集群生成一个 bootstrap token，只要持有这个 token，任何一个安装了 kubelet 和 kubadm 的节点，都可以通过 kubeadm join 加入到这个集群当中；
7. kubeadm 会将 ca.crt 等 Master 节点的重要信息，通过 ConfigMap 的方式保存在 Etcd 当中，供后续部署 Node 节点使用。这个 ConfigMap 的名字是 cluster-info；
8. 安装默认插件(kube-proxy 和 DNS，分别用来提供整个集群的服务发现和 DNS 功能)。

在 Kubernetes 中使用“Static Pod”的方式启动容器，它允许你把要部署的 Pod 的 YAML 文件放在指定目录中，这样，当这台机器上的 kubelet 启动时，它会自动检查这个目录，加载所有的 Pod YAML 文件，然后在机器上启动它们。

在 kubeadm 中，Master 组件的 YAML 文件会被生成在 /etc/kubernetes/manifests 路径下。