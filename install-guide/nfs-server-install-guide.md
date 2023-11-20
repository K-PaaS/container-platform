### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > NFS Server 설치


# NFS 서버 설치

## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  

2. [NFS Server 설치](#2)  
  2.1. [Prerequisite](#2.1)  
  2.2. [설치](#2.2)  
  2.3. [동작확인](#2.3)

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (Kubespray 설치 가이드) 는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에 배포되는 컨테이터 플랫폼을 위한 제반환경을 위한 NFS Storage Server를 설치하는 방법을 기술하였다.

PaaS-TA 6.0 버전부터는 Kuberspray에서 배포되는 기본클러스터에 컨테이너플랫폼용 서비스를 설치하고자 할 경우에는 Persistence Volume으로 사용할 스토리지가 필수다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 Kubernetes Native를 검증하기 위한 Kubespray 기본 설치를 기준으로 작성하였다.

<br>

## <div id='2'> 2. NFS Server 설치

<br>

### <div id='2.1'> 2.1. Prerequisite
본 설치 가이드는 **Ubuntu 18.04** 환경에서 설치하는 것을 기준으로 하였다. Kubespray로 배포된 Cluster에서 사용할 Storage용이기에 Storage용 별도 VM에 설치한다.


### <div id='2.2'> 2.2. 설치
- 패키지 업데이트
```
$ sudo apt-get update
```

- NFS 서버를 위한 패키지를 설치

```
$ sudo apt-get install -y nfs-common nfs-kernel-server portmap
```

- NFS에서 사용될 디렉토리 생성 및 권한 부여
```
$ sudo mkdir -p /home/share/nfs
$ sudo chmod 777 /home/share/nfs
```

- 공유 디렉토리 설정
```
$ sudo vi /etc/exports

## 형식 : [/공유디렉토리] [접근IP] [옵션]
## 예시 : /home/share/nfs 10.0.0.1(rw,no_root_squash,async)
/home/share/nfs {{MASTER_NODE_PRIVATE_IP}}(rw,no_root_squash,async)
/home/share/nfs {{WORKER1_NODE_PRIVATE_IP}}(rw,no_root_squash,async)
/home/share/nfs {{WORKER2_NODE_PRIVATE_IP}}(rw,no_root_squash,async)
...
```
>`rw - 읽기쓰기` <br>
>       `no_root_squash - 클라이언트가 root 권한 획득 가능, 파일생성 시 클라이언트 권한으로 생성됨.`<br>
>       `async - 요청에 의해 변경되기 전에 요청에 응답, 성능 향상용`


- nfs 서버 재시작
```
$ sudo /etc/init.d/nfs-kernel-server restart
$ sudo systemctl restart portmap
```


### <div id='2.2'> 2.2. 동작 확인

- 설정 확인
```
$ sudo exportfs -v
```

- 정상 결과
```
/home/share/nfs
                <world>(rw,async,wdelay,no_root_squash,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide) > > NFS Server 설치
