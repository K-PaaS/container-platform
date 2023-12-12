### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 Karmada 설치 가이드

<br>

## Table of Contents

1. [문서 개요](#1)<br>
  1.1. [목적](#1.1)<br>
  1.2. [범위](#1.2)<br>
  1.3. [시스템 구성도](#1.3)<br>
  1.4. [참고 자료](#1.4) 

2. [Prerequisite](#2)

3. [Karmada 설치](#3)<br>
  3.1. [Karmadactl 설치](#3.1)<br>
  3.1.1. [Karmada 최신 릴리즈 다운로드](#3.1.1)<br>
  3.1.2. [Karmada Control Plane 설치](#3.1.2)<br>
  3.1.3. [Karmada Context 설정](#3.1.3)<br>
  3.2. [Member 클러스터 설정](#3.2)<br>
  3.2.1. [Config 파일 설정](#3.2.1)<br>
  3.2.2. [Member 클러스터 등록](#3.2.2)

4. [샘플 애플리케이션 배포](#4)<br>
  4.1. [PropagationPolicy 정책 생성 및 배포](#4.1)<br>
  4.2. [샘플 yaml 생성](#4.2)<br>
  4.3. [샘플 애플리케이션 확인](#4.3)

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (K-PaaS 컨테이너 플랫폼 Karmada 설치 가이드) 는 기획자, 개발자, 운영자 지원 환경의 개방형 PaaS 플랫폼인 K-PaaS 컨테이너 플랫폼의 클러스터에 Karmada를 설치하여 `멀티 클러스터`를 구성하는 방법을 기술하였다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 K-PaaS 컨테이너 플랫폼 환경의 기반이 되는 클러스터 설치를 `멀티 클라우드` 환경 기준으로 설치한 후 Karmada를 이용하여 `멀티 클러스터`를 구성하도록 작성하였다.

<br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 쿠버네티스 3개의 `멀티 클러스터` (Control Plane, Worker) 환경으로 구성되어 있다. (2개의 멀티 클러스터, 1개의 Karmada Host 클러스터)

K-PaaS 컨테이너 플랫폼 Deployment를 통해 쿠버네티스 `멀티 클러스터`를 구성하고 추가로 Karmada Host 클러스터 용도로 `단일 클러스터`를 구성한다.

<br>

### <div id='1.4'> 1.4. 참고 자료
> https://karmada.io/<br>
> https://github.com/karmada-io/karmada

<br>

## <div id='2'> 2. Prerequisite

### K-PaaS 컨테이너 플랫폼 클러스터
K-PaaS 컨테이너 플랫폼 클러스터 설치는 아래 가이드를 참고한다.

> [K-PaaS 컨테이너 플랫폼 멀티 클러스터 설치 가이드](https://github.com/K-PaaS/container-platform/blob/master/install-guide/standalone/cp-cluster-install-multi.md)<br>
> [K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드](https://github.com/K-PaaS/container-platform/blob/master/install-guide/standalone/cp-cluster-install-single.md)

<br>

## <div id='3'> 3. Karmada 설치

### <div id='3.1'> 3.1. Karmadactl 설치

### <div id='3.1.1'> 3.1.1. Karmada 최신 릴리즈 다운로드
karmada 최신 릴리즈를 다운로드한다.
```bash
$ curl -s https://raw.githubusercontent.com/karmada-io/karmada/master/hack/install-cli.sh | sudo bash
```

<br>

### <div id='3.1.2'> 3.1.2. Karmada Control Plane 설치
Karmada Control Plane 설치를 진행한다.<br>
Karmada의 etcd는 K-PaaS 컨테이너 플랫폼의 스토리지 클래스를 사용하여 안정적인 데이터 저장을 보장한다.
```bash
$ sudo karmadactl init --etcd-storage-mode PVC --storage-classes-name {STORAGE_CLASS_NAME} --kubeconfig={KUBE_CONFIG_PATH}

## 예시
$ sudo karmadactl init --etcd-storage-mode PVC --storage-classes-name cp-storageclass --kubeconfig=/home/ubuntu/.kube/config
```

```bash
## Karmada 설치 과정
I1109 13:37:22.987256  106456 deploy.go:184] kubeconfig file: /home/ubuntu/.kube/config, kubernetes: https://127.0.0.1:6443
I1109 13:37:23.006789  106456 deploy.go:204] karmada apiserver ip: [172.16.11.106]
I1109 13:37:23.355645  106456 cert.go:229] Generate ca certificate success.
I1109 13:37:23.523217  106456 cert.go:229] Generate karmada certificate success.
I1109 13:37:23.711487  106456 cert.go:229] Generate apiserver certificate success.
I1109 13:37:24.461727  106456 cert.go:229] Generate front-proxy-ca certificate success.
I1109 13:37:25.016618  106456 cert.go:229] Generate front-proxy-client certificate success.
I1109 13:37:25.172154  106456 cert.go:229] Generate etcd-ca certificate success.
I1109 13:37:25.706451  106456 cert.go:229] Generate etcd-server certificate success.
I1109 13:37:25.887367  106456 cert.go:229] Generate etcd-client certificate success.
I1109 13:37:25.887728  106456 deploy.go:298] download crds file:https://github.com/karmada-io/karmada/releases/download/v1.7.1/crds.tar.gz
Downloading...[ 100.00% ]
	..
	..
	...skipped
	..
	..
------------------------------------------------------------------------------------------------------
 █████   ████   █████████   ███████████   ██████   ██████   █████████   ██████████     █████████
░░███   ███░   ███░░░░░███ ░░███░░░░░███ ░░██████ ██████   ███░░░░░███ ░░███░░░░███   ███░░░░░███
 ░███  ███    ░███    ░███  ░███    ░███  ░███░█████░███  ░███    ░███  ░███   ░░███ ░███    ░███
 ░███████     ░███████████  ░██████████   ░███░░███ ░███  ░███████████  ░███    ░███ ░███████████
 ░███░░███    ░███░░░░░███  ░███░░░░░███  ░███ ░░░  ░███  ░███░░░░░███  ░███    ░███ ░███░░░░░███
 ░███ ░░███   ░███    ░███  ░███    ░███  ░███      ░███  ░███    ░███  ░███    ███  ░███    ░███
 █████ ░░████ █████   █████ █████   █████ █████     █████ █████   █████ ██████████   █████   █████
░░░░░   ░░░░ ░░░░░   ░░░░░ ░░░░░   ░░░░░ ░░░░░     ░░░░░ ░░░░░   ░░░░░ ░░░░░░░░░░   ░░░░░   ░░░░░
------------------------------------------------------------------------------------------------------
Karmada is installed successfully.

Register Kubernetes cluster to Karmada control plane.

Register cluster with 'Push' mode

Step 1: Use "karmadactl join" command to register the cluster to Karmada control plane. --cluster-kubeconfig is kubeconfig of the member cluster.
(In karmada)~# MEMBER_CLUSTER_NAME=$(cat ~/.kube/config  | grep current-context | sed 's/: /\n/g'| sed '1d')
(In karmada)~# karmadactl --kubeconfig /etc/karmada/karmada-apiserver.config  join ${MEMBER_CLUSTER_NAME} --cluster-kubeconfig=$HOME/.kube/config

Step 2: Show members of karmada
(In karmada)~# kubectl --kubeconfig /etc/karmada/karmada-apiserver.config get clusters


Register cluster with 'Pull' mode

Step 1: Use "karmadactl register" command to register the cluster to Karmada control plane. "--cluster-name" is set to cluster of current-context by default.
(In member cluster)~# karmadactl register 172.16.11.106:32443 --token 1rgb7l.tdk5q1bg077t19z6 --discovery-token-ca-cert-hash sha256:551bab3fcfa57d0917d1b29a2c051c7959c53eb76326ea6f6e2471251ed2ef40

Step 2: Show members of karmada
(In karmada)~# kubectl --kubeconfig /etc/karmada/karmada-apiserver.config get clusters
```

<br>

Karmada-System 네임스페이스를 확인한다.
```bash
$ kubectl get all -n karmada-system
NAME                                                READY   STATUS    RESTARTS   AGE
pod/etcd-0                                          1/1     Running   0          3m31s
pod/karmada-aggregated-apiserver-5d7955b64b-4s4xj   1/1     Running   0          2m54s
pod/karmada-apiserver-58f66d6f76-z7sfc              1/1     Running   0          3m25s
pod/karmada-controller-manager-7cdddb5476-hv2mk     1/1     Running   0          2m30s
pod/karmada-scheduler-5555687bb-qqlmn               1/1     Running   0          2m38s
pod/karmada-webhook-8699496649-bcf92                1/1     Running   0          2m22s
pod/kube-controller-manager-675dff8bd8-zmprr        1/1     Running   0          2m43s

NAME                                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/etcd                           ClusterIP   None            <none>        2379/TCP,2380/TCP   3m31s
service/karmada-aggregated-apiserver   ClusterIP   10.233.28.84    <none>        443/TCP             2m54s
service/karmada-apiserver              NodePort    10.233.55.116   <none>        5443:32443/TCP      3m25s
service/karmada-webhook                ClusterIP   10.233.51.202   <none>        443/TCP             2m22s
service/kube-controller-manager        ClusterIP   10.233.41.78    <none>        10257/TCP           2m43s

NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/karmada-aggregated-apiserver   1/1     1            1           2m54s
deployment.apps/karmada-apiserver              1/1     1            1           3m25s
deployment.apps/karmada-controller-manager     1/1     1            1           2m30s
deployment.apps/karmada-scheduler              1/1     1            1           2m38s
deployment.apps/karmada-webhook                1/1     1            1           2m22s
deployment.apps/kube-controller-manager        1/1     1            1           2m43s

NAME                                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/karmada-aggregated-apiserver-5d7955b64b   1         1         1       2m54s
replicaset.apps/karmada-apiserver-58f66d6f76              1         1         1       3m25s
replicaset.apps/karmada-controller-manager-7cdddb5476     1         1         1       2m30s
replicaset.apps/karmada-scheduler-5555687bb               1         1         1       2m38s
replicaset.apps/karmada-webhook-8699496649                1         1         1       2m22s
replicaset.apps/kube-controller-manager-675dff8bd8        1         1         1       2m43s

NAME                    READY   AGE
statefulset.apps/etcd   1/1     3m31s
```

<br>

### <div id='3.1.3'> 3.1.3. Karmada Context 설정
클러스터의 기본 Context 외 Karmada Context를 추가 및 설정한다.
```bash
$ sudo cp /etc/karmada/karmada-apiserver.config ~/.kube
$ export KUBECONFIG=~/.kube/config:~/.kube/karmada-apiserver.config
$ kubectl config get-contexts
CURRENT   NAME                             CLUSTER             AUTHINFO        NAMESPACE
		  karmada-apiserver                karmada-apiserver   karmada-admin
*         kubernetes-admin@cluster.local   cluster.local       kubernetes-admin
```

```bash
$ kubectl config use-context karmada-apiserver 
Switched to context "karmada-apiserver".
```

<br>

### <div id='3.2'> 3.2. Member 클러스터 설정

### <div id='3.2.1'> 3.2.1, Config 파일 설정
멀티 클러스터의 Kube Config 파일을 Karmada Host 클러스터의 다음 경로에 위치시킨다.<br>
Member 클러스터의 Kube Config 파일은 각 클러스터의 `$HOME/.kube/config` 에 저장되어 있다.<br>
`server: https://127.0.0.1:6443`을 `server: https://{Master_Node_IP}:6443`로 변경 해야 한다.
```bash
$ mkdir -p $HOME/.kube/member

## Config 파일 복사 후 조회
$ ls /home/ubuntu/.kube/member
member1_config member2_config
```

<br>

member1_cofing 파일 예시
```bash
$ vi member1_config
```

```yaml
apiVersion: v1
clusters:
- cluster:
	certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJWHBOaDJRQVFLVzh3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1Wlh
  ..
  ..
  ...skipped
  ..
  ..
	server: https://103.xxx.xxx.xxx:6443
  name: cluster.local
contexts:
- context:
	cluster: cluster.local
	user: kubernetes-admin
  name: kubernetes-admin@cluster.local
current-context: kubernetes-admin@cluster.local
kind: Config
preferences: {}
..
..
...skipped
..
..
```

<br>

member2_config 파일 예시
```bash
$ vi member2_config
```

```yaml
apiVersion: v1
clusters:
- cluster:
	certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJVk94MDFEOE5IdHN3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1Wlh
 ..
 ..
 ...skipped
 ..
 ..
	server: https://104.xxx.xxx.xxx:6443
  name: cluster.local
contexts:
- context:
	cluster: cluster.local
	user: kubernetes-admin
  name: kubernetes-admin@cluster.local
current-context: kubernetes-admin@cluster.local
kind: Config
..
..
...skipped
..
..
```

<br>

#### <div id='3.2.2'> 3.2.2. Member 클러스터 등록 
복사한 Kube Config 파일을 이용하여 Karmada Host 클러스터에서 각 클러스터를 Member 클러스터로 등록한다.

<br>

Member1 Cluster를 등록한다.
```bash
$ karmadactl join member1 --cluster-kubeconfig=/home/ubuntu/.kube/member/member1_config
cluster(member1) is joined successfully
```

<br>

Member2 Cluster를 등록한다.
```bash
$ karmadactl join member2 --cluster-kubeconfig=/home/ubuntu/.kube/member/member2_config
cluster(member2) is joined successfully
```

<br>

등록한 Member Cluster를 확인한다.
```bash
$ kubectl get clusters
NAME      VERSION   MODE   READY   AGE
member1   v1.27.5   Push   True    10m
member2   v1.27.5   Push   True    4m4s
```

<br>

## <div id='4'> 4. 샘플 애플리케이션 배포

### <div id='4.1'> 4.1. PropagationPolicy 정책 생성 및 배포 
샘플 애플리케이션 확인을 위해 PropagationPolicy 정책을 생성한다.

```bash
$ vi propagationpolicy.yaml
```

```yaml
apiVersion: policy.karmada.io/v1alpha1
kind: PropagationPolicy
metadata:
name: example-policy # The default namespace is `default`.
spec:
resourceSelectors:
- apiVersion: apps/v1
  kind: Deployment
  name: nginx # If no namespace is specified, the namespace is inherited from the parent object scope.
placement:
clusterAffinity:
  clusterNames:
	- member1
	- member2
```

```bash
$ kubectl apply -f propagationpolicy.yaml
propagationpolicy.policy.karmada.io/example-policy created
```

<br>

### <div id='4.2'> 4.2. 샘플 yaml 생성
샘플 애플리케이션을 배포한다.
```bash
$ vi nginx.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: nginx
labels:
app: nginx
spec:
replicas: 2
selector:
matchLabels:
  app: nginx
template:
metadata:
  labels:
	app: nginx
spec:
  containers:
  - image: nginx
	name: nginx
```

```bash
$kubectl apply -f nginx.yaml
deployment.apps/nginx created
```

<br>

### <div id='4.3'> 4.3. 샘플 애플리케이션 확인
샘플 애플리케이션이 Member 클러스터에 정상 배포되었는지 확인한다.

```bash
$ karmadactl get deployment -l app=nginx
NAME    CLUSTER   READY   UP-TO-DATE   AVAILABLE   AGE   ADOPTION
nginx   member1   2/2     2            2           12s   Y
nginx   member2   2/2     2            2           12s   Y
```

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 Karmada 설치 가이드