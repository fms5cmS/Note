极客时间[《深入剖析 Kubernetes》](https://time.geekbang.org/column/article/14642) 学习笔记。

容器可分为两部分：
- 一组联合挂载在 /var/lib/docker/overlay2 上的 rootfs，称为“容器镜像”（Container Image），是容器的静态视图；
- 一个由 Namespace+Cgroups 构成的隔离环境，称为“容器运行时”（Container Runtime），是容器的动态视图。

容器标准 Open Container Initiative（OCI）主要定义三个规范

- Runtime Specification 运行时标准：文件系统包如何解压至硬盘并运行
- Image Specification 镜像标准：如何通过构建系统打包，生成镜像清单（Manifest）、文件系统序列化文件、镜像配置
- Distribution Specification 分发标准：如何分发容器镜像

很多的集群管理项目(如 Yarn、Mesos，以及 Swarm)所擅长的，都是把一个容器，按照某种规则，放置在某个最佳节点上运行起来。这种功能称为“调度”；

Kubernetes 项目可以按照用户的意愿和整个系统的规则，完全自动化地处理好容器之间的各种关系。这种功能就是**编排**。

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

Kubernetes 项目的架构由 master 和 Node 两种节点组成，分别对应控制节点和计算节点。

控制节点(master) 由三部分组成：

- 负责 API 服务的 kube-apiserver
  - 整个集群的持久化数据由 kube-apiserver 处理后保存到 etcd(分布式存储系统) 中
  - 所有组件都和 apiserver 进行连接，组件间一般没有独立的连接，都依赖于 apiserver 进行消息传送
  - 可以水平扩展
- 负责调度的 kube-scheduler
  - 把用户提交的 container 依据其对 cpu、内存请求大小来寻找合适的节点进行放置
  - 可以进行热备
- 负责容器编排的 kube-controller-manager
  - 完成对集群状态的管理
  - 可以进行热备

计算节点(Node)：

最核心的是 **kubelet 组件**，该组件**主要负责与容器运行时(如 Docker 项目)通过 CRI(Container Runtime Interface) 进行交互**，CRI 这个接口定义了容器运行时的各项核心操作，如启动一个容器所需的所有参数；

具体的容器运行时，如 Docker 项目，一般通过 OCI 这个容器运行时规范同底层的 Linux 操作系统进行交互，即：把 CRI 请求翻译成对 Linux 操作系统的调用(操作 Linux Namespace 和 Cgroups 等)；

kubelet 还通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互，该插件是 Kubernetes 项目用来管理 GPU 等宿主机物理设备的主要组件；

kubelet 还会通过 CNI(Container Networking Interface) 调用网络插件(Networking)为容器配置网络；通过 CSI(Container Storage Interface) 调用存储插件(Volume Plugin)来持久化存储。



# K8s 核心概念

Kubernetes 项目最主要的设计思想是，从更宏观的角度，以统一的方式来定义任务之间的各种关系，并且为将来支持更多种类的关系留有余地。

如：

多个应用间需要非常频繁的交互和访问，又或者会直接通过本地文件进行信息交换，常规环境下，这些应用会被直接部署在同一台机器上，通过 localhost 通信，通过本地磁盘目录交换文件，而在 Kubernetes 项目中，这些容器会被划分为一个 Pod。**同一个 Pod 中的容器共享同一个 Network Namespace、同一组数据卷，从而达到高效率交换信息的目的。**

Web 应用与数据库之间的访问关系，Kubernetes 项目则提供了一种叫 Service 的服务。这样的两个应用往往故意不部署在同一台机器上，即使 Web 应用所在机器宕机了，数据库也完全不受影响。但是，对于一个容器而言，其 IP 地址等信息不是固定的，那么 Web 应用要怎么找到数据库容器的 Pod 呢？Kubernetes 会给 Pod 绑定一个 Service 服务，而 Service 服务声明的 IP 地址等信息是不变的。

Kubernetes 项目核心概念：

- Pod 是最小的调度及资源单元，由一个或多个容器组成
  - **Pod 就是 Kubernetes 世界里的“应用”；而一个应用可以由多个容器组成**。
  - 用于定义容器运行的方式(Command、环境变量等)
  - 提供给容器共享的运行环境(网络、进程空间)
- Volume 用于管理存储
  - 声明在 Pod 中的容器可访问的文件目录
  - 可以被挂载在 Pod 中一个(或多个)容器的指定路径下
  - 支持多种后端存储的抽象，如本地、分布式、云存储...
- Deployment
  - 定义一组 Pod 的副本数目、版本等，是 Pod 的多实例管理器
  - 通过 Controller 维持 Pod 的数目，自动恢复失败的 Pod
  - 通过 Controller 指定的策略控制版本，如滚动升级、重新生成、回滚等
  - 还负责在 Pod 定义发生变化时，对每个副本进行滚动更新（Rolling Update）
- Service
  - 会给 Pod 绑定一个 Service，Service会作为 Pod 的代理入口(Portal)，代替 Pod 对外暴露一个固定的网络地址
  - 提供访问一个或多个 Pod 实例的稳定访问地址
  - 支持多种方式实现，如 ClusterIP、NodePort、LoadBalancer
- Namespace
  - 一个集群内部的逻辑隔离机制(鉴权、资源额度)
  - 每个资源都属于一个 Namespace
  - 同一个 Namespace 中的资源命名唯一，不同 Namespace 中资源可重名

“一切皆对象”，如应用是 Pod 对象，应用的配置是 ConfigMap 对象，应用要访问的密码是 Secret 对象，针对 Pod 进行批量化、自动化修改的 PodPreset 对象。

# 声明式 API

- 应用与应用之间的关系：
  - 为了一次启动多个应用的实例，所以需要 Deployment 这个 Pod 的多实例管理器；
  - 有了这样一组相同的 Pod 后，需要通过一个固定的 IP 地址和端口以负载均衡的方式访问它，所以有了 Service 服务，该服务作为 Pod 的代理入口(Portal)，从而代替 Pod 对外暴露一个固定的网络地址；
  - 如果现在两个不同 Pod 之间不仅有“访问关系”，还要求在发起时加上授权信息，Kubernetes 项目提供了一种叫作 Secret 的对象(一个保存在 etcd 里的键值对数据)，当指定的 Pod 启动时，自动把 Secret 里的数据以 Volume 的方式挂载到容器里
- 应用运行形态。Kubernetes 定义了新的、基于 Pod 改进后的对象：
  - Job 用来描述一次性运行的 Pod (如，大数据任务)
  - DaemonSet，用来描述每个宿主机上必须且只能运行一个副本的守护进程服务
  - CronJob，用于描述定时任务
  - 等等...

在 Kubernetes 项目中，推崇的使用方法是：

1. 通过一个“编排对象”，如 Pod、Job、CronJob 等，来描述你试图管理的应用；
2. 为它定义一些“服务对象”，比如 Service、Secret、Horizontal Pod Autoscaler（自动水平扩展器）等。这些对象，会负责具体的平台级功能

这种使用方法，就是所谓的“声明式 API”。这种 API 对应的“编排对象”和“服务对象”，都是 Kubernetes 项目中的 API 对象（API Object）



# 安装

使用 kubeadm 安装时，master 节点至少需要 2 个 CPU，否则会报错！

步骤见[官网](https://kubernetes.io/docs/tasks/tools/install-kubectl/)，可以看[Kubernetes 中文社区](https://www.kubernetes.org.cn/5846.html)，也可查看 http://www.iexhaustion.com/2019/03/13/kubeadm/ 中的教程。

## Static Pod

在 Kubernetes 中有一种特殊的容器启动方法—— Static Pod。

把要部署的 Pod 的 yaml 文件放在一个指定目录中，这样当这台机器上的 kubelet 启动时会自动检查这个目录，加载所有的 Pod yaml 文件，然后在这台机器上启动它们。

在 kubeadm 中，master 组件(kube-apiserver、kube-controller-manager、kube-scheduler)的 yaml 文件会被生成在 **/etc/kubernetes/manifests** 路径下。假设要修改一个已有集群的 kube-apiserver 配置，就需要修改 kube-apiserver.yaml 文件。

etcd 也会通过 Static Pod 的方式启动，文件在同一目录下。

## 安装 kubeadm

```shell
# 安装依赖
yum install -y ebtables socat
# 替换镜像源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
# 下载
yum install -y kubelet kubeadm kubectl
# 关闭 swap
swapoff -a
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
# 停用 selinux：
sed -i s/SELINUX=enforcing/SELINUX=disabled/g /etc/selinux/config
setenforce 0
getenforce  #查看
```

修改 hostname：

```shell
# 修改 hostname
hostnamectl set-hostname zzk
# 查看修改结果
hostnamectl status
# 设置 hostname 解析
echo "127.0.0.1 $(hostname)" >> /etc/hosts
```

## 部署 master 节点

kubeadm 是一个部署工具，**把 kubelet 直接运行在宿主机上，然后使用容器部署其他的 Kubernetes 组件**。

```shell
# 查看需要依赖的镜像
kubeadm config images list
# 注意：先利用 `docker pull` 命令拉取镜像，
# 然后 `docker image tag 旧tag 新tag` 修改镜像名为上面得到的那些镜像！
```

`kubeadm init --config=配置文件` 部署 master 节点，可以不指定配置文件：

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
controllerManager:
    extraArgs:
        horizontal-pod-autoscaler-use-rest-clients: "true"
        horizontal-pod-autoscaler-sync-period: "10s"
        node-monitor-grace-period: "10s"
apiServer:
    extraArgs:
        runtime-config: "api/all=true"
kubernetesVersion: "stable-1.21"
```

如果提示 版本不一致，可以使用 `kubeadm config print init-defaults` 查看 kubeadm 默认支持的配置。

如果执行失败

```shell
# 重置
kubeadm reset
# 重新部署 master 节点，后面加上 --v=6 可以看到更详细的日志
kubeadm init --config xxx.yaml --v=6
```

执行完成后会有以下提示：

```shell
# 第一次使用 Kubernetes 集群所需要的配置命令
# Kubernetes 集群默认需要加密方式访问，这里是将刚刚部署生成的 Kubernetes 集群的安全配置文件，保存到当前用户的.kube 目录下，kubectl 默认会使用这个目录下的授权信息访问 Kubernetes 集群
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 将一个Node节点加入到当前集群中，在 worker 节点中执行的
kubeadm join 192.168.1.5:6443 --token etfhmx.idof4kx7ealogg3k --discovery-token-ca-cert-hash sha256:de1d693bc723009bb629eb43aa62b70324cdb1a945cdfaa4d956d624a5460095
# 如果没有记下该命令，可以使用下面的命令查看
kubeadm token create --print-join-command
```

## 部署网络插件

```shell
# 查看 Kubernetes 集群中节点的状态
> kubectl get nodes
NAME   STATUS     ROLES                  AGE     VERSION
kube   NotReady   control-plane,master   9m28s   v1.21.0

# 查看节点的详细信息、状态、事件
> kubectl describe node kube
# 会看到 master 节点 NotReady 的原因是因为网络插件 not ready

# 检查当前节点上各个系统 pod 状态
# kube-system 是 Kubernetes 项目预留的系统 Pod 的工作空间（Namepsace，注意它并不是 Linux Namespace，只是 Kubernetes 划分不同工作空间的单位）
> kubectl get pods -n kube-system
NAME                           READY   STATUS    RESTARTS   AGE
coredns-558bd4d5db-dgmgs       0/1     Pending   0          26m
coredns-558bd4d5db-lktkk       0/1     Pending   0          26m
etcd-kube                      1/1     Running   0          26m
kube-apiserver-kube            1/1     Running   0          26m
kube-controller-manager-kube   1/1     Running   0          26m
kube-proxy-47vsj               1/1     Running   0          26m
kube-scheduler-kube            1/1     Running   0          26m

# 部署 Weave 网络插件
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# 然后过一会重新使用 kubectl get pods -n kube-system 查看各个 pod 状态，可以看到所有的系统 pod 都成功启动了
NAME                           READY   STATUS    RESTARTS   AGE
coredns-558bd4d5db-8r5fb       1/1     Running   0          7m22s
coredns-558bd4d5db-ljx79       1/1     Running   0          7m22s
etcd-kube                      1/1     Running   0          7m21s
kube-apiserver-kube            1/1     Running   1          7m21s
kube-controller-manager-kube   1/1     Running   0          7m21s
kube-proxy-lb96r               1/1     Running   0          7m22s
kube-scheduler-kube            1/1     Running   0          7m21s
weave-net-mmrn8                2/2     Running   1          2m9s

# 查看 kubeadm.yaml 版本信息
kubeadm config print init-defaults
```

## 部署 Dashboard 插件

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

注：本地虚拟机访问时可能会有网络问题，可以将 yaml 文件下载下来后，直接 `kubectl apply -f xx.yaml`

可以给用户提供一个可视化的 Web 界面来查看当前集群的各种信息。

```shell
# 后面的命名空间可以在 yaml 文件中看到
> kubectl get pod -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-5594697f48-f5zqt   1/1     Running   1          148m
kubernetes-dashboard-57c9bfc8c8-5npxp        1/1     Running   1          148m
```

Dashboard 是一个 Web Server，在公有云上暴露 Dashboard 的端口，可能会造成安全隐患。1.7 版本之后的 Dashboard 项目部署完成后，默认只能通过 Proxy 的方式在本地访问。

如果要在集群外访问这个 Dashboard，需要用到 Ingress。

## 部署容器存储插件

**容器是无状态的**，如果在某台机器上启动一个容器，是看不到其他机器上容器在它们的数据卷中写入的文件的。

容器的持久化存储是用于保存容器存储状态的重要手段：存储插件会在容器中挂载一个基于网络或其他机制的远程数据卷，使得在容器中创建的文件实际保存在远程存储服务器上，或者以分布式的方式保存在多个节点上，而与当前宿主机没有任何绑定关系。

这样在其他宿主机上启动新的容器后，都可以请求挂载指定的持久化存储卷，从而访问到数据卷中保存的内容。

[Rook 项目](https://rook.github.io/docs/rook/master/ceph-quickstart.html)是基于 Ceph 的 Kubernetes 存储插件。

```shell
git clone --single-branch --branch master https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl create -f crds.yaml -f common.yaml -f operator.yaml
kubectl create -f cluster.yaml
```

部署完成后

```shell
kubectl get pods -n rook-ceph
```

这样，一个基于 Rook 持久化存储集群就以容器的方式运行起来了，而接下来在 Kubernetes 项目上创建的所有 Pod 就能够通过 Persistent Volume（PV）和 Persistent Volume Claim（PVC）的方式，在容器里挂载由 Ceph 提供的数据卷了。

而 Rook 项目，则会负责这些数据卷的生命周期管理、灾难备份等运维工作。

## kubeadm init

`kubeadm init` 工作流程如下：

1. pre-flight checks，进行一系列检查工作(Preflight Checks)；
2. 生成 Kubernetes 对外提供服务所需的各种证书，都放在 master节点的 /etc/kubernetes/pki 目录下(Kubernetes 对外提供服务时，默认通过 HTTPS 才能访问 kube-apiserver，所以需要为 Kubernetes集群配置好证书文件)
   1. 其中最主要的证书文件是 ca.crt 核对应的私钥 ca.key
   2. 用户通过 kubectl 获取容器日志等 streaming 操作时，需要通过 kube-apiserver 项 kubelet 发起请求，而这个连接也必须时安全的，证书文件为 apiserver-kubelet-client.crt，私钥为 apiserver-kubelet-client.key
3. 为其他组件生成访问 kube-apiserver 所需的配置文件，放在 /etc/kubernetes/xxx.conf；
   1. 记录了当前 master 节点的服务器地址、监听端口、证书目录等信息
   2. 对应客户端（scheduler、kubelet 等）可直接加载相应文件信息与 kube-apiserver 建立安全连接
4. 为 master 组件(kube-apiserver、kube-controller-manager、kube-scheduler)生成 Pod 配置文件；
   1. 这些组件都会被使用 Pod 方式部署（这里是一种特殊的容器启动方法 [Static Pod](#Static Pod)）
   2. 还会再生成一个 etcd 的 Pod yaml 文件，用来通过 Static Pod 方式启动 etcd
5. master 容器启动后，kubeadm 会通过检查 localhost:6443/healthz 这个 master 组件的健康检查 URL，等待 master 组件完全运行起来；
6. kubeadm 会为集群生成一个 bootstrap token，只要持有这个 token，任何一个安装了 kubelet 和 kubadm 的节点，都可以通过 `kubeadm join` 加入到这个集群当中；
7. kubeadm 会将 ca.crt 等 master 节点的重要信息，通过 ConfigMap 的方式保存在 etcd 当中，供后续部署 Node 节点使用。这个 ConfigMap 的名字是 cluster-info；
8. 安装默认插件(kube-proxy 和 DNS，分别用来提供整个集群的服务发现和 DNS 功能)。

## kubeadm join

`kubeadm init` 生成 bootstrap token 后，就可以在任意一台安装了 kubelet、kubeadm 的机器上执行 `kubeadm join`。

任何一台机器想要成为 Kubernetes 集群中的一个节点，就必须在集群的 kube-apiserver 中注册。

但是要想与 kube-apiserver 通信，这台机器就必须获取到相应的证书文件（CA 文件），但是又不能让用户去 master 节点手动拷贝这些文件。所以 kubeadm 至少需要发起一次“不安全模式”的访问到 kube-apiserver，从而拿到保存在 ConfigMap 中的 cluster-info（保存了 kube-apiserver 的授权信息）。而 bootstrap token，扮演的就是这个过程中的安全验证的角色。

只要有了 cluster-info 里的 kube-apiserver 的地址、端口、证书，kubelet 就可以以“安全模式”连接到 apiserver 上，这样一个新的节点就部署完成了

# 配置文件

K8s 虽然支持命令行方式直接原型容器(`kubectl run`)，但更推荐使用 yaml 配置文件的方式，把容器的定义、参数、配置记录在 yaml 文件中，然后使用 `kubectl create -f xx.yaml` 的方式运行。

一个 yaml 文件对应到 Kubernetes 就是一个 API Object(API 对象)，示例 nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment # 指定该 API 对象的类型为 Deployment(定义多副本应用（即多个副本 Pod）的对象)
metadata:        # metadata 是该对象的元数据，对于所有 API 对象而言，格式基本相同
  name: nginx-deployment
spec:              # spec 是该对象独有的定义，用于描述它所要表达的功能
  selector:        # 指定过滤规则
    matchLabels:
      app: nginx
  replicas: 2        # 指定满足上面 selectir 规则的 Pod 副本个数为 2
  template:          # Pod 摸板
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9   # 指定容器镜像
        ports:
        - containerPort: 80  # 指定容器监听端口
```

每个 API 对象都有一个叫做 metadata 的字段，该字段是 API 对象的标识，主要使用到的字段为 labels。labels 是一组 key-value 格式的标签。

Deployment 这样的控制器对象就可以通过 labels 字段从 Kubernetes 中过滤出它所关心的被控制对象。上例中，Deployment 会把所有正在运行的、携带 “app: nginx” 标签的 Pod 识别为被管理的对象，并确保这些 Pod 的总数严格等于两个。如果在这个集群中，携带 app=nginx 标签的 Pod 的个数大于 2 的时候，就会有旧的 Pod 被删除；反之，就会有新的 Pod 被创建。

而过滤规则的定义则是在 Deployment 的 “spec.selector.matchLabels” 字段，也成为 Label Selector。

```shell
kubectl create -f ngingx-deployment.yaml
# kubectl get 指令用于从 Kubernetes 中获取指定的 API 对象
# -l 用于获取所有匹配指定标签的 API 对象
kubectl get pods -l app=nginx  # 检查该 yaml 文件运行后的状态
# kubectl describe 指令查看每一个 API 对象的细节
# Kubernetes 执行过程中，对 API 对象的所有重要操作都会被记录在该对象的 Events 中，Debug 时很有用
kubectl describe pod nginx-deployment-5d59d67564-222lq
```

如果要对 Nginx 服务升级镜像版本，可以直接修改 yaml 文件中的镜像版本，然后：

```shell
kubectl replace -f nginx-deployment.yaml
```

推荐**无论是新建还是更新操作，统一使用 `kubectl apply -f xx.yaml`！**

**实际使用 Kubernetes 中，相比编写一个单独的 Pod 的 yaml 文件，更推荐使用一个 replicas=1 的 Deployment。后者可以在 Pod 所在节点故障时将其调度到健康的节点上，而单独的 Pod 只能在节点健康的情况下由 kubelet 保证 Pod 的健康状态**

# kubectl 命令

```shell
# 指定 namespace 进行其他操作
kubectl -n 指定namespace 其他操作
kubectl -n kube-system get pods # 获取 kube-syatem namespace 下的pod
kubectl -n kube-system exec -it etcd-vm210 sh # 进入 kube-syatem namespace 下的 etcd-vm210

# 创建 API 对象
kubectl create -f xx.yaml
# 更新镜像
kubectl replace -f xxx.yaml
# 建议创建、更新统一使用 apply
kubectl apply -f xx.yaml

# kubectl get 指令用于从 Kubernetes 中获取指定的 API 对象
# 指
# -l 用于获取所有匹配指定标签的 API 对象
kubectl get pods -l app=nginx  # 检查该 yaml 文件运行后的状态
# 将指定的 API 对象以 yaml 方式展示，如：kubectl get configmaps ui-config -o yaml
kubectl get API类型 对象名 -o yaml
# DESIRED 代表用户期望的 Pod 副本数（spec.replicas 值）
# CURRENT 当前处于  Running 状态的 Pod 数
# UP-TO-DATE 当前处于最新版本的 Pod 个数
# AVAILABLE 当前已经可用的 Pod（既是 Running、又是最新版本、且处于 Ready 状态）个数
kubectl get deployments
# 查看 Deployment 所控制的 ReplicaSet
kubectl get rs

# 实时查看 Deployment 对象的状态变化
kubectl rollout status deployment/nginx-deployment

# kubectl describe 指令查看每一个 API 对象的细节
# Kubernetes 执行过程中，对 API 对象的所有重要操作都会被记录在该对象的 Events 中，Debug 时很有用
kubectl describe pod nginx-deployment-5d59d67564-222lq

# 删除资源
kubectl delete -f xxx.yaml # 根据 yaml 文件删除对应的资源
kubectl delete pods xx
kubectl delete nodes xx

# 查看 Pod 日志
kubectl logs test-downwardapi-volume
```
