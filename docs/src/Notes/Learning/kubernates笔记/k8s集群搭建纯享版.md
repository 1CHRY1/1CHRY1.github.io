# K8S集群搭建纯享版

### step1. 虚拟机环境设置

```
# 关闭所有防火墙
systemctl stop firewalld 
systemctl disable firewalld
```

```
# 禁用swap
vim /etc/fstab
# /dev/mapper/rl - swap     none                    swap    defaults （注释这行）
swapon -s 
swapoff /dev/dm-1
```

```
# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

```
# ipv4转发设置
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

```
#连校园网
curl -G "http://172.28.255.156:801/eportal/portal/login" --data-urlencode "callback=dr1003" --data-urlencode "login_method=1" --data-urlencode "user_account=Temp.dky.model2" --data-urlencode "user_password=Temp_dky@model2.nnu"
```

### step2. 软件安装

```
# 设置镜像
dnf config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo（镜像网站）
```

```
# 升级linux内核
yum makecache & yum -y update
```

```
# 装yum-utils
yum install -y yum-utils
```

```
# 安装containerd并配置
yum install containerd.io
containerd config default > /etc/containerd/config.toml

# 安装cni插件
# 下载cni-plugins.tar 从https://github.com/containernetworking/plugins/releases
# 在/opt/cni/bin下解压：
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
vim /etc/containerd/config.toml
# disabled_plugins = ["cri"] 中移除cri

# 配置containerd运行的systemd cgroup驱动控制器
vim /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
sudo sed -i "s#SystemdCgroup\\ \\=\\ false#SystemdCgroup\\ \\=\\ true#g" /etc/containerd/config.toml
cat /etc/containerd/config.toml | grep SystemdCgroup

# 在config.toml中修改sandbox基础镜像
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"

systemctl restart containerd
systemctl enable containerd
```

```
# 添加k8s仓库
cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.30/rpm/repodata/repomd.xml.key
EOF
```

```
# 安装kubelet，kubeadm，kubectl
yum install -y --nogpgcheck  kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet

# 删除并防止更新
rm -r /etc/yum.repos.d/kubernetes.repo

#版本检查
kubeadm version
kubelet --version
kubectl version
```

```
# 配置容器运行时，以便后续通过crictl管理 集群内的容器和镜像
crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```



### step3. 集群启动

```
# 提前拉取需要的image
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers

# 查看拉取的镜像
$ crictl images    

# 初始化集群
kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.30.3 --service-cidr=20.1.0.0/16 --pod-network-cidr=55.2.0.0/16
```

```
# worker node加入
kubeadm token create --print-join-command  
kubeadm token list
kubectl label node opengms-master node-role.kubernetes.io/worker=worker 
```

```
# 配置kubectl变量
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```



### step4. 插件安装

```
# 安装pod网络插件
# 下载 **yml** 配置文件
curl <https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml> -O
# 编辑**kube-flannel.yml，修改子网10.244.0.0/16 为 55.2.0.0/16
# 找到containers-args，添加 --iface-regex=^192.168..
# 替换docker.io为m.daocloud.io/docker.io
kubectl apply -f kube-flannel.yml
```

```
# 包管理工具
curl -fsSL -o get_helm.sh <https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3>
chmod 700 get_helm.sh
sudo ./get_helm.sh
```

