# K8S

## 1. 概念

- **Container**：轻量级的系统虚拟化技术，使用namespace隔离环境。

- **Pod**：**K8S** 的调度的基本单位，**Pod**是一组紧密关联的容器集合，内部的容器共享PID、IPC、Network和UTS namespace。可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

- Node: 是Pod运行的主机，可以为物理机，也可以为虚拟机。每个Node上要运行container runtime （docker或者rkt）、kubelet和kube-proxy服务

- Service: 一个Pod只是一个运行服务的实例，可能在一个节点上停止，在另一个节点以一个新的IP启动一个新的Pod。在K8S集群中，客户端需要访问的服务就是Service对象。每个Service会对应一个集群内部有效的虚拟IP，集群内部通过虚拟IP访问一个服务。

- Kubelet: 每个Node的任务和资源管理

- Kube-proxy: 负责每个节点的硬件负载均衡

  

## 2. 组件

- Node: 可运行多个Pod
- Pod: Container的抽象
- Ingress: 集群外部的 HTTP 和 HTTPS 路由公开给集群内的 **Service**
- Service: Pod间的通信
- ConfigMap: 配置文件，如路径
- Secret: 安全配置，如密码
- Volume: 持久化数据卷
- Deployment: Pod的抽象，创建并管理ReplicaSet
  - ReplicaSet: 一组Pod的副本
- StatefulSet: 管理一组Pod， 适用于需要持久化存储（数据库）或稳定、唯一网络标识的应用（一般数据库在K8S以外部署，以避免使用statefull）

## 3. 结构

![image-20241220165517405](C:\Users\22171\AppData\Roaming\Typora\typora-user-images\image-20241220165517405.png)

- node（每个Node都需要安装以下三个process）
  - container runtime(docker)
  - Kubelet: 与Container和Node都能交互，创建Pod，分配Node的资源给Pod
  - KubeProxy: 转发请求，从Service转发到Pod
- cluster（
  - master node
    - API Server
      - cluster gateway（集群的门户，用户交互）
      - gatekeeper (网关)
    - Scheduler（调度器）
      - 启动Pod
      - 分配Pod至合适的worker node中运行（负载均衡）
    - Controller manager
      - detect cluster state change(when pod dies / when Kubelet restart new pods)
      - reschedule pods
    - etcd
      - cluster brain
      - store cluster changes in the key value store
  - slave node / worker node
  - 附加组件
    - kube-dns
    - ingress-controller 外部网络访问
    - Heapster, Prometheus 资源监控
    - Dashboard 控制台界面
    - Federation 跨可用区集群
    - Fluentd-elasticsearch 集群日志采集存储与查询

**Note**

- cluster由多个master node组成
- API Server是负载均衡的
- etcd形成分布式存储
- master比worker占用的资源少
- 可通过UI, API, CLI共3种方式与Api Server交互

## 4.  minikube

- 测试 / 本地集群设置 —— master and worker processes run on **one** machine
- 创建virtual box，因此需要Hypervisor（虚拟机管理程序）

## 5. kubectl

- Command line tool for K8s cluster

