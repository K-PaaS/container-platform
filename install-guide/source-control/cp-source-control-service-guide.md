### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 서비스 형 컨테이너 플랫폼 소스 컨트롤 배포 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
   1.1. [목적](#1.1)  
   1.2. [범위](#1.2)  
   1.3. [시스템 구성도](#1.3)  
   1.4. [참고 자료](#1.4)

2. [Prerequisite](#2)  
   2.1. [컨테이너 플랫폼 포털 설치](#2.1)

3. [컨테이너 플랫폼 소스 컨트롤 배포](#3)  
   3.1. [컨테이너 플랫폼 소스 컨트롤 Deployment 파일 다운로드](#3.1)  
   3.2. [컨테이너 플랫폼 소스 컨트롤 변수 정의](#3.2)    
   3.3. [컨테이너 플랫폼 소스 컨트롤 배포 스크립트 실행](#3.3)    
   3.4. [(참고) 컨테이너 플랫폼 소스 컨트롤 리소스 삭제](#3.4)

4. [컨테이너 플랫폼 소스 컨트롤 서비스 브로커](#4)   
   4.1. [컨테이너 플랫폼 소스 컨트롤 사용자 인증 서비스 구성](#4.1)   
   4.2. [컨테이너 플랫폼 소스 컨트롤 서비스 브로커 등록](#4.2)  
   4.3. [컨테이너 플랫폼 소스 컨트롤 서비스 조회 설정](#4.3)    
   4.4. [컨테이너 플랫폼 소스 컨트롤 사용 가이드](#4.4)



## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(서비스 형 컨테이너 플랫폼 소스 컨트롤 배포 가이드)는 쿠버네티스 클러스터 및 서비스 형 컨테이너 플랫폼 포털을 설치하고 서비스 형 소스 컨트롤 배포 방법을 기술하였다.

<br>

### <div id='1.2'>1.2. 범위
설치 범위는 쿠버네티스 클러스터 배포를 기준으로 작성하였다.

<br>

### <div id='1.3'>1.3. 시스템 구성도
<p align="center"><img src="../images/portal/cp-001.png" width="850" height="530"></p>

<br>

시스템 구성은 **Kubernetes Cluster(Master, Worker)** 환경과 데이터 관리를 위한 스토리지 서버로 구성되어 있다. Kubespray를 통해 설치된 Kubernetes Cluster 환경에 컨테이너 플랫폼 소스 컨트롤 이미지 및 Helm Chart를 관리하는 **Harbor**,
컨테이너 플랫폼 소스 컨트롤 사용자 인증을 관리하는 **Keycloak**, 컨테이너 플랫폼 소스 컨트롤 메타 데이터를 관리하는 **MariaDB(RDBMS)** 가 컨테이너 플랫폼 포털을 통해서 제공된다.
컨테이너 플랫폼 소스 컨트롤에서는 소스를 관리하는 **SCM-Server**를 컨테이너로 제공한다. 총 필요한 VM 환경으로는 **Master VM: 1개, Worker VM: 3개 이상**이 필요하고 본 문서는 Kubernetes Cluster에 컨테이너 플랫폼 소스 컨트롤 환경을 배포하는 내용이다.

<br>

### <div id='1.4'>1.4. 참고 자료
> https://kubernetes.io/ko/docs

<br>

## <div id='2'>2. Prerequisite

### <div id='2.1'>2.1. 컨테이너 플랫폼 포털 설치
컨테이너 플랫폼 소스 컨트롤에서 사용할 인프라로 인증서버 **KeyCloak**, 데이터베이스 **MariaDB**, 레포지토리 서버 **Harbor** 설치가 사전에 진행되어야 한다.
컨테이너 플랫폼 포털 배포 시 해당 인프라를 모두 설치하므로 아래 가이드를 참조하여 사전 설치를 진행한다.
> [[서비스 형 컨테이너 플랫폼 포털 배포]](../portal/cp-portal-service-guide.md)

<br>

## <div id='3'>3. 컨테이너 플랫폼 소스 컨트롤 배포
### <div id='3.1'>3.1. 컨테이너 플랫폼 소스 컨트롤 Deployment 파일 다운로드
컨테이너 플랫폼 소스 컨트롤 배포를 위해 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
:bulb: 해당 내용은 Kubernetes **Master Node**에서 진행한다.
+ 컨테이너 플랫폼 소스 컨트롤 Deployment 파일 다운로드 :  
  [cp-source-control-deployment-v1.5.1.tar.gz](https://nextcloud.k-paas.org/index.php/s/N7AxwSdQt7HzNTn/download)

```bash
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.k-paas.org/index.php/s/N7AxwSdQt7HzNTn/download

$ ls ~/workspace/container-platform
  cp-source-control-deployment-v1.5.1.tar.gz ...
  
# Deployment 파일 압축 해제
$ tar -xvf cp-source-control-deployment-v1.5.1.tar.gz
```

- Deployment 파일 디렉토리 구성
```bash
├── script          # 컨테이너 플랫폼 소스 컨트롤 배포 관련 변수 및 스크립트 파일 위치
├── images          # 컨테이너 플랫폼 소스 컨트롤 이미지 파일 위치
├── charts          # 컨테이너 플랫폼 소스 컨트롤 Helm Charts 파일 위치
├── values_orig     # 컨테이너 플랫폼 소스 컨트롤 Helm Charts values 파일 위치
```

<br>

### <div id='3.2'>3.2. 컨테이너 플랫폼 소스 컨트롤 변수 정의
컨테이너 플랫폼 소스 컨트롤을 배포하기 전 변수 값 정의가 필요하다. 배포에 필요한 정보를 확인하여 변수를 설정한다.

```bash
$ cd ~/workspace/container-platform/cp-source-control-deployment/script
$ vi cp-source-control-vars.sh
```
```bash                                                    
K8S_MASTER_NODE_IP="{k8s master node public ip}"                      # Kubernetes master node public ip
K8S_STORAGECLASS="cp-storageclass"                                    # Kubernetes StorageClass Name (e.g. cp-storageclass)
HOST_DOMAIN="{host domain}"                                           # Host Domain (e.g. xx.xxx.xxx.xx.nip.io)
PROVIDER_TYPE="{container platform source control provider type}"     # Container platform source control provider type (Please enter 'standalone' or 'service')    
```
```bash  
K8S_MASTER_NODE_IP="103.xxx.xxx.xxx"
K8S_STORAGECLASS="cp-storageclass"
HOST_DOMAIN="105.xxx.xxx.xxx.nip.io"
PROVIDER_TYPE="service"
```
|변수|설명|상세 내용|
|---|---|---|
|**K8S_MASTER_NODE_IP**|Kubernetes Master Node Public IP 입력|Master Node에 접근하기 어려운 경우<br>[[3.1.2. 컨테이너 플랫폼 포털 변수 정의]](../portal/cp-portal-service-guide.md#3.1.2)에서<br>정의한 `K8S_MASTER_NODE_IP` 값 입력| 
|**K8S_STORAGECLASS**|StorageClass 명 입력|컨테이너 플랫폼을 통해 배포된 클러스터는 <br> 기본으로 <b>`cp-storageclass`</b>이다. <br> 다른 StorageClass 사용 시 해당 리소스 명을 입력한다.|
|**HOST_DOMAIN**|Host Domain 값 입력|[[3.1.2. 컨테이너 플랫폼 포털 변수 정의]](../portal/cp-portal-service-guide.md#3.1.2)에서<br>정의한 `HOST_DOMAIN` 값 입력|
|**PROVIDER_TYPE**|컨테이너 플랫폼 소스 컨트롤 제공 타입 입력|본 가이드는 서비스 배포 형 설치 가이드로<br> **service** 값 입력 필요|
|**IS_MULTI_CLUSTER**|멀티 클러스터 환경 여부|멀티 클러스터 환경에 배포할 경우 "Y" 입력|

<br>

:bulb: Keycloak 기본 배포 프로토콜은 **HTTP**이며 인증서를 통한 **HTTPS**가 설정된 경우<br>
컨테이너 플랫폼 소스 컨트롤 변수 파일 내 아래 내용을 수정한다.

```bash
$ vi cp-source-control-vars.sh
```    
```bash
# KEYCLOAK_URL 변수 값 http -> https 로 변경 
....  
#keycloak
KEYCLOAK_URL="https://keycloak.${HOST_DOMAIN}"     #if apply TLS, https://
....     
```

<br>

### <div id='3.3'>3.3. 컨테이너 플랫폼 소스 컨트롤 배포 스크립트 실행
컨테이너 플랫폼 소스 컨트롤 배포를 위한 배포 스크립트를 실행한다.

```bash
$ chmod +x deploy-cp-source-control.sh
$ ./deploy-cp-source-control.sh
```

<br>

컨테이너 플랫폼 소스 컨트롤 관련 리소스가 정상적으로 배포되었는지 확인한다.<br>
리소스 Pod의 경우 Node에 바인딩 및 컨테이너 생성 후 Running 상태로 전환되기까지 몇 초가 소요된다.

- **컨테이너 플랫폼 소스 컨트롤 리소스 조회**

```bash
$ kubectl get all -n cp-source-control
```

```bash
NAME                                                        READY   STATUS    RESTARTS   AGE
pod/cp-source-control-api-deployment-588fdfbfd7-pngz6       1/1     Running   0          55s
pod/cp-source-control-broker-deployment-84f687c698-8sgbr    1/1     Running   0          51s
pod/cp-source-control-manager-deployment-69b9b87cfd-6mvkd   1/1     Running   0          53s
pod/cp-source-control-ui-deployment-867557b66d-bg2bg        1/1     Running   0          49s

NAME                                        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/cp-source-control-api-service       NodePort   10.233.43.223   <none>        8091:30091/TCP   55s
service/cp-source-control-broker-service    NodePort   10.233.26.117   <none>        8093:30093/TCP   51s
service/cp-source-control-manager-service   NodePort   10.233.39.233   <none>        8080:30092/TCP   53s
service/cp-source-control-ui-service        NodePort   10.233.62.110   <none>        8094:30094/TCP   49s

NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cp-source-control-api-deployment       1/1     1            1           55s
deployment.apps/cp-source-control-broker-deployment    1/1     1            1           51s
deployment.apps/cp-source-control-manager-deployment   1/1     1            1           53s
deployment.apps/cp-source-control-ui-deployment        1/1     1            1           49s

NAME                                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/cp-source-control-api-deployment-588fdfbfd7       1         1         1       55s
replicaset.apps/cp-source-control-broker-deployment-84f687c698    1         1         1       51s
replicaset.apps/cp-source-control-manager-deployment-69b9b87cfd   1         1         1       53s
replicaset.apps/cp-source-control-ui-deployment-867557b66d        1         1         1       49s
```    

<br>

### <div id='3.4'>3.4. (참고) 컨테이너 플랫폼 소스 컨트롤 리소스 삭제
배포된 컨테이너 플랫폼 소스 컨트롤 리소스의 삭제를 원하는 경우 아래 스크립트를 실행한다.<br>
> :loudspeaker: 컨테이너 플랫폼을 통해 설치된 클러스터의 StorageClass 타입이 `NFS`인 경우 reclaim 정책은 `Retain`이다.<br>
> `Retain`정책은 Persistent Volume을 삭제하여도 스토리지 NFS 서버에 데이터가 여전히 존재하므로<br> 수동으로 데이터 정리가 필요하다.
```bash
$ cd ~/workspace/container-platform/cp-source-control-deployment/script
$ chmod +x uninstall-cp-source-control.sh
$ ./uninstall-cp-source-control.sh
```
```bash
Are you sure you want to delete the container platform source control? <y/n> # y 입력
release "cp-source-control-api" uninstalled
release "cp-source-control-manager" uninstalled
release "cp-source-control-broker" uninstalled
release "cp-source-control-ui" uninstalled
namespace "cp-source-control" deleted
...
```

<br>


## <div id='4'>4. 컨테이너 플랫폼 소스 컨트롤 서비스 브로커
애플리케이션 플랫폼 K-PaaS 포털과 컨테이너 플랫폼 소스 컨트롤 서비스 연동을 위해서 브로커 등록이 필요하다.
K-PaaS 운영자 포털을 통해 서비스를 등록하고 공개하면 K-PaaS 사용자 포털에서 서비스를 신청하여 사용할 수 있다. <br>

### <div id='4.1'>4.1. 컨테이너 플랫폼 소스 컨트롤 사용자 인증 서비스 구성
애플리케이션 플랫폼 포털에서 컨테이너 플랫폼 소스 컨트롤을 서비스로 사용하기 위해서는 **사용자 인증 서비스** 구성이 사전에 진행되어야 한다.
사용자 인증 서비스 구성은 아래 가이드를 참조한다.
> [[컨테이너 플랫폼 포털 사용자 인증 서비스 구성]](../portal/cp-portal-service-guide.md#4)

<br>

### <div id='4.2'>4.2. 컨테이너 플랫폼 소스 컨트롤 서비스 브로커 등록
:bulb: 해당 작업은 애플리케이션 플랫폼이 설치된 **BOSH Inception**에서 진행한다.

##### 서비스 브로커 목록을 확인한다.
>`$ cf service-brokers`
```bash
$ cf service-brokers
Getting service brokers as admin...
No service brokers found
```

##### 컨테이너 플랫폼 소스 컨트롤 서비스 브로커를 등록한다.
>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL}`

서비스팩 이름 : 서비스팩 명칭<br>
서비스팩 사용자 ID/비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID/비밀번호<br>
서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL<br>


###### 컨테이너 플랫폼 소스 컨트롤 서비스 브로커 등록
>`$ cf create-service-broker cp-source-control-service-broker admin cloudfoundry http://{K8S_MASTER_NODE_IP}:30093`
```bash
$ cf create-service-broker cp-source-control-service-broker admin cloudfoundry http://xx.xxx.xxx.xx:30093
Creating service broker cp-source-control-service-broker as admin...
OK
```    


##### 등록된 컨테이너 플랫폼 소스 컨트롤 서비스 브로커를 확인한다.
>`$ cf service-brokers`
```bash
$ cf service-brokers 
Getting service brokers as admin... 
name                               url 
cp-source-control-service-broker   http://xx.xxx.xxx.xx:30093
```


##### 접근 가능한 서비스 목록을 확인한다.
>`$ cf service-access`
```bash
$ cf service-access 
Getting service access as admin... 
broker: cp-source-control-service-broker
   offering      plan     access   orgs
   scm-manager   Shared   none
```


##### 특정 조직에 해당 서비스 접근 허용을 할당한다.

###### 컨테이너 플랫폼 소스 컨트롤 서비스 접근 허용 할당
>`$ cf enable-service-access scm-manager`
```bash
$ cf enable-service-access scm-manager
Enabling access to all plans of service offering scm-manager for all orgs as admin...
OK
```

##### 접근 가능한 서비스 목록을 확인한다.
>`$ cf service-access`
```bash
$ cf service-access 
Getting service access as admin... 
broker: cp-source-control-service-broker
   offering      plan     access   orgs
   scm-manager   Shared   all
```

<br>

### <div id='4.3'>4.3. 컨테이너 플랫폼 소스 컨트롤 서비스 조회 설정
해당 설정은 애플리케이션 플랫폼 K-PaaS 포털에서 컨테이너 플랫폼 소스 컨트롤 서비스를 조회하고 신청할 수 있도록 하기 위한 설정이다.

##### K-PaaS 운영자 포털에 접속한다.
![image 007]

##### 메뉴 [운영관리]-[카탈로그] 의 '앱 서비스' 목록 중 'Container Platform Source Control' 을 선택한다.
![image 019]

##### 'Container Platform Source Control' 상세 정보를 아래와 같이 설정 후 저장한다.
>`'서비스' 항목 : 'scm-manager' 로 선택` <br>
>`'공개' 항목 : 'Y' 로 체크`

![image 020]

##### K-PaaS 사용자 포털에 접속하여 컨테이너 플랫폼 소스 컨트롤 서비스를 생성한다.
![image 021]

<br>

### <div id='4.4'/>4.4. 컨테이너 플랫폼 소스 컨트롤 사용 가이드
- 컨테이너 플랫폼 소스 컨트롤 사용방법은 아래 사용가이드를 참고한다.
    + [컨테이너 플랫폼 소스 컨트롤 사용 가이드](../../use-guide/source-control/cp-source-control-use-guide.md)

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 서비스 형 컨테이너 플랫폼 소스 컨트롤 배포 가이드

[image 001]:../images/portal/cp-001.png
[image 002]:../images/portal/cp-002.png
[image 003]:../images/portal/cp-003.png
[image 004]:../images/portal/cp-004.png
[image 005]:../images/portal/cp-005.png
[image 006]:../images/portal/cp-006.png
[image 007]:../images/portal/cp-007.png
[image 008]:../images/portal/cp-008.png
[image 009]:../images/portal/cp-009.png
[image 010]:../images/portal/cp-010.png
[image 011]:../images/portal/cp-011.png
[image 012]:../images/portal/cp-012.png
[image 013]:../images/portal/cp-013.png
[image 014]:../images/portal/cp-014.png
[image 015]:../images/portal/cp-015.png
[image 016]:../images/portal/cp-016.png
[image 017]:../images/portal/cp-017.png
[image 018]:../images/portal/cp-018.png
[image 019]:../images/portal/cp-019.png
[image 020]:../images/portal/cp-020.png
[image 021]:../images/portal/cp-021.png