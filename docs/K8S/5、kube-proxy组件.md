
### **1、主要作用**

每台机器上都运行一个 kube-proxy进程服务，负责Node节点内部Pod之间的负载均衡和服务发现等。比如：Pod和Pod之间的网络通信就是通过kube-proxy来实现的。

### **2、kube-proxy 当前支持以下几种实现**

* userspace：最早的负载均衡方案，它在用户空间监听一个端口，所有服务通过 iptables 转发到这个端口，然后在其内部负载均衡到实际的 Pod。该方式最主要的问题是效率低，有明显的性能瓶颈。
* iptables：目前推荐的方案，完全以 iptables 规则的方式来实现 service 负载均衡。该方式最主要的问题是在服务多的时候产生太多的 iptables 规则，非增量式更新会引入一定的时延，大规模情况下有明显的性能问题
* ipvs：为解决 iptables 模式的性能问题，v1.11 新增了 ipvs 模式（v1.8 开始支持测试版，并在 v1.11 GA），采用增量式更新，并可以保证 service 更新期间连接保持不断开
* win-userspace：同 userspace，但仅工作在 windows 节点上
