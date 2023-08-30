### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > Pipeline 설치 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
    1.1. [목적](#1.1)  
    1.2. [범위](#1.2)  
    1.3. [시스템 구성도](#1.3)  
    1.4. [참고 자료](#1.4)  

2. [Prerequisite](#2)  
    2.1. [컨테이너 플랫폼 포털 설치](#2.1)  
    2.2. [클러스터 환경](#2.2)   
        
3. [컨테이너 플랫폼 파이프라인 배포](#3)    
    3.1. [컨테이너 플랫폼 파이프라인 Deployment 파일 다운로드](#3.1)  
    3.2. [컨테이너 플랫폼 파이프라인 변수 정의](#3.2)    
    3.3. [컨테이너 플랫폼 파이프라인 배포 스크립트 실행](#3.3)    
    3.4. [(참조) 컨테이너 플랫폼 파이프라인 리소스 삭제](#3.4)    

4. [컨테이너 플랫폼 파이프라인 서비스 브로커](#4)   
    4.1. [컨테이너 플랫폼 파이프라인 사용자 인증 서비스 구성](#4.1)   
    4.2. [컨테이너 플랫폼 파이프라인 서비스 브로커 등록](#4.2)  
    4.3. [컨테이너 플랫폼 파이프라인 서비스 조회 설정](#4.3)    
    4.4. [컨테이너 플랫폼 파이프라인 사용 가이드](#4.4)       



## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(Container Platform Pipeline 서비스 배포 설치 가이드)는 Kubernetes  Cluster 및 컨테이너 플랫폼 서비스 배포 형 포털을 설치하고 컨테이너 플랫폼 서비스 배포형 파이프라인 배포 방법을 기술하였다.<br>

<br>

### <div id='1.2'>1.2. 범위
설치 범위는 Kubernetes Cluster 배포를 기준으로 작성하였다.

<br>

### <div id='1.3'>1.3. 시스템 구성도
<p align="center"><img src="https://user-images.githubusercontent.com/33216551/209299431-af201419-9220-425a-8552-d15e379f8ee7.png" width="850" height="530">
<br>

시스템 구성은 Kubernetes Cluster(Master, Worker) 환경과 데이터 관리를 위한 스토리지 서버로 구성되어 있다. 
Kubespray를 통해 설치된 Kubernetes Cluster 환경에 컨테이너 플랫폼 파이프라인 이미지 및 Helm Chart를 관리하는 Harbor, 컨테이너 플랫폼 파이프라인 사용자 인증을 관리하는 Keycloak, 컨테이너 플랫폼 파이프라인 메타 데이터를 관리하는 MariaDB(RDBMS)가 컨테이너 플랫폼 포털을 통해서 제공된다.
컨테이너 플랫폼 파이프라인에서는 지속적 통합과 배포 기능을 관리하는 Jenkins 서버(Ci-Server) 와 정적 분석을 위한 Sonarqube(Inspection-Server), 배포되는 애플리케이션의 Config를 관리하는 Spring Config Server(Config-Server) 등 파이프라인 동작에 필요한 환경을 컨테이너로 제공한다. 
총 필요한 VM 환경으로는 Master VM: 1개, Worker VM: 3개 이상이 필요하고 본 문서는 Kubernetes Cluster에 컨테이너 플랫폼 파이프라인 환경을 배포하는 내용이다. 

<br>

### <div id='1.4'>1.4. 참고 자료
> https://kubernetes.io/ko/docs  

<br>

## <div id='2'>2. Prerequisite
### <div id='2.1'>2.1. 컨테이너 플랫폼 포털 설치
컨테이너 플랫폼 파이프라인에서 사용할 인프라로 인증서버 **KeyCloak Server**, 데이터베이스 **MariaDB**, 레포지토리 서버 **Harbor** 설치가 사전에 진행되어야 한다.
파스타 컨테이너 플랫폼 포털 배포 시 해당 인프라를 모두 설치한다.
컨테이너 플랫폼 포털 설치는 아래 가이드를 참조한다.
> [파스타 컨테이너 플랫폼 포털 배포](../container-platform-portal/paas-ta-container-platform-portal-deployment-service-guide.md)     

<br>
    
### <div id='2.2'>2.2. Cluster 환경
컨테이너 플랫폼 파이프라인 배포를 위해서는 sonarqube, postgresql 등의 image를 public network에서 다운받기 때문에 **외부 네트워크 통신**이 가능한 환경에서 설치해야 한다. <br>
컨테이너 플랫폼 파이프라인 설치 완료 시 idle 상태에서의 사용 resource는 다음과 같다.
```
NAME                                                     CPU(cores)   MEMORY(bytes)
cp-pipeline-api-deployment-bb6f7bd46-nl4j4               1m           224Mi
cp-pipeline-broker-deployment-b76b6ff4-l6ndw             2m           189Mi
cp-pipeline-common-api-deployment-54c646c95c-87k5g       2m           255Mi
cp-pipeline-config-server-deployment-78675b565d-8kxf9    2m           170Mi
cp-pipeline-inspection-api-deployment-6bf9c4479d-d6xgb   1m           158Mi
cp-pipeline-jenkins-deployment-779c6d7bc9-pn782          3m           1319Mi
cp-pipeline-postgresql-postgresql-0                      4m           58Mi
cp-pipeline-sonarqube-sonarqube-6d9c6b579f-2xkkw         7m           1744Mi
cp-pipeline-ui-deployment-5db955b77b-snkpl               1m           337Mi
```
컨테이너 플랫폼 파이프라인을 설치할 클러스터 환경에는 클러스터 총합 최소 **4Gi**의 여유 메모리를 권장한다.<br>

컨테이너 플랫폼 파이프라인 설치 완료 시 Persistent Volume 사용 resource는 다음과 같다.    
```
NAME                                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS
cp-pipeline-jenkins-pv                     Bound    pvc-4bf64900-d25c-482f-9aa3-baa07c11cdd1   20Gi       RWO            paasta-cp-storageclass
data-cp-pipeline-postgresql-postgresql-0   Bound    pvc-f61096ac-5e2b-4105-9ed3-04a9a7d999cb   8Gi        RWX            paasta-cp-storageclass
```
컨테이너 플랫폼 파이프라인을 설치할 클러스터 환경에는 NFS 스토리지 용량 **28Gi**의 여유 용량을 권장한다.<br>    
    
<br>    
    
## <div id='3'>3. 컨테이너 플랫폼 파이프라인 배포  
    
### <div id='3.1'>3.1. 컨테이너 플랫폼 파이프라인 Deployment 파일 다운로드
컨테이너 플랫폼 파이프라인 배포를 위해 컨테이너 플랫폼 파이프라인 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
:bulb: 해당 내용은 Kubernetes **Master Node**에서 진행한다.

+ 컨테이너 플랫폼 파이프라인 Deployment 파일 다운로드 :  
   [cp-pipeline-deployment-v1.4.0.tar.gz](https://nextcloud.paas-ta.org/index.php/s/wjiQ5dScS6pPQkp/download)  

```
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.paas-ta.org/index.php/s/wjiQ5dScS6pPQkp/download

$ ls ~/workspace/container-platform
  ...
  cp-pipeline-deployment-v1.4.0.tar.gz
  ...
  
# Deployment 파일 압축 해제
$ tar xvfz cp-pipeline-deployment-v1.4.0.tar.gz
```

- Deployment 파일 디렉토리 구성
```
├── script          # 컨테이너 플랫폼 파이프라인 배포 관련 변수 및 스크립트 파일 위치
├── images          # 컨테이너 플랫폼 파이프라인 이미지 파일 위치
├── charts          # 컨테이너 플랫폼 파이프라인 Helm Charts 파일 위치
├── values_orig     # 컨테이너 플랫폼 파이프라인 Helm Charts values.yaml 원본 파일 위치 
```

<br>

### <div id='3.2'>3.2. 컨테이너 플랫폼 파이프라인 변수 정의
컨테이너 플랫폼 파이프라인을 배포하기 전 변수 값 정의가 필요하다. 배포에 필요한 정보를 확인하여 변수를 설정한다.

```
$ cd ~/workspace/container-platform/cp-pipeline-deployment/script
$ vi cp-pipeline-vars.sh
```

```                                                     
# COMMON VARIABLE
K8S_MASTER_NODE_IP="{k8s master node public ip}"                # Kubernetes master node public ip
PROVIDER_TYPE="{container platform pipeline provider type}"     # Container platform pipeline provider type (Please enter 'standalone' or 'service')
CF_API_URL="https://{paas-ta-api-domain}"                       # e.g) https://api.10.0.0.120.nip.io, PaaS-TA API Domain, PROVIDER_TYPE=service 인 경우 입력
....    
```
```    
# Example
K8S_MASTER_NODE_IP="xx.xxx.xxx.xx"
PROVIDER_TYPE="service"
CF_API_URL="https://api.xx.xxx.xxx.xx.nip.io"
```

- **K8S_MASTER_NODE_IP** <br>Kubernetes Master Node Public IP 입력<br><br>
- **PROVIDER_TYPE** <br>컨테이너 플랫폼 파이프라인 제공 타입 입력 <br>
   + 본 가이드는 서비스 배포 설치 가이드로 **'service'** 값 입력 필요<br><br>
- **CF_API_URL** <br>서비스 연동할 PaaS-TA의 api domain 입력 <br>
<br>    

:bulb: Keycloak 기본 배포 방식은 **HTTP**이며 인증서를 통한 **HTTPS**를 설정되어 있는 경우
> [Keycloak TLS 설정](../container-platform-portal/paas-ta-container-platform-portal-deployment-keycloak-tls-setting-guide.md)

컨테이너 플랫폼 파이프라인 변수 파일 내 아래 내용을 수정한다.
```
$ vi cp-pipeline-vars.sh
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
    
### <div id='3.3'>3.3. 컨테이너 플랫폼 파이프라인 배포 스크립트 실행
컨테이너 플랫폼 파이프라인 배포를 위한 배포 스크립트를 실행한다.

```
$ chmod +x deploy-cp-pipeline.sh
$ ./deploy-cp-pipeline.sh
```
    
<br>

컨테이너 플랫폼 파이프라인 관련 리소스가 정상적으로 배포되었는지 확인한다.<br>
리소스 Pod의 경우 Node에 바인딩 및 컨테이너 생성 후 Running 상태로 전환되기까지 몇 초가 소요된다.

- **컨테이너 플랫폼 파이프라인 리소스 조회**

```
$ kubectl get all -n cp-pipeline
```

```
NAME                                                         READY   STATUS    RESTARTS      AGE
pod/cp-pipeline-api-deployment-bb6f7bd46-xmtxx               1/1     Running   0             71s
pod/cp-pipeline-broker-deployment-b76b6ff4-hs2m8             1/1     Running   1 (63s ago)   70s
pod/cp-pipeline-common-api-deployment-54c646c95c-pr8d4       1/1     Running   0             71s
pod/cp-pipeline-config-server-deployment-78675b565d-jsg6z    1/1     Running   0             68s
pod/cp-pipeline-inspection-api-deployment-6bf9c4479d-qvt2s   1/1     Running   0             69s
pod/cp-pipeline-jenkins-deployment-779c6d7bc9-fphrv          1/1     Running   0             69s
pod/cp-pipeline-postgresql-postgresql-0                      1/1     Running   0             65s
pod/cp-pipeline-sonarqube-sonarqube-6d9c6b579f-ndjmj         0/1     Running   0             65s
pod/cp-pipeline-ui-deployment-5db955b77b-6zgtg               1/1     Running   0             70s

NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/cp-pipeline-api-service              NodePort    10.233.33.178   <none>        8082:30082/TCP   71s
service/cp-pipeline-broker-service           NodePort    10.233.47.79    <none>        8083:30083/TCP   70s
service/cp-pipeline-common-api-service       NodePort    10.233.42.55    <none>        8081:30081/TCP   71s
service/cp-pipeline-config-server-service    NodePort    10.233.29.96    <none>        8080:30088/TCP   68s
service/cp-pipeline-inspection-api-service   NodePort    10.233.18.126   <none>        8085:30085/TCP   69s
service/cp-pipeline-jenkins-service          NodePort    10.233.43.30    <none>        8080:30086/TCP   69s
service/cp-pipeline-postgresql               ClusterIP   10.233.27.243   <none>        5432/TCP         67s
service/cp-pipeline-postgresql-headless      ClusterIP   None            <none>        5432/TCP         67s
service/cp-pipeline-sonarqube-sonarqube      NodePort    10.233.52.4     <none>        9000:30087/TCP   66s
service/cp-pipeline-ui-service               NodePort    10.233.57.132   <none>        8084:30084/TCP   70s

NAME                                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cp-pipeline-api-deployment              1/1     1            1           71s
deployment.apps/cp-pipeline-broker-deployment           1/1     1            1           70s
deployment.apps/cp-pipeline-common-api-deployment       1/1     1            1           71s
deployment.apps/cp-pipeline-config-server-deployment    1/1     1            1           68s
deployment.apps/cp-pipeline-inspection-api-deployment   1/1     1            1           69s
deployment.apps/cp-pipeline-jenkins-deployment          1/1     1            1           69s
deployment.apps/cp-pipeline-sonarqube-sonarqube         0/1     1            0           65s
deployment.apps/cp-pipeline-ui-deployment               1/1     1            1           70s

NAME                                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/cp-pipeline-api-deployment-bb6f7bd46               1         1         1       71s
replicaset.apps/cp-pipeline-broker-deployment-b76b6ff4             1         1         1       70s
replicaset.apps/cp-pipeline-common-api-deployment-54c646c95c       1         1         1       71s
replicaset.apps/cp-pipeline-config-server-deployment-78675b565d    1         1         1       68s
replicaset.apps/cp-pipeline-inspection-api-deployment-6bf9c4479d   1         1         1       69s
replicaset.apps/cp-pipeline-jenkins-deployment-779c6d7bc9          1         1         1       69s
replicaset.apps/cp-pipeline-sonarqube-sonarqube-6d9c6b579f         1         1         0       65s
replicaset.apps/cp-pipeline-ui-deployment-5db955b77b               1         1         1       70s

NAME                                                 READY   AGE
statefulset.apps/cp-pipeline-postgresql-postgresql   1/1     66s
```    

<br>    
    
### <div id='3.4'>3.4. (참조) 컨테이너 플랫폼 파이프라인 리소스 삭제
배포된 컨테이너 플랫폼 파이프라인 리소스의 삭제를 원하는 경우 아래 스크립트를 실행한다.<br>

```
$ cd ~/workspace/container-platform/cp-pipeline-deployment/script
$ chmod +x uninstall-cp-pipeline.sh
$ ./uninstall-cp-pipeline.sh
Are you sure you want to delete the container platform pipeline? <y/n> # y 입력
```
```
release "cp-pipeline-api" uninstalled
release "cp-pipeline-common-api" uninstalled
release "cp-pipeline-broker" uninstalled
release "cp-pipeline-ui" uninstalled
release "cp-pipeline-inspection-api" uninstalled
release "cp-pipeline-jenkins" uninstalled
release "cp-pipeline-config-server" uninstalled
release "cp-pipeline-postgresql" uninstalled
release "cp-pipeline-sonarqube" uninstalled
namespace "cp-pipeline" deleted
...
...
```

<br>
  
## <div id='4'>4. 컨테이너 플랫폼 파이프라인 서비스 브로커
컨테이너 플랫폼 PaaS-TA 서비스 형 파이프라인으로 설치하는 경우 CF와 Kubernetes에 배포된 컨테이너 플랫폼 파이프라인 서비스 연동을 위해서 브로커를 등록해 주어야 한다.
PaaS-TA 운영자 포털을 통해 서비스를 등록하고 공개하면, PaaS-TA 사용자 포털을 통해 서비스를 신청하여 사용할 수 있다.
  
### <div id='4.1'>4.1. 컨테이너 플랫폼 파이프라인 사용자 인증 서비스 구성
컨테이너 플랫폼 파이프라인을 서비스로 사용하기 위해서는 **사용자 인증 서비스** 구성이 사전에 진행되어야 한다.<br>
사용자 인증 서비스 구성은 아래 가이드를 참조한다.
> [사용자 인증 서비스 구성](../container-platform-portal/paas-ta-container-platform-portal-deployment-service-guide.md#4)      
컨테이너 플랫폼 포털 사용자 인증 서비스 구성 시, 파이프라인에도 적용된다.

<br>
    
### <div id='4.2'>4.2. 컨테이너 플랫폼 파이프라인 서비스 브로커 등록
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
    
    
##### 컨테이너 플랫폼 파이프라인 서비스 브로커를 등록한다.
>`$ cf create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{서비스팩 URL}`

서비스팩 이름 : 서비스 팩 관리를 위해 개방형 클라우드 플랫폼에서 보여지는 명칭<br>
서비스팩 사용자 ID/비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID/비밀번호<br>
서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL<br>


###### 컨테이너 플랫폼 파이프라인 서비스 브로커 등록 
>`$ cf create-service-broker cp-pipeline-service-broker admin cloudfoundry http://{K8S_MASTER_NODE_IP}:30083`   


```
$ cf create-service-broker cp-pipeline-service-broker admin cloudfoundry http://xx.xxx.xxx.xx:30083
Creating service broker cp-pipeline-service-broker as admin...
OK
```    

    
##### 등록된 컨테이너 플랫폼 파이프라인 서비스 브로커를 확인한다.
>`$ cf service-brokers` 
```
$ cf service-brokers 
Getting service brokers as admin... 
name                         url 
cp-pipeline-service-broker   http://xx.xxx.xxx.xx:30083
```

    
##### 접근 가능한 서비스 목록을 확인한다.
>`$ cf service-access`     
```
$ cf service-access 
Getting service access as admin... 
broker: cp-pipeline-service-broker 
   offering                      plan                                 access   orgs 
   container-platform-pipeline   container-platform-pipeline-shared   none
```

        
##### 특정 조직에 해당 서비스 접근 허용을 할당한다.

###### 컨테이너 플랫폼 파이프라인 서비스 접근 허용 할당  
>`$ cf enable-service-access container-platform-pipeline`   

```
$ cf enable-service-access container-platform-pipeline 
Enabling access to all plans of service offering container-platform-pipeline for all orgs as admin... 
OK
```
        
##### 접근 가능한 서비스 목록을 확인한다.
>`$ cf service-access` 

```
$ cf service-access 
Getting service access as admin... 
broker: cp-pipeline-service-broker 
   offering                      plan                                 access   orgs 
   container-platform-pipeline   container-platform-pipeline-shared   all
```

<br>
    
### <div id='4.3'>4.3. 컨테이너 플랫폼 파이프라인 서비스 조회 설정
해당 설정은 PaaS-TA 포털에서 컨테이너 플랫폼 파이프라인 서비스를 조회하고 신청할 수 있도록 하기 위한 설정이다.

##### PaaS-TA 운영자 포털에 접속한다.


##### 메뉴 [운영관리]-[카탈로그] 에서 앱서비스 탭 안에 Container Platform Pipeline 서비스를 선택하여 설정을 변경한다.
![image](https://user-images.githubusercontent.com/80228983/146296230-2e3a90fa-44ac-4e13-9472-dfb3a1655a98.png)

##### Container Platform Pipeline 서비스를 선택하여 아래와 같이 설정 변경 후 저장한다.
>`'서비스' 항목 : 'container-platform-pipeline' 으로 선택` <br>
>`'공개' 항목 : 'Y' 로 체크`    

![image](https://user-images.githubusercontent.com/80228983/146296316-3bbb70d4-ce31-42f6-9ec0-019c0f12d774.png)

##### PaaS-TA 사용자 포털에 접속한다.

##### 메뉴 [카탈로그]-[서비스] 에서 서비스 탭 안에 Container Platform Pipeline 서비스를 선택하여 서비스를 생성한다.
![image](https://user-images.githubusercontent.com/80228983/146296949-fceac26c-86b6-40fb-b005-dcc84b3f081c.png)

<br>

    
### <div id='4.4'/>4.4. 컨테이너 플랫폼 파이프라인 사용 가이드
- 컨테이너 플랫폼 파이프라인 사용방법은 아래 사용가이드를 참고한다.  
  + [컨테이너 플랫폼 파이프라인 사용 가이드](../../use-guide/pipeline/paas-ta-container-platform-pipeline-use-guide.md)   

<br>

### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > Pipeline 설치 가이드