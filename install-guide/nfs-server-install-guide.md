### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > NFS Server 설치 가이드

<br>

# NFS Server 설치 가이드

<br>

## Table of Contents

1. [문서 개요](#1)<br>
   1.1. [목적](#1.1)<br>
   1.2. [범위](#1.2)

2. [NFS Server 설치](#2)<br>
   2.1. [Prerequisite](#2.1)<br>
   2.2. [설치](#2.2)<br>
   2.3. [동작확인](#2.3)

3. [NFS Subdir External Provisioner 설치](#3)

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (NFS Server 설치 가이드) 는 기획자, 개발자, 운영자 지원 환경의 개방형 PaaS 플랫폼인 K-PaaS 컨테이너 플랫폼의 스토리지 환경 구성을 위한 NFS Server를 설치하는 방법을 기술하였다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 K-PaaS 컨테이너 플랫폼 스토리지 환경 구성을 기준으로 작성하였다.

<br>

## <div id='2'> 2. NFS Server 설치

### <div id='2.1'> 2.1. Prerequisite
NFS Server 설치를 위해서는 1개 인스턴스가 필요하며 설치에 필요한 전제조건은 아래 기술하였다.

### OS (***필수 확인***)
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 OS 환경 정보는 다음과 같다.

|지원 OS|버전|
|---|---|
|Ubuntu|18.04|
|Ubuntu|20.04|
|Ubuntu|22.04|

<br>

### <div id='2.2'> 2.2. 설치
APT 패키지 목록을 업데이트한다.
```
$ sudo apt-get update
```

<br>

NFS Server 관련 APT 패키지를 설치한다.
```
$ sudo apt-get install -y nfs-common nfs-kernel-server portmap
```

<br>

NFS Server에서 사용할 디렉토리 생성 및 권한 부여를 진행한다.
```
$ sudo mkdir -p /home/share/nfs
$ sudo chmod 777 /home/share/nfs
```

<br>

공유 디렉토리 설정을 진행한다.
```
$ sudo vi /etc/exports
```

```
## 형식 : [/공유디렉토리] [접근IP] [옵션]
## 예시 : /home/share/nfs 10.0.0.1(rw,no_root_squash,async)
...
/home/share/nfs {{MASTER_NODE_PRIVATE_IP}}(rw,no_root_squash,async)
/home/share/nfs {{WORKER1_NODE_PRIVATE_IP}}(rw,no_root_squash,async)
/home/share/nfs {{WORKER2_NODE_PRIVATE_IP}}(rw,no_root_squash,async)
...
```

> `rw - 읽기쓰기` <br>
> `no_root_squash - 클라이언트가 root 권한 획득 가능, 파일생성 시 클라이언트 권한으로 생성됨.`<br>
> `async - 요청에 의해 변경되기 전에 요청에 응답, 성능 향상용`

<br>

NFS Server 재시작을 진행한다.
```
$ sudo /etc/init.d/nfs-kernel-server restart
$ sudo systemctl restart portmap
```

<br>

### <div id='2.3'> 2.3. 동작 확인
NFS Export 설정을 확인한다.
```
$ sudo exportfs -v
```

```
/home/share/nfs
                <world>(rw,async,wdelay,no_root_squash,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```


<br>


## <div id='3'> 3. NFS Subdir External Provisioner 설치
> **참고:** K-PaaS 클러스터 설치 시 스토리지 구성이 NFS 일 경우 NFS Subdir External Provisioner가 자동으로 설치된다. K-PaaS 클러스터 설치 사용자는 본 절의 수동 설치 과정을 생략한다. 아래 내용은 NFS Provisioner를 수동 설치하는 경우에 참고한다.

<br>

Kubernetes 클러스터에서 NFS 기반 동적 PersistentVolume 프로비저닝을 지원하기 위해 [`nfs-subdir-external-provisioner`](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)를 설치한다.  
해당 Provisioner는 PVC 생성 요청 시 NFS 서버의 지정된 디렉터리 하위에 전용 하위 디렉터리를 자동으로 생성한다.

### 사전 준비
- **NFS Server 설치 및 구성 완료**  
  Provisioner가 사용할 NFS Server는 사전에 [설치 및 구성](#2)되어 있어야 한다. <br>
  `/home/share/nfs` 경로를 NFS Export로 공유하도록 설정한다.

<br>

### values.yaml 작성
> **참고 :** 아래 values.yaml은 K-PaaS 클러스터 설치 시 사용되는 기본 값이다. 필요 시 환경에 맞게 추가 설정을 적용한다.<br>

⚠️ `nfs.server` 값은 반드시 실제 설치된 NFS Server IP로 변경한다.
```yaml
nfs:
  server: {NFS Server IP}                     # NFS Server IP 입력
  path: /home/share/nfs                       # NFS Export Path (Provisioner가 사용할 디렉터리)

storageClass:
  provisionerName: cp-nfs-provisioner         # StorageClass에 사용할 Provisioner 이름 설정
  defaultClass: true                          # 기본 StorageClass로 지정할지 여부
  name: cp-storageclass                       # 생성할 StorageClass 이름 설정
  reclaimPolicy: Retain                       # PV Reclaim 정책 (Retain: PV 유지, Delete: 자동 삭제)
  allowVolumeExpansion: false                 # PVC 확장 기능 여부
  archiveOnDelete: false                      # PVC 삭제 시 NFS 디렉터리 아카이브 여부
```

<br>

### Helm Chart 설치

#### 1. Helm Repo 추가

```bash
$ helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
$ helm repo update nfs-subdir-external-provisioner
```

#### 2. Provisioner 설치

```bash
$ helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  -f values.yaml --version 4.0.18
```

<br>

### 설치 확인

설치 후 다음 명령어로 정상 설치 여부를 확인한다.

```bash
$ helm list -n default
NAME                            NAMESPACE       REVISION       STATUS         CHART                                    APP VERSION
nfs-subdir-external-provisioner default         1              deployed       nfs-subdir-external-provisioner-4.0.18   4.0.2
```
```bash
$ kubectl get all -l app=nfs-subdir-external-provisioner
NAME                                                   READY   STATUS    RESTARTS   AGE
pod/nfs-subdir-external-provisioner-5cdc76bcd9-jtjh4   1/1     Running   0          3m20s

NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nfs-subdir-external-provisioner   1/1     1            1           3m20s

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/nfs-subdir-external-provisioner-5cdc76bcd9   1         1         1       3m20s
```
```bash
$ kubectl get storageclass
NAME                        PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
cp-storageclass (default)   cp-nfs-provisioner   Retain          Immediate           false                  3m46s
```

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > NFS Server 설치 가이드