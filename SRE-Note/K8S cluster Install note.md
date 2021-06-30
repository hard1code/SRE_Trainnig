# 05/27 Kublet K8s系統架構流程

### 關閉 swap
```
$ sudo swapoff -a
```


```
swap加上註解
$ sudo nano /etc/fstab
#/swap.img       none    swap    sw      0       0
```

當電腦記憶體RAM不夠用的時候,就會去使用局部硬碟空間HD作為RAM
網管人員通常會關閉
fstab=firesystem table
當pod記憶體不足的時候K8s會自己把pod移動到負載低的pod(電腦上)執行 所以不用swap

```
$ sudo nano /etc/sysctl.conf
vm.swappiness = 0(這個不用做)
net.ipv4.ip_forward = 1

$ sudo sysctl -p
```




### 安裝 cri-o
```
$ sudo nano /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /
```
- 把cri-o網址加入apt指令去 
- ubuntu不知道這個套件
```
sudo nano /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:1.20.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.20/xUbuntu_20.04/ /
```
- 上面打1.20版本後面裝k8s的時候也要裝這個版本的cri-o



```
$ curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:1.20/xUbuntu_20.04/Release.key | sudo  apt-key add -
$ curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key | sudo apt-key add -
```

- 透過公私鑰下載


```
$ sudo apt-get update
$ sudo apt-get install cri-o cri-o-runc -y
$ sudo sed -i 's/10.85.0.0\/16/10.244.0.0\/16/g' /etc/cni/net.d/100-crio-bridge.conf
```
- 修改pod ip位置範圍把10.85.0.0 改成 10.244.0.0


```
$ sudo systemctl daemon-reload
$ sudo systemctl status crio
● crio.service - Container Runtime Interface for OCI (CRI-O)
     Loaded: loaded (/lib/systemd/system/crio.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: https://github.com/cri-o/cri-o
$ sudo systemctl start crio
$ sudo systemctl enable crio
```




### 查看 crio 版本
`$ crio version`
INFO[0000] Starting CRI-O, version: 1.20.2, git: d5a999ad0a35d895ded554e1e18c142075501a98(clean) 
Version:       1.20.2
GitCommit:     d5a999ad0a35d895ded554e1e18c142075501a98
GitTreeState:  clean
BuildDate:     2021-04-30T13:47:16Z
GoVersion:     go1.15.2
Compiler:      gc
Platform:      linux/amd64
Linkmode:      dynamic

```
$ sudo reboot
```



# 安裝k8s
```
$ sudo nano /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
```
```
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
```
$ sudo apt update
$ sudo apt-cache madison kubeadm | grep 1.20.2
```
- kubeadm |  1.20.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
- 看套件清單是否有包含1.20k8s
```
$ export K_VER=1.20.2-00
$ sudo apt install -y kubelet=${K_VER} kubectl=${K_VER} kubeadm=${K_VER}
#K_VER=版本代號,指定1.20.2-00版本
$ sudo apt-mark hold kubelet kubeadm kubectl
#宣告kubelet kubeadm kubectl 版本鎖死再1.20.2
```
```
$ sudo nano /etc/default/kubelet
KUBELET_EXTRA_ARGS=--feature-gates="AllAlpha=false,RunAsGroup=true" --container runtime=remote --cgroup-driver=systemd --
container-runtime-endpoint='unix:///var/run/crio/crio.sock' --runtime-request-timeout=5m
```
- 要求k8s等等產生pod要使用cri-o sockt
```
$ sudo systemctl daemon-reload
$ sudo systemctl enable kubelet
```
```
$ sudo kubeadm config images pull --kubernetes-version=v1.20.2
#下載k8s的主要images k8s是由pod組成的
```

* [config/images] Pulled k8s.gcr.io/kube-apiserver:v1.20.2
* [config/images] Pulled k8s.gcr.io/kube-controller-manager:v1.20.2
* [config/images] Pulled k8s.gcr.io/kube-scheduler:v1.20.2
* [config/images] Pulled k8s.gcr.io/kube-proxy:v1.20.2
* [config/images] Pulled k8s.gcr.io/pause:3.2
* [config/images] Pulled k8s.gcr.io/etcd:3.4.13-0
* [config/images] Pulled k8s.gcr.io/coredns:1.7.0

```
$ sudo modprobe br_netfilter
```
- 把kernel的網路過濾器模組啟動 針對CNI部分
```
$ sudo reboot
```
以上要在master,worker node上執行


---

## K8s master

只有Control Plane(m1)要做，
```
$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=/var/run/crio/crio.sock
```
- 建立一個使用CIDR規格切出來的網段/重開後要重執行上面命令
- sudo modprobe br_netfilter
- 或是執行把kernel的網路過濾器模組啟動 針對CNI部分
```
$ sudo nano /etc/modules
加上 br_netfilter
```


## :warning: 重要:warning: 
### [copy cluster token to home]
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
- 把K8S的admin 複製到家目錄.kube/config (針對要改成管理者的使用者名稱的家目錄)
```
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- 改變.kube資料夾的擁有者與群組
```
$ kubectl get pods -n kube-system
```
- 看有哪些pod....-n為namespace


