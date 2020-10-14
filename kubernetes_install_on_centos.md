# 前提：
1、操作系统Centos 8
```
[root@hanyu ~]# uname -a
Linux hanyu 4.18.0-147.el8.x86_64 #1 SMP Wed Dec 4 21:51:45 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
[root@hanyu ~]# 
[root@hanyu ~]# cat /etc/redhat-release 
CentOS Linux release 8.1.1911 (Core
```
2、通过阿里云相关镜像源安装

**以下均为root用户下操作**

# 操作步骤：
## 1、修改主机名
```
hostnamectl set-hostname hanyu
```

## 2、关闭防火墙
```
systemctl stop firewalld.service
systemctl disable firewalld.service
```

## 3、安装docker
**安装必要的工具**
```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

**配置官方docker源或者aliyun docker源**
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
或者
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

**查询docker版本（否则yum install默认安装最新的版本）**
```
yum list docker-ce --showduplicates | sort -r
```

**安装指定版本docker**
```
yum -y install docker-ce-18.09.3-3.el7
```
如果提示containerd.io版本过低，解决方法：

```
wget https://download.docker.com/linux/centos/7/x86_64/edge/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm

yum install -y  containerd.io-1.2.6-3.3.el7.x86_64.rpm
```

**启动docker**
```
systemctl enable docker && systemctl start docker
```

**查询docker服务状态**
```
systemctl status docker
```

**查看docker版本**
```
docker version
```

## 4、安装kubelet、kubeadm 和 kubectl
**配置kubernetes.repo的源，由于官方源国内无法访问，这里使用阿里云yum源**

```
[root@hanyu ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

**在所有节点上安装指定版本 kubelet、kubeadm 和 kubectl**
```
yum list kubectl --showduplicates | sort -r
yum list kubeadm --showduplicates | sort -r
yum list kubelet --showduplicates | sort -r
yum install -y kubectl-1.18.2.-0 kubeadm-1.18.2-0 kubelet-1.18.2-0
```

**启动kubelet服务**
```
systemctl enable kubelet && systemctl start kubelet 
```

## 5、关闭swap分区
```
swapoff -a
```

## 6、初始化master节点

```
kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.2 --pod-network-cidr=10.244.0.0/16 --token-ttl 0 --ignore-preflight-errors=Swap
```

```
[root@hanyu ~]# kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.2 --pod-network-cidr=10.244.0.0/16 --token-ttl 0
W0506 22:23:59.530662    3903 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.2
[preflight] Running pre-flight checks
	[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
	[WARNING Hostname]: hostname "hanyu" could not be reached
	[WARNING Hostname]: hostname "hanyu": lookup hanyu on 10.255.9.2:53: no such host
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [hanyu kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.104.125]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [hanyu localhost] and IPs [10.0.104.125 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [hanyu localhost] and IPs [10.0.104.125 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0506 22:24:31.410049    3903 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0506 22:24:31.410678    3903 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 21.505955 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node hanyu as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node hanyu as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 9o697n.syq87aguxtdmcn76
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.104.125:6443 --token 9o697n.syq87aguxtdmcn76 \
    --discovery-token-ca-cert-hash sha256:294e07d4f7f09e4c99ef96a8563546e002672b8535ee7bc339e423d5cf38aa4c
```

按照kubeadm init成功后打印提示，继续操作：
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

kubectl get nodes查询到节点处于NotReady状态，是因为网络插件还未就位，也就是这里要求运行的
```
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
https://kubernetes.io/docs/concepts/cluster-administration/addons/
```

如果要暂时忽略让节点Ready，将如下文件的--network-plugin=cni字段去掉, (该文件是在kubeadm init或者kubeadm join过程中生成的), 修改完后重启kubelet即可(systemctl restart kubelet)

```
[root@hanyu ~]# cat /var/lib/kubelet/kubeadm-flags.env
KUBELET_KUBEADM_ARGS="--cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2"
```

此处安装flannel后（kubectl apply -f kube-flannel.yaml），node变为ready，

## 7、join node节点
```
[root@hanyu2 ~]# kubeadm join 10.0.104.125:6443 --token 9o697n.syq87aguxtdmcn76 --discovery-token-ca-cert-hash sha256:294e07d4f7f09e4c99ef96a8563546e002672b8535ee7bc339e423d5cf38aa4c
[preflight] Running pre-flight checks
[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
This node has joined the cluster:
  Certificate signing request was sent to apiserver and a response was received.
  The Kubelet was informed of the new secure connection details.
Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

## 查询所有node及pod状态
```
kubectl get node -o wide
kubectl get pod --all-namespaces -o wide
```

## 增加kubectl命令自动补全功能
```
source <(kubectl completion bash)
```

注意：有时需要额外安装bash-completion包

## 默认master节点不会调度pod，去掉此限制(可选做)
``` 
kubectl taint node hanyu node-role.kubernetes.io/master:NoSchedule-
```
hanyu为主机名

## node节点ROLES默认显示none，将其修改为显示worker(可选做)
```
kubectl label node hanyu2 node-role.kubernetes.io/worker= --overwrite
kubectl label node hanyu3 node-role.kubernetes.io/worker= --overwrite
[root@hanyu ~]# kubectl get node
NAME    STATUS   ROLES    AGE   VERSION
hanyu   Ready    master   23h   v1.18.2
hanyu2   Ready    worker   23h   v1.18.2
hanyu3   Ready    worker   23h   v1.18.2
```
