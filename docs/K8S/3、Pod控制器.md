
### **1、在kubernetes中，有很多类型的pod控制器，每种都有自己的适合的场景，常见的有下面这些：**

* ReplicationController：比较原始的pod控制器，已经被废弃，由ReplicaSet替代
* ReplicaSet：保证副本数量一直维持在期望值，并支持pod数量扩缩容，镜像版本升级
* Deployment：通过控制ReplicaSet来控制Pod，并支持滚动升级、回退版本
* Horizontal Pod Autoscaler（简称HPA）：可以根据集群负载自动水平调整Pod的数量，实现削峰填谷
* DaemonSet：在集群中的指定Node上运行且仅运行一个副本，一般用于守护进程类的任务
* Job：它创建出来的pod只要完成任务就立即退出，不需要重启或重建，用于执行一次性任务
* Cronjob：它创建的Pod负责周期性任务控制，不需要持续后台运行
* StatefulSet：管理有状态应用

### **2、ReplicaSet(RS)控制器**

ReplicaSet的主要作用是保证一定数量的pod正常运行，它会持续监听这些Pod的运行状态，一旦Pod发生故障，就会重启或重建。同时它还支持对pod数量的扩缩容和镜像版本的升降级。
ReplicaSet 和 ReplicationController 都是副本控制器，其中：
* 相同点：都有前面所描述的功能
* 不同点：标签选择器的功能不同。ReplicaSet 可以使用标签选择器进行 单选 和 复合选择；而 ReplicationController 只支持 单选操作。

### **3、Deployment(Deploy)控制器**

为了更好的解决服务编排的问题，kubernetes在V1.2版本开始，引入了Deployment控制器。值得一提的是，这种控制器并不直接管理pod，而是通过管理ReplicaSet来简介管理Pod，即：**<span style='color:red'>Deployment管理ReplicaSet，ReplicaSet管理Pod。</span>**

Deployment主要功能有下面几个：
* 支持ReplicaSet的所有功能
* 支持发布的停止、继续
* 支持滚动升级和回滚版本

### **4、Horizontal Pod Autoscaler（简称 HPA控制器）**

与 ReplicationController(RC)、Deployment 一样，也属于一种Kubernetes 资源对象。<span style='color:red'>HPA 的 实现原理</span>:通过追踪分析 RC 控制的所有目标 Pod 的负载变化情况，来确定是否需要针对性地调整目标 Pod 的副本数。
Kubernetes 对 Pod 扩容与缩容提供了手动和自动两种模式，手动模式通过 kubectl scale命令对一个 Deployment/RC 进行 Pod 副本数量的设置。自动模式则需要用户根据某个性能指标或者自定义业务指标，并指定 Pod 副本数量的范围，系统将自动在这个范围内根据性能指标的变化进行调整。
（1）手动扩容和缩容
kubectl scale deployment frontend --replicas 1
（2）自动扩容和缩容
HPA 控制器基于Master 的 kube-controller-manager 服务启动参数 --horizontal-pod-autoscaler-sync-period 定义的时长(默认值为 30s),周期性地监测 Pod 的 CPU 使用率，并在满足条件时对 RC 或 Deployment 中的 Pod 副 本数量进行调整，以符合用户定义的平均Pod CPU 使用率。

### **5、DaemonSet控制器**

DaemonSet 确保全部（或者一些）Node 上运行一个 Pod 的副本。当有 Node 加入集群时，也会为他们新增一个 Pod 。当有 Node 从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。 使用 DaemonSet 的一些典型用法：
* 运行集群存储 daemon，例如在每个 Node 上运行 glusterd、ceph。
* 在每个 Node 上运行日志收集 daemon，例如filebeat、logstash。
* 在每个 Node 上运行监控 daemon，例如 Prometheus Node Exporter、collectd、Datadog 代理、Zabbix Agent。

### **6、StatefulSet控制器**

通过deployment控制器部署无状态应用。如果想部署有状态应用需要使用StatefulSet控制器

#### 6.1 无状态和有状态
无状态概念：
* 认为pod都是一样的
* 没有顺序要求
* 不用考虑在哪个node上运行
* 随意进行伸缩和扩展
有状态概念：
* 上述因素都要考虑到
* 每个pod都是独立的，保持pod的启动顺序（如mysql主从结构，先启动主再启动从）和唯一性（通过唯一的网络标识符进行区分）

#### 6.2、适合使用StatefulSets控制器的服务包含以下特点：
* 稳定性，唯一的网络标识符；
* 稳定性，持久化存储；
* 有序部署和扩展；
* 有序删除和终止；
* 有序的自动滚动更新。

#### 6.3、部署和扩展：
具有N个副本的StatefulSet，当部署Pod时顺序是从{0,1,2....N-1}开始创建；
删除时顺序从{N-1...2,1,0}删除；
在pod执行扩展和缩放之前，前面的所有pod必须处于Running和Ready状态；
