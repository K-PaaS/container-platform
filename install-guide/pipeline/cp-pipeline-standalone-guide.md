### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 컨테이너 플랫폼 파이프라인 배포 가이드

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
   3.4. [(참고) 컨테이너 플랫폼 파이프라인 리소스 삭제](#3.4)

4. [컨테이너 플랫폼 파이프라인 접속](#4)      
   4.1. [컨테이너 플랫폼 파이프라인 관리자 로그인](#4.1)      
   4.2. [컨테이너 플랫폼 파이프라인 사용자 회원가입/로그인](#4.2)  
   4.3. [컨테이너 플랫폼 파이프라인 사용 가이드](#4.3)


<br>

## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(컨테이너 플랫폼 파이프라인 배포 가이드)는 쿠버네티스 클러스터 및 컨테이너 플랫폼 포털을 설치하고 파이프라인 배포 방법을 기술하였다.

<br>

### <div id='1.2'>1.2. 범위
설치 범위는 쿠버네티스 클러스터 배포를 기준으로 작성하였다.

<br>

### <div id='1.3'>1.3. 시스템 구성도
<p align="center"><img src="../images/portal/cp-001.png" width="850" height="530"></p>
<br>

시스템 구성은 **Kubernetes Cluster(Master, Worker)** 환경과 데이터 관리를 위한 스토리지 서버로 구성되어 있다.
Kubespray를 통해 설치된 Kubernetes Cluster 환경에 컨테이너 플랫폼 파이프라인 이미지 및 Helm Chart를 관리하는 **Harbor**, 컨테이너 플랫폼 파이프라인 사용자 인증을 관리하는 **Keycloak**, 컨테이너 플랫폼 파이프라인 메타 데이터를 관리하는 **MariaDB(RDBMS)** 가 컨테이너 플랫폼 포털을 통해서 제공된다. 컨테이너 플랫폼 파이프라인에서는 지속적 통합과 배포 기능을 관리하는 **Jenkins 서버(Ci-Server)** 와 정적 분석을 위한 **Sonarqube(Inspection-Server)**, 배포되는 애플리케이션의 Config를 관리하는 **Spring Config Server(Config-Server)** 등 파이프라인 동작에 필요한 환경을 컨테이너로 제공한다. 총 필요한 VM 환경으로는 **Master VM: 1개, Worker VM: 3개 이상**이 필요하고 본 문서는 Kubernetes Cluster에 컨테이너 플랫폼 파이프라인 환경을 배포하는 내용이다.

<br>

### <div id='1.4'>1.4. 참고 자료
> https://kubernetes.io/ko/docs

<br>

## <div id='2'>2. Prerequisite

### <div id='2.1'>2.1. 컨테이너 플랫폼 포털 설치
컨테이너 플랫폼 파이프라인에서 사용할 인프라로 인증서버 **KeyCloak**, 데이터베이스 **MariaDB**, 레포지토리 서버 **Harbor** 설치가 사전에 진행되어야 한다.
K-PaaS 컨테이너 플랫폼 포털 배포 시 해당 인프라를 모두 설치하므로 아래 가이드를 참조하여 사전 설치를 진행한다.

##### 싱글 클러스터 환경
> [[싱글 클러스터 컨테이너 플랫폼 포털 배포 가이드]](../portal/cp-portal-standalone-guide.md)
##### 멀티 클러스터 환경
> [[멀티 클러스터 컨테이너 플랫폼 포털 배포 가이드]](../portal/cp-portal-standalone-guide-mc.md)


<br>

### <div id='2.2'>2.2. Cluster 환경
컨테이너 플랫폼 파이프라인 배포를 위해서는 sonarqube, postgresql 등의 image를 public network에서 다운받기 때문에 **외부 네트워크 통신**이 가능한 환경에서 설치해야 한다.
컨테이너 플랫폼 파이프라인 설치 완료 시 idle 상태에서의 사용 resource는 다음과 같다.
```bash
NAME                                                     CPU(cores)   MEMORY(bytes)
cp-pipeline-api-deployment-6d99f4c7d7-88h64              1m           224Mi
cp-pipeline-common-api-deployment-7488bc6d5d-q5wvt       2m           255Mi
cp-pipeline-config-server-deployment-5f676b858c-vxd6d    2m           170Mi
cp-pipeline-inspection-api-deployment-f5bfbdccc-pk5bk    1m           158Mi
cp-pipeline-jenkins-deployment-5f75b74b6d-vc25p          3m           1319Mi
cp-pipeline-postgresql-postgresql-0                      4m           58Mi
cp-pipeline-sonarqube-sonarqube-5f6c94bdbc-rqrf8         7m           1744Mi
cp-pipeline-ui-deployment-845bb6999f-kr78p               1m           337Mi
```
컨테이너 플랫폼 파이프라인을 설치할 클러스터 환경에는 클러스터 총합 최소 **4Gi**의 여유 메모리를 권장한다.

컨테이너 플랫폼 파이프라인 설치 완료 시 Persistent Volume 사용 resource는 다음과 같다.
```bash
NAME                                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS
cp-pipeline-jenkins-pvc                    Bound    pvc-4bf64900-d25c-482f-9aa3-baa07c11cdd1   20Gi       RWO            cp-storageclass
data-cp-pipeline-postgresql-postgresql-0   Bound    pvc-f61096ac-5e2b-4105-9ed3-04a9a7d999cb   8Gi        RWX            cp-storageclass
```
컨테이너 플랫폼 파이프라인을 설치할 클러스터 환경에는 NFS 스토리지 용량 **28Gi**의 여유 용량을 권장한다.<br>

<br>    

## <div id='3'>3. 컨테이너 플랫폼 파이프라인 배포

### <div id='3.1'>3.1. 컨테이너 플랫폼 파이프라인 Deployment 파일 다운로드
컨테이너 플랫폼 파이프라인 배포를 위해 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
##### :bulb: 해당 내용은 **Master Node**에서 진행한다. <br>
> 멀티 클러스터 환경의 경우 컨테이너 플랫폼 포털 배포 시 **Cluster1**로 지정했던 클러스터에서 진행한다.

+ 컨테이너 플랫폼 파이프라인 Deployment 파일 다운로드 :  
  [cp-pipeline-deployment-v1.6.0.tar.gz](https://nextcloud.k-paas.org/index.php/s/QZcw7ALyL7ReQmn/download)

```bash
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.k-paas.org/index.php/s/QZcw7ALyL7ReQmn/download

$ ls ~/workspace/container-platform
  cp-pipeline-deployment-v1.6.0.tar.gz ...
  
# Deployment 파일 압축 해제
$ tar -xvf cp-pipeline-deployment-v1.6.0.tar.gz
```

- Deployment 파일 디렉토리 구성
```bash
cp-pipeline-deployment
 ├── script        # 파이프라인 배포를 위한 변수 및 스크립트 파일 위치
 ├── images        # 이미지 파일 위치
 ├── charts        # Helm 차트 파일 위치
 └── values_orig   # Helm 차트 values 파일 위치
```

<br>

### <div id='3.2'>3.2. 컨테이너 플랫폼 파이프라인 변수 정의
컨테이너 플랫폼 파이프라인을 배포하기 전 변수 값 정의가 필요하다. 배포에 필요한 정보를 확인하여 변수를 설정한다.

```bash
$ cd ~/workspace/container-platform/cp-pipeline-deployment/script
$ vi cp-pipeline-vars.sh
```
```bash                                                  
# COMMON VARIABLE (Please change the value of the variables below.)
HOST_DOMAIN="{host domain}"                           # Host Domain (e.g. xx.xxx.xxx.xx.nip.io)
K8S_STORAGECLASS="cp-storageclass"                    # Kubernetes StorageClass Name (e.g. cp-storageclass)
IS_MULTI_CLUSTER="N"                                  # Please enter "Y" if deploy in a multi-cluster environment
```
```bash
# (예시)
HOST_DOMAIN="105.xxx.xxx.xxx.nip.io"
K8S_STORAGECLASS="cp-storageclass"
IS_MULTI_CLUSTER="N"
```

|변수|설명|상세 내용|
|---|---|---|
|**HOST_DOMAIN**|Host Domain 값 입력|[[3.2. 컨테이너 플랫폼 포털 변수 정의]](../portal/cp-portal-standalone-guide.md#3.2)에서<br>정의한 `HOST_DOMAIN` 값 입력|
|**K8S_STORAGECLASS**|StorageClass 명 입력|컨테이너 플랫폼을 통해 배포된 클러스터는 <br> 기본으로 `cp-storageclass`이다. <br> 다른 StorageClass 사용 시 해당 리소스 명을 입력한다.|
|**IS_MULTI_CLUSTER**|멀티 클러스터 환경 여부|멀티 클러스터 환경에 배포할 경우 "Y" 입력|

<br>

### <div id='3.3'>3.3. 컨테이너 플랫폼 파이프라인 배포 스크립트 실행
컨테이너 플랫폼 파이프라인 배포를 위한 배포 스크립트를 실행한다.

```bash
$ chmod +x deploy-cp-pipeline.sh
$ ./deploy-cp-pipeline.sh
```

<br>

컨테이너 플랫폼 파이프라인 관련 리소스가 정상적으로 배포되었는지 확인한다.<br>
리소스 Pod의 경우 Node에 바인딩 및 컨테이너 생성 후 Running 상태로 전환되기까지 몇 초가 소요된다.

#### 컨테이너 플랫폼 파이프라인 리소스 조회

```bash
$ kubectl get all -n cp-pipeline
```

```bash
NAME                                                        READY   STATUS    RESTARTS      AGE
pod/cp-pipeline-api-deployment-6d99f4c7d7-88h64             1/1     Running   0             100s
pod/cp-pipeline-common-api-deployment-7488bc6d5d-q5wvt      1/1     Running   0             100s
pod/cp-pipeline-config-server-deployment-5f676b858c-vxd6d   1/1     Running   0             100s
pod/cp-pipeline-inspection-api-deployment-f5bfbdccc-pk5bk   1/1     Running   0             100s
pod/cp-pipeline-jenkins-deployment-5f75b74b6d-vc25p         1/1     Running   0             103s
pod/cp-pipeline-postgresql-postgresql-0                     1/1     Running   0             103s
pod/cp-pipeline-sonarqube-sonarqube-5f6c94bdbc-rqrf8        1/1     Running   0             101s
pod/cp-pipeline-ui-deployment-845bb6999f-kr78p              1/1     Running   0             100s

NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/cp-pipeline-api-service              ClusterIP   10.233.27.151   <none>        8082/TCP   100s
service/cp-pipeline-common-api-service       ClusterIP   10.233.11.81    <none>        8081/TCP   100s
service/cp-pipeline-config-server-service    ClusterIP   10.233.50.72    <none>        8080/TCP   100s
service/cp-pipeline-inspection-api-service   ClusterIP   10.233.44.110   <none>        8085/TCP   100s
service/cp-pipeline-jenkins-service          ClusterIP   10.233.33.33    <none>        8080/TCP   103s
service/cp-pipeline-postgresql               ClusterIP   10.233.19.130   <none>        5432/TCP   103s
service/cp-pipeline-postgresql-headless      ClusterIP   None            <none>        5432/TCP   103s
service/cp-pipeline-sonarqube-sonarqube      ClusterIP   10.233.8.200    <none>        9000/TCP   101s
service/cp-pipeline-ui-service               ClusterIP   10.233.8.123    <none>        8084/TCP   100s

NAME                                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cp-pipeline-api-deployment              1/1     1            1           100s
deployment.apps/cp-pipeline-common-api-deployment       1/1     1            1           100s
deployment.apps/cp-pipeline-config-server-deployment    1/1     1            1           100s
deployment.apps/cp-pipeline-inspection-api-deployment   1/1     1            1           100s
deployment.apps/cp-pipeline-jenkins-deployment          1/1     1            1           103s
deployment.apps/cp-pipeline-sonarqube-sonarqube         1/1     1            1           101s
deployment.apps/cp-pipeline-ui-deployment               1/1     1            1           100s

NAME                                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/cp-pipeline-api-deployment-6d99f4c7d7             1         1         1       100s
replicaset.apps/cp-pipeline-common-api-deployment-7488bc6d5d      1         1         1       100s
replicaset.apps/cp-pipeline-config-server-deployment-5f676b858c   1         1         1       100s
replicaset.apps/cp-pipeline-inspection-api-deployment-f5bfbdccc   1         1         1       100s
replicaset.apps/cp-pipeline-jenkins-deployment-5f75b74b6d         1         1         1       103s
replicaset.apps/cp-pipeline-sonarqube-sonarqube-5f6c94bdbc        1         1         1       101s
replicaset.apps/cp-pipeline-ui-deployment-845bb6999f              1         1         1       100s

NAME                                                 READY   AGE
statefulset.apps/cp-pipeline-postgresql-postgresql   1/1     103s
```    
```bash
# 멀티 클러스터 환경에 배포 시 Pod 현황
NAME                                                     READY   STATUS    RESTARTS      AGE
cp-pipeline-api-deployment-6978c48fbb-wkhmk              1/1     Running   0             3m45s
cp-pipeline-common-api-deployment-558bb4cbdc-wxf2h       2/2     Running   0             3m43s
cp-pipeline-config-server-deployment-68674cf765-kcwls    2/2     Running   0             3m41s
cp-pipeline-inspection-api-deployment-75c58bcffd-szflk   1/1     Running   0             3m45s
cp-pipeline-jenkins-deployment-64ff4ccd64-bvf6r          1/1     Running   0             3m40s
cp-pipeline-postgresql-postgresql-0                      1/1     Running   0             3m39s
cp-pipeline-sonarqube-sonarqube-5b8d4b4856-297qn         1/1     Running   0             3m37s
cp-pipeline-ui-deployment-58c5ffdc6c-2lgq9               2/2     Running   0             3m42s
```

<br>


### <div id='3.4'>3.4. (참고) 컨테이너 플랫폼 파이프라인 리소스 삭제
배포된 컨테이너 플랫폼 파이프라인 리소스의 삭제를 원하는 경우 아래 스크립트를 실행한다.<br>
> :loudspeaker: 컨테이너 플랫폼을 통해 설치된 클러스터의 StorageClass 타입이 `NFS`인 경우 reclaim 정책은 `Retain`이다.<br>
> `Retain`정책은 Persistent Volume을 삭제하여도 스토리지 NFS 서버에 데이터가 여전히 존재하므로<br> 수동으로 데이터 정리가 필요하다.

```bash
$ cd ~/workspace/container-platform/cp-pipeline-deployment/script
$ chmod +x uninstall-cp-pipeline.sh
$ ./uninstall-cp-pipeline.sh
Are you sure you want to delete the container platform pipeline? <y/n> y # y 입력
```

<br>

## <div id='4'>4. 컨테이너 플랫폼 파이프라인 접속
컨테이너 플랫폼 파이프라인에 접속한다.<br><br>
**컨테이너 플랫폼 파이프라인 URL** : `https://pipeline.${HOST_DOMAIN}`
+ [[3.2. 컨테이너 플랫폼 파이프라인 변수 정의]](#3.2) 에서 정의한 `HOST_DOMAIN` 값 입력

<br>

### <div id='4.1'/>4.1. 컨테이너 플랫폼 파이프라인 관리자 로그인
컨테이너 플랫폼 포털 관리자 계정으로 파이프라인에 로그인한다.
> [[(참고) 컨테이너 플랫폼 포털 관리자 계정 로그인]](../portal/cp-portal-standalone-guide.md#4.1)
+ **Username** : `admin`
+ **Password** : `초기화한 패스워드 값`

![image 002]

<br>    


### <div id='4.2'/>4.2. 컨테이너 플랫폼 파이프라인 사용자 계정 로그인
#### 사용자 회원가입
- 하단의 'Register' 버튼을 클릭한다.

![image 003]

- 등록할 사용자 계정정보를 입력 후 'Register' 버튼을 클릭하여 컨테이너 플랫폼 파이프라인에 회원가입한다.

![image 004]

#### 사용자 로그인
- 회원가입을 통해 등록된 계정으로 컨테이너 플랫폼 파이프라인에 로그인한다.

![image 006]

<br>    

### <div id='4.3'/>4.3. 컨테이너 플랫폼 파이프라인 사용 가이드
- 컨테이너 플랫폼 파이프라인 사용방법은 아래 사용가이드를 참고한다.
   + [컨테이너 플랫폼 파이프라인 사용 가이드](../../use-guide/pipeline/cp-pipeline-use-guide.md)

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > 컨테이너 플랫폼 파이프라인 배포 가이드

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
