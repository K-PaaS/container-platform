### [Index](https://github.com/K-PAAS/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > CSP 쿠버네티스 서비스 Linkerd 멀티 클러스터 설치 가이드

<br>

## Table of Contents
1. [문서 개요](#1)<br>
   1.1. [목적](#1.1)<br>
   1.2. [범위](#1.2)<br>
   1.3. [시스템 구성도](#1.3)<br>
   1.4. [참고자료](#1.4)

2. [Prerequisite](#2)<br>
   2.1. [설치 목록](#2.1)<br>
   2.2. [방화벽 정보](#2.2)<br>
   2.3. [CLI 설치 확인](#2.3)<br>
   2.4. [멀티 클러스터 접근 구성](#2.4)<br>
   2.5. [환경 변수 설정](#2.5)<br>
   2.6. [Linkerd CLI, step 설치](#2.6)

3. [Linkerd 멀티 클러스터 설치](#3)<br>
   3.1. [인증서 생성](#3.1)<br>
   3.2. [linkerd-crds 설치](#3.2)<br>
   3.3. [linkerd-control-plane 설치](#3.3)<br>
   3.4. [linkerd-multicluster 설치](#3.4)<br>
   3.5. [linkerd-smi 설치](#3.5)

4. [샘플 애플리케이션 배포](#4)<br>
   4.1. [클러스터 cluster1에 샘플 애플리케이션 배포](#4.1)<br>
   4.2. [클러스터 cluster2에 샘플 애플리케이션 배포](#4.2)<br>
   4.3. [멀티 클러스터 통신 테스트](#4.3)

<br>

## <span id='1'> 1. 문서 개요

### <span id='1.1'> 1.1. 목적
본 문서 (CSP 쿠버네티스 서비스 Linkerd 멀티 클러스터 설치 가이드) 는 클라우드 서비스 제공업체의 Kubernetes 서비스에 Linkerd 멀티 클러스터 설치 방법을 기술하여 독립적인 두 Kubernetes 클러스터의 서비스 간 통신이 가능하도록 한다.

<br>

### <span id='1.2'> 1.2. 범위
설치 범위는 `CSP 쿠버네티스 서비스` 클러스터를 대상으로 Linkerd를 이용하여 `멀티 클러스터`를 구성하도록 작성하였다.

<br>

### <span id='1.3'> 1.3. 시스템 구성도
`CSP 쿠버네티스 서비스`의 클러스터 2개를 각각 구성하고 Linkerd를 설치하여 `멀티 클러스터` 환경을 구성한다.

<br>

### <span id='1.4'> 1.4. 참고자료
> [[Linkerd Docs]](https://linkerd.io/2.14/overview/) <br>
> [[Linkerd Installing Multi-cluster Components]](https://linkerd.io/2.14/tasks/installing-multicluster/) <br>
> [[Linkerd Multi-cluster communication]](https://linkerd.io/2.14/features/multicluster/)

<br>

## <span id='2'> 2. Prerequisite
- **두 개의 클러스터**를 대상으로 Linkerd 멀티 클러스터를 구성한다.
- 각 클러스터는 **로드 밸런서(LoadBalancer)** 유형의 서비스 지원이 필요하다.

<br>

### <span id='2.1'>2.1. 설치 목록
설치되는 도구 목록은 아래와 같다.
#### Linkerd
> [Linkerd Releases and Versions](https://linkerd.io/releases/) <br>
> [Linkerd Extension List](https://linkerd.io/2.14/reference/extension-list/)

|NAME|NAMESPACE|Extensions|
|:---|:---|:---|
|<b>[linkerd-crds](https://artifacthub.io/packages/helm/linkerd2/linkerd-crds)</b>|-|-|
|<b>[linkerd-control-plane](https://artifacthub.io/packages/helm/linkerd2/linkerd-control-plane)</b>|linkerd|-|
|<b>[linkerd-multicluster](https://artifacthub.io/packages/helm/linkerd2/linkerd-multicluster)</b>|linkerd-multicluster|O|
|<b>[linkerd-smi](https://github.com/linkerd/linkerd-smi)</b>|linkerd-smi|O|

#### step ([`v0.24.4`](https://github.com/smallstep/cli/releases/tag/v0.24.4))
> [step CLI](https://smallstep.com/docs/step-cli/index.html)


<br>

### <span id='2.2'>2.2. 방화벽 정보
IaaS Security Group의 열어줘야할 Port를 설정한다.
|프로토콜|포트|비고|
|---|---|---|
|TCP|4191|Linkerd Gateway Probe|
|TCP|4143|Linkerd Gateway|
|TCP|5000|Linkerd Sample App|

<br>

### <span id='2.3'>2.3. CLI 설치 확인
Linkerd 멀티 클러스터 설치에 필요한 CLI `kubectl`, `Helm` 설치 여부를 확인한다.
> [kubectl 설치](https://kubernetes.io/ko/docs/tasks/tools/#kubectl) <br>
> [Helm 설치](https://helm.sh/ko/docs/intro/install/)
```bash
# kubectl 버전 확인 (대상 클러스터의 마이너(minor) 버전 차이 내에 있는 kubectl 버전 사용 필요)
$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.3", ...}

# Helm 버전 확인
$ helm version
version.BuildInfo{Version:"v3.12.3", ...}
```
<br>

### <span id='2.4'>2.4. 멀티 클러스터 접근 구성
Linkerd 멀티 클러스터를 설치할 클러스터 cluster1, cluster2에 접근할 수 있도록 컨텍스트 구성이 필요하다. <br>
:loudspeaker: cluster1, cluster2 두 kubeconfig 파일 내 cluster, context, user 명이 중복되지 않는지 확인한다. <br>
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

### <span id='2.5'>2.5. 환경 변수 설정
Linkerd 멀티 클러스터 설치를 위한 클러스터 정보, CLI 등 환경 변수를 설정한다.
- 클러스터 명, 컨텍스트 정보, step 버전 설정
```bash
export CLUSTER1_CTX="{클러스터 cluster1 컨텍스트 명}"  #(e.g. ctx-1)
export CLUSTER1_NAME="{Linkerd 멀티 클러스터 연결 시 사용할 클러스터 cluster1 명}" #(e.g. cluster1)
export CLUSTER2_CTX="{클러스터 cluster2 컨텍스트 명}"  #(e.g. ctx-2)
export CLUSTER2_NAME="{Linkerd 멀티 클러스터 연결 시 사용할 클러스터 cluster2 명}" #(e.g. cluster2)
export STEP_VERSION="0.24.4"
```

<br>

### <span id='2.6'>2.6. Linkerd CLI, step 설치
Linkerd와 상호작용을 위한 Linkerd CLI, 인증서 생성을 위한 step을 설치한다.
- Linkerd CLI 설치
```bash
# Linkerd CLI 설치
$ curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh -

Download complete!

Validating checksum...
Checksum valid.

Linkerd stable-2.x was successfully installed 🎉
...
```
```bash
$ sudo mv $HOME/.linkerd2/bin/linkerd /usr/local/bin/linkerd
$ linkerd version --client
Client version: stable-2.x
```
- step 설치
```bash
# step 설치
$ wget https://dl.smallstep.com/gh-release/cli/docs-cli-install/v${STEP_VERSION}/step-cli_${STEP_VERSION}_amd64.deb
$ sudo dpkg -i step-cli_${STEP_VERSION}_amd64.deb
$ step version
Smallstep CLI/0.24.4 (linux/amd64)
Release Date: 2023-05-11T19:52:34Z
```
<br>

## <span id='3'>3. Linkerd 멀티 클러스터 설치
### <span id='3.1'>3.1. 인증서 생성
서비스 메시 형 Pod 간 [mTLS 통신](https://linkerd.io/2.14/tasks/generate-certificates/)을 지원하기 위해 Linkerd는 trust anchor 인증서와 해당 키가 포함된 issuer 인증서가 필요하다.<br>
사전에 설치한 step을 통해 인증서 생성을 진행한다.
```bash
# 디렉토리 생성 
$ mkdir -p $HOME/linkerd/certs
$ cd $HOME/linkerd/certs

# Trust anchor 인증서 생성
$ step certificate create root.linkerd.cluster.local ca.crt ca.key \
--profile root-ca --no-password --insecure

# issuer 인증서 및 키 생성
$ step certificate create identity.linkerd.cluster.local issuer.crt issuer.key \
--profile intermediate-ca --not-after 8760h --no-password --insecure \
--ca ca.crt --ca-key ca.key

# 인증서 및 키 생성 확인
$ ls
ca.crt  ca.key  issuer.crt  issuer.key
```

<br>

### <span id='3.2'>3.2. linkerd-crds 설치
Helm을 통해 `linkerd-crds`를 설치한다.
```bash
# linkerd repo 등록
$ helm repo add linkerd https://helm.linkerd.io/stable

$ helm repo list
NAME    URL
linkerd https://helm.linkerd.io/stable (등록 확인)

# cluster1에 linkerd-crds 설치
$ helm install linkerd-crds linkerd/linkerd-crds -n linkerd --create-namespace --kube-context="${CLUSTER1_CTX}"

# cluster2에 linkerd-crds 설치
$ helm install linkerd-crds linkerd/linkerd-crds -n linkerd --create-namespace --kube-context="${CLUSTER2_CTX}"
```

<br> 

### <span id='3.3'>3.3. linkerd-control-plane 설치
- [[3.1. 인증서 생성]](#3.1)에서 생성한 인증서, 키와 함께 `linkerd-control-plane`를 설치한다.

```bash
# 인증서, 키 파일 위치로 이동
$ cd $HOME/linkerd/certs

# cluster1에 linkerd-control-plane 설치
$ helm install linkerd-control-plane linkerd/linkerd-control-plane -n linkerd \
  --set-file identityTrustAnchorsPEM=ca.crt \
  --set-file identity.issuer.tls.crtPEM=issuer.crt \
  --set-file identity.issuer.tls.keyPEM=issuer.key \
  --kube-context="${CLUSTER1_CTX}"

# cluster2에 linkerd-control-plane 설치
$ helm install linkerd-control-plane linkerd/linkerd-control-plane -n linkerd \
  --set-file identityTrustAnchorsPEM=ca.crt \
  --set-file identity.issuer.tls.crtPEM=issuer.crt \
  --set-file identity.issuer.tls.keyPEM=issuer.key \
  --kube-context="${CLUSTER2_CTX}"
```
<br>

### <span id='3.4'>3.4. linkerd-multicluster 설치
`linkerd-multicluster`를 설치한다.
```bash
# cluster1에 linkerd-multicluster 설치
$ helm install linkerd-multicluster linkerd/linkerd-multicluster -n linkerd-multicluster --create-namespace \
--kube-context="${CLUSTER1_CTX}"

# cluster2에 linkerd-multicluster 설치
$ helm install linkerd-multicluster linkerd/linkerd-multicluster -n linkerd-multicluster --create-namespace \
--kube-context="${CLUSTER2_CTX}"
```
<br>

배포 후 각 클러스터 내 `linkerd-gateway` 서비스의 EXTERNAL-IP가 할당되었는지 확인한다.
```bash
# cluster1의 linkerd-gateway에 EXTERNAL-IP 할당 확인 
$ kubectl get svc -n linkerd-multicluster --context="${CLUSTER1_CTX}"
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                         AGE
linkerd-gateway   LoadBalancer   10.254.93.121   xxx.xxx.xxx.xxx   4143:30926/TCP,4191:32314/TCP   78s

# cluster2의 linkerd-gateway에 EXTERNAL-IP 할당 확인
$ kubectl get svc -n linkerd-multicluster --context="${CLUSTER2_CTX}"
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                         AGE
linkerd-gateway   LoadBalancer   10.254.62.203   xxx.xxx.xxx.xxx   4143:30964/TCP,4191:32367/TCP   51s
```

<br>

#### 클러스터 cluster1, cluster2가 서로 연결되도록 link 설정
Linkerd 멀티 클러스터 통신은 클러스터 간 서비스 정보를 <b>'미러링'</b>하는 방식으로 작동한다. 클러스터 간 link 설정을 통해 해당 클러스터의 credentials 및 서비스 미러 구성 요소들을
상대 클러스터에 구성한다.
```bash
# cluster1의 credentials 추출 후 cluster2에 리소스 생성
linkerd multicluster link --context="${CLUSTER1_CTX}" --cluster-name "${CLUSTER1_NAME}" \
| kubectl --context="${CLUSTER2_CTX}" apply -f -

# cluster2의 credentials 추출 후 cluster1에 리소스 생성
linkerd multicluster link --context="${CLUSTER2_CTX}" --cluster-name "${CLUSTER2_NAME}" \
| kubectl --context="${CLUSTER1_CTX}" apply -f -
```

```bash
# 생성되는 리소스 목록
secret/cluster-credentials-cluster1 created (linkerd)
secret/cluster-credentials-cluster1 created (linkerd-multicluster)
link.multicluster.linkerd.io/cluster1 created
clusterrole.rbac.authorization.k8s.io/linkerd-service-mirror-access-local-resources-cluster1 created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-service-mirror-access-local-resources-cluster1 created
role.rbac.authorization.k8s.io/linkerd-service-mirror-read-remote-creds-cluster1 created
rolebinding.rbac.authorization.k8s.io/linkerd-service-mirror-read-remote-creds-cluster1 created
serviceaccount/linkerd-service-mirror-cluster1 created
deployment.apps/linkerd-service-mirror-cluster1 created
service/probe-gateway-cluster1 created
```
<br>

#### linkerd-multicluster 리소스 배포 현황
```bash
# cluster1의 linkerd-multicluster 리소스 배포 현황 확인 
$ kubectl get all -n linkerd-multicluster --context="${CLUSTER1_CTX}"
NAME                                                   READY   STATUS    RESTARTS   AGE
pod/linkerd-gateway-58ccc85959-fsgjh                   2/2     Running   0          3m3s
pod/linkerd-service-mirror-cluster2-7ddc698cc8-7szwv   2/2     Running   0          98s

NAME                             TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                         AGE
service/linkerd-gateway          LoadBalancer   198.19.154.231   xxx.xxx.xxx.xxx   4143:30903/TCP,4191:32408/TCP   3m3s
service/probe-gateway-cluster2   ClusterIP      198.19.240.220   <none>            4191/TCP                        98s

NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/linkerd-gateway                   1/1     1            1           3m3s
deployment.apps/linkerd-service-mirror-cluster2   1/1     1            1           98s

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/linkerd-gateway-58ccc85959                   1         1         1       3m3s
replicaset.apps/linkerd-service-mirror-cluster2-7ddc698cc8   1         1         1       98s
```
```bash
# cluster2의 linkerd-multicluster 리소스 배포 현황 확인  
$ kubectl get all -n linkerd-multicluster --context="${CLUSTER2_CTX}"
NAME                                                   READY   STATUS    RESTARTS   AGE
pod/linkerd-gateway-847b97dd4c-fbfwk                   2/2     Running   0          28m
pod/linkerd-service-mirror-cluster1-84cf9df486-zntv9   2/2     Running   0          13m

NAME                             TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                         AGE
service/linkerd-gateway          LoadBalancer   10.254.159.222   xxx.xxx.xxx.xxx   4143:32668/TCP,4191:32712/TCP   28m
service/probe-gateway-cluster1   ClusterIP      10.254.101.162   <none>            4191/TCP                        13m

NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/linkerd-gateway                   1/1     1            1           28m
deployment.apps/linkerd-service-mirror-cluster1   1/1     1            1           13m

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/linkerd-gateway-847b97dd4c                   1         1         1       28m
replicaset.apps/linkerd-service-mirror-cluster1-84cf9df486   1         1         1       13m
```
<br>

#### 멀티 클러스터 연결 상태 확인
각 클러스터에서 상대 클러스터의 연결 상태를 확인한다.
```bash
# cluster1 -> cluster2 연결 상태 확인
$ linkerd multicluster check --context="${CLUSTER1_CTX}"
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
```
```bash
# cluster2 -> cluster1 연결 상태 확인
$ linkerd multicluster check --context="${CLUSTER2_CTX}"
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

### <span id='3.5'>3.5. linkerd-smi 설치
트래픽 분할을 위한 Linkerd CRD `TrafficSplit`를 사용하기 위해 `linkerd-smi` extenstion 설치한다.<br>

```bash
# linkerd-smi extension repo 등록
$ helm repo add l5d-smi https://linkerd.github.io/linkerd-smi

$ helm repo list
NAME    URL
linkerd https://helm.linkerd.io/stable
l5d-smi https://linkerd.github.io/linkerd-smi (등록 확인)

# cluster1에 linkerd-smi 설치
helm install linkerd-smi l5d-smi/linkerd-smi -n linkerd-smi --create-namespace \
--kube-context="${CLUSTER1_CTX}"

# cluster2에 linkerd-smi 설치
helm install linkerd-smi l5d-smi/linkerd-smi -n linkerd-smi --create-namespace \
--kube-context="${CLUSTER2_CTX}"
```

<br>

## <span id='4'>4. 샘플 애플리케이션 배포
클러스터 cluster1, cluster2에 멀티 클러스터 통신 샘플 애플리케이션 배포를 진행한다.

#### 애플리케이션 배포 위치
| 애플리케이션| Cluster1 | Cluster2 | 정보 |
| :---: |:---: | :---: | :---: |  
| helloworld-v1 | :heavy_check_mark: | | 해당 버전<b>(v1)</b>과 호스트 명을 반환 |
| helloworld-v2 |  | :heavy_check_mark: | 해당 버전<b>(v2)</b>과 호스트 명을 반환 |
| sleep |  | :heavy_check_mark: | 다른 서비스를 호출하기 위한 요청 소스로 사용|


```bash
# 디렉토리 생성 
$ mkdir -p $HOME/linkerd/sample
$ cd $HOME/linkerd/sample

# helloworld 서비스 배포 스크립트 다운로드
$ wget https://raw.githubusercontent.com/istio/istio/master/samples/helloworld/gen-helloworld.sh
$ chmod +x gen-helloworld.sh

# sleep 서비스 배포 yaml 다운로드
$ wget https://raw.githubusercontent.com/istio/istio/master/samples/sleep/sleep.yaml

# cluster1에 sample 네임스페이스 생성 및 linkerd-proxy 주입 어노테이션 추가
$ kubectl create namespace sample --context="${CLUSTER1_CTX}"
$ kubectl annotate namespace sample "linkerd.io/inject=enabled" --context="${CLUSTER1_CTX}"

# cluster2에 sample 네임스페이스 생성 및 linkerd-proxy 주입 어노테이션 추가
$ kubectl create namespace sample --context="${CLUSTER2_CTX}"
$ kubectl annotate namespace sample "linkerd.io/inject=enabled" --context="${CLUSTER2_CTX}"
```

<br>

### <span id='4.1'>4.1. 클러스터 cluster1에 샘플 애플리케이션 배포
```bash
# cluster1에 helloworld-v1 애플리케이션 배포 
$ ./gen-helloworld.sh --version v1 --includeService true --includeDeployment true  | \
    kubectl --context="${CLUSTER1_CTX}" -n sample apply -f -
```
```bash
# helloworld-v1 서비스 리소스 생성
service/helloworld created
deployment.apps/helloworld-v1 created
```

<br>

#### 클러스터 cluster2에 서비스 미러링
클러스터 cluster1에 생성된 서비스 `helloworld` 를 클러스터 cluster2로 미러링한다.
> `mirror.linkerd.io/exported=true` 라벨 추가
```bash
$ kubectl label svc helloworld mirror.linkerd.io/exported=true --context="${CLUSTER1_CTX}" -n sample
```

<br>

클러스터 cluster2에 클러스터 cluster1 서비스 `helloworld`가 미러링되었는지 확인한다.
```bash
# cluster2에 `helloworld-{cluster1 명}`으로 서비스 생성되었는지 확인
$ kubectl get svc --context="${CLUSTER2_CTX}" -n sample
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
helloworld-cluster1   ClusterIP   10.254.83.67   <none>        5000/TCP   3m11s
```

<br>

### <span id='4.2'>4.2. 클러스터 cluster2에 샘플 애플리케이션 배포
```bash
# cluster2에 helloworld-v2 애플리케이션 배포 
$ ./gen-helloworld.sh --version v2 --includeService true --includeDeployment true  | \
    kubectl --context="${CLUSTER2_CTX}" -n sample apply -f -

# cluster2에 sleep 애플리케이션 배포
$ kubectl --context="${CLUSTER2_CTX}" -n sample apply -f sleep.yaml
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

#### `TrafficSplit` 배포
서비스 간의 트래픽 비율을 지정할 수 있는 리소스 `TrafficSplit`을 배포한다. <br>
서비스 helloworld로 들어오는 트래픽의 50%를 `helloworld-cluster1`로, 50%를 `helloworld`로 라우팅하도록 설정한다.
```yaml
cat << EOF | kubectl apply --context="${CLUSTER2_CTX}" -n sample -f -
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

### <span id='4.3'>4.3. 멀티 클러스터 통신 테스트
<b>TrafficSplit</b>을 통해 cluster1의 `helloworld-v1`, cluster2의 `helloworld-v2`로 트래픽 분할되어 통신되는 것을 확인할 수 있다.
```bash
# cluster1의 helloword-v1 pod 조회
$ kubectl get pods -l app=helloworld --context="${CLUSTER1_CTX}" -n sample
NAME                             READY   STATUS    RESTARTS   AGE
helloworld-v1-665db7d464-lcl75   2/2     Running   0          3m29s

# cluster2의 helloworld-v2 pod 조회
$ kubectl get pods -l app=helloworld --context="${CLUSTER2_CTX}" -n sample
NAME                            READY   STATUS    RESTARTS   AGE
helloworld-v2-f58d4c8cb-mzpsb   2/2     Running   0          3m3s
```

```bash
# cluster2의 sleep pod에 접속 
$ kubectl exec -it -n sample --context="${CLUSTER2_CTX}" \
"$(kubectl get pod -n sample --context="${CLUSTER2_CTX}" -l app=sleep -o jsonpath='{.items[0].metadata.name}')" -c sleep sh

# helloworld 서비스 curl 통신
~ $  curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-665db7d464-lcl75 (버전 v1, cluster1의 helloword-v1 pod명 출력 확인)
~ $  curl helloworld:5000/hello
Hello version: v2, instance: helloworld-v2-f58d4c8cb-mzpsb  (버전 v2, cluster2의 helloword-v2 pod명 출력 확인)
~ $  curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-665db7d464-lcl75
~ $  curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-665db7d464-lcl75
~ $  curl helloworld:5000/hello
Hello version: v2, instance: helloworld-v2-f58d4c8cb-mzpsb
~ $  curl helloworld:5000/hello
Hello version: v1, instance: helloworld-v1-665db7d464-lcl75
~ $  curl helloworld:5000/hello
Hello version: v2, instance: helloworld-v2-f58d4c8cb-mzpsb
~ $  curl helloworld:5000/hello
Hello version: v2, instance: helloworld-v2-f58d4c8cb-mzpsb
```

<br>

### [Index](https://github.com/K-PAAS/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > CSP 쿠버네티스 서비스 Linkerd 멀티 클러스터 설치 가이드