### [Index](https://github.com/K-PAAS/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > CSP 쿠버네티스 서비스 Istio 멀티 클러스터 구성 가이드

<br>

## Table of Contents
1. [문서 개요](#1)    
   1.1. [목적](#1.1)  
   1.2. [범위](#1.2)  
   1.3. [시스템 구성도](#1.3)  
   1.4. [참고 자료](#1.4)

2. [정보](#2)  
   2.1. [Prerequisite](#2.1)  
   2.2. [설치 목록](#2.2)  
   2.3. [방화벽 정보](#2.3)  
   2.4. [시연 클러스터 환경](#2.4)  
   
3. [Istio 멀티 클러스터 구성](#3)  
   3.1. [Deployment 파일 다운로드](#3.1)   
   3.2. [도구 설치](#3.2)  
   3.3. [멀티 클러스터 접근 구성](#3.3)       
   3.4. [Istio 멀티 클러스터 구성 스크립트 실행](#3.4)   

4. [샘플 애플리케이션 배포](#4)  
   4.1. [멀티 클러스터 샘플 애플리케이션 배포](#4.1)     
   4.2. [멀티 클러스터 통신 테스트](#4.2)    

5. [Istio 멀티 클러스터 구성 삭제](#5)  

6. [컨테이너 플랫폼 포털 배포 시 사전 설정](#6)    
   6.1. [StorageClass 설정](#6.1)  
   6.2. [Metrics Server 설치](#6.2)  
   6.3. [Cilium CNI 사용 클러스터 설정 변경](#6.3)    
  
7. [참고](#7)  
   7.1. [Istio Ingressgateway EXTERNAL-IP 수동 구성](#7.1)  
   7.2. [Horizontal Pod Autoscaling unknown 조치](#7.2)  
    

<br>

## <span id='1'> 1. 문서 개요
### <span id='1.1'> 1.1. 목적
본 문서(CSP 쿠버네티스 서비스 Istio 멀티 클러스터 구성 가이드)는 클라우드 서비스 제공업체의 Kubernetes 서비스에 Istio 멀티 클러스터 구성 방법을 기술하여 독립적인 Kubernetes 클러스터의 서비스 간 통신이 가능하도록 한다.

<br>

### <span id='1.2'>1.2. 범위
`CSP 쿠버네티스 서비스` 클러스터를 대상으로 Istio를 이용하여 `멀티 클러스터`를 구성하도록 작성하였다.

<br>

### <span id='1.3'>1.3. 시스템 구성도
시스템 구성은 서로 다른 네트워크인 N개의 Kubernetes 클러스터에 Istio Control Plane을 설치하여 Istio Mesh를 구성한다.<br>
클러스터 cluster N의 기본 네트워크는 network N으로 설정되며 클러스터 Pod 간의 직접적인 통신 없이 Istio Gateway를 통해 클러스터는 통신한다.

> *이미지 출처* <br>
> [Install Multi-Primary on different networks](https://istio.io/latest/docs/setup/install/multicluster/multi-primary_multi-network/)<br>

<img src="https://istio.io/latest/docs/setup/install/multicluster/multi-primary_multi-network/arch.svg" width="800">


<br>

### <span id='1.4'>1.4. 참고 자료
> [Istio Install Multi-Primary on different networks](https://istio.io/latest/docs/setup/install/multicluster/multi-primary_multi-network/)<br>

<br>

## <span id='2'> 2. 정보

### <span id='2.1'>2.1. Prerequisite
- 작업 인스턴스는 **Ubuntu 22.04** 환경에서 진행하는 것을 기준으로 한다.
- 각 클러스터는 **로드 밸런서(LoadBalancer)** 유형의 서비스 지원이 필요하다.

<br>

### <span id='2.2'>2.2. 설치 목록
설치되는 도구 목록은 아래와 같다.
| 도구 | 버전 |
| :---: | :---: |  
| kubectl | v1.32.3 |
| Helm | v3.16.4 |
| step | 0.24.4 |
| Podman | - |
| ca-certificates | - |
| Istio | 1.26.0 |

| Istio 버전 | [Kubernetes 지원 버전](https://istio.io/latest/docs/releases/supported-releases/#support-status-of-istio-releases)|  
|:--------:| :---: |  
|  `1.26`  |1.29, 1.30, 1.31, 1.32|  

<br>

### <span id='2.3'>2.3. 방화벽 정보
IaaS Security Group의 열어줘야할 Port를 설정한다.
|프로토콜|포트|비고|
| :---: | :---: | --- | 
|TCP|15021|Istio ingressgateway status|
|TCP|80|Istio ingressgateway http|
|TCP|443|Istio ingressgateway https|
|TCP|31400|Istio ingressgateway tcp|
|TCP|15443|Istio ingressgateway mtls|
|TCP|15012|Istio ingressgateway istiod|
|TCP|15017|Istio ingressgateway webhook|


<br>
<br>

### <span id='2.4'>2.4. 시연 클러스터 환경
Istio를 활용하여 **`3개의 클러스터`** 를 기반으로 멀티 클러스터 환경을 구성하는 예제를 제공한다.
> 본 가이드는 국내 CSP 3사 쿠버네티스 클러스터 서비스를 대상으로 한다.

#### CSP 쿠버네티스 클러스터 환경
| Kubernetes Service                    | CNI     |
|---------------------------------------|---------| 
| KT Kubernetes Service (K2P Standard)  | Calico  |
| Ncloud Kubernetes Service (NKS)       | Cilium  |
| NHN kubernetes Service (NKS)          | Calico  | 


<br>

## <span id='3'>3. Istio 멀티 클러스터 구성
>  Istio를 활용하여 **`3개의 클러스터`** 를 기반으로 멀티 클러스터 환경을 구성하는 예제를 제공한다.

### <span id='3.1'>3.1. Deployment 파일 다운로드
Istio 멀티 클러스터 구성을 위해 컨테이너 플랫폼 포털 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>

+ 컨테이너 플랫폼 포털 Deployment 파일 다운로드 :
  [cp-portal-deployment-v1.6.1.1.tar.gz](https://nextcloud.k-paas.org/index.php/s/jyjGsowwx3AHNPk/download)

```bash
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.k-paas.org/index.php/s/jyjGsowwx3AHNPk/download

$ ls ~/workspace/container-platform
  cp-portal-deployment-v1.6.1.1.tar.gz

# Deployment 파일 압축 해제
$ tar -xvf cp-portal-deployment-v1.6.1.1.tar.gz
```

<br>

### <span id='3.2'>3.2. 도구 설치
> [[2.1. 설치 목록]](#2.1) <br>

Istio 멀티 클러스터 구성을 위해 필요한 커맨드 라인 등 도구의 사전 설치가 필요하다. <br>
`kubectl`의 경우 클러스터의 마이너(minor) 버전 차이 내에 있는 버전 사용이 필요하므로 클러스터 버전을 확인하여<br>
변경이 필요한 경우 아래 파일을 수정한다.
```bash
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc
$ vi istio-vars-mc.sh
```
```bash
# 설치할 버전으로 변경 
# command line tool
KUBECTL_VERSION="1.32.3"
HELM_VERSION="3.16.4"
STEP_VERSION="0.24.4"

# Istio
ISTIO_VERSION="1.26.0"
...
```

도구 설치 스크립트를 실행한다.
```bash
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc
$ chmod +x install_tools.sh
$ ./install_tools.sh
```

<br>

### <span id='3.3'>3.3. 멀티 클러스터 접근 구성
Istio 멀티 클러스터 구성은 `kubectl config get-contexts` 명령어로 출력된 컨텍스트 목록을 기반으로 진행된다.<br>
따라서 각 클러스터에 접근할 수 있도록 컨텍스트를 미리 구성해야 한다.<br>
:loudspeaker: 각 클러스터 kubeconfig 파일 내 cluster, context, user 명이 중복되지 않도록 한다.
```bash
# .kube 디렉터리 생성 및 이동
$ mkdir -p ${HOME}/.kube
$ cd ${HOME}/.kube

# 각 cluster kubeconfig 파일 위치
$ ls ${HOME}/.kube
cluster1-config  cluster2-config  cluster3-config

# kubeconfig 파일 경로 설정
$ export KUBECONFIG="${HOME}/.kube/cluster1-config:${HOME}/.kube/cluster2-config:${HOME}/.kube/cluster3-config"
```
##### :bulb:  `NAME`, `CLUSTER`, `AUTHINFO` 값 출력 정상 확인
```bash
$ kubectl config get-contexts
CURRENT   NAME     CLUSTER      AUTHINFO     NAMESPACE
*         kt       kt-k2p       kt-k2p
          ncloud   ncloud-nks   ncloud-nks
          nhn      nhn-nks      nhn-nks
```
##### 클러스터 리소스 조회 정상 확인
```bash
# cluster1 (kt) 노드 조회
$ kubectl get nodes --context=kt
NAME                       STATUS   ROLES           AGE    VERSION
kt-k2p-standard.master01   Ready    control-plane   5d6h   v1.30.4
kt-k2p-standard.worker01   Ready    <none>          5d6h   v1.30.4
kt-k2p-standard.worker02   Ready    <none>          5d6h   v1.30.4

# cluster2 (ncloud) 노드 조회
$ kubectl get nodes --context=ncloud
NAME                    STATUS   ROLES    AGE    VERSION
ncloud-nks-w-2lmr       Ready    <none>   5d6h   v1.28.10
ncloud-nks-w-2lms       Ready    <none>   5d6h   v1.28.10

# cluster3 (nhn) 노드 조회
$ kubectl get nodes --context=nhn
NAME                                STATUS   ROLES    AGE    VERSION
nhn-nks-default-worker-node-0       Ready    <none>   5d6h   v1.30.3
nhn-nks-default-worker-node-1       Ready    <none>   5d6h   v1.30.3
```

<br>


### <span id='3.4'>3.4. Istio 멀티 클러스터 구성 스크립트 실행
Istio 멀티 클러스터 구성을 위한 스크립트를 실행한다.
```bash
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc
$ chmod +x deploy-istio-mc.sh
$ ./deploy-istio-mc.sh
```
```bash
# (예시) 다음은 클러스터 3개를 대상으로 실행한 Istio 설치 스크립트 로그이다.
[Generate Root CA]...
Your certificate has been saved in certs/root-cert.pem.
Your private key has been saved in certs/root-ca.key.
[Install Istio in cluster1]...
Your certificate has been saved in certs/kt-k2p/ca-cert.pem.
Your private key has been saved in certs/kt-k2p/ca-key.pem.
namespace/istio-system created
secret/cacerts created
        |\
        | \
        |  \
        |   \
      /||    \
     / ||     \
    /  ||      \
   /   ||       \
  /    ||        \
 /     ||         \
/______||__________\
____________________
  \__       _____/
     \_____/

✔ Istio core installed ⛵️
✔ Istiod installed 🧠
✔ Installation complete
        |\
        | \
        |  \
        |   \
      /||    \
     / ||     \
    /  ||      \
   /   ||       \
  /    ||        \
 /     ||         \
/______||__________\
____________________
  \__       _____/
     \_____/

✔ Ingress gateways installed 🛬
✔ Installation complete
gateway.networking.istio.io/cross-network-gateway created
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
[Install Istio in cluster2]...
Your certificate has been saved in certs/ncloud-nks/ca-cert.pem.
Your private key has been saved in certs/ncloud-nks/ca-key.pem.
namespace/istio-system created
secret/cacerts created
        |\
        | \
        |  \
        |   \
      /||    \
     / ||     \
    /  ||      \
   /   ||       \
  /    ||        \
 /     ||         \
/______||__________\
____________________
  \__       _____/
     \_____/

✔ Istio core installed ⛵️
✔ Istiod installed 🧠
✔ Installation complete
        |\
        | \
        |  \
        |   \
      /||    \
     / ||     \
    /  ||      \
   /   ||       \
  /    ||        \
 /     ||         \
/______||__________\
____________________
  \__       _____/
     \_____/

✔ Ingress gateways installed 🛬
✔ Installation complete
gateway.networking.istio.io/cross-network-gateway created
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
[Install Istio in cluster3]...
Your certificate has been saved in certs/nhn-nks/ca-cert.pem.
Your private key has been saved in certs/nhn-nks/ca-key.pem.
namespace/istio-system created
secret/cacerts created
        |\
        | \
        |  \
        |   \
      /||    \
     / ||     \
    /  ||      \
   /   ||       \
  /    ||        \
 /     ||         \
/______||__________\
____________________
  \__       _____/
     \_____/

✔ Istio core installed ⛵️
✔ Istiod installed 🧠
✔ Installation complete
        |\
        | \
        |  \
        |   \
      /||    \
     / ||     \
    /  ||      \
   /   ||       \
  /    ||        \
 /     ||         \
/______||__________\
____________________
  \__       _____/
     \_____/

✔ Ingress gateways installed 🛬
✔ Installation complete
gateway.networking.istio.io/cross-network-gateway created
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
secret/istio-remote-secret-kt-k2p created
secret/istio-remote-secret-kt-k2p created
secret/istio-remote-secret-ncloud-nks created
secret/istio-remote-secret-ncloud-nks created
secret/istio-remote-secret-nhn-nks created
secret/istio-remote-secret-nhn-nks created

--------------------------------------------------------------
[cluster1 (kt)] $ istioctl remote-clusters
--------------------------------------------------------------
NAME           SECRET                                          STATUS     ISTIOD
kt-k2p                                                         synced     istiod-69f99b4667-2rbmd
ncloud-nks     istio-system/istio-remote-secret-ncloud-nks     synced     istiod-69f99b4667-2rbmd
nhn-nks        istio-system/istio-remote-secret-nhn-nks        synced     istiod-69f99b4667-2rbmd

--------------------------------------------------------------
[cluster2 (ncloud)] $ istioctl remote-clusters
--------------------------------------------------------------
NAME           SECRET                                       STATUS     ISTIOD
ncloud-nks                                                  synced     istiod-5c6c96f9c4-xg66w
kt-k2p         istio-system/istio-remote-secret-kt-k2p      synced     istiod-5c6c96f9c4-xg66w
nhn-nks        istio-system/istio-remote-secret-nhn-nks     synced     istiod-5c6c96f9c4-xg66w

--------------------------------------------------------------
[cluster3 (nhn)] $ istioctl remote-clusters
--------------------------------------------------------------
NAME           SECRET                                          STATUS     ISTIOD
nhn-nks                                                        synced     istiod-5bdd85f847-swxk4
kt-k2p         istio-system/istio-remote-secret-kt-k2p         synced     istiod-5bdd85f847-swxk4
ncloud-nks     istio-system/istio-remote-secret-ncloud-nks     synced     istiod-5bdd85f847-swxk4
```
<br>

#### Istio 리소스 배포 현황
> 클러스터 컨텍스트 명 환경변수 설정
```bash
$ source ~/workspace/container-platform/cp-portal-deployment/istio_mc/istio-vars-mc.sh
```
#### cluster1 (KT K2P) Istio 리소스 배포 현황 확인
**KT K2P Standard(K8S)** 는 LoadBalancer 서비스 타입(자동 생성) 미지원으로 istio-ingressgateway의 EXTERNAL-IP 값이 `Pending` 상태이다. <br>
LoadBalancer 생성 및 서비스 istio-ingressgateway의 EXTERNAL-IP 값 할당을 위해 아래 가이드를 참고하여 처리한다. <br>
또한 HorizontalPodAutoscaler Targets 값이 `unknown`인 경우 아래 가이드를 참고한다.
> [[7.1. Istio Ingressgateway EXTERNAL-IP 수동 구성]](#7.1) <br>
> [[7.2. Horizontal Pod Autoscaling unknown 조치]](#7.2) 

```bash
$ kubectl get all -n ${ISTIO_NAMESPACE} --context=${CLUSTER1_CONFIG[CTX]}
NAME                                       READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-fd6887d6f-6kzzn   1/1     Running   0          8m46s
pod/istiod-69f99b4667-2rbmd                1/1     Running   0          8m51s

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP        PORT(S)                                                                                                      AGE
service/istio-ingressgateway   LoadBalancer   10.103.54.27     210.xxx.xxx.xxx    15021:32654/TCP,80:31323/TCP,443:30658/TCP,31400:32070/TCP,15443:31125/TCP,15012:30960/TCP,15017:31546/TCP   8m46s
service/istiod                 ClusterIP      10.100.252.167   <none>             15010/TCP,15012/TCP,443/TCP,15014/TCP                                                                        8m51s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   1/1     1            1           8m46s
deployment.apps/istiod                 1/1     1            1           8m51s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-ingressgateway-fd6887d6f   1         1         1       8m46s
replicaset.apps/istiod-69f99b4667                1         1         1       8m51s

NAME                                                       REFERENCE                         TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   cpu: 3%/80%   1         5         1          8m46s

```

#### cluster2 (Ncloud NKS) Istio 리소스 배포 현황 확인
**Ncloud Kubernetes Service**는 기본 CNI로 `Cilium`을 제공한다. <br>
Cilium을 사용하는 Kubernetes 클러스터에서 Istio 사용과 관련된 내용을 참고한다.
> [[Cilium’s integration with Istio]](https://docs.cilium.io/en/stable/network/servicemesh/istio) <br>
> [(참고) [6.3.Cilium CNI 사용 클러스터 설정 변경]](#6.3) 


```bash
$ kubectl get all -n ${ISTIO_NAMESPACE} --context=${CLUSTER2_CONFIG[CTX]}
NAME                                        READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-7596586df5-77kwt   1/1     Running   0          9m14s
pod/istiod-5c6c96f9c4-xg66w                 1/1     Running   0          9m22s

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP                         PORT(S)                                                                                                      AGE
service/istio-ingressgateway   LoadBalancer   198.19.236.169   istio-syste-istio-ingres-xxx.com    15021:30852/TCP,80:30966/TCP,443:30811/TCP,31400:30812/TCP,15443:31121/TCP,15012:31116/TCP,15017:30571/TCP   9m14s
service/istiod                 ClusterIP      198.19.150.8     <none>                              15010/TCP,15012/TCP,443/TCP,15014/TCP                                                                        9m22s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   1/1     1            1           9m15s
deployment.apps/istiod                 1/1     1            1           9m23s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-ingressgateway-7596586df5   1         1         1       9m16s
replicaset.apps/istiod-5c6c96f9c4                 1         1         1       9m24s

NAME                                                       REFERENCE                         TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   3%/80%    1         5         1          9m16s
```

#### cluster3 (NHN NKS) Istio 리소스 배포 현황 확인
```
$ kubectl get all -n ${ISTIO_NAMESPACE} --context=${CLUSTER3_CONFIG[CTX]}
NAME                                        READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-5df95c4895-kcrrv   1/1     Running   0          10m
pod/istiod-5bdd85f847-swxk4                 1/1     Running   0          10m

NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                                                                                                      AGE
service/istio-ingressgateway   LoadBalancer   10.254.216.17   133.xxx.xxx.xxx   15021:32380/TCP,80:32215/TCP,443:31685/TCP,31400:31715/TCP,15443:31549/TCP,15012:32498/TCP,15017:31299/TCP   10m
service/istiod                 ClusterIP      10.254.166.12   <none>            15010/TCP,15012/TCP,443/TCP,15014/TCP                                                                        10m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   1/1     1            1           10m
deployment.apps/istiod                 1/1     1            1           10m

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-ingressgateway-5df95c4895   1         1         1       10m
replicaset.apps/istiod-5bdd85f847                 1         1         1       10m

NAME                                                       REFERENCE                         TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   cpu: 2%/80%   1         5         1          10m
```
<br> 


## <span id='4'>4. 샘플 애플리케이션 배포
멀티 클러스터 통신 테스트를 위해 클러스터 cluster1, cluster2, cluster3에 샘플 애플리케이션을 배포한다.
> [Istio Bookinfo Application](https://istio.io/latest/docs/examples/bookinfo/)의 일부 애플리케이션 배포
#### 애플리케이션 배포 위치
| 애플리케이션| cluster1 | cluster2 | cluster3 | 정보 |
| :---: |:---: | :---: | :---: | :---: | 
| reviews-v1 | :heavy_check_mark: | | | 해당 버전(v1)과 호스트 명을 반환 | 
| reviews-v2 | |:heavy_check_mark: | | 해당 버전(v2)과 호스트 명을 반환 | 
| reviews-v3 | | |:heavy_check_mark: | 해당 버전(v3)과 호스트 명을 반환 | 
| sleep | | | :heavy_check_mark: |다른 서비스를 호출하기 위한 요청 소스로 사용| 


```bash
# 클러스터 컨텍스트 명 환경변수 설정
$ source ~/workspace/container-platform/cp-portal-deployment/istio_mc/istio-vars-mc.sh

# sample 디렉토리 생성 
$ mkdir -p ~/workspace/container-platform/cp-portal-deployment/istio_mc/sample
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc/sample

# bookinfo 서비스 배포 yaml 다운로드
$ wget https://raw.githubusercontent.com/istio/istio/release-1.24/samples/bookinfo/platform/kube/bookinfo.yaml
# sleep 서비스 배포 yaml 다운로드
$ wget https://raw.githubusercontent.com/istio/istio/master/samples/sleep/sleep.yaml
```

<br>

### <span id='4.1'>4.1. 멀티 클러스터 샘플 애플리케이션 배포
```bash
# 각 클러스터에 sample 네임스페이스 생성 및 Istio 사이드카 프록시 자동 주입 활성화
# clusterN에 reviews-N 애플리케이션 배포
for IDX in $(seq 1 "$CLUSTER_CNT"); do CTX="CLUSTER${IDX}_CONFIG[CTX]"; \
  echo "context: ${!CTX}"
  kubectl --context=${!CTX} create ns sample; \
  kubectl --context=${!CTX} label namespace sample istio-injection=enabled; \
  kubectl --context=${!CTX} -n sample apply -f bookinfo.yaml -l service=reviews; \
  kubectl --context=${!CTX} -n sample apply -f bookinfo.yaml -l account=reviews; \
  kubectl --context=${!CTX} -n sample apply -f bookinfo.yaml -l app=reviews,version=v${IDX}; \
done
```
```bash
# cluster3에 sleep 애플리케이션 배포
$ kubectl --context="${CLUSTER3_CONFIG[CTX]}" -n sample apply -f sleep.yaml
```

<br>


### <span id='4.2'>4.2. 멀티 클러스터 통신 테스트
```bash
# cluster1의 reviews-v1 pod 조회
$ kubectl get pods -l app=reviews --context="${CLUSTER1_CONFIG[CTX]}" -n sample
NAME                          READY   STATUS    RESTARTS   AGE
reviews-v1-67896867f4-2rtf4   2/2     Running   0          2m17s

# cluster2의 reviews-v2 pod 조회
$ kubectl get pods -l app=reviews --context="${CLUSTER2_CONFIG[CTX]}" -n sample
NAME                          READY   STATUS    RESTARTS   AGE
reviews-v2-65c9797659-zp6mf   2/2     Running   0          2m42s

# cluster3의 reviews-v3, sleep pod 조회
$ kubectl get pods -l app=reviews --context="${CLUSTER3_CONFIG[CTX]}" -n sample
NAME                          READY   STATUS    RESTARTS   AGE
reviews-v3-77947c4c78-7vvkf   2/2     Running   0          3m9s

$ kubectl get pods -l app=sleep --context="${CLUSTER3_CONFIG[CTX]}" -n sample
NAME                     READY   STATUS    RESTARTS   AGE
sleep-5577c64d7c-9zg7w   2/2     Running   0          74s
```

```bash
# cluster3의 sleep pod에 접속 
$ kubectl exec -it -n sample --context="${CLUSTER3_CONFIG[CTX]}" \
"$(kubectl get pod -n sample --context="${CLUSTER3_CONFIG[CTX]}" -l app=sleep -o jsonpath='{.items[0].metadata.name}')" -c sleep -- sh

# reviews 서비스 curl 통신
~ $ curl reviews:9080/reviews/1
{"id": "1","podname": "reviews-v3-77947c4c78-7vvkf","clustername": "null","reviews":...  # cluster3의 reviews-v3 호출 
~ $ curl reviews:9080/reviews/1
{"id": "1","podname": "reviews-v2-65c9797659-zp6mf","clustername": "null","reviews":...  # cluster2의 reviews-v2 호출 
~ $ curl reviews:9080/reviews/1
{"id": "1","podname": "reviews-v2-65c9797659-zp6mf","clustername": "null","reviews":...  # cluster2의 reviews-v2 호출 
~ $ curl reviews:9080/reviews/1
{"id": "1","podname": "reviews-v1-67896867f4-2rtf4","clustername": "null","reviews":...  # cluster1의 reviews-v1 호출 
```

<br>

##### :wastebasket: 샘플 애플리케이션 삭제
샘플 애플리케이션 테스트가 종료되면 다음 스크립트를 실행하여 리소스를 정리한다.
```bash
# sample 네임스페이스 삭제
for IDX in $(seq 1 "$CLUSTER_CNT"); do CTX="CLUSTER${IDX}_CONFIG[CTX]"; \
  echo "context: ${!CTX}"; kubectl --context=${!CTX} delete ns sample; \
done
```

<br>

## <span id='5'>5. Istio 멀티 클러스터 구성 삭제
Istio 멀티 클러스터 구성의 삭제를 원하는 경우 아래 스크립트를 실행한다.<br>
:loudspeaker: (주의) 해당 스크립트 실행 시, **Istio 멀티클러스터 구성이 모두 제거**되므로 주의가 필요하다.<br>

```bash
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc
$ chmod +x uninstall-istio-mc.sh
$ ./uninstall-istio-mc.sh
Are you sure you want to uninstall istio in all clusters? <y/n> y # y 입력
```

<br>

## <span id='6'>6. 컨테이너 플랫폼 포털 배포 시 사전 설정
해당 가이드를 통해 구성된 Istio 멀티 클러스터 환경에 컨테이너 플랫폼 포털을 배포할 경우 아래 사전 설정이 필요하다.
### <span id='6.1'>6.1. StorageClass 설정
각 클러스터에 `StorageClass`가 설정되어 있는지 확인한다.
> StorageClass를 제공하지 않는다면 별도 설치 필요<br>
> [NFS Server 설치 가이드](../nfs-server-install-guide.md)
```bash
# 클러스터 cluster1의 StorageClass 조회 (별도 설치)
$ kubectl get storageclass --context=${CLUSTER1_CONFIG[CTX]}
NAME              PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
cp-storageclass   cp-nfs-provisioner   Delete          Immediate           false                  2m46s

# 클러스터 cluster2의 StorageClass 조회
$ kubectl get storageclass --context=${CLUSTER2_CONFIG[CTX]}
NAME                          PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
nks-block-storage (default)   blk.csi.ncloud.com   Delete          WaitForFirstConsumer   true                   5h14m
```

<br>

### <span id='6.2'>6.2. Metrics Server 설치
컨테이너 플랫폼 관리 클러스터의 Metrics 정보 수집을 위해 Metrics Server 설치가 필요하다.
> [Metrics Server 설치](#metrics-server-설치) 

<br>

### <span id='6.3'>6.3. Cilium CNI 사용 클러스터 설정 변경 
CNI로 `Cilium`을 사용하는 클러스터에서 Istio 구성 시 아래 설정 변경이 필요하다.  
> [Cilium’s integration with Istio](https://docs.cilium.io/en/stable/network/servicemesh/istio) 

#### Cilium Configmap 수정
```bash
$ kubectl edit configmaps cilium-config -n kube-system
```
```yaml
# 추가
bpf-lb-sock-hostns-only: "true"
cni-exclusive: "false"

# 수정
enable-l7-proxy: "false" 
```

#### Cilium DaemonSet 재시작
```bash
# cilium daemonset 재시작
$ kubectl rollout restart daemonset cilium -n kube-system
daemonset.apps/cilium restarted

# cilium pod 상태 'running' 조회
$ kubectl get pods -n kube-system -l k8s-app=cilium
NAME           READY   STATUS    RESTARTS   AGE
cilium-b4l8k   1/1     Running   0          4m32s
cilium-sqps5   1/1     Running   0          4m32s
```

<br> 

## <span id='7'>7. 참고 

### <span id='7.1'>7.1. Istio Ingressgateway EXTERNAL-IP 수동 구성
> KT K2P Standard(K8S) 서비스를 예시로 한다.

Istio 설치 후 서비스 istio-ingressgateway의 초기 상태는 아래와 같다.
- EXTERNAL-IP `pending` 상태
- NodePort 자동할당
```bash
$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                      AGE
istio-ingressgateway   LoadBalancer   10.103.54.27     <pending>     15021:32654/TCP,80:31323/TCP,443:30658/TCP,31400:32070/TCP,15443:31125/TCP,15012:30960/TCP,15017:31546/TCP   14m
istiod                 ClusterIP      10.100.252.167   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                                                        14m   
```

#### <span id='7.1.1'>7.1.1 LoadBalancer 생성
서비스 istio-ingressgateway의 Port와 NodPort 정보를 확인하여 LoadBalancer를 생성한다. <br>
> kt cloud 콘솔 화면 접속 > Load Balancer > Load Balancer 관리 > Load Balancer 생성

<table>
<thead>
  <tr>
    <th>No</th>
    <th>Service Port</th>
    <th>Service IP</th>
    <th>Zone</th>
    <th>Tier</th>
    <th>Service Type</th>
    <th>Service Options</th>
    <th>Health Check</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td align="center">⑴</td>
    <td align="center">15021</td>
    <td align="center">신규 IP 할당</td>
    <td align="center" rowspan="7">클러스터 노드와<br>동일한 네트워크<br>Zone 선택<br>(VPC와 동일 개념)</td>
    <td align="center" rowspan="7">클러스터 노드와<br>동일한 네트워크<br>Tier 선택<br>(Subnet과 동일 개념)</td>
    <td align="center" rowspan="7">TCP</td>
    <td align="center" rowspan="7">Round Robin</td>
    <td align="center" rowspan="7">TCP</td>
  </tr>
  <tr>
    <td align="center">⑵</td>
    <td align="center">80</td>
    <td align="center" rowspan="6">⑴을 통해<br> 생성된 IP 선택</td>
  </tr>
  <tr>
    <td align="center">⑶</td>
    <td align="center">443</td>
  </tr>
  <tr>
    <td align="center">⑷</td>
    <td align="center">31400</td>
  </tr>
  <tr>
    <td align="center">⑸</td>
    <td align="center">15443</td>
  </tr>
  <tr>
    <td align="center">⑹</td>
    <td align="center">15012</td>
  </tr>
  <tr>
    <td align="center">⑺</td>
    <td align="center">15017</td>
  </tr>
</tbody>
</table>

###### (예시) 포트 `15021` LoadBalancer 생성
![image 001]

###### (예시) 포트 `80` LoadBalancer 생성 
![image 003]

###### (예시) 생성한 LoadBalancer 목록 
![image 004]

#### VM 연결
생성한 LoadBalancer에 클러스터 전체 노드를 순차적으로 연결한다.
> kt cloud 콘솔 화면 접속 > Load Balancer > Load Balancer 관리 >  `⑴ 15021 ~ ⑺ 15017 모두 진행` LoadBalancer 선택 > VM 연결/해제 클릭

|항목|설명|
|---|---|
|Tier|클러스터 노드와 동일한 네트워크 Tier 선택|
|VM|클러스터 전체 노드를 순차적으로 선택|
|Public Port|포트에 할당된 노드포트 정보 입력|

![image 002]



<br>

#### <span id='7.1.2'>7.1.2 Public IP 생성
서비스 istio-ingressgateway의 EXTERNAL-IP에 할당할 Public IP를 생성한다.<br>
> kt cloud 콘솔 화면 접속 > Servers > Networking > `IP 생성` 버튼 클릭하여 Public IP 생성

![image 005]

##### 생성한 Public IP 선택 > 'Static NAT' 버튼 클릭 > Resource 'Load-Balancer' 탭 클릭 > 생성한 LoadBalancer 중 1개 선택
![image 006]

##### Public IP의 Static NAT에 LoadBalancer 설정 확인
![image 007]

##### Public IP 선택 > '방화벽 설정' 버튼 클릭 > 방화벽 Rule 등록
![image 008]


<br>

#### <span id='7.1.3'>7.1.3 MetalLB 설치
> **[[MetalLB 설치]](https://metallb.universe.tf/installation/)** 베어메탈 클러스터 환경에서 로드 밸런서 기능 제공 <br>

서비스 istio-ingressgateway의 EXTERNAL-IP가 pending 또는 Public IP로 등록되지 않은 클러스터의 경우 MetalLB를 설치한다.
```bash
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
```

##### IPAddressPool 생성
> 생성한 [Public IP](#7.1.2)로 `spec.addresses` 값 변경
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  annotations:
  name: istio-ingressgateway
  namespace: metallb-system
spec:
  addresses:
  - 210.xxx.xxx.xxx/32 # (ip 변경)
  autoAssign: false
  avoidBuggyIPs: false
EOF
```
##### 서비스 istio-ingressgateway `loadBalancerIP` 값 없데이트
```bash
$ kubectl patch svc istio-ingressgateway -n istio-system -p '{"spec":{"loadBalancerIP":"210.xxx.xxx.xxx"}}' # (ip 변경)
```
```bash
$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP
istio-ingressgateway   LoadBalancer   10.101.94.112    210.xxx.xxx.xxx # (Public IP 할당 확인) ...
```

<br>

### <span id='7.2'>7.2. Horizontal Pod Autoscaling unknown 조치 
Istio 설치 후 HorizontalPodAutoscaler Targets 값이 `unknown`인 경우 **Metrics Server** 설치가 필요하다.
```bash
$ kubectl get horizontalpodautoscaler.autoscaling -n istio-system
NAME                   REFERENCE                         TARGETS              MINPODS   MAXPODS   REPLICAS
istio-ingressgateway   Deployment/istio-ingressgateway   cpu: <unknown>/80%   1         5         1       
```

#### Metrics Server 설치
> [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
```bash
$ wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
##### args `--kubelet-insecure-tls` 추가
```yaml
···
114 apiVersion: apps/v1
115 kind: Deployment
···
132     spec:
133       containers:
134       - args:
            - --kubelet-insecure-tls # (추가)
135         - --cert-dir=/tmp
136         - --secure-port=10250
···
```
##### Metrics Server 배포 
```bash
$ kubectl apply -f components.yaml
```

##### Istio Ingressgateway Horizontal Pod Autoscaling 확인
```bash
# metric-server 배포 정상 확인
$ kubectl get pods -n kube-system -l k8s-app=metrics-server
NAME                            READY   STATUS    RESTARTS   AGE
metrics-server-544865cd-n4cdp   1/1     Running   0          2m55s

# kubectl top 명령어 확인
$ kubectl top nodes
NAME                       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
kt-k2p-standard.master01   132m         6%     1833Mi          48%
kt-k2p-standard.worker01   38m          0%     968Mi           6%
kt-k2p-standard.worker02   51m          1%     1062Mi          6%

# horizontalpodautoscaler targets cpu 출력 확인
$ kubectl get horizontalpodautoscaler.autoscaling -n istio-system
NAME                   REFERENCE                         TARGETS              MINPODS   MAXPODS   REPLICAS
istio-ingressgateway   Deployment/istio-ingressgateway   cpu: 4%/80% (확인)   1         5         1        
```

<br>

### [Index](https://github.com/K-PAAS/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > CSP 쿠버네티스 서비스 Istio 멀티 클러스터 구성 가이드


[image 001]:../images/csp/csp-001.png
[image 002]:../images/csp/csp-002.png
[image 003]:../images/csp/csp-003.png
[image 004]:../images/csp/csp-004.png
[image 005]:../images/csp/csp-005.png
[image 006]:../images/csp/csp-006.png
[image 007]:../images/csp/csp-007.png
[image 008]:../images/csp/csp-008.png