---


### setup kube-router設定CNI標準
```
$ wget https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter-all-features.yaml
```
- k8s是要用pod系統幫企業維運,結果他自己本身就是用pod製作

```
$ sudo sed -i 's/rbac.authorization.k8s.io\/v1beta1/rbac.authorization.k8s.io\/v1/g' kubeadm-kuberouter-all-features.yaml
```
```
$ kubectl create -f kubeadm-kuberouter-all-features.yaml
```
把kuberouter建立起來

```
$ kubectl get pod -n kube-system
查看k8s系統pod執行狀態,可設置成快速指令
alias ks='kubectl get pod -n kube-system'
```
### :warning:可使用 flannel 代替kuberouter
```
$ kubeadm init --pod-network-cidr=10.244.0.0/16
$ kubectl create -f https://raw.githubusercontent.com/coreos/flannel/v0.12.0/Documentation/kube-flannel.yml
```
- 安裝Flannel(k8s底層靜態路由技術)並從google雲下載flannel的images
---
```
$ kubectl taint node 120-96-143-106-m1 node-role.kubernetes.io/master:NoSchedule-
```
- master這台主機等等可以執行pod, 內定master不能跑pod只有worker可以,這行指令要突破這個限制

#### :exclamation:測試k8s是否成功
```
產生pod指令
$ kubectl run a1 --image-pull-policy Never --image=dafu/alpine.derby --port=8888
產生pod,使用老師的images ...--image-pull-policy Never不行上網抓
```

```
刪除pod指令
$ kubectl delete pods a1
--------pod "a1" deleted

查看pod狀態指令
$ kubectl get pods a1

NAME   READY   STATUS    RESTARTS   AGE
a1     1/1     Running   0          5m48s
查看詳細pod所有資訊參數 -o wide
$ kubectl get pods a1 -o wide

NAME   READY   STATUS    RESTARTS   AGE    IP           NODE                NOMINATED NODE   READINESS GATES
a1     1/1     Running   0          3m7s   10.244.0.6   120-96-143-106-m1   <none>           <none>
-----------------------------------------------
這個a1 Pod 是由m1這台主機所建立的 IP是自己CIDR出來的網段內的IP :10.244.0.6
------------------------------------------------
$ curl http://10.244.0.6:8888

<h1>Welcome to Spring Boot</h1>
```

### :warning: 把工作機worker node加入k8s叢集:warning: 
### [join master]
```
$ sudo modprobe br_netfilter
$ echo " sudo `kubeadm token create --print-join-command 2>/dev/null`"
--------------------------------------------------------
這行命令一定要在m1做,token產生一個系統密碼還有一個摘要代號,這些要給要加入這個群組的worker工作機上,形成一個叢集cluster(群組)
--------------------------------------------------------
 sudo kubeadm join 120.96.143.106:6443 --token rp59g0.ud032a71s5fsxnwm     --discovery-token-ca-cert-hash sha256
:6f6993925d1b891c26af8087875cb49f3fe29c5ae654665a6202a704ecad6a94
------------------------------把上面噴出的代碼整理成一行貼到W1,W2
```
------------------------------------------------------------------------------------------------------------
sudo kubeadm join 120.96.143.106:6443 --token rp59g0.ud032a71s5fsxnwm --discovery-token-ca-cert-hash 

sha256:6f6993925d1b891c26af8087875cb49f3fe29c5ae654665a6202a704ecad6a94


```
m1~: $kubectl get nodes
```
NAME                STATUS   ROLES                  AGE     VERSION
120-96-143-106-m1   Ready    control-plane,master   65m     v1.20.2
120-96-143-107-w1   Ready    <none>                 5m36s   v1.20.2
120-96-143-108-w2   Ready    <none>                 110s    v1.20.2
- 在m1查看叢集上節點的設備



