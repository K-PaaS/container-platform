### [Index](https://github.com/K-PAAS/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > Istio 멀티 클러스터 설치 가이드
<br>

## Table of Contents
1. [문서 개요](#1)    
    1.1. [목적](#1.1)  
    1.2. [범위](#1.2)  
    1.3. [시스템 구성도](#1.3)  
    1.4. [참고 자료](#1.4)  

2. [Prerequisite](#2)  
    2.1. [설치 목록](#2.1)  
    2.2. [방화벽 정보](#2.2)     

3. [Istio 멀티 클러스터 설치](#3)  
    3.1. [Deployment 파일 다운로드](#3.1)   
    3.2. [도구 설치](#3.2)  
    3.3. [멀티 클러스터 접근 구성](#3.3)       
    3.4. [Istio 멀티 클러스터 설치 변수 정의](#3.4)   
    3.5. [Istio 멀티 클러스터 설치 스크립트 실행](#3.5)  
    3.6. [(참고) Istio 멀티 클러스터 삭제](#3.6)   
       
4. [샘플 애플리케이션 배포](#4)  
    4.1. [클러스터 cluster1에 샘플 애플리케이션 배포](#4.1)     
    4.2. [클러스터 cluster2에 샘플 애플리케이션 배포](#4.2)    
    4.3. [멀티 클러스터 통신 테스트](#4.3)    

5. [(참고) 컨테이너 플랫폼 포털 배포 시 사전 설정](#5)  
    5.1. [StorageClass 설정](#5.1)  
    5.2. [Ingress NGINX Controller 배포](#5.2)    

   
<br>

## <span id='1'> 1. 문서 개요  
### <span id='1.1'> 1.1. 목적
본 문서(Istio 멀티 클러스터 설치 가이드)는 Kubernetes 클러스터 내 서비스 메시를 지원하는 Istio의 설치 방법을 기술하여 독립적인 두 Kubernetes 클러스터의 서비스 간 통신이 가능하도록 한다.

<br>

### <span id='1.2'>1.2. 범위
설치 범위는 Istio 멀티 클러스터 설치 기준으로 작성하였다.

<br>

### <span id='1.3'>1.3. 시스템 구성도
시스템 구성은 서로 다른 네트워크의 두 Kubernetes 클러스터에 Istio Control Plane을 설치하여 Istio Mesh를 구성한다.
클러스터 cluster1의 기본 네트워크는 network1로, 클러스터 cluster2의 기본 네트워크는 network2로 설정하여 클러스터 Pod 간의 직접적인 통신 없이 Istio Gateway를 통해 클러스터는 통신한다.

> *이미지 출처* <br>
> [Install Multi-Primary on different networks](https://istio.io/latest/docs/setup/install/multicluster/multi-primary_multi-network/)<br>
 
<img src="https://istio.io/latest/docs/setup/install/multicluster/multi-primary_multi-network/arch.svg" width="800">
 

<br>

### <span id='1.4'>1.4. 참고 자료
> [Istio Install Multi-Primary on different networks](https://istio.io/latest/docs/setup/install/multicluster/multi-primary_multi-network/)<br>

<br>

## <span id='2'> 2. Prerequisite
- 작업 인스턴스는 **Ubuntu 22.04** 또는 **Ubuntu 20.04** 환경에서 설치하는 것을 기준으로 한다.
- **두 개의 클러스터**를 대상으로 Istio 멀티 클러스터를 구성한다.
- 각 클러스터는 **로드 밸런서(LoadBalancer)** 유형의 서비스 지원이 필요하다.
<br>

### <span id='2.1'>2.1. 설치 목록
설치되는 도구 목록은 아래와 같다.
| 도구 | 버전 |
| :---: | :---: |  
| kubectl | 1.27.5 |
| Helm | 3.12.3 |
| step | 0.24.4 |
| Podman | - |
| ca-certificates | - |
| Istio | 1.19.1 |

| Istio 버전 | [Kubernetes 지원 버전](https://istio.io/latest/docs/releases/supported-releases/#support-status-of-istio-releases)|  
| :---: | :---: |  
| 1.19.1 |1.25, 1.26, 1.27, 1.28|  

<br>

### <span id='2.2'>2.2. 방화벽 정보
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

## <span id='3'>3. Istio 멀티 클러스터 설치

### <span id='3.1'>3.1. Deployment 파일 다운로드
Istio 멀티 클러스터 설치를 위해 컨테이너 플랫폼 포털 Deployment 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>

+ 컨테이너 플랫폼 포털 Deployment 파일 다운로드 :
   [cp-portal-deployment-v1.5.0.tar.gz](https://nextcloud.k-paas.org/index.php/s/SSo9H3qjLsFn3ob/download)

```bash
# Deployment 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/container-platform
$ cd ~/workspace/container-platform

# Deployment 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.k-paas.org/index.php/s/SSo9H3qjLsFn3ob/download

$ ls ~/workspace/container-platform
  cp-portal-deployment-v1.5.0.tar.gz

# Deployment 파일 압축 해제
$ tar -xvf cp-portal-deployment-v1.5.0.tar.gz
```

<br>

### <span id='3.2'>3.2. 도구 설치
> [[2.1. 설치 목록]](#2.1) <br>

Istio 멀티 클러스터 설치를 위해 필요한 커맨드 라인 등 도구의 사전 설치가 필요하다.<br>
`kubectl`의 경우 대상 클러스터의 마이너(minor) 버전 차이 내에 있는 버전 사용이 필요하므로 클러스터 버전을 확인하여<br>
버전 변경이 필요한 경우 아래파일을 수정한다.
```bash
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc
$ vi istio-vars-mc.sh
```
```bash
# 설치할 버전으로 변경 
# TOOL
KUBECTL_VERSION="1.27.5"
HELM_VERSION="3.12.3"
STEP_VERSION="0.24.4"

# ISTIO
ISTIO_VERSION="1.19.1"
```

도구 설치 스크립트를 실행한다.
```bash
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc
$ chmod +x install_tools.sh
$ ./install_tools.sh
```

<br>

### <span id='3.3'>3.3. 멀티 클러스터 접근 구성
Istio 멀티 클러스터를 설치할 클러스터 cluster1, cluster2에 접근할 수 있도록 컨텍스트 구성이 필요하다. <br>
:loudspeaker: cluster1, cluster2 두 kubeconfig 파일 내 cluster, context, user 명이 중복되지 않는지 확인한다.
```bash
# .kube 디렉터리 생성
$ mkdir -p ${HOME}/.kube

# cluster1, cluster2 kubeconfig 파일 위치
$ ls ${HOME}/.kube
cluster1-config  cluster2-config

# kubeconfig 파일 경로 설정
$ export KUBECONFIG="${HOME}/.kube/cluster1-config:${HOME}/.kube/cluster2-config"
```
- 컨텍스트 목록 조회 정상 확인
```bash
$ kubectl config get-contexts
CURRENT   NAME    CLUSTER    AUTHINFO         NAMESPACE
*         ctx-1   cluster1   cluster1-admin
          ctx-2   cluster2   cluster2-admin
```
- 클러스터 접근 정상 확인
```bash
# cluster1 노드 조회 
$ kubectl get nodes --context=ctx-1
NAME                                  STATUS   ROLES    AGE     VERSION
k8s-cluster-1-default-worker-node-0   Ready    <none>   5h44m   v1.27.3

# cluster2 노드 조회 
$ kubectl get nodes --context=ctx-2
NAME                                  STATUS   ROLES    AGE     VERSION
k8s-cluster-2-default-worker-node-0   Ready    <none>   5h43m   v1.27.3
```

<br>

### <span id='3.4'>3.4. Istio 멀티 클러스터 설치 변수 정의
Istio 멀티 클러스터를 설치하기 전 클러스터 컨텍스트 값 정의가 필요하다. 설정에 필요한 정보를 확인하여 변수를 설정한다. 

```bash
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc
$ vi istio-vars-mc.sh
```
```bash                                                     
# COMMON VARIABLE (Please change the value of the variables below.)
CLUSTER1_CONFIG[CTX]="{cluster1 context name}"    # cluster1 Context Name
CLUSTER2_CONFIG[CTX]="{cluster2 context name}"    # cluster2 Context Name
```

<b>CLUSTER1_CONFIG[CTX]</b><br>
클러스터 cluster1의 컨텍스트 명 입력

<b>CLUSTER2_CONFIG[CTX]</b><br>
클러스터 cluster2의 컨텍스트 명 입력

<br>

> 예시
```bash
# 컨텍스트 목록 조회
$ kubectl config get-contexts
CURRENT   NAME          CLUSTER    AUTHINFO         NAMESPACE
*         ctx-1 (입력)  cluster1   cluster1-admin
          ctx-2 (입력)  cluster2   cluster2-admin

# 컨텍스트 명 입력
CLUSTER1_CONFIG[CTX]="ctx-1"
CLUSTER2_CONFIG[CTX]="ctx-2"
```

<br>

### <span id='3.5'>3.5. Istio 멀티 클러스터 설치 스크립트 실행
Istio 멀티 클러스터를 설치를 위한 스크립트를 실행한다.
```bash
$ chmod +x deploy-istio-mc.sh
$ ./deploy-istio-mc.sh
```
```bash
Istio 1.19.1 Download Complete!
...
Your certificate has been saved in certs/root-cert.pem.
Your private key has been saved in certs/root-ca.key.
[Install Istio in cluster1]...
Your certificate has been saved in certs/cluster1/ca-cert.pem.
Your private key has been saved in certs/cluster1/ca-key.pem.
namespace/istio-system created
secret/cacerts created
✔ Istio core installed
✔ Istiod installed
✔ Installation complete
Made this installation the default for injection and validation.
✔ Ingress gateways installed
✔ Installation complete
gateway.networking.istio.io/cross-network-gateway created
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
[Install Istio in cluster2]...
Your certificate has been saved in certs/cluster2/ca-cert.pem.
Your private key has been saved in certs/cluster2/ca-key.pem.
namespace/istio-system created
secret/cacerts created
✔ Istio core installed
✔ Istiod installed
✔ Installation complete
Made this installation the default for injection and validation.
✔ Ingress gateways installed
✔ Installation complete
gateway.networking.istio.io/cross-network-gateway created
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
secret/istio-remote-secret-cluster1 created
secret/istio-remote-secret-cluster2 created

--------------------------------------------------------------
[cluster1 (ctx-1)] $ istioctl remote-clusters
--------------------------------------------------------------
NAME         SECRET                                        STATUS      ISTIOD
cluster2     istio-system/istio-remote-secret-cluster2     syncing     istiod-6fbbdd4794-x284j

--------------------------------------------------------------
[cluster2 (ctx-2)] $ istioctl remote-clusters
--------------------------------------------------------------
NAME         SECRET                                        STATUS     ISTIOD
cluster1     istio-system/istio-remote-secret-cluster1     synced     istiod-5fc66dc77-q9nxk
```
<br>

#### Istio 리소스 배포 현황
> 클러스터 컨텍스트 명 환경변수 설정 
```bash
$ source ~/workspace/container-platform/cp-portal-deployment/istio_mc/istio-vars-mc.sh
```
> cluster1의 Istio 리소스 배포 현황 확인
```bash
$ kubectl get all -n ${ISTIO_NAMESPACE} --context=${CLUSTER1_CONFIG[CTX]}
NAME                                        READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-6b9585db67-g4kb7   1/1     Running   0          15m
pod/istiod-6fbbdd4794-x284j                 1/1     Running   0          15m

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                                                                                                      AGE
service/istio-ingressgateway   LoadBalancer   198.19.183.236   xxx.xxx.xxx.xxx   15021:31376/TCP,80:31611/TCP,443:32702/TCP,31400:32264/TCP,15443:32551/TCP,15012:32101/TCP,15017:30217/TCP   15m
service/istiod                 ClusterIP      198.19.144.155   <none>            15010/TCP,15012/TCP,443/TCP,15014/TCP                                                                        15m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   1/1     1            1           15m
deployment.apps/istiod                 1/1     1            1           15m

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-ingressgateway-6b9585db67   1         1         1       15m
replicaset.apps/istiod-6fbbdd4794                 1         1         1       15m

NAME                                                       REFERENCE                         TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   3%/80%    1         5         1          15m
```
> cluster2의 Istio 리소스 배포 현황 확인
```bash
$ kubectl get all -n ${ISTIO_NAMESPACE} --context=${CLUSTER2_CONFIG[CTX]}
NAME                                        READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-5865966d59-sc9nt   1/1     Running   0          16m
pod/istiod-5fc66dc77-q9nxk                  1/1     Running   0          16m

NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                                                                                                      AGE
service/istio-ingressgateway   LoadBalancer   10.254.154.61   xxx.xxx.xxx.xxx   15021:32740/TCP,80:32192/TCP,443:31917/TCP,31400:31048/TCP,15443:32579/TCP,15012:31579/TCP,15017:31694/TCP   16m
service/istiod                 ClusterIP      10.254.12.197   <none>            15010/TCP,15012/TCP,443/TCP,15014/TCP                                                                        16m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   1/1     1            1           16m
deployment.apps/istiod                 1/1     1            1           16m

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-ingressgateway-5865966d59   1         1         1       16m
replicaset.apps/istiod-5fc66dc77                  1         1         1       16m

NAME                                                       REFERENCE                         TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   3%/80%    1         5         1          16m
```

<br> 

### <span id='3.6'>3.6. (참고) Istio 멀티 클러스터 삭제
설치된 Istio 멀티 클러스터 구성의 삭제를 원하는 경우 아래 스크립트를 실행한다.<br>
:loudspeaker: (주의) 해당 스크립트 실행 시, **Istio 멀티클러스터 구성이 모두 제거**되므로 주의가 필요하다.<br>

```bash
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc
$ chmod +x uninstall-istio-mc.sh
$ ./uninstall-istio-mc.sh
```

<br>

## <span id='4'>4. 샘플 애플리케이션 배포
클러스터 cluster1, cluster2에 멀티 클러스터 통신 샘플 애플리케이션 배포를 진행한다.

#### 애플리케이션 배포 위치 
| 애플리케이션| cluster1 | cluster2 | 정보 |
| :---: |:---: | :---: | :---: |  
| helloworld-v1 | :heavy_check_mark: | | 해당 버전<b>(v1)</b>과 호스트 명을 반환 |
| helloworld-v2 |  | :heavy_check_mark: | 해당 버전<b>(v2)</b>과 호스트 명을 반환 |
| sleep |  | :heavy_check_mark: | 다른 서비스를 호출하기 위한 요청 소스로 사용|


```bash
# 클러스터 컨텍스트 명 환경변수 설정
$ source ~/workspace/container-platform/cp-portal-deployment/istio_mc/istio-vars-mc.sh

# sample 디렉토리 생성 
$ mkdir -p ~/workspace/container-platform/cp-portal-deployment/istio_mc/sample
$ cd ~/workspace/container-platform/cp-portal-deployment/istio_mc/sample

# helloworld 서비스 배포 스크립트 다운로드
$ wget https://raw.githubusercontent.com/istio/istio/master/samples/helloworld/gen-helloworld.sh
$ chmod +x gen-helloworld.sh

# sleep 서비스 배포 yaml 다운로드
$ wget https://raw.githubusercontent.com/istio/istio/master/samples/sleep/sleep.yaml

# cluster1에 sample 네임스페이스 생성 및 Istio 사이드카 프록시 자동 주입 활성화
$ kubectl create namespace sample --context="${CLUSTER1_CONFIG[CTX]}"
$ kubectl label namespace sample istio-injection=enabled --context="${CLUSTER1_CONFIG[CTX]}"

# cluster2에 sample 네임스페이스 생성 및 Istio 사이드카 프록시 자동 주입 활성화
$ kubectl create namespace sample --context="${CLUSTER2_CONFIG[CTX]}"
$ kubectl label namespace sample istio-injection=enabled --context="${CLUSTER2_CONFIG[CTX]}"
```

<br>

### <span id='4.1'>4.1. 클러스터 cluster1에 샘플 애플리케이션 배포
```bash
# cluster1에 helloworld-v1 애플리케이션 배포 
$ ./gen-helloworld.sh --version v1 --includeService true --includeDeployment true  | \
    kubectl --context="${CLUSTER1_CONFIG[CTX]}" -n sample apply -f -
```
```bash
# helloworld-v1 서비스 리소스 생성
service/helloworld created
deployment.apps/helloworld-v1 created
```
<br>

### <span id='4.2'>4.2. 클러스터 cluster2에 샘플 애플리케이션 배포
```bash
# cluster2에 helloworld-v2 애플리케이션 배포 
$ ./gen-helloworld.sh --version v2 --includeService true --includeDeployment true  | \
    kubectl --context="${CLUSTER2_CONFIG[CTX]}" -n sample apply -f -

# cluster2에 sleep 애플리케이션 배포
$ kubectl --context="${CLUSTER2_CONFIG[CTX]}" -n sample apply -f sleep.yaml
```
```bash
# helloworld-v2 서비스 리소스 생성
service/helloworld created
deployment.apps/helloworld-v2 created

# sleep 서비스 리소스 생성
serviceaccount/sleep created
service/sleep created
deployment.apps/sleep created
```
<br>

### <span id='4.3'>4.3. 멀티 클러스터 통신 테스트 
```bash
# cluster1의 helloword-v1 pod 조회
$ kubectl get pods -l app=helloworld --context="${CLUSTER1_CONFIG[CTX]}" -n sample
NAME                           READY   STATUS    RESTARTS   AGE
helloworld-v1-d666d896-krkqg   2/2     Running   0          100s

# cluster2의 helloworld-v2 pod 조회
$ kubectl get pods -l app=helloworld --context="${CLUSTER2_CONFIG[CTX]}" -n sample
NAME                            READY   STATUS    RESTARTS   AGE
helloworld-v2-f58d4c8cb-jkjnw   2/2     Running   0          93s

# cluster2의 sleep pod 조회
$ kubectl get pods -l app=sleep --context="${CLUSTER2_CONFIG[CTX]}" -n sample
NAME                    READY   STATUS    RESTARTS   AGE
sleep-9454cc476-6vlqt   2/2     Running   0          2m17s
```

```bash
# cluster2의 sleep pod에 접속 
$ kubectl exec -it -n sample --context="${CLUSTER2_CONFIG[CTX]}" \
"$(kubectl get pod -n sample --context="${CLUSTER2_CONFIG[CTX]}" -l app=sleep -o jsonpath='{.items[0].metadata.name}')" -c sleep sh

# helloworld 서비스 curl 통신
~ $ curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-d666d896-krkqg  (버전 v1, cluster1의 helloword-v1 pod명 출력 확인)
~ $ curl helloworld:5000/hello
Hello version: v2, instance: helloworld-v2-f58d4c8cb-jkjnw (버전 v2, cluster2의 helloword-v2 pod명 출력 확인)
~ $ curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-d666d896-krkqg
~ $ curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-d666d896-krkqg
~ $ curl helloworld:5000/hello
Hello version: v2, instance: helloworld-v2-f58d4c8cb-jkjnw
~ $ curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-d666d896-krkqg
~ $ curl helloworld:5000/hello
Hello version: v2, instance: helloworld-v2-f58d4c8cb-jkjnw
```
<br>

## <span id='5'>5. (참고) 컨테이너 플랫폼 포털 배포 시 사전 설정
해당 가이드를 통해 구성된 Istio 멀티 클러스터 환경에 컨테이너 플랫폼 포털 멀티 클러스터 (단독/서비스) 형 을 배포할 경우 아래 사전 설정이 필요하다.
### <span id='5.1'>5.1. StorageClass 설정
각 Kubernetes 클러스터에 `StorageClass`가 설정되어 있는지 확인한다.
> StorageClass를 제공하지 않는다면 별도 설치 필요 
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

### <span id='5.2'>5.2. Ingress NGINX Controller 배포
컨테이너 플랫폼 포털은 Kubernetes 리소스 Ingress를 통해 각 서비스를 라우팅하며 Ingress NGINX Controller를 기본 IngressClass로 사용한다.<br>
`ingress-nginx`가 사전에 배포되어 있는지 확인하고 미설치 시 아래 명령어를 통해 설치한다.

> 클러스터 cluster1, cluster2에 Ingress NGINX Controller 배포
```bash
# Helm으로 cluster1에 Ingress NGINX Controller 배포 
$ helm upgrade --install ingress-nginx ingress-nginx \
    --repo https://kubernetes.github.io/ingress-nginx \
    --namespace ingress-nginx --create-namespace \
    --kube-context ${CLUSTER1_CONFIG[CTX]}

# Helm으로 cluster2에 Ingress NGINX Controller 배포 
$ helm upgrade --install ingress-nginx ingress-nginx \
    --repo https://kubernetes.github.io/ingress-nginx \
    --namespace ingress-nginx --create-namespace \
    --kube-context ${CLUSTER2_CONFIG[CTX]}
```

<br>
