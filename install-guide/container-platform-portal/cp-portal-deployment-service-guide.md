### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 서비스형 배포 포털 설치 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
    1.1. [목적](#1.1)  
    1.2. [범위](#1.2)  
    1.3. [시스템 구성도](#1.3)  
    1.4. [참고 자료](#1.4)  

2. [Prerequisite](#2)  
    2.1. [방화벽 정보](#2.1)  

3. [컨테이너 플랫폼 포털 배포](#3)  
    3.1. [컨테이너 플랫폼 포털 배포](#3.1)  
    3.1.1. [컨테이너 플랫폼 포털 Deployment 파일 다운로드](#3.1.1)  
    3.1.2. [컨테이너 플랫폼 포털 변수 정의](#3.1.2)    
    3.1.3. [컨테이너 플랫폼 포털 배포 스크립트 실행](#3.1.3)  
    3.1.4. [(참조) 컨테이너 플랫폼 포털 리소스 삭제](#3.1.4)

4. [컨테이너 플랫폼 포털 사용자 인증 서비스 구성](#4)      
    4.1. [컨테이너 플랫폼 포털 사용자 인증 구성 Deployment 다운로드](#4.1)      
    4.2. [컨테이너 플랫폼 포털 사용자 인증 구성 변수 정의](#4.2)      
    4.3. [컨테이너 플랫폼 포털 사용자 인증 구성 스크립트 실행](#4.3)          
    4.4. [(참조) 컨테이너 플랫폼 포털 사용자 인증 구성 해제](#4.4)    

5. [컨테이너 플랫폼 포털 서비스 브로커](#5)       
    5.1. [컨테이너 플랫폼 포털 서비스 브로커 등록](#5.1)  
    5.2. [컨테이너 플랫폼 포털 서비스 조회 설정](#5.2)    
    5.3. [컨테이너 플랫폼 포털 사용 가이드](#5.3)

6. [컨네이너 플랫폼 포털 참고](#6)   
    6.1. [Kubernetes 리소스 생성 시 주의사항](#6.1)      


## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(컨테이너 플랫폼 서비스 배포 형 포털 설치 가이드)는 Kubernetes Cluster를 설치하고 컨테이너 플랫폼 서비스 배포 형 포털 배포 방법을 기술하였다.<br>
<br>

### <div id='1.2'>1.2. 범위
설치 범위는 Kubernetes Cluster 배포를 기준으로 작성하였다.

<br>

### <div id='1.3'>1.3. 시스템 구성도
<p align="center"><img src="images/cp-001.png" width="850" height="530"></p>

시스템 구성은 **Kubernetes Cluster(Master, Worker)** 환경과 데이터 관리를 위한 스토리지 서버로 구성되어 있다.
Kubespray를 통해 설치된 Kubernetes Cluster 환경에 컨테이너 플랫폼 포털 이미지 및 Helm Chart를 관리하는 **Harbor**, 컨테이너 플랫폼 포털 사용자 인증을 관리하는 **Keycloak**, 인증 데이터를 관리하는 **Vault**, 메타 데이터를 관리하는 **MariaDB(RDBMS)** 등 미들웨어 환경을 컨테이너로 제공한다. 
총 필요한 VM 환경으로는 **Master VM: 1개, Worker VM: 3개 이상**이 필요하고 본 문서는 Kubernetes Cluster에 컨테이너 플랫폼 포털 환경을 배포하는 내용이다. 

<br> 

### <div id='1.4'>1.4. 참고 자료
> https://kubernetes.io/ko/docs<br>
> https://goharbor.io/docs<br>
> https://www.keycloak.org/documentation

<br>

## <div id='2'>2. Prerequisite
본 설치 가이드는 **Ubuntu 20.04** 환경에서 설치하는 것을 기준으로 작성하였다.

### <div id='2.1'>2.1. 방화벽 정보
IaaS Security Group의 열어줘야할 Port를 설정한다.

- Master Node

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

## <div id='3'>3. 컨테이너 플랫폼 포털 배포

### <div id='3.1'>3.1. 컨테이너 플랫폼 포털 배포

#### <div id='3.1.1'>3.1.1. 컨테이너 플랫폼 포털 Deployment 파일 다운로드
컨테이너 플랫폼 포털 배포를 위해 컨테이너 플랫폼 포털 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
:bulb: 해당 내용은 Kubernetes **Master Node**에서 진행한다.

+ 컨테이너 플랫폼 포털 Deployment 파일 다운로드 :
   [cp-portal-deployment-v1.4.0.tar.gz](https://nextcloud.paas-ta.org/index.php/s/WtNQn2agk6epFHC/download)

```
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.paas-ta.org/index.php/s/WtNQn2agk6epFHC/download

$ ls ~/workspace/container-platform
  cp-portal-deployment-v1.4.0.tar.gz

# Deployment 파일 압축 해제
$ tar -xvf cp-portal-deployment-v1.4.0.tar.gz
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

#### <div id='3.1.2'>3.1.2. 컨테이너 플랫폼 포털 변수 정의
컨테이너 플랫폼 포털을 배포하기 전 변수 값 정의가 필요하다. 배포에 필요한 정보를 확인하여 변수를 설정한다.

:bulb: Keycloak 기본 배포 방식은 **HTTP**이며 인증서를 통한 **HTTPS**를 설정하고자 하는 경우 아래 가이드를 참조하여 선처리한다.
> [Keycloak TLS 설정](paas-ta-container-platform-portal-deployment-keycloak-tls-setting-guide.md#2-keycloak-tls-설정)       

<br>

```
$ cd ~/workspace/container-platform/cp-portal-deployment/script
$ vi cp-portal-vars.sh
```

```                                                     
# COMMON VARIABLE (Please change the values of the four variables below.)
K8S_MASTER_NODE_IP="{k8s master node public ip}"            # Kubernetes Master Node Public IP
HOST_CLUSTER_IAAS_TYPE="{host cluster iaas type}"           # Host Cluster IaaS Type (Please enter 'AWS' or 'OPENSTACK')
PROVIDER_TYPE="{container platform portal provider type}"   # Container Platform Portal Provider Type (Please enter 'standalone' or 'service')
....    
```
```    
# Example
K8S_MASTER_NODE_IP="xx.xxx.xxx.xx"
HOST_CLUSTER_IAAS_TYPE="AWS"
PROVIDER_TYPE="service"
```

- **K8S_MASTER_NODE_IP** <br>Kubernetes Master Node Public IP 입력<br><br>
- **HOST_CLUSTER_IAAS_TYPE** <br>Kubernetes Cluster IaaS 환경 입력 <br><br>
- **PROVIDER_TYPE** <br>컨테이너 플랫폼 포털 제공 타입 입력 <br>
   + 본 가이드는 포털 서비스 배포 형 설치 가이드로 **'service'** 값 입력 필요

<br>


#### <div id='3.1.3'>3.1.3. 컨테이너 플랫폼 포털 배포 스크립트 실행
컨테이너 플랫폼 포털 배포를 위한 배포 스크립트를 실행한다.

```
$ chmod +x deploy-cp-portal.sh
$ ./deploy-cp-portal.sh
```
<br>

컨테이너 플랫폼 포털 관련 리소스가 정상적으로 배포되었는지 확인한다.<br>
리소스 Pod의 경우 Node에 바인딩 및 컨테이너 생성 후 Running 상태로 전환되기까지 몇 초가 소요된다.


- **Harbor 리소스 조회**
>`$ kubectl get all -n harbor`      
```
$ kubectl get all -n harbor
NAME                                           READY   STATUS    RESTARTS        AGE
pod/cp-harbor-chartmuseum-6fdd486868-266t9     1/1     Running   0               3m14s
pod/cp-harbor-core-794489c7b4-hbtmb            1/1     Running   0               3m14s
pod/cp-harbor-database-0                       1/1     Running   0               3m14s
pod/cp-harbor-jobservice-5fdbf6cb6b-vjxxd      1/1     Running   3 (2m27s ago)   3m14s
pod/cp-harbor-nginx-6db895bdbb-dp7zk           1/1     Running   0               3m14s
pod/cp-harbor-notary-server-57676cff76-7f292   1/1     Running   0               3m14s
pod/cp-harbor-notary-signer-64cc867bbb-hwvxp   1/1     Running   0               3m14s
pod/cp-harbor-portal-7f9d57dcf4-5f68g          1/1     Running   0               3m14s
pod/cp-harbor-redis-0                          1/1     Running   0               3m14s
pod/cp-harbor-registry-5fdf76f6cf-jhsxl        2/2     Running   0               3m14s
pod/cp-harbor-trivy-0                          1/1     Running   0               3m14s

NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                       AGE
service/cp-harbor-chartmuseum     ClusterIP   10.233.10.224   <none>        80/TCP                        3m15s
service/cp-harbor-core            ClusterIP   10.233.7.180    <none>        80/TCP                        3m15s
service/cp-harbor-database        ClusterIP   10.233.55.195   <none>        5432/TCP                      3m15s
service/cp-harbor-jobservice      ClusterIP   10.233.19.242   <none>        80/TCP                        3m15s
service/cp-harbor-notary-server   ClusterIP   10.233.55.160   <none>        4443/TCP                      3m15s
service/cp-harbor-notary-signer   ClusterIP   10.233.15.151   <none>        7899/TCP                      3m15s
service/cp-harbor-portal          ClusterIP   10.233.30.190   <none>        80/TCP                        3m15s
service/cp-harbor-redis           ClusterIP   10.233.29.164   <none>        6379/TCP                      3m15s
service/cp-harbor-registry        ClusterIP   10.233.49.111   <none>        5000/TCP,8080/TCP             3m15s
service/cp-harbor-trivy           ClusterIP   10.233.48.64    <none>        8080/TCP                      3m15s
service/harbor                    NodePort    10.233.46.29    <none>        80:30002/TCP,4443:30004/TCP   3m15s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cp-harbor-chartmuseum     1/1     1            1           3m15s
deployment.apps/cp-harbor-core            1/1     1            1           3m15s
deployment.apps/cp-harbor-jobservice      1/1     1            1           3m15s
deployment.apps/cp-harbor-nginx           1/1     1            1           3m15s
deployment.apps/cp-harbor-notary-server   1/1     1            1           3m15s
deployment.apps/cp-harbor-notary-signer   1/1     1            1           3m15s
deployment.apps/cp-harbor-portal          1/1     1            1           3m15s
deployment.apps/cp-harbor-registry        1/1     1            1           3m15s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/cp-harbor-chartmuseum-6fdd486868     1         1         1       3m15s
replicaset.apps/cp-harbor-core-794489c7b4            1         1         1       3m15s
replicaset.apps/cp-harbor-jobservice-5fdbf6cb6b      1         1         1       3m15s
replicaset.apps/cp-harbor-nginx-6db895bdbb           1         1         1       3m15s
replicaset.apps/cp-harbor-notary-server-57676cff76   1         1         1       3m15s
replicaset.apps/cp-harbor-notary-signer-64cc867bbb   1         1         1       3m15s
replicaset.apps/cp-harbor-portal-7f9d57dcf4          1         1         1       3m15s
replicaset.apps/cp-harbor-registry-5fdf76f6cf        1         1         1       3m15s

NAME                                  READY   AGE
statefulset.apps/cp-harbor-database   1/1     3m15s
statefulset.apps/cp-harbor-redis      1/1     3m15s
statefulset.apps/cp-harbor-trivy      1/1     3m15s
```  

- **MariaDB 리소스 조회**
>`$ kubectl get all -n mariadb`       
```
$ kubectl get all -n mariadb
NAME               READY   STATUS    RESTARTS   AGE
pod/cp-mariadb-0   1/1     Running   0          96s

NAME                 TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/cp-mariadb   NodePort   10.233.19.252   <none>        3306:31306/TCP   97s

NAME                          READY   AGE
statefulset.apps/cp-mariadb   1/1     97s
```    

- **Keycloak 리소스 조회**
>`$ kubectl get all -n keycloak`     
```
$ kubectl get all -n keycloak
NAME                               READY   STATUS    RESTARTS      AGE
pod/cp-keycloak-7d49f84bc6-qdljr   1/1     Running   1 (55s ago)   119s
pod/cp-keycloak-7d49f84bc6-xbg92   1/1     Running   0             119s

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/cp-keycloak           NodePort    10.233.41.247   <none>        8080:32710/TCP   119s
service/cp-keycloak-cluster   ClusterIP   None            <none>        8080/TCP         119s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cp-keycloak   2/2     2            2           119s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/cp-keycloak-7d49f84bc6   2         2         2       119s
```

- **컨테이너 플랫폼 포털 리소스 조회**
>`$ kubectl get all -n cp-portal`        
```
$ kubectl get all -n cp-portal
NAME                                                      READY   STATUS    RESTARTS   AGE
pod/cp-portal-api-deployment-595dd4dfb6-smt9v             1/1     Running   0          69s
pod/cp-portal-common-api-deployment-c54d88fbc-fq27x       1/1     Running   0          66s
pod/cp-portal-metric-api-deployment-6599f47b4b-p8sm4      1/1     Running   0          61s
pod/cp-portal-service-broker-deployment-59b99677b-grnkw   1/1     Running   0          52s
pod/cp-portal-terraman-deployment-5ccfbf67fc-2tdcq        1/1     Running   0          59s
pod/cp-portal-ui-deployment-669db699c-8lmlr               1/1     Running   0          74s

NAME                                       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/cp-portal-api-service              NodePort   10.233.32.224   <none>        3333:32701/TCP   69s
service/cp-portal-common-api-service       NodePort   10.233.49.204   <none>        3334:32700/TCP   66s
service/cp-portal-metric-api-service       NodePort   10.233.19.109   <none>        8900:30329/TCP   61s
service/cp-portal-service-broker-service   NodePort   10.233.11.159   <none>        3330:32704/TCP   52s
service/cp-portal-terraman-service         NodePort   10.233.12.100   <none>        8091:32707/TCP   59s
service/cp-portal-ui-service               NodePort   10.233.54.138   <none>        8090:32703/TCP   74s

NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cp-portal-api-deployment              1/1     1            1           69s
deployment.apps/cp-portal-common-api-deployment       1/1     1            1           66s
deployment.apps/cp-portal-metric-api-deployment       1/1     1            1           61s
deployment.apps/cp-portal-service-broker-deployment   1/1     1            1           52s
deployment.apps/cp-portal-terraman-deployment         1/1     1            1           59s
deployment.apps/cp-portal-ui-deployment               1/1     1            1           74s

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/cp-portal-api-deployment-595dd4dfb6             1         1         1       69s
replicaset.apps/cp-portal-common-api-deployment-c54d88fbc       1         1         1       66s
replicaset.apps/cp-portal-metric-api-deployment-6599f47b4b      1         1         1       61s
replicaset.apps/cp-portal-service-broker-deployment-59b99677b   1         1         1       52s
replicaset.apps/cp-portal-terraman-deployment-5ccfbf67fc        1         1         1       59s
replicaset.apps/cp-portal-ui-deployment-669db699c               1         1         1       74s
```    

<br>

#### <div id='3.1.4'>3.1.4. (참조) 컨테이너 플랫폼 포털 리소스 삭제
배포된 컨테이너 플랫폼 포털 리소스의 삭제를 원하는 경우 아래 스크립트를 실행한다.<br>
:loudspeaker: (주의) 컨테이너 플랫폼 포털이 운영되는 상태에서 해당 스크립트 실행 시, **운영에 필요한 리소스가 모두 삭제**되므로 주의가 필요하다.<br>

```
$ cd ~/workspace/container-platform/cp-portal-deployment/script
$ chmod +x uninstall-cp-portal.sh
$ ./uninstall-cp-portal.sh
```
```    
Are you sure you want to delete the container platform portal? <y/n> y
.... 
    
release "cp-harbor" uninstalled
namespace "harbor" deleted
release "cp-mariadb" uninstalled
namespace "mariadb" deleted
release "cp-keycloak" uninstalled
namespace "keycloak" deleted
release "cp-portal-api" uninstalled
release "cp-portal-common-api" uninstalled
release "cp-portal-configmap" uninstalled
release "cp-portal-metric-api" uninstalled
release "cp-portal-service-broker" uninstalled
release "cp-portal-terraman" uninstalled
release "cp-portal-ui" uninstalled
namespace "cp-portal" deleted
"cp-portal-repository" has been removed from your repositories
Uninstalled plugin: cm-push

....    
```

<br>    
  

## <div id='4'>4. 컨테이너 플랫폼 포털 사용자 인증 서비스 구성
컨테이너 플랫폼 포털 사용자 인증은 Keycloak 서비스를 통해 관리된다. PaaS-TA 포털의 사용자 인증 서비스 UAA의 사용자 계정으로 컨테이너 플랫폼 포털 접속을 위해
UAA 서비스를 ID 제공자(Identity Provider)로, Keycloak 서비스를 서비스 제공자(Service Provider)로 구성하는 단계가 필요하다.

#### <div id='4.1'>4.1. 컨테이너 플랫폼 포털 사용자 인증 구성 Deployment 다운로드
UAA 서비스와 Keycloak 서비스 인증 구성을 위한 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
:bulb: 해당 내용은 PaaS-TA 포털이 설치된 **BOSH Inception**에서 진행한다.

+ 컨테이너 플랫폼 포털 사용자 인증 구성 Deployment 다운로드 :  
   [cp-saml-deployment-v1.4.0.tar.gz](https://nextcloud.paas-ta.org/index.php/s/MajerbG3ZHQZQJ8/download)  

```
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.paas-ta.org/index.php/s/MajerbG3ZHQZQJ8/download

$ ls ~/workspace/container-platform
  cp-saml-deployment-v1.4.0.tar.gz

# Deployment 파일 압축 해제
$ tar -xvf cp-saml-deployment-v1.4.0.tar.gz
```
<br>

#### <div id='4.2'>4.2. 컨테이너 플랫폼 포털 사용자 인증 구성 변수 정의
UAA 서비스와 Keycloak 서비스 인증 구성을 위한 변수 값 정의가 필요하다. 구성에 필요한 정보를 확인하여 변수를 설정한다.

:bulb: **Keycloak TLS HTTPS** 설정이 적용된 경우, Keycloak URL 변수 값 변경이 필요하다. <br>
아래 가이드를 참조하여 변수 값을 변경한다.
> [(서비스형 배포) 사용자 인증 서비스 구성 변경](paas-ta-container-platform-portal-deployment-keycloak-tls-setting-guide.md#3-서비스형-배포-사용자-인증-서비스-구성-변경)       

<br>

```
$ cd ~/workspace/container-platform/cp-saml-deployment
$ vi cp-saml-vars.sh
```

```                                                     
# COMMON VARIABLE
PAASTA_SYSTEM_DOMAIN="xx.xxx.xx.xxx.nip.io"                       # PaaS-TA System Domain
K8S_MASTER_NODE_IP="xx.xxx.xx.xxx"                                # Kubernetes Master Node Public IP
UAA_CLIENT_ADMIN_ID="admin"                                       # UAA Admin Client ID (e.g. admin)
UAA_CLIENT_ADMIN_SECRET="admin-secret"                            # UAA Admin Client Secret (e.g. admin-secret)
....    
```


- **PAASTA_SYSTEM_DOMAIN** <br> PaaS-TA 배포 시 지정했던 System Domain 명 입력<br><br>
- **K8S_MASTER_NODE_IP** <br>Kubernetes Master Node Public IP 입력<br><br>
- **UAA_CLIENT_ADMIN_ID** <br>UAAC Admin Client Admin ID 입력 (기본 값 : admin)<br><br>
- **UAA_CLIENT_ADMIN_SECRET** <br>UAAC Admin Client에 접근하기 위한 Secret 변수 (기본 값 : admin-secret)<br><br>

<br>

#### <div id='4.3'>4.3. 컨테이너 플랫폼 포털 사용자 인증 구성 스크립트 실행
UAA 서비스와 Keycloak 서비스 인증 구성을 위한 스크립트를 실행한다.

```
$ chmod +x create-service-provider.sh
$ ./create-service-provider.sh
```

<br>

구성이 정상적으로 처리되었는지 확인한다. (**RESPONSE BODY 내 결과 확인**)
- UAAC Service Providers 조회   
>`$ uaac curl /saml/service-providers --insecure`     
```    
$ uaac curl /saml/service-providers --insecure
GET https://uaa.xx.xxx.xxx.xx.nip.io/saml/service-providers

200 OK
RESPONSE HEADERS:
  Cache-Control: no-cache, no-store, max-age=0, must-revalidate
  Content-Type: application/json
....   
RESPONSE BODY:
[
  {
    "config": "{\"metaDataLocation\": .... }",
    "id": "c86dd09a-2d47-4005-943d-a4fc717efd0e",
    "entityId": "http://xx.xxx.xxx.xx:32710/auth/realms/cp-realm",
    "name": "cp-saml-sp",
    "version": 0,
    "created": 1651815898042,
    "lastModified": 1651815898042,
    "active": true,
    "identityZoneId": "uaa"
  }
]
```    

<br>

#### <div id='4.4'>4.4. (참조) 컨테이너 플랫폼 포털 사용자 인증 구성 해제
UAA 서비스와 Keycloak 서비스 인증 구성 해제를 원하는 경우 아래 스크립트를 실행한다.<br>
:loudspeaker: (주의) 컨테이너 플랫폼 포털이 운영되는 상태에서 해당 스크립트 실행 시, 사용자 인증 구성이 불가하므로 주의가 필요하다.<br>


##### 해제할 Service Provider ID 조회
UAAC Service Providers 조회 후 **RESPONSE BODY** 결과 내 아래 조건을 가진 **Service Provider ID**를 조회한다.
- `entityId : http://{K8S_MASTER_NODE_IP}:32710/auth/realms/cp-realm` <br>
- `name : cp-saml-sp` <br>

```  
$ uaac curl /saml/service-providers --insecure

....
RESPONSE BODY:
[
  {
    "config": "{\"metaDataLocation\": .... }",
    "id": "c86dd09a-2d47-4005-943d-a4fc717efd0e",   # 해제할 Service Provider ID
    "entityId": "http://xx.xxx.xxx.xx:32710/auth/realms/cp-realm",
    "name": "cp-saml-sp",
    "version": 0,
    "created": 1651815898042,
    "lastModified": 1651815898042,
    "active": true,
    "identityZoneId": "uaa"
  }
....    
]
```    

<br>

해제할 **Service Provider ID** 조회 후 인증 구성 해제 스크립트를 실행한다.

```
$ cd ~/workspace/container-platform/cp-saml-deployment
$ chmod +x uninstall-service-provider.sh
$ ./uninstall-service-provider.sh {Service_Provider_ID}
```

```    
$ ./uninstall-service-provider.sh c86dd09a-2d47-4005-943d-a4fc717efd0e
....  
Are you sure you want to delete this service provider? <y/n> y
DELETE https://uaa.xx.xxx.xxx.xx.nip.io/saml/service-providers/c86dd09a-2d47-4005-943d-a4fc717efd0e
....    
```

<br>

## <div id='5'>5. 컨테이너 플랫폼 포털 서비스 브로커
컨테이너 플랫폼 PaaS-TA 서비스 형 포털로 설치하는 경우 CF와 Kubernetes에 배포된 컨테이너 플랫폼 포털 서비스 연동을 위해서 브로커를 등록해 주어야 한다.
PaaS-TA 운영자 포털을 통해 서비스를 등록하고 공개하면, PaaS-TA 사용자 포털을 통해 서비스를 신청하여 사용할 수 있다.

### <div id='5.1'>5.1. 컨테이너 플랫폼 포털 서비스 브로커 등록
서비스 브로커 등록 시 개방형 클라우드 플랫폼에서 서비스 브로커를 등록할 수 있는 사용자로 로그인이 되어있어야 한다.

##### 서비스 브로커 목록을 확인한다.
>`$ cf service-brokers`
```
$ cf service-brokers
Getting service brokers as admin...
No service brokers found
```


##### 컨테이너 플랫폼 포털 서비스 브로커를 등록한다.
>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL}`

서비스팩 이름 : 서비스팩 관리를 위해 개방형 클라우드 플랫폼에서 보여지는 명칭<br>
서비스팩 사용자 ID/비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID/비밀번호<br>
서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL<br>


###### 컨테이너 플랫폼 포털 서비스 브로커 등록
>`$ cf create-service-broker cp-portal-service-broker admin cloudfoundry http://{K8S_MASTER_NODE_IP}:32704`   

```
$ cf create-service-broker cp-portal-service-broker admin cloudfoundry http://xx.xxx.xxx.xx:32704
Creating service broker cp-portal-service-broker as admin...
OK
```    

##### 등록된 컨테이너 플랫폼 포털 서비스 브로커를 확인한다.
>`$ cf service-brokers`
```
$ cf service-brokers
Getting service brokers as admin...
name                       url
cp-portal-service-broker   http://xx.xxx.xxx.xx:32704
```

##### 특정 조직에 해당 서비스 접근 허용을 할당한다.

###### 컨테이너 플랫폼 운영자 포털 서비스 접근 허용 할당  
>`$ cf enable-service-access cp-portal-service-broker`   

```
$ cf enable-service-access cp-portal-service-broker
Enabling access to all plans of service offering cp-portal-service-broker for all orgs as admin...
OK
```


##### 접근 가능한 서비스 목록을 확인한다.
>`$ cf service-access`

```
$ cf service-access
Getting service access as admin...

broker: cp-portal-service-broker
   offering                   plan       access   orgs
   cp-portal-service-broker   Advenced   all
```

<br>

### <div id='5.2'>5.2. 컨테이너 플랫폼 포털 서비스 조회 설정
해당 설정은 PaaS-TA 포털에서 컨테이너 플랫폼 포털 서비스를 조회하고 신청할 수 있도록 하기 위한 설정이다.

##### PaaS-TA 운영자 포털에 접속한다.
![image 007]


##### 메뉴 [운영관리]-[카탈로그] 의 '앱 서비스' 목록 중 'Container Platform Portal 서비스' 를 선택한다.
![image 008]

##### 'Container Platform Portal 서비스' 상세 정보를 아래와 같이 설정 후 저장한다.
>`'서비스' 항목 : 'cp-portal-service-broker' 로 선택` <br>
>`'공개' 항목 : 'Y' 로 체크`
    
![image 009]

##### PaaS-TA 사용자 포털에 접속하여 컨테이너 플랫폼 포털 서비스를 생성한다.
![image 010]

<br>

### <div id='5.3'/>5.3. 컨테이너 플랫폼 포털 사용 가이드
- 컨테이너 플랫폼 포털 사용방법은 아래 사용가이드를 참고한다.  
  + [컨테이너 플랫폼 포털 사용 가이드](../../use-guide/portal/container-platform-portal-guide.md)    

<br>

## <div id='6'>6. 컨네이너 플랫폼 포털 참고

### <div id='6.1'>6.1. Kubernetes 리소스 생성 시 주의사항

컨테이너 플랫폼 이용 중 리소스 생성 시 다음과 같은 prefix를 사용하지 않도록 주의한다.

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

### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 서비스형 배포 포털 설치 가이드

[image 001]:images/cp-001.png
[image 002]:images/cp-002.png
[image 003]:images/cp-003.png
[image 004]:images/cp-004.png
[image 005]:images/cp-005.png
[image 006]:images/cp-006.png
[image 007]:images/cp-007.png
[image 008]:images/cp-008.png
[image 009]:images/cp-009.png
[image 010]:images/cp-010.png