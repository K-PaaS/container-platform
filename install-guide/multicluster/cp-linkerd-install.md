### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 Linkerd 멀티 클러스터 설치 가이드

<br>

## Table of Contents

1. [문서 개요](#1)<br>
  1.1. [목적](#1.1)<br>
  1.2. [범위](#1.2)<br>
  1.3. [시스템 구성도](#1.3)<br>
  1.4. [참고자료](#1.4)

2. [Prerequisite](#2)<br>
  2.1. [컨텍스트 설정](#2.1)<br>
  2.2. [Helm 확인](#2.2)<br>
  2.3. [MetalLB 확인](#2.3)<br>
  2.4. [Step CLI 설치](#2.4)<br>
  2.5. [Linkerd CLI 설치](#2.5)

3. [인증서 생성](#3)

4. [Linkerd 설치](#4)<br>
  4.1. [Linkerd-crds 설치](#4.1)<br>
  4.2. [Linkerd-control-plane 설치](#4.2)<br>
  4.3. [Linkerd-viz 설치](#4.3)<br>
  4.4. [Linkerd-viz Dashboard 접속](#4.4)

5. [Linkerd multi-cluster 구성](#5)<br>
  5.1. [Linkerd-multicluster 설치](#5.1)<br>
  5.2. [Linkerd-smi 설치](#5.2)

6. [Sample App 배포](#6)

7. [이슈사항](#7)

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (K-PaaS 컨테이너 플랫폼 Linkerd 멀티 클러스터 설치 가이드) 는 기획자, 개발자, 운영자 지원 환경의 개방형 PaaS 플랫폼인 K-PaaS 컨테이너 플랫폼의 클러스터에 Linkerd를 설치하여 `멀티 클러스터`를 구성하는 방법을 기술하였다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 K-PaaS 컨테이너 플랫폼 환경의 기반이 되는 클러스터 설치를 `멀티 클라우드` 환경 기준으로 설치한 후 Linkerd를 이용하여 `멀티 클러스터`를 구성하도록 작성하였다.

<br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 쿠버네티스 `멀티 클러스터` (Control Plane, Worker) 환경으로 구성되어 있다.

K-PaaS 컨테이너 플랫폼 Deployment를 통해 쿠버네티스 `단일 클러스터` 2개를 각각 구성하고 Linkerd를 설치하여 `멀티 클러스터` 환경을 구성한다.

<br>

### <div id='1.4'> 1.4. 참고자료
> https://linkerd.io<br>
> https://github.com/linkerd/linkerd2

<br>

## <div id='2'> 2. Prerequisite
본 가이드 (K-PaaS 컨테이너 플랫폼 Linkerd 멀티 클러스터 설치 가이드)는 K-PaaS 컨테이너 플랫폼 클러스터 2개(`cluster1`, `cluster2`)에 Linkerd 멀티 클러스터 서비스 메시를 구성한다.

Linkerd Control Plane 및 Extensions 설치는 클러스터 `cluster1` 에서 작업을 진행한다. 진행하기 전 아래 사항이 미리 구성되어야 한다.

<br>

### K-PaaS 컨테이너 플랫폼 클러스터
K-PaaS 컨테이너 플랫폼 클러스터 설치는 아래 가이드를 참고한다.

> [K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드](https://github.com/K-PaaS/container-platform/blob/master/install-guide/standalone/cp-cluster-install-single.md)

<br>

### Linkerd 버전
> [Linkerd Extension List](https://linkerd.io/2.13/reference/extension-list/#)

|NAME|NAMESPACE|CHART VERSION|APP VERSION|Extensions|
|---|---|---|---|---|
|linkerd-crds|linkerd|linkerd-crds-1.6.1|-|-|
|linkerd-control-plane|linkerd|linkerd-control-plane-1.12.5| stable-2.13.5|-|
|linkerd-viz|linkerd-viz|linkerd-viz-30.8.5|stable-2.13.5|O|
|linkerd-multicluster|linkerd-multicluster|linkerd-multicluster-30.7.5|stable-2.13.5|O|
|linkerd-smi|linkerd-smi|linkerd-smi-1.0.1 |v0.2.1|O|

<br>

### 방화벽 설정
|프로토콜|포트|비고|
|---|---|---|
|TCP|4191|Linkerd|
|TCP|5000|Linkerd Sample|
|TCP|4143|Linkerd Multi Cluster Gateway|

<br>


### <div id='2.1'> 2.1. 컨텍스트 설정
클러스터 `cluster1`에서 `cluster1`, `cluster2` 컨텍스트 설정을 확인한다.
```bash
## cluster api server 주소는 외부에서 접근가능해야한다. https://127.0.0.1:6443 안됨
$ kubectl config view
```

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.100.0.172:6443
  name: cluster1
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.100.0.122:6443
  name: cluster2
...
```

```bash
$ kubectl config get-contexts
CURRENT   NAME    CLUSTER    AUTHINFO         NAMESPACE
*         ctx-1   cluster1   cluster1-admin
          ctx-2   cluster2   cluster2-admin
```

<br>

### <div id='2.2'> 2.2. Helm 확인
K-PaaS 컨테이너 플랫폼 클러스터 배포에서 Helm 설치를 기본으로 제공한다.
```bash
$ helm version
version.BuildInfo{Version:"v3.9.4", GitCommit:"dbc6d8e20fe1d58d50e6ed30f09a04a77e4c68db", GitTreeState:"clean", GoVersion:"go1.17.13"}
```

<br>

### <div id='2.3'> 2.3. MetalLB 확인
K-PaaS 컨테이너 플랫폼 클러스터 배포에서 MetalLB 설치를 기본으로 제공한다.<br>
클러스터 `cluster1`, `cluster2`에서 MetalLB 설치를 확인한다.
```bash
$ kubectl get pods -n metallb-system --context=ctx-1
NAME                          READY   STATUS    RESTARTS   AGE
controller-6c58495cbb-46q4z   1/1     Running   0          18d
speaker-9jhfq                 1/1     Running   0          18d
speaker-zq55b                 1/1     Running   0          18d

$ kubectl get pods -n metallb-system --context=ctx-2
NAME                          READY   STATUS    RESTARTS   AGE
controller-6c58495cbb-ngxn9   1/1     Running   0          18d
speaker-gbkv7                 1/1     Running   0          18d
speaker-whkjr                 1/1     Running   0          18d
```

<br>

### <div id='2.4'> 2.4. Step CLI 설치
클러스터 `cluster1`에서 인증서와 키 생성을 위해 Step CLI를 설치한다.
```bash
## step 다운로드 및 /usr/bin 배치
$ wget https://github.com/smallstep/cli/releases/download/v0.24.4/step_linux_0.24.4_amd64.tar.gz -O step.tar.gz
$ tar -xvzf step.tar.gz
$ sudo mv step_0.24.4/bin/step /usr/bin/
$ sudo chmod +x /usr/bin/step

$ step version
Smallstep CLI/0.24.4 (linux/amd64)
Release Date: 2023-05-11T19:52:34Z
```

<br>

### <div id='2.5'> 2.5. Linkerd CLI 설치 
클러스터 `cluster1`에서 Linkerd CLI 설치를 진행한다.
```bash
## linkerd cli 수동설치
$ curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh

## 설치 중
Download complete!

Validating checksum...
Checksum valid.

Linkerd stable-2.13.5 was successfully installed 🎉

## 설치 완료
$ sudo mv $HOME/.linkerd2/bin/linkerd /usr/bin/
$ sudo chmod +x /usr/bin/linkerd

$ linkerd version
Client version: stable-2.13.5
Server version: unavailable
```

<br>

## <div id='3'> 3. 인증서 생성
Pod 간 mTLS 통신을 지원하기 위해 Linkerd는 trust anchor 인증서와 해당키의 issue 인증서가 필요하다.<br>
Helm을 통한 Linkerd 설치의 경우 사용자가 수동으로 생성해야 한다.
```bash
## 디렉토리 생성
$ mkdir -p $HOME/linkerd/certs
$ cd $HOME/linkerd/certs

## 루트 인증서 및 키 생성
$ step certificate create root.linkerd.cluster.local ca.crt ca.key \
--profile root-ca --no-password --insecure

## 중간 인증서 및 키 생성
$ step certificate create identity.linkerd.cluster.local issuer.crt issuer.key \
--profile intermediate-ca --not-after 8760h --no-password --insecure \
--ca ca.crt --ca-key ca.key
```

<br>

## <div id='4'> 4. Linkerd 설치

### <div id='4.1'> 4.1. Linkerd-crds 설치
Helm을 통해 Linkerd CRDS를 설치한다.
```bash
## linkerd 레파지토리 등록
$ helm repo add linkerd https://helm.linkerd.io/stable

$ helm repo list
NAME    URL
linkerd https://helm.linkerd.io/stable

## cluster1(ctx-1)에 linkerd-crds 설치
$ helm install linkerd-crds linkerd/linkerd-crds -n linkerd --create-namespace --kube-context=ctx-1

## cluster2(ctx-2)에 linkerd-crds 설치
$ helm install linkerd-crds linkerd/linkerd-crds -n linkerd --create-namespace --kube-context=ctx-2
```

<br>

### <div id='4.2'> 4.2.  Linkerd-control-plane 설치
[3. 인증서 생성](#3) 과정에서 생성한 인증서과 함께 linkerd-control-plane을 설치한다.
```bash
## cluster1(ctx-1)에 linkerd-control-plane 설치
$ helm install linkerd-control-plane -n linkerd \
  --set-file identityTrustAnchorsPEM=ca.crt \
  --set-file identity.issuer.tls.crtPEM=issuer.crt \
  --set-file identity.issuer.tls.keyPEM=issuer.key \
  linkerd/linkerd-control-plane --kube-context=ctx-1

## cluster2(ctx-2)에 linkerd-control-plane 설치
$ helm install linkerd-control-plane -n linkerd \
  --set-file identityTrustAnchorsPEM=ca.crt \
  --set-file identity.issuer.tls.crtPEM=issuer.crt \
  --set-file identity.issuer.tls.keyPEM=issuer.key \
  linkerd/linkerd-control-plane --kube-context=ctx-2
```

<br>

### <div id='4.3'> 4.3. Linkerd-viz 설치
linkerd-viz Dashboard 를 설치한다.
```bash
## cluster1(ctx-1)에 linkerd-viz 설치
$ helm install linkerd-viz -n linkerd-viz --create-namespace linkerd/linkerd-viz --kube-context=ctx-1

## cluster2(ctx-2)에 linkerd-viz 설치
$ helm install linkerd-viz -n linkerd-viz --create-namespace linkerd/linkerd-viz --kube-context=ctx-2
```

<br>

### <div id='4.4'> 4.4. Linkerd-viz Dashboard 접속
Linkerd-viz Dashboard 접속을 위해 Ingress를 생성한다.
```bash
## 디렉토리 생성
$ mkdir -p $HOME/linkerd/yaml
$ cd $HOME/linkerd/yaml

$ vi linkerd-viz-ingress.yaml
```

```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: web-ingress-auth
  namespace: linkerd-viz
data:
  auth: YWRtaW46JGFwcjEkbjdDdTZnSGwkRTQ3b2dmN0NPOE5SWWpFakJPa1dNLgoK
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  namespace: linkerd-viz
  annotations:
    nginx.ingress.kubernetes.io/upstream-vhost: $service_name.$namespace.svc.cluster.local:8084
    nginx.ingress.kubernetes.io/configuration-snippet: |
      proxy_set_header Origin "";
      proxy_hide_header l5d-remote-ip;
      proxy_hide_header l5d-server-id;
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: web-ingress-auth
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required'
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 8084
```

```bash
## cluster1(ctx-1)에 ingress 생성
$ kubectl apply -f linkerd-viz-ingress.yaml --context=ctx-1

## cluster2(ctx-2)에 ingress 생성
$ kubectl apply -f linkerd-viz-ingress.yaml --context=ctx-2
```

<br>

ingress-nginx-controller 서비스의 NodePort를 통해 linkerd viz dashboard 에 접속한다.
ex) http://{node-ip}:32699
```bash
$ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.233.37.246   <none>        80:32699/TCP,443:30590/TCP   19d
ingress-nginx-controller-admission   ClusterIP   10.233.28.54    <none>        443/TCP                      19d
```

<br>

기본 로그인 정보 (`id: admin`, `password: admin`)

![image 001]

![image 002]

<br>

## <div id='5'> 5. Linkerd Multi Cluster 구성

### <div id='5.1'> 5.1. Linkerd-multicluster 설치
Helm을 통해 Linkerd-multicluster를 설치한다.
```bash
## cluster1(ctx-1)에 linkerd-multicluster 설치
$ helm install linkerd-multicluster -n linkerd-multicluster --create-namespace linkerd/linkerd-multicluster --kube-context=ctx-1

## cluster2(ctx-2)에 linkerd-multicluster 설치
$ helm install linkerd-multicluster -n linkerd-multicluster --create-namespace linkerd/linkerd-multicluster --kube-context=ctx-2
```

<br>

자격 증명을 위한 Secret 및 미러 컨트롤러를 생성한다.
```bash
## cluster1(ctx-1) 자격증명 추출 후 cluster2(ctx-2)에 생성
$ linkerd multicluster link --context=ctx-1  --cluster-name cluster1  |  kubectl --context=ctx-2 apply -f -

## cluster2(ctx-2) 자격증명 추출 후 cluster1(ctx-1)에 생성
$ linkerd multicluster link --context=ctx-2  --cluster-name cluster2  |  kubectl --context=ctx-1 apply -f -
```

```bash
## 생성되는 리소스 목록
secret/cluster-credentials-cluster2 created
link.multicluster.linkerd.io/cluster2 created
clusterrole.rbac.authorization.k8s.io/linkerd-service-mirror-access-local-resources-cluster2 created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-service-mirror-access-local-resources-cluster2 created
role.rbac.authorization.k8s.io/linkerd-service-mirror-read-remote-creds-cluster2 created
rolebinding.rbac.authorization.k8s.io/linkerd-service-mirror-read-remote-creds-cluster2 created
serviceaccount/linkerd-service-mirror-cluster2 created
deployment.apps/linkerd-service-mirror-cluster2 created
service/probe-gateway-cluster2 created
```

<br>

Multi Cluster 통신을 위한 Gateway 서비스에 External IP 할당, 인터페이스 추가, iptable 정책을 추가한다.

```bash
$ kubectl get svc linkerd-gateway -n linkerd-multicluster --context=ctx-1

$ kubectl get svc linkerd-gateway -n linkerd-multicluster --context=ctx-2
```

> [쿠버네티스 서비스 External IP 설정](https://github.com/K-PaaS/container-platform/blob/branch/1.5.x_origin/install-guide/standalone/cp-cluster-install-single.md#2.1.6)

```bash

```

Multi Cluster 연결 상태를 확인한다.
```bash
## ctx-1 에서 ctx-1 ↔ ctx2 연결상태 확인
$ linkerd multicluster check --context=ctx-1
linkerd-multicluster
--------------------
√ Link CRD exists
√ Link resources are valid
        * cluster2
√ remote cluster access credentials are valid
        * cluster2
√ clusters share trust anchors
        * cluster2
√ service mirror controller has required permissions
        * cluster2
√ service mirror controllers are running
        * cluster2
√ probe services able to communicate with all gateway mirrors
        * cluster2
√ multicluster extension proxies are healthy
√ multicluster extension proxies are up-to-date
√ multicluster extension proxies and cli versions match

Status check results are √

## ctx-2 에서 ctx-1 ↔ ctx2 연결상태 확인
$ linkerd multicluster check --context=ctx-2
linkerd-multicluster
--------------------
√ Link CRD exists
√ Link resources are valid
        * cluster1
√ remote cluster access credentials are valid
        * cluster1
√ clusters share trust anchors
        * cluster1
√ service mirror controller has required permissions
        * cluster1
√ service mirror controllers are running
        * cluster1
√ probe services able to communicate with all gateway mirrors
        * cluster1
√ multicluster extension proxies are healthy
√ multicluster extension proxies are up-to-date
√ multicluster extension proxies and cli versions match

Status check results are √
```

<br>

### <div id='5.2'> 5.2. Linkerd-smi 설치
트래픽 분할을 위한 CRD `TrafficSplit`를 사용하기 위해선 Linkerd-smi extenstions 설치가 필요하다.

> [[Getting started with Linkerd SMI extension]](https://linkerd.io/2.13/tasks/linkerd-smi/)

<br>

```bash
## linkerd-smi extension 레파지토리 등록
$ helm repo add l5d-smi https://linkerd.github.io/linkerd-smi

$ helm repo list
NAME    URL
linkerd https://helm.linkerd.io/stable
l5d-smi https://linkerd.github.io/linkerd-smi

## cluster1(ctx-1)에 linkerd-smi 설치
$ helm install linkerd-smi l5d-smi/linkerd-smi -n linkerd-smi --create-namespace --kube-context=ctx-1

## cluster1(ctx-2)에 linkerd-smi 설치
$ helm install linkerd-smi l5d-smi/linkerd-smi -n linkerd-smi --create-namespace --kube-context=ctx-2
```

<br>


## <div id='6'> 6. Sample App 배포 
클러스터 `cluster1`, `cluster2`에 멀티 클러스터 통신 확인을 위한 샘플 앱을 배포한다.

![image 003]

<br>

### `cluster1`에 HelloWorld(v1) APP 배포
```bash
## cluster1(ctx-1)에 helloworld(v1) deployment-svc 생성
## mirror.linkerd.io/exported: "true" > 미러링을 통한 서비스 내보내기
## linkerd.io/inject: enabled > pod에 linkerd-proxy 주입
cat << EOF | kubectl apply --context=ctx-1 -f -
apiVersion: v1
kind: Service
metadata:
  name: helloworld
  labels:
    app: helloworld
    service: helloworld
    mirror.linkerd.io/exported: "true"
spec:
  ports:
  - port: 5000
    name: http
  selector:
    app: helloworld
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-v1
  labels:
    app: helloworld
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
      version: v1
  template:
    metadata:
      labels:
        app: helloworld
        version: v1
      annotations:
        linkerd.io/inject: enabled
    spec:
      containers:
      - name: helloworld
        image: docker.io/istio/examples-helloworld-v1
        resources:
          requests:
            cpu: "100m"
        imagePullPolicy: IfNotPresent #Always
        ports:
        - containerPort: 5000
EOF
```

<br>

`cluster1` 내 HelloWorld(v1) 배포를 확인한다.
```bash
$ kubectl get all -l app=helloworld --context=ctx-1
NAME                                READY   STATUS    RESTARTS   AGE
pod/helloworld-v1-994b78d57-vp86w   2/2     Running   0          26s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/helloworld   ClusterIP   10.233.11.77   <none>        5000/TCP   26s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helloworld-v1   1/1     1            1           26s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/helloworld-v1-994b78d57   1         1         1       26
```

<br>

`cluster2` 내 서비스 미러링을 통한 cluster1 helloworld service 생성을 확인한다.
```bash
$ kubectl get all -l app=helloworld --context=ctx-2
NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/helloworld-cluster1   ClusterIP   10.233.20.139   <none>        5000/TCP   57s
```

<br>

### `cluster2`에 HelloWorld(v2) 배포
```bash
## cluster2(ctx-2)에 helloworld(v2) deployment-svc 생성
## linkerd.io/inject: enabled > pod에 linkerd-proxy 주입
cat << EOF | kubectl apply --context=ctx-2 -f -
apiVersion: v1
kind: Service
metadata:
  name: helloworld
  labels:
    app: helloworld
    service: helloworld
spec:
  ports:
  - port: 5000
    name: http
  selector:
    app: helloworld
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-v2
  labels:
    app: helloworld
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
      version: v2
  template:
    metadata:
      labels:
        app: helloworld
        version: v2
      annotations:
        linkerd.io/inject: enabled
    spec:
      containers:
      - name: helloworld
        image: docker.io/istio/examples-helloworld-v2
        resources:
          requests:
            cpu: "100m"
        imagePullPolicy: IfNotPresent #Always
        ports:
        - containerPort: 5000
EOF
```

<br>

### `cluster2`에 `TrafficSplit` 배포
```bash
## 서비스 helloworld-cluster1(cluster1) weight 50, 서비스 helloworld(cluster2) weight 50으로 트래픽 분할
cat << EOF | kubectl apply --context=ctx-2 -f -
apiVersion: split.smi-spec.io/v1alpha1
kind: TrafficSplit
metadata:
  name: helloworld
spec:
  service: helloworld
  backends:
  - service: helloworld
    weight: 50
  - service: helloworld-cluster1
    weight: 50
EOF
```

<br>

### `cluster2`에 Sleep APP 배포
```bash
## linkerd.io/inject: enabled > pod에 linkerd-proxy 주입
cat << EOF | kubectl apply --context=ctx-2 -f -
apiVersion: v1
kind: Service
metadata:
  name: sleep
  labels:
    app: sleep
spec:
  ports:
  - port: 80
    name: http
  selector:
    app: sleep
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sleep
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sleep
  template:
    metadata:
      labels:
        app: sleep
      annotations:
        linkerd.io/inject: enabled
    spec:
      containers:
      - name: sleep
        image: curlimages/curl
        command: ["/bin/sleep", "3650d"]
        imagePullPolicy: IfNotPresent
EOF
```

<br>

`cluster2` 내 HelloWorld(v2), TrafficSplit, Sleep 배포를 확인한다.
```bash
$ kubectl get all -l app=helloworld  --context=ctx-2
NAME                                READY   STATUS    RESTARTS   AGE
pod/helloworld-v2-8948bdcb5-z4pz6   2/2     Running   0          108s

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/helloworld            ClusterIP   10.233.3.145    <none>        5000/TCP   108s
service/helloworld-cluster1   ClusterIP   10.233.20.139   <none>        5000/TCP   18m

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helloworld-v2   1/1     1            1           108s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/helloworld-v2-8948bdcb5   1         1         1       108s
```

```bash
$ kubectl get trafficsplits.split.smi-spec.io  --context=ctx-2
NAME         SERVICE
helloworld   helloworld
```

```bash
$ kubectl get all -l app=sleep  --context=ctx-2
NAME                         READY   STATUS    RESTARTS   AGE
pod/sleep-7b88956949-mtrpf   2/2     Running   0          58s

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/sleep   ClusterIP   10.233.27.45   <none>        80/TCP    58s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/sleep-7b88956949   1         1         1       58s
```

<br>

### Sleep -> Helloworld 통신 테스트 
`TrafficSplit`을 통해 `cluster1`의 helloworld-v1, `cluster2`의 helloworld-v2로 트래픽 분할되어 통신되는 것을 확인할 수 있다.
```bash
$ kubectl exec --context=ctx-2 "$(kubectl get pod --context=ctx-2 -l app=sleep -o jsonpath='{.items[0].metadata.name}')" \
-c sleep -- curl -sS helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-994b78d57-p6dvp

## pod 접속해서 curl 통신
~ $ curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-994b78d57-vp86w
~ $ curl helloworld:5000/hello
Hello version: v2, instance: helloworld-v2-8948bdcb5-z4pz6
~ $ curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-994b78d57-vp86w
~ $ curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-994b78d57-vp86w
~ $ curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-994b78d57-vp86w
~ $ curl helloworld:5000/hello
Hello version: v2, instance: helloworld-v2-8948bdcb5-z4pz6
```

<br>

Linkerd viz stat service 상태를 확인한다.
```bash
$ linkerd viz stat services
NAME                  MESHED   SUCCESS      RPS   LATENCY_P50   LATENCY_P95   LATENCY_P99   TCP_CONN
helloworld                 -   100.00%   0.1rps           0ms           0ms           0ms          0
helloworld-cluster1        -   100.00%   0.1rps           0ms           0ms           0ms          0
```

<br>

Linkerd Dashboard Service 상태를 확인한다.

![image 004]

<br>

## <div id='7'> 7. 이슈사항
Linkerd viz 에서 CRD `TrafficSplit`을 관리하지 하지 않아 Dashboard에서 `TrafficSplit` 확인이 불가하다.

> [Linkerd cli viz](https://linkerd.io/2.13/reference/cli/viz/#stat)<br>
> [linkerd viz stat for traffic split invalid](https://github.com/linkerd/linkerd2/issues/10990)

```
So, I see, the SMI is dead, in favor of the Gateway API
Anyway, the docs need some updates and cleanup ;-). I'll keep this issue open, therefore.
```
```
 Valid resource types include:
  * cronjobs
  * daemonsets
  * deployments
  * namespaces
  * jobs
  * pods
  * replicasets
  * replicationcontrollers
  * statefulsets
  * authorities (not supported in --from)
  * authorizationpolicies (not supported in --from)
  * httproutes (not supported in --from)
  * services (not supported in --from)
  * servers (not supported in --from)
  * serverauthorizations (not supported in --from)
  * all (all resource types, not supported in --from or --to)
```

<br>

[image 001]:images/kpaas-cp-linkerd-1.png
[image 002]:images/kpaas-cp-linkerd-2.png
[image 003]:images/kpaas-cp-linkerd-3.png
[image 004]:images/kpaas-cp-linkerd-4.png

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > K-PaaS 컨테이너 플랫폼 Linkerd 멀티 클러스터 설치 가이드