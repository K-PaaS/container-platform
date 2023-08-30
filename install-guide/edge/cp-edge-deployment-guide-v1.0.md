## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [시스템 구성도](#1.3)  
  1.4. [참고자료](#1.4)  

2. [KubeEdge 설치](#2)  
  2.1. [Prerequisite](#2.1)  
  2.2. [Kubernetes Native Cluster 배포](#2.2)  
  2.3. [KubeEdge keadm 설치](#2.3)  
  2.4. [KubeEdge CloudCore 설치](#2.4)  
  2.5. [KubeEdge EdgeCore 설치](#2.5)  
  2.6. [KubeEdge Flannel CNI 활성화](#2.6)  
  2.7. [kubectl logs 기능 활성화](#2.7)  
  2.8. [KubeEdge 설치 확인](#2.8)  

3. [KubeEdge Reset (참고)](#3)  

4. [컨테이너 플랫폼 운영자 생성 및 Token 획득](#4)  
  4.1. [Cluster Role 운영자 생성 및 Token 획득](#4.1)  
  4.2. [Namespace 사용자 Token 획득](#4.2)  

5. [Resource 생성 시 주의사항](#5)

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (KubeEdge 설치 가이드) 는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에 배포되는 컨테이터 플랫폼을 설치하기 위한 KubeEdge를 설치하는 방법을 기술하였다.

PaaS-TA 5.5 버전부터는 KubeEdge 기반으로 단독 배포를 지원한다. 기존 Container 서비스 기반으로 설치를 원할 경우에는 PaaS-TA 5.0 이하 버전의 문서를 참고한다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 KubeEdge를 검증하기 위한 기본 설치를 기준으로 작성하였다.

<br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 Kubernetes Cluster(Master, Worker, Edge)와 BOSH Inception(DBMS, HAProxy, Private Registry)환경으로 구성되어 있다. <br>
Kubespray를 통해 Kubernetes Cluster(Master, Worker)를 설치하고 Kubernetes 환경에 KubeEdge를 설치한다. BOSH release로는 Database, Private registry 등 미들웨어 환경을 제공하여 Docker Image로 Kubernetes Cluster에 Container Platform 포털 환경을 배포한다. <br>
총 필요한 VM 환경으로는 **Master VM: 1개, Worker VM: 1개 이상, Edge 1개 이상, BOSH Inception VM: 1개**가 필요하고 본 문서는 Kubernetes Cluster 환경을 구성하기 위한 Master VM 과 Worker VM, Edge VM 설치 내용이다.

![image 001]

<br>

### <div id='1.4'> 1.4. 참고자료
> https://docs.docker.com/engine/install/  
> https://kubeedge.io/en/docs/   
> https://github.com/kubeedge/kubeedge

<br>

## <div id='2'> 2. KubeEdge 설치

### <div id='2.1'> 2.1. Prerequisite
본 설치 가이드는 **Ubuntu 18.04** 환경에서 설치하는 것을 기준으로 하였다. KubeEdge 설치를 위해서는 Docker, Kubernetes Native Cluster가 시스템에 배포되어 있어야 한다.

KubeEdge 설치에 필요한 주요 소프트웨어 및 패키지 Version 정보는 다음과 같다.

|주요 소프트웨어|Version|
|---|---|
|KubeEdge|v1.6.1|
|Kubernetes Native|v1.18.6|
|Docker|v20.10.2|

Kubernetes 공식 가이드 문서에서는 Cluster 배포 시 다음을 권고하고 있다.

- deb / rpm 호환 Linux OS를 실행하는 하나 이상의 머신 (Ubuntu 또는 CentOS)
- 머신 당 2G 이상의 RAM
- control-plane 노드로 사용하는 머신에 2 개 이상의 CPU
- 클러스터의 모든 시스템 간의 완전한 네트워크 연결

#### 방화벽 정보
- Master Node

| <center>프로토콜</center> | <center>포트</center> | <center>Source</center> | <center>Destination</center> | <center>비고</center> |  
| :---: | :---: | :---: | :---: | :--- |  
| TCP | 6443 ||| kubernetes API Server |  
| TCP | 2379-2380 ||| etcd server client API |  
| TCP | 10250 ||| Kubelet API |  
| TCP | 10251 ||| kube-scheduler |  
| TCP | 10252 ||| kube-controller-manager |  
| TCP | 10255 ||| Read-Only Kubelet API |  
| UDP | 8285 | Worker Nodes || flannel overlay network |  
| UDP | 8472 | Worker Nodes || flannel overlay network |  
| TCP | 179 | Worker Nodes || Calio BGP network |  

---
- Worker Node

| <center>프로토콜</center> | <center>포트</center> | <center>Source</center> | <center>Destination</center> | <center>비고</center> |  
| :---: | :---: | :---: | :---: | :--- |  
| TCP | 10250 ||| Kubelet API |  
| TCP | 10255 ||| Read-Only Kubelet API |  
| TCP | 30000-32767 ||| NodePort Services |  
| UDP | 8285 | Worker Nodes || flannel overlay network |  
| UDP | 8472 | Worker Nodes || flannel overlay network |  
| TCP | 179 | Worker Nodes || Calio BGP network |

<br>

### <div id='2.2'> 2.2. Kubernetes Native Cluster 배포
KubeEdge 설치를 위해서는 Cloud 영역에 Kubernetes Cluster가 배포되어있어야 하며, 배포 이후 Edge 영역에 Edge Node를 배포하여야 한다.

- Cloud 영역에 Kubernetes Cluster 배포를 진행한다. Cluster 배포는 Kubespray를 통해 진행하며, 배포과정에서 Edge Node에 CNI를 배포하기 위해 아래와 같이 Kubespray 일부 설정 변경이 필요하다.

> https://github.com/PaaS-TA/paas-ta-container-platform/blob/dev/install-guide/standalone/paas-ta-container-platform-standalone-deployment-guide-v1.0.md

- CNI Plugin을 calico에서 flannel로 변경한다.
```
$ vi inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml

...
kube_network_plugin: flannel (수정)
...
```

<br>

### <div id='2.3'> 2.3. KubeEdge keadm 설치
KubeEdge 설치를 위한 keadm 설치를 진행한다. keadm 실행 시 Super User 혹은 root 권한이 필요하므로 root 권한으로 설치를 진행한다.

- Cloud 영역의 **Master Node**와 Edge 영역의 **Edge Node**로 사용할 VM에 keadm 다운로드 및 설치를 진행한다.

```
$ sudo su -

# git clone https://github.com/PaaS-TA/paas-ta-container-platform-deployment.git

# cp paas-ta-container-platform-deployment/edge/keadm /usr/bin/keadm
```

<br>

### <div id='2.4'> 2.4. KubeEdge CloudCore 설치
Cloud 영역의 Master Node에 KubeEdge CloudCore를 설치하여 설정을 진행한다.

- keadm init 명령으로 Cloud 영역의 **Master Node**에 CloudCore 설치를 진행한다.
```
## {MASTER_PUB_IP} : Master Node Public IP
## {MASTER_PRIV_IP} : Master Node Private IP

# keadm init --advertise-address={MASTER_PUB_IP} --master=https://{MASTER_PRIV_IP}:6443 --kubeedge-version 1.6.1
```

- Edge 영역에 EdgeCore를 설치하기 위한 Token값을 가져온다.
```
# keadm gettoken
```

<br>

### <div id='2.5'> 2.5. KubeEdge EdgeCore 설치
Edge 영역의 **Edge Node**에 Docker 설치를 사전 진행 후, KubeEdge EdgeCore를 설치하여 설정을 진행한다.

- apt-get update를 진행한다.
```
# apt-get update
```

- Docker 설치에 필요한 Package 설치를 진행한다.
```
# apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

- Docker Download 및 설치를 위한 apt-key 및 apt-repository를 추가한다.
```
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# apt-key fingerprint 0EBFCD88

# add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

- 설치 가능한 Docker 버전 정보를 확인한다.
```
# apt-cache madison docker-ce
 docker-ce | 5:20.10.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.2~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.1~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:20.10.0~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.15~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.14~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.13~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.12~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.11~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.10~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.9~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.8~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.7~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.6~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.2~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.1~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.0~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.9~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.8~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.7~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.6~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.2~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.1~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.0~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.3~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.2~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.0~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.03.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
```

- Docker 설치를 진행한다.
```
## {VERSION_STRING} : Version 정보. (ex : 5:20.10.2~3-0~ubuntu-bionic)

# apt-get install -y docker-ce={VERSION_STRING} docker-ce-cli={VERSION_STRING} containerd.io
```

- keadm join 명령으로 **Edge Node**에 EdgeCore 설치를 진행한다.
```
## {MASTER_PUB_IP} : Master Node Public IP
## {INTERFACE_NAME} : 실제 Edge Node에서 사용중인 인터페이스 이름 (ex: ens5)
## {GET_TOKEN} : Cloud 영역에서 CloudCore 설치 이후 호출한 Token 값

# keadm join --cloudcore-ipport={MASTER_PUB_IP}:10000 --token={GET_TOKEN} --kubeedge-version 1.6.1
```

- **Master Node**에서 Edge Node에 배포할 Flannel CNI DaemonSet YAML 수정을 위해 기본 YAML을 가져온다.

```
# kubectl get ds kube-flannel -n kube-system -o yaml > edge-flannel.yaml
```

- **Master Node**에서 Kubespray를 통해 배포된 Flannel CNI가 Edge Node에 배포되지 않도록 DaemonSet yaml 수정을 진행한다.
```
# kubectl edit ds kube-flannel -n kube-system
```

- spec.template.spec 경로에 아래 내용을 추가한다.
```
...
     affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/edge (추가)
                operator: DoesNotExist (추가)
...
```

- **Master Node**에서 Edge Node에 배포할 Flannel CNI DaemonSet을 신규로 배포한다.
```
# vi edge-flannel.yaml

...
  labels:
    k8s-app: flannel-edge (수정)
    tier: node
...
  name: kube-flannel-edge (수정)
...
spec:
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: flannel-edge (수정)
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: flannel-edge (수정)
        tier: node
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
              - key: kubernetes.io/arch
                operator: In
                values:
                - amd64
              - key: node-role.kubernetes.io/agent (추가)
                operator: Exists (추가)
      containers:
      - command:
        - /opt/bin/flanneld
        - --ip-masq
        - --kube-subnet-mgr
        - --kube-api-url=http://127.0.0.1:10550 (추가)
...
        name: kube-flannel-edge (수정)
...
```

```
# kubectl apply -f edge-flannel.yaml
```

- KubeEdge에서는 본 설치 가이드 작성 시점에 Ingress를 지원하지 않으므로 Edge Node에 Ingress Controller가 배포되지 않도록 조치가 필요하다.

```
## Ingress Controller Error 상태 확인

# kubectl get pods -n ingress-nginx
NAME                             READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-drq98   1/1     Running   0          64m
ingress-nginx-controller-nfckc   1/1     Running   0          64m
ingress-nginx-controller-z7bxk   0/1     Error     18         53m
```

- **Master Node**에서 Edge Node에 배포되지 않도록 DaemonSet yaml 수정을 진행한다.
```
# kubectl edit daemonsets.apps ingress-nginx-controller -n ingress-nginx
```

- spec.template.spec 경로에 아래 내용을 추가한다.
```
     affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/edge
                operator: DoesNotExist
```

<br>

### <div id='2.6'> 2.6. KubeEdge Flannel CNI 활성화
KubeEdge에서 Flannel CNI를 정상 배포 및 사용하기 위한 설정을 진행한다.

- **Master Node**에서 CloudCore dynamicController를 활성화한다.
```
# vi /etc/kubeedge/config/cloudcore.yaml

...
dynamicController:
  enable: true (추가)
...
```

- CloudCore 서비스를 재시작한다.
```
# pkill cloudcore ; nohup /usr/local/bin/cloudcore > /var/log/kubeedge/cloudcore.log 2>&1 &
```

- **Edge Node**에서 CNI 바이너리를 다운로드 한다.
```
# wget "https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz"

# mkdir -p /opt/cni/bin

# tar xf cni-plugins-linux-amd64-v0.9.1.tgz -C /opt/cni/bin
```


- **Edge Node**에서 EdgeCore 설정을 수정한다.
```
# vi /etc/kubeedge/config/edgecore.yaml

modules:
...
  edgeMesh:
    enable: false (수정)
...
  edged:
...
    networkPluginName: cni (추가)
    clusterDNS: "10.233.0.3" (수정)
    clusterDomain: "cluster.local" (수정)
...
  metaManager:
...
    mataServer:
      enable: true (추가)
...
```

- EdgeCore 서비스를 재시작한다.
```
# service edgecore restart
```

<br>

### <div id='2.7'> 2.7. kubectl logs 기능 활성화
KubeEdge에서는 기본적으로 kubectl logs 명령을 사용할 수 없는 이슈가 존재한다. 본 설치 가이드에서는 해당 기능을 활성화 하기 위한 설정 가이드를 제공한다.  

- **Master Node**에서 kubernetes ca.crt 및 ca.key 파일을 확인한다.
```
# ls /etc/kubernetes/pki/
```

- **Master Node**에서 CLOUDCOREIPS 환경변수 설정 및 확인을 진행한다. (HA Cluster 구성 시 VIP 설정)
```
## {MASTER_PUB_IP} : Master Node Public IP

# export CLOUDCOREIPS="{MASTER_PUB_IP}"

# echo $CLOUDCOREIPS
```

- **Master Node**에서 certgen.sh 다운로드 및 인증서 생성을 진행한다.
```
# cd /etc/kubeedge

# wget https://raw.githubusercontent.com/kubeedge/kubeedge/master/build/tools/certgen.sh

# chmod +x certgen.sh

# /etc/kubeedge/certgen.sh stream
```

- **Master Node**에서 iptables을 설정한다.
```
# iptables -t nat -A OUTPUT -p tcp --dport 10350 -j DNAT --to $CLOUDCOREIPS:10003
```

- **Master Node**에서 cloudcore.yaml 파일을 수정한다. (enable: true 로 변경)
```
# vi /etc/kubeedge/config/cloudcore.yaml
```

```
cloudStream:
  enable: true (수정)
  streamPort: 10003
  tlsStreamCAFile: /etc/kubeedge/ca/streamCA.crt
  tlsStreamCertFile: /etc/kubeedge/certs/stream.crt
  tlsStreamPrivateKeyFile: /etc/kubeedge/certs/stream.key
  tlsTunnelCAFile: /etc/kubeedge/ca/rootCA.crt
  tlsTunnelCertFile: /etc/kubeedge/certs/server.crt
  tlsTunnelPrivateKeyFile: /etc/kubeedge/certs/server.key
  tunnelPort: 10004
```

- **Master Node**에서 cloudcore를 재시작한다.
```
# pkill cloudcore
# nohup cloudcore > cloudcore.log 2>&1 &
```

- **Edge Node**에서 edgecore.yaml 파일을 수정한다. (enable: true, server: {MASTER_PUB_IP}:10004)
```
# vi /etc/kubeedge/config/edgecore.yaml
```

```
## {MASTER_PUB_IP} : Master Node Public IP

edgeStream:
  enable: true
  handshakeTimeout: 30
  readDeadline: 15
  server: {MASTER_PUB_IP}:10004
  tlsTunnelCAFile: /etc/kubeedge/ca/rootCA.crt
  tlsTunnelCertFile: /etc/kubeedge/certs/server.crt
  tlsTunnelPrivateKeyFile: /etc/kubeedge/certs/server.key
  writeDeadline: 15
```



- kube-proxy 배포되어 있을 경우 edgecore 재시작이 불가능하므로 edgecore 재시작 시 kube-proxy 배포 여부를 우회할 수 있는 방법을 기술한다.

- edgecore.service 파일을 수정한다.
```
# vi /etc/kubeedge/edgecore.service
```

- edgecore.service 파일의 [Service]에 다음을 추가한다.
```
Environment="CHECK_EDGECORE_ENVIRONMENT=false"
```

- **Edge Node**에서 edgecore를 재시작한다.
```
# systemctl daemon-reload
# systemctl restart edgecore.service
```

<br>

### <div id='2.8'> 2.8. KubeEdge 설치 확인
Kubernetes Node 및 kube-system Namespace의 Pod를 확인하여 KubeEdge 설치를 확인한다.

```
# kubectl get nodes
NAME            STATUS   ROLES        AGE   VERSION
ip-10-0-0-231   Ready    agent,edge   55m   v1.18.6-kubeedge-v1.4.0
ip-10-0-0-40    Ready    <none>       67m   v1.18.6
ip-10-0-0-60    Ready    master       68m   v1.18.6

# kubectl get pods -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-6d654c9787-x7bs9   1/1     Running   0          67m
calico-node-m795z                          1/1     Running   0          5m27s
calico-node-ttrzd                          1/1     Running   0          5m12s
coredns-7d45cfccd7-67crs                   1/1     Running   0          66m
coredns-7d45cfccd7-wmlvq                   1/1     Running   0          66m
dns-autoscaler-757b95599b-lhbkl            1/1     Running   0          66m
kube-apiserver-ip-10-0-0-60                1/1     Running   0          68m
kube-controller-manager-ip-10-0-0-60       1/1     Running   0          68m
kube-proxy-9fmll                           1/1     Running   0          55m
kube-proxy-hhh5r                           1/1     Running   0          67m
kube-proxy-q2lvt                           1/1     Running   0          67m
kube-scheduler-ip-10-0-0-60                1/1     Running   0          68m
metrics-server-7fb7789fb-vxsnr             2/2     Running   0          66m
nginx-proxy-ip-10-0-0-40                   1/1     Running   0          67m
nodelocaldns-kxdpj                         1/1     Running   0          66m
nodelocaldns-p6krc                         1/1     Running   0          55m
nodelocaldns-pg8md                         1/1     Running   0          66m
```

<br>

## <div id='3'> 3. KubeEdge Reset (참고)
Cloud Side, Edge Side에서 KubeEdge를 중지한다. 필수구성요소는 삭제하지 않는다.

- Cloud Side에서 cloudcore를 중지하고 kubeedge Namespace와 같은 Kubernetes Master에서 KubeEdge 관련 리소스를 삭제한다.
```
# keadm reset --kube-config=$HOME/.kube/config
```

- Edge Side에서 edgecore를 중지한다.
```
# keadm reset
```

<br>

## <div id='4'> 4. 컨테이너 플랫폼 운영자 생성 및 Token 획득 (참고)

### <div id='4.1'> 4.1. Cluster Role 운영자 생성 및 Token 획득
KubeEdge 설치 이후에 Cluster Role을 가진 운영자의 Service Account를 생성한다. 해당 Service Account의 Token은 운영자 포털에서 Super Admin 계정 생성 시 이용된다.

- Service Account를 생성한다.
```
## {SERVICE_ACCOUNT} : Service Account 명

$ kubectl create serviceaccount {SERVICE_ACCOUNT} -n kube-system
(eg. kubectl create serviceaccount k8sadmin -n kube-system)
```

- Cluster Role을 생성한 Service Account에 바인딩한다.
```
$ kubectl create clusterrolebinding {SERVICE_ACCOUNT} --clusterrole=cluster-admin --serviceaccount=kube-system:{SERVICE_ACCOUNT}
```

- 생성한 Service Account의 Token을 획득한다.
```
## {SECRET_NAME} : Mountable secrets 값 확인

$ kubectl describe serviceaccount {SERVICE_ACCOUNT} -n kube-system

$ kubectl describe secret {SECRET_NAME} -n kube-system | grep -E '^token' | cut -f2 -d':' | tr -d " "
```

### <div id='4.2'> 4.2. Namespace 사용자 Token 획득
포털에서 Namespace 생성 및 사용자 등록 이후 Token값을 획득 시 이용된다.

- Namespace 사용자의 Token을 획득한다.
```
## {SECRET_NAME} : Mountable secrets 값 확인
## {NAMESPACE} : Namespace 명

$ kubectl describe serviceaccount {SERVICE_ACCOUNT} -n {NAMESPACE}

$ kubectl describe secret {SECRET_NAME} -n {NAMESPACE} | grep -E '^token' | cut -f2 -d':' | tr -d " "
```

<br>

## <div id='5'> 5. Resource 생성 시 주의사항
사용자가 직접 Resource를 생성 시 다음과 같은 prefix를 사용하지 않도록 주의한다.

|Resource 명|생성 시 제외해야 할 prefix|
|---|---|
|전체 Resource|kube*|
|Namespace|all|
||kubernetes-dashboard|
||paas-ta-container-platform-temp-namespace|
|Role|paas-ta-container-platform-init-role|
||paas-ta-container-platform-admin-role|
|ResourceQuota|paas-ta-container-platform-low-rq|
||paas-ta-container-platform-medium-rq|
||paas-ta-container-platform-high-rq|
|LimitRanges|paas-ta-container-platform-low-limit-range|
||paas-ta-container-platform-medium-limit-range|
||paas-ta-container-platform-high-limit-range|
|Pod|nodes|
||resources|

<br>


----
[image 001]:images/cp-001.png
