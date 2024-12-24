- 

# K8S集群搭建教程

## 1 前述

### 1.1 云原生定义

- Pivotal《传统应用和SOA向云原生转型指南》（2015）的云原生的特征：

  - 符合 12 要素应用

    - 基准代码：一份基准代码，多份部署（类似于Git版本控制系统的main分支）；
    - 显示声明所有依赖关系：通过依赖清单，确切地声明所有依赖项；
    - 把后端服务当作附加资源：各种后端服务（如数据库、消息队列、邮件服务、缓存系统），不区别对待本地或第三方服务；
    - 构建、发布、运行：严格区分构建、发布、运行这三个步骤；
    - 无状态进程：应用的进程必须无状态；
    - 端口绑定：互联网应用通过端口绑定来提供服务，并监听发送至该端口的请求。应用完全自我加载，不依赖于任何网络服务器；
    - 并发：通过进程模型进行扩展。进程是一等公民；
    - 易处理：进程可以瞬间开启或停止，有利于快速、弹性的伸缩应用。进程应追求最小启动时间；进程一旦接受终止信号就会优化的终止；进程在面对突然死亡时保持健壮；
    - 开发环境与线上环境等价：尽可能的保持开发，预发布，线上环境相同，以尽量做到持续部署；
    - 日志：应用本身从不存储自己的输出流，不应该试图去写或者管理日志文件，相反，每一个运行的进程都会直接的标准输出（stdout）事件流；
    - 管理进程：后台管理任务当作一次性进程运行；

  - 面向微服务架构

    - 微服务将分解为多个“仅做好一件事”的可独立部署的服务。这件事通常代表某项业务能力，或者最小可提供业务价值的“原子“服务单元。具备以下优点：

      单体系统

      - 变更周期解耦：只要变更限于单一有界的环境，并且服务继续履行其现有合约；实现了更频繁和快速的部署，从而实现了持续的价值流动；
      - 减少业务领域和现有代码的学习负担；
      - 可以加快采用新技术的步伐；
      - 提供独立、高效的服务扩展；

  - 自服务敏捷架构（可以认为是DevOps）：

    - 一个能够持续部署和运行这些微服务的平台；如代码以Git形式“推送”。 然后，自服务敏捷平台构建应用程序工件，构建应用程序环境，部署应用程序，并启动必要的进程。 团队不必考虑他们的代码在哪里运行或如何到达那里，这些对用户都是透明得，因为平台会关注这些。

  - 基于 API 的协作

  - 抗脆弱性

- 云原生计算基金会（2015）定义的特征

  - 应用容器化
  - 面向微服务架构
  - 应用支持容器的编排制度

- 云原生计算基金会（2018）定义

  - 云原生技术有利于各组织在公有云、私有云和混合云等新型动态环境中，构建和运行可弹性扩展的应用。云原生的代表技术包括容器、服务网格、微服务、不可变基础设施和声明式 API
  - 这些技术能够构建容错性好、易于管理和便于观察的松耦合系统。结合可靠的自动化手段，云原生技术使工程师能够轻松地对系统作出频繁和可预测的重大变更

### 1.2 容器、虚拟机、Docker、Openstack 和 K8S

- **容器&虚拟机**：均为虚拟化技术，容器更为轻量化、效率更高、启动更快；虚拟机需数分钟启动，容器仅需数十毫秒；
- **Docker**： 容器化虚拟技术事实上的标准；
- **OpenStack**：分布式的虚拟机服务平台，相比于普通的虚拟机软件（如Vmare），多了分布式虚拟机调度管理的功能和节点的负载均衡；
- **K8S**：分布式的容器调度管理平台，相比于Docker，多了分布式的容器调度管理和节点的负载均衡；
- **注意**：常见的中文资料均言K8S是容器编排软件，这里的编排是指调度、管理的意思，而非工作流编排的意思，容易有误导性；
- **注意**：无论是Openstack还是K8S，均不支持跨节点的容器或虚拟机的创建；所以将多台电脑合并成一台电脑的想法是不现实的；

### 1.3 K8S 和 云原生

在单机上运行容器，无法发挥它的最大效能，只有形成集群，才能最大程度发挥容器的良好隔离、资源分配与编排管理的优势，而对于容器的编排管理，Swarm、Mesos 和 Kubernetes 的大战已经基本宣告结束，Kubernetes 成为了无可争议的赢家。

- Kubernetes 成为云原生应用的基石
- 有机会成为跨云的真正的云原生应用的操作系统

### 1.4 K8S 介绍

- **官方**：**Kubernetes** 也称为 **K8S**，是用于自动部署、扩缩和管理容器化应用程序的开源系统。
- **发展历史**：由**Google**设计并捐赠给**Cloud Native Computing Foundation**（今属**Linux**基金会）来使用。
- **能力**：**Google** 每周运行数十亿个容器，能够在不扩张运维团队的情况下进行规模扩展。
- **作用**： 相当于一个操作系统，可以快速提供**PaaS**服务：1）创建各种容器化测试化环境；2）发布各种容器化服务；3）快速安装各种容器化服务，如MongoDB、**Hbase**、**Postgresql**、**Redis**、**Spark**等；快速提供IaaS服务：通过安装**Openstack**或**KubeVirt**等软件；快速提供**FaaS**服务：通过安装**Kube** **Native**等软件；

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/image-20230414170948460.png

