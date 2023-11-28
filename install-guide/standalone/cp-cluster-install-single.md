### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드

<br>

## Table of Contents

1. [문서 개요](#1)<br>
  1.1. [목적](#1.1)<br>
  1.2. [범위](#1.2)<br>
  1.3. [시스템 구성도](#1.3)<br>
  1.4. [참고자료](#1.4)

2. [K-PaaS 컨테이너 플랫폼 클러스터 설치](#2)<br>
  2.1. [Prerequisite](#2.1)<br>
  2.2. [SSH Key 생성 및 배포](#2.2)<br>
  2.3. [K-PaaS 컨테이너 플랫폼 클러스터 Deployment 다운로드](#2.3)<br>
  2.4. [K-PaaS 컨테이너 플랫폼 클러스터 설치 준비](#2.4)<br>
  2.5. [K-PaaS 컨테이너 플랫폼 클러스터 설치](#2.5)<br>
  2.6. [K-PaaS 컨테이너 플랫폼 클러스터 설치 확인](#2.6)

3. [K-PaaS 컨테이너 플랫폼 클러스터 삭제 (참고)](#3)

4. [Resource 생성 시 주의사항](#4)

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드) 는 기획자, 개발자, 운영자 지원 환경의 개방형 PaaS 플랫폼인 K-PaaS 컨테이너 플랫폼의 클러스터를 설치하는 방법을 기술하였다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 K-PaaS 컨테이너 플랫폼 환경의 기반이 되는 클러스터 설치를 단일 클라우드 환경 기준으로 작성하였다.

<br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 쿠버네티스 단일 클러스터 (Control Plane, Worker) 환경으로 구성되어 있다.<br>
K-PaaS 컨테이너 플랫폼 Deployment를 통해 쿠버네티스 단일 클러스터를 구성하고 각 리소스를 통해 K-PaaS 컨테이너 플랫폼 포털 환경을 배포하여 대시보드, 데이터베이스, 레파지토리 등의 환경을 제공한다.<br>
K-PaaS 컨테이너 플랫폼 클러스터에 필요한 인스턴스 환경으로는 아래 구성을 참고한다.

|인스턴스 종류|인스턴스 갯수|비고|
|---|---|---|
|Install|1개|Install 인스턴스 구성을 권장<br>Control Plane 인스턴스로 대체 가능|
|Control Plane|1개 이상|테스트 환경 1개<br>운영 환경 3개 이상|
|Worker|1~3개 이상|NFS 스토리지 사용 시 1개 이상<br>Rook-Ceph 스토리지 사용시 3개 이상|
|Storage|1개|NFS 스토리지 사용 시 필요|
|LoadBalancer|1~2개|HA Control Plane 구성 시 필요|

<br>

각 배포 유형 별 시스템 구성도는 다음과 같다.

단일 Control Plane, NFS 스토리지 구성

![image 001]

<br>

단일 Control Plane, Rook-Ceph 스토리지 구성

![image 002]

<br>

HA Control Plane, ETCD Stacked, NFS 스토리지 구성

![image 003]

<br>

HA Control Plane, ETCD External, NFS 스토리지 구성

![image 004]

<br>

HA Control Plane, ETCD Stacked, Rook-Ceph 스토리지 구성

![image 005]

<br>

HA Control Plane, ETCD External, Rook-Ceph 스토리지 구성

![image 006]

<br>

### <div id='1.4'> 1.4. 참고자료
> https://kubespray.io<br>
> https://github.com/kubernetes-sigs/kubespray

<br>

## <div id='2'> 2. K-PaaS 컨테이너 플랫폼 클러스터 설치

### <div id='2.1'> 2.1. Prerequisite
본 문서 (K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드) 는 K-PaaS 컨테이너 플랫폼 클러스터 배포 시 다음을 권고하고 있다.

- deb / rpm 호환 Linux OS를 실행하는 하나 이상의 머신 (Ubuntu)
- 머신 당 8G (최소사양) or 16G (권장사양) 이상의 RAM
- control-plane 노드로 사용하는 머신에 2 개 (최소사양) or 4 개 (권장사양) 이상의 CPU
- 클러스터의 모든 시스템 간의 완전한 네트워크 연결

<br>

K-PaaS 컨테이너 플랫폼 클러스터 설치를 위한 전제조건은 아래 기술하였다.

#### OS
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 OS 환경 정보는 다음과 같다. (***필수 확인***)

|지원 OS|OS 버전|
|---|---|
|Ubuntu|22.04|

<br>

#### Python 패키지
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 주요 Python 패키지 정보는 다음과 같다. (참고)

|Python 패키지|버전|
|---|---|
|ansible|7.6.0|
|cryptography|41.0.1|
|jinja2|3.1.2|
|jmespath|1.0.1|
|MarkupSafe|2.1.3|
|netaddr|0.8.0|
|pbr|5.11.1|
|ruamel.yaml|0.17.31|
|ruamel.yaml.clib|0.2.7|

<br>

#### 주요 소프트웨어
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 주요 소프트웨어 정보는 다음과 같다. (참고)

|주요 소프트웨어|버전|
|---|---|
|Kubespray|2.23.0|
|Kubernetes Native|1.27.5|
|CRI-O|1.27.1|
|MetalLB|0.13.9|
|Ingress Nginx Controller|1.8.2|
|Helm|3.12.3|
|Istio|1.19.0|
|Podman|3.4.4|
|Terraform|1.3.4|
|NFS Common|-|
|nfs-provisioner|4.0.2|
|Rook Ceph|1.12.3|
|Kubeflow|1.7.0|

<br>

#### 방화벽
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 방화벽 정보는 다음과 같다. (***필수 설정***)

Control Plane 노드

|프로토콜|포트|비고|
|---|---|---|
|TCP|111|NFS PortMapper|
|TCP|2049|NFS|
|TCP|2379-2380|etcd server client API|
|TCP|6443|Kubernetes API Server|
|TCP|10250|Kubelet API|
|TCP|10251|kube-scheduler|
|TCP|10252|kube-controller-manager|
|TCP|10255|Read-Only Kubelet API|
|UDP|4789|Calico networking VXLAN|

<br>

Worker 노드

|프로토콜|포트|비고|
|---|---|---|
|TCP|111|NFS PortMapper|
|TCP|2049|NFS|
|TCP|10250|Kubelet API|
|TCP|10255|Read-Only Kubelet API|
|TCP|30000-32767| NodePort Services|
|UDP|4789|Calico networking VXLAN|

<br>

#### 스토리지
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 스토리지 정보는 다음과 같다. (***필수 설정***)

NFS 스토리지 구성 시<br>
[NFS Server 설치 가이드](../nfs-server-install-guide.md)

Rook-Ceph 스토리지 구성 시<br>
Root Volume 이외에 추가 Volume을 각 Worker 노드에 사전에 할당해야 한다.

<br>

#### 로드밸런서
K-PaaS 컨테이너 플랫폼 클러스터를 HA Control Plane으로 구성할 경우 필요한 로드밸런서 정보는 다음과 같다. (***HA Control Plane 구성 시 필수 설정***)

Public 클라우드 환경의 경우 각 CSP에서 제공하는 로드밸런서를 생성한다.

NHN Cloud 환경 로드밸런서 생성 (예시)

Load Balancer 메뉴에서 "로드밸런서 생성" 버튼을 클릭한다.

![image 008]

<br>

로드밸런서 정보를 입력한 후 생성을 진행한다.

|항목|설명|비고|
|---|---|---|
|이름|로드밸런서 이름 입력||
|VPC|사용할 VPC 선택||
|서브넷|사용할 서브넷 선택||
|프로토콜|TCP 선택||
|로드밸런서 포트|6443 입력||
|인스턴스 포트|6443 입력||
|상태 확인 프로토콜|선택된 인스턴스 상태 확인에 사용할 프로토콜 입력||
|상태 확인 포트|선택된 인스턴스 상태 확인에 사용할 포트 입력||
|선택된 인스턴스|Control Plane 인스턴스 선택||

<br>

![image 009]

<br>

생성된 로드밸런서 선택 후 "플로팅 IP 관리" 버튼을 클릭한다.

![image 010]

<br>

플로팅 IP 연결을 진행한다.

![image 011]

<br>

Private 클라우드 환경의 경우 Keepalived, HAProxy를 이용하여 로드밸런서를 구성한다.

|환경|구성|비고|
|---|---|---|
|테스트 환경|Keepalived, HAProxy 단일 구성|로드밸런서 인스턴스 Private IP를 VIP로 사용|
|운영 환경|Keepalived, HAProxy HA 구성|별도 인터페이스 생성 후 인스턴스에 할당하여 VIP로 사용|

<br>

Private 클라우드 환경의 로드밸런서 구성은 아래 절차를 참고하여 진행한다.

로드밸런서 HA 구성의 경우 다음과 같이 VIP를 구성한다.

- 신규 인터페이스 생성
- 신규 생성한 인터페이스에 Public IP 할당
- 로드밸런서 인스턴스에 신규 생성한 인터페이스 할당

<br>

로드밸런서 HA 구성의 경우 2개 로드밸런서 인스턴스에서 Keepalived, HAProxy 설치 과정을 각각 진행한다.

<br>

로드밸런서 인스턴스에 다음과 같이 Keepalived 설치를 진행한다.
```
$ sudo su -

# apt-get update

# apt-get install -y keepalived

# echo 'net.ipv4.ip_nonlocal_bind=1' >> /etc/sysctl.conf
# echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
# sysctl -p
```

<br>

Keepalived 설정을 진행한다.
```
# vi /etc/keepalived/keepalived.conf
```

|환경변수|설명|비고|
|---|---|---|
|STATE|1번 로드밸런서 인스턴스에는 "MASTER", 2번 로드밸런서 인스턴스에는 "BACKUP"을 입력|단일 로드밸런서 인스턴스 구성의 경우 MASTER만 설정|
|INTERFACE_NAME|각 인스턴스의 쉘에서 ifconfig 입력 후 확인||
|VIP|로드밸런서 인스턴스에 할당한 인터페이스의 Private IP|테스트 환경의 경우 로드밸런서 인스턴스의 Private IP 지정<br>운영 환경의 경우 별도 구성한 VIP 지정|

```
vrrp_instance VI_1 {
  state {STATE}
  interface {{INTERFACE_NAME}}
  virtual_router_id 51
  priority 110
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  virtual_ipaddress {
    {{VIP}}
  }
}
```

<br>

Keepalived 서비스를 시작한다.
```
# systemctl start keepalived
# systemctl enable keepalived
```

<br>

로드밸런서 인스턴스에 다음과 같이 HAProxy 설치를 진행한다.
```
# apt-get install -y haproxy
```

<br>

HAProxy 설정을 진행한다. haproxy.cfg 파일 최하단에 다음 내용을 추가한다.
```
# vi /etc/haproxy/haproxy.cfg
```

|환경변수|설명|비고|
|---|---|---|
|VIP|로드밸런서 인스턴스에 할당한 인터페이스의 Private IP|테스트 환경의 경우 로드밸런서 인스턴스의 Private IP 지정<br>운영 환경의 경우 별도 구성한 VIP 지정|
|MASTER_NODE_IP{n}|각 Control Plane 노드의 Private IP|아래 예시는 3개의 Control Plane 노드 구성 시 이며 Control Plane 노드 구성에 따라 추가|

```
...
listen kubernetes-apiserver-https
  bind {{VIP}}:6443
  mode tcp
  option log-health-checks
  timeout client 3h
  timeout server 3h
  server master1 {{MASTER_NODE_IP1}}:6443 check check-ssl verify none inter 10000
  server master2 {{MASTER_NODE_IP2}}:6443 check check-ssl verify none inter 10000
  server master3 {{MASTER_NODE_IP3}}:6443 check check-ssl verify none inter 10000
  balance roundrobin
```

<br>

HAProxy 서비스를 재시작한다.
```
# systemctl restart haproxy
```

<br>

#### 쿠버네티스 로드밸런서 타입 서비스 External IP
K-PaaS 컨테이너 플랫폼 클러스터 서비스에 필요한 External IP 설정 정보는 다음과 같다. (***필수 설정***)

Ingress Nginx Controller 서비스의 External IP는 다음과 같이 설정한다.<br>
- 신규 인터페이스 생성
- 신규 생성한 인터페이스에 Public IP 할당
- 1번 Control Plane 노드에 신규 생성한 인터페이스 할당
- 아래 과정에서 설정할 환경변수 "INGRESS_NGINX_PRIVATE_IP"에 신규 생성한 인터페이스 Private IP 설정

<br>

### <div id='2.2'> 2.2. SSH Key 생성 및 배포
K-PaaS 컨테이너 플랫폼 클러스터 설치를 위해서는 SSH Key가 인벤토리의 모든 서버들에 복사되어야 한다.<br>
본 문서 (K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드) 에서는 RSA 공개키를 이용하여 SSH 접속 설정을 진행한다.

SSH Key 생성 및 배포 이후의 모든 설치과정은 **Install 인스턴스 또는 1번 Control Plane 노드**에서 진행한다.

**Install 인스턴스 또는 1번 Control Plane 노드**에서 RSA 공개키를 생성한다.
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

<br>

사용할 **Install, Control Plane, Worker 노드**에 공개키를 복사한다.
```
## 출력된 공개키 복사

$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5QrbqzV6g4iZT4iR1u+EKKVQGqBy4DbGqH7/PVfmAYEo3CcFGhRhzLcVz3rKb+C25mOne+MaQGynZFpZk4muEAUdkpieoo+B6r2eJHjBLopn5quWJ561H7EZb/GlfC5ThjHFF+hTf5trF4boW1iZRvUM56KAwXiYosLLRBXeNlub4SKfApe8ojQh4RRzFBZP/wNbOKr+Fo6g4RQCWrr5xQCZMK3ugBzTHM+zh9Ra7tG0oCySRcFTAXXoyXnJm+PFhdR6jbkerDlUYP9RD/87p/YKS1wSXExpBkEglpbTUPMCj+t1kXXEJ68JkMrVMpeznuuopgjHYWWD2FgjFFNkp ubuntu@cp-master
```

<br>

사용할 **Install, Control Plane, Worker 노드**의 authorized_keys 파일 본문의 마지막 부분(기존 본문 내용 아래 추가)에 공개키를 복사한다.
```
$ vi .ssh/authorized_keys

ex)
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRueywSiuwyfmCSecHu7iwyi3xYS1xigAnhR/RMg/Ws3yOuwbKfeDFUprQR24BoMaD360uyuRaPpfqSL3LS9oRFrj0BSaQfmLcMM1+dWv+NbH/vvq7QWhIszVCLzwTqlHrhgNsh0+EMhqc15KEo5kHm7d7vLc0fB5tZkmovsUFzp01Ceo9+Qye6+j+UM6ssxdTmatoMP3ZZKZzUPF0EZwTcGG6+8rVK2G8GhTqwGLj9E+As3GB1YdOvr/fsTAi2PoxxFsypNR4NX8ZTDvRdAUzIxz8wv2VV4mADStSjFpE7HWrzr4tZUjvvVFptU4LbyON9YY4brMzjxA7kTuf/e3j Generated-by-Nova
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5QrbqzV6g4iZT4iR1u+EKKVQGqBy4DbGqH7/PVfmAYEo3CcFGhRhzLcVz3rKb+C25mOne+MaQGynZFpZk4muEAUdkpieoo+B6r2eJHjBLopn5quWJ561H7EZb/GlfC5ThjHFF+hTf5trF4boW1iZRvUM56KAwXiYosLLRBXeNlub4SKfApe8ojQh4RRzFBZP/wNbOKr+Fo6g4RQCWrr5xQCZMK3ugBzTHM+zh9Ra7tG0oCySRcFTAXXoyXnJm+PFhdR6jbkerDlUYP9RD/87p/YKS1wSXExpBkEglpbTUPMCj+t1kXXEJ68JkMrVMpeznuuopgjHYWWD2FgjFFNkp ubuntu@cp-master
```

<br>

### <div id='2.3'> 2.3. K-PaaS 컨테이너 플랫폼 클러스터 Deployment 다운로드

> 2.3.부터는 **Install 인스턴스 또는 1번 Control Plane 노드**에서만 진행.

<br>

K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 Deployment를 다운로드 후 설치 작업 경로로 이동하여 설치를 진행한다.

> K-PaaS 컨테이너 플랫폼 클러스터 Deployment Download URL : https://github.com/k-paas/cp-deployment

<br>

git clone 명령을 통해 HOME 디렉토리 경로에서 K-PaaS 컨테이너 플랫폼 클러스터 Deployment 다운로드를 진행한다.
```
$ git clone https://github.com/K-PaaS/cp-deployment.git -b branch_v1.5.x
```

<br>

### <div id='2.4'> 2.4. K-PaaS 컨테이너 플랫폼 클러스터 설치 준비
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 환경변수를 사전 정의 후 쉘 스크립트를 통해 설치를 진행한다.

K-PaaS 컨테이너 플랫폼 클러스터 설치경로 이동한다.
```
$ cd ~/cp-deployment/single
```

<br>

K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 환경변수를 정의한다.
```
$ vi cp-cluster-vars.sh
```

<br>

K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 정보를 입력한다.

Control Plane

|환경변수|설명|비고|
|---|---|---|
|KUBE_CONTROL_HOSTS|Control Plane 노드의 갯수||
|ETCD_TYPE|ETCD 배포 방식<br>external : 별도의 노드에 ETCD 구성<br>stacked : Control Plane 노드에 ETCD 구성|Control Plane 노드의 갯수가 1개 이상일 경우 설정|
|LOADBALANCER_DOMAIN|사전에 구성한 로드밸런서의 IP 또는 Domain 정보|Control Plane 노드의 갯수가 1개 이상일 경우 설정|
|ETCD1_NODE_HOSTNAME|ETCD 1번 노드의 호스트명|Control Plane 노드의 갯수가 1개 이상일 경우 설정<br>ETCD 배포방식이 external 일 경우 설정|
|ETCD1_NODE_PRIVATE_IP|ETCD 1번 노드의 Private IP|Control Plane 노드의 갯수가 1개 이상일 경우 설정<br>ETCD 배포방식이 external 일 경우 설정|
|ETCD{n}_NODE_HOSTNAME|ETCD n번 노드의 호스트명|Control Plane 노드의 갯수가 1개 이상일 경우 설정<br>ETCD 배포방식이 external 일 경우 설정<br>KUBE_CONTROL_HOSTS 갯수만큼 설정|
|ETCD{n}_NODE_PRIVATE_IP|ETCD n번 노드의 Private IP|Control Plane 노드의 갯수가 1개 이상일 경우 설정<br>ETCD 배포방식이 external 일 경우 설정<br>KUBE_CONTROL_HOSTS 갯수만큼 설정|
|MASTER1_NODE_HOSTNAME|Control Plane 1번 노드의 호스트명||
|MASTER1_NODE_PUBLIC_IP|Control Plane 1번 노드의 Public IP|Control Plane 1번 노드만 Public IP 정보 필요|
|MASTER1_NODE_PRIVATE_IP|Control Plane 1번 노드의 Private IP||
|MASTER{n}_NODE_HOSTNAME|Control Plane n번 노드의 호스트명|Control Plane 노드의 갯수가 1개 이상일 경우 설정<br>KUBE_CONTROL_HOSTS 갯수만큼 설정|
|MASTER{n}_NODE_PRIVATE_IP|Control Plane n번 노드의 Private IP|Control Plane 노드의 갯수가 1개 이상일 경우 설정<br>KUBE_CONTROL_HOSTS 갯수만큼 설정|

<br>

Worker

|환경변수|설명|비고|
|---|---|---|
|KUBE_WORKER_HOSTS|Worker 노드의 갯수||
|WORKER1_NODE_HOSTNAME|Worker 1번 노드의 호스트명||
|WORKER1_NODE_PRIVATE_IP|Worker 1번 노드의 Private IP||
|WORKER{n}_NODE_HOSTNAME|Worker n번 노드의 호스트명|KUBE_WORKER_HOSTS 갯수만큼 설정|
|WORKER{n}_NODE_PRIVATE_IP|Worker n번 노드의 Private IP|KUBE_WORKER_HOSTS 갯수만큼 설정|

<br>

Storage

|환경변수|설명|비고|
|---|---|---|
|STORAGE_TYPE|Storage 정보<br>nfs : NFS 스토리지<br>rook-ceph : Rook Ceph 스토리지||
|NFS_SERVER_PRIVATE_IP|NFS Server 인스턴스의 Private IP|STORAGE_TYPE 값 nfs 일 경우 설정|

<br>

LoadBalancer Service

|환경변수|설명|비고|
|---|---|---|
|METALLB_IP_RANGE|MetalLB에서 사용할 Private IP 대역||
|INGRESS_NGINX_PRIVATE_IP|MetalLB를 통해 Ingress Nginx Controller Service에서 사용할 Private IP|METALLB_IP_RANGE 값과 중복되지 않도록 설정|

<br>

```
#!/bin/bash

# Control Plane Node Count Variable (eg. 1, 3, 5 ...)
export KUBE_CONTROL_HOSTS=

# if KUBE_CONTROL_HOSTS > 1 (eg. external, stacked)
export ETCD_TYPE=

# if KUBE_CONTROL_HOSTS > 1
# HA Control Plane LoadBalanncer IP or Domain
export LOADBALANCER_DOMAIN=

# if ETCD_TYPE=external
# The number of ETCD node variable is set equal to the number of KUBE_CONTROL_HOSTS
export ETCD1_NODE_HOSTNAME=
export ETCD1_NODE_PRIVATE_IP=
...
export ETCD{n}_NODE_HOSTNAME=
export ETCD{n}_NODE_PRIVATE_IP=
...

# Master Node Info Variable
# The number of MASTER node variable is set equal to the number of KUBE_CONTROL_HOSTS
export MASTER1_NODE_HOSTNAME=
export MASTER1_NODE_PUBLIC_IP=
export MASTER1_NODE_PRIVATE_IP=
...
export MASTER{n}_NODE_HOSTNAME=
export MASTER{n}_NODE_PRIVATE_IP=
...

# Worker Node Count Variable
export KUBE_WORKER_HOSTS=

# Worker Node Info Variable
# The number of Worker node variable is set equal to the number of KUBE_WORKER_HOSTS
export WORKER1_NODE_HOSTNAME=
export WORKER1_NODE_PRIVATE_IP=
...
export WORKER{n}_NODE_HOSTNAME=
export WORKER{n}_NODE_PRIVATE_IP=
...

# Storage Variable (eg. nfs, rook-ceph)
export STORAGE_TYPE=

# if STORATE_TYPE=nfs
export NFS_SERVER_PRIVATE_IP=

# MetalLB Variable (eg. 192.168.0.150-192.168.0.160)
export METALLB_IP_RANGE=

# MetalLB Ingress Nginx Controller LoadBalancer Service External IP
export INGRESS_NGINX_PRIVATE_IP=
```

<br>

### <div id='2.5'> 2.5. K-PaaS 컨테이너 플랫폼 클러스터 설치
쉘 스크립트를 통해 필요 패키지 설치, 클러스터 설치 환경변수 설정, Ansible playbook을 통한 K-PaaS 컨테이너 플랫폼 클러스터 설치를 순차적으로 진행한다.

```
$ source deploy-cp-cluster.sh
```

<br>

### <div id='2.6'> 2.6. K-PaaS 컨테이너 플랫폼 클러스터 설치 확인
Kubernetes 노드 및 kube-system 네임스페이스의 Pod를 확인하여 K-PaaS 컨테이너 플랫폼 클러스터 설치를 확인한다.<br>
구성에 따라 조회되는 노드 및 Pod의 정보는 다를 수 있으며 아래는 단일 Control Plane 구성으로 배포했을때의 조회 결과이다. 

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

## <div id='3'> 3. K-PaaS 컨테이너 플랫폼 클러스터 삭제 (참고)
쉘 스크립트를 통해 K-PaaS 컨테이너 플랫폼 클러스터 삭제를 진행한다.

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

[image 001]:images/kpaas-cp-cluster-1.png
[image 002]:images/kpaas-cp-cluster-2.png
[image 003]:images/kpaas-cp-cluster-3.png
[image 004]:images/kpaas-cp-cluster-4.png
[image 005]:images/kpaas-cp-cluster-5.png
[image 006]:images/kpaas-cp-cluster-6.png
[image 008]:images/kpaas-cp-cluster-8.png
[image 009]:images/kpaas-cp-cluster-9.png
[image 010]:images/kpaas-cp-cluster-10.png
[image 011]:images/kpaas-cp-cluster-11.png

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드