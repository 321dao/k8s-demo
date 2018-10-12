
### 1 Kubernetes基础概念

官网：https://kubernetes.io

Kubernetes是一个用于容器集群的自动化部署、扩容以及运维的开源平台，它是master-nodes架构，master只需几个来做高可用，任何时候只有一个master在管理集群，nodes可以多个，是真正部署容器的节点。

Kubernetes的特点：

- 自动装箱。基于资源和依赖自动部署服务。
- 自我修复。当有一个容器挂了，能够自动启动一个新的同样服务替换故障的容器。
- 自动实现水平扩展。只要物理资源充足，设置触发阈值，可以实现自动实现水平伸缩服务。
- 自动服务发现和负载均衡。service能够提供稳定的访问端点和负载均衡。
- 自动发布和回滚。
- 秘钥和配置管理。
- 存储编排，存储卷动态供给。
- 批量处理执行。

<br>

Kubernetes的架构图：

![k8s架构图](https://pic.zhuyasen.com/815193_01_k8s%E6%9E%B6%E6%9E%84%E5%9B%BE.jpg)

<br>

Kubernetes主要模块概念：

| 名称 | 说明 |
| :------ | :------ |
| Cluster | Cluster是指由Kubernetes使用一系列的物理机、虚拟机和其他基础资源来运行你的应用程序。 |
| Node | 一个node就是一个运行着Kubernetes的物理机或虚拟机，并且pod可以在其上面被调度。 |
| Pod | 一个pod对应一个由相关容器和卷组成的容器组。 |
| Label | 一个label是一个被附加到资源上的键/值对，例如附加到一个Pod上，为它传递一个用户自定的并且可识别的属性，Label还可以被应用来组织和选择子网中的资源。 |
| selector | selector是一个通过匹配labels来定义资源之间关系得表达式，例如为一个负载均衡的service指定所目标Pod。 |
| Replication Controller | replication controller 是为了保证一定数量被指定的Pod的复制品在任何时间都能正常工作，它不仅允许复制的系统易于扩展，还会处理当pod在机器在重启或发生故障的时候再次创建一个。 |
| Service | 一个service定义了访问pod的方式，就像单个固定的IP地址和与其相对应的DNS名之间的关系。 |
| Volume | 一个volume是一个目录，可能会被容器作为未见系统的一部分来访问。 |
| Kubernetes volume | 构建在Docker Volumes之上,并且支持添加和配置volume目录或者其他存储设备。 |
| Secret | Secret 存储了敏感数据，例如能允许容器接收请求的权限令牌。 |
| Name | 用户为Kubernetes中资源定义的名字。 |
| Namespace | Namespace 好比一个资源名字的前缀。它帮助不同的项目、团队或是客户可以共享cluster，例如防止相互独立的团队间出现命名冲突。 |
| Annotation | 相对于label来说可以容纳更大的键值对，它对我们来说可能是不可读的数据，只是为了存储不可识别的辅助数据，尤其是一些被工具或系统扩展用来操作的数据。 |

<br>

Kubernetes设计图：

![k8s设计图](https://pic.zhuyasen.com/815193_04_k8s%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

<br>

#### 1.1 master节点介绍

![master架构图](https://pic.zhuyasen.com/815193_02_k8s%E7%9A%84master%E6%9E%B6%E6%9E%84%E5%9B%BE.jpg)

<br>

| 名称 | 说明 |
| :------ | :------ |
| Kubernetes API Server | API服务提供Kubernetes API服务。客户端通过这个服务对资源的增删改查等操作，它主要提供标准的REST接口调用，而且也是kubernetes集群其他模块(etcd、scheduler等)的客户端。 |
| Scheduler | 调度器把未调度的pod通过binding api绑定到节点上。调度器是可插拔的，并且我们期待支持多集群的调度，未来甚至希望可以支持用户自定义的调度器。 |
| Kubernetes控制面板 | Kubernetes控制面板可以分为多个部分。目前它们都运行在一个master节点，然而为了达到高可用性，这需要改变。不同部分一起协作提供一个统一的关于集群的视图。 |
| etcd | 所有master的持续状态都存在etcd的一个实例中。这可以很好地存储配置数据。因为有watch(观察者)的支持，各部件协调中的改变可以很快被察觉。 |

<br>

#### 1.2 node节点介绍

![node架构图](https://pic.zhuyasen.com/815193_03_k8s%E7%9A%84node%E6%9E%B6%E6%9E%84%E5%9B%BE.jpg)

<br>

| 名称 | 说明 |
| :------ | :------ |
| Pod | Pod是k8s管理的最小单位，k8s不会对容器直接操作，Pod和容器是什么关系呢？Pod可以包括一个或多个容器，每个Pod都有自己的namespace(网络、存储等资源)，在一个Pod内部的多个容器之间关系类似系统间的进程之间关系，使用网络直接用localhost就可以通信。 |
| kubelet | kubelet负责管理pods和它们上面的容器，images镜像、volumes等。 |
| kube-proxy | 每一个节点也运行一个简单的网络代理和负载均衡，例如创建service资源需要kube-proxy。 |
| fluentd | 日志收集工具 |

<br>

Pod分类：

- 自主式Pod
- 由控制器管理的Pod，管理不同类型的Pod资源
  - ReplicaSet(替代ReplicationController)
  - Deployment(只能管理无状态的应用)
  - StatefulSet(管理有状态的应用)
  - DaemonSet(在每一个node上运行一个副本)
  - Job(运行结束自动删除)
  - Ctonjob(周期性job)

其中DaemonSet、Job、Ctonjob是给特殊应用使用，另外Deployment还支持二级控制器HPA(HorizontalPodAutoscaler)，HPA控制器自动监控资源，当前服务能力不能满足时，自动扩展Pod，当Pod资源空闲时，自动回收资源，这些阈值由用户设置。

<br><br>

### 2 安装kubernetes集群

搭建Kubernetes集群的一些工具：

| 名称 | 说明 |
| :------ | :------ |
| [minikube](https://github.com/kubernetes/minikube) | 搭建单节点集群 |
| [kubeadm](https://github.com/kubernetes/kubeadm) | 搭建多节点集群  |
| [kops](https://github.com/kubernetes/kops) | 在云上安装多节点集群 |
| [kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way) | 搭建多节点集群，这是最困难的搭建k8s集群方式，没有借助任何工具，使用命令行来一步一步搭建。 |

这里使用kubeadm来搭建集群，搭建集群概要：

- 在master和node节点上安装kubeadm、kubelet、docker
- 在master节点安装kubectl，初始化集群kubeadm init
- 在node节点中加入k8s集群

![集群安装框架图](https://pic.zhuyasen.com/815193_05_%E9%83%A8%E7%BD%B2%E9%9B%86%E7%BE%A4%E6%A1%86%E6%9E%B6.png)

<br>

#### 2.1 准备虚拟机

搭建的kubernetes集群共3个节点，一个master和两个node：

- master：192.168.8.90
- node01：192.168.8.91
- node02：192.168.8.92。

<br>

首先在VMware上安装一个centos7系统(master节点)，在master虚拟机关闭情况下，复制master虚拟机文件夹作为node01节点，再复制一份master虚拟机文件夹作为node02节点，复制的两个虚拟机的硬件信息是一样的(包括网络)，所以需要修改node01和node02节点的网络。打开VMware软件，导入虚拟机文件，在启动node01虚拟机前，点击编辑虚拟街配置-->网络适配器-->高级-->生成，使用新的MAC地址，不能和master虚拟机MAC地址一样。然后启动虚拟，启动时弹出当前是克隆还会复制虚拟机，选择复制，修改node02虚拟机网络的操作也一样。

进入虚拟机修改网络信息：

```bash
# 查看网络ens33物理地址
ip addr
    # 例如得到物理地址为00:50:56:2c:68:37

# 在网卡配置，文件名可能不一样，目录是一样的
vim /etc/udev/rules.d/90-eno-fix.rules
    # 把ATTR{address}的值改为00:50:56:2c:68:37

# 生成UUID
uuidgen ens33
    # 例如生成的UUID值f92eec86-8cd5-4ba3-8b3d-ee7a415252c2

# 把UUID填写到ens33网络接口配置中，并修改静态ip地址
vim /etc/sysconfig/network-scripts/ifcfg-eno16777736

# 完整网络配置内容如下：
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=eno16777736
UUID=f92eec86-8cd5-4ba3-8b3d-ee7a415252c2
DEVICE=eno16777736
ONBOOT=yes
IPADDR=192.168.8.91
NETMASK=255.255.255.0
GATEWAY=192.168.8.1

DNS1=202.96.128.86
DNS2=114.114.114.114

# 重启系统
reboot
```

然后在master和node节点上修改主机名称和添加主机名记录值：

```bash
# master虚拟机
echo 'master' > /etc/hostname
echo "192.168.8.90 master" >> /etc/hosts

# node01虚拟机
echo 'node01' > /etc/hostname
echo "192.168.8.91 node01" >> /etc/hosts

# node02虚拟机
echo 'node02' > /etc/hostname
echo "192.168.8.92 node02" >> /etc/hosts
```

<br>

#### 2.2 在master和node节点都安装docker、kubeadm、kubelet

**(1) 在master和node节点上都关闭firewalld、打开桥接功能、关闭swap**

```bash
# 关闭防火墙
systemctl disable firewalld && systemctl stop firewalld

# 自动开启桥接功能
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables

# 关闭swap
echo "vm.swappiness = 0" >> /etc/sysctl.conf
swapoff -a && swapon -a
```

<br>

**(2) 在master和node节点上都安装docker**

```bash
# 安装一些工具
yum install -y git vim gcc glibc-static telnet bridge-utils net-tools

# 安装docker
yum install -y docker

# 把用户添加docker用户组
groupadd docker
gpasswd -a vison docker

# 设置开机启动docker和启动docker
systemctl enable docker
systemctl start docker
```

<br>

**(3) 在master和node节点上都安装kubernetes**

由于安装kubernetes需要翻墙，为了省去翻墙，这里使用[阿里云镜像地址](https://opsx.alibaba.com/mirror)，打开阿里云镜像网站，找到kubernetes仓库，然后在各节点上yum安装源目录上添加kubernetes仓库地址。

```bash
# 新建kubernetes仓库文件
vim /etc/yum.repos.d/kubernetes.repo

# 添加内容：
[kubernetes]
name=Kubernetes Repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
enabled=1

# 检查kubernetes.repo配置有没有错误
yum repolist

# 下载依赖包
wget https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

# 导入依赖包
rpm --import rpm-package-key.gpg

# 在master节点安装kubeadm、kubelet、kubectl
yum install -y kubeadm kubelet kubectl
systemctl enable kubelet
systemctl start kubelet # 虽然启动失败，但是也得启动，否则集群初始化会卡死

# 在node节点安装kubeadm、kubelet
yum install -y kubeadm kubelet
systemctl enable kubelet
systemctl start kubelet
```

<br>

#### 2.3 在master节点上初始化集群

在master节点初始化集群时去官方下载(需翻墙)相关镜像，下载官方镜像有两种方式：

(1) 不翻墙情况下导入镜像

已经准备好1.11.1版本所有相关镜像压缩包。

```bash
# 下载镜像
链接：https://pan.baidu.com/s/17ObSpbcgjZipIClQ-ynp2w
提取码：sa9a

# 在master和node节点都导入镜像
tar xf k8s-v1.11.1.images.tar.gz
docker load -i k8s-v1.11.1.images
```

<br>

(2) 配置docker翻墙代理

```bash
# 打开docker系统配置
vim /usr/lib/systemd/system/docker.service

# 添加环境变量
Environment="HTTPS_PROXY=http://127.0.0.1:1080"
Environment="NO_PROXY=127.0.0.0/8"

# 重新加载docker配置和重启docker
systemctl daemon-reload
systemctl restart docker

# 查看是否有代理配置
docker info

# 安装完成需要回去注释掉添加的环境变量
```

<br>

初始化kubernetes集群：

```bash
# 忽略swap错误
echo 'KUBELET_EXTRA_ARGS="--fail-swap-on=false"' > /etc/sysconfig/kubelet

# 初始化
kubeadm init --kubernetes-version=v1.11.1 --pod-network-cidr=10.244.0.0/16  --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap

# 当看到提示Your Kubernetes master has initialized successfully!说明集群初始完成，并记录下加入集群join命令(重要)
kubeadm join 192.168.8.90:6443 --token fs5eos.m8p4pab3fxlefap5 --discovery-token-ca-cert-hash sha256:c353514a313f41ae9124e6cf5a45cfbd67e12e7762b28bbfe44538f07efba363

# 给普通用户vison操作集群权限
mkdir -p /home/vison/.kube
cp -i /etc/kubernetes/admin.conf /home/vison/.kube/config
chown -R vison.vison /home/vison/.kube

# 查看节点
kubectl get nodes
  # NAME      STATUS     ROLES     AGE       VERSION
  # master    NotReady   master    38m       v1.11.3
  # 发现主节点在notready状态，因为还没安装网络插件，列如：funnel

# 安装网络插件funnel，等待一会自动拉取镜像并启动
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 查看集群系统pods是否都在Running状态，如果都在Running状态，说明master节点启动完成。
kubectl get pods -n kube-system
  # NAME                             READY     STATUS    RESTARTS   AGE
  # coredns-78fcdf6894-24v8q         1/1       Running   0          48m
  # coredns-78fcdf6894-kkk9t         1/1       Running   0          48m
  # etcd-master                      1/1       Running   0          4m
  # kube-apiserver-master            1/1       Running   0          4m
  # kube-controller-manager-master   1/1       Running   0          4m
  # kube-flannel-ds-amd64-g4q5v      1/1       Running   0          4m
  # kube-proxy-hqsq5                 1/1       Running   0          48m
  # kube-scheduler-master            1/1       Running   0          4m

# 查看版本
kubectl version

# 查看集群信息
kubectl cluster-info
```

<br>

#### 2.4 在node节点上加入集群

在node节点上加入集群很简单，直接执行master节点初始化时生成join命令即可。

```bash
# 忽略swap错误
echo 'KUBELET_EXTRA_ARGS="--fail-swap-on=false"' > /etc/sysconfig/kubelet

# 加入集群，需要等待一会拉取镜像并启动
kubeadm join 192.168.8.90:6443 --token fs5eos.m8p4pab3fxlefap5 --discovery-token-ca-cert-hash sha256:c353514a313f41ae9124e6cf5a45cfbd67e12e7762b28bbfe44538f07efba363 --ignore-preflight-errors=Swap

# 查看节点
kubectl get nodes
  # NAME      STATUS    ROLES     AGE       VERSION
  # master    Ready     master    13h       v1.11.3
  # node01    Ready     <none>    12h       v1.11.3
  # node02    Ready     <none>    12h       v1.11.3

# 查看启动的镜像
docker ps
  # CONTAINER ID        IMAGE                  COMMAND                  NAMES
  # d621966213ca        f0fad859c909           "/opt/bin/flanneld..."   k8s_kube-flannel_kube-flannel-ds-amd64...
  # 589dc087efc9        d5c25579d0ff           "/usr/local/bin/ku..."   k8s_kube-proxy_kube-proxy-6l46n...
  # 821e5a9a871f        k8s.gcr.io/pause:3.1   "/pause"                 k8s_POD_kube-flannel-ds-amd64-b27mp...
  # e5d792594a3d        k8s.gcr.io/pause:3.1   "/pause"                 k8s_POD_kube-proxy-6l46n...
```

<br>

#### 2.5 搭建集群遇到的问题

搭建集群时首先保证正常kubelet运行和开机启动，只有kubelet运行才有后面的初始化集群和加入集群操作。

(1) 启动kubelet失败

```bash
# 查看启动状态
systemctl status kubelet
  # 提示信息kubelet.service failed.

# 查看报错日志
tail /var/log/messages
  # failed to run Kubelet: Running with swap on is not supported, please disable swap!

# 解决办法，忽略swap错误
echo 'KUBELET_EXTRA_ARGS="--fail-swap-on=false"' > /etc/sysconfig/kubelet
```

<br><br>

### 3 kubectl基本操作

kubectl 主要是对pod、service、replicaset、deployment、statefulset、daemonset、job、cronjob、node资源的增删改查。

#### 3.1 安装kubectl命令自动补全

```bash
# 安装kubectl自动补全命令包
yum install -y  bash-completion

# 添加的当前shell
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```

<br>

#### 3.2 kubectl run命令

kubectl run启动的是deployment、job等类型资源。

```bash
# 启动一个测试pod，退出pod自动删除
kubectl run ${RANDOM} --rm -it --image=cirros:0.4 -- /bin/sh
  # CirrOS是一个很小的测试镜像，它经常用于测试OpenStack部署

# 在集群启动一个deployment类型的nginx服务
kubectl run nginx-deploy --image=nginx

# 查看deployment。
kubectl get deployments
  # NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  # nginx-deploy   1         1         1            1           1m

# 查看pods，并显示在哪个node节点
kubectl get pods -o wide
  # NAME                            READY     STATUS    RESTARTS   AGE       IP           NODE
  # nginx-deploy-59c86578c8-vqdqc   1/1       Running   0          1m        10.244.1.2   node01

# 在集群内部任意一个节点都可访问nginx，暂时外部不可访问
curl 10.244.1.2

# 这个nginx服务ip地址是不定的，假如删除了pod，重建了新的pod的Ip地址就变化了，为了使nginx服务ip地址固定，需要创建service资源，分配给deployment类型的nginx一个固定的虚拟ip地址
kubectl expose deployment nginx-deploy --name=nginx-service --port=80 --target-port=80

# 查看service信息，可以查看到固定的ip，删除pod或新建的pod的Ip地址虽然会变化，但service的ip不变，形成固定的访问端点
kubectl get service -o wide
  # NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE       SELECTOR
  # kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP   14h       <none>
  # nginx-service   ClusterIP   10.102.228.39   <none>        80/TCP    2m        run=nginx-deploy

# 在集群内任意一个节点访问nginx服务，此时集群外仍然不可访问
curl 10.102.228.39

# 在客户端上通过service名称直接访问，不需要知道service的ip地址。启动一个测试pod：
kubectl run ${RANDOM} --rm -it --image=cirros:0.4 -- /bin/sh

# 在容器里可以直接使用名称直接访问，即便删除service再重建，service的ip改变了，也不影响域名解析。
curl nginx-service

# 集群是怎样能通过名称直接访问的呢？
# 查看集群系统级别的service，有个域名解析服务，ip地址为10.96.0.10
kubectl get service -n kube-system
  # NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
  # kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   14h

# 在容器里查看域名解析
cat /etc/resolv.conf
  # nameserver 10.96.0.10
  # search default.svc.cluster.local svc.cluster.local cluster.local
  # options ndots:5
  # 有了10.96.0.10和search后缀内容，在访问的域名不完整的时候，会把不完整的域名和后缀(/etc/resolv.conf文件search内容)拼接成为完整域名，通过完整的域名，可以解析service名称对应的ip地址了。

# 解析完整的域名
nslookup nginx-service.default.svc.cluster.local


# 通过service详情可以看出关联哪些pods
kubectl describe service nginx-service
  # Name:              nginx-service
  # Namespace:         default
  # Labels:            run=nginx-deploy
  # Annotations:       <none>
  # Selector:          run=nginx-deploy
  # Type:              ClusterIP
  # IP:                10.102.228.39
  # Port:              <unset>  80/TCP
  # TargetPort:        80/TCP
  # Endpoints:         10.244.1.3:80
  # Session Affinity:  None
  # Events:            <none>
  # 其中Labels和Selector标签选择器字段，在pods也有，就是通过selector关联和管理

# 查看pod标签
kubectl get pods --show-labels
  # NAME                            READY     STATUS    RESTARTS   AGE       LABELS
  # nginx-deploy-59c86578c8-2h8w7   1/1       Running   0          12m       pod-template-hash=1574213474,run=nginx-deploy

# 删除原来service
kubectl delete service nginx-service

# 创建集群外可以访问nginx服务的service
kubectl expose deployment nginx-deploy --name=nginx-service --port=80 --target-port=80

# 查看端口映射
kubectl get service

# 在集群外访问nginx服务
curl masster:30193
curl node01:30193
curl node02:30193
```

<br>

#### 3.3 动态缩放pod的数量和动态更新版本

```bash
# 启动一个测试pod
kubectl run myapp --image=ikubernetes/myapp:v1 --replicas=3

# 监控查看deployment类型的myapp是否创建完成
kubectl get deployments -w

# 查看pod
kubectl get pods -o wide
  # NAME                            READY     STATUS    RESTARTS   AGE       IP           NODE
  # myapp-848b5b879b-5v5fz          1/1       Running   0          1m        10.244.2.4   node02
  # myapp-848b5b879b-8prhr          1/1       Running   0          1m        10.244.1.4   node01
  # myapp-848b5b879b-vhkcd          1/1       Running   0          1m        10.244.2.3   node02

# 创建service
kubectl expose deployment myapp --name=myapp --port=80 --target-port=80

# 查看service
kubectl get service | grep myapp

# 启动一个交互式busybody来测试
kubectl run bb01 --image=busybox -it

# 在交互式终端是输入下面命令，获取pod名称
while true; do wget -O - -q http://myapp/hostname.html; sleep 1; done;
  # 从结果上看出，获取pod的名称是随机的，说明集群本身带有负载均衡。

# 动态扩展或缩小pod的数量
kubectl scale deployment myapp --replicas=5


# 动态更新版本
  # 在busybody容器执行命令打印当前版本信息
  while true; do wget -O - -q http://myapp; sleep 1; done;

  # 更新镜像
  kubectl set image deployment myapp myapp=ikubernetes/myapp:v2

  # 查看更新过程
  kubectl rollout status deployment myapp

  # 查看pod，发现myapp的pod名称已经全部修改了
  kubectl get pods

  # 在busybody容器可以动态看到版本更新，也可以从deployment中查看当前镜像版本
  kubectl get deployments -o wide


# 回滚旧版本
  # 如果发现新版本有问题，可以回滚到上一个版本
  kubectl rollout undo deployment myapp

  # 查看回滚过程
  kubectl rollout status deployment myapp

  # 在busybody容器打印内容中可以动态看到版本回滚
```

<br><br>

### 4 kubernetes集群资源

#### 4.1 资源分类

kubernetes的所有资源都可以抽象为对象，分为几大类：

- 工作负载(workload)：Pod、ReplicaSet、Deployment、StatefuleSet、DaemonSet、Job、Cronjob
- 服务发现和负载均衡：Service、Ingress
- 配置与存储：Volume、CSI、ConfigMap、Secret、DownwardAPI
- 集群级资源：Namespace、Node、Role、ClusterRole、RoleBinding、ClusterRoleBinding
- 元数据资源：HPA、PodTemplate、LimitRange

<br>

```bash
# 通过固定格式路径获取所有资源信息
/api/<GROUP>/<VERSION>/namespace/<NAMESPACE>/<TYPE>/<NAME>

# 使用命令查看路径
kubectl get deployment nginx-deploy -o yaml | grep selfLink
```

<br>

#### 4.2 资源配置文件说明

查看资源的配置清单：

```bash
# yaml格式
kubectl get pod nginx-deploy-59c86578c8-2h8w7 -o yaml

# json格式
kubectl get pod nginx-deploy-59c86578c8-2h8w7 -o json
```

<br>

其实kubernetes集群的apiServer只能解析json格式定义的资源，master节点的apiserver会对非json格式定义的资源或命令统一转换为json格式，例如yaml格式提供的配置清单，apiserver会自动将其转换为json格式。

大部分资源配置清单由五大部分组成(一级字段)：

| 名称 | 说明 |
| :------ | :------ |
| apiVersion | 指明创建的资源属于哪个api群组版本，使用命令kubectl api-versions查看当前有哪些群组版本。 |
| kind | 资源类别，例如Deployment、Service等。 |
| metadata | 元数据，字段name，同一类资源中，name字段保证唯一；字段labels，可以有多个标签。 |
| spec | 资源规格，是用户期望的状态。 |
| status | 当前资源状态，由kubernetes集群维护，对于用户为只读，不可修改 |

<br>

资源类型这么多，怎么知道那种资源有哪些字段？这些字段意义什么的？kubernetes提供了完整资源定义文档，例如要定义pod类型资源：

```bash
# 查看pod资源使用说明
kubectl explain pod

# 可以得知pod资源的属于哪个版本类型VERSION，并显示的是字段的说明，其中需要注意下字段的类型和是否标记为required字段，如果是required字段，是用户定义资源时必填字段。

# 查看下二级字段说明，例如查看spec使用说明
kubectl explain pod.spec

# 查看三级字段说明，例如查看spec下container使用说明
kubectl explain pod.spec.containers

# 如此类推，凡是字段类型为object，都可以通过逐级方式获取使用说明。
```

<br>

#### 4.3 生成资源配置模版文件

用户提交命令给apiServer去创建资源，apiserver会把命令行转为json格式资源清单，用户可以拿这个资源清单来做模板，可以生成清单模板的资源如下：

| 名称 | 说明 |
| :------ | :------ |
| deployment | 使用指定的名称创建部署 |
| service | 使用指定的子命令创建服务 |
| configmap | 从本地文件，目录或文字值创建配置映射 |
| secret | 使用指定的子命令创建密钥 |
| clusterrole | 创建ClusterRole |
| clusterrolebinding | 为特定ClusterRole创建ClusterRoleBinding |
| job | 创建具有指定名称的作业 |
| namespace | 创建具有指定名称的命名空间 |
| poddisruptionbudget | 使用指定的名称创建pod中断预算 |
| priorityclass | 使用指定的名称创建priorityclass |
| quota | 创建具有指定名称的配额 |
| role | 使用单个规则创建角色 |
| rolebinding | 为特定Role或ClusterRole创建RoleBinding |
| serviceaccount | 创建具有指定名称的服务帐户 |

<br>

创建资源模板的一些示例：

```bash
# 创建deployment资源模板
kubectl create deployment bb-dm --image=busyboxy --dry-run -o yaml > bb-dm.yml

# 创建sevice资源模板
kubectl create service clusterip bb-svc-cip --tcp=80:80 --dry-run -o yaml > bb-svc-cip.yml

# 创建serviceaccount资源模板
kubectl create serviceaccount vison --dry-run -o yaml > vison-sa.yml
```

<br><br>

### 5 pod资源

pod的生命周期：

- pod启动过程：用户提交的pod资源清单配置提交给apiserver，apiserver把资源清单存储在etcd当中，apiserver拿着资源配置清单给scheduler，scheduler根据资源配置清单分配资源，然后把调度结果(可以在哪个节点运行)保存到etcd中pod资源清单下status中。apiserver根据scheduler调度结果把把资源配置清单发送给对应节点的kubelet，kubelet根据资源配置清单启动pod，并把最终启动状态保存到etcd中pod资源清单下status，如果创建pod当前状态和目标状态不一致，kubelet会尽可能使其一致。
- pod状态：Pending、Running、successed、failed、Unknown。
- pod生命周期中的重要行为：初始化容器、liveness容器探测、readiness容器探测。
- pod的容器重启策略：用户可以资源配置清单spec下设置restartPolicy，共有三种策略：总是重启(默认)、不重启、错误下重启(正常结束不重启)。总是重启并不是真正意义马上重启，第一次立刻重启，第二次延时10秒，第三次延时20秒，第四次延时40秒，最大延时300秒。
- pod终止：当用户删除pod时，pod会发送终止信号给容器，让容器平滑关闭，避免数据丢失，如果pod等待30秒后，发现容器还没终止，会发送kill信号给容器，强行杀掉容器。

一般实际使用中很少使用自主创建pod的资源，都是pod控制器创建，例如Deployment、StatefulSet、job等。

<br>

#### 5.1 创建一个Pod资源示例

创建pod资源清单配置文件pod-demo.yml，内容如下：

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: nginx-demo
  # 标签，给选择器过滤
  labels:
    app: mynginx
  # 注释，仅提供元数据
  #annotations:
  #  key: value

spec:
  containers:
  - name: nginx
    image: nginx
    # 拉取镜像策略，存在则不重新拉取
    imagePullPolicy: IfNotPresent
    # 这里暴露端口只是信息说明，不能真正意义上的暴露端口
    ports:
    - name: http
      containerPort: 80
    # 这里命令不是在shell执行，如果要在shell上执行，在前面加/bin/sh -c
    #command:["/bin/sh"]
    #agrs:["-c", "while true; do echo hello; sleep 10;done"]

  # 节点选择器
  #nodeSelector:
  #  key: value
  # 选择在哪个节点上运行
  #nodeName: node01
```

<br>

当配置中使用command命令时，注意镜像中的Entrypoint、Cmd和资源配置中容器的command、agrs的关系，它们的关系如下图所示：

![镜像命令和容器启动命令关系](https://pic.zhuyasen.com/815193_06_%E9%95%9C%E5%83%8F%E5%91%BD%E4%BB%A4%E5%92%8C%E5%AE%B9%E5%99%A8%E5%91%BD%E4%BB%A4%E5%85%B3%E7%B3%BB.png)

<br>

kubectl创建资源有几种用法：命令式用法、资源配置清单式用法，这里使用资源配置清单式用法。

```bash
# 创建资源
kubectl create -f pod-demo.yml
kubectl apply -f pod-demo.yml # 如果资源已创建，忽略，如果不存在，创建。

# 查看详情，详情最后Events是启动过程日志
kubectl describe -f pod-demo.yml

# 删除
kubectl delete -f pod-demo.yml
```

<br>

#### 5.2 容器探测

容器探测分为就绪探测和存活性探测，两者都有三种探测类型ExecAction、HTTPGetAtion、TCPSocketAtion，根据需要使用一种。

**(1) ExecAction存活探测**

创建容器存活探测资源配置文件pod-exec-live.yml，内容如下：

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: liveness-exec

spec:
  containers:
  - name: busybox
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "touch /tmp/healthy; sleep 100; rm -f /tmp/healthy; sleep 1000"]
    livenessProbe:
      # 探测方式exec
      exec:
        command: ["test", "-e", "/tmp/healthy"]
      # 等待初始化时间
      initialDelaySeconds: 3
      # 探测间隔时间
      periodSeconds: 10
```

从资源配置清单可以看出3次探测失败(30s)后重启容器。

<br>

**(2) HTTPGetAtion存活探测和就绪性探测**

为什么需要就绪性探测？有些时候虽然容器启动了，可能服务还没初始化化完成，在服务还没初始化完成情况下容器启动就马上对外提供服务，造成客户端访问出错，这时需要做服务就绪性探测，探测到服务完成准备完成就可以对外提供服务了。

创建容器的存活探测和就绪性探测的资源配置文件pod-httpget-live.yml，内容如下：

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: liveness-httpget

spec:
  containers:
  - name: mynginx
    image: nginx
    imagePullPolicy: IfNotPresent
    ports:
    - name: indexfile
      containerPort: 80
    livenessProbe:
      # 探测方式httpget
      httpGet:
        port: indexfile
        path: /index.html
      # 等待初始化时间
      initialDelaySeconds: 3
      # 探测间隔时间
      periodSeconds: 10
    # 就绪探测
    readinessProbe:
      # 探测方式httpget
      httpGet:
        port: indexfile
        path: /50x.html
      # 等待初始化时间
      initialDelaySeconds: 3
      # 探测间隔时间
      periodSeconds: 3
```

<br>

检验存活性探测和就绪性是否有效：

```bash
# 进去容器
kubectl exec -it liveness-httpget -- /bin/bash

# 存活性探测测试
  # 删除index.html
  rm -f /usr/share/nginx/html/index.html
  # 因为删除了index.html，探测肯定失败的，如果连续3次失败后能够重启容器，说明正常。
  # 查看pod的重启次数
  kubectl get pod
  # 查看pod详情信息
  kubectl describe -f pod-httpget-live.yml


# 就绪性探测测试
    # 修改文件名
    mv /usr/share/nginx/html/50x.html /usr/share/nginx/html/50x.html.bk
    # 等待10s查看就绪情况，应该为0
    kubectl get pod
    # 恢复文件
    mv /usr/share/nginx/html/50x.html.bk /usr/share/nginx/html/50x.html
    # 再次查看就绪情况，就绪情况状态值马上恢复为1
    kubectl get pod
```

<br>

#### 5.3 标签选择器

给Pod资源定义标签作用是给Pod控制器分类管理，可以在命令行给资源添加标签，也可以在资源清单配置文件添加标签。

标签操作：

```bash
# 添加标签
kubectl label pods nginx-demo version=release

kubectl label -f pod-demo.yml version=release

# 显示标签
kubectl get pod --show-labels

# 更新标签
kubectl label -f pod-demo.yml version=release --overwrite
```

<br>

标签选择器：

```bash
# 等值标签选择器，只显示满足标签条件的资源
kubectl get pod -l app
kubectl get pod -l app=mynginx
kubectl get pod -l app=mynginx,version=stable
kubectl get pod -l app!=mynginx

# 集合关系选择器
kubectl get pod -l "version in (beta, stable, release)"
kubectl get pod -l "version notin (beta, stable, release)"
```

<br><br>

### 6 Pod控制器

Pod的控制器有：ReplicaSet、Deployment、StatefuleSet、DaemonSet、Job、Cronjob。

#### 6.1 ReplicaSet资源

ReplicaSet资源是在Pod之上新创建的一层管理Pod的资源，支持动态伸缩Pod的数量，所以非常适合部署无状态服务；支持更新容器镜像版本，但是需要手动重启Pod中的容器才会使用最新版本镜像，已运行的pod中容器使用镜像版本不会更改。使用Deployment资源可以解决手动重启pod来更新版本问题。所以实际中使用ReplicaSet资源也不多。

创建ReplicaSet资源配置文件replicaSet-demo.yml，内容如下：

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-rs

spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: nginx-pod
      # 注意：这里标签一定要包括上面自定义的selector标签
      labels:
        tier: frontend
        app: myapp
    # 下面就是pod的spec
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: IfNotPresent
        ports:
        - name: nginx
          containerPort: 80
```

<br>

#### 6.2 Deployment资源

Deployment资源是在ReplicaSet之上新创建的一层用来管理ReplicaSet的资源，除了拥有ReplicaSet特性之外，还有自动更新版本、回滚、自定义更新策略功能。

![层级关系](https://pic.zhuyasen.com/815193_07_deploy%E5%B1%82%E7%BA%A7%E5%85%B3%E7%B3%BB.jpg)

<br>

创建Deployment资源配置文件deployment-demo.yml，内容如下：

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-dm

# 下面是ReplicaSet的spec
spec:
  replicas: 3
  # 更新策略
  strategy:
    type: RollingUpdate
    rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: nginx-pod
      # 注意：这里标签一定要包括上面自定义的selector标签
      labels:
        tier: frontend
        app: myapp
    # 下面就是pod的spec
    spec:
      containers:
      - name: nginx
        image: nginx:1.12.2
        #imagePullPolicy: IfNotPresent
        ports:
        - name: nginx
          containerPort: 80
```

<br>

测试动态扩容、更新镜像版本、回滚：

```bash
# 创建资源，这里使用apply替换create，apply可以执行多次，可以识别文件修改值，create只能执行一次。
kubectl apply -f deployment-demo.yml

# 修改配置方式有三种：
  # (1) 修改资源配置文件示例：kubectl apply -f deployment-demo.yml
  # (2) 打补丁方式示例：kubectl patch -f deployment-demo.yml -p '{"spec":{"replicas":5}}'
  # (3) 在线编辑示例：kubectl edit -f deployment-demo.yml

# 动态扩容，扩容为5个pod运行
kubectl patch -f deployment-demo.yml -p '{"spec":{"replicas":5}}'

# 更新镜像版本
kubectl set image -f deployment-demo.yml nginx=nginx:1.15.2

# 回滚版本(默认是上一个版本)
kubectl rollout undo deployment nginx-dm

# 查看历史版本
kubectl rollout history deployment nginx-dm

# 回滚到指定版本
kubectl rollout undo deployment nginx-dm --to-revision=2
```

<br>

默认更新方式是滚动更新RollingUpdate，滚动更新策略是pod数量最少不能小于25%，最大不能超过25%，不足1个pod当作一个处理，如下图所示：

![滚动更新](https://pic.zhuyasen.com/815193_08_deploy%E7%89%88%E6%9C%AC%E6%9B%B4%E6%96%B0.jpg)

<br>

有时发布的新版本想在线上试运行一段时间，又不想让当前版本的所有pod都更新为最新版本，等新版本试运营一段时间没问题后，再把当前版本更新为最新版本。很明显自动滚动更新不能满足需求，需要用户改变滚动更新策略。

```bash
# 更新镜像时马上暂停，根据自定义更新策略，新添加一个pod，pod启动后暂停更新操作，让新版本运行一段时间。
kubectl set image -f deployment-demo.yml nginx=nginx:1.14.0 && kubectl rollout pause deployment nginx-dm

# 监控状态
kubectl rollout status deployment nginx-dm
kubectl get pod -o wide -w

# 确认新版本服务没问题后，继续更新剩余的pod
kubectl rollout resume deployment nginx-dm
```

<br>

#### 6.3 DaemonSet资源

DaemonSet资源是在Pod之上新创建的一层用来管理Pod的资源，每个node节点都会运行一个守护进程，支持滚动更新，更新策略是先终止再启动新的Pod。也支持有限个节点上运行DaemonSet，DaemonSet的一些典型用法是：

- 在每个节点上运行集群存储守护程序，例如glusterd，ceph等。
- 在每个节点上运行日志收集守护程序，例如fluentd、logstash等。
- 在每个节点上运行节点监视守护程序，例如Prometheus Node Exporter，collectd等。

<br>

创建DaemonSet资源配置文件daemonSet-demo.yml，内容如下：

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: nginx-ds

spec:
  selector:
    matchLabels:
      app: mynginx
  template:
    metadata:
      labels:
        app: mynginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.2
        ports:
        - name: nginx-port
          containerPort: 80
```

<br>

#### 6.4 StatefulSet资源

有些分布式集群(例如redis集群)的应用是有状态的，集群各个节点应用程序不能随便删除和创建，每个节点都有各自的状态，即使有节点挂了，需要在原节点上重启，kubernetes集群的Deployment资源不能满足有状态程序的部署，可以用StatefulSet资源来部署有状态的应用程序，在kubernetes集群部署有状态的应用程序是比较麻烦的，需要关联的pv、pvc、headless service等资源，StatefulSet管理的Pod都有各自专用的pv和pvc，用来存储数据，关联headless service才能给每一个Pod固定的标识符，即使重启或删除Pod，Pod名称、在节点位置都不会改变，StatefulSet也支持滚动更新和伸缩扩容。

使用StatefulSet部署Pod的特点：

- 稳定且性一的网络标识符
- 稳定且持久的存储
- 有序、平滑地部署和扩展
- 有序、平滑地删除和终止
- 有序的滚动更新

<br>

创建StatefulSet资源前，在集群外的虚拟机先准备3个挂nfs载点目录/data/volumes/v01、/data/volumes/v02、/data/volumes/v03。然后在集群上创建3个存储卷pv，具体配置文件看9.4节的pv资源创建示例。

```bash
# 在集群外的虚拟机上的三个挂载点目录下创建index.html文件给应用程序使用
echo '<h1> volume 01 data </h1>' > /data/volumes/v01/index.html
echo '<h1> volume 02 data </h1>' > /data/volumes/v02/index.html
echo '<h1> volume 03 data </h1>' > /data/volumes/v03/index.html
```

<br>

创建statefulSet资源配置文件statefulSet-demo.yml，配置文件中必须包含三个组件：headless service、StatefulSet、volumeClaimTemplate(存储卷申请模板)，内容如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    app: httpd-pod
  clusterIP: None
  ports:
  - name: httpd-ports
    port: 80
    targetPort: 80

---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: httpd-sts
spec:
  serviceName: httpd-svc
  replicas: 2
  selector:
    matchLabels:
      app: httpd-pod
  template:
    metadata:
      labels:
        app: httpd-pod
    spec:
      containers:
      - name: httpd
        image: busybox
        imagePullPolicy: IfNotPresent
        command: ["/bin/sh","-c", "/bin/httpd -f -h /usr/share/web/"]
        ports:
        - name: httpd
          containerPort: 80
        volumeMounts:
        - name: htmlfiles
          mountPath: /usr/share/web/

  # 为每个pod动态生成一个pvc，创建statefulSet之前必须先准备好pv
  volumeClaimTemplates:
  - metadata:
      name: htmlfiles
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 2Gi
```

<br>

测试是否访问存储卷的index.html文件、重启、动态伸缩：

```bash
# 启动
kubectl apply -f statefulSet-demo.yml

# 查看pv
kubectl get pv
  # NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIMAGE
  # pv001     2Gi        RWO,RWX        Retain           Bound       default/htmlfiles-httpd-sts-0
  # pv002     2Gi        RWO,RWX        Retain           Bound       default/htmlfiles-httpd-sts-1
  # pv003     2Gi        RWO,RWX        Retain           Available

# 查看pvc
kubectl get pvc
  # NAME                    STATUS    VOLUME    CAPACITY   ACCESS MODES
  # htmlfiles-httpd-sts-0   Bound     pv001     2Gi        RWO,RWX
  # htmlfiles-httpd-sts-1   Bound     pv002     2Gi        RWO,RWX

# 查看pod的ip地址
kubectl get pod -o wide
  # NAME          READY     STATUS    RESTARTS   AGE       IP            NODE
  # httpd-sts-0   1/1       Running   0          36s       10.244.1.66   node01
  # httpd-sts-1   1/1       Running   0          34s       10.244.2.77   node02

# 测试访问是否正常
curl 10.244.1.66
  # <h1> volume 01 data </h1>
curl 10.244.2.77
  # <h1> volume 02 data </h1>

# 动态伸缩，把配置中replicas的值改为3，wq保存。
kubectl edit sts httpd-sts

# 查看扩展的pod是否正常运行
kubectl get pod -o wide
  # NAME          READY     STATUS    RESTARTS   AGE       IP            NODE
  # httpd-sts-0   1/1       Running   0          29m       10.244.1.66   node01
  # httpd-sts-1   1/1       Running   0          29m       10.244.2.77   node02
  # httpd-sts-2   1/1       Running   0          21s       10.244.1.67   node01

# 退出所有pod，然后重新重启，查看pod的名字、对应的存储卷、运行的node是否一致
kubectl delete -f statefulSet-demo.yml  # pvc会保留，保证pod挂载存储卷不变。
kubectl apply -f statefulSet-demo.yml

# 查看pod的ip地址
get pod -o wide
  # NAME          READY     STATUS    RESTARTS   AGE       IP            NODE
  # httpd-sts-0   1/1       Running   0          17s       10.244.1.68   node01
  # httpd-sts-1   1/1       Running   0          16s       10.244.2.78   node02

# 再次访问pod，数据也保持一致，说明pod退出前后的状态不变，启动和关闭pod都是有顺序的。
```

<br>

#### 6.5 HPA资源

HPA全称Horizontal Pod Autoscaling，即pod的水平自动扩展，根据Pod当前系统的负载来自动水平扩容，如果系统负载超过预定值，就开始增加Pod的个数，如果低于某个值，就自动减少Pod的个数，HPA的v2版本除了根据CPU资源使用情况去度量系统的负载，还可以根据其他资源作为度量(例如内存、访问数)。HPA不属于pod控制，是元数据资源。

创建HPA资源配置文件hpa-demo.yml，内容如下：

```yaml
# ---------------------- Deployment 资源 -------------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dm
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
      tier: frontend
  template:
    metadata:
      name: nginx-pod
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.2
        ports:
        - name: nginx
          containerPort: 80
        resources:
          requests:
            cpu: 1m
            memory: 10Mi
          limits:
            cpu: 100m
            memory: 100Mi

---

# ---------------------- Service资源 -------------------------
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-np
spec:
  selector:
    app: nginx
    tier: frontend
  type: NodePort
  ports:
  - name: nginx-ports
    port: 80
    targetPort: 80
    nodePort: 30080

---

# ---------------------- HPA 资源 -------------------------
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
  labels:
    project: nginx-pod
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-dm
  minReplicas: 1
  maxReplicas: 10
  # autoscaling/v1群组，根据CPU资源使用情况去度量
  # targetCPUUtilizationPercentage: 66

  # autoscaling/v2beta1群组，根据CPU、内存资源使用情况去度量
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 66
  - type: Resource
    resource:
      name: memory
      targetAverageValue: 66Mi
```

<br>

测试是否能够自动伸缩Pod：

```bash
# 安装压测工具ab
yum install -y httpd-tools

# 在master节点上压测
ab -c 10 -n 500000 http://master:30080/index.html

# 监控pod数量
kubectl get pod -w

# 查看hpa
 kubectl describe hpa nginx-hpa
```

<br><br>

### 7 Service资源

因为Pod资源是有生命周期的，Pod的删除或新创建，ip地址会改变，为了给访问Pod的客户端一个固定的访问端点(ip:port)，需要在客户端和Pod之间添加一层Service资源，Service的ip和端口固定的，Service资源依赖于安装集群时部署的coreDNS附件。

模型：userspace、iptable(默认)、ipvs(新版本支持ipvs，安装集群时要手动配置，系统也要求支持ipvs，如果不支持会降级为iptable)

<br>

Service分类：

| 名称 | 说明 |
| :------ | :------ |
| ClusterIP | 只在集群内部可达的私网地址，集群外部无法访问 |
| NodePort | 通过节点端口映射使得集群外部可达，是在ClusterIP基础上添加一层节点端口映射，访问路径：集群外客户端-->节点ip:port-->ClusterIP:port-->PodIP:port |
| LoadBalancer | 如果客户端只访问一个节点，会对该节点造成很大压力，一般在NodePort之前加入负载均衡，加入负载均衡器有两种方式：一是使用公有云的LBaaS，二是自己搭建负载均衡器 |
| ExternalName | 当Pod客户端想访问集群之外的服务时，使用前几种类型Service是无法绕过集群直接访问外部服务的，使用ExternalName类型的主机名或域名映射方式可以和集群外通信 |
| HeadLess | 特殊类型Service(无头服务)，直接把服务名称映射到Pod的Ip上 |

<br>

kubernetes网络分为三类：node network、pod network、cluster network，前两种是真实存在的，而cluster network是虚拟的，仅用于Service资源。

<br>

Service工作模式：

在用户空间的Client Pod需要访问Server Pod时，需要到内核空间的Service IP中查找符合标签选择器的Server Pod的ip地址，拿到Server Pod真实ip地址后，经过overlay网络直接去访问对应Server Pod。当Server Pod的地址发生改变后，kube-apiserver会保存改变信息保存到etcd中，每个节点的kube-proxy监听到kube-apiserver有信息(ip地址)改变，kube-proxy会把改变后的ip地址同步到Service IP中，各个模块信息更新是同步的，使得Client Pod每次都能找到正确的Server Pod地址，相对于Client Pod来说，服务端点地址是固定的。而且Serice IP对同一类型Server Pod内置了负载均衡，如下图所示：

![Service工作原理](https://pic.zhuyasen.com/815193_09_service%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.jpg)

<br>

#### 7.1 ClusterIP类型的Service资源

创建ClusterIP类型的Service资源配置文件service-clusterIP-demo.yml，这个Service资源关联使用Deployment创建的3个Pod，内容如下：

```yaml
# ---------------------- Deployment资源 ----------------------
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-dm

# 下面是ReplicaSet的spec
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx-pod
      # 注意：这里标签一定要包括上面自定义的selector标签
      labels:
        app: nginx
        tier: frontend
    # 下面就是pod的spec
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.2
        imagePullPolicy: IfNotPresent
        ports:
        - name: nginx
          containerPort: 80

# 资源定义分割符
---

# ---------------------- Service资源 ----------------------
apiVersion: v1
kind: Service

metadata:
  name: nginx-svc

spec:
  # 这里选择器要关联到哪些Pod资源
  selector:
    app: nginx
    tier: frontend
  type: ClusterIP
  ports:
  - name: nginx-ports
    port: 80
    targetPort: 80
```

<br>

测试Service是否关联Deployment下的3个Pod：

```bash
# 查看select是否匹配
kubectl get deployments -o wide
kubectl get pod --show-labels -o wide
kubectl get service -o wide

# 判断是否关联成功，从获取service中得到虚拟ip地址10.106.176.161，
curl 10.106.176.161

# service资源内置负载均衡器，每次访问都会随机访问到3个nginx服务中的一个。
```

<br>

#### 7.2 NodePort类型的Service资源

创建NodePort类型的Service资源配置文件service-notePort-demo.yml，这个Service资源关联使用Deployment创建的3个Pod，内容如下：

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-svc-np

spec:
  # 这里选择器要关联到哪些Pod资源
  selector:
    app: nginx
    tier: frontend
  type: NodePort
  ports:
  - name: nginx-ports
    port: 80
    targetPort: 80
    # 确保节点端口没被使用过，也可不指定，让系统分配随机端口
    nodePort: 30080
```

测试在集群外部浏览器 http://<集群节点ip>:30080 是否可以访问。

<br>

#### 7.3 Headless类型的Service资源

创建Headless类型的Service资源配置文件service-headless-demo.yml，这个Service资源关联使用Deployment创建的3个Pod，内容如下：

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-svc-none

spec:
  # 这里选择器要关联到哪些Pod资源
  selector:
    app: nginx
    tier: frontend
  clusterIP: None
  ports:
  - name: nginx-ports
    port: 80
    targetPort: 80
```

<br>

通过Service的名称nginx-svc-none查看域名解析结果是否直接解析到各个关联Pod的ip地址上。

```bash
dig -t A nginx-svc-none.default.svc.cluster.local @10.96.0.10

# 解析结果，Service名称会直接映射到Pod的Ip上
  # nginx-svc-none.default.svc.cluster.local. 5 IN A 10.244.1.38
  # nginx-svc-none.default.svc.cluster.local. 5 IN A 10.244.2.66
  # nginx-svc-none.default.svc.cluster.local. 5 IN A 10.244.2.67
```

<br>

#### 7.4 通过Service名称访问

在kubernetes集群创建的每个容器中，容器的DNS中动态添加资源记录(A记录、服务记录)，使得在每个容器中都能使用Service名称来访问。资源记录格式：SVC_NAME.NS_NAME.DOMAIN.LTD，例如：nginx-svc.default.svc.cluster.local

```bash
# 每个容器创建后都会添加同样的nameserver和search到/etc/resolv.conf文件
cat /etc/resolv.conf
    # nameserver 10.96.0.10
    # search default.svc.cluster.local svc.cluster.local cluster.local
    # options ndots:5

# /etc/resolv.conf文件的ip地址10.96.0.10是怎么来的？
# 在master节点有很多个系统级别pod，其中有两个pod名称为coredns开头的pod，查看系统级别的pod：
kubectl get pods -n kube-system -o wide

# 而这两个coredns开头的pod又是被名称为kube-dns(系统级别的Service)关联和管理的，查看系统级别的Service：
kubectl get service -n kube-system -o wide

# 其中名为kube-dns的Service的ip地址就是10.96.0.10


# 有了10.96.0.10(kube-dns)和search后缀内容，在访问的域名(Service名称)不完整的时候，会把不完整的域名和后缀(/etc/resolv.conf文件search内容)拼接成为完整域名，通过完整的域名，可以解析service名称对应的ip地址了。

# 例如解析Service名称对应的地址
nslookup -t A nginx-svc.default.svc.cluster.local

# 通过Service名称和端口访问，只要Service名称不变，Service的虚拟ip地址怎样变都不会影响客户端正确访问。解决访问只需要知道Service名称和端口，不需要理会Service的ip地址和端口、Pod的ip地址和端口。
wget -O - -q nginx-svc:80
```

<br><br>

### 8 ingress资源

Service属于四层网络模型，是在网络协议TCP或UDP之上，而http或https属于七层网络模型，无法使用Service来解析七层的http或https，为了解决这个问题，在节点运行一个代理服务的Pod，这个Pod里的容器共享宿主机网络(host network)，外部可以直接访问，并且运行属于七层网络模型代理服务(Nginx、Traefik、Envoy)，所有的客户端的请求都先经过代理服务的Pod，代理服务Pod把请求转发给其他Pod，而且各个Pod之间是同一个网段，可以直接通信。为了解决代理服务Pod单点问题，使用DaemonSet类型的Pod控制器，DaemonSet控制的Pod会在每个node节点运行一个代理服务Pod，如果集群的节点很多的话，选择3个节点专门用来运行服务代理即可，不必每个节点都运行代理服务。

这种代理服务的Pod属于ingress controller，ingress controller是独立于集群master中manager controller。而ingress是集群的一种特殊资源，定义ingress资源时需要指明ingress controller的前端是虚拟主机或url映射。集群中运行很多组的pod(例如：用户、电商、社交、交易、物流等)，ingress controller怎么知道哪些请求是指向哪组Pod？可以使用虚拟主机或url映射(/usr/、/shop/等)，因为Pod是有生命周期的，Pod的Ip地址有可能随时改变，ingress controller经过url映射的Pod就保证存在呢？解决方法是借助headless ClusterIp类型的Service资源，Service不是用来代理，只用来对Pod进行分组，ingress实时监控Service中Pod的ip地址是否变化，如果有变化，ingress实时获取到变化后Pod的ip信息，然后注入到ingress controller的配置文件中，并触发ingress controller中容器进程重载配置文件。

![ingress工作原理](https://pic.zhuyasen.com/815193_10_ingress%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.jpg)

<br>

#### 8.1 安装ingress controller

ingress使用详情请看[官方文档](https://kubernetes.io/docs/concepts/services-networking/ingress/)，ingress controller的安装请看[官方文档](https://kubernetes.github.io/ingress-nginx/deploy/)

```bash
# 安装ingress-nginx controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml

# 安装后可以看到ingress-nginx在集群创建了一堆资源，包括namespace、service、configMap、deployment、pod

# 如果node节点不能翻墙，无法正常启动相关pod，原因是下载镜像失败，解决办法是设置设置docker代理翻墙或预先下载导入镜像(具体镜像版本查看mandatory.yaml文件)。

# 查看ingress-nginx各类资源列表
kubectl get ns -n ingress-nginx
kubectl get cm -n ingress-nginx
kubectl get svc -n ingress-nginx
kubectl get deployment -n ingress-nginx
kubectl get pod -n ingress-nginx
```

<br>

#### 8.2 添加外网接入的service资源

在本地测试为了接入集群外的请求，需要另外创建一个NodePort类型service做转发，如果是在公有云部署的集群，建议使用LoadBalancer类型service，参考[service](https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/cloud-generic.yaml)。

创建NodePort类型的service资源配置文件ingress-nginx-serivce.yml，内容如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  ports:
    - name: http
      port: 80
      targetPort: 80
      # 指定宿主机端口
      nodePort: 30080
    - name: https
      port: 443
      targetPort: 443
      # 指定宿主机端口
      nodePort: 30443
```

<br>

查看调度器nginx ingress controller是否工作正常：

```bash
# 所有从集群外进来的http使用30080，https使用30443
curl mater:30080

# 如果返回default backend - 404，说明调度器已经正常工作了，只是没有后端而已
```

<br>

#### 9.3 ingress nginx代理http示例

开始部署app，创建headless类型的service资源和Deployment资源配置文件myapp-service-deployment.yml，其中service资源只用来对Deployment的Pod进行分组的，获取到Pod的ip给ingress资源使用，内容如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  clusterIP: None
  ports:
  - name: http
    port: 80
    targetPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-dm
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      name: myapp
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: ikubernetes/myapp:v3
        ports:
        - name: http
          containerPort: 80
```

<br>

创建ingress资源配置文件http-myapp-ingress.yml，ingress会监听service服务myapp-svc，一旦发现myapp-svc发生改变，ingress会从myapp-svc获取最新的Pod信息，把最新配置信息注入到nginx配置信息。内容如下：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name:
  annotations:
    # 声明使用类型：nginx、traefik、envoy，让ingress controller匹配对应的规则
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  # 这里使用虚拟主机，也可以使用url映射
  - host: demo.myapp.com
    http:
      paths:
      - path:
        backend:
          serviceName: myapp-svc
          servicePort: 80
```

测试ingress是否能够动态注入nginx.conf，并触发nginx重载配置信息：

```bash
# 获取ingress nginx的pod名称
kubectl get pod -n ingress-nginx

# 进入容器
kubectl exec -it -n ingress-nginx nginx-ingress-controller-6bd7c597cb-tf7gx -- /bin/sh

# 查看nginx配置是否包含虚拟主机demo.myapp.com配置信息，Deployment类型资源myapp-dm下的pod的ip列表是否也存在
cat nginx.conf

# 如果后端的Pod挂掉了，Pod的ip地址变化会被ingress通过service检测到，ingress把变化的信息重新注入到nginx配置文件中。

# 在节点宿主机添加测试域名demo.myapp.com
echo '192.168.8.90  demo.myapp.com' > /etc/hosts

# 在一个终端不断获取myapp名称，查看删除pod后获取的名称是否有更新
while true; do curl demo.myapp.com:30080/hostname.html; sleep 1;done

# 获取pod
kubectl get pod

# 删除两个pod
kubectl delete pod myapp-dm-59f7c855f7-5tmhw myapp-dm-59f7c855f7-5v6zw

# 从结果上看，pod后获取的名称是更新的，说明ingress能把最新pod的信息动态注入nginx配置中，并触发nginx服务重载配置。
```

<br>

#### 9.3 ingress nginx代理https示例

在http示例上做ingress nginx代理https示例，因为https访问需要证书，所以在本地制作一个测使用的自签证书：

```bash
# 生成私钥tls.key
openssl genrsa -out tls.key 2048

# 生成自签证书
openssl req -new -x509 -key tls.key -out tls.crt -subj /C=CN/ST=Guangdong/L=Guangzhou/O=IgressDemo/CN=demo.myapp.com

# 注意生成的证书不能直接复制到ingress nginx pod的nginx.conf中，先要使用k8s的secret资源转成base64格式。

# 创建secret资源
kubectl create secret tls myapp-ingress-secret --cert=tls.crt --key=tls.key

# 查看已创建的myapp-ingress-secret
kubectl get secrets  myapp-ingress-secret -o yaml
```

<br>

创建ingress资源配置文件https-myapp-ingress.yml，内容如下：

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myapp-tls-ingress
  annotations:
    # 声明使用类型：nginx、traefik、envoy，让ingress controller匹配对应的规则
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  # 使用虚拟主机，也可以使用url映射
  - host: demo.myapp.com
    http:
      paths:
      - path:
        backend:
          serviceName: myapp-svc
          servicePort: 80
  # 证书信息
  tls:
  # 可以多个host
  - hosts:
    - demo.myapp.com
    secretName: myapp-ingress-secret
```

<br>

测试ingress是否能够动态注入nginx.conf，并触发nginx重载配置信息：

```bash
# 获取ingress nginx的pod名称
kubectl get pod -n ingress-nginx

# 进入容器
kubectl exec -it -n ingress-nginx nginx-ingress-controller-6bd7c597cb-tf7gx -- /bin/sh

# 查看nginx配置信息是否包含了证书信息
cat nginx.conf

# 在宿主机的chrome浏览器上访问https://demo.myapp.com:30443/hostname.html，第一次访问会提示不安全信息，点击高级继续前往访问，如果访问正常，说明签名有效
```

<br><br>

### 9 存储卷资源

Pod支持的存储卷类型非常多：emptyDir、gitRepo、nfs......，更多信息看官网 https://kubernetes.io/docs/concepts/storage/volumes

<br>

#### 9.1 emptyDir类型存储卷

emptyDir类型存储卷只作为临时存储，存储数据随着Pod的生命周期结束而消失。

创建emptyDir临时类型存储卷资源配置文件volume-empty-demo.yml，在一个容器里持续更新index.html内容，在另一个容器读取index.html，内容如下：

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: httpd-empty

spec:
  containers:
  - name: busybox
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c", "while true; do echo $(date) > /data/index.html; sleep 3; done"]
    ports:
    - name: httpd
      containerPort: 80
    volumeMounts:
    - name: htmlfiles
      mountPath: /data/
  - name: httpd
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c", "/bin/httpd -f -h /data/"]
    volumeMounts:
    - name: htmlfiles
      mountPath: /data/

  volumes:
  - name: htmlfiles
    # 临时存储，Pod删除，数据也就消失了
    emptyDir: {}
```

其中gitRepo类型是在emptyDir类型增加了自动从git仓库拉取代码的功能，但是修改数据不会自动提交到git的仓库。

<br>

#### 9.2 hostPath类型存储卷

hostPath类型存储数据独立于Pod之外，Pod结束不会影响存储的数据，数据一直存储在Pod的宿主机自定义目录里，hostPath类型存储卷的缺点是当Pod结束了，如果被调度器分配在另一个节点上运行，之前共享数据就无法找到，当然可以指定Pod运行在固定的宿主上。

创建hostPath类型存储卷资源配置文件volume-hostPath-demo.yml，内容如下：

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: httpd-hostpath
  labels:
    app: httpd

spec:
  containers:
  - name: httpd
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c", "/bin/httpd -f -h /usr/share/web/"]
    ports:
    - name: httpd
      containerPort: 80
    volumeMounts:
    - name: htmlfiles
      # 容器存储目录
      mountPath: /usr/share/web/

  # Pod固定运行在节点node01
  nodeName: node01
  
  volumes:
  - name: htmlfiles
    # 宿主机存储路径
    hostPath:
      path: /data/web/
      type: DirectoryOrCreate
```

<br>

测试共享是否成功：

```bash
# 启动pod
kubectl create -f valuem-hostPath.yml

# 在node01节点上创建index.html文件
echo '<h1> volume host path node01:/data/web/ </h1>' > /data/web/index.html

# 查看pod的ip地址
kubectl get pod -o wide

# 测试是否能够访问
curl 10.244.1.47
```

<br>

#### 9.3 nfs类型存储卷

emptyDir、hostPath这些的存储数据都是存储在节点上，如果要想在集群节点之外存储数据(例如使用nfs、RBD等)，这种节点外存储是真正意义上的持久性存储，缺点是存在单点问题。

想在集群节点之外使用nfs存储数据，首先测试node节点能否成功挂载集群外的nfs：

```bash
# 准备一个新虚拟机，在集群外这个虚拟机安装nfs服务。
  # 安装nfs
  yum install -y nfs-utils

  # 启动nfs
  systemctl start nfs

  # 查看启动状态和监听端口2049
  systemctl status nfs
  ss -lnt

  # 准备挂载点
  mkdir -p /data/volumes
  echo '/data/volumes 192.168.0.0/16(rw,no_root_squash)' >> /etc/exports

  # 查看已准备的fs
  exportfs -arv
  showmount -e


# 在集群的各个node节点上测试能否挂载成功
  # 在node节点安装nfs
  yum install -y nfs-utils

  # 挂载nfs，其中192.168.8.200为nfs服务的ip
  mount -t nfs 192.168.8.200:/data/volumes /mnt

  # 查看是否挂载成功
  mount | grep "data/volumes"

  # 在节点的/mnt/下面创建一个测试文件test
  echo 'hello nfs' > /mnt/test
  # 如果在nfs服务器上也能看到test文件，说明nfs挂载正常。

  # 在各个节点测试挂载成功后取消挂载nfs
  umount /mnt
```

<br>

创建nfs类型存储卷配置文件volume-nfs-demo.yml，内容如下：

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: httpd-vl-nfs
  labels:
    app: httpd

spec:
  containers:
  - name: httpd
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c", "/bin/httpd -f -h /usr/share/web/"]
    ports:
    - name: httpd
      containerPort: 80
    # 首先挂载存储卷再执行command
    volumeMounts:
    - name: htmlfiles
      # 容器挂载路径，路径不存在会自动创建
      mountPath: /usr/share/web/

  volumes:
  - name: htmlfiles
    # nfs类型
    nfs:
      path: /data/volumes/
      # 域名或ip
      server: 192.168.8.200
```

测试共享是否成功：

```bash
# 在nfs服务器所在的主机上创建index.html文件
mkdir -p /data/volumes/
echo '<h1> volume nfs:/data/volumes/ </h1>' > /data/volumes/index.html

# 启动pod
kubectl create -f valuem-nfs.yml

# 查看pod的ip地址
kubectl get pod -o wide

# 测试是否能够访问
curl 10.244.1.47
```

<br>

#### 9.4 pvc和pv资源

kubernetes为了方便管理，提供了独立的pvc(Persistent Volume Claim)资源和pv(Persistent Volume)资源。使用时应先创建pvc、pv资源(创建时各自声明自己的存储容量大小、权限)，在创建Pod资源时的volumes去绑定pvc，pvc根据容量大小和权限去匹配pv，如果存在符合条件的pv，pv和pvc会绑定，而Pod和pvc也是绑定的，相当于建立了一条Pod<--> pvc <--> pv独立通道，其他pvc不能再绑定已使用的pv；如果没有符合条件的pv，Pod会一直在pending状态。其实最好预先规划好应用程序需要多少储存空间要，pvc、pv资源才知道要分配多大存储空间。

pv属于集群级别，所有命名空间都可以使用，而pvc是属于名称空间级别的。

![pvc和pv关系](https://pic.zhuyasen.com/815193_11_pvc%E5%92%8Cpv%E6%9E%B6%E6%9E%84.jpg)

<br>

在集群之外的nfs服务器上创建挂载目录

```bash
# 创建3个挂载目录
mkdir /data/volumes/v{01,02,03}

# 准备挂载点
echo '/data/volumes/v01 192.168.0.0/16(rw,no_root_squash)' >> /etc/exports
echo '/data/volumes/v02 192.168.0.0/16(rw,no_root_squash)' >> /etc/exports
echo '/data/volumes/v03 192.168.0.0/16(rw,no_root_squash)' >> /etc/exports

# 在集群外的虚拟机上的三个挂载点目录下创建index.html文件给应用程序使用
echo '<h1> volume 01 data </h1>' > /data/volumes/v01/index.html
echo '<h1> volume 02 data </h1>' > /data/volumes/v02/index.html
echo '<h1> volume 03 data </h1>' > /data/volumes/v03/index.html

# 查看已准备的fs
exportfs -arv
showmount -e
```

<br>

创建pv资源配置文件volume-pv.yml，内容如下：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
  labels:
    app: pv001
spec:
  # nfs类型
  nfs:
    path: /data/volumes/v01
    server: 192.168.8.200
  accessModes: ["ReadWriteMany","ReadWriteOnce"]
  capacity:
    storage: 1Gi

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv002
  labels:
    app: pv002
spec:
  # nfs类型
  nfs:
    path: /data/volumes/v02
    server: 192.168.8.200
  accessModes: ["ReadWriteMany","ReadWriteOnce"]
  capacity:
    storage: 2Gi

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv003
  labels:
    app: pv003
spec:
  # nfs类型
  nfs:
    path: /data/volumes/v03
    server: 192.168.8.200
  accessModes: ["ReadWriteMany","ReadWriteOnce"]
  capacity:
    storage: 4Gi
```

<br>

```bash
# 启动
kubectl create -f volume-pv.yml

# 查看pv资源
kubectl get pv
  # NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
  # pv001     1Gi        RWO,RWX        Retain           Available                                      16s
  # pv002     2Gi        RWO,RWX        Retain           Available                                      16s
  # pv003     4Gi        RWO,RWX        Retain           Available                                      16s

# 默认回收策略是保留，如果pvc和pv绑定成功，pv的当前状态显示bound，此时不运行其他pvc再次绑定，也不允许删除。
```

<br>

创建pvc和pod资源配置文件volume-pvc-pod.yml，内容如下：

```yaml
# 定义pvc资源
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: httpd-vl-pvc
spec:
  # 这里权限必须是pv的accessModes权限的子集
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 2Gi

---

# 定义pod资源
apiVersion: v1
kind: Pod
metadata:
  name: httpd-vl-pod

spec:
  containers:
  - name: httpd
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c", "/bin/httpd -f -h /usr/share/web/"]
    ports:
    - name: httpd
      containerPort: 80
    volumeMounts:
    - name: htmlfiles
      mountPath: /usr/share/web/

  volumes:
  - name: htmlfiles
    persistentVolumeClaim:
      # 绑定的pvc
      claimName: httpd-vl-pvc
```

测试是否共享数据成功：

```bash
# 启动
kubectl create -f volume-pvc-pod.yml

# 查看pv资源
kubectl get pv

# 查看pvc资源
kubectl get pvc

# 查看绑定nfs的挂载点
kubectl describe pod httpd-vl-pod

# 查看信息得知绑定的nfs服务器挂载点是v02，在nfs服务器上的v02挂载点添加index.html
echo '<h1> volume nfs:/data/volumes/ </h1>' > /data/volumes/v02/index.html

# 查看pod的ip地址
kubectl get pod -o wide

# 测试访问是否能成功
curl 10.244.1.47
```

<br><br>

### 10 configMap资源

configMap资源存储key-value配置数据，这些key-value是怎样传递给容器的呢？通过env方式和挂载存储卷方式两种方式。

#### 10.1 env方式

使用命令方式创建env：

```bash
# 数据来自key-value
kubectl create configmap httpd-cm-kv-env --from-literal=testEnv="configmap test" --from-literal=portEnv="8080"

# 数据来自文件，文件名为key，文件内容为value
kubectl create configmap httpd-cm-file-env --from-file=./app.toml
```

<br>

创建configMap资源配置文件configMap-env-demo.yml，configMap数据通过env传递给容器的环境变量，内容如下：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: httpd-cm-env
# key-value
data: {"testEnv": "configMap test", "portEnv": "8080"}

---

# Pod资源，不断读取环境变量值
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod-env

spec:
  containers:
  - name: httpd
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c", "mkdir -p /usr/share/web/; httpd -f -h /usr/share/web/ & while true; do echo $TEST_ENV > /usr/share/web/index.html; sleep 5;done;"]
    ports:
    - name: httpd
      containerPort: 80
    env:
    - name: TEST_ENV
      valueFrom:
        configMapKeyRef:
          name: httpd-cm-env
          # configMap资源的testEnv值传给容器环境变量TEST_ENV
          key: testEnv
          # 如果为true，指定的configMap的key必须存在，否则容器即启动报错
          optional: false
    - name: Port_ENV
      valueFrom:
        configMapKeyRef:
          name: httpd-cm-env
          key: portEnv
```

<br>

测试环境变量是否传递过去：

```bash
# 查看configMap
kubectl describe cm httpd-pod-env

# 查看pod的ip
kubectl get pod -o wide

# 查看是否输出环境变量
curl 10.244.2.77

# 修改环境变量，看看是否能够动态传递给Pod里容器的环境变量
kubectl edit cm httpd-cm-env

# 再次查看环境变量
curl 10.244.2.77

# 输出的环境变量值没有改变，也就是说使用configMap传递给容器的env时，不支持动态修改env值
```

<br>

#### 10.2 存储卷挂载方式

无论创建的configMap数据来自key-value还是来自文件，创建容器时使用存储卷挂载方式关联configMap资源，会在容器的挂载目录下configMap的key作为文件名，对应的value为文件内容。通过存储卷挂载方式，修改configMap的资源内容会直接反映到容器的挂载目录上，因此可作为应用程序的配置中心，各个容器的应用程序配置统一管理，并且只需修改configMap一次，各个容器的挂载目录能够实时同步配置内容。

首先创建一个来自配置文件的configMap资源httpd-cm-file，文件名为key，文件内容为value，httpd-cm-file作为存储卷挂载内容。

> kubectl create configmap httpd-cm-file --from-file=./app.toml

<br>

然后创建一个Pod资源配置文件configMap-volume-demo.yml，Pod挂载存储卷类型为configmap，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod-vl

spec:
  containers:
  - name: httpd
    image: busybox
    imagePullPolicy: IfNotPresent
    # 复制配置文件app.toml内容作为index.html的内容
    command: ["/bin/sh","-c", "mkdir -p /usr/share/web/; httpd -f -h /usr/share/web/ & while true; do cp /usr/config/app.toml /usr/share/web/index.html; sleep 5;done;"]
    ports:
    - name: httpd
      containerPort: 80
    volumeMounts:
    - name: htmlfiles
      mountPath: /usr/config/
      readOnly: true

  volumes:
  - name: htmlfiles
    # configMap类型存储卷，把key作为文件名，value作为文件内容映射到到容器中，在Pod启动前要先创建名为httpd-cm-file的configMap
    configMap:
      name: httpd-cm-file
```

<br>

测试能否动态修改配置文件内容：

```bash
# 查看configMap
kubectl describe cm httpd-cm-file

# 查看pod的ip
kubectl get pod -o wide

# 查看是否输出配置文件内容
curl 10.244.1.73

# 修改配置文件内容，看看是否能够动态显示改变后的配置文件内容
kubectl edit cm httpd-cm-file

# 查看输出配置文件内容
curl 10.244.1.73

# 从结果看来可以动态修改配置文件内容，但是容器的app需要重载配置才可以使修改的配置文件生效。
```

<br><br>

### 11 secret资源

secret资源用来存储敏感的key-value数据，类似configMap资源。

secret资源有三种类型：

- generic：普通密码类型，例如mysql_root_password
- tls：证书类型，例如：https需要证书
- docker-registry：docker镜像仓库认证信息，当Pod需要去拉取需要鉴权的镜像仓库时，Pod配置文件需要填写字段pod.spec.[imagePullSecrets](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names)。

使用方法和configMap类似。

<br>

一个创建secret示例：

```bash
# 数据来自key-value
kubectl create secret generic mysql-secret --from-literal=user=root --from-literal=passwor=123456

# 数据来自文件
kubectl create secret generic my-secret --from-file=ssh-privatekey=~/.ssh/id_rsa
--from-file=ssh-publickey=~/.ssh/id_rsa.pub

# 查看secret，只显示大小
kubectl describe secret mysql-secret

# 查看secret方法
# 获取mysql-secret的base64值
kubectl get secrets mysql-secret -o yaml

# 经过bash64反编码即可得到原始值
echo MTIzNDU2 | base64 -d

# 由此说明secret资源也不是那么的安全。
```

<br>

建议不要使用env方式传递给Pod，使用env方式传递到Pod之后变成了明文，任何人进入容器都可以获取明文密码，建议使用volume挂载方式，然后把挂载的问权限设置为400，制作镜像时设置进入容器的用户为非root用户，使其没权限查看。

创建一个Pod资源配置文件secret-volum-demo.yml，Pod挂载存储卷类型为secret，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod-secret-vl

spec:
  containers:
  - name: httpd
    image: busybox
    imagePullPolicy: IfNotPresent
    # 复制配置文件app.toml内容作为index.html的内容
    command: ["/bin/sh","-c", "mkdir -p /usr/share/web/; httpd -f -h /usr/share/web/ & while true; do cp /usr/secret/user /usr/share/web/index.html; sleep 5;done;"]
    ports:
    - name: httpd
      containerPort: 80
    volumeMounts:
    - name: htmlfiles
      mountPath: /usr/secret/
      readOnly: true

  volumes:
  - name: htmlfiles
    # secret类型存储卷，把key作为文件名，value作为文件内容映射到到容器中
    secret:
      secretName: mysql-secret
```

secret与configmap挂载存储卷一样，修改value值动态映射到Pod里。

<br><br>

### 12 kubernetes认证、授权

kubernetes集群有两类认证账号，一是用户账号(人)；二是服务账号，例如创建的Pod托管在集群上，想要操作其他Pod，需要有Pod对应服务账号认证授权。

![认证类型](https://pic.zhuyasen.com/815193_12_%E7%94%A8%E6%88%B7%E7%B1%BB%E5%9E%8B.jpg)

<br>

kubernetes认证授权过程：用户 --> 认证 --> 授权 --> 准入控制

- 认证阶段：有RESTful的token、ssh等认证方式。
- 授权：检查用户是否拥有操作资源权限。
- 准入控制：用户虽然有操作某些资源，检查用户操作级联到的资源和关联的环境是否有权限，是否在操作范围内。

<br>

所有kubectl命令操作经过apiServer后会转化为固定api格式：

```bash
# 非核心群组格式：master地址:端口+apis+apiVersion+名称空间key+名称空间名+资源类型+资源名称
# 示例：http://172.168.8.90:6443/apis/apps/v1/namespaces/default/deployments/myapp-dm

# 核心群组格式： master地址:端口+api+名称空间名
```

<br>

配置一个kubectl客户端连接多个kubernetes集群方式，一是使用命令方式添加集群认证；二是在~/.kube/config配置文件添加集群认证信息。

```bash
# 查看集群上下文列表
kubectl config get-contexts

# 查看kubectl当前操作的集群
kubectl config current-context

# 切换kubectl操作的集群
kubectl config set-context vison@kubernetes
```

<br>

#### 12.1 创建一个serviceaccount资源

```bash
# 创建一个账号，会自动生成一个对应的secret资源(里面有token值)
kubectl create serviceaccount vison

# 查看账号信息
kubectl describe sa vison

# 查看对应的secret
kubectl get secret
# 获取serviceaccount生成的token值
kubectl describe secret vison-token-g28sv

# 账号vison能通过集群的认证，但不代表能操作资源，还需要授权，即绑定
```

<br>

对Pod资源添加固定账号授权示例，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
  labels:
    app: mynginx
spec:
  containers:
  - name: nginx
    image: nginx
  serviceAccountName: vison
```

<br>

#### 12.2 RBAC角色管理

kubernetes集群可以对资源管理力度做得很精细，使用命名空间级别(role、rolebinding)、集群级别(clusterrole、clusterrolebinding)和用户(user、group、serviceaccount)来实现资源精细化管理，如果使用clusterrole集群级别去绑定(rolebinding)空间级别，集群级别的权限范围降级为命名空间级别。

![RBAC模型](https://pic.zhuyasen.com/815193_13_RBAC%E6%A8%A1%E5%9E%8B.jpg)

<br>

(1) 用户和角色绑定，角色和绑定类型都是命名空间级别

```bash
# 创建一个role
kubectl create role pods-reader --verb=get,list,watch --resource=pods

# 绑定role和user
kubectl create rolebinding zhuyasen-pods-reader --role=pods-reader --user=zhuyasen

# 查看role、rolebinding
kubectl describe role pods-reader
kubectl describe rolebinding zhuyasen-pods-reader
```

<br>

(2) 用户和角色绑定，角色和绑定类型都是集群级别

```bash
# 创建一个clusterrole
kubectl create clusterrole pods-all-reader --verb=get,list,watch --resource=pods

# 绑定clusterrole和user
kubectl create clusterrolebinding zhuyasen-pods-all-reader --clusterrole=pods-all-reader --user=zhuyasen

# 查看clusterrole、clusterrolebinding 
kubectl describe clusterrole pods-all-reader
kubectl describe clusterrolebinding zhuyasen-pods-all-reader
```

<br>

(3) 用户和角色绑定，角色为集群级别，而绑定为命名空间级别，则用户权限会降级为命名空间级别

```bash
# 创建一个clusterrole
kubectl create clusterrole pods-all-reader --verb=get,list,watch --resource=pods

# 使用命名空间级别绑定clusterrole和user
kubectl create rolebinding zhuyasen-pods-reader --clusterrole=pods-all-reader --user=zhuyasen

# 查看clusterrole、rolebinding
kubectl describe clusterrole pods-all-reader
kubectl describe rolebinding zhuyasen-pods-reader

# 用户权限会降级为命名空间级别
```

<br>

有时托管在集群的Pod需要操作其他Pod资源权限(例如监控)，需要把绑定用户改为绑定serviceAccount资源

```bash
# 创建一个role或clusterrole
kubectl create role pods-reader --verb=get,list,watch --resource=pods

# 创建一个账号，会自动生成一个对应的secret资源(里面有token值)
kubectl create serviceaccount vison-sa

# 把角色(role或clusterrole)和serviceaccount绑定
kubectl create rolebinding vison-pods-reader --clusterrole=pods-reader --serviceaccount=vison-sa
```

<br>

#### 12.3 kubernetes集群UI管理

kubernetes集群UI访问是默认使用认证的https，而dashboard默认使用的自签署证书，默认自生成的证书很明显不是当前使用kubernetes集群签署的，在浏览器上是无权限访问，安装dashboard前要生成自己的kubernetes集群签署证书给运行dashboard的Pod使用。

**(1) 生成dashboard证书**

```bash
# 切换到集群证书目录
cd /etc/kubernetes/pki

# 生成私钥dashboard.key
openssl genrsa -out dashboard.key 2048

# 生成证书请求文件dashboard.csr
openssl req -new -key dashboard.key -out dashboard.csr -subj "/O=dashboard/CN=demo.dashboard.com"

# 生成证书文件dashboard.crt
openssl x509 -req -in dashboard.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out dashboard.crt -days 365

# 把dashboard证书复制到新建目录/etc/kubernetes/pki/dashboard
mkdir -p /etc/kubernetes/pki/dashboard
mv dashboard.* /etc/kubernetes/pki/dashboard
```

<br>

**(2) 安装dashboard插件**

```bash
# 下载安装dashboard配置文件
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

# 如果node节点不能翻墙，无法正常启动相关pod，原因是下载镜像失败，解决办法是设置docker代理翻墙或预先下载导入镜像(具体镜像版本查看kubernetes-dashboard.yaml文件)

# 默认配置Dashboard Service的类型是ClusterIP，集群外是无法访问的，把它改为NodeIp类型，并且设置映射端口固定为为30303
# 搜索Dashboard Service
# spec:
#   type: NodePort
#   ports:
#     - port: 443
#       targetPort: 8443
#       nodePort: 30303

# 安装dashboard插件
kubectl apply -f kubernetes-dashboard.yaml

# 删除默认的自签名证书
kubectl delete secret kubernetes-dashboard-certs -n kube-system

# 添加自己签名证书
kubectl create secret generic kubernetes-dashboard-certs --from-file=/etc/kubernetes/pki/dashboard -n kube-system

# 因为证书请求文件使用demo.dashboard.com域名，在master节点上添加域名解析
echo '192.168.8.90 demo.dashboard.com' >> /etc/hosts

# 查看是Dashboard Service是否NodeIp，并且端口为30303
kubectl get svc -n kube-system

# 在master宿主机的chrome浏览器访问https://demo.dashboard.com:30303，然后点击高级继续前往访问，进入到Kubernetes的仪表板登录界面。
```

<br>

**(3) 生成登录token和授权**

```bash
# 生成登录token
  # 创建系统级别的serviceaccount
  kubectl create serviceaccount dashboard-admin -n kube-system
  # 创建帐号时自动生成token，拿到token值虽然可以登录，但是没有操作资源权限，还需要授权

# 授权
  #把dashboard-admin帐号绑定到cluster-admin(管理员角色， 使用命令kubectl get clusterrole -n kube-system查看)
  kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

# 获取token值，复制token值到浏览器上登录即可。
kubectl get secret -n kube-system | grep dashboard-admin # 获取对应secret名称
kubectl describe secret <secrect名称> -n kube-system | awk '/^token:/{print $2}'
# token值：
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtc2EtdG9rZW4tdDJoNGYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLXNhIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZmU1NDk1ZWEtYzZkOC0xMWU4LWI2MjktMDAwYzI5NmY4YzY0Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1zYSJ9.LLXz3UoEhPrN1l_edxx8K0kG04SSvqkpBERH5PlfPkhV_o8vaHOpWyzyuo5Ik3d6yEXIPoHVr7e-ZTH4fZDTnul94rMChYLUimIEvmu0JI0lxhp9FrkHbjvLtW-Xa34J-1-lg73xBi_nboFSKJAqRtw2rse3y0zQPb8tcIpV_ptgTJmDOXgFfwjmToUAqlieePIlxj3w8QoHInGHziTf0MYmn7O2725VyGoHxi2PSROtXASmVOeghwn52zRXQtDe7SVhhkPHGlCcbK8Wki5N5lZIpWWoD8xv_DOGq3o1tImr02yTsPlrmiCWgJTXHxgwOmXIQD3X3sP5iYxYP04EAg
```

<br>

上面创建serviceaccount和绑定使用了参数-n kube-system，说明该帐号拥有集群管理权限，如果需要创建命名空间级别的帐号，去掉参数默认使用default命名空间即可。

```bash
# 生成只有default命名空间权限的账号
kubectl create serviceaccount dashboard-vison
kubectl create rolebinding dashboard-default-vison --clusterrole=cluster-admin --serviceaccount=default:dashboard-vison
kubectl get secret | grep dashboard-vison
kubectl describe secret <secrect名称>  | awk '/^token:/{print $2}'
# token值：
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC12aXNvbi10b2tlbi1saDQ0ZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQtdmlzb24iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIzOTZhNzUwOC1jNmUxLTExZTgtYjYyOS0wMDBjMjk2ZjhjNjQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtdmlzb24ifQ.DyYsIjpxN6MREBEfDQQyA9Hlm9iU69JRH5oRkkUjA1bz-EkTLbb3Slc7mDztMeXsDxXKu1gBJ63_FAOrikCAXCoj8keH_JPrJGAFi7jHwlvYWqboy59fhcN81rhDxeBvuGUgafeRoHfgZ5j0ZypW9-85LOwYL-QbB4gj-NW77lDjGbrcuNGSrLQYYZSB0aAnyRi5QrT0c8-5aEf27xjTglBcR_xIbUO-WcdfVYdl08inbM5D6wqAyKFSqV5d_FPJDMg0svAcRthM_qaArdj1MKBNtxa_smBrYXUj3hgc49admYNHHlTqB13xv0aELwshzUzjJtKMXdwlqyrN7VDq8Q
```

<br><br>

#### 13 Kubernetes集群网络通信

具体介绍查看官网[cluster-networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)。

Kubernetes与Docker的通信方式不同，Kubernetes集群有4个不同的网络问题需要解决：

- 容器间通信：不需要通过NAT方式，同一个Pod内的多个容器使用lo。
- Pod之间通信：不需要通过NAT方式，在同一网段。
- Pod与Service通信：ip规则。
- Service与集群外通信。

kubernetes本身不提供网络解决方案，只是定义了标准的CNI接口，所有满足CNI接口的网络插件都可以作为kubernetes网络插件，比较流行的网络插件有：flannet、canal、calico等，安装网络插件时把插件配置复制到/etc/cni/net.d下，使用kubelet去加载作为集群网络使用。

| 名称 | 说明 |
| :------ | :------ |
| flannet | 部署简单，不支持网络策略，所有Pod网络没有隔离(在同一网段)，虽然有名称空间隔离，但网络不隔离 |
| calico | 部署比较麻烦，既支持网络策略，也支持地址分配 |
| canal | flannet和calico结合体，通信使用flannel网络，网络策略使用calico网络策略部分 |

<br>

![网络拓扑图](https://pic.zhuyasen.com/815193_15_%E7%BD%91%E7%BB%9C%E6%8B%93%E6%89%91%E5%9B%BE.png)

图中的node1(172.17.4.101)和node2(172.17.4.102)可以通信，在node1和node2都安装网络插件flannel，node1通过overlay直接到达node的Pod，无需地址转换，也就是图中的红色虚线。

<br>

#### 13.1 flannel网络插件

flannel支持后端：VxLAN、host-gw、UDP。VxLAN有vxlan(默认使用)和Directrouting(默认不使用)工作方式，VxLAN使用的overlay技术支持不同网段之间主机通信，但性能没那么好，而host-gw只能在同一网段之间主机通信，性能比VxLAN好，是因为host-gw把主机接口当作网关使用，没有经过多余叠加信息。flannel默认使用的是VxLAN方式通信，如果需要提高通信性能，可以设置在同一个网段使用host-gw，不在同一个网段自动降级为VxLAN。

当使用kubeadm安装kubernetes集群时，安装完后集群是不能正常运行的，因为缺少网络插件flannel，flannel可以直接安装在宿主机上，也可以安装到集群中(daemonSet守护进程)，当网络插件运行在Pod上时，所有安装有kubelet的节点都需要安装网络插件flannel，因为kubelet创建Pod时候需要去flannel插件获取网络，才能启动Pod。

<br>

查看flannel资源：

```bash
# 查看daemonset
kubectl get daemonset -n kube-system

# 查看flannel各个节点运行的Pod
kubectl get pod -n kube-system -o wide | grep flannel

# 查看configmaps
kubectl get configmaps -n kube-system

# 查看flannel配置参数
kubectl get configmap kube-flannel-cfg -n kube-system -o json
```

<br>

设置flannel配置，设置在同一个网段使用host-gw，不在同一个网段自动降级为VxLAN。

```bash
# 下载网络插件flannel
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 打开配置文件修改net-conf.json，添加"Directrouting": true，注意前面需要添加逗号
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "Directrouting": true
      }
    }

# 重新启动使配置生效
kubectl -f delete kube-flannel.yml
kubectl apply -f delete kube-flannel.yml

# 新建一个Pod，然后查看路由，如果是直接从网络端口出去，说明配置生效
ip route show

# 注意：flannel配置应该是安装集群的时候就规划好的，如果中途去修改，会影响已经运行的Pod。
```

#### 13.2 网络策略canal

canal使用网络插件flannel和calico的结合体，通信使用flannel，通信策略使用calico控制，例如有dev名称空间和prod名称空间，dev名称空间和prod名称空间下的Pod的之间默认是可以直接通信的，如果要隔离两个名称空间下Pod通信，使用calico的通信策略来控制，网络控制策略可以控制Pod的网络的流入和流出两个维度，如下图所示：

![网络控制策略](https://pic.zhuyasen.com/815193_14_%E7%BD%91%E7%BB%9C%E6%8E%A7%E5%88%B6%E7%AD%96%E7%95%A5.jpg)

<br>

安装网络插件canal，官网地址https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/flannel

```bash
# rbac认证、授权
kubectl apply -f https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/hosted/canal/rbac.yaml

# 安装canal，需要拉取镜像，需要等待一段时间。
kubectl apply -f https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/hosted/canal/canal.yaml

# 查看canal资源
kubectl get pod -n kube-system | grep canal
```

<br>

首先创建两个新的名称空间dev和prod，用来测试网络策略控制：

```bash
kubectl create namespace dev
kubectl run dev-myapp --image=ikubernetes/myapp:v1 --replicas=2 -n dev

kubectl create namespace prod
kubectl run prod-myapp --image=ikubernetes/myapp:v2 --replicas=2 -n prod

# 默认情况下是没有限制dev和prod名称空间下Pod通信的，都可相互ping通
```

<br>

**创建networkPolicy控制策略**

(1) 禁止一切外部访问，但可以出栈访问

创建禁止访问的NetworkPolicy资源配置文件forbidden-access.yml，内容如下：

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: forbidden-ingress
  # 管理dev名称空间下pod的网络
  namespace: dev
spec:
  podSelector: {}
  # 明确指定管理入栈网络，如果入栈网络不指明规则，表示禁止所有入栈网络，如果指明ingress为空，表示允许所有入栈访问
  policyTypes: ["Ingress"]
```

<br>

测试是否能ping通：

```bash
kubectl exec -it dev-myapp-7f4b4954b4-4b6l7 -n dev -- ping 10.244.2.2
kubectl exec -it prod-myapp-9b6bc5657-ffqsz -n prod -- ping 10.244.2.2
ping 10.244.2.2

# 禁止一切外部访问，并且连同一个名称空间的其他pod也是禁止了访问。
```

<br>

(2) 允许特定标签访问

给ip地址为10.244.2.2添加标签：

> kubectl label pod dev-myapp-7f4b4954b4-4b6l7 -n dev app=myapp

<br>

创建允许特定标签访问NetworkPolicy资源清单文件selector-access.yml，内容如下：

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: selector-ingress
  # 管理dev名称空间下pod的网络
  namespace: dev
spec:
  # 管理dev下指定的标签pod的网络
  podSelector:
    matchLabels:
      app: myapp

  ingress:
  - from:
    # ip地址块
    - ipBlock:
        cidr: 10.244.0.0/16
    # 只允许访问的端口
    # ports:
    # - port: 80
```

测试是否能ping通

```bash
kubectl exec -it dev-myapp-7f4b4954b4-4b6l7 -n dev -- ping 10.244.2.2
kubectl exec -it prod-myapp-9b6bc5657-ffqsz -n prod -- ping 10.244.1.2
curl 10.244.2.2

# 10.244.2.2可以访问，而10.244.1.2还是禁止访问的
```

<br><br>

### 14 调度器

当用户需要创建一个Pod时，Pod到底运行在哪个node节点上呢，这是有调度器来完成的，调度器经过预选(Predicate) --> 优选(Priority) --> 选择结果(随机Select)这三个阶段选择出一个节点来运行Pod。

<br>

#### 14.1 预选策略

从所有节点中选出符合条件的节点，符合条件策略有：

| 策略名称 | 说明 |
| :------ | :------ |
| GeneralPredicates | HostName：检查Pod对象是否定义了pod.spec.hostname；PodFitsHostPorts：pods.spec.containers.ports.hostPort；MatchNodeSelector：pods.spec.nodeSelector；PodFitsResources：检查Pod的资源需求是否能被节点所满足 |
| CheckNodeCondition |  |
| NoDiskConflict | 检查Pod依赖的存储卷是否能满足需求 |
| PodToleratesNodeTaints | 检查Pod上的spec.tolerations可容忍的污点是否完全包含节点上的污点 |
| PodToleratesNodeNoExecuteTaints |  |
| PodToleratesNodeNoExecuteTaints |  |
| CheckNodeLabelPresence |  |
| CheckServiceAffinity |  |
| MaxEBSVolumeCount |  |
| MaxGCEPDVolumeCount |  |
| MaxAzureDiskVolumeCount |  |
| CheckVolumeBinding |  |
| NoVolumeZoneConflict |  |
| CheckNodeMemoryPressure |  |
| CheckNodePIDPressure |  |
| CheckNodeDiskPressure |  |
| MatchInterPodAffinity |  |

<br>

#### 14.2 优先函数

在符合预选条件的节点当中使用优选函数给这些节点打分排序。优先函数有：

| 优先函数 | 说明 |
| :------ | :------ |
| LeastRequested | (cpu((capacity-sum(requested))*10/capacity)+memory((capacity-sum(requested))*10/capacity))/2 |
| BalancedResourceAllocation |  |
| NodePreferAvoidPods | 节点注解信息"scheduler.alpha.kubernetes.io/preferAvoidPods" |
| TaintToleration | 将Pod对象的spec.tolerations列表项与节点的taints列表项进行匹配度检查，匹配条目越多，得分越低 |
| ImageLocality | 根据满足当前Pod对象需求的已有镜像的体积大小之和 |
| SeletorSpreading |  |
| InterPodAffinity |  |
| NodeAffinity |  |
| MostRequested |  |
| NodeLabel |  |

<br>

#### 14.3 高级调度设置

在某些调度场景中，期望做一些自己的预设，去影响调度器的调度方式，例如让Pod执行在特定的节点中，可以设置节点选择器(nodeSelector、nodeName)或节点亲和调度(nodeAffinity)去影响预选阶段中节点的预选。

(1) 通过nodeSelector选择器让pod运行在特定节点

```bash
# 给node02添加标签disktype=ssd
kubectl label node node02 disktype=ssd
```

<br>

创建pod资源清单文件schedule-nodeSelector.yml，pod只运行在带有标签disktype=ssd的节点上，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: mynginx

spec:
  containers:
  - name: nginx
    image: nginx:1.15.2
  nodeSelector:
    disktype: ssd
```

pod会运行在标签为disktype=ssd的node02，删除pod和重启都一样。注意：nodeSelector是硬性要求，如果pod根据标签找不到node，pod一直在pending状态。

<br>

(2) 通过affinity亲和调度让两个pod运行在附近的节点上

创建pod资源清单文件schedule-affinity.yml，两个pod运行附近的节点上，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: mynginx
spec:
  containers:
  - name: nginx
    image: nginx:1.15.2
  nodeSelector:
    disktype: ssd

---

apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: mybusybox
spec:
  containers:
  - name: busybox
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c", "while true; do echo $(date); sleep 3; done"]
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          # 亲和pod的标签
          matchExpressions:
          - {key: app, operator: In, values: ["mynginx"]}
        # 亲和依据key
        topologyKey: disktype
```

<br>

(3) 通过污点和容忍度让pod运行在特定节点上

taint的effect定义对Pod排斥效果：

| 名称 | 排斥效果 |
| :------ | :------ |
| NoSchedule | 仅影响调度过程，对现存的Pod对象不产生影响 |
| NoExecute | 既影响调度过程，也影响现在的Pod对象；不容忍的Pod对象将被驱逐 |
| PreferNoSchedule | NoSchedule的柔性版本，最好别调度过来，实在没地方运行调过来也行 |

<br>

在node01和node02添加污点，排斥效果为NoSchedule

```bash
# 添加污点
kubectl taint node node01 node-type=production:NoSchedule
kubectl taint node node02 node-type=development:NoSchedule

# 删除污点
kubectl taint nodes node01 node-type-
kubectl taint nodes node02 node-type-
```

创建pod资源清单文件schedule-taint.yml，让pod运行在node01上，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: mynginx

spec:
  containers:
  - name: nginx
    image: nginx:1.15.2

  # 容忍污点
  tolerations:
  - key: node-type
    value: production
    operator: Equal
    effect: NoSchedule
```

如果要Pod能运行在节点node02上，配置时必须容忍node02的node-type=development污点

<br><br>

### 15 kubernetes监控

1.11.1版本之前的kubernetes监控使用插件heapster，heapster提供了指标数据的采集、存储和监控等基本功能，并支持多个第三方存储后端(例如influxdb)，依赖第三方插件的弊端是假如有一天第三方插件的组织更新了不兼容的版本或者不维护，甚至丢弃该项目了，驻留在heapster中第三方驱动代码也要做相应变更，而且每适配一个第三方插件都要驻留一个插件驱动代码，使得heapster非常庞大，使得维护heapster越来越麻烦，代价也越来越高，kubernetes官方从1.11.1版本就开始移除heapster，使用新一代监控插件metrics-server。

<br>

#### 15.1 新一代kubernetes监控架构

(1) 核心指标流水线

核心指标流水线由kubelet、metrics-server以及由API server提供的api组成。主要核心指标有：CPU累积使用率、内存实时使用率、Pod的资源占用率及容器的磁盘占用率。metrics-server是托管在Pod的第三方服务，只用于收集主要核心指标数据。而metrics-server并不是kubernetes的组成部分，而且有自己的api接口，要想让kubernetes像使用本地api接口一样无缝的获取核心指标数据，需要把kubernetes的api接口和metrics-server的api接口聚合在一起，这个聚合器名称叫kube-aggregator。

<br>

(2) 监控流水线

监控流水线用于从系统收集各种指标数据并提供终端用户、存储系统以及HPA，它们包含核心指标及许多非核心指标。prometheus既是一个监控系统，也是特殊资源指标的提供者。因为prometheus不是kubernetes内建的标准核心指标，如果promethuse想把资源指标数据提供给kubernetes解析，需要经过k8s-prometheus-adapter插件数据转换。

<br>

#### 15.2 安装metrics-server

**(1) 下载metrics-server配置文件**

根据kubernetes版本选择源码，复制kubernetes/cluster/addons/metrics-server配置文件出来，由于kubernetes集群默认获取api接口kubernetes.summary_api使用https(端口10250)，禁用了http(端口10255)，而metrics-server的默认配置文件默认使用http(端口10255)，所以需要修改为https才能正常通信。

```bash
# 修改metrics-server-deployment.yaml配置文件
把--source=kubernetes.summary_api:''改为
--source=kubernetes.summary_api:https://kubernetes.default?kubeletHttps=true&kubeletPort=10250&insecure=true

# 修改resource-reader.yaml配置文件
在resource下添加资源类型nodes/stats，如下所示：
  resources:
  - pods
  - nodes
  - nodes/stats
  - namespaces
```

<br>

**(2) 安装metrics-server**

```bash
# 如果node节点不能翻墙，无法正常启动相关pod，原因是下载镜像失败，解决办法是设置docker代理翻墙或预先下载导入镜像(具体镜像版本查看metrics-server-deployment.yaml文件)

# 安装
cd kubernetes/cluster/addons/metrics-server
kubectl apply -f ./

# 查看kubernetes聚合的metrics-server的群组是否存在
kubectl api-versions | grep metrics

# 查看节点核心指标
kubectl top node
  # NAME      CPU(cores)   CPU%      MEMORY(bytes)   MEMORY%
  # master    136m         6%        1048Mi          75%
  # node01    54m          2%        553Mi           40%
  # node02    61m          3%        714Mi           51%

# 查看pod核心指标
kubectl top pod
```

<br>

#### 15.3 安装prometheus和相关附件

prometheus是有状态应用，默认安装使用statefulSet部署，这里为了部署简单，改为Deployment部署，实际生产中要部署为statefulSet，要准备存储卷pv。

![prometheus模块结构图](https://pic.zhuyasen.com/815193_16_prometheus%E6%A8%A1%E5%9D%97%E7%BB%93%E6%9E%84%E5%9B%BE.jpg)

```bash
# 下载
git clone https://github.com/iKubernetes/k8s-prom.git
cd k8s-prom

# 创建一个独立名称空间prom
kubectl apply -f namespace.yaml

# 安装node_exporter，node_exporter将采集到系统和节点指标数据传给prometheus
kubectl apply -f node_exporter/

# 安装prometheus
kubectl apply -f prometheus/
# 在浏览器打开http://192.168.8.90:30090，测试能否获取数据

# 安装kube-state-metrics，prometheus通过kube-state-metrics，把PromQL查询语句转为kubernetes的service API识别语句
# 需要翻墙下载镜像
kubectl apply -f kube-state-metrics/

# 安装k8s-prometheus-adapter，经过kube-state-metrics转换后的查询语句后再通过k8s-prometheus-adapter获取指标数据，并把api聚合到kubernetes上
  # 因为kubernetes集群默认是使用https通信，所以需要生成自签证书
  # 打开新的终端，切换到集群证书目录
  cd /etc/kubernetes/pki
  # 生成私钥serving.key
  openssl genrsa -out serving.key 2048
  # 生成证书请求文件serving.csr
  openssl req -new -key serving.key -out serving.csr -subj "/CN=serving"
  # 生成证书文件serving.crt
  openssl x509 -req -in serving.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out serving.crt -days 3650
  # 把证书文件移动到k8s-prometheus-adapter文件夹下
  mv serving.* /home/vison/work/kubernetes/k8s-promethues/k8s-prometheus-adapter/
  # 创建secret资源，名称固定为cm-adapter-serving-certs(具体名称从文件k8s-prometheus-adapter/custom-metrics-apiserver-deployment.yaml中获取)
  kubectl create secret generic cm-adapter-serving-certs --from-file=serving.crt=./k8s-prometheus-adapter/serving.crt  --from-file=serving.key=./k8s-prometheus-adapter/serving.key -n prom
  # 查看secret资源
  kubectl get secrets -n prom

  # 安装k8s-prometheus-adapter，注意k8s-prometheus-adapter镜像版本和github版本要一致
  kubectl apply -f k8s-prometheus-adapter/

  # 查看pod是否正常运行
  kubectl get pod -n prom

  # 查看群组是否存在
  kubectl api-versions | grep custom.metrics.k8s.io

# 安装grafana，service使用NodePort类型，端口30808
kubectl apply -f grafana/
```

<br>

在浏览器打开http://192.168.8.90:30808访问grafana，进入界面后设置数据源，如下图所示：

![grafana配置数据源](https://pic.zhuyasen.com/815193_16_grafana%E9%85%8D%E7%BD%AE%E6%95%B0%E6%8D%AE%E6%BA%90prometheus.png)

<br>

然后去[grafana官网](https://grafana.com/dashboards)下载prometheus相关面板(例如 https://grafana.com/dashboards/315)  ，最后导入面板模板，界面如下图所示：

![grafana界面](https://pic.zhuyasen.com/815193_17_grafana%E7%95%8C%E9%9D%A2.png)

<br><br>

### 16 helm

helm的功能类似centos下yum，helm使用Chart帮助我们管理kubernetes应用，主要概念：

| 名称 | 说明 |
| :------ | :------ |
| Chart | 一个helm程序包，程序包结构相对固定的目录或者tgz压缩文件，Chart之间可相互依赖。 |
| Release | 特定的Chart部署于目标集群上的一个实例。 |
| Repository | Charts仓库，https或http服务器。 |
| helm | 客户端，管理本地的Chart仓库，管理Chart, 与Tiller服务器交互，发送Chart，实例安装、查询、卸载等。 |
| Tiller | 服务端，接收helm发来的Charts与Config，合并生成relase。 |

helm部署应用流程，官方下载或自制Chart -> 修改默认配置参数(Config) -> 部署到kubernetes集群(Release)

<br>

#### 16.1 安装helm和tiller

```bash
# 下载安装包
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz

# 把helm移动到/usr/bin
tar zxvf helm-v2.9.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/bin/

# 在kubernetes集群部署helm的服务端Tiller
  # (1) Tiller需要管理员权限才能对kubernetes应用增删改查，需要创建一个角色tiller并绑定到cluster-admin，从官网文档复制rbac配置https://github.com/helm/helm/blob/master/docs/rbac.md

  kubectl apply -f rbac-config.yml

  # (2) 部署Tiller时，helm需要通过apiserver认证授权，因此helm获取kubectl的config配置文件，扮演kubectl角色获取kubernetes管理员权限，helm初始化，需要翻墙获取镜像。

  helm init --service-account tiller
  
# 查看tiller是否启动正常
kubectl get pod -n kube-system | grep tiller

# 查看版本
helm version
  # Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
  # Server: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
```

<br>

#### 16.2 helm常用命令

官方可用的Chart列表：https://hub.kubeapps.com

```bash
# 查看chart仓库，一个本地仓库和一个官方仓库
helm repo list
  # NAME    URL
  # stable  https://kubernetes-charts.storage.googleapis.com
  # local   http://127.0.0.1:8879/charts

# 搜索仓库
helm search <应用名称>
```

<br>

chart管理：

```bash
# 创建chart
helm create

# 下载chart到当前目录
helm fetch

# 下载release
helm get

# 查看chart详细信息
helm inspect

# 打包chart
helm package

# chart校验
helm verify

# 添加仓库
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com
```

<br>

release管理命令：

```bash
# 安装应用，下载chart到目录.helm/cache/archive/
helm install

# 删除应用
helm delete

# 更新/回滚应用
helm upgrade/rollback

# 列出已安装的应用
helm list

# release的历史信息
helm history

# 获取release状态信息
helm status
```

<br>

#### 16.3 使用helm安装EFK

**(1) 安装elasticsearch**

```bash
# 创建名称空间efk
kubectl create ns efk

# 下载elasticsearch
helm fetch incubator/elasticsearch

# 修改默认值
tar xf elasticsearch-1.8.2.tgz
vim elasticsearch/values.yaml
# 禁用persistence存储，把enabled改为false

# 安装elasticsearch，需要翻墙
helm install --name es1 --namespace=efk -f elasticsearch/values.yaml incubator/elasticsearch --version 1.8.2

# 获取elasticsearch域名
helm status es1

# 启动一个pod测试elasticsearch是否运行正常
kubectl run cirros01 --rm -it --image=cirros -- /bin/sh
  # 解析域名
  nslookup es1-elasticsearch-client.efk.svc.cluster.local
  # 查询
  curl es1-elasticsearch-client.efk.svc.cluster.local:9200/_cat

# 删除elasticsearch
helm delete es1 --purge
```

<br>

**(2) 安装fluentd**

```bash
# 下载fluentd-elasticsearch
helm fetch stable/fluentd-elasticsearch

# 修改默认值
tar xf fluentd-elasticsearch-1.0.3.tgz
vim fluentd-elasticsearch/values.yaml
# 修改elasticsearch的host值为es1-elasticsearch-client.efk.svc.cluster.local

# 安装fluentd
helm install --name fluentd1 --namespace=efk -f fluentd-elasticsearch/values.yaml stable/fluentd-elasticsearch

# 删除fluentd
helm delete fluentd1 --purge
```

<br>

**(3) 安装kibana**

```bash
# 下载kibana，注意下载kibana版本要和elasticsearch版本一致
helm fetch stable/kibana

# 修改默认值
tar xf kibana-0.14.7.tgz
vim kibana/values.yaml
# 修改elasticsearch.url值为http://es1-elasticsearch-client.efk.svc.cluster.local:9200
# 修改service的type值为NodePort

# 安装kibana
helm install --name kibana1 --namespace=efk -f kibana/values.yaml stable/kibana

# 删除fluentd
helm delete kibana1 --purge
```

在浏览器打开 http://192.168.8.90:31122 进入kibana界面，点击菜单management，添加索引logstash-*，选择时间序列，就可以搜索按时间顺序的日志。

<br><br>

### 17 kubernetes集群问题排查

```bash
# (1) 查看Pod是否在running状态
kubectl get pod <Pod名称> -n <名称空间>

# (2) 如果Pod在非running状态，查看Pod系统Event事件，确认Pod启动失败原因
kubectl describe pod <Pod名称> -n <名称空间>
  # 一般原因有几种：
  # 正在下载镜像或镜像下载失败。
  # 没有Node节点可调度，导致一直在pending状态。
  # 有Node节点，但Node资源(例如内存、cpu)紧缺，导致一直在pending状态。
  # 有Node节点，集群资源充足，Pod没有找到相应的持久化存储pv或Pod无法容忍Node的污点，导致一直在pending状态。

# (3) 如果Pod在running状态，但程序功能异常，查看Pod日志或指定容器日志
  # 查看指定pod的日志
  kubectl logs <pod名称>
  kubectl logs -f <pod名称>
  # 查看指定pod中指定容器的日志
  kubectl logs <pod名称> -c <容器名称>

  # 示例：
  kubectl logs --tail=100
  kubectl logs --since=1h nginx
```

<br>

常见问题和解决办法：

- Pod状态为Pending、failed、Unknown、pullImageFailed。
  - 解决办法：查看Pod启动事件，确认其原因。
- Pod创建后不断重启。
  - 原因：容器程序异常退出或容器启动命令不是阻塞式命令。
  - 解决办法：查看Pod的日志或容器日志，确认其原因。
