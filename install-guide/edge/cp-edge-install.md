### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 Edge 배포 가이드

<br><br>

## Table of Contents

1. [문서 개요](#1)<br>
  1.1. [목적](#1.1)<br>
  1.2. [범위](#1.2)<br>
  1.3. [시스템 구성도](#1.3)<br>
  1.4. [참고자료](#1.4)

2. [K-PaaS 컨테이너 플랫폼 Edge 배포](#2)<br>
  2.1. [Prerequisite](#2.1)<br>
  2.1.1. [클러스터](#2.1.1)<br>
  2.1.2. [OS](#2.1.2)<br>
  2.1.3. [주요 소프트웨어](#2.1.3)<br>
  2.1.4. [방화벽](#2.1.4)<br>
  2.1.5. [CloudCore 서비스 설정](#2.1.5)<br>
  2.2. [K-PaaS 컨테이너 플랫폼 클러스터 설치](#2.2)<br>
  2.3. [K-PaaS 컨테이너 플랫폼 Edge 배포 준비](#2.3)<br>
  2.4. [K-PaaS 컨테이너 플랫폼 Edge 배포](#2.4)<br>
  2.5. [K-PaaS 컨테이너 플랫폼 Edge 배포 확인](#2.5)

3. [K-PaaS 컨테이너 플랫폼 Edge 배포 삭제 (참고)](#3)

4. [Resource 생성 시 주의사항](#4)

<br><br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (K-PaaS 컨테이너 플랫폼 클러스터 Edge 배포 가이드) 는 기획자, 개발자, 운영자 지원 환경의 개방형 PaaS 플랫폼인 K-PaaS 컨테이너 플랫폼 클러스터의 Edge 노드를 구성하는 방법을 기술하였다.

<br><br>

### <div id='1.2'> 1.2. 범위
설치 범위는 K-PaaS 컨테이너 플랫폼 Edge 환경의 기반이 되는 Edge 노드 구성을 `단일 클라우드` 환경 기준으로 작성하였다.

<br><br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 쿠버네티스 `단일 클러스터` (Control Plane, Worker, Edge) 환경으로 구성되어 있다.

K-PaaS 컨테이너 플랫폼 Deployment를 통해 쿠버네티스 `단일 클러스터`를 구성하고 Edge 배포를 진행하여 클러스터에 Edge 노드 환경을 제공한다.

K-PaaS 컨테이너 플랫폼 Edge 노드 구성에 필요한 인스턴스 환경으로는 아래 구성을 참고한다.

|환경|인스턴스 종류|인스턴스 갯수|비고|
|---|---|---|---|
|Cloud 환경|-|-|기존 배포한 클러스터의 노드 사용|
|Edge 환경|Edge|1개 이상|`ARM64` 아키텍쳐 OS로 인스턴스 생성|

<br><br>

각 배포 유형 별 시스템 구성도는 다음과 같다.

***[ 단일 클라우드 클러스터, Edge 노드 구성 ]***

![image 001]

<br><br>

### <div id='1.4'> 1.4. 참고자료
> https://kubeedge.io/en/docs/<br>
> https://github.com/kubeedge/kubeedge

<br><br>

## <div id='2'> 2. K-PaaS 컨테이너 플랫폼 Edge 배포

### <div id='2.1'> 2.1. Prerequisite
K-PaaS 컨테이너 플랫폼 Edge 배포를 위한 전제조건은 아래 기술하였다.

<br><br>

### <div id='2.1.1'> 2.1.1. 클러스터 (***필수 설치***)
K-PaaS 컨테이너 플랫폼 `단일 클라우드` 설치를 진행한다.

<br><br>

### <div id='2.1.2'> 2.1.2. OS (***필수 확인***)
K-PaaS 컨테이너 플랫폼 Edge 배포에 필요한 OS 환경 정보는 다음과 같다.

|환경|노드|지원 OS|버전|아키텍쳐|
|---|---|---|---|---|
|Cloud 환경|Control Plane<br>Worker|Ubuntu|22.04|amd64|
|Edge 환경|Edge|Ubuntu|20.04|arm64|
|Edge 환경|Edge|Ubuntu|22.04|arm64|

<br><br>

### <div id='2.1.3'> 2.1.3. 주요 소프트웨어 (참고)
K-PaaS 컨테이너 플랫폼 Edge 배포에 필요한 주요 소프트웨어 정보는 다음과 같다.

|주요 소프트웨어|버전|
|---|---|
|Kubernetes Native|v1.30.4|
|Kubernetes Native (Edge Node)|v1.24.17|
|CRI-O|1.30.3|
|CRI-O (Edge Node)|v1.24.0|
|KubeEdge|v1.14.4|
|EdgeMesh|v1.12.0|

<br><br>

### <div id='2.1.4'> 2.1.4. 방화벽 (***필수 설정***)
K-PaaS 컨테이너 플랫폼 Edge 배포에 필요한 방화벽 정보는 다음과 같다.

> Edge 배포에서 일부 방화벽이 추가되었다.

Control Plane 노드

|프로토콜|포트|비고|
|---|---|---|
|TCP|111|NFS PortMapper|
|TCP|2049|NFS|
|TCP|2379-2380|etcd server client API|
|TCP|6443|kubernetes API Server|
|TCP|9443|cloudcore router port|
|TCP|10001|cloudHub quic port|
|TCP|10002|cloudHub https port|
|TCP|10003|cloudStream streamPort|
|TCP|10004|cloudStream tunnelPort|
|TCP|10250|Kubelet API|
|TCP|10251|kube-scheduler|
|TCP|10252|kube-controller-manager|
|TCP|10255|Read-Only Kubelet API|
|TCP|20004|edgeMesh server containerPort|
|TCP|20006|edgeMesh tunnel listenPort|
|TCP|40001|edgeMesh edgeProxy listenPort|
|TCP|30000-32767|NodePort Services|
|UDP|4789|Calico networking VXLAN|

<br>

Worker 노드

|프로토콜|포트|비고|
|---|---|---|
|TCP|111|NFS PortMapper|
|TCP|2049|NFS|
|TCP|10250|Kubelet API|
|TCP|10255|Read-Only Kubelet API|
|TCP|20006|edgeMesh tunnel listenPort|
|TCP|40001|edgeMesh edgeProxy listenPort|
|TCP|30000-32767|NodePort Services|
|UDP|4789|Calico networking VXLAN|

<br>

Edge 노드

|프로토콜|포트|비고|
|---|---|---|
|TCP|111|NFS PortMapper|
|TCP|1883-1884|eventBus mqttPort|
|TCP|2049|NFS|
|TCP|10001|edgeHub quic port|
|TCP|10250|Kubelet API|
|TCP|10255|Read-Only Kubelet API|
|TCP|10350|Use kubectl logs|
|TCP|10550|edgecore list-watch port|
|TCP|20006|edgeMesh tunnel listenPort|
|TCP|40001|edgeMesh edgeProxy listenPort|
|TCP|30000-32767|NodePort Services|

<br><br>

### <div id='2.1.6'> 2.1.6. CloudCore 서비스 설정 (***필수 설정***)
K-PaaS 컨테이너 플랫폼 서비스 구성을 위해 필요한 CloudCore 서비스 설정 정보는 다음과 같다.

<br>

|서비스|설명|비고|
|---|---|---|
|CloudCore|K-PaaS 컨테이너 플랫폼 서비스에서 EdgeNode와 연결하기 위한 서비스|***`1개 인터페이스 또는 1개 로드밸런서`*** 생성 필요<br>Public IP 할당 필요|

<br>

K-PaaS 컨테이너 플랫폼 클러스터에서는 MetalLB를 통해 로드밸런서 타입 서비스의 External IP를 할당한다.<br>
해당 External IP로 외부 통신 및 서비스를 지원하기 위해 ***`다음 두가지 중 한가지 방식을 선택`*** 하여 설정을 진행한다.

<br>

|방식|설명|비고|
|---|---|---|
|인터페이스 추가|1개 Control Plane 노드에 Public IP가 할당된 신규 인터페이스 추가|Public IP 사용에 대한 비용만 발생<br>HA 구성에서 해당 노드 장애 시 CloudCore 서비스 외부 접근 불가|
|로드밸런서 생성|Public IP가 할당된 로드밸런서 서비스 생성|로드밸런서 서비스에 대한 비용 추가 발생<br>HA 구성에서 일부 Control Plane 노드 장애 발생시에도 CloudCore 서비스 정상<br>운영 환경에서 권장|

<br>

CloudCore 서비스 설정 방법은 K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드 내 [2.1.6.1. Control Plane 노드 추가 인터페이스](https://github.com/K-PaaS/container-platform/blob/master/install-guide/standalone/cp-cluster-install-single.md#2.1.6.1) 항목과 [2.1.6.2. 클라우드 로드밸런서 서비스](https://github.com/K-PaaS/container-platform/blob/master/install-guide/standalone/cp-cluster-install-single.md#2.1.6.2) 항목을 참고한다.

<br>

또한, 로드밸런서 서비스 방식으로 설정할 경우 다음의 포트 매핑 정보를 기준으로 로드밸런서를 생성한다.

|프로토콜|포트|노드포트|비고|
|---|---|---|---|
|TCP|10000|30000||
|TCP|10001|30001||
|TCP|10002|30002||
|TCP|10003|30003||
|TCP|10004|30004||

<br><br>

### <div id='2.2'> 2.2. K-PaaS 컨테이너 플랫폼 클러스터 설치
K-PaaS 컨테이너 플랫폼 Edge 배포를 위해서는 클라우드 환경에 K-PaaS 컨테이너 플랫폼 클러스터가 배포되어 있어야 하며, 이후 Edge 환경에 Edge 노드를 배포하여야 한다.

<br>

Cloud 환경에 K-PaaS 컨테이너 플랫폼 클러스터 Deployment 통해 쿠버네티스 클러스터 배포를 진행한다.

> [K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드](https://github.com/K-PaaS/container-platform/blob/master/install-guide/standalone/cp-cluster-install-single.md)

<br><br>

### <div id='2.3'> 2.3. K-PaaS 컨테이너 플랫폼 Edge 배포 준비
K-PaaS 컨테이너 플랫폼 Edge 배포에 필요한 환경변수를 사전 정의 후 쉘 스크립트를 통해 설치를 진행한다.

Edge 노드의 환경이 `라즈베리파이`일 경우 다음 정보를 추가한다.<br>
`라즈베리파이` 환경이 아닐 경우 아래의 과정을 생략하고 K-PaaS 컨테이너 플랫폼 Edge 배포 환경변수 정의부터 진행한다.
```
# vi /boot/firmware/cmdline.txt

... cgroup_enable=memory cgroup_memory=1 (맨 뒤에 추가)
```

<br>

`라즈베리파이` Reboot을 진행한다.
```
# reboot
```

<br>

K-PaaS 컨테이너 플랫폼 Edge 배포 설치경로로 이동한다. 이후 부터는 **Control Plane 노드**에서만 진행을 하면 된다.
```
$ cd ~/cp-deployment/standalone
```

<br>

K-PaaS 컨테이너 플랫폼 Edge 배포에 필요한 환경변수를 정의한다.
```
$ vi cp-edge-vars.sh
```

<br>

|환경변수|설명|비고|
|---|---|---|
|CLOUDCORE_PRIVATE_VIP|MetalLB를 통해 CloudCore Service에서 사용할 인터페이스 Private IP|로드밸런서 서비스 이용 시 **`CLOUDCORE_VIP`** 값과 중복 입력|
|CLOUDCORE_VIP|CloudCore Service에서 사용할 인터페이스 또는 로드밸런서 서비스에 할당한 Public IP||
|CLOUDCORE1_NODE_HOSTNAME|CloudCore가 설치될 노드의 호스트명|Control Plane 또는 Worker 노드 중 1개 노드 정보 입력|
|CLOUDCORE2_NODE_HOSTNAME|CloudCore가 설치될 노드의 호스트명|Control Plane 또는 Worker 노드 중 1개 노드 정보 입력|
|EDGE_NODE_CNT|Edge 노드의 갯수||
|EDGE1_NODE_HOSTNAME|Edge 1번 노드의 호스트명||
|EDGE1_NODE_PUBLIC_IP|Edge 1번 노드의 Public IP||
|EDGE{n}_NODE_HOSTNAME|Edge n번 노드의 호스트명|**`EDGE_NODE_CNT`** 값이 2 이상일 경우 설정<br>**`EDGE_NODE_CNT`** 값만큼 설정|
|EDGE{n}_NODE_PUBLIC_IP|Edge n번 노드의 Public IP|**`EDGE_NODE_CNT`** 값이 2 이상일 경우 설정<br>**`EDGE_NODE_CNT`** 값만큼 설정|

<br>

```
#!/bin/bash

export CLOUDCORE_PRIVATE_VIP=
export CLOUDCORE_VIP=

export CLOUDCORE1_NODE_HOSTNAME=
export CLOUDCORE2_NODE_HOSTNAME=

export EDGE_NODE_CNT=

export EDGE1_NODE_HOSTNAME=
export EDGE1_NODE_PUBLIC_IP=
...
export EDGE{n}_NODE_HOSTNAME=
export EDGE{n}_NODE_PUBLIC_IP=
```

<br><br>


### <div id='2.4'> 2.4. K-PaaS 컨테이너 플랫폼 Edge 배포
쉘 스크립트를 통해 필요 패키지 설치, Edge 배포 환경변수 설정, Ansible playbook을 통한 K-PaaS 컨테이너 플랫폼 Edge 배포를 순차적으로 진행한다.

```
$ source deploy-cp-edge.sh
```

<br><br>

### <div id='2.5'> 2.5. K-PaaS 컨테이너 플랫폼 Edge 배포 확인
노드 및 kube-system, kubeedge 네임스페이스의 Pod를 확인하여 K-PaaS 컨테이너 플랫폼 Edge 배포를 확인한다.<br>
구성에 따라 조회되는 노드 및 Pod의 정보는 다를 수 있으며 아래는 단일 Control Plane 구성으로 배포했을때의 조회 결과이다. 

```
$ kubectl get nodes
NAME                 STATUS   ROLES                  AGE     VERSION
cp-edge              Ready    agent,edge             5m40s   v1.24.17-kubeedge-v1.14.4
cp-master            Ready    control-plane,master   39m     v1.30.4
cp-worker-1          Ready    <none>                 38m     v1.30.4
cp-worker-2          Ready    <none>                 38m     v1.30.4
cp-worker-3          Ready    <none>                 38m     v1.30.4

$ kubectl get pods -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-b5f8f6849-hhbgh    1/1     Running   0          9m22s
calico-node-4hbw5                          1/1     Running   0          4m34s
calico-node-8q5tv                          1/1     Running   0          5m9s
calico-node-qlq5k                          1/1     Running   0          5m26s
calico-node-xc55c                          1/1     Running   0          4m53s
coredns-657959df74-grflz                   1/1     Running   0          37m
coredns-657959df74-wbdl6                   1/1     Running   0          37m
dns-autoscaler-b5c786945-cbcv9             1/1     Running   0          37m
kube-apiserver-cp-master                   1/1     Running   0          36m
kube-controller-manager-cp-master          1/1     Running   1          39m
kube-proxy-2ckfd                           1/1     Running   0          38m
kube-proxy-hb8p2                           1/1     Running   0          38m
kube-proxy-nnh6d                           1/1     Running   0          38m
kube-proxy-p9srm                           1/1     Running   0          6m4s
kube-scheduler-cp-master                   1/1     Running   1          39m
metrics-server-5cd75b7749-57sc2            2/2     Running   0          37m
nodelocaldns-24vq4                         1/1     Running   0          6m4s
nodelocaldns-jjrjj                         1/1     Running   0          37m
nodelocaldns-kgzxb                         1/1     Running   0          37m
nodelocaldns-l9s47                         1/1     Running   0          37m

$ kubectl get pods -n kubeedge
NAME                         READY   STATUS    RESTARTS   AGE
cloudcore-758b4f4b97-4s57p   1/1     Running   0          3m49s
cloudcore-758b4f4b97-ndxbl   1/1     Running   0          3m49s
edgemesh-agent-9cmt2         1/1     Running   0          25s
edgemesh-agent-b8btq         1/1     Running   0          87s
edgemesh-agent-ntf24         1/1     Running   0          87s
edgemesh-agent-vhggk         1/1     Running   0          87s
edgemesh-agent-vzpdj         1/1     Running   0          87s
```

<br><br>

## <div id='3'> 3. K-PaaS 컨테이너 플랫폼 Edge 배포 삭제 (참고)
쉘 스크립트를 통해 K-PaaS 컨테이너 플랫폼 Edge 배포 삭제를 진행한다.
```
$ source reset-cp-edge.sh
```

<br><br>

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

<br><br>

[image 001]:images/edge-v1.2.png

[image 008]:../standalone/images/kpaas-cp-cluster-8.png
[image 009]:../standalone/images/kpaas-cp-cluster-9.png
[image 010]:../standalone/images/kpaas-cp-cluster-10.png
[image 011]:../standalone/images/kpaas-cp-cluster-11.png
[image 012]:../standalone/images/kpaas-cp-cluster-12.png
[image 013]:../standalone/images/kpaas-cp-cluster-13.png
[image 014]:../standalone/images/kpaas-cp-cluster-14.png
[image 015]:../standalone/images/kpaas-cp-cluster-15.png
[image 016]:../standalone/images/kpaas-cp-cluster-16.png
[image 017]:../standalone/images/kpaas-cp-cluster-17.png
[image 018]:../standalone/images/kpaas-cp-cluster-18.png
[image 019]:../standalone/images/kpaas-cp-cluster-19.png
[image 020]:../standalone/images/kpaas-cp-cluster-20.png

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 Edge 배포 가이드