image-20230414170948460

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/image-20230414171119324.png

image-20230414171119324

### 1.5 基本概念

- **Container**：轻量级的系统虚拟化技术，使用namespace隔离环境。
- **Pod**：**K8S** 的调度的基本单位，**Pod**是一组紧密关联的容器集合，内部的容器共享PID、IPC、Network和UTS namespace。可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/image-20230415141540422.png

```
    Pod的设计理念基础是微服务，不同类型的业务组合由不同类型的Pod执行，一个Pod对应一个微服务
```

- **Node:** 是Pod运行的主机，可以为物理机，也可以为虚拟机。每个Node上要运行container runtime （docker或者rkt）、kubelet和kube-proxy服务

  !https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/image-20230415142123300.png

- **Service:** 一个Pod只是一个运行服务的实例，可能在一个节点上停止，在另一个节点以一个新的IP启动一个新的Pod。在K8S集群中，客户端需要访问的服务就是Service对象。每个Service会对应一个集群内部有效的虚拟IP，集群内部通过虚拟IP访问一个服务。

- **Kubelet:** 每个Node的任务和资源管理

- **Kube-proxy:** 负责每个节点的硬件负载均衡

### 1.6 K8S 常见命令

- 运行一个Nginix容器：

  - `kubectl run --image=nginx:alpine nginx-app --port=80`

- Kubectl 命令：

  - `kubectl get - 类似于 docker ps，查询资源列表`
  - `kubectl describe - 类似于 docker inspect，获取资源的详细信息`
  - `kubectl logs - 类似于 docker logs，获取容器的日志`
  - `kubectl exec - 类似于 docker exec，在容器内执行一个命令`
  - `kubectl create -f file.yaml – 根据yaml创建Deployment资源`
  - `kubectl delete 删除命令，可删除node、pod、svc、depolyment`

- Volume：

  - 一个**Pod**一旦发生异常，**Pod** 产生的数据会随着 **Pod** 消亡而自动消失。**Volume** 用于持久化容器数据。
  - 如：为 **redis** 容器指定一个 **hostPath** 来存储 **redis** 数据

- Service：

  - kubectl创建Pod，Pob的IP地址会随着Pod的重启而变化

  - 为了访问Pod提供的服务，采用Service提供为一组Pod一个统一的入口，并提供负载均衡和

    自动服务发现

    。

    - `kubectl expose deployment nginx-app --port=80 --target-port=80 --type=NodePort`

- Replicas set：

  - 在一个Service中,可为Pod设置数个副本，以确保服务永不掉线
    - `kubectl scale --replicas=3 deployment/nginx-app`

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/image-20230415143046325.png

- 滚动升级（Rolling Update）：
  - 滚动升级（Rolling Update）通过逐个副本容器替代升级的方式来实现无中断的服务升级：
    - `kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2`
  - 滚动升级中若发生错误，可随时回滚：
    - `kubectl rolling-update frontend-v1 frontend-v2 --rollback`
- 资源限制：
  - K8S通过 cgroups 提供容器资源管理的功能，可限制每个容器的 CPU 和内存使用，比如对于刚才创建的 deployment，可以通过下面的命令限制 nginx 容器最多只用 50% 的 CPU 和 128MB 的内存：
    - `kubectl set resources deployment nginx-app -c=nginx --limits=cpu=500m,memory=128Mi`
  - 或者在yaml中指定资源限制
- 健康检查：
  - K8S Kubernetes 提供了两种探针（Probe，支持 exec、tcpSocket 和 http 方式）来探测容器的状态：
    - LivenessProbe：探测应用是否处于健康状态，如果不健康则删除并重新创建容器
    - ReadinessProbe：探测应用是否启动完成并且处于正常服务状态，如果不正常则不会接收来自 Kubernetes Service 的流量

### 1.7 K8S 常用运维命令

- 查看pod，及所在的节点：
  - `kubectl get pods -o wide`
- 查看pod运行状态
  - kubectl describe pod -n xxx podname
  - kubectl logs -n xxx podname
