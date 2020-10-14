# 前提：
1、操作系统Ubuntu 19.10
```
root@ubuntu-001:~# uname -a
Linux ubuntu-001 5.3.0-51-generic #44-Ubuntu SMP Wed Apr 22 21:09:44 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
root@ubuntu-001:~# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 19.10
Release:	19.10
Codename:	eoan
```
2、通过阿里云相关镜像源安装


**以下均为root用户下操作**

# 操作步骤：
## 1、修改主机名
```
hostnamectl set-hostname ubuntu-001
```

## 2、关闭防火墙
```
apt-get install ufw
ufw disable
```

## 3、安装docker
**安装必要的工具及GPG证书**
```
apt-get update
apt-get -y install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

**配置阿里云docker源**
```
add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

```

**查询docker版本（否则默认安装最新的版本）**
```
apt-cache madison docker-ce
#   docker-ce | 5:19.03.8~3-0~ubuntu-eoan | https://mirrors.aliyun.com/docker-ce/linux/ubuntu eoan/stable amd64 Packages
#   docker-ce | 5:19.03.7~3-0~ubuntu-eoan | https://mirrors.aliyun.com/docker-ce/linux/ubuntu eoan/stable amd64 Packages
#   docker-ce | 5:19.03.6~3-0~ubuntu-eoan | https://mirrors.aliyun.com/docker-ce/linux/ubuntu eoan/stable amd64 Packages
```

**安装指定版本docker**
```
apt-get -y update
apt-get -y install docker-ce=5:19.03.8~3-0~ubuntu-eoan
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
**配置kubernetes.repo的源，由于官方源国内无法访问，这里使用阿里云源**

```
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  

```

**在所有节点上安装指定版本 kubelet、kubeadm 和 kubectl**
```
apt-get update
apt-cache madison kubectl
apt-cache madison kubeadm
apt-cache madison kubelet
apt-get install -y kubelet=1.18.0-00 kubeadm=1.18.0-00 kubectl=1.18.0-00

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
kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --pod-network-cidr=10.244.0.0/16 --token-ttl 0 --ignore-preflight-errors=Swap
```

```
root@ubuntu-001:~# kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --pod-network-cidr=10.244.0.0/16 --token-ttl 0 --ignore-preflight-errors=Swap
W0507 17:11:30.761887   14961 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.0
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [ubuntu-001 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.105.107]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [ubuntu-001 localhost] and IPs [10.0.105.107 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [ubuntu-001 localhost] and IPs [10.0.105.107 127.0.0.1 ::1]
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
W0507 17:11:34.680144   14961 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0507 17:11:34.682546   14961 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 20.502602 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node ubuntu-001 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node ubuntu-001 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 10y2lc.v55p1f47j3lp15gg
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

kubeadm join 10.0.105.107:6443 --token 10y2lc.v55p1f47j3lp15gg \
    --discovery-token-ca-cert-hash sha256:1093b027cf31d755dcaa7109ba890962c140e2d6e9c46bf4207a0c519ce7bf36 

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
root@ubuntu-001:~# cat /var/lib/kubelet/kubeadm-flags.env
KUBELET_KUBEADM_ARGS="--cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.2 --resolv-conf=/run/systemd/resolve/resolv.conf"
```
安装flannel，kubectl apply -f kube-flannel.yaml

## 7、join node节点
```
root@ubuntu-002:~# kubeadm join 10.0.105.107:6443 --token 10y2lc.v55p1f47j3lp15gg --discovery-token-ca-cert-hash sha256:1093b027cf31d755dcaa7109ba890962c140e2d6e9c46bf4207a0c519ce7bf36
W0507 17:16:18.524687   26000 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

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

## 默认master节点不会调度pod，去掉此限制(可选做)
``` 
kubectl taint node ubuntu-001 node-role.kubernetes.io/master:NoSchedule-
```
ubuntu-001为主机名

## node节点ROLES默认显示none，将其修改为显示worker(可选做)
```
kubectl label node ubuntu-002 node-role.kubernetes.io/worker= --overwrite
kubectl label node ubuntu-003 node-role.kubernetes.io/worker= --overwrite
root@ubuntu-001:~# kubectl get node -o wide
NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION     CONTAINER-RUNTIME
ubuntu-001   Ready    master   16h   v1.18.0   10.0.105.107   <none>        Ubuntu 19.10   5.3.0-51-generic   docker://19.3.8
ubuntu-002   Ready    worker   16h   v1.18.0   10.0.105.21    <none>        Ubuntu 19.10   5.3.0-46-generic   docker://19.3.8
ubuntu-003   Ready    worker   61m   v1.18.0   10.0.105.62    <none>        Ubuntu 19.10   5.3.0-46-generic   docker://19.3.8
```
