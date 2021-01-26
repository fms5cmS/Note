学习参考：https://www.prometheus.wang/quickstart/install-prometheus-server.html

[官网](https://prometheus.io/docs/prometheus/latest/getting_started/)

![](https://prometheus.io/assets/architecture.png)





端口暴露：

|       服务        | 端口 |
| :---------------: | :--: |
| Prometheus Server | 9090 |
|   node Exporter   | 9100 |
|      Grafana      | 3000 |



# Prometheus Server

```shell
# 下载 Prometheus
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.24.0/prometheus-2.24.0.linux-amd64.tar.gz 
# 解压后进入目录可直接运行 ./prometheus
```

Prometheus 架构中，Prometheus Server 并不直接服务监控特定的目标，而是负责数据的收集、存储，并对外提供数据查询支持。启动后可访问 9090 端口查看 Prometheus 的可视化管理界面。

- 可通过静态配置管理监控目标，也可以配合使用服务发现的方式动态监控目标，并从中获取数据；
- 本身是一个时序数据库，会将采集到的监控数据按时间序列的方式存储在本地磁盘中；
- 对外提供了自定义的 PromQL 语言，实现对数据的查询及分析。

Prometheus Server 的联邦集群能力可以使其从其他的 Prometheus Server 实例中获取数据，因此在大规模监控的情况下，可以通过联邦集群以及功能分区的方式对 Prometheus Server 进行扩展。

# Exporter

为了能监控到某些东西，如主机 CPU 使用率，就需要用大 Exporter，Prometheus 会周期性的从 Exporter 暴露的 HTTP 服务地址拉取监控样本数据。如采集主机运行指标(CPU、内存、磁盘等信息)可以使用 Node Exporter。

一般将 Exportes 分为两类：

- 直接采集：Exporter直接内置了对Prometheus监控的支持。如cAdvisor，Kubernetes，Etcd，Gokit等。
- 间接采集：原有监控目标并不直接支持Prometheus，需要通过 Prometheus 提供的 Client Library 编写该监控目标的监控采集程序。如： Mysql Exporter，JMX Exporter，Consul Exporter等。



## Node Exporter

Node Exporter 下载见 https://prometheus.io/download/，下载后解压即可直接运行，可以将解压后的二进制文件移动到 /usr/local/bin，然后就可以直接通过 node_exporter 运行了。

启动后可以访问 http://localhost:9100 看到 Node Exporter 的提示，点击 Metrics 或直接访问 http://localhost:9100/metrics 可以看到当前 Node Exporter 获取到的当前主机的所有监控数据。

每个监控之前前面会有两行注释，示例：

```shell
# HELP node_load1 1m load average.
# TYPE node_load1 gauge
node_load1 0
# HELP node_cpu_seconds_total Seconds the cpus spent in each mode.
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 9336.14
node_cpu_seconds_total{cpu="0",mode="iowait"} 20.59
```

其中 HELP 用于解释当前指标含义；TYPE 说明当前指标的数据类型。

上例中的两种指标分别为

- 当前主机一分钟内的负载情况；指标数据类型为仪表盘(gauge)
  - 系统负载会随系统资源使用而变化，故 node_load1 反应的是当前状态
- 每个进程(mode) 占用 CPU 的总时间；指标数据类型为计数器(counter)
  - CPU 占用时间是一个只增不减的指标，多以数据类型是计数器

还会有其他的指标，如：

- node_boot_time：系统启动时间
- node*disk**：磁盘IO
- node*filesystem**：文件系统用量
- node*memeory**：内存使用量
- node*network**：网络带宽
- node_time：当前系统时间
- go_*：node exporter 中 go 相关指标
- process_*：node exporter自身进程相关运行指标

Node Exporter 监控到数据后，Prometheus Server 就可以从 Node Exporter 中获取数据了，需要修改 Prometheus 的配置文件，在 prometheus.yml 的 scrape_config 节点下添加内容：

```yaml
scrape_configs:
	# prometheus 这个 job 是默认的
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'node'  # 新增
    static_configs:
    - targets: ['localhost:9100']
```

重启 Prometheus Server 后，访问 http://localhost:9090 ，输入 `up` 并执行，如果 Prometheus 可以正常从 Node Exporter 中读取数据，会看到（1表示正常，0为异常）：

```
up{instance="localhost:9090", job="prometheus"}             1
up{instance="localhost:9100", job="node"}                   1
```



## 任务和实例

当需要采集不同的监控指标(如：主机、MySQL、Nginx)时，只需要运行相应的监控采集程序，并且让Prometheus Server 知道这些 Exporter 实例的访问地址即可。

在 Prometheus 中，每一个暴露监控样本数据的HTTP服务称为一个**实例(Instance)**。如上面的 Node Exporter 就可以被称为一个实例。

而一组用于相同采集目的的实例，或者同一个采集进程的多个副本则通过一个一个**任务(Job)**进行管理。

当前的每个 Job 采用的是静态配置(static_configs)的方式来指定监控目标，除了这种静态配置每个 Job 的采集 Instance 地址外，Prometheus 还支持与 DNS、Consul、Kubernetes 等集成，从而自动发现 Instance，并从这些 Instance 上获取监控数据。

之前是通过 `up` 来查询所有 Instance 的状态，还可以通过 Prometheus 的 Status ——> Targets 页面查看所有的监控采集任务及各个任务下所有实例的状态。

# AlertManager

Prometheus Server 中支持基于 PromQL 创建告警规则，如果满足 PromQL 定义的规则，则会产生一条告警，而告警的后续处理流程则由 AlertManager 进行管理。

在 AlertManager 中我们可以与邮件，Slack等等内置的通知方式进行集成，也可以通过 Webhook 自定义告警处理方式。AlertManager 即 Prometheus 体系中的告警处理中心。



# PushGateway

由于 Prometheus 数据采集基于 Pull 模型进行设计，因此在网络环境的配置上必须要让 Prometheus Server 能够直接与 Exporter 进行通信。 

而当这种网络需求无法直接满足时，就可以利用 PushGateway 来进行中转。可以通过 PushGateway 将内部网络的监控数据主动 Push 到 Gateway 当中。而 Prometheus Server 则可以采用同样 Pull 的方式从 PushGateway 中获取到监控数据。



# PromQL

在访问 Prometheus 的可视化管理界面时，在 Graph 面板中，用户可以通过 PromQL 实时查询监控数据。如通过 `node_load1` 查询 Prometheus 采集到的主机负载样本数据，数据会以时间顺序展示，形成主机负载随时间变化的趋势图。

PromQL是Prometheus自定义的一套强大的数据查询语言，除了**使用监控指标作为查询关键字**以外，还内置了大量的**函数**，帮助用户进一步对时序数据进行处理。

如 `rate()` 函数可以计算单位时间内样本数据的变化情况，即增长率。

# 可视化

大多数场景下引入监控系统后还需要构建长期使用的监控数据可视化面板(Dashboard)，可以考虑类似 Grafana 这样的第三方可视化工具，Grafana 是一个开源的可视化平台，提供了对 Prometheus 的完整支持。

```shell
docker run -p 3000:3000 --name grafana -d grafana/grafana
```

然后可以访问 http://localhost:3000 进入 Grafana 界面中，默认账户为 admin/admin。

注：添加好 Dashboard，输入查询的表达式后需要键入 Shift+Enter 才会成功显示。

在 https://grafana.com/grafana/dashboards 可以找到大量可直接使用的 Dashboard。

