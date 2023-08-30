### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > SourceControl 설치 가이드

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
    3.4. [(참조) 컨테이너 플랫폼 소스 컨트롤 리소스 삭제](#3.4)    

4. [컨테이너 플랫폼 소스 컨트롤 서비스 브로커](#4)   
    4.1. [컨테이너 플랫폼 소스 컨트롤 사용자 인증 서비스 구성](#4.1)   
    4.2. [컨테이너 플랫폼 소스 컨트롤 서비스 브로커 등록](#4.2)  
    4.3. [컨테이너 플랫폼 소스 컨트롤 서비스 조회 설정](#4.3)    
    4.4. [컨테이너 플랫폼 소스 컨트롤 사용 가이드](#4.4)       



## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(Container Platform Source Control 서비스 배포 설치 가이드)는 Kubernetes  Cluster 및 컨테이너 플랫폼 서비스 배포형 포털을 설치하고 컨테이너 플랫폼 서비스 배포형 소스 컨트롤 배포 방법을 기술하였다.<br>

<br>

### <div id='1.2'>1.2. 범위
설치 범위는 Kubernetes Cluster 배포를 기준으로 작성하였다.

<br>

### <div id='1.3'>1.3. 시스템 구성도
<p align="center"><img src="https://user-images.githubusercontent.com/33216551/209299431-af201419-9220-425a-8552-d15e379f8ee7.png" width="850" height="530">
<br>

시스템 구성은 Kubernetes Cluster(Master, Worker) 환경과 데이터 관리를 위한 스토리지 서버로 구성되어 있다. 
Kubespray를 통해 설치된 Kubernetes Cluster 환경에 컨테이너 플랫폼 소스 컨트롤 이미지 및 Helm Chart를 관리하는 Harbor, 컨테이너 플랫폼 소스 컨트롤 사용자 인증을 관리하는 Keycloak, 컨테이너 플랫폼 소스 컨트롤 메타 데이터를 관리하는 MariaDB(RDBMS)가 컨테이너 플랫폼 포털을 통해서 제공된다.
컨테이너 플랫폼 소스 컨트롤에서는 소스를 관리하는 SCM-Server를 컨테이너로 제공한다. 
총 필요한 VM 환경으로는 Master VM: 1개, Worker VM: 3개 이상이 필요하고 본 문서는 Kubernetes Cluster에 컨테이너 플랫폼 소스 컨트롤 환경을 배포하는 내용이다.

<br>

### <div id='1.4'>1.4. 참고 자료
> https://kubernetes.io/ko/docs  

<br>

## <div id='2'>2. Prerequisite
    
### <div id='2.1'>2.1. 컨테이너 플랫폼 포털 설치
컨테이너 플랫폼 소스 컨트롤에서 사용할 인프라로 인증서버 **KeyCloak Server**, 데이터베이스 **MariaDB**, 레포지토리 서버 **Harbor** 설치가 사전에 진행되어야 한다.
파스타 컨테이너 플랫폼 포털 배포 시 해당 인프라를 모두 설치한다.
컨테이너 플랫폼 포털 설치는 아래 가이드를 참조한다.
> [파스타 컨테이너 플랫폼 포털 배포](../container-platform-portal/paas-ta-container-platform-portal-deployment-service-guide.md)     

<br>
  
## <div id='3'>3. 컨테이너 플랫폼 소스 컨트롤 배포
    
### <div id='3.1'>3.1. 컨테이너 플랫폼 소스 컨트롤 Deployment 파일 다운로드
컨테이너 플랫폼 소스 컨트롤 배포를 위해 컨테이너 플랫폼 소스 컨트롤 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
:bulb: 해당 내용은 Kubernetes **Master Node**에서 진행한다.

+ 컨테이너 플랫폼 소스 컨트롤 Deployment 파일 다운로드 :  
   [cp-source-control-deployment-v1.4.0.tar.gz](https://nextcloud.paas-ta.org/index.php/s/bBKm3JcQFHRw6mB/download)  

```
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.paas-ta.org/index.php/s/bBKm3JcQFHRw6mB/download

$ ls ~/workspace/container-platform
  ...
  cp-source-control-deployment-v1.4.0.tar.gz
  ...

# Deployment 파일 압축 해제
$ tar xvfz cp-source-control-deployment-v1.4.0.tar.gz
```

- Deployment 파일 디렉토리 구성
```
├── script          # 컨테이너 플랫폼 소스 컨트롤 배포 관련 변수 및 스크립트 파일 위치
├── images          # 컨테이너 플랫폼 소스 컨트롤 이미지 파일 위치
├── charts          # 컨테이너 플랫폼 소스 컨트롤 Helm Charts 파일 위치
├── values_orig     # 컨테이너 플랫폼 소스 컨트롤 Helm Charts values.yaml 원본 파일 위치 
```

<br>

### <div id='3.2'>3.2. 컨테이너 플랫폼 소스 컨트롤 변수 정의
컨테이너 플랫폼 소스 컨트롤을 배포하기 전 변수 값 정의가 필요하다. 배포에 필요한 정보를 확인하여 변수를 설정한다.

```
$ cd ~/workspace/container-platform/cp-source-control-deployment/script
$ vi cp-source-control-vars.sh
```

```                                                     
# COMMON VARIABLE
K8S_MASTER_NODE_IP="{k8s master node public ip}"                       # Kubernetes master node public ip
PROVIDER_TYPE="{container platform source control provider type}"      # Container platform source-control provider type (Please enter 'standalone' or 'service')
....    
```
```    
# Example
K8S_MASTER_NODE_IP="xx.xxx.xxx.xx"                                       
PROVIDER_TYPE="service"
```

- **K8S_MASTER_NODE_IP** <br>Kubernetes Master Node Public IP 입력<br><br>
- **PROVIDER_TYPE** <br>컨테이너 플랫폼 소스 컨트롤 제공 타입 입력 <br>
   + 본 가이드는 서비스 배포 설치 가이드로 **'service'** 값 입력 필요
<br>    

:bulb: Keycloak 기본 배포 방식은 **HTTP**이며 인증서를 통한 **HTTPS**를 설정되어 있는 경우
> [Keycloak TLS 설정](../container-platform-portal/paas-ta-container-platform-portal-deployment-keycloak-tls-setting-guide.md)

컨테이너 플랫폼 소스 컨트롤 변수 파일 내 아래 내용을 수정한다.
```
$ vi cp-source-control-vars.sh
```    
```
# KEYCLOAK_URL 값 http -> https 로 변경 
# Domain으로 nip.io를 사용하는 경우 아래와 같이 변경
    
....  
# KEYCLOAK    
KEYCLOAK_URL="https://${K8S_MASTER_NODE_IP}.nip.io:32710"   #if apply TLS, https://
....     
```

<br>

### <div id='3.3'>3.3. 컨테이너 플랫폼 소스 컨트롤 배포 스크립트 실행
컨테이너 플랫폼 소스 컨트롤 배포를 위한 배포 스크립트를 실행한다.

```
$ chmod +x deploy-cp-source-control.sh
$ ./deploy-cp-source-control.sh
```

<br>

컨테이너 플랫폼 소스 컨트롤 관련 리소스가 정상적으로 배포되었는지 확인한다.<br>
리소스 Pod의 경우 Node에 바인딩 및 컨테이너 생성 후 Running 상태로 전환되기까지 몇 초가 소요된다.

- **컨테이너 플랫폼 소스 컨트롤 리소스 조회**

```
$ kubectl get all -n cp-source-control
```
    
```
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

### <div id='3.4'>3.4. (참조) 컨테이너 플랫폼 소스 컨트롤 리소스 삭제
배포된 컨테이너 플랫폼 소스 컨트롤 리소스의 삭제를 원하는 경우 아래 스크립트를 실행한다.<br>

```
$ cd ~/workspace/container-platform/cp-source-control-deployment/script
$ chmod +x uninstall-cp-source-control.sh
$ ./uninstall-cp-source-control.sh
```
```
Are you sure you want to delete the container platform source control? <y/n> # y 입력
release "cp-source-control-api" uninstalled
release "cp-source-control-manager" uninstalled
release "cp-source-control-broker" uninstalled
release "cp-source-control-ui" uninstalled
namespace "cp-source-control" deleted
...
...
```

<br>
  
## <div id='4'>4. 컨테이너 플랫폼 소스 컨트롤 서비스 브로커
컨테이너 플랫폼 PaaS-TA 서비스 형 소스 컨트롤으로 설치하는 경우 CF와 Kubernetes에 배포된 컨테이너 플랫폼 소스 컨트롤 서비스 연동을 위해서 브로커를 등록해 주어야 한다.
PaaS-TA 운영자 포털을 통해 서비스를 등록하고 공개하면, PaaS-TA 사용자 포털을 통해 서비스를 신청하여 사용할 수 있다.
  
### <div id='4.1'>4.1. 컨테이너 플랫폼 소스 컨트롤 사용자 인증 서비스 구성
컨테이너 플랫폼 소스 컨트롤을 서비스로 사용하기 위해서는 **사용자 인증 서비스** 구성이 사전에 진행되어야 한다.<br>
사용자 인증 서비스 구성은 아래 가이드를 참조한다.
> [사용자 인증 서비스 구성](../container-platform-portal/paas-ta-container-platform-portal-deployment-service-guide.md#4)      
컨테이너 플랫폼 포털 사용자 인증 서비스 구성 시, 소스 컨트롤에도 적용된다.

<br>

### <div id='4.2'>4.2. 컨테이너 플랫폼 소스 컨트롤 서비스 브로커 등록
:bulb: 해당 내용은 PaaS-TA 포털이 설치된 **BOSH Inception**에서 진행한다.
서비스 브로커 등록 시 개방형 클라우드 플랫폼에서 서비스 브로커를 등록할 수 있는 사용자로 로그인이 되어있어야 한다.

##### 서비스 브로커 목록을 확인한다.
>`$ cf service-brokers` 
```
$ cf service-brokers
Getting service brokers as admin...

name   url
No service brokers found
```
    
    
##### 컨테이너 플랫폼 소스 컨트롤 서비스 브로커를 등록한다.
>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL}`

서비스팩 이름 : 서비스 팩 관리를 위해 개방형 클라우드 플랫폼에서 보여지는 명칭<br>
서비스팩 사용자 ID/비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID/비밀번호<br>
서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL<br>


###### 컨테이너 플랫폼 소스 컨트롤 서비스 브로커 등록 
>`$ cf create-service-broker cp-source-control-service-broker admin cloudfoundry http://{K8S_MASTER_NODE_IP}:30093`   


```
$ cf create-service-broker cp-source-control-service-broker admin cloudfoundry http://xx.xxx.xxx.xx:30093
Creating service broker cp-source-control-service-broker as admin...
OK
```    

    
##### 등록된 컨테이너 플랫폼 소스 컨트롤 서비스 브로커를 확인한다.
>`$ cf service-brokers` 
```
$ cf service-brokers 
Getting service brokers as admin... 
name                               url 
cp-source-control-service-broker   http://xx.xxx.xxx.xx:30093
```

    
##### 접근 가능한 서비스 목록을 확인한다.
>`$ cf service-access`     
```
$ cf service-access 
Getting service access as admin... 
broker: cp-source-control-service-broker
   offering      plan     access   orgs
   scm-manager   Shared   none
```

        
##### 특정 조직에 해당 서비스 접근 허용을 할당한다.

###### 컨테이너 플랫폼 소스 컨트롤 서비스 접근 허용 할당  
>`$ cf enable-service-access scm-manager`  

```
$ cf enable-service-access scm-manager
Enabling access to all plans of service offering scm-manager for all orgs as admin...
OK
```
        
##### 접근 가능한 서비스 목록을 확인한다.
>`$ cf service-access` 

```
$ cf service-access 
Getting service access as admin... 
broker: cp-source-control-service-broker
   offering      plan     access   orgs
   scm-manager   Shared   all
```

<br>
    
### <div id='4.3'>4.3. 컨테이너 플랫폼 소스 컨트롤 서비스 조회 설정
해당 설정은 PaaS-TA 포털에서 컨테이너 플랫폼 소스 컨트롤 서비스를 조회하고 신청할 수 있도록 하기 위한 설정이다.

##### PaaS-TA 운영자 포털에 접속한다.


##### 메뉴 [운영관리]-[카탈로그] 에서 앱서비스 탭 안에 Container Platform Source Control 서비스를 선택하여 설정을 변경한다.
![image](https://user-images.githubusercontent.com/80228983/146296230-2e3a90fa-44ac-4e13-9472-dfb3a1655a98.png)

##### Container Platform Source-control 서비스를 선택하여 아래와 같이 설정 변경 후 저장한다.
>`'서비스' 항목 : 'scm-manager' 으로 선택` <br>
>`'공개' 항목 : 'Y' 로 체크`    

![image](https://user-images.githubusercontent.com/80228983/146360677-bd0878f4-85ac-48fc-9e30-6bc49a74381f.png)


##### PaaS-TA 사용자 포털에 접속한다.

##### 메뉴 [카탈로그]-[서비스] 에서 서비스 탭 안에 Container Platform Source Control 서비스를 선택하여 서비스를 생성한다.
![image](https://user-images.githubusercontent.com/80228983/146360859-7388527a-e570-4985-b4bc-e5b4b3f19c55.png)

<br>

    
### <div id='4.4'/>4.4. 컨테이너 플랫폼 소스 컨트롤 사용 가이드
- 컨테이너 플랫폼 소스 컨트롤 사용방법은 아래 사용가이드를 참고한다.  
  + [컨테이너 플랫폼 소스 컨트롤 사용 가이드](../../use-guide/source-control/paas-ta-container-platform-source-control-use-guide.md)   

<br>

### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > SourceControl 설치 가이드