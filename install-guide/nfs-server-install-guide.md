### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > NFS Server 설치 가이드


# NFS 서버 설치

## Table of Contents

1. [문서 개요](#1)<br>
  1.1. [목적](#1.1)<br>
  1.2. [범위](#1.2)

2. [NFS Server 설치](#2)<br>
  2.1. [Prerequisite](#2.1)<br>
  2.2. [설치](#2.2)<br>
  2.3. [동작확인](#2.3)

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

#### OS
K-PaaS 컨테이너 플랫폼 클러스터 설치에 필요한 OS 환경 정보는 다음과 같다. (***필수 확인***)

|지원 OS|버전|
|---|---|
|Ubuntu|18.04|
|Ubuntu|20.04|
|Ubuntu|22.04|

<br>

### <div id='2.2'> 2.2. 설치
APT 업데이트를 진행한다.
```
$ sudo apt-get update
```

<br>

NFS Server를 위한 APT 패키지 설치를 진행한다.
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

### <div id='2.2'> 2.2. 동작 확인
NFS Server 설정을 확인한다.
```
$ sudo exportfs -v
```

```
/home/share/nfs
                <world>(rw,async,wdelay,no_root_squash,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > NFS Server 설치 가이드
