### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 단독형 배포 포털 설치 가이드

<br>

>v1.2 (Current)

>[v1.3](/install-guide/container-platform-portal/paas-ta-container-platform-portal-deployment-standalone-guide.md)


## Table of Contents

1. [문서 개요](#1)  
    1.1. [목적](#1.1)  
    1.2. [범위](#1.2)  
    1.3. [시스템 구성도](#1.3)  
    1.4. [참고 자료](#1.4)  

2. [Prerequisite](#2)  
    2.1. [방화벽 정보](#2.1)  
    2.2. [NFS Server 설치](#2.2)    
     
3. [컨테이너 플랫폼 포털 배포](#3)  
    3.1. [CRI-O insecure-registry 설정](#3.1)  
    3.2. [컨테이너 플랫폼 포털 배포](#3.2)  
    3.2.1. [컨테이너 플랫폼 포털 Deployment 파일 다운로드](#3.2.1)  
    3.2.2. [컨테이너 플랫폼 포털 변수 정의](#3.2.2)    
    3.2.3. [컨테이너 플랫폼 포털 배포 스크립트 실행](#3.2.3)  
    3.2.4. [(참조) 컨테이너 플랫폼 포털 리소스 삭제](#3.2.4) 

4. [컨테이너 플랫폼 운영자/사용자 포털 접속](#4)      
    4.1. [컨테이너 플랫폼 운영자 포털 로그인](#4.1)      
    4.2. [컨테이너 플랫폼 사용자 포털 회원가입/로그인](#4.2)      
    4.3. [컨테이너 플랫폼 사용자/운영자 포털 사용 가이드](#4.3)          

5. [컨네이너 플랫폼 포털 참고](#5)       
    5.1. [운영자 Cluster Role Token 생성](#5.1)  
    5.2. [Kubernetes 리소스 생성 시 주의사항](#5.2)    


## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(컨테이너 플랫폼 단독 배포 형 포털 설치 가이드)는 Kubernetes Cluster를 설치하고 컨테이너 플랫폼 단독 배포 형 포털 배포 방법을 기술하였다.<br>
<br>

### <div id='1.2'>1.2. 범위
설치 범위는 Kubernetes Cluster 배포를 기준으로 작성하였다.

<br>

### <div id='1.3'>1.3. 시스템 구성도
<p align="center"><img src="images-v1.2/cp-001.png"></p>    
    
시스템 구성은 **Kubernetes Cluster(Master, Worker)** 환경과 데이터 관리를 위한 **네트워크 파일 시스템(NFS)** 스토리지 서버로 구성되어 있다. Kubespray를 통해 설치된 Kubernetes Cluster 환경에 컨테이너 플랫폼 포털 이미지 및 Helm Chart를 관리하는 **Harbor**, 컨테이너 플랫폼 포털 사용자 인증을 관리하는 **Keycloak**, 컨테이너 플랫폼 포털 메타 데이터를 관리하는 **MariaDB(RDBMS)** 등 미들웨어 환경을 컨테이너로 제공한다. 총 필요한 VM 환경으로는 **Master Node VM: 1개, Worker Node VM: 1개 이상, NFS Server : 1개**가 필요하고 본 문서는 Kubernetes Cluster에 컨테이너 플랫폼 포털 환경을 배포하는 내용이다. **네트워크 파일 시스템(NFS)** 은 컨테이너플랫폼에서 기본으로 제공하는 스토리지로 사용자 환경에 따라 다양한 종류의 스토리지를 사용할 수 있다.  
        
<br>    

### <div id='1.4'>1.4. 참고 자료
> https://kubernetes.io/ko/docs<br>
> https://goharbor.io/docs<br>
> https://www.keycloak.org/documentation
    
<br>

## <div id='2'>2. Prerequisite
본 설치 가이드는 **Ubuntu 18.04** 환경에서 설치하는 것을 기준으로 작성하였다. 

### <div id='2.1'>2.1. 방화벽 정보
IaaS Security Group의 열어줘야할 Port를 설정한다.

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

### <div id='2.2'>2.2. NFS Server 설치
컨테이너 플랫폼 포털 서비스에서 사용할 스토리지 **NFS Storage Server** 설치가 사전에 진행되어야 한다.<br>
NFS Storage Server 설치는 아래 가이드를 참조한다.  
> [NFS Server 설치](../nfs-server-install-guide.md)      
    
<br>
    
## <div id='3'>3. 컨테이너 플랫폼 포털 배포

### <div id='3.1'>3.1. CRI-O insecure-registry 설정
컨테이너 플랫폼 포털 배포는 Private Repository(Harbor) 배포를 포함하고 있다. Private Repository에 컨테이너 플랫폼 포털 관련 이미지 및 패키지 파일 업로드 그리고 HTTP 접속 설정을 위해 배포 전 Kubernetes **Master Node, Worker Node** 내 podman 설치 및 config 파일에 'insecure-registries' 설정을 진행한다.

- **:bulb: Master Node, Worker Node에 모두 설정 추가 필요**
- **{K8S_MASTER_NODE_IP} 값은 Kubernetes Master Node Public IP 입력**
    
#### 1. podman 설치
    
```
$ sudo apt-get update
$ sudo apt-get install -y podman
```

#### 2. crio.conf 내 'insecure-registries' 설정
```
$ sudo vi /etc/crio/crio.conf
``` 
    
```
# 'insecure_registries' 항목에 "{K8S_MASTER_NODE_IP}:30002" 추가 
...    
insecure_registries = [
 "xx.xxx.xxx.xx:30002"
  ]
...    
```

```
# crio 재시작
$ sudo systemctl restart crio
```

#### 3. registries.conf 내 'registry' 항목 insecure 설정
```
$ sudo vi /etc/containers/registries.conf
```

```
# registries.conf 파일 내 '[[registry]]'가 주석 처리 되어있으므로 주석 해제 필요     
# 아래 항목을 추가, location 값은 "{K8S_MASTER_NODE_IP}:30002" 설정
...    
[[registry]]
insecure = true
location = "xx.xxx.xxx.xx:30002"
...
```

```
# podman 재시작
$ sudo systemctl restart podman
```

<br>

### <div id='3.2'>3.2. 컨테이너 플랫폼 포털 배포
    
#### <div id='3.2.1'>3.2.1. 컨테이너 플랫폼 포털 Deployment 파일 다운로드
컨테이너 플랫폼 포털 배포를 위해 컨테이너 플랫폼 포털 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
:bulb: 해당 내용은 Kubernetes **Master Node**에서 진행한다.

+ 컨테이너 플랫폼 포털 Deployment 파일 다운로드 :
   [paas-ta-container-platform-portal-deployment_v1.2.3.tar.gz](https://nextcloud.paas-ta.org/index.php/s/2WxffSnLTp4eLxL/download)

```
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.paas-ta.org/index.php/s/2WxffSnLTp4eLxL/download

$ ls ~/workspace/container-platform
  paas-ta-container-platform-portal-deployment_v1.2.3.tar.gz

# Deployment 파일 압축 해제
$ tar -xvf paas-ta-container-platform-portal-deployment_v1.2.3.tar.gz
```

- Deployment 파일 디렉토리 구성
```
├── script          # 컨테이너 플랫폼 포털 배포 관련 변수 및 스크립트 파일 위치
├── images          # 컨테이너 플랫폼 포털 이미지 파일 위치
├── charts          # 컨테이너 플랫폼 포털 Helm Charts 파일 위치
├── values_orig     # 컨테이너 플랫폼 포털 Helm Charts values.yaml 파일 위치
└── keycloak_orig   # 컨테이너 플랫폼 포털 사용자 인증 관리를 위한 Keycloak 배포 관련 파일 위치
```

<br>

#### <div id='3.2.2'>3.2.2. 컨테이너 플랫폼 포털 변수 정의
컨테이너 플랫폼 포털을 배포하기 전 변수 값 정의가 필요하다. 배포에 필요한 정보를 확인하여 변수를 설정한다.

:bulb: Keycloak 기본 배포 방식은 **HTTP**이며 인증서를 통한 **HTTPS**를 설정하고자 하는 경우 아래 가이드를 참조하여 선처리한다.
> [Keycloak TLS 설정](paas-ta-container-platform-portal-deployment-keycloak-tls-setting-guide-v1.2.md#2-keycloak-tls-설정)       

<br>
    
```
$ cd ~/workspace/container-platform/paas-ta-container-platform-portal-deployment/script
$ vi container-platform-portal-vars.sh
```

```                                                     
# COMMON VARIABLE
K8S_MASTER_NODE_IP="{k8s master node public ip}"             # Kubernetes Master Node Public IP
K8S_AUTH_BEARER_TOKEN="{k8s auth bearer token}"              # Kubernetes Authorization Bearer Token
NFS_SERVER_IP="{nfs server ip}"                              # NFS Server IP
PROVIDER_TYPE="{container platform portal provider type}"    # Container Platform Portal Provider Type (Please enter 'standalone' or 'service')
....    
```
```    
# Example
K8S_MASTER_NODE_IP="xx.xxx.xxx.xx"                 
K8S_AUTH_BEARER_TOKEN="qY3k2xaZpNbw3AJxxxxx...."                 
NFS_SERVER_IP="xx.xxx.xxx.xx"                                  
PROVIDER_TYPE="standalone"           
```

- **K8S_MASTER_NODE_IP** <br>Kubernetes Master Node Public IP 입력<br><br>
- **K8S_AUTH_BEARER_TOKEN** <br>Kubernetes Bearer Token 입력<br>
   + [[5.1. 운영자 Cluster Role Token 생성]](#5.1) 참고하여 Token 값 생성 후 입력 <br><br>
- **NFS_SERVER_IP** <br>NFS Server Private IP 입력<br>
   + 가이드 [[NFS Server 설치](../nfs-server-install-guide.md)]를 통해 설치된 NFS Server Private IP 입력<br><br>
- **PROVIDER_TYPE** <br>컨테이너 플랫폼 포털 제공 타입 입력 <br>
   + 본 가이드는 포털 단독 배포 형 설치 가이드로 **'standalone'** 값 입력 필요

<br>

    
#### <div id='3.2.3'>3.2.3. 컨테이너 플랫폼 포털 배포 스크립트 실행
컨테이너 플랫폼 포털 배포를 위한 배포 스크립트를 실행한다.

```
$ chmod +x deploy-container-platform-portal.sh
$ ./deploy-container-platform-portal.sh
```
<br>
    
컨테이너 플랫폼 포털 관련 리소스가 정상적으로 배포되었는지 확인한다.<br>
리소스 Pod의 경우 Node에 바인딩 및 컨테이너 생성 후 Running 상태로 전환되기까지 몇 초가 소요된다.

    
- **NFS 리소스 조회**
>`$ kubectl get all -n nfs-storageclass`  
```
$ kubectl get all -n nfs-storageclass
NAME                                       READY   STATUS    RESTARTS   AGE
pod/nfs-pod-provisioner-7f7c444487-nxztc   1/1     Running   0          3m22s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nfs-pod-provisioner   1/1     1            1           3m22s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/nfs-pod-provisioner-7f7c444487   1         1         1       3m22s
```
    
- **Harbor 리소스 조회**
>`$ kubectl get all -n harbor`      
```
$ kubectl get all -n harbor
NAME                                                                  READY   STATUS    RESTARTS   AGE
pod/paas-ta-container-platform-harbor-chartmuseum-59b784bfd4-55twb    1/1     Running   0          4m
pod/paas-ta-container-platform-harbor-core-6b7b8768f9-r27nn           1/1     Running   0          4m
pod/paas-ta-container-platform-harbor-database-0                      1/1     Running   0          4m
pod/paas-ta-container-platform-harbor-jobservice-69556cd7cd-hk75s     1/1     Running   0          4m
pod/paas-ta-container-platform-harbor-nginx-755cf6876f-rkwk2          1/1     Running   0          4m
pod/paas-ta-container-platform-harbor-notary-server-896c47dcd-lg78k   1/1     Running   0          4m
pod/paas-ta-container-platform-harbor-notary-signer-b747fb89d-lpw86   1/1     Running   0          4m
pod/paas-ta-container-platform-harbor-portal-c79c78f5b-nk5vm          1/1     Running   0          4m
pod/paas-ta-container-platform-harbor-redis-0                         1/1     Running   0          4m
pod/paas-ta-container-platform-harbor-registry-54c4f69579-zwjmp       2/2     Running   0          4m
pod/paas-ta-container-platform-harbor-trivy-0                         1/1     Running   0          4m

NAME                                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                       AGE
service/harbor                                            NodePort    10.233.3.114    <none>        80:30002/TCP,4443:30004/TCP   4m1s
service/paas-ta-container-platform-harbor-chartmuseum     ClusterIP   10.233.28.112   <none>        80/TCP                        4m1s
service/paas-ta-container-platform-harbor-core            ClusterIP   10.233.14.27    <none>        80/TCP                        4m
service/paas-ta-container-platform-harbor-database        ClusterIP   10.233.9.5      <none>        5432/TCP                      4m1s
service/paas-ta-container-platform-harbor-jobservice      ClusterIP   10.233.5.82     <none>        80/TCP                        4m1s
service/paas-ta-container-platform-harbor-notary-server   ClusterIP   10.233.22.118   <none>        4443/TCP                      4m1s
service/paas-ta-container-platform-harbor-notary-signer   ClusterIP   10.233.57.71    <none>        7899/TCP                      4m1s
service/paas-ta-container-platform-harbor-portal          ClusterIP   10.233.48.102   <none>        80/TCP                        4m1s
service/paas-ta-container-platform-harbor-redis           ClusterIP   10.233.34.72    <none>        6379/TCP                      4m1s
service/paas-ta-container-platform-harbor-registry        ClusterIP   10.233.41.153   <none>        5000/TCP,8080/TCP             4m1s
service/paas-ta-container-platform-harbor-trivy           ClusterIP   10.233.33.126   <none>        8080/TCP                      4m1s

NAME                                                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/paas-ta-container-platform-harbor-chartmuseum     1/1     1            1           4m
deployment.apps/paas-ta-container-platform-harbor-core            1/1     1            1           4m
deployment.apps/paas-ta-container-platform-harbor-jobservice      1/1     1            1           4m
deployment.apps/paas-ta-container-platform-harbor-nginx           1/1     1            1           4m
deployment.apps/paas-ta-container-platform-harbor-notary-server   1/1     1            1           4m
deployment.apps/paas-ta-container-platform-harbor-notary-signer   1/1     1            1           4m
deployment.apps/paas-ta-container-platform-harbor-portal          1/1     1            1           4m
deployment.apps/paas-ta-container-platform-harbor-registry        1/1     1            1           4m

NAME                                                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/paas-ta-container-platform-harbor-chartmuseum-59b784bfd4    1         1         1       4m
replicaset.apps/paas-ta-container-platform-harbor-core-6b7b8768f9           1         1         1       4m
replicaset.apps/paas-ta-container-platform-harbor-jobservice-69556cd7cd     1         1         1       4m
replicaset.apps/paas-ta-container-platform-harbor-nginx-755cf6876f          1         1         1       4m
replicaset.apps/paas-ta-container-platform-harbor-notary-server-896c47dcd   1         1         1       4m
replicaset.apps/paas-ta-container-platform-harbor-notary-signer-b747fb89d   1         1         1       4m
replicaset.apps/paas-ta-container-platform-harbor-portal-c79c78f5b          1         1         1       4m
replicaset.apps/paas-ta-container-platform-harbor-registry-54c4f69579       1         1         1       4m

NAME                                                          READY   AGE
statefulset.apps/paas-ta-container-platform-harbor-database   1/1     4m
statefulset.apps/paas-ta-container-platform-harbor-redis      1/1     4m
statefulset.apps/paas-ta-container-platform-harbor-trivy      1/1     4m
```  

- **MariaDB 리소스 조회**
>`$ kubectl get all -n mariadb`       
```
$ kubectl get all -n mariadb
NAME                                       READY   STATUS    RESTARTS   AGE
pod/paas-ta-container-platform-mariadb-0   1/1     Running   0          3m4s

NAME                                         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/paas-ta-container-platform-mariadb   NodePort   10.233.37.208   <none>        3306:31306/TCP   3m4s

NAME                                                  READY   AGE
statefulset.apps/paas-ta-container-platform-mariadb   1/1     3m4s
```    

- **Keycloak 리소스 조회**
>`$ kubectl get all -n keycloak`     
```
$ kubectl get all -n keycloak
NAME                            READY   STATUS    RESTARTS   AGE
pod/keycloak-7d47d9f577-7vfzj   1/1     Running   0          4m18s
pod/keycloak-7d47d9f577-zfwj7   1/1     Running   0          4m18s

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/keycloak           NodePort    10.233.42.192   <none>        8080:32710/TCP   4m18s
service/keycloak-cluster   ClusterIP   None            <none>        8080/TCP         4m18s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/keycloak   2/2     2            2           4m18s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/keycloak-7d47d9f577   2         2         2       4m18s
```

- **컨테이너 플랫폼 포털 리소스 조회**
>`$ kubectl get all -n paas-ta-container-platform-portal`        
```
$ kubectl get all -n paas-ta-container-platform-portal
NAME                                                            READY   STATUS    RESTARTS   AGE
pod/container-platform-api-deployment-55d995498d-9vlpg          1/1     Running   0          5m9s
pod/container-platform-common-api-deployment-5c959bcdf6-kwfpt   1/1     Running   0          5m8s
pod/container-platform-webadmin-deployment-b75dc47cc-sxvl2      1/1     Running   0          5m8s
pod/container-platform-webuser-deployment-d4755ccf7-sn48b       1/1     Running   0          5m7s

NAME                                            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/container-platform-api-service          NodePort   10.233.52.211   <none>        3333:32701/TCP   5m9s
service/container-platform-common-api-service   NodePort   10.233.44.252   <none>        3334:32700/TCP   5m8s
service/container-platform-webadmin-service     NodePort   10.233.12.200   <none>        8090:32703/TCP   5m8s
service/container-platform-webuser-service      NodePort   10.233.15.118   <none>        8091:32702/TCP   5m7s

NAME                                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/container-platform-api-deployment          1/1     1            1           5m9s
deployment.apps/container-platform-common-api-deployment   1/1     1            1           5m8s
deployment.apps/container-platform-webadmin-deployment     1/1     1            1           5m8s
deployment.apps/container-platform-webuser-deployment      1/1     1            1           5m7s

NAME                                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/container-platform-api-deployment-55d995498d          1         1         1       5m9s
replicaset.apps/container-platform-common-api-deployment-5c959bcdf6   1         1         1       5m8s
replicaset.apps/container-platform-webadmin-deployment-b75dc47cc      1         1         1       5m8s
replicaset.apps/container-platform-webuser-deployment-d4755ccf7       1         1         1       5m7s
```    

<br>

#### <div id='3.2.4'>3.2.4. (참조) 컨테이너 플랫폼 포털 리소스 삭제
배포된 컨테이너 플랫폼 포털 리소스의 삭제를 원하는 경우 아래 스크립트를 실행한다.<br>
:loudspeaker: (주의) 컨테이너 플랫폼 포털이 운영되는 상태에서 해당 스크립트 실행 시, **운영에 필요한 리소스가 모두 삭제**되므로 주의가 필요하다.<br>

```
$ cd ~/workspace/container-platform/paas-ta-container-platform-portal-deployment/script
$ chmod +x uninstall-container-platform-portal.sh
$ ./uninstall-container-platform-portal.sh
```
```    
Are you sure you want to delete the container platform portal? <y/n> y
....
release "paas-ta-container-platform-harbor" uninstalled
release "paas-ta-container-platform-mariadb" uninstalled
release "paas-ta-container-platform-keycloak" uninstalled
release "paas-ta-container-platform-nfs-storageclass" uninstalled
release "paas-ta-container-platform-api" uninstalled
release "paas-ta-container-platform-common-api" uninstalled
release "paas-ta-container-platform-webadmin" uninstalled
release "paas-ta-container-platform-webuser" uninstalled
namespace "harbor" deleted
namespace "mariadb" deleted
namespace "keycloak" deleted
namespace "nfs-storageclass" deleted
namespace "paas-ta-container-platform-portal" deleted
"paas-ta-container-platform-repository" has been removed from your repositories
Uninstalled plugin: cm-push
....    
```

<br>    
    

## <div id='4'>4. 컨테이너 플랫폼 운영자/사용자 포털 접속  
컨테이너 플랫폼 운영자/사용자 포털은 아래 주소로 접속 가능하다.<br>
{K8S_MASTER_NODE_IP} 값은 **Kubernetes Master Node Public IP** 값을 입력한다.

- 컨테이너 플랫폼 운영자포털 접속 URI : **http://{K8S_MASTER_NODE_IP}:32703** <br>
- 컨테이너 플랫폼 사용자포털 접속 URI : **http://{K8S_MASTER_NODE_IP}:32702** <br>

<br>
    
### <div id='4.1'/>4.1. 컨테이너 플랫폼 운영자 포털 로그인
컨테이너 플랫폼 운영자 포털 접속 초기 정보는 아래와 같다.
- http://{K8S_MASTER_NODE_IP}:32703에 접속한다.   
- username : **admin** / password : **admin** 계정으로 컨테이너 플랫폼 운영자 포털에 로그인한다.
![image 002]
    
<br>    


### <div id='4.2'/>4.2. 컨테이너 플랫폼 사용자 포털 회원가입/로그인
#### 사용자 회원가입    
- http://{K8S_MASTER_NODE_IP}:32702에 접속한다.
- 하단의 'Register' 버튼을 클릭한다.
![image 003]
    
- 등록할 사용자 계정정보를 입력 후 'Register' 버튼을 클릭하여 컨테이너 플랫폼 사용자 포털에 회원가입한다.
![image 004]  

- 컨테이너 플랫폼 사용자 포털은 회원가입 후 바로 이용이 불가하며 Cluster 관리자 혹은 Namespace 관리자로부터 해당 사용자가 이용할 Namespace와 Role을 할당 받은 후 포털 이용이 가능하다.
Namespace와 Role 할당은 [[4.3. 컨테이너 플랫폼 사용자/운영자 포털 사용 가이드]](#4.3) 를 참고한다.    
![image 005]    

#### 사용자 로그인   
- http://{K8S_MASTER_NODE_IP}:32702에 접속한다.
- 회원가입을 통해 등록된 계정으로 컨테이너 플랫폼 사용자 포털에 로그인한다.
![image 006]
    
<br>    

### <div id='4.3'/>4.3. 컨테이너 플랫폼 사용자/운영자 포털 사용 가이드
- 컨테이너 플랫폼 포털 사용방법은 아래 사용가이드를 참고한다.  
  + [컨테이너 플랫폼 운영자 포털 사용 가이드](../../use-guide/portal/container-platform-admin-portal-guide.md)    
  + [컨테이너 플랫폼 사용자 포털 사용 가이드](../../use-guide/portal/container-platform-user-portal-guide.md)


<br>

## <div id='5'>5. 컨네이너 플랫폼 포털 참고

### <div id='5.1'>5.1. 운영자 Cluster Role Token 생성
Cluster Role을 가진 운영자의 Service Account를 생성하고 해당 Service Account의 Token 값을 획득한다.<br>
획득한 Token 값은 컨테이너 플랫폼 포털 배포 시 사용된다.

- Service Account를 생성한다.
```
## {SERVICE_ACCOUNT} : 생성할 Service Account 명

$ kubectl create serviceaccount {SERVICE_ACCOUNT} -n kube-system
(ex. kubectl create serviceaccount k8sadmin -n kube-system)
```

- 생성한 Service Account와 kubernetes에서 제공하는 ClusterRole 'cluster-admin'을 바인딩한다.
```
$ kubectl create clusterrolebinding {SERVICE_ACCOUNT} --clusterrole=cluster-admin --serviceaccount=kube-system:{SERVICE_ACCOUNT}
(ex. kubectl create clusterrolebinding k8sadmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8sadmin)
```

- Service Account의 Mountable secrets 값을 확인한다.
```
$ kubectl describe serviceaccount {SERVICE_ACCOUNT} -n kube-system
(ex. kubectl describe serviceaccount k8sadmin -n kube-system)

...

Mountable secrets:   k8sadmin-token-xxxx
```

- Service Account의 Token을 획득한다.

```
## {SECRET_NAME} : Mountable secrets 값 입력

$ kubectl describe secret {SECRET_NAME} -n kube-system | grep -E '^token' | cut -f2 -d':' | tr -d " "
```

<br>

### <div id='5.2'>5.2. Kubernetes 리소스 생성 시 주의사항

컨테이너 플랫폼 이용 중 리소스 생성 시 다음과 같은 prefix를 사용하지 않도록 주의한다.

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

### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 단독형 배포 포털 설치 가이드

[image 001]:images-v1.2/cp-001.png
[image 002]:images-v1.2/cp-002.png
[image 003]:images-v1.2/cp-003.png
[image 004]:images-v1.2/cp-004.png
[image 005]:images-v1.2/cp-005.png
[image 006]:images-v1.2/cp-006.png
[image 007]:images-v1.2/cp-007.png
[image 008]:images-v1.2/cp-008.png
[image 009]:images-v1.2/cp-009.png
[image 010]:images-v1.2/cp-010.png
[image 011]:images-v1.2/cp-011.png