- 拉取image失败
  - 使用镜像： [yaml中的docker.io替换为如m.daocloud.io/docker.io](http://xn--yamldocker-yt2pu689a.xn--iom-e88du92cl4n83f.daocloud.io/docker.io)
- **若有节点warn，回收垃圾失败：**

```bash
kubectl drain --delete-local-data --ignore-daemonsets NODENAME
kubectl uncordon NODENAME
```

## 2 K8S 集群基础环境部署

### 2.1 服务器

- **网络资源：**子网越牛越好，最低应为万兆
- 管理节点：
  - yudayu-2 (223.2.47.148)
- 计算节点：
  - yudayu-1 (223.2.47.229)
- 持久化存储资源：
  - 暂无：建议NFS

### 2.2 安装过程

### 2.2.1 前提条件

1. 节点之中不可以有重复的主机名、`MAC` 地址或 `product_uuid`

```bash
cat /sys/class/dmi/id/product_uuid
```

1. 检查网络适配器：若有多个网卡，确保每个node的子网通过默认路由可达
2. 防火墙开放端口(所有节点)

```bash
firewall-cmd --zone=public --add-port=443/tcp --permanent
 firewall-cmd --zone=public --add-port=6443/tcp --permanent
firewall-cmd --zone=public --add-port=2379-2380/tcp --permanent
firewall-cmd --zone=public --add-port=10250/tcp --permanent
firewall-cmd --zone=public --add-port=10259/tcp --permanent
firewall-cmd --zone=public --add-port=10257/tcp --permanent
firewall-cmd --zone=public --add-port=10256/tcp --permanent
firewall-cmd --zone=public --add-port=30000-32767/tcp --permanent
```

1. 关闭防火墙(所有节点）——  首选

```bash
systemctl stop firewalld 
systemctl disable firewalld 
```

1. 禁用swap

```jsx
vim /etc/fstab
# /dev/mapper/rl - swap     none                    swap    defaults （注释这行）
查看并停止使用当前swap
swapon -s 
swapoff /dev/dm-1
```

### 2.2.2 升级 Linux 内核到5+：

```jsx
yum makecache & yum -y update
```

### 2.2.3 安装Container Runtime(选用containerd)：

- **Docker-engine+cir-dockerd方案（舍弃）**：从kubernetes 1.24开始，dockershim已经从kubelet中移除，但因为历史问题docker却不支持kubernetes主推的CRI（容器运行时接口）标准，需要在kubelet和docker之间加上一个中间层cri-docker。cri-docker是一个支持CRI标准的shim。一头通过CRI跟kubelet交互，另一头跟docker api交互，从而间接的实现了kubernetes以docker作为容器运行时。但是这种架构缺点也很明显，**调用链更长，效率更低**。因此选用containerd作为容器runtime
- **containerd**方案: **containerd**是一个**docker**的容器**runtime**，成为**CNCF**的官方项目

官方安装教程：https://github.com/containerd/containerd/blob/main/docs/getting-started.md

```bash   
# **装containerd**
yum install -y yum-utils
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install containerd.io 
containerd config default > /etc/containerd/config.toml
# **安装CNI插件**：
# 下载cni-plugins.tar 从https://github.com/containernetworking/plugins/releases
# 在/opt/cni/bin下解压：
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
vim /etc/containerd/config.toml
# disabled_plugins = ["cri"] 中移除cri
systemctl restart containerd
systemctl enable containerd
```

### 2.2.4  **配置systemd cgroup驱动 for container runtime**

```jsx
vim /etc/containerd/config.toml

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true

sudo sed -i "s#SystemdCgroup\\ \\=\\ false#SystemdCgroup\\ \\=\\ true#g" /etc/containerd/config.toml
cat /etc/containerd/config.toml | grep SystemdCgroup

**# 修改sandbox镜像在config.toml中**
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
****systemctl restart containerd
```

### 2.2.5 安装 kubeadm, kubelet and kubectl（所有节点）

```bash
**# 登录校园网：**
curl <http://portal.njnu.edu.cn/portal_io/login> -d "username=lszh2021060810&password=20210608cm&submitBtn="
# 或者：
curl -G "http://172.28.255.156:801/eportal/portal/login" --data-urlencode "callback=dr1003" --data-urlencode "login_method=1" --data-urlencode "user_account=Temp.dky.model2" --data-urlencode "user_password=Temp_dky@model2.nnu"

**# 将 SELinux 设置为 permissive 模式（相当于将其禁用）**
sudo setenforce 0

sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.30/rpm/repodata/repomd.xml.key
EOF

yum install -y --nogpgcheck  kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet

**# 删除防止更新：**
rm -r /etc/yum.repos.d/kubernetes.repo

**# 检查版本**
kubeadm version
kubelet --version
kubectl version

**# 配置容器运行时，以便后续通过crictl管理 集群内的容器和镜像**
crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```

### 2.2.6 转发ipv4

```bash
cat > /etc/sysctl.conf << EOF
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
EOF
sysctl -p 

cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.conf.default.rp_filter=1
net.ipv4.conf.all.rp_filter=1
EOF

sysctl --system # 生效

# 加载内核模块
modprobe br_netfilter  # 网络桥接模块
modprobe overlay # 联合文件系统模块
lsmod | grep -e br_netfilter -e overlay
```

### 2.2.7 启动集群

```bash
# 提前拉取需要的image
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers

# 查看拉取的镜像
$ crictl images                                                            

# 初始化集群
# --apiserver-advertise-address 指定 Kubernetes API Server 的宣告地址，可以不设置让其自动检测
# 其他节点和客户端将使用此地址连接到 API Server
# --image-repository 指定了 Docker 镜像的仓库地址，用于下载 Kubernetes 组件所需的容器镜像。在这里，使用了阿里云容器镜像地址，可以加速镜像的下载。
# --service-cidr 指定 Kubernetes 集群中 Service 的 IP 地址范围，Service IP 地址将分配给 Kubernetes Service，以允许它们在集群内通信
# --pod-network-cidr 指定 Kubernetes 集群中 Pod 网络的 IP 地址范围。Pod IP 地址将分配给容器化的应用程序 Pod，以便它们可以相互通信。
kubeadm init \\
--image-repository registry.aliyuncs.com/google_containers \\
--kubernetes-version v1.30.3 \\
--service-cidr=20.1.0.0/16 \\
--pod-network-cidr=55.2.0.0/16

# worker node加入
kubeadm token create --print-join-command  
kubeadm token list
kubectl label node yudayu-1 node-role.kubernetes.io/worker=worker 
```

### 2.2.8 配置kubectl变量

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

### 2.2.10 将 master 作为node（管理节点上执行）

- 检查 node 是否存在污点，删除污点
  - 污点值有三种：
    - NoSchedule：一定不被调度
    - PreferNoSchedule：尽量不被调度【也有被调度的几率】
    - NoExecute：不会调度，并且还会驱逐Node已有Pod

```bash
kubectl describe nodes xxx |grep Taints
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

### 2.2.9 安装 Pod 网络插件（CNI：Container Network Interface）(master)

你必须部署一个基于 Pod 网络插件的 容器网络接口 (CNI)，以便你的 Pod 可以相互通信。

- 下载 **yml** 配置文件

```bash
curl <https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml> -O
```

- 编辑**kube-flannel.yml，修改子网**    **10.244.0.0/16**   **为** 55.2.0.0/16
- 找到如下位置，添加 - --iface-regex=^192.168..

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/image-20230408114839608.png

```bash
kubectl apply -f kube-flannel.yml
#拉取镜像失败，则将yaml中，替换docker.io为m.daocloud.io/docker.io
```

## 3  dashboard 仪表盘

```bash
# Add kubernetes-dashboard repository
helm repo add kubernetes-dashboard <https://kubernetes.github.io/dashboard/>
helm pull kubernetes-dashboard/kubernetes-dashboard –untar
Vim values.yaml
# 替换仓库docker.io为镜像如，m.daocloud.io/docker.io
helm upgrade --install kubernetes-dashboard ./kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
# 将其中的，type: ClusterIP修改成type: NodePort，保存退出即可。
kubectl get svc -A
# 权限认证：
kubectl apply -f <https://kuboard.cn/install-script/k8s-dashboard/auth.yaml>
kubectl -n kubernetes-dashboard create token admin-user 
#过期了，重新运行上句命令得到key
```

## 4 包管理工具Helm

```bash
curl -fsSL -o get_helm.sh <https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3>
chmod 700 get_helm.sh
sudo ./get_helm.sh
```

## 6 为 K8S 创建 PV 持久卷

### 6.1 PV和PVC

- 持久卷(PersistentVolume，PV)是集群中由管理员配置的一段网络存储。它是集群中的资源，就像节点是集群资源一样。PV持久卷和普通的Volume一样，也是使用卷插件来实现的，只是它们拥有独立于任何使用PV的Pod的生命周期。此API对象捕获存储实现的详细信息，包括NFS，iSCSI或特定于云提供程序的存储系统。
- 持久卷申领(PersistentVolumeClaim，PVC)表达的是用户对存储的请求。概念上与Pod类似。Pod会耗用节点资源，而PVC申领会耗用PV资源。

### 6.2 用 storageClass 动态创建 PV

- 对1PB的大量目录创建NFS服务，gisweb1-4，以gisweb4为例子

```bash
安装NFS:
yum -y install nfs-utils rpcbind
```

- 设置持久卷权限

```bash
# 执行权限chown -R nobody:nfsnobody /mnt/storage/k8s/pv
#chmod -R 777 /mnt/storage/k8s/pv
```

- 配置 nfs

```bash
vim /etc/exports
# 添加：/mnt/storage/k8s/pv 192.168.0.0/24(rw,sync,no_root_squash)# 以上设置让所有的 IP 都有效
systemctl start rpcbind
systemctl enable rpcbind
systemctl enable nfs
systemctl start nfs
systemctl start nfs-server
systemctl enable nfs-server
systemctl start firewalld
firewall-cmd --permanent --add-service=nfs
firewall-cmd  --reloadsystemctl stop firewalld && sudo systemctl disable firewalld
```

- 检查

```bash
exportfs -rvshowmount -e 127.0.0.1
```

- 所有节点安装nfs客户端

```bash
yum install -y nfs-utils
# 每个节点挂载nfs客户端的存储目录，本次nfs客户端在gisweb4（192.168.0.204）上mount -t nfs 192.168.0.204:/mnt/storage/k8s/pv /mnt/storage/k8s/pv
# 检查挂载情况df -h
```

- 安装nfs-client-provisioner (需要翻墙)

参考：[**](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner**

```bash
# 更新helm repohelm repo update
# 搜索helm库中nfs版本helm search repo nfs-subdir-external-provisioner
# 添加 helm 仓库helm repo add nfs-subdir-external-provisioner <https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/>
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \\--set nfs.server=192.168.0.204 \\--set nfs.path=/mnt/storage/k8s/pv   \\--set image.repository=registry.cn-hangzhou.aliyuncs.com/xzjs/nfs-subdir-external-provisioner \\--set image.tag=v4.0.0
```

- 手动安装 nfs-client-provisioner

参考：http://www.mydlq.club/article/109/#创建-nfs-subdir-external-provisioner-部署文件

- 成功后，安装时添加持久化参数，其中 nfs-storage 为安装的 storageclass 的 provisioner 字段名字

## 7 安装 kubeAPPS 可视化软件管理工具

参考：[**](https://kubeapps.dev/docs/latest/tutorials/getting-started/)https://kubeapps.dev/docs/latest/tutorials/getting-started/**

- 安装

```bash
# 添加 kubeapps 仓库helm repo add bitnami <https://charts.bitnami.com/bitnami>
# 创建 kubeapps 的命名空间kubectl create namespace kubeapps
# 安装helm install kubeapps --namespace kubeapps bitnami/kubeapps
```

- 创建证书

```bash
# 创建用于访问 Kubeapps 和 Kubernetes 的演示凭证kubectl create --namespace default serviceaccount kubeapps-operator
kubectl create clusterrolebinding kubeapps-operator --clusterrole=cluster-admin --serviceaccount=default:kubeapps-operator
cat <<EOF | kubectl apply -f -apiVersion: v1kind: Secretmetadata:  name: kubeapps-operator-token  namespace: default  annotations:    kubernetes.io/service-account.name: kubeapps-operatortype: kubernetes.io/service-account-tokenEOF
```

- 查看令牌 token

```bash
kubectl get --namespace default secret kubeapps-operator-token -o go-template='{{.data.token | base64decode}}'
eyJhbGciOiJSUzI1NiIsImtpZCI6IkdVQTZzb3JEM1FHdkpxVDNsSEwtVEZWc2hyR08tbmFFWnFGX2Q2OGt5cEkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Imt1YmVhcHBzLW9wZXJhdG9yLXRva2VuIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Imt1YmVhcHBzLW9wZXJhdG9yIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNTNjY2M0N2YtZWFmMS00NDY4LWJkN2ItYTVhMzliMzJjMzExIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6a3ViZWFwcHMtb3BlcmF0b3IifQ.qsTBQODZLD1EUP5WjF_ju0-_ZFoJa2pEGCGf2zoLK71TjZeytD0GUGp4Z5ACNFuJMtedtx8tRgWhioU2oimxGdCIL4f7Szt0dOQgXD15HmoiUjYEcDQNsfTdcmfZw-m3-zwtTqa3kTTG3Wio0wf_f_ayw8qZCDL2i3PK-7h0QeAb1rQhtCz_e8huNrcshjixGlyw8aKUvdi2hPe6yvpxKJqQeOalNhT22b-ax28oIyqmC-NXYUMyRbEsgOjyuJAv6XdjqsQKbOGMKsTtNyf7CvnHl88hfRZpF0W-GuKj1ggKGYClTHuXnsv9QP-AQN1UaEtcAbUp08bHN9isedJL6w
```

- 修改服务模式，将其改为 NodePort

```bash
# 因为是测试环境，因此直接采用 NodePort 方式暴露服务端口kubectl edit svc kubeapps -n kubeapps
```

- 找到端口，在安全组放行

```bash
kubectl get svc -A |grep kubeapps
```

- 访问：http://125.220.153.23:31885/

## 8 部署MetalLB - service load balancer for K8S

- 生产平台时，构建一个网段，专门用来服务发布

- kubectl edit configmap -n kube-system kube-proxy

- set:

  - apiVersion: [kubeproxy.config.k8s.io/v1alpha1](http://kubeproxy.config.k8s.io/v1alpha1) kind: KubeProxyConfiguration mode: "ipvs" ipvs: strictARP: true

- kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml

- **Layer 2 模式配置**

  - **创建 IPAdressPool**

  ```bash
  cat<<EOF > IPAddressPool.yaml
  apiVersion: metallb.io/v1beta1
  kind: IPAddressPool
  metadata:
    name: pool223
    namespace: metallb-system
  spec:
    addresses:
    - 55.2.1.20-55.2.1.40  #（注意修改）
  EOF
  
  kubectl apply -f IPAddressPool.yaml
  ```

  ------

  - **创建 IPAdvertisement：标记adresses pool的name，用于给不同的服务分配不同的网段**

  ```bash
  cat <<EOF > L2Advertisement.yaml
  apiVersion: metallb.io/v1beta1
  kind: L2Advertisement
  metadata:
    name: example123
    namespace: metallb-system
  spec:
    ipAddressPools:
    - pool223 #上一步创建的 ip 地址池，通过名字进行关联
  EOF
  
  kubectl apply -f L2Advertisement.yaml
  ```

## 9 在 K8S 上部署 PostgreSQL

- 安装
  - 注意：数据库安装需要持久卷，需提前创建满足要求的**pv**，或者创建**nas**的**stroageclass**，以自动根据**mysql**的**pvc**创建**pv**。
  - 集群已经配置23服务器的`/mnt/storage/k8s/pv`为NAS,并已经配置名字为**nas-storage**的**sc**

```bash
helm repo add bitnami <https://charts.bitnami.com/bitnami>
# 首先检查是否有oge这个命名空间，否则执行如下进行创建kubectl create ns oge
# postgresql 这个名字可以自己定义，但后面每一步都要注意对应更改helm install -n oge  bitnami/postgresql \\--set global.storageClass=nfs-client \\--set readReplicas.persistence.storageClass=nfs-client \\--set primary.persistence.storageClass=nfs-client \\--set primary.persistence.size=200Gi \\--set readReplicas.persistence.size=200Gi \\--set image.tag=14.5.0-debian-11-r6
helm install -n geoctap  bitnami/postgresql \\--set global.storageClass=nfs-client \\--set readReplicas.persistence.storageClass=nfs-client \\--set primary.persistence.storageClass=nfs-client \\--set primary.persistence.size=200Gi \\--set readReplicas.persistence.size=200Gi \\--set image.tag=14.5.0-debian-11-r6
# 指定版本，可在kubeapps里面查看# --set image.tag=14.5.0-debian-11-r6
```

- 查看 postgresql 密码

```bash
kubectl get secret --namespace oge postgresql -o jsonpath="{.data.postgres-password}" | base64 -d# 密码7jXf2gsmUX
```

- 更改服务端口

```bash
kubectl edit svc --namespace oge postgresql
# 将 type=ClusterIP 改为 NodePort# b8:85:84:71:64:28echo "SUBSYSTEM==\\"net\\", ACTION==\\"add\\", DRIVERS==\\"?*\\", ATTR{address}==\\" b8:85:84:71:64:28\\", ATTR{type}==\\"1\\", KERNEL==\\"eno*\\", NAME=\\"eno1\\"" >> /etc/udev/rules.d/70-persistent-net.rules
```

- 缩放副本集

```bash
kubectl get deployment
# 发现并没有postgresqlkubectl get all -n oge
# 发现有statefulset.apps/postgresql# 设置副本集个数为1kubectl scale --replicas=1 statefulset.apps/postgresql -n oge
```

- 命令行进入pgsql

```bash
# 进入pgsql的podkubectl exec -it -n oge postgresql-0 bash
# 用户登录psql -U postgres
# 输入密码7jXf2gsmUX
```

- 在pod外面执行sql

```bash
psql -h 125.220.153.23 -p 30865 -U postgres -W -f ./public.sql
```

## 10 在 K8S 上部署 MySQL

- 安装
  - 注意：数据库安装需要持久卷，需提前创建满足要求的`pv`，或者创建`nas`的 `stroageclass`，以自动根据postgresql的pvc创建pv。
  - 集群已经配置23服务器的`/mnt/storage/k8s/pv`为 `NAS`,并已经配置名字为 `nas-storage` 的 `sc`

```bash
helm repo add bitnami <https://charts.bitnami.com/bitnami>
# 安装helm install -n oge mysql bitnami/mysql \\--set global.storageClass=nfs-client \\--set readReplicas.persistence.storageClass=nfs-client \\--set primary.persistence.storageClass=nfs-client \\--set primary.persistence.size=200Gi \\--set readReplicas.persistence.size=200Gi
```

- 查看 MySQL 密码

```bash
kubectl get secret --namespace oge mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d# 密码VubCMiHvT1
```

- 更改服务端口

```bash
kubectl edit svc --namespace oge mysql
# 将type=ClusterIP改为NodePort# b8:85:84:71:64:28echo "SUBSYSTEM==\\"net\\", ACTION==\\"add\\", DRIVERS==\\"?*\\", ATTR{address}==\\" b8:85:84:71:64:28\\", ATTR{type}==\\"1\\", KERNEL==\\"eno*\\", NAME=\\"eno1\\"" >> /etc/udev/rules.d/70-persistent-net.rules
```

1. 缩放副本集

```bash
kubectl get deployment
# 发现并没有mysqlkubectl get all -n oge
# 发现有statefulset.apps/mysqlkubectl scale --replicas=1 statefulset.apps/mysql -n oge
```

1. 在K8S中进入数据库

```bash
kubectl exec -it -n oge mysql-1  bash
# 进入后登录用户mysql -u root -p# 输入密码
```

## 11 在K8S上部署 MongoDB

- 安装
  - 注意：数据库安装需要持久卷，需提前创建满足要求的pv，或者创建nas的stroageclass，以自动根据postgresql的pvc创建pv。
  - 集群已经配置23服务器的`/mnt/storage/k8s/pv`为NAS,并已经配置名字为nas-storage的sc

```bash
helm repo add bitnami <https://charts.bitnami.com/bitnami>
# 安装helm install -n ydy mongodb bitnami/mongodb \\--set global.storageClass=nfs-client \\--set readReplicas.persistence.storageClass=nfs-client \\--set primary.persistence.storageClass=nfs-client \\--set primary.persistence.size=100Gi \\--set readReplicas.persistence.size=100Gi
```

- 查看 MongoDB 密码

```bash
kubectl get secret --namespace ydy mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 -d# 密码WUL9FPQ2V9
```

- 更改服务端口

```bash
kubectl edit svc --namespace ydy mongodb
# 将type=ClusterIP改为NodePort# b8:85:84:71:64:28echo "SUBSYSTEM==\\"net\\", ACTION==\\"add\\", DRIVERS==\\"?*\\", ATTR{address}==\\" b8:85:84:71:64:28\\", ATTR{type}==\\"1\\", KERNEL==\\"eno*\\", NAME=\\"eno1\\"" >> /etc/udev/rules.d/70-persistent-net.rules
```

1. 缩放副本集

```bash
kubectl get deployment
# 发现并没有mongodbkubectl get all -n ydy
# 发现有statefulset.apps/mongodbkubectl scale --replicas=1 statefulset.apps/mongodb -n ydy
```

1. 在K8S中进入数据库

```bash
kubectl exec -it -n ydy mongodb-644c657c4f-x62cn bash
```

## 12 在 K8S 上部署 Apache Spark

两个方式，第一种方式为Spark官方提出的；第二种为Google提出的，更符合K8S原生概念

1. Spark On K8S
2. spark-on-k8s-operator

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/image-20230408170401365.png

image-20230408170401365

Spark On K8S

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/image-20230408170444023.png

image-20230408170444023

spark-on-k8s-operator

### 12.1 安装 spark-on-k8s-operator

参考 ：https://blog.csdn.net/w8998036/article/details/122217230

- 安装

```bash
helm repo add spark-operator <https://googlecloudplatform.github.io/spark-on-k8s-operator>
# 注意是否存在 spark-operator 命名空间，没有则创建kubectl create ns spark-operator
# 安装helm install spark-operator spark-operator/spark-operator --namespace spark-operator  --set sparkJobNamespace=default  --set webhook.enable=true
```

- 创建服务账户

```bash
vim spark-application-rbac.yaml
# 内容如下
apiVersion: v1kind: ServiceAccountmetadata:  name: spark  namespace: spark---apiVersion: rbac.authorization.k8s.io/v1kind: Rolemetadata:  namespace: spark  name: spark-rolerules:- apiGroups: [""]  resources: ["pods"]  verbs: ["*"]- apiGroups: [""]  resources: ["services"]  verbs: ["*"]---apiVersion: rbac.authorization.k8s.io/v1kind: RoleBindingmetadata:  name: spark-role-binding  namespace: sparksubjects:- kind: ServiceAccount  name: spark  namespace: sparkroleRef:  kind: Role  name: spark-role  apiGroup: rbac.authorization.k8s.io
kubectl create clusterrolebinding root-cluster-admin-binding --clusterrole=cluster-admin --user=root
```

- 编写作业模板并提交作业

*创建一个Spark作业的YAML配置文件，并进行部署。*

1. 创建spark-pi.yaml文件

```yaml
apiVersion: "sparkoperator.k8s.io/v1beta2"kind: SparkApplicationmetadata:  name: spark-pi  namespace: sparkspec:  type: Scala  mode: cluster  image: "registry.cn-hangzhou.aliyuncs.com/yudayu/spark:v3.1.1"  # 1gcr.io/spark-operator/spark:v3.1.1需要更换镜像，gcr.io目前国内无法访问。可以先对docker挂代理，pull到阿里云镜像后  imagePullPolicy: IfNotPresent  mainClass: org.apache.spark.examples.SparkPi  mainApplicationFile: "local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar"  # 需要更换自己的jar包，local指该jar位于image内，可换成所有节点都能访问的web路径，或者通过指定nas挂载pv，将jar包放在nas的pv里  sparkVersion: "3.1.1"  restartPolicy:    type: Never  volumes:    - name: "test-volume"      hostPath:        path: "/tmp"        type: Directory  driver:    cores: 1    coreLimit: "1200m"    memory: "512m"    labels:      version: 3.1.1    serviceAccount: spark    volumeMounts:      - name: "test-volume"        mountPath: "/tmp"  executor:    cores: 1    instances: 2    memory: "512m"    labels:      version: 3.1.1    volumeMounts:      - name: "test-volume"        mountPath: "/tmp"
```

1. 部署一个Spark计算任务

```bash
kubectl apply -f spark-pi.yaml
```

运维

```bash
kubectl get sparkapplications
kubectl describe sparkapplications
kubectl get svc  # 查看该任务的spark ui
```

### 12.2 安装 Spark On K8S

```bash
helm repo add bitnami <https://charts.bitnami.com/bitnami>
# 注意是否存在 spark-operator 命名空间，没有则创建kubectl create ns spark-on-k8s
helm install -n spark-on-k8s spark bitnami/spark \\  --set worker.coreLimit=28
./bin/spark-submit    \\    --class org.apache.spark.examples.SparkPi \\    --conf spark.kubernetes.container.image=bitnami/spark:3 \\    --master k8s://https://125.220.153.23:6443 \\    --conf spark.kubernetes.driverEnv.SPARK_MASTER_URL=spark://10.97.43.141:7077  \\--deploy-mode cluster \\  --executor-memory 20G \\  --num-executors 10 \\--conf spark.executor.instances=5 \\https:///data/spark-examples_2.12-3.3.0.jar 1000
kubectl run --namespace spark-on-k8s spark-oge --rm --tty -i --restart='Never' \\--image bitnami/spark:3 \\-- spark-submit --master spark://10.97.43.141:7077 \\--class org.apache.spark.examples.SparkPi \\    --deploy-mode cluster \\/data/spark-examples_2.12-3.3.0.jar 100000
```

## 13 在K8S上部署redis集群

- 待更

## 14 在K8S上部署nginx

### 14.1 创建pv

```bash
vim nginx-pv.yaml
apiVersion: v1kind: PersistentVolumemetadata:  name: nginx-ydy-pv  namespace: ydyspec:  capacity:    storage: 10Gi  accessModes:    - ReadWriteOnce  persistentVolumeReclaimPolicy: Retain  storageClassName: manual  hostPath:    path: /mnt/storage/k8s/pv/ydy-nginx-pvc
```

### 14.2 创建pvc

```bash
vim nginx-pvc.yaml
apiVersion: v1kind: PersistentVolumeClaimmetadata:  name: nginx-ydy-pvc  namespace: ydyspec:  accessModes:    - ReadWriteOnce  resources:    requests:      storage: 10Gi  storageClassName: manual
```

### 14.3 安装nginx并设置静态资源挂载的pvc

将nginx中的`/app`挂载到`/mnt/storage/k8s/pv/luluancheng-nginx-pvc`下

```bash
helm install -n ydy nginx bitnami/nginx \\--set staticSitePVC=nginx-ydy-pvc
```

## 附录：疑难问题解决：

### 1 CNI网络错误

- 当迁移集群之后，拉取镜像报cni网络错误，如下：

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/8d5d49703c8ac59f24fde81b3982b616.png

8d5d49703c8ac59f24fde81b3982b616

- 从上面的截图中看到问题出现在给Pod分配IP上，意思是 cni0 的IP不同于``10.244.9.1/24`，下面我们使用 `ifconfig`命令查看IP信息

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/79e65e4f797200ad98feac6f8b2d4254.png

79e65e4f797200ad98feac6f8b2d4254

- 从上面的图中我们可以看到`flannel.1`的 **IP** 为`10.244.9.0`，然后我们又使用`cat /run/flannel/subnet.env`，该文件内容如下：

!https://cdn.jsdelivr.net/gh/binwenwu/picgo_demo/img/310efbdb614636a17aa48eaf4a8dc2c5.png

310efbdb614636a17aa48eaf4a8dc2c5

- 其实现在的问题就比较明确了，我们使用的Overlay network为Flannel，也就是说Pod的IP地址段应该在Flannel的subnet下，而现在我们看到cni0的IP地址段与flannel subnet地址段不同，所以就出现了问题。
- 解决方案
  - 方法1是将 cni0 的 IP 段修改为`10.244.9.1`
  - 方法2是将这个错误的网卡删除掉，之后会自动重建

```bash
# 下面我们删除错误的cni0，然后让它自己重建ifconfig cni0 down
ip link delete cni0
```

### 2 kubeadmin安装失败重置

kubeadmin安全证书问题

![kubeadmin.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ad792d4a-5e14-4539-9667-46651c4cdd84/4f28c43e-f333-468e-bd1e-94dbe82ef56d/kubeadmin.png)

需要清除残留文件后重置kubeadmin环境

```jsx
rm -rf /etc/kubernetes/kubelet.conf /etc/kubernetes/pki/ca.crt
kubeadm reset
加入全局变量文件
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
source /etc/profile
```

### 4 异常断电等导致 etcd 心跳检测出现问题

- [Kubernetes API Server cannot be started after improper reboot](https://github.com/kubernetes/kubernetes/issues/107491)
- [K8S: etcd 集群备份灾难恢复操作手册](https://blog.51cto.com/liruilong/6060676)