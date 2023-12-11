### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 단독 형 컨테이너 플랫폼 소스 컨트롤 배포 가이드

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

4. [컨테이너 플랫폼 소스 컨트롤 접속](#4)      
   4.1. [컨테이너 플랫폼 소스 컨트롤 관리자 로그인](#4.1)      
   4.2. [컨테이너 플랫폼 소스 컨트롤 사용자 회원가입/로그인](#4.2)  
   4.3. [컨테이너 플랫폼 소스 컨트롤 사용 가이드](#4.3)


<br>

## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(단독 형 컨테이너 플랫폼 소스 컨트롤 배포 가이드)는 쿠버네티스 클러스터 및 단독 형 컨테이너 플랫폼 포털을 설치하고 단독 형 소스 컨트롤 배포 방법을 기술하였다.

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
> [[단독 형 컨테이너 플랫폼 포털 배포]](../portal/cp-portal-standalone-guide.md)

<br>

## <div id='3'>3. 컨테이너 플랫폼 소스 컨트롤 배포
### <div id='3.1'>3.1. 컨테이너 플랫폼 소스 컨트롤 Deployment 파일 다운로드
컨테이너 플랫폼 소스 컨트롤 배포를 위해 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
:bulb: 해당 내용은 Kubernetes **Master Node**에서 진행한다.
+ 컨테이너 플랫폼 소스 컨트롤 Deployment 파일 다운로드 :  
  [cp-source-control-deployment-v1.5.0.tar.gz](https://nextcloud.k-paas.org/index.php/s/rdHLcsrG9KnZjcd/download)

```bash
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.k-paas.org/index.php/s/rdHLcsrG9KnZjcd/download

$ ls ~/workspace/container-platform
  cp-source-control-deployment-v1.5.0.tar.gz ...
  
# Deployment 파일 압축 해제
$ tar -xvf cp-source-control-deployment-v1.5.0.tar.gz
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
PROVIDER_TYPE="standalone"
```
|변수|설명|상세 내용|
|---|---|---|
|**K8S_MASTER_NODE_IP**|Kubernetes Master Node Public IP 입력|Master Node에 접근하기 어려운 경우<br>[[3.1.2. 컨테이너 플랫폼 포털 변수 정의]](../portal/cp-portal-standalone-guide.md#3.1.2)에서<br>정의한 `K8S_MASTER_NODE_IP` 값 입력| 
|**K8S_STORAGECLASS**|StorageClass 명 입력|컨테이너 플랫폼을 통해 배포된 클러스터는 <br> 기본으로 <b>`cp-storageclass`</b>이다. <br> 다른 StorageClass 사용 시 해당 리소스 명을 입력한다.|
|**HOST_DOMAIN**|Host Domain 값 입력|[[3.1.2. 컨테이너 플랫폼 포털 변수 정의]](../portal/cp-portal-standalone-guide.md#3.1.2)에서<br>정의한 `HOST_DOMAIN` 값 입력|
|**PROVIDER_TYPE**|컨테이너 플랫폼 소스 컨트롤 제공 타입 입력|본 가이드는 단독 배포 형 설치 가이드로<br> **standalone** 값 입력 필요|

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
pod/cp-source-control-api-deployment-588fdfbfd7-727w2       1/1     Running   0          54s
pod/cp-source-control-manager-deployment-69b9b87cfd-kcv8l   1/1     Running   0          53s
pod/cp-source-control-ui-deployment-867557b66d-pq5w9        1/1     Running   0          51s

NAME                                        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/cp-source-control-api-service       NodePort   10.233.21.183   <none>        8091:30091/TCP   54s
service/cp-source-control-manager-service   NodePort   10.233.22.163   <none>        8080:30092/TCP   53s
service/cp-source-control-ui-service        NodePort   10.233.14.98    <none>        8094:30094/TCP   51s

NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cp-source-control-api-deployment       1/1     1            1           54s
deployment.apps/cp-source-control-manager-deployment   1/1     1            1           53s
deployment.apps/cp-source-control-ui-deployment        1/1     1            1           51s

NAME                                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/cp-source-control-api-deployment-588fdfbfd7       1         1         1       54s
replicaset.apps/cp-source-control-manager-deployment-69b9b87cfd   1         1         1       53s
replicaset.apps/cp-source-control-ui-deployment-867557b66d        1         1         1       51s
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
release "cp-source-control-ui" uninstalled
namespace "cp-source-control" deleted
...
```

<br>

## <div id='4'>4. 컨테이너 플랫폼 소스 컨트롤 접속
컨테이너 플랫폼 소스 컨트롤에 접속한다.<br><br>
**컨테이너 플랫폼 소스 컨트롤 URL** : `http://{K8S_MASTER_NODE_IP}:30094`
+ [[3.2. 컨테이너 플랫폼 소스 컨트롤 변수 정의]](#3.2) 에서 정의한 `K8S_MASTER_NODE_IP` 값 입력

<br>

### <div id='4.1'/>4.1. 컨테이너 플랫폼 소스 컨트롤 관리자 로그인
컨테이너 플랫폼 포털 관리자 계정으로 소스 컨트롤에 로그인한다.
> [[(참고) 컨테이너 플랫폼 포털 관리자 계정 로그인]](../portal/cp-portal-standalone-guide.md#4.1)
+ **Username** : `admin`
+ **Password** : `초기화한 패스워드 값`

![image 002]

<br>    


### <div id='4.2'/>4.2. 컨테이너 플랫폼 소스 컨트롤 사용자 계정 로그인
#### 사용자 회원가입
- 하단의 'Register' 버튼을 클릭한다.

![image 003]

- 등록할 사용자 계정정보를 입력 후 'Register' 버튼을 클릭하여 컨테이너 플랫폼 소스 컨트롤에 회원가입한다.

![image 004]

#### 사용자 로그인
- 회원가입을 통해 등록된 계정으로 컨테이너 플랫폼 소스 컨트롤에 로그인한다.

![image 006]

<br>

### <div id='4.3'/>4.3. 컨테이너 플랫폼 소스 컨트롤 사용 가이드
- 컨테이너 플랫폼 소스컨트롤 사용 방법은 아래 사용가이드를 참고한다.
    + [컨테이너 플랫폼 소스 컨트롤 사용 가이드](../../use-guide/source-control/cp-source-control-use-guide.md)

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 단독 형 컨테이너 플랫폼 소스 컨트롤 배포 가이드

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