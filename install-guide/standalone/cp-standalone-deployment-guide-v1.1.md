
## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [시스템 구성도](#1.3)  
  1.4. [참고자료](#1.4)  

2. [Kubespray 설치](#2)  
  2.1. [Prerequisite](#2.1)  
  2.2. [SSH Key 생성 및 배포](#2.2)  
  2.3. [Kubespray 다운로드](#2.3)  
  2.4. [Ubuntu, Python Package 설치](#2.4)  
  2.5. [Kubespray 파일 수정](#2.5)  
  2.6. [Kubespray 설치](#2.6)  
  2.7. [Kubespray 설치 확인](#2.7)  

3. [Kubespray 삭제 (참고)](#3)  

4. [컨테이너 플랫폼 운영자 생성 및 Token 획득 (참고)](#4)  
  4.1. [Cluster Role 운영자 생성 및 Token 획득](#4.1)  
  4.2. [Namespace 사용자 Token 획득](#4.2)  

5. [Resource 생성 시 주의사항](#5)  

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (Kubespray 설치 가이드) 는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에 배포되는 컨테이터 플랫폼을 설치하기 위한 Kubernetes Native를 Kubespray를 이용하여 설치하는 방법을 기술하였다.

PaaS-TA 5.5 버전부터는 Kubespray 기반으로 단독 배포를 지원한다. 기존 Container 서비스 기반으로 설치를 원할 경우에는 PaaS-TA 5.0 이하 버전의 문서를 참고한다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 Kubernetes Native를 검증하기 위한 Kubespray 기본 설치를 기준으로 작성하였다.

<br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 Kubernetes Cluster(Master, Worker)와 BOSH Inception(DBMS, HAProxy, Private Registry)환경으로 구성되어 있다.<br>
Kubespary를 통해 Kubernetes Cluster를 설치하고 BOSH release로 Database, Private registry 등 미들웨어 환경을 제공하여 Docker Image로 Kubernetes Cluster에 Container Platform 포털 환경을 배포한다. <br>
총 필요한 VM 환경으로는 **Master VM: 1개, Worker VM: 1개 이상, BOSH Inception VM: 1개**가 필요하고 본 문서는 Kubernetes Cluster 환경을 구성하기 위한 Master VM 과 Worker VM 설치 내용이다.

![image 001]

<br>

### <div id='1.4'> 1.4. 참고자료
> https://kubespray.io  
> https://github.com/kubernetes-sigs/kubespray  

<br>

## <div id='2'> 2. Kubespray 설치

### <div id='2.1'> 2.1. Prerequisite
본 설치 가이드는 **Ubuntu 18.04** 환경에서 설치하는 것을 기준으로 하였다. Kubespray 설치를 위해서는 Ansible v2.9 +, Jinja 2.11+ 및 python-netaddr이 Ansible 명령을 실행할 시스템에 설치되어 있어야 하며 설치 가이드에 따라 순차적으로 설치가 진행된다.


Kubespray 설치에 필요한 주요 소프트웨어 및 패키지 Version 정보는 다음과 같다.

|주요 소프트웨어|Version|Python Package|Version
|---|---|---|---|
|Kubespray|v2.16.0|ansible|2.9.20|
|Kubernetes Native|v1.20.5|jinja2|2.11.3|
|CRI-O|v1.20.0|netaddr|0.7.19|
|||pbr|5.4.4|
|||jmespath|0.9.5|
|||ruamel.yaml|0.16.10|
|||cryptography|2.8|
|||MarkupSafe|1.1.1|

Kubernetes 공식 가이드 문서에서는 Cluster 배포 시 다음을 권고하고 있다.

- deb / rpm 호환 Linux OS를 실행하는 하나 이상의 머신 (Ubuntu 또는 CentOS)
- 머신 당 2G 이상의 RAM
- control-plane 노드로 사용하는 머신에 2 개 이상의 CPU
- 클러스터의 모든 시스템 간의 완전한 네트워크 연결


#### 방화벽 정보
- Master Node

| <center>프로토콜</center> | <center>포트</center> | <center>비고</center> |  
| :---: | :---: | :--- |  
| TCP | 179 | Calio BGP Network |  
| TCP | 2379-2380 | etcd server client API |  
| TCP | 6443 | Kubernetes API Server |  
| TCP | 10250 | Kubelet API |  
| TCP | 10251 | kube-scheduler |  
| TCP | 10252 | kube-controller-manager |  
| TCP | 10255 | Read-Only Kubelet API |  
| IP-in-IP (Protocol Num 4) || Calico Overlay Network |  

- Worker Node

| <center>프로토콜</center> | <center>포트</center> | <center>비고</center> |  
| :---: | :---: | :--- |  
| TCP | 179 | Calio BGP network |
| TCP | 10250 | Kubelet API |  
| TCP | 10255 | Read-Only Kubelet API |  
| TCP | 30000-32767 | NodePort Services |  
| IP-in-IP (Protocol Num 4) || Calico Overlay Network |  

<br>

### <div id='2.2'> 2.2. SSH Key 생성 및 배포
Kubespray 설치를 위해서는 SSH Key가 인벤토리의 모든 서버들에 복사되어야 한다. 본 설치 가이드에서는 RSA 공개키를 이용하여 SSH 접속 설정을 진행한다.  

SSH Key 생성 및 배포 이후의 모든 설치과정은 **Master Node**에서 진행한다.

- Master Node에서 RSA 공개키를 생성한다.
```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): [엔터키 입력]
Enter passphrase (empty for no passphrase): [엔터키 입력]
Enter same passphrase again: [엔터키 입력]
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:pIG4/G309Dof305mWjdNz1OORx9nQgQ3b8yUP5DzC3w ubuntu@paasta-cp-master
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

- 사용할 Master, Worker Node에 공개키를 복사한다.
```
## 출력된 공개키 복사

$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5QrbqzV6g4iZT4iR1u+EKKVQGqBy4DbGqH7/PVfmAYEo3CcFGhRhzLcVz3rKb+C25mOne+MaQGynZFpZk4muEAUdkpieoo+B6r2eJHjBLopn5quWJ561H7EZb/GlfC5ThjHFF+hTf5trF4boW1iZRvUM56KAwXiYosLLRBXeNlub4SKfApe8ojQh4RRzFBZP/wNbOKr+Fo6g4RQCWrr5xQCZMK3ugBzTHM+zh9Ra7tG0oCySRcFTAXXoyXnJm+PFhdR6jbkerDlUYP9RD/87p/YKS1wSXExpBkEglpbTUPMCj+t1kXXEJ68JkMrVMpeznuuopgjHYWWD2FgjFFNkp ubuntu@paasta-cp-master
```

- 사용할 Master, Worker Node의 authorized_keys 파일 본문의 마지막 부분(기존 본문 내용 아래 추가)에 공개키를 복사한다.
```
$ vi .ssh/authorized_keys

ex)
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRueywSiuwyfmCSecHu7iwyi3xYS1xigAnhR/RMg/Ws3yOuwbKfeDFUprQR24BoMaD360uyuRaPpfqSL3LS9oRFrj0BSaQfmLcMM1+dWv+NbH/vvq7QWhIszVCLzwTqlHrhgNsh0+EMhqc15KEo5kHm7d7vLc0fB5tZkmovsUFzp01Ceo9+Qye6+j+UM6ssxdTmatoMP3ZZKZzUPF0EZwTcGG6+8rVK2G8GhTqwGLj9E+As3GB1YdOvr/fsTAi2PoxxFsypNR4NX8ZTDvRdAUzIxz8wv2VV4mADStSjFpE7HWrzr4tZUjvvVFptU4LbyON9YY4brMzjxA7kTuf/e3j Generated-by-Nova
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5QrbqzV6g4iZT4iR1u+EKKVQGqBy4DbGqH7/PVfmAYEo3CcFGhRhzLcVz3rKb+C25mOne+MaQGynZFpZk4muEAUdkpieoo+B6r2eJHjBLopn5quWJ561H7EZb/GlfC5ThjHFF+hTf5trF4boW1iZRvUM56KAwXiYosLLRBXeNlub4SKfApe8ojQh4RRzFBZP/wNbOKr+Fo6g4RQCWrr5xQCZMK3ugBzTHM+zh9Ra7tG0oCySRcFTAXXoyXnJm+PFhdR6jbkerDlUYP9RD/87p/YKS1wSXExpBkEglpbTUPMCj+t1kXXEJ68JkMrVMpeznuuopgjHYWWD2FgjFFNkp ubuntu@paasta-cp-master
```

<br>

### <div id='2.3'> 2.3. Kubespray 다운로드
2.3.부터는 **Master Node**에서만 진행을 하면 된다.(Worker Node에는 더 이상 추가 작업이 없음)
Kubespray 설치에 필요한 Source File을 Download 받아 Kubespray 설치 작업 경로로 위치시킨다.

- Kubespray Download URL : https://github.com/PaaS-TA/paas-ta-container-platform-deployment

- git clone 명령을 통해 다음 경로에서 Kubespray 다운로드를 진행한다. 본 설치 가이드에서의 Kubespray 버전은 v2.16.0 이다.
```
$ git clone https://github.com/PaaS-TA/paas-ta-container-platform-deployment.git
```

<br>

### <div id='2.4'> 2.4. Ubuntu, Python Package 설치
Kubespray 설치에 필요한 Ansible, Jinja 등 Python Package 설치를 진행한다.

- apt-get update를 진행한다.
```
$ sudo apt-get update
```

- python3-pip Package를 설치한다.
```
$ sudo apt-get install -y python3-pip
```

- Kubespray 설치경로 이동, pip를 이용하여 Kubespray 설치에 필요한 Python Package 설치를 진행한다.
```
## AWS 환경 설치 시

$ cd paas-ta-container-platform-deployment/standalone/aws
```

```
## Openstack 환경 설치 시

$ cd paas-ta-container-platform-deployment/standalone/openstack
```

```
$ sudo pip3 install -r requirements.txt
```

<br>

### <div id='2.5'> 2.5. Kubespray 파일 수정
Kubespray inventory 파일에는 배포할 Master, Worker Node의 구성을 정의한다.
본 설치 가이드에서는 1개의 Master Node와 3개의 Worker Node, 1개의 etcd 배포를 기준으로 가이드를 진행하며 기본 CNI는 calico로 설정되어있다.
(기본 Cluster 환경 구성은 Master Node 1개와 Worker Node 1개 이상을 필요로 한다.)

- mycluster 디렉토리의 inventory.ini 파일을 설정한다.
```
$ vi inventory/mycluster/inventory.ini
```

```
## *_HOST_NAME = 각 호스트의 쉘에서 hostname 명령어 입력
## *_NODE_IP = 각 호스트의 쉘에서 ifconfig 입력 후 inet ip 입력

[all]
{MASTER_HOST_NAME} ansible_host={MASTER_NODE_IP} ip={MASTER_NODE_IP} etcd_member_name=etcd1
{WORKER_HOST_NAME1} ansible_host={WORKER_NODE_IP1} ip={WORKER_NODE_IP1}      # 사용할 WORKER_NODE 개수(1개 이상)에 따라 작성
{WORKER_HOST_NAME2} ansible_host={WORKER_NODE_IP2} ip={WORKER_NODE_IP2}
{WORKER_HOST_NAME3} ansible_host={WORKER_NODE_IP3} ip={WORKER_NODE_IP3}

[kube_control_plane]
paasta-cp-master           #{MASTER_HOST_NAME}

[etcd]
paasta-cp-master           #{MASTER_HOST_NAME}

[kube_node]
paasta-cp-worker-1           #{WORKER_HOST_NAME1}: 사용할 WORKER_NODE 개수(1개 이상)에 따라 작성
{WORKER_HOST_NAME2}
{WORKER_HOST_NAME3}

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

```
ex)
paasta-cp-master  ansible_host=10.x.x.x ip=10.x.x.x etcd_member_name=etcd1
paasta-cp-worker-1  ansible_host=10.x.x.x ip=10.x.x.x
...
[kube_control_plane]
paasta-cp-master
...
```

- Metrics-server 를 배포할 Master Node의 HostName 정보를 추가한다.

```
$ vi roles/kubernetes-apps/metrics_server/defaults/main.yml
```

```
...
master_node_hostname: {MASTER_NODE_HOSTNAME}
````

- 외부에서 kubectl 명령어를 사용하기 위해 Master Node의 Public IP 정보를 추가한다.
```
$ vi roles/kubernetes/control-plane/tasks/kubeadm-setup.yml
```

```
- name: kubeadm | aggregate all SANs
  set_fact:
    apiserver_sans: "{{ (sans_base + groups['kube_control_plane'] + sans_lb + sans_lb_ip + sans_supp + sans_access_ip + sans_ip + sans_address + sans_override + sans_hostname + sans_fqdn) | unique }}"
  vars:
    sans_base:
      - "kubernetes"
      - "kubernetes.default"
      - "kubernetes.default.svc"
      - "kubernetes.default.svc.{{ dns_domain }}"
      - "{MASTER_NODE_PUBLIC_IP}" (추가)
      - "{{ kube_apiserver_ip }}"
      - "localhost"
      - "127.0.0.1"
```

<br>

### <div id='2.6'> 2.6. Kubespray 설치
Ansible playbook을 이용하여 Kubespray 설치를 진행한다.

- 인벤토리 빌더로 Ansible 인벤토리 파일을 업데이트한다.
```
## {MASTER_NODE_IP}, {WORKER_NODE_IP} : Master, Worker Node Private IP
## {WORKER_NODE_IP}는 사용할 WORKER_NODE 개수(1개 이상)에 따라 작성

$ declare -a IPS=({MASTER_NODE_IP} {WORKER_NODE_IP1} {WORKER_NODE_IP2} {WORKER_NODE_IP3})

ex)
declare -a IPS=(10.x.x.x 10.x.x.x 10.x.x.x 10.x.x.x)
```

```
## ${IPS[@]}는 변수가 아니라 명령어의 일부분이므로 주의

$ CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

- Openstack 환경에 설치 시 추가적인 환경변수 설정이 필요하며 설정 파일을 다운로드 받아 자동으로 환경변수 등록이 가능하다.
```
## Openstack UI 로그인 > 프로젝트 선택 > API 액세스 메뉴 선택 > OpenStack RC File 다운로드 클릭
## 스크립트 파일 실행 후 Openstack 계정 패스워드 입력

$ source {OPENSTACK_PROJECT_NAME}-openrc.sh
Please enter your OpenStack Password for project admin as user admin: {패스워드 입력}
```

- Openstack 네트워크 인터페이스의 MTU값이 기본값 1450이 아닐 경우 CNI Plugin MTU 설정 변경이 필요하다.
```
## MTU 확인 (ex mtu 1400)

$ ifconfig
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1400
        inet 10.100.2.226  netmask 255.255.255.0  broadcast 10.100.2.255
        inet6 fe80::f816:3eff:fe2f:a831  prefixlen 64  scopeid 0x20<link>
        ether fa:16:3e:2f:a8:31  txqueuelen 1000  (Ethernet)
        RX packets 19850  bytes 167795140 (167.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17667  bytes 1920463 (1.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...
```

```
$ vi inventory/mycluster/group_vars/k8s_cluster/k8s-net-calico.yml
```

```
...
calico_mtu: 1400
...
```

- Ansible playbook으로 Kubespray 배포를 진행한다. playbook은 root로 실행하도록 옵션을 지정한다. (--become-user=root)
```
$ ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```

- Kubespray 설치 완료 후 Cluster 사용을 위하여 다음 과정을 실행한다.
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

<br>

### <div id='2.7'> 2.7. Kubespray 설치 확인
Kubernetes Node 및 kube-system Namespace의 Pod를 확인하여 Kubespray 설치를 확인한다.

```
$ kubectl get nodes
NAME                 STATUS   ROLES                  AGE   VERSION
paasta-cp-master     Ready    control-plane,master   12m   v1.20.5
paasta-cp-worker-1   Ready    <none>                 10m   v1.20.5
paasta-cp-worker-2   Ready    <none>                 10m   v1.20.5
paasta-cp-worker-3   Ready    <none>                 10m   v1.20.5

$ kubectl get pods -n kube-system
NAME                                          READY   STATUS    RESTARTS   AGE
calico-kube-controllers-7c5b64bf96-xwdgn      1/1     Running   0          8m52s
calico-node-d8sg6                             1/1     Running   0          9m22s
calico-node-kfvjx                             1/1     Running   0          10m
calico-node-khwdz                             1/1     Running   0          10m
calico-node-nc58v                             1/1     Running   0          10m
coredns-657959df74-td5c2                      1/1     Running   0          8m15s
coredns-657959df74-ztnjj                      1/1     Running   0          8m7s
csi-cinder-controllerplugin-99c9dd87b-hz62k   5/5     Running   0          7m42s
csi-cinder-nodeplugin-jjkg5                   2/2     Running   0          7m41s
csi-cinder-nodeplugin-njdrc                   2/2     Running   0          7m41s
csi-cinder-nodeplugin-sb9vg                   2/2     Running   0          7m41s
csi-cinder-nodeplugin-t5fxh                   2/2     Running   0          7m41s
dns-autoscaler-b5c786945-rhlkd                1/1     Running   0          8m9s
kube-apiserver-paasta-cp-master               1/1     Running   0          12m
kube-controller-manager-paasta-cp-master      1/1     Running   1          12m
kube-proxy-dj5c8                              1/1     Running   0          10m
kube-proxy-kkvhk                              1/1     Running   0          10m
kube-proxy-nfttc                              1/1     Running   0          10m
kube-proxy-znfgk                              1/1     Running   0          10m
kube-scheduler-paasta-cp-master               1/1     Running   1          12m
metrics-server-5cd75b7749-xcrps               2/2     Running   0          7m57s
nginx-proxy-paasta-cp-worker-1                1/1     Running   0          10m
nginx-proxy-paasta-cp-worker-2                1/1     Running   0          10m
nginx-proxy-paasta-cp-worker-3                1/1     Running   0          10m
nodelocaldns-556gb                            1/1     Running   0          8m8s
nodelocaldns-8dpnt                            1/1     Running   0          8m8s
nodelocaldns-pvl6z                            1/1     Running   0          8m8s
nodelocaldns-x7grn                            1/1     Running   0          8m8s
openstack-cloud-controller-manager-mct28      1/1     Running   0          8m57s
snapshot-controller-0                         1/1     Running   0          7m33s
```

<br>

## <div id='3'> 3. Kubespray 삭제 (참고)
Ansible playbook을 이용하여 Kubespray 삭제를 진행한다.

```
$ ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root reset.yml
```

<br>

## <div id='4'> 4. 컨테이너 플랫폼 운영자 생성 및 Token 획득 (참고)

### <div id='4.1'> 4.1. Cluster Role 운영자 생성 및 Token 획득
Kubespray 설치 이후에 Cluster Role을 가진 운영자의 Service Account를 생성한다. 해당 Service Account의 Token은 운영자 포털에서 Super Admin 계정 생성 시 이용된다.

- Service Account를 생성한다.
```
## {SERVICE_ACCOUNT} : 생성할 Service Account 명

$ kubectl create serviceaccount {SERVICE_ACCOUNT} -n kube-system
(ex. kubectl create serviceaccount k8sadmin -n kube-system)
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
