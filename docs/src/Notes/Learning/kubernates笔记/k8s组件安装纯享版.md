# K8S组件安装纯享版

### step1. Kuboard安装

官网教程：[安装 Kuboard v3 - kubernetes | Kuboard](https://www.kuboard.cn/install/v3/install-in-k8s.html#安装)

```
配置kuboard-v3.yml
---
apiVersion: v1
kind: Namespace
metadata:
  name: kuboard

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: kuboard-v3-config
  namespace: kuboard
data:
  # 关于如下参数的解释，请参考文档 https://kuboard.cn/install/v3/install-built-in.html
  # [common]
  KUBOARD_SERVER_NODE_PORT: '30080'
  KUBOARD_AGENT_SERVER_UDP_PORT: '30081'
  KUBOARD_AGENT_SERVER_TCP_PORT: '30081'
  KUBOARD_SERVER_LOGRUS_LEVEL: info  # error / debug / trace
  # KUBOARD_AGENT_KEY 是 Agent 与 Kuboard 通信时的密钥，请修改为一个任意的包含字母、数字的32位字符串，此密钥变更后，需要删除 Kuboard Agent 重新导入。
  KUBOARD_AGENT_KEY: 32b7d6572c6255211b4eec9009e4a816
  KUBOARD_AGENT_IMAG: 223.2.47.202:8000/kuboard/kuboard-agent:v3
  KUBOARD_QUESTDB_IMAGE: 223.2.47.202:8000/kuboard/questdb:6.0.5
  KUBOARD_DISABLE_AUDIT: 'false' # 如果要禁用 Kuboard 审计功能，将此参数的值设置为 'true'，必须带引号。
  # 这里直接手动配置kuboard所依赖etcd的地址
  KUBOARD_ETCD_ENDPOINTS: "http://223.2.43.228:2381"

  # 关于如下参数的解释，请参考文档 https://kuboard.cn/install/v3/install-gitlab.html
  # [gitlab login]
  # KUBOARD_LOGIN_TYPE: "gitlab"
  # KUBOARD_ROOT_USER: "your-user-name-in-gitlab"
  # GITLAB_BASE_URL: "http://gitlab.mycompany.com"
  # GITLAB_APPLICATION_ID: "7c10882aa46810a0402d17c66103894ac5e43d6130b81c17f7f2d8ae182040b5"
  # GITLAB_CLIENT_SECRET: "77c149bd3a4b6870bffa1a1afaf37cba28a1817f4cf518699065f5a8fe958889"
  
  # 关于如下参数的解释，请参考文档 https://kuboard.cn/install/v3/install-github.html
  # [github login]
  # KUBOARD_LOGIN_TYPE: "github"
  # KUBOARD_ROOT_USER: "your-user-name-in-github"
  # GITHUB_CLIENT_ID: "17577d45e4de7dad88e0"
  # GITHUB_CLIENT_SECRET: "ff738553a8c7e9ad39569c8d02c1d85ec19115a7"

  # 关于如下参数的解释，请参考文档 https://kuboard.cn/install/v3/install-ldap.html
  # [ldap login]
  # KUBOARD_LOGIN_TYPE: "ldap"
  # KUBOARD_ROOT_USER: "your-user-name-in-ldap"
  # LDAP_HOST: "ldap-ip-address:389"
  # LDAP_BIND_DN: "cn=admin,dc=example,dc=org"
  # LDAP_BIND_PASSWORD: "admin"
  # LDAP_BASE_DN: "dc=example,dc=org"
  # LDAP_FILTER: "(objectClass=posixAccount)"
  # LDAP_ID_ATTRIBUTE: "uid"
  # LDAP_USER_NAME_ATTRIBUTE: "uid"
  # LDAP_EMAIL_ATTRIBUTE: "mail"
  # LDAP_DISPLAY_NAME_ATTRIBUTE: "cn"
  # LDAP_GROUP_SEARCH_BASE_DN: "dc=example,dc=org"
  # LDAP_GROUP_SEARCH_FILTER: "(objectClass=posixGroup)"
  # LDAP_USER_MACHER_USER_ATTRIBUTE: "gidNumber"
  # LDAP_USER_MACHER_GROUP_ATTRIBUTE: "gidNumber"
  # LDAP_GROUP_NAME_ATTRIBUTE: "cn"

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kuboard-boostrap
  namespace: kuboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kuboard-boostrap-crb
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kuboard-boostrap
  namespace: kuboard

---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    k8s.kuboard.cn/name: kuboard-etcd
  name: kuboard-etcd
  namespace: kuboard
spec:
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s.kuboard.cn/name: kuboard-etcd
  template:
    metadata:
      labels:
        k8s.kuboard.cn/name: kuboard-etcd
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: node-role.kubernetes.io/master
                    operator: Exists
              - matchExpressions:
                  - key: node-role.kubernetes.io/control-plane
                    operator: Exists
              - matchExpressions:
                  - key: k8s.kuboard.cn/role
                    operator: In
                    values:
                      - etcd
      containers:
        - env:
            - name: HOSTNAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
            - name: HOSTIP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
          # 本地镜像仓库镜像地址
          image: '223.2.47.202:8000/kuboard/etcd-host:3.4.16-1'
          imagePullPolicy: Always
          name: etcd
          ports:
            - containerPort: 2381
              hostPort: 2381
              name: server
              protocol: TCP
            - containerPort: 2382
              hostPort: 2382
              name: peer
              protocol: TCP
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /health
              port: 2381
              scheme: HTTP
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          volumeMounts:
            - mountPath: /data
              name: data
      dnsPolicy: ClusterFirst
      hostNetwork: true
      restartPolicy: Always
      serviceAccount: kuboard-boostrap
      serviceAccountName: kuboard-boostrap
      tolerations:
        - key: node-role.kubernetes.io/master
          operator: Exists
        - key: node-role.kubernetes.io/control-plane
          operator: Exists
      volumes:
        - hostPath:
            path: /usr/share/kuboard/etcd
          name: data
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate


---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations: {}
  labels:
    k8s.kuboard.cn/name: kuboard-v3
  name: kuboard-v3
  namespace: kuboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s.kuboard.cn/name: kuboard-v3
  template:
    metadata:
      labels:
        k8s.kuboard.cn/name: kuboard-v3
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - preference:
                matchExpressions:
                  - key: node-role.kubernetes.io/master
                    operator: Exists
              weight: 100
            - preference:
                matchExpressions:
                  - key: node-role.kubernetes.io/control-plane
                    operator: Exists
              weight: 100
      containers:
        - env:
            # 直接硬编码指向etcd
            - name: KUBOARD_ETCD_ENDPOINTS
              value: "http://223.2.43.228:2381"
            - name: HOSTIP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.hostIP
            - name: HOSTNAME
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: spec.nodeName
          #官网教程使用，这里由于会覆盖etcd的ip配置而弃用，使用硬编码方式
          #envFrom:
            #- configMapRef:
                #name: kuboard-v3-config
          # 本地镜像仓库镜像地址
          image: '223.2.47.202:8000/eipwork/kuboard:v3'
          imagePullPolicy: Always
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /kuboard-resources/version.json
              port: 80
              scheme: HTTP
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          name: kuboard
          ports:
            - containerPort: 80
              name: web
              protocol: TCP
            - containerPort: 443
              name: https
              protocol: TCP
            - containerPort: 10081
              name: peer
              protocol: TCP
            - containerPort: 10081
              name: peer-u
              protocol: UDP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /kuboard-resources/version.json
              port: 80
              scheme: HTTP
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          resources: {}
          # startupProbe:
          #   failureThreshold: 20
          #   httpGet:
          #     path: /kuboard-resources/version.json
          #     port: 80
          #     scheme: HTTP
          #   initialDelaySeconds: 5
          #   periodSeconds: 10
          #   successThreshold: 1
          #   timeoutSeconds: 1
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      serviceAccount: kuboard-boostrap
      serviceAccountName: kuboard-boostrap
      tolerations:
        - key: node-role.kubernetes.io/master
          operator: Exists

---
apiVersion: v1
kind: Service
metadata:
  annotations: {}
  labels:
    k8s.kuboard.cn/name: kuboard-v3
  name: kuboard-v3
  namespace: kuboard
spec:
  ports:
    - name: web
      nodePort: 30080
      port: 80
      protocol: TCP
      targetPort: 80
    - name: tcp
      nodePort: 30081
      port: 10081
      protocol: TCP
      targetPort: 10081
    - name: udp
      nodePort: 30081
      port: 10081
      protocol: UDP
      targetPort: 10081
  selector:
    k8s.kuboard.cn/name: kuboard-v3
  sessionAffinity: None
  type: NodePort

```

### step2. nfs使用

##### 对于NFS服务器

```bash
安装NFS:
yum -y install nfs-utils rpcbind
```

- 设置持久卷权限

```bash
chmod -R 777 /mnt/storage/k8s/pv
```

- 配置 nfs

```bash
systemctl start rpcbind
systemctl enable rpcbind
systemctl start nfs-server
systemctl enable nfs-server
systemctl start firewalld
firewall-cmd --permanent --add-service=nfs
firewall-cmd  --reloadsystemctl stop firewalld && sudo systemctl disable firewalld

配置共享路径
echo "/mnt/storage/k8s/pv *(rw,sync,no_subtree_check,no_root_squash)" > /etc/exports
刷新导出检测
exportfs -rv
showmount -e 223.2.38.237

添加nfs服务规则
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload

验证防火墙规则
firewall-cmd --list-all
```

##### 对于NFS客户端

安装nfs客户端（每台节点）

```
yum install -y nfs-utils
```

手动安装 nfs-client-provisioner（[Kubernetes 中部署 NFS-Subdir-External-Provisioner 为 NFS 提供动态分配卷 | 小豆丁技术栈](http://www.mydlq.club/article/109/#四部署-nfs-subdir-external-provisioner)）

```
配置yaml：

---
---
# 1. 创建 ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  namespace: kube-system

---
# 2. 创建 ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
  - apiGroups: [""]
    resources: ["endpoints"]  # 新增这一部分
    verbs: ["get", "list", "watch", "create", "update", "patch"]  # 领导选举所需权限
---
# 3. 创建 ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io

---
# 4. 创建 Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-client-provisioner
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner  # 引用 ServiceAccount
      containers:
        - name: nfs-client-provisioner
          image: registry.cn-hangzhou.aliyuncs.com/xzjs/nfs-subdir-external-provisioner:v4.0.0
          env:
            - name: PROVISIONER_NAME
              value: nfs-client  # StorageClass 中使用的 Provisioner 名称
            - name: NFS_SERVER
              value: "223.2.38.237"  # 你的 NFS 服务器 IP
            - name: NFS_PATH
              value: "/mnt/storage/k8s/pv"  # 你的 NFS 共享路径
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
      volumes:
        - name: nfs-client-root
          nfs:
            server: 223.2.38.237
            path: /mnt/storage/k8s/pv

---
# 5. 创建 StorageClass（可选）
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: nfs-client  # 与 PROVISIONER_NAME 一致
parameters:
  archiveOnDelete: "false"  # 删除 PVC 时是否归档
```

执行

```
kubectl apply -f nfs-provisioner-deploy.yaml -n kube-system
```

