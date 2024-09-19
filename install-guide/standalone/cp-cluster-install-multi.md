### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드

<br><br>

## Table of Contents

1. [문서 개요](#1)<br>
  1.1. [목적](#1.1)<br>
  1.2. [범위](#1.2)<br>
  1.3. [시스템 구성도](#1.3)<br>
  1.4. [참고자료](#1.4)

2. [K-PaaS 컨테이너 플랫폼 클러스터 설치](#2)<br>
  2.1. [Prerequisite](#2.1)<br>
  2.1.1. [OS](#2.1.1)<br>
  2.1.2. [Python 패키지](#2.1.2)<br>
  2.1.3. [주요 소프트웨어](#2.1.3)<br>
  2.1.4. [방화벽](#2.1.4)<br>
  2.1.5. [스토리지](#2.1.5)<br>
  2.1.6. [Ingress Nginx 서비스 설정](#2.1.6)<br>
  2.1.6.1. [Control Plane 노드 추가 인터페이스](#2.1.6.1)<br>
  2.1.6.2. [클라우드 로드밸런서 서비스](#2.1.6.2)<br>
  2.1.7. [HA Control Plane 로드밸런서](#2.1.7)<br>
  2.1.7.1. [클라우드 로드밸런서 서비스](#2.1.7.1)<br>
  2.1.7.2. [HAProxy](#2.1.7.2)<br>
  2.2. [SSH Key 생성 및 배포](#2.2)<br>
  2.3. [K-PaaS 컨테이너 플랫폼 클러스터 Deployment 다운로드](#2.3)<br>
  2.4. [K-PaaS 컨테이너 플랫폼 클러스터 설치 준비](#2.4)<br>
  2.5. [K-PaaS 컨테이너 플랫폼 클러스터 설치](#2.5)<br>
  2.6. [K-PaaS 컨테이너 플랫폼 클러스터 설치 확인](#2.6)

3. [K-PaaS 컨테이너 플랫폼 클러스터 삭제 (참고)](#3)

4. [Resource 생성 시 주의사항](#4)

<br><br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드) 는 기획자, 개발자, 운영자 지원 환경의 개방형 PaaS 플랫폼인 K-PaaS 컨테이너 플랫폼의 클러스터를 설치하는 방법을 기술하였다.

<br><br>

### <div id='1.2'> 1.2. 범위
설치 범위는 K-PaaS 컨테이너 플랫폼 환경의 기반이 되는 클러스터 설치를 **`멀티 클라우드`** 환경 기준으로 작성하였다.

<br><br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 쿠버네티스 **`멀티 클러스터`** (Control Plane, Worker) 환경으로 구성되어 있다.

<br>

K-PaaS 컨테이너 플랫폼 Deployment를 통해 쿠버네티스 **`멀티 클러스터`** 를 구성하고 각 리소스를 통해 K-PaaS 컨테이너 플랫폼 포털 환경을 배포하여 대시보드, 데이터베이스, 레파지토리 등의 환경을 제공한다.

<br>

K-PaaS 컨테이너 플랫폼 클러스터에 필요한 인스턴스 환경으로는 아래 구성을 참고한다.

> 멀티 클라우드 배포에서는 Install 인스턴스를 제외한 나머지 인스턴스는 클러스터마다 필요에 따라 별도 구성한다.

<br>

|인스턴스 종류|인스턴스 갯수|필수|비고|
|---|---|---|---|
|Install|1개||Install 인스턴스 구성을 권장<br>Control Plane 인스턴스로 대체 가능|
|Control Plane|각 클라우드 당 1개 이상|O|테스트 환경 1개<br>운영 환경 3개 이상|
|Worker|각 클라우드 당 1~3개 이상|O|NFS 스토리지 사용 시 1개 이상<br>Rook-Ceph 스토리지 사용시 3개 이상|
|Storage|각 클라우드 당 1개||NFS 스토리지 사용 시 필요|
|LoadBalancer|각 클라우드 당 1~2개||Private 클라우드 HA Control Plane 구성 시 필요|

<br>

각 배포 유형 별 시스템 구성도는 다음과 같다.

<br>

<details>
<summary>시스템 구성도</summary>
<br>

***[ 단일 Control Plane, NFS 스토리지 구성 ]***

![image 001]

<br><br>

***[ 단일 Control Plane, Rook-Ceph 스토리지 구성 ]***

![image 002]

<br><br>

***[ HA Control Plane, ETCD Stacked, NFS 스토리지 구성 ]***

![image 003]

<br><br>

***[ HA Control Plane, ETCD External, NFS 스토리지 구성 ]***

![image 004]

<br><br>

***[ HA Control Plane, ETCD Stacked, Rook-Ceph 스토리지 구성 ]***

![image 005]

<br><br>

***[ HA Control Plane, ETCD External, Rook-Ceph 스토리지 구성 ]***

![image 006]

<br><br>

> 멀티 클라우드 환경에서의 각 클러스터 구성도는 단일 클라우드 환경과 동일하며, 아래는 멀티 클라우드 환경에서의 클러스터간 서비스 흐름에 대한 구성도이다.

<br><br>

***[ 멀티 클라우드 환경 클러스터간 서비스 흐름 구성 ]***

![image 007]

</details>

<br><br>

### <div id='1.4'> 1.4. 참고자료
> https://kubespray.io<br>
> https://github.com/kubernetes-sigs/kubespray

<br><br>

## <div id='2'> 2. K-PaaS 컨테이너 플랫폼 클러스터 설치

### <div id='2.1'> 2.1. Prerequisite
본 문서 (K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드) 는 K-PaaS 컨테이너 플랫폼 클러스터 배포 시 다음을 권고하고 있다.

- deb / rpm 호환 Linux OS를 실행하는 하나 이상의 머신 (Ubuntu)
- 머신 당 8G (최소사양) or 16G (권장사양) 이상의 RAM
- control-plane 노드로 사용하는 머신에 2 개 (최소사양) or 4 개 (권장사양) 이상의 CPU
- 클러스터의 모든 시스템 간의 완전한 네트워크 연결

<br>

K-PaaS 컨테이너 플랫폼 클러스터 설치를 위한 전제조건은 아래 기술하였다.

<br><br>

### <div id='2.1.1'> 2.1.1. OS (***`필수 확인`***)
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 OS 환경 정보는 다음과 같다.

<br>

|지원 OS|버전|
|---|---|
|Ubuntu|22.04|

<br>

각 CSP 별 Ubuntu 22.04 기준 인스턴스 생성 시 아래 사항을 진행한다.

<br>

<details>
<summary>KT 클라우드</summary>
<br>
ubuntu 22.04 이미지로 인스턴스 생성 시 Last password change 일자가 2023-07-04 로 설정되어 일정시간 경과 후 ssh 접속 시 패스워드 방식의 로그인을 시도한다.

<br>

ubuntu 계정에는 기본적으로 패스워드가 설정되어 있지 않기 때문에 인스턴스에 접근할수 없는 이슈가 발생하므로 현재일 기준으로 변경을 진행한다.

```
$ sudo chage -d 20xx-xx-xx ubuntu
```

<br>

</details>

<br>

<details>
<summary>Naver 클라우드</summary>
<br>
메인 계정으로 인스턴스 생성 시 root, 서브 계정으로 인스턴스 생성 시 ncloud 계정을 기본으로 사용한다.
컨테이너 플랫폼의 경우 ubuntu 계정 사용을 기본값으로 하기때문에 ubuntu 계정 생성 절차를 진행한다.

<br>

Install 인스턴스 또는 첫번째 Control Plane 노드 인스턴스에서 아래 과정을 진행한다.

```
$ sudo useradd -m -s /bin/bash ubuntu
$ echo "ubuntu ALL=(ALL) NOPASSWD: ALL" | sudo tee -a /etc/sudoers

$ sudo mkdir -p /home/ubuntu/.ssh
$ sudo ssh-keygen -t rsa -m PEM -N '' -f /home/ubuntu/.ssh/id_rsa
$ sudo cat /home/ubuntu/.ssh/id_rsa.pub | sudo tee -a /home/ubuntu/.ssh/authorized_keys
$ sudo chown -R ubuntu:ubuntu /home/ubuntu/.ssh
```

<br>

인스턴스에 접근할 로컬 환경에 개인키를 복사한다.

```
## 출력된 개인키 복사

$ sudo cat /home/ubuntu/.ssh/id_rsa
```

<br>

공개키를 조회 및 복사한다.

```
## 출력된 공개키 복사

$ sudo cat /home/ubuntu/.ssh/id_rsa.pub
```

<br>


그 외 인스턴스에서 아래 과정을 진행한다.

```
$ sudo useradd -m -s /bin/bash ubuntu
$ echo "ubuntu ALL=(ALL) NOPASSWD: ALL" | sudo tee -a /etc/sudoers

$ sudo mkdir -p /home/ubuntu/.ssh
$ echo "{{ 공개키 }}" | sudo tee -a /home/ubuntu/.ssh/authorized_keys
$ sudo chown -R ubuntu:ubuntu /home/ubuntu/.ssh
```

<br>

해당 과정 진행 후, 생성한 ***`ubuntu 계정으로 인스턴스에 접속`*** 하여 이후 설치 과정을 진행한다.

<br>

</details>

<br><br>

### <div id='2.1.2'> 2.1.2. Python 패키지 (참고)
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 주요 Python 패키지 정보는 다음과 같다.

<br>

|Python 패키지|버전|
|---|---|
|ansible|9.5.1|
|cryptography|42.0.7|
|jinja2|3.1.4|
|jmespath|1.0.1|
|MarkupSafe|2.1.5|
|netaddr|1.2.1|
|pbr|6.0.0|
|ruamel.yaml|0.18.6|
|ruamel.yaml.clib|0.2.8|
|jsonchema|4.22.0|

<br><br>

### <div id='2.1.3'> 2.1.3. 주요 소프트웨어 (참고)
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 주요 소프트웨어 정보는 다음과 같다.

<br>

|주요 소프트웨어|버전|
|---|---|
|Kubespray|2.25.0|
|Kubernetes Native|1.29.5|
|CRI-O|1.29.1|
|Calico|3.27.3|
|MetalLB|0.13.9|
|Ingress Nginx Controller|1.11.1|
|Helm|3.14.2|
|Istio|1.22.3|
|Podman|3.4.4|
|OpenTofu|1.8.1|
|NFS Common|-|
|nfs-subdir-external-provisioner|4.0.2|
|Rook Ceph|1.14.9|
|Kubeflow|1.7.0|

<br><br>

### <div id='2.1.4'> 2.1.4. 방화벽 (***`필수 설정`***)
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 방화벽 정보는 다음과 같다.

> 멀티 클라우드 배포에서 Istio East-West Gateway 방화벽이 추가되었다.

<br>

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
|TCP|30000-32767| NodePort Services|
|UDP|4789|Calico networking VXLAN|
|TCP|15021|Istio East-West Gateway Status|
|TCP|15443|Istio East-West Gateway mTLS|
|TCP|15012|Istio East-West Gateway istiod|
|TCP|15017|Istio East-West Gateway webhook|

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
|TCP|15021|Istio East-West Gateway Status|
|TCP|15443|Istio East-West Gateway mTLS|
|TCP|15012|Istio East-West Gateway istiod|
|TCP|15017|Istio East-West Gateway webhook|

<br><br>

### <div id='2.1.5'> 2.1.5. 스토리지 (***`필수 설정`***)
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 스토리지 정보는 다음과 같다.

> 멀티 클라우드 배포에서 스토리지는 2개 클라우드 환경에 각각 별도로 구성한다.

<br>

|지원 스토리지|비고|
|---|---|
|NFS Server|신규 인스턴스 생성|
|Rook-Ceph|기존 인스턴스에 볼륨 추가|

<br>

NFS 스토리지 구성 시<br>
[NFS Server 설치 가이드](../nfs-server-install-guide.md)

<br>

Rook-Ceph 스토리지 구성 시<br>
Root Volume 이외에 ***`추가 Volume을 각 Worker 노드에 사전에 할당`*** 해야 한다.

<br><br>

### <div id='2.1.6'> 2.1.6. Ingress Nginx 서비스 설정 (***`필수 설정`***)
K-PaaS 컨테이너 플랫폼 서비스 구성을 위해 필요한 Ingress Nginx 서비스 설정 정보는 다음과 같다.

<br>

|서비스|설명|비고|
|---|---|---|
|Ingress Nginx Controller|K-PaaS 컨테이너 플랫폼 서비스를<br>Ingress로 외부 노출하기 위한 서비스|각 클라우드에 ***`1개 인터페이스 또는 1개 로드밸런서`*** 생성 필요<br>Public IP 할당 필요|
|Istio EastWest Gateway|멀티 클라우드간 서비스 통신을 위한<br>게이트웨이 서비스|각 클라우드에 ***`1개 인터페이스 또는 1개 로드밸런서`*** 생성 필요<br>Public IP 할당 필요|

<br>

> 멀티 클라우드 배포에서 서비스에 할당할 인터페이스 또는 로드밸런서 서비스는 2개 클라우드 환경에 각각 별도로 구성한다.

<br>

K-PaaS 컨테이너 플랫폼 클러스터에서는 MetalLB를 통해 로드밸런서 타입 서비스의 External IP를 할당한다.<br>
해당 External IP로 외부 통신 및 서비스를 지원하기 위해 ***`다음 두가지 중 한가지 방식을 선택`*** 하여 설정을 진행한다.

<br>

|방식|설명|비고|
|---|---|---|
|인터페이스 추가|1개 Control Plane 노드에 Public IP가 할당된 신규 인터페이스 추가|Public IP 사용에 대한 비용만 발생<br>HA 구성에서 해당 노드 장애 시 Ingress Nginx 서비스 외부 접근 불가|
|로드밸런서 생성|Public IP가 할당된 로드밸런서 서비스 생성|로드밸런서 서비스에 대한 비용 추가 발생<br>HA 구성에서 일부 Control Plane 노드 장애 발생시에도 Ingress Nginx 서비스 정상<br>운영 환경에서 권장|

<br><br>

### <div id='2.1.6.1'> 2.1.6.1. Control Plane 노드 추가 인터페이스

<br>

<details>
<summary>NHN 클라우드 인터페이스 추가</summary>
<br>
1. <b><code>Network > Network Interface</code></b> 메뉴로 이동하여 "네트워크 인터페이스 생성" 버튼을 클릭한다.
<br><br>

![image if nhn 001]

<br><br>
2. 이름 입력, Control Plane 노드와 동일한 네트워크 VPC, 서브넷 선택 후 "확인" 버튼을 클릭한다.

![image if nhn 002]

<br><br>
3. 생성한 인터페이스 선택 후 "플로팅 IP 관리" 버튼을 클릭한다.

![image if nhn 003]

<br><br>
4. 플로팅 IP 생성 또는 기존에 생성된 플로팅 IP 선택 후 "연결" 버튼을 클릭한다.

![image if nhn 004]

<br><br>
5. <b><code>Compute > Instance</code></b> 메뉴로 이동하여 Control Plane 노드 (HA Control Plane 구성 시 인터페이스 연결 추가 할 Control Plane 노드)를 선택 후 "인스턴스 중지" 버튼 클릭한다.

![image if nhn 005]

<br><br>
6. 하단 네트워크 탭에서 "네트워크 인터페이스 연결 추가" 버튼 클릭한다.

![image if nhn 006]

<br><br>
7. 기존 네트워크 인터페이스 지정 선택 후 생성한 인터페이스를 선택 후 "확인" 버튼을 클릭한다.

![image if nhn 007]

<br><br>
8. "인스턴스 시작" 버튼을 클릭한다.

![image if nhn 008]

<br>

</details>

<br>

<details>
<summary>KT 클라우드 인터페이스 추가</summary>
<br>
1. <b><code>Servers > Virtual IP</code></b> 메뉴에서 "Virtual IP 생성" 버튼을 클릭한다.

<br><br>
2. Control Plane 노드와 동일한 네트워크 Zone, Tier 선택, Name 입력 후 "생성" 버튼을 클릭한다.

![image if kt 001]

<br><br>
3. Virtual IP 선택 후 "연결" 버튼을 클릭하여 Control Plane VM에 Virtual IP를 연결한다.

![image if kt 002]

<br><br>
4. <b><code>Servers > Networking</code></b> 메뉴에서 "IP 생성" 버튼 클릭하여 Control Plane 노드와 동일한 네트워크 Zone에 Public IP를 생성한다.

![image if kt 003]

<br><br>
5. 생성된 Public IP 선택 후 "접속 설정" 버튼 클릭하여 Virtual IP 선택 후 80, 443 포트에 대한 Port Forwarding 설정을 진행한다.

![image if kt 004]

![image if kt 005]

<br><br>
6. "방화벽 설정" 버튼 클릭하여 등록한 접속 설정으로 방화벽을 설정한다. (Source CIDR, Destination CIDR은 설치하는 각각의 환경에 맞춰 설정)

![image if kt 006]

<br><br>
7. Control Plane 노드에서 다음 명령어를 실행한다. (HA Control Plane 구성 시 Virtual IP 연결한 Control Plane 노드에서 실행)

```
## 인터페이스 조회
## 인터페이스명 예 : eth0, ens3 ...

$ sudo ifconfig
```

```
## 인터페이스 추가
## 예시 : sudo ifconfig ens3:1 192.168.0.10 up

$ sudo ifconfig {{ 인터페이스명 }}:1 {{ Virtual IP (Private) }} up
```

<br>

</details>

<br>

<details>
<summary>Naver 클라우드 인터페이스 추가</summary>
<br>
Naver 클라우드는 정책 상 1개의 인스턴스에 2개의 Public IP 할당이 불가능하다.<br>
따라서, 아래 기술한 Naver 클라우드의 로드밸런서 서비스 생성 및 설정 진행이 필요하다.

<br>

</details>

<br><br>

### <div id='2.1.6.2'> 2.1.6.2. 클라우드 로드밸런서 서비스
> 클라우드 로드밸런서 서비스 생성 예시는 Ingress Nginx Controller 서비스를 기준으로 작성되었으며,<br>
> Istio EastWest Gateway 서비스의 경우에는 [2.1.4. 방화벽](#2.1.4) 정보를 참고하여 해당 서비스의 포트와 노드포트의 정보를 확인 후 설정을 진행한다.

<br>

<details>
<summary>NHN 클라우드 로드밸런서 서비스 생성</summary>
<br>

로드밸런서 서비스 생성 전 로드밸런서 서비스에 할당할 ***`Public IP만 우선 생성`*** 한 후, ***`클러스터 배포 완료 이후`*** 에 ***`로드밸런서 서비스를 생성`*** 한다.

<br>
1. <b><code>Network > Floating IP</code></b> 메뉴에서 "플로팅 IP 생성" 버튼 클릭하여 Public IP를 생성한다.
<br><br>

![image lb nhn 001]

<br>

로드밸런서 생성 시 서비스에 할당된 포트와 노드포트 정보 확인 이후 로드밸런서 설정이 가능하기때문에, 아래 기술한 로드밸런서 생성 및 설정 과정은 [2.6. K-PaaS 컨테이너 플랫폼 클러스터 설치 확인](#2.6) 단계 이후에 진행한다.

<br><br>
2. <b><code>Network > Load Balancer > 관리</code></b> 메뉴에서 "로드 밸런서 생성" 버튼을 클릭, 로드 밸런서 생성 모드 선택 팝업에서 "L4 라우팅"을 선택한다.

![image lb nhn 002]

<br><br>
3. 설정 정보를 입력한다.

|항목|설명|비고|
|---|---|---|
|이름|로드밸런서 이름 입력||
|VPC|Control Plane 노드와 동일한 네트워크 VPC 선택||
|서브넷|Control Plane 노드와 동일한 Public 서브넷 선택||

<br>

![image lb nhn 003]

<br><br>
4. 80 포트의 리스너, 맴버 그룹 정보를 입력한다.

리스너

|항목|설명|비고|
|---|---|---|
|이름|리스너 이름 입력||
|프로토콜|HTTP 선택||
|로드 밸런서 포트|80 입력||

<br>

맴버 그룹

|항목|설명|비고|
|---|---|---|
|이름|맴버 그룹 이름을 입력||
|프로토콜|HTTP 선택||
|포트|Ingress Nginx 서비스의 80 포트에 할당된 노드포트 값 입력||
|상태 확인 프로토콜|TCP 선택||
|상태 확인 포트|인스턴스 상태체크가 가능한 포트 입력|예 : 인스턴스 SSH 포트 (TCP 22)|
|맴버 목록|전체 노드 인스턴스 추가||

<br>

> Ingress Nginx 서비스 포트 확인<br>

```
$ kubectl get svc ingress-nginx-controller -n ingress-nginx
```

<br>

![image lb nhn 004]

<br><br>
5. "리스너추가" 버튼을 클릭 후 443 포트의 리스너, 맴버 그룹 정보를 입력한다.

리스너

|항목|설명|비고|
|---|---|---|
|이름|리스너 이름 입력||
|프로토콜|HTTPS 선택||
|로드 밸런서 포트|443 입력||

<br>

맴버 그룹

|항목|설명|비고|
|---|---|---|
|이름|맴버 그룹 이름을 입력||
|프로토콜|HTTPS 선택||
|포트|Ingress Nginx 서비스의 443 포트에 할당된 노드포트 값 입력||
|상태 확인 프로토콜|TCP 선택||
|상태 확인 포트|인스턴스 상태체크가 가능한 포트 입력|예 : 인스턴스 SSH 포트 (TCP 22)|
|맴버 목록|전체 노드 인스턴스 추가||

<br>

![image lb nhn 005]

<br><br>
6. 하단의 "로드 밸런서 생성" 버튼을 클릭한다.

<br><br>
7. 생성한 로드밸런서 선택 후 "플로팅 IP 관리" 버튼을 클릭한다.

<br><br>
8. 기존에 생성된 플로팅 IP 선택(1번 과정에서 생성), 생성한 인터페이스 선택 후 "연결" 버튼을 클릭한다.

![image lb nhn 006]

<br>

</details>

<br>

<details>
<summary>KT 클라우드 로드밸런서 서비스 생성</summary>
<br>

로드밸런서 서비스 생성 전 로드밸런서 서비스에 할당할 ***`Public IP만 우선 생성`*** 한 후, ***`클러스터 배포 완료 이후`*** 에 ***`로드밸런서 서비스를 생성`*** 한다.

<br>
1. <b><code>Servers > Networking</code></b> 메뉴에서 "IP 생성" 버튼을 클릭하여 Public IP를 생성한다.
<br><br>

![image if kt 003]

<br>

로드밸런서 생성 시 서비스에 할당된 포트와 노드포트 정보 확인 이후 로드밸런서 설정이 가능하기때문에, 아래 기술한 로드밸런서 생성 및 설정 과정은 [2.6. K-PaaS 컨테이너 플랫폼 클러스터 설치 확인](#2.6) 단계 이후에 진행한다.

<br><br>
2. <b><code>Load Balancer > Load Balancer 관리</code></b> 메뉴에서 "Load Balancer 생성" 버튼을 클릭한다.

![image lb kt 001]

<br><br>
3. 로드밸런서 정보를 입력 후 "확인" 버튼을 클릭한다.

|항목|설명|비고|
|---|---|---|
|Zone|Control Plane 노드와 동일한 네트워크 Zone 선택|VPC와 동일 개념|
|Tier|Control Plane 노드와 동일한 네트워크 Tier 선택|서브넷과 동일 개념|
|Name|로드밸런서 이름 입력||
|Service IP / Port|신규 IP 할당, 80 포트 입력||
|Service Type|HTTP 선택||

<br>

![image lb kt 002]

<br><br>
4. "Load Balancer 생성" 버튼을 클릭한다.

<br><br>
5. 로드밸런서 정보를 입력 후 "확인" 버튼을 클릭한다.

|항목|설명|비고|
|---|---|---|
|Zone|Control Plane 노드와 동일한 네트워크 Zone 선택|VPC와 동일 개념|
|Tier|Control Plane 노드와 동일한 네트워크 Tier 선택|서브넷과 동일 개념|
|Name|로드밸런서 이름 입력||
|Service IP / Port|80 포트로 생성한 로드밸런서 Private IP 선택, 443 포트 입력||
|Service Type|HTTPS(Bridge) 선택||

<br>

![image lb kt 003]

<br><br>
6. 생성한 로드밸런서 선택 후 "VM 연결/해제" 버튼을 클릭한다. (80, 443 포트 로드밸런서 모두)

<br><br>
7. 정보를 입력 후 "추가" 버튼을 클릭한다.

|항목|설명|비고|
|---|---|---|
|Tier|Control Plane 노드와 동일한 네트워크 Tier 선택||
|VM|클러스터 전체 노드를 순차적으로 선택||
|Public Port|80, 443 포트에 할당된 노드포트 정보 입력||

<br>

> Ingress Nginx 서비스 포트 확인<br>

```
$ kubectl get svc ingress-nginx-controller -n ingress-nginx
```

<br>

![image lb kt 004]

![image lb kt 005]

![image lb kt 006]

<br><br>
8. <b><code>Servers > Networking</code></b> 메뉴로 이동하여 기존에 생성한 Public IP 선택(1번 과정에서 생성), "Static NAT" 버튼을 클릭하여 생성한 로드밸런서 중 1개를 선택한다.

![image lb kt 004]

![image lb kt 005]

![image lb kt 006]

<br><br>
9. "방화벽 설정" 버튼 클릭하여 등록한 Static NAT 설정으로 방화벽을 등록한다.

![image lb kt 008]

<br>

</details>

<br>

<details>
<summary>Naver 클라우드 로드밸런서 서비스 생성</summary>
<br>

로드밸런서 서비스 생성 전 로드밸런서 서비스에 할당할 ***`Public IP만 우선 생성`*** 한 후, ***`클러스터 배포 완료 이후`*** 에 ***`로드밸런서 서비스를 생성`*** 한다.

<br>
1. <b><code>Server > Public IP</code></b> 메뉴에서 "공인 IP 신청" 버튼 클릭하여 미할당 Public IP를 생성한다.
<br><br>

![image lb naver 001]

<br>

로드밸런서 생성 시 서비스에 할당된 포트와 노드포트 정보 확인 이후 로드밸런서 설정이 가능하기때문에, 아래 기술한 로드밸런서 생성 및 설정 과정은 [2.6. K-PaaS 컨테이너 플랫폼 클러스터 설치 확인](#2.6) 단계 이후에 진행한다.

<br><br>
2. <b><code>Load Balancer > Target Group</code></b> 메뉴에서 "Target Group 생성" 버튼을 클릭한다.

![image lb naver 002]

<br><br>
3. 아래 Target Group 정보를 입력 후 "다음" 버튼을 클릭한다.

|항목|설명|비고|
|---|---|---|
|Target Group 이름|Target Group 이름 입력||
|Target 유형|VPC Server 선택||
|VPC|Control Plane 노드와 동일한 네트워크 VPC 선택||
|프로토콜|PROXY_TCP 선택||
|포트|80 포트에 할당된 노드포트 정보 입력||

<br>

> Ingress Nginx 서비스 포트 확인<br>

```
$ kubectl get svc ingress-nginx-controller -n ingress-nginx
```

<br>

![image lb naver 003]

<br><br>
4. 아래 Helth Check 설정 정보를 입력 후 "다음" 버튼을 클릭한다.

|항목|설명|비고|
|---|---|---|
|프로토콜|TCP 선택||
|포트|인스턴스 상태체크가 가능한 포트 입력|예 : 인스턴스 SSH 포트 (TCP 22)|

<br>

![image lb naver 004]

<br><br>
5. 전체 노드를 선택하여 적용 Target에 포함시킨 후 Target Group을 생성한다.

![image lb naver 005]

![image lb naver 006]

<br><br>
6. 2~5번 과정을 반복하여 TCP 443 포트로 Target Group을 추가 생성한다.

<br><br>
7. <b><code>Load Balancer > Load Balancer</code></b> 메뉴에서 "로드밸런서 생성" 버튼 클릭 후 "네트워크 프록시 로드밸런서" 버튼을 클릭한다.

![image lb naver 007]

<br><br>
8. 아래 로드밸런서 정보를 입력 후 "다음" 버튼을 클릭한다.

|항목|설명|비고|
|---|---|---|
|로드밸런서 이름|로드밸런서 이름을 입력||
|네트워크|Public 선택||
|대상 VPC|Control Plane 노드와 동일한 네트워크 VPC 선택||
|서브넷 선택|로드밸런서 서브넷 선택||
|공인 IP|기존에 생성된 Public IP 선택(1번 과정에서 생성)||

<br>

![image lb naver 008]

<br><br>  
9. 리스너 설정 정보를 입력 후 "추가", "다음" 버튼을 클릭한다.

|항목|설명|비고|
|---|---|---|
|프로토콜|TCP 선택||
|로드밸런서 포트|80 포트 입력||

<br>

![image lb naver 009]

<br><br>
10. Target Group 선택 후 로드밸런서를 생성한다.

![image lb naver 010]

<br><br>
11. 생성한 로드밸런서 선택 후 "리스너 설정 변경" 버튼을 클릭, "리스너 추가" 버튼을 클릭한다.

![image lb naver 011]

<br><br>
12. 443 포트에 대한 리스너 설정 정보를 입력 후 리스너를 추가한다.

|항목|설명|비고|
|---|---|---|
|프로토콜|TCP 선택||
|포트|443 포트 입력||
|Target Group|443 포트 기준으로 설정한 Target Group 선택||

<br>

![image lb naver 012]

![image lb naver 013]

<br>

</details>

<br><br>

### <div id='2.1.7'> 2.1.7. HA Control Plane 로드밸런서 (***`HA Control Plane 구성 시 필수 설정`***)
> HA Control Plane 구성이 아닐 경우 로드밸런서 구성 과정은 생략하고 다음 과정을 진행한다.<br> [2.2. SSH Key 생성 및 배포](#2.2)

<br>

K-PaaS 컨테이너 플랫폼 클러스터를 HA Control Plane으로 구성할 경우 필요한 로드밸런서 정보는 다음과 같다.

<br>

|클라우드|로드밸런서|비고|
|---|---|---|
|Public|각 CSP사에서 제공하는 로드밸런서 서비스 이용||
|Private|Keepalived, HAProxy로 로드밸런서 인스턴스 구성||

<br><br>

### <div id='2.1.7.1'> 2.1.7.1. 클라우드 로드밸런서 서비스
Public 클라우드 환경의 경우 각 CSP에서 제공하는 로드밸런서 서비스를 생성한다.

<br>

각 CSP의 로드밸런서 서비스 생성은 [2.1.6.2. 클라우드 로드밸런서 서비스](#2.1.6.2)를 참고하며, 설정 시 아래 정보를 변경하여 진행한다.

<br>

<details>
<summary>NHN 클라우드</summary>
<br>

리스너

|항목|변경사항|비고|
|---|---|---|
|프로토콜|TCP 선택||
|로드밸런서 포트|6443 입력||

<br>

멤버 그룹

|항목|변경사항|비고|
|---|---|---|
|프로토콜|TCP 선택||
|포트|6443 입력||
|멤버 목록|Control Plane 인스턴스 추가||

<br>

</details>

<br>

<details>
<summary>KT 클라우드</summary>
<br>

로드밸런서 정보

|항목|변경사항|비고|
|---|---|---|
|Service IP / Port|신규 IP 할당, 6443 입력||
|Service Type|TCP 선택||

<br>

로드밸런서 VM 연결정보

|항목|변경사항|비고|
|---|---|---|
|VM|Control Plane 노드를 순차적으로 선택||
|Public Port|6443 입력||

<br>

</details>

<br>

<details>
<summary>Naver 클라우드</summary>
<br>

Target Group 정보

|항목|변경사항|비고|
|---|---|---|
|포트|6443 입력||
|적용 Target|Control Plane 인스턴스 추가||

<br>

로드밸런서 리스너 정보

|항목|변경사항|비고|
|---|---|---|
|포트|6443 입력||

<br>

</details>

<br><br>

### <div id='2.1.7.2'> 2.1.7.2. HAProxy
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

<details>
<summary>Keepalived, HAProxy 설치</summary>
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

</details>

<br><br>

### <div id='2.2'> 2.2. SSH Key 생성 및 배포
K-PaaS 컨테이너 플랫폼 클러스터 설치를 위해서는 SSH Key가 인벤토리의 모든 서버들에 복사되어야 한다.<br>
본 문서 (K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드) 에서는 RSA 공개키를 이용하여 SSH 접속 설정을 진행한다.

SSH Key 생성 및 배포 이후의 모든 설치과정은 ***`Install 인스턴스 또는 Control Plane 노드`*** 에서 진행한다.

<br>

> Naver 클라우드의 경우 ubuntu 계정 생성 시 키 생성 및 배포를 진행하기 때문에 해당 과정을 생략한다.

<br>

***`Install 인스턴스 또는 Control Plane 노드`*** 에서 RSA 공개키를 생성한다.
```
$ ssh-keygen -t rsa -m PEM -N '' -f $HOME/.ssh/id_rsa
Generating public/private rsa key pair.
Your identification has been saved in /home/ubuntu/.ssh/id_rsa
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:odWdv3PDIEpkPuoS53yM0hrsEQZL4mHvM0KwLK2uC57 ubuntu@cp-master
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|         . . .   |
|.+ o    = . o    |
|++= o  * .   .   |
|oo+o .. S . . .  |
|.+..o  o o . + . |
|o +o O. .     *  |
|=o.o=o*        o |
|E++o ++          |
+----[SHA256]-----+
```

<br>

사용할 ***`Install, Control Plane, Worker 노드`*** 에 공개키를 복사한다.
```
## 출력된 공개키 복사

$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5QrbqzV6g4iZT4iR1u+EKKVQGqBy4DbGqH7/PVfmAYEo3CcFGhRhzLcVz3rKb+C25mOne+MaQGynZFpZk4muEAUdkpieoo+B6r2eJHjBLopn5quWJ561H7EZb/GlfC5ThjHFF+hTf5trF4boW1iZRvUM56KAwXiYosLLRBXeNlub4SKfApe8ojQh4RRzFBZP/wNbOKr+Fo6g4RQCWrr5xQCZMK3ugBzTHM+zh9Ra7tG0oCySRcFTAXXoyXnJm+PFhdR6jbkerDlUYP9RD/87p/YKS1wSXExpBkEglpbTUPMCj+t1kXXEJ68JkMrVMpeznuuopgjHYWWD2FgjFFNkp ubuntu@cp-master
```

<br>

사용할 ***`Install, Control Plane, Worker 노드`*** 의 authorized_keys 파일 본문의 마지막 부분(기존 본문 내용 아래 추가)에 공개키를 복사한다.
```
$ vi .ssh/authorized_keys

ex)
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRueywSiuwyfmCSecHu7iwyi3xYS1xigAnhR/RMg/Ws3yOuwbKfeDFUprQR24BoMaD360uyuRaPpfqSL3LS9oRFrj0BSaQfmLcMM1+dWv+NbH/vvq7QWhIszVCLzwTqlHrhgNsh0+EMhqc15KEo5kHm7d7vLc0fB5tZkmovsUFzp01Ceo9+Qye6+j+UM6ssxdTmatoMP3ZZKZzUPF0EZwTcGG6+8rVK2G8GhTqwGLj9E+As3GB1YdOvr/fsTAi2PoxxFsypNR4NX8ZTDvRdAUzIxz8wv2VV4mADStSjFpE7HWrzr4tZUjvvVFptU4LbyON9YY4brMzjxA7kTuf/e3j Generated-by-Nova
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5QrbqzV6g4iZT4iR1u+EKKVQGqBy4DbGqH7/PVfmAYEo3CcFGhRhzLcVz3rKb+C25mOne+MaQGynZFpZk4muEAUdkpieoo+B6r2eJHjBLopn5quWJ561H7EZb/GlfC5ThjHFF+hTf5trF4boW1iZRvUM56KAwXiYosLLRBXeNlub4SKfApe8ojQh4RRzFBZP/wNbOKr+Fo6g4RQCWrr5xQCZMK3ugBzTHM+zh9Ra7tG0oCySRcFTAXXoyXnJm+PFhdR6jbkerDlUYP9RD/87p/YKS1wSXExpBkEglpbTUPMCj+t1kXXEJ68JkMrVMpeznuuopgjHYWWD2FgjFFNkp ubuntu@cp-master
```

<br><br>

### <div id='2.3'> 2.3. K-PaaS 컨테이너 플랫폼 클러스터 Deployment 다운로드

> 2.3.부터는 ***`Install 인스턴스 또는 Control Plane 노드`*** 에서만 진행.

<br>

K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 Deployment를 다운로드 후 설치 작업 경로로 이동하여 설치를 진행한다.

> K-PaaS 컨테이너 플랫폼 클러스터 Deployment Download URL : https://github.com/k-paas/cp-deployment

<br>

git clone 명령을 통해 HOME 디렉토리 경로에서 K-PaaS 컨테이너 플랫폼 클러스터 Deployment 다운로드를 진행한다.
```
$ git clone https://github.com/K-PaaS/cp-deployment.git -b branch_v1.5.x
```

<br><br>

### <div id='2.4'> 2.4. K-PaaS 컨테이너 플랫폼 클러스터 설치 준비
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 환경변수를 사전 정의 후 쉘 스크립트를 통해 설치를 진행한다.

> 멀티 클라우드 배포에서는 설치 경로 및 vars 파일 구성요소에 일부 차이가 존재한다.

<br>

K-PaaS 컨테이너 플랫폼 클러스터 설치경로로 이동한다.
```
$ cd ~/cp-deployment/multi
```

<br>

K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 환경변수 정보를 입력한다.
```
## 1번 클라우드 K-PaaS 컨테이너 플랫폼 클러스터 환경변수
$ vi cp-cluster-vars1.sh

## 2번 클라우드 K-PaaS 컨테이너 플랫폼 클러스터 환경변수
$ vi cp-cluster-vars2.sh
```

<br>

Control Plane

|환경변수|설명|비고|
|---|---|---|
|KUBE_CONTROL_HOSTS|Control Plane 노드의 갯수||
|ETCD_TYPE|ETCD 배포 방식<br>external : 별도의 노드에 ETCD 구성<br>stacked : Control Plane 노드에 ETCD 구성|**`KUBE_CONTROL_HOSTS`** 값이 2 이상일 경우 설정|
|LOADBALANCER_DOMAIN|사전에 구성한 로드밸런서의 VIP 또는 Domain 정보|**`KUBE_CONTROL_HOSTS`** 값이 2 이상일 경우 설정|
|ETCD1_NODE_HOSTNAME|ETCD 1번 노드의 호스트명|**`KUBE_CONTROL_HOSTS`** 값이 2 이상일 경우 설정<br>**`ETCD_TYPE`** 값이 external 일 경우 설정|
|ETCD1_NODE_PRIVATE_IP|ETCD 1번 노드의 Private IP|**`KUBE_CONTROL_HOSTS`** 값이 2 이상일 경우 설정<br>**`ETCD_TYPE`** 값이 external 일 경우 설정|
|ETCD{n}_NODE_HOSTNAME|ETCD n번 노드의 호스트명|**`KUBE_CONTROL_HOSTS`** 값이 2 이상일 경우 설정<br>**`ETCD_TYPE`** 값이 external 일 경우 설정<br>**`KUBE_CONTROL_HOSTS`** 값만큼 설정|
|ETCD{n}_NODE_PRIVATE_IP|ETCD n번 노드의 Private IP|**`KUBE_CONTROL_HOSTS`** 값이 2 이상일 경우 설정<br>**`ETCD_TYPE`** 값이 external 일 경우 설정<br>**`KUBE_CONTROL_HOSTS`** 값만큼 설정|
|MASTER1_NODE_HOSTNAME|Control Plane 1번 노드의 호스트명||
|MASTER1_NODE_PUBLIC_IP|Control Plane 1번 노드의 Public IP|Control Plane 1번 노드만 Public IP 정보 필요|
|MASTER1_NODE_PRIVATE_IP|Control Plane 1번 노드의 Private IP||
|MASTER{n}_NODE_HOSTNAME|Control Plane n번 노드의 호스트명|**`KUBE_CONTROL_HOSTS`** 값이 2 이상일 경우 설정<br>**`KUBE_CONTROL_HOSTS`** 값만큼 설정|
|MASTER{n}_NODE_PRIVATE_IP|Control Plane n번 노드의 Private IP|**`KUBE_CONTROL_HOSTS`** 값이 2 이상일 경우 설정<br>**`KUBE_CONTROL_HOSTS`** 값만큼 설정|

<br>

Worker

|환경변수|설명|비고|
|---|---|---|
|KUBE_WORKER_HOSTS|Worker 노드의 갯수||
|WORKER1_NODE_HOSTNAME|Worker 1번 노드의 호스트명||
|WORKER1_NODE_PRIVATE_IP|Worker 1번 노드의 Private IP||
|WORKER{n}_NODE_HOSTNAME|Worker n번 노드의 호스트명|**`KUBE_WORKER_HOSTS`** 값만큼 설정|
|WORKER{n}_NODE_PRIVATE_IP|Worker n번 노드의 Private IP|**`KUBE_WORKER_HOSTS`** 값만큼 설정|

<br>

Storage

|환경변수|설명|비고|
|---|---|---|
|STORAGE_TYPE|Storage 정보<br>nfs : NFS 스토리지<br>rook-ceph : Rook Ceph 스토리지||
|NFS_SERVER_PRIVATE_IP|NFS Server 인스턴스의 Private IP|**`STORAGE_TYPE`** 값 nfs 일 경우 설정|

<br>

LoadBalancer Service

|환경변수|설명|비고|
|---|---|---|
|METALLB_IP_RANGE|MetalLB에서 사용할 Private IP 대역|Control Plane 노드와 동일한 네트워크 서브넷 대역 설정|
|INGRESS_NGINX_PRIVATE_IP|MetalLB를 통해 Ingress Nginx Controller Service에서 사용할 ***`Private IP (인터페이스 일 경우) 또는 Public IP (로드밸런서 서비스 일 경우)`***|**`METALLB_IP_RANGE`** 값과 중복되지 않도록 설정|

<br>

> 멀티 클라우드 배포에서는 Istio 서비스에 대한 설정이 추가되었다.

Istio Service

|환경변수|설명|비고|
|---|---|---|
|ISTIO_INGRESS_PRIVATE_IP|MetalLB를 통해 Istio Ingress Gateway Service에서 사용할 Private IP|**`METALLB_IP_RANGE`** 값과 중복되지 않도록 설정|
|ISTIO_EASTWEST_PRIVATE_IP|MetalLB를 통해 Istio EastWest Gateway Service에서 사용할 ***`Private IP (인터페이스 일 경우) 또는 Public IP (로드밸런서 서비스 일 경우)`***|**`METALLB_IP_RANGE`** 값과 중복되지 않도록 설정|
|ISTIO_EASTWEST_PUBLIC_IP|Istio EastWest Gateway Service에서 사용할 인터페이스에 할당한 ***`Public IP (인터페이스, 로드밸런서 서비스)`***||

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

# MetalLB Istio Ingress Gateway LoadBalancer Service External IP
export ISTIO_INGRESS_PRIVATE_IP=

# MetalLB Istio EastWest Gateway LoadBalancer Service External IP
export ISTIO_EASTWEST_PRIVATE_IP=

# Istio eastwestgateway Service Public IP Variable
export ISTIO_EASTWEST_PUBLIC_IP=
```

<br><br>

### <div id='2.5'> 2.5. K-PaaS 컨테이너 플랫폼 클러스터 설치
쉘 스크립트를 통해 필요 패키지 설치, 클러스터 설치 환경변수 설정, Ansible playbook을 통한 K-PaaS 컨테이너 플랫폼 클러스터 설치를 순차적으로 진행한다.

```
$ source deploy-cp-cluster.sh
```

<br><br>

### <div id='2.6'> 2.6. K-PaaS 컨테이너 플랫폼 클러스터 설치 확인
노드 및 kube-system 네임스페이스의 Pod를 확인하여 K-PaaS 컨테이너 플랫폼 클러스터 설치를 확인한다.<br>
구성에 따라 조회되는 노드 및 Pod의 정보는 다를 수 있으며 아래는 단일 Control Plane 구성으로 배포했을때의 조회 결과이다. 

```
$ kubectl get nodes --context=cluster1
NAME                   STATUS   ROLES                  AGE   VERSION
cp-cluster1-master     Ready    control-plane          12m   v1.29.5
cp-cluster1-worker-1   Ready    <none>                 10m   v1.29.5
cp-cluster1-worker-2   Ready    <none>                 10m   v1.29.5
cp-cluster1-worker-3   Ready    <none>                 10m   v1.29.5

$ kubectl get pods -n kube-system --context=cluster1
NAME                                          READY   STATUS    RESTARTS      AGE
calico-node-d8sg6                             1/1     Running   0             9m22s
calico-node-kfvjx                             1/1     Running   0             10m
calico-node-khwdz                             1/1     Running   0             10m
calico-node-nc58v                             1/1     Running   0             10m
coredns-657959df74-td5c2                      1/1     Running   0             8m15s
coredns-657959df74-ztnjj                      1/1     Running   0             8m7s
dns-autoscaler-b5c786945-rhlkd                1/1     Running   0             8m9s
kube-apiserver-cp-cluster1-master             1/1     Running   0             12m
kube-controller-manager-cp-cluster1-master    1/1     Running   1 (11m ago)   12m
kube-proxy-dj5c8                              1/1     Running   0             10m
kube-proxy-kkvhk                              1/1     Running   0             10m
kube-proxy-nfttc                              1/1     Running   0             10m
kube-proxy-znfgk                              1/1     Running   0             10m
kube-scheduler-cp-cluster1-master             1/1     Running   1 (11m ago)   12m
metrics-server-5cd75b7749-xcrps               2/2     Running   0             7m57s
nginx-proxy-cp-cluster1-worker-1              1/1     Running   0             10m
nginx-proxy-cp-cluster1-worker-2              1/1     Running   0             10m
nginx-proxy-cp-cluster1-worker-3              1/1     Running   0             10m
nodelocaldns-556gb                            1/1     Running   0             8m8s
nodelocaldns-8dpnt                            1/1     Running   0             8m8s
nodelocaldns-pvl6z                            1/1     Running   0             8m8s
nodelocaldns-x7grn                            1/1     Running   0             8m8s
```

<br>

```
$ kubectl get nodes --context=cluster2
NAME                   STATUS   ROLES                  AGE   VERSION
cp-cluster2-master     Ready    control-plane          12m   v1.29.5
cp-cluster2-worker-1   Ready    <none>                 10m   v1.29.5
cp-cluster2-worker-2   Ready    <none>                 10m   v1.29.5
cp-cluster2-worker-3   Ready    <none>                 10m   v1.29.5

$ kubectl get pods -n kube-system --context=cluster2
NAME                                          READY   STATUS    RESTARTS      AGE
calico-node-d8sg6                             1/1     Running   0             9m22s
calico-node-kfvjx                             1/1     Running   0             10m
calico-node-khwdz                             1/1     Running   0             10m
calico-node-nc58v                             1/1     Running   0             10m
coredns-657959df74-td5c2                      1/1     Running   0             8m15s
coredns-657959df74-ztnjj                      1/1     Running   0             8m7s
dns-autoscaler-b5c786945-rhlkd                1/1     Running   0             8m9s
kube-apiserver-cp-cluster2-master             1/1     Running   0             12m
kube-controller-manager-cp-cluster2-master    1/1     Running   1 (11m ago)   12m
kube-proxy-dj5c8                              1/1     Running   0             10m
kube-proxy-kkvhk                              1/1     Running   0             10m
kube-proxy-nfttc                              1/1     Running   0             10m
kube-proxy-znfgk                              1/1     Running   0             10m
kube-scheduler-cp-cluster2-master             1/1     Running   1 (11m ago)   12m
metrics-server-5cd75b7749-xcrps               2/2     Running   0             7m57s
nginx-proxy-cp-cluster2-worker-1              1/1     Running   0             10m
nginx-proxy-cp-cluster2-worker-2              1/1     Running   0             10m
nginx-proxy-cp-cluster2-worker-3              1/1     Running   0             10m
nodelocaldns-556gb                            1/1     Running   0             8m8s
nodelocaldns-8dpnt                            1/1     Running   0             8m8s
nodelocaldns-pvl6z                            1/1     Running   0             8m8s
nodelocaldns-x7grn                            1/1     Running   0             8m8s
```

<br><br>

## <div id='3'> 3. K-PaaS 컨테이너 플랫폼 클러스터 삭제 (참고)
쉘 스크립트를 통해 K-PaaS 컨테이너 플랫폼 클러스터 삭제를 진행한다.

<br>

```
$ source reset-cp-cluster.sh
```

<br><br>

## <div id='4'> 4. Resource 생성 시 주의사항
사용자가 직접 Resource를 생성 시 다음과 같은 prefix를 사용하지 않도록 주의한다.

<br>

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

[image 001]:images/kpaas-cp-cluster-1.png
[image 002]:images/kpaas-cp-cluster-2.png
[image 003]:images/kpaas-cp-cluster-3.png
[image 004]:images/kpaas-cp-cluster-4.png
[image 005]:images/kpaas-cp-cluster-5.png
[image 006]:images/kpaas-cp-cluster-6.png
[image 007]:images/kpaas-cp-cluster-7.png

[image if nhn 001]:images/kpaas-cp-cluster-if-nhn-01.png
[image if nhn 002]:images/kpaas-cp-cluster-if-nhn-02.png
[image if nhn 003]:images/kpaas-cp-cluster-if-nhn-03.png
[image if nhn 004]:images/kpaas-cp-cluster-if-nhn-04.png
[image if nhn 005]:images/kpaas-cp-cluster-if-nhn-05.png
[image if nhn 006]:images/kpaas-cp-cluster-if-nhn-06.png
[image if nhn 007]:images/kpaas-cp-cluster-if-nhn-07.png
[image if nhn 008]:images/kpaas-cp-cluster-if-nhn-08.png

[image if kt 001]:images/kpaas-cp-cluster-if-kt-01.png
[image if kt 002]:images/kpaas-cp-cluster-if-kt-02.png
[image if kt 003]:images/kpaas-cp-cluster-if-kt-03.png
[image if kt 004]:images/kpaas-cp-cluster-if-kt-04.png
[image if kt 005]:images/kpaas-cp-cluster-if-kt-05.png
[image if kt 006]:images/kpaas-cp-cluster-if-kt-06.png

[image lb nhn 001]:images/kpaas-cp-cluster-lb-nhn-01.png
[image lb nhn 002]:images/kpaas-cp-cluster-lb-nhn-02.png
[image lb nhn 003]:images/kpaas-cp-cluster-lb-nhn-03.png
[image lb nhn 004]:images/kpaas-cp-cluster-lb-nhn-04.png
[image lb nhn 005]:images/kpaas-cp-cluster-lb-nhn-05.png
[image lb nhn 006]:images/kpaas-cp-cluster-lb-nhn-06.png

[image lb kt 001]:images/kpaas-cp-cluster-lb-kt-01.png
[image lb kt 002]:images/kpaas-cp-cluster-lb-kt-02.png
[image lb kt 003]:images/kpaas-cp-cluster-lb-kt-03.png
[image lb kt 004]:images/kpaas-cp-cluster-lb-kt-04.png
[image lb kt 005]:images/kpaas-cp-cluster-lb-kt-05.png
[image lb kt 006]:images/kpaas-cp-cluster-lb-kt-06.png
[image lb kt 007]:images/kpaas-cp-cluster-lb-kt-07.png
[image lb kt 008]:images/kpaas-cp-cluster-lb-kt-08.png

[image lb naver 001]:images/kpaas-cp-cluster-lb-naver-01.png
[image lb naver 002]:images/kpaas-cp-cluster-lb-naver-02.png
[image lb naver 003]:images/kpaas-cp-cluster-lb-naver-03.png
[image lb naver 004]:images/kpaas-cp-cluster-lb-naver-04.png
[image lb naver 005]:images/kpaas-cp-cluster-lb-naver-05.png
[image lb naver 006]:images/kpaas-cp-cluster-lb-naver-06.png
[image lb naver 007]:images/kpaas-cp-cluster-lb-naver-07.png
[image lb naver 008]:images/kpaas-cp-cluster-lb-naver-08.png
[image lb naver 009]:images/kpaas-cp-cluster-lb-naver-09.png
[image lb naver 010]:images/kpaas-cp-cluster-lb-naver-10.png
[image lb naver 011]:images/kpaas-cp-cluster-lb-naver-11.png
[image lb naver 012]:images/kpaas-cp-cluster-lb-naver-12.png
[image lb naver 013]:images/kpaas-cp-cluster-lb-naver-13.png

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드