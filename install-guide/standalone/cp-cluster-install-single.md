### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 클러스터 설치 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [시스템 구성도](#1.3)  
  1.4. [참고자료](#1.4)  

2. [Container Platform Cluster 설치](#2)  
  2.1. [Prerequisite](#2.1)  
  2.2. [SSH Key 생성 및 배포](#2.2)  
  2.3. [Container Platform Cluster Deployment 다운로드](#2.3)  
  2.4. [Container Platform Cluster 설치 준비](#2.4)  
  2.5. [Container Platform Cluster 설치](#2.5)  
  2.6. [Container Platform Cluster 설치 확인](#2.6)  

3. [Container Platform Cluster 삭제 (참고)](#3)  

4. [Resource 생성 시 주의사항](#4)  

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (Container Platform Cluster 설치 가이드) 는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에 배포되는 컨테이터 플랫폼을 설치하기 위한 Kubernetes Native를 설치하는 방법을 기술하였다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 Kubernetes Native를 검증하기 위한 Container Platform Cluster 기본 설치를 기준으로 작성하였다.

<br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 Kubernetes Cluster(Master, Worker) 환경으로 구성되어 있다.<br>
Container Platform Cluster Deployment를 통해 Kubernetes Cluster를 설치하고 Pod를 통해 Database, Private registry 등 미들웨어 환경을 제공하여 Container Image로 Kubernetes Cluster에 Container Platform 포털 환경을 배포한다. <br>
총 필요한 VM 환경으로는 **Install VM: 1개 (Master VM으로 대체 가능), Master VM: 1개 이상, Worker VM: 3개 이상**이 필요하고 본 문서는 Kubernetes Cluster 환경을 구성하기 위한 Master VM 과 Worker VM 설치 내용이다.

> NFS 배포 시 NFS-Server 설치가 우선 진행되어야 한다.  
> [NFS Server 설치](../nfs-server-install-guide.md)  

> Rook-Ceph 배포 시 **각 Worker VM에 Root Volume 외에 추가 Volume 할당**이 우선 진행되어야 한다.

![image 001]

<br>

### <div id='1.4'> 1.4. 참고자료
> https://kubespray.io  
> https://github.com/kubernetes-sigs/kubespray  

<br>

## <div id='2'> 2. Container Platform Cluster 설치

### <div id='2.1'> 2.1. Prerequisite
본 설치 가이드는 **Ubuntu 22.04** 환경에서 설치하는 것을 기준으로 하였다. Container Platform Cluster 설치를 위해서는 Ansible v7.6+, Jinja 3.1+ 및 python-netaddr이 Ansible 명령을 실행할 시스템에 설치되어 있어야 하며 설치 가이드에 따라 순차적으로 설치가 진행된다.


Container Platform Cluster 설치에 필요한 주요 소프트웨어 및 패키지 Version 정보는 다음과 같다.

|주요 소프트웨어|Version|Python Package|Version
|---|---|---|---|
|Kubespray|v2.23.0|ansible|7.6.0|
|Kubernetes Native|v1.27.5|cryptography|41.0.1|
|CRI-O|v1.27.1|jinja2|3.1.2|
|Helm|v3.12.3|jmespath|1.0.1|
|Istio|1.19.0|MarkupSafe|2.1.3|
|Podman|3.4.4|netaddr|0.8.0|
|Terraform|1.3.4|pbr|5.11.1|
|NFS Common||ruamel.yaml|0.17.31|
|Rook Ceph|1.12.3|ruamel.yaml.clib|0.2.7|
|Kubeflow|1.7.0||
|Ingress Nginx Controller|1.8.2||
|nfs-provisioner|4.0.0||


본 가이드 문서에서는 Container Platform 배포 시 다음을 권고하고 있다.

- deb / rpm 호환 Linux OS를 실행하는 하나 이상의 머신 (Ubuntu)
- 머신 당 8G (최소사양) or 16G (권장사양) 이상의 RAM
- control-plane 노드로 사용하는 머신에 2 개 (최소사양) or 4 개 (권장사양) 이상의 CPU
- 클러스터의 모든 시스템 간의 완전한 네트워크 연결


#### 방화벽 정보
- Master Nodes

| <center>프로토콜</center> | <center>포트</center> | <center>비고</center> |  
| :---: | :---: | :--- |  
| TCP | 111 | NFS PortMapper |  
| TCP | 2049 | NFS |  
| TCP | 2379-2380 | etcd server client API |  
| TCP | 6443 | Kubernetes API Server |  
| TCP | 10250 | Kubelet API |  
| TCP | 10251 | kube-scheduler |  
| TCP | 10252 | kube-controller-manager |  
| TCP | 10255 | Read-Only Kubelet API |  
| UDP | 4789 | Calico networking VXLAN |  

- Worker Node

| <center>프로토콜</center> | <center>포트</center> | <center>비고</center> |  
| :---: | :---: | :--- |  
| TCP | 111 | NFS PortMapper |  
| TCP | 2049 | NFS |  
| TCP | 10250 | Kubelet API |  
| TCP | 10255 | Read-Only Kubelet API |  
| TCP | 30000-32767 | NodePort Services |  
| UDP | 4789 | Calico networking VXLAN |  

<br>

### <div id='2.2'> 2.2. SSH Key 생성 및 배포
Container Platform Cluster 설치를 위해서는 SSH Key가 인벤토리의 모든 서버들에 복사되어야 한다. 본 설치 가이드에서는 RSA 공개키를 이용하여 SSH 접속 설정을 진행한다.  

SSH Key 생성 및 배포 이후의 모든 설치과정은 **Install VM 또는 Master Node**에서 진행한다.

- **Install VM 또는 Master Node**에서 RSA 공개키를 생성한다.
```
$ ssh-keygen -t rsa -m PEM
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): [엔터키 입력]
Enter passphrase (empty for no passphrase): [엔터키 입력]
Enter same passphrase again: [엔터키 입력]
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:pIG4/G309Dof305mWjdNz1OORx9nQgQ3b8yUP5DzC3w ubuntu@cp-master
The key's randomart image is:
+---[RSA 2048]----+
|            ..= o|
|   . .       * B |
|  . . . .   . = *|
| . .   +     + E.|
|  o   o S     +.O|
|   . o o .     XB|
|    . o . o   *oO|
|     .  .. o B oo|
|        .o. o.o  |
+----[SHA256]-----+
```

- 사용할 **Install, Master, Worker Node**에 공개키를 복사한다.
```
## 출력된 공개키 복사

$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5QrbqzV6g4iZT4iR1u+EKKVQGqBy4DbGqH7/PVfmAYEo3CcFGhRhzLcVz3rKb+C25mOne+MaQGynZFpZk4muEAUdkpieoo+B6r2eJHjBLopn5quWJ561H7EZb/GlfC5ThjHFF+hTf5trF4boW1iZRvUM56KAwXiYosLLRBXeNlub4SKfApe8ojQh4RRzFBZP/wNbOKr+Fo6g4RQCWrr5xQCZMK3ugBzTHM+zh9Ra7tG0oCySRcFTAXXoyXnJm+PFhdR6jbkerDlUYP9RD/87p/YKS1wSXExpBkEglpbTUPMCj+t1kXXEJ68JkMrVMpeznuuopgjHYWWD2FgjFFNkp ubuntu@cp-master
```

- 사용할 **Install, Master, Worker Node**의 authorized_keys 파일 본문의 마지막 부분(기존 본문 내용 아래 추가)에 공개키를 복사한다.
```
$ vi .ssh/authorized_keys

ex)
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRueywSiuwyfmCSecHu7iwyi3xYS1xigAnhR/RMg/Ws3yOuwbKfeDFUprQR24BoMaD360uyuRaPpfqSL3LS9oRFrj0BSaQfmLcMM1+dWv+NbH/vvq7QWhIszVCLzwTqlHrhgNsh0+EMhqc15KEo5kHm7d7vLc0fB5tZkmovsUFzp01Ceo9+Qye6+j+UM6ssxdTmatoMP3ZZKZzUPF0EZwTcGG6+8rVK2G8GhTqwGLj9E+As3GB1YdOvr/fsTAi2PoxxFsypNR4NX8ZTDvRdAUzIxz8wv2VV4mADStSjFpE7HWrzr4tZUjvvVFptU4LbyON9YY4brMzjxA7kTuf/e3j Generated-by-Nova
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5QrbqzV6g4iZT4iR1u+EKKVQGqBy4DbGqH7/PVfmAYEo3CcFGhRhzLcVz3rKb+C25mOne+MaQGynZFpZk4muEAUdkpieoo+B6r2eJHjBLopn5quWJ561H7EZb/GlfC5ThjHFF+hTf5trF4boW1iZRvUM56KAwXiYosLLRBXeNlub4SKfApe8ojQh4RRzFBZP/wNbOKr+Fo6g4RQCWrr5xQCZMK3ugBzTHM+zh9Ra7tG0oCySRcFTAXXoyXnJm+PFhdR6jbkerDlUYP9RD/87p/YKS1wSXExpBkEglpbTUPMCj+t1kXXEJ68JkMrVMpeznuuopgjHYWWD2FgjFFNkp ubuntu@cp-master
```

<br>

### <div id='2.3'> 2.3. Container Platform Cluster Deployment 다운로드
2.3.부터는 **Install VM 또는 Master Node**에서만 진행을 하면 된다.(Worker Node에는 더 이상 추가 작업이 없음)
Container Platform Cluster 설치에 필요한 Source File을 Download 받아 Container Platform Cluster 설치 작업 경로로 위치시킨다.

- Container Platform Cluster Deployment Download URL : https://github.com/k-paas/cp-deployment

- git clone 명령을 통해 다음 경로에서 Container Platform Cluster Deployment 다운로드를 진행한다.
```
$ git clone https://github.com/K-PaaS/cp-deployment.git -b branch_v1.5.x
```

<br>

### <div id='2.4'> 2.4. Container Platform Cluster 설치 준비
Container Platform Cluster 설치에 필요한 환경변수를 사전 정의 후 쉘 스크립트를 통해 설치를 진행한다.

- Container Platform Cluster 설치경로 이동한다.
```
$ cd cp-deployment/single
```

- Container Platform Cluster 설치에 필요한 환경변수를 정의한다. HostName, IP 정보는 다음을 통해 확인할 수 있다.
```
$ vi cp-cluster-vars.sh
```

- Node의 정보를 입력한다.
Master Node가 1개일 경우에는 ETCD, LoadBalancer 정보를 입력하지 않는다.
Master, ETCD, Worker Node의 정보는 사용자가 설정한 갯수에 따라 변수를 추가하여 설정한다.
```
#!/bin/bash

# Master Node Count Variable (eg. 1, 3, 5 ...)
export KUBE_CONTROL_HOSTS={Master Node의 갯수, 1개 일 경우 ETCD, LoadBalancer 설정 미진행}

# if KUBE_CONTROL_HOSTS > 1 (eg. external, stacked)
export ETCD_TYPE={Master Node의 갯수가 1개 이상일때 ETCD 배포 방식}

# if KUBE_CONTROL_HOSTS > 1
# HA Control Plane LoadBalanncer IP or Domain
export LOADBALANCER_DOMAIN={Master Node의 갯수가 1개 이상일때 LoadBalancer의 IP 또는 Domain 정보 입력}

# if ETCD_TYPE=external
# The number of ETCD node variable is set equal to the number of KUBE_CONTROL_HOSTS
export ETCD1_NODE_HOSTNAME={ETCD 배포방식이 External 일때 ETCD 1번 Node의 Hostname 정보 입력}
export ETCD1_NODE_PRIVATE_IP={ETCD 배포방식이 External 일때 ETCD 1번 Node의 Private IP 정보 입력}
...
export ETCD{n}_NODE_HOSTNAME={ETCD 배포방식이 External 일때 ETCD Node의 갯수에 맞춰 Hostname 정보 입력}
export ETCD{n}_NODE_PRIVATE_IP={ETCD 배포방식이 External 일때 ETCD Node의 갯수에 맞춰 Private IP 정보 입력}
...

# Master Node Info Variable
# The number of MASTER node variable is set equal to the number of KUBE_CONTROL_HOSTS
export MASTER1_NODE_HOSTNAME={Master 1번 Node의 HostName 정보 입력}
export MASTER1_NODE_PUBLIC_IP={Master 1번 Node의 Public IP 정보 입력}
export MASTER1_NODE_PRIVATE_IP={Master 1번 Node의 Private IP 정보 입력}
...
export MASTER{n}_NODE_HOSTNAME={Master Node의 갯수에 맞춰 HostName 정보 입력}
export MASTER{n}_NODE_PRIVATE_IP={Master Node의 갯수에 맞춰 Private IP 정보 입력}
...

# Worker Node Count Variable
export KUBE_WORKER_HOSTS={Worker Node의 갯수}

# Worker Node Info Variable
# The number of Worker node variable is set equal to the number of KUBE_WORKER_HOSTS
export WORKER1_NODE_HOSTNAME={Worker 1번 Node의 HostName 정보 입력}
export WORKER1_NODE_PRIVATE_IP={Worker 1번 Node의 Private IP 정보 입력}
...
export WORKER{n}_NODE_HOSTNAME={Worker Node의 갯수에 맞춰 HostName 정보 입력}
export WORKER{n}_NODE_PRIVATE_IP={Worker Node의 갯수에 맞춰 Private IP 정보 입력}
...
```

- 설치할 스토리지를 선택한다.   
Node에 볼륨 추가가 불가능할 경우 NFS를, 볼륨 추가가 가능할 경우 Rook-Ceph을 선택하는 것을 권장한다.  
NFS 선택 시 NFS Server의 Private IP 정보를 입력한다.
```
...
# Storage Variable (eg. nfs, rook-ceph)
export STORAGE_TYPE={설치할 Storage Type 정보 입력}

# if STORATE_TYPE=nfs
export NFS_SERVER_PRIVATE_IP={Storage Type nfs 설정 시 NFS Server의 Private IP 정보 입력}
...
```

- LoadBalancer Service Type 설정에 필요한 MetalLB IP 정보를 입력한다.
MetalLB에서 사용할 IP 대역 (VM이 해당된 서브넷에서 사용하지 않는 IP 대역)을 입력 한다.
Ingress Nginx Controller Service에서 사용할 IP는 설정한 MetalLB IP 대역과 중복되지 않도록 별도 할당한다.
```
...
# MetalLB Variable (eg. 192.168.0.150-192.168.0.160)
export METALLB_IP_RANGE={MetalLB에서 사용할 Private IP 대역 정보 입력}

# MetalLB Ingress Nginx Controller LoadBalancer Service External IP
export INGRESS_NGINX_PRIVATE_IP={MetalLB를 통해 Ingress Nginx Controller Service에서 사용할 Private IP 정보 입력}
```

<br>

### <div id='2.5'> 2.5. Container Platform Cluster 설치
쉘 스크립트를 통해 필요 패키지 설치, Node 구성정보 설정, Container Platform Cluster 설치정보 설정, Ansible playbook을 통한 Container Platform Cluster 설치를 일괄적으로 진행한다.

- 쉘 스크립트를 통해 설치를 진행한다.
```
$ source deploy-cp-cluster.sh
```

<br>

### <div id='2.6'> 2.6. Container Platform Cluster 설치 확인
Kubernetes Node 및 kube-system Namespace의 Pod를 확인하여 Container Platform Cluster 설치를 확인한다.

```
$ kubectl get nodes
NAME                 STATUS   ROLES                  AGE   VERSION
cp-master            Ready    control-plane          12m   v1.27.5
cp-worker-1          Ready    <none>                 10m   v1.27.5
cp-worker-2          Ready    <none>                 10m   v1.27.5
cp-worker-3          Ready    <none>                 10m   v1.27.5

$ kubectl get pods -n kube-system
NAME                                          READY   STATUS    RESTARTS      AGE
calico-node-d8sg6                             1/1     Running   0             9m22s
calico-node-kfvjx                             1/1     Running   0             10m
calico-node-khwdz                             1/1     Running   0             10m
calico-node-nc58v                             1/1     Running   0             10m
coredns-657959df74-td5c2                      1/1     Running   0             8m15s
coredns-657959df74-ztnjj                      1/1     Running   0             8m7s
dns-autoscaler-b5c786945-rhlkd                1/1     Running   0             8m9s
kube-apiserver-cp-master                      1/1     Running   0             12m
kube-controller-manager-cp-master             1/1     Running   1 (11m ago)   12m
kube-proxy-dj5c8                              1/1     Running   0             10m
kube-proxy-kkvhk                              1/1     Running   0             10m
kube-proxy-nfttc                              1/1     Running   0             10m
kube-proxy-znfgk                              1/1     Running   0             10m
kube-scheduler-cp-master                      1/1     Running   1 (11m ago)   12m
metrics-server-5cd75b7749-xcrps               2/2     Running   0             7m57s
nginx-proxy-cp-worker-1                       1/1     Running   0             10m
nginx-proxy-cp-worker-2                       1/1     Running   0             10m
nginx-proxy-cp-worker-3                       1/1     Running   0             10m
nodelocaldns-556gb                            1/1     Running   0             8m8s
nodelocaldns-8dpnt                            1/1     Running   0             8m8s
nodelocaldns-pvl6z                            1/1     Running   0             8m8s
nodelocaldns-x7grn                            1/1     Running   0             8m8s
```

<br>

## <div id='3'> 3. Container Platform Cluster 삭제 (참고)
Ansible playbook을 이용하여 Container Platform Cluster 삭제를 진행한다.

```
$ source reset-cp-cluster.sh
```

<br>

## <div id='4'> 4. Resource 생성 시 주의사항
사용자가 직접 Resource를 생성 시 다음과 같은 prefix를 사용하지 않도록 주의한다.

|Resource 명|생성 시 제외해야 할 prefix|
|---|---|
|전체 Resource|kube*|
|Namespace|all|
||kubernetes-dashboard|
||cp-portal-temp-namespace|
|Role|cp-init-role|
||cp-admin-role|
|ResourceQuota|cp-low-resourcequota|
||cp-medium-resourcequota|
||cp-high-resourcequota|
|LimitRanges|cp-low-limitrange|
||cp-medium-limitrange|
||cp-high-limitrange|
|Pod|nodes|
||resources|

<br>

[image 001]:images/standalone-v1.2.png

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > 클러스터 설치 가이드