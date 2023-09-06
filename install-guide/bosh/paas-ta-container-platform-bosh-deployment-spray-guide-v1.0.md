## Table of Contents

1. [문서 개요](#1)  
    1.1. [목적](#1.1)  
    1.2. [범위](#1.2)  
    1.3. [시스템 구성도](#1.3)  
    1.4. [참고자료](#1.4)  

2. [컨테이너 플랫폼 설치](#2)  
    2.1. [Prerequisite](#2.1)  
    2.2. [Stemcell 확인](#2.2)  
    2.3. [Deployment 다운로드](#2.3)  
    2.4. [Deployment 파일 수정](#2.4)    
    2.5. [릴리즈 설치](#2.5)      
    2.6. [릴리즈 설치 - 다운로드 된 릴리즈 파일 이용 방식](#2.6)    
    2.7. [릴리즈 설치 확인](#2.7)      
    2.8. [CVE/CCE 진단항목 적용 ](#2.8)          

3. [컨테이너 플랫폼 배포](#3)  
    3.1. [Docker insecure-registry 설정](#3.1)  
    3.2. [컨테이너 플랫폼 이미지 업로드](#3.2)  
    3.3. [컨테이너 플랫폼 배포](#3.3)   
    3.3.1. [컨테이너 플랫폼 배포 YAML 내 환경변수 정의](#3.3.1)  
    3.3.2. [컨테이너 플랫폼 리소스 배포](#3.3.2)    
    3.3.3. [(참고) 컨테이너 플랫폼 리소스 삭제](#3.3.3)    

4. [컨테이너 플랫폼 운영자/사용자 포털 회원가입](#4)      
    4.1. [컨테이너 플랫폼 운영자 포털 회원가입](#4.1)      
    4.2. [컨테이너 플랫폼 운영자 포털 로그인](#4.2)      
    4.3. [컨테이너 플랫폼 사용자 포털 회원가입](#4.3)      
    4.4. [컨테이너 플랫폼 사용자 Namespace/Role 할당](#4.4)      
    4.5. [컨테이너 플랫폼 사용자 포털 로그인](#4.5)      
    4.6. [컨테이너 플랫폼 사용자/운영자 포털 사용 가이드](#4.6)    

5. [컨네이너 플랫폼 참고](#5)       
    5.1. [Cluster Role 운영자 생성 및 Token 획득](#5.1)  
    5.2. [kubernetes 리소스 생성 시 주의사항](#5.2)    

<br>

## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(컨테이너 플랫폼 설치 가이드)는 단독배포된 Kubernetes를 사용하기 위해 Bosh 기반 릴리즈 설치 방법을 기술하였다.

K-PaaS 3.5 버전부터는 Bosh 2.0 기반으로 배포(deploy)를 진행한다.

### <div id='1.2'>1.2. 범위
설치 범위는 Kubernetes 단독 배포를 기준으로 작성하였다.

### <div id='1.3'>1.3. 시스템 구성도
시스템 구성은 Kubernetes Cluster(Master, Worker)와 BOSH Inception(DBMS, HAProxy, Private Repository)환경으로 구성되어 있다. <br>
Kubespary를 통해 Kubernetes Cluster를 설치하고 BOSH 릴리즈로 Database, Private Repository 등 미들웨어 환경을 제공하여 Docker Image로 Kubernetes Cluster에 컨테이너 플랫폼 포털 환경을 배포한다. <br>
총 필요한 VM 환경으로는 **Master Node VM: 1개, Worker Node VM: 1개 이상, BOSH Inception VM: 1개**가 필요하고 본 문서는 BOSH Inception 환경을 구성하기 위한 VM 설치와 Kubernetes Cluster에 컨테이너 플랫폼을 배포하는 내용이다.

![image 001]

### <div id='1.4'>1.4. 참고 자료
> http://bosh.io/docs  
> https://docs.cloudfoundry.org

<br>

## <div id='2'>2. 컨테이너 플랫폼 설치
### <div id='2.1'>2.1. Prerequisite
본 설치 가이드는 **Ubuntu 18.04** 환경에서 설치하는 것을 기준으로 작성하였다. 단독 배포를 위해서는 Inception 환경이 구축 되어야 하므로 BOSH 2.0 설치와 K-PaaS 5.5 가이드의 Stemcell 업로드, Cloud Config 설정, Runtime Config 설정이 사전에 진행이 되어야 한다.
- [BOSH 2.0 설치 가이드](https://github.com/PaaS-TA/Guide/blob/master/install-guide/bosh/PAAS-TA_BOSH2_INSTALL_GUIDE_V5.0.md)
- [K-PaaS 5.5 설치 가이드](https://github.com/PaaS-TA/Guide/blob/master/install-guide/paasta/PAAS-TA_CORE_INSTALL_GUIDE_V5.0.md)

#### 방화벽 정보
IaaS Security Group의 열어줘야할 Port를 설정한다.

- Master Node

| <center>프로토콜</center> | <center>포트</center> | <center>Source</center> | <center>Destination</center> | <center>비고</center> |  
| :---: | :---: | :---: | :---: | :--- |  
| TCP | 6443 ||| kubernetes API Server |  
| TCP | 2379-2380 ||| etcd server client API |  
| TCP | 10250 ||| Kubelet API |  
| TCP | 10251 ||| kube-scheduler |  
| TCP | 10252 ||| kube-controller-manager |  
| TCP | 10255 ||| Read-Only Kubelet API |  
| UDP | 8285 | Worker Nodes || flannel overlay network |  
| UDP | 8472 | Worker Nodes || flannel overlay network |  
| TCP | 179 | Worker Nodes || Calio BGP network |  

---
- Worker Node

| <center>프로토콜</center> | <center>포트</center> | <center>Source</center> | <center>Destination</center> | <center>비고</center> |  
| :---: | :---: | :---: | :---: | :--- |  
| TCP | 10250 ||| Kubelet API |  
| TCP | 10255 ||| Read-Only Kubelet API |  
| TCP | 30000-32767 ||| NodePort Services |  
| UDP | 8285 | Worker Nodes || flannel overlay network |  
| UDP | 8472 | Worker Nodes || flannel overlay network |  
| TCP | 179 | Worker Nodes || Calio BGP network |

<br>

### <div id='2.2'>2.2. Stemcell 확인
Stemcell 목록을 확인하여 서비스 설치에 필요한 Stemcell이 업로드 되어 있는 것을 확인한다. (K-PaaS 5.5 와 동일 Stemcell 사용)
- Stemcell 업로드 및 Cloud Config, Runtime Config 설정 부분은 [K-PaaS 5.5 설치가이드](https://github.com/PaaS-TA/Guide/blob/master/install-guide/paasta/PAAS-TA_CORE_INSTALL_GUIDE_V5.0.md)를 참고 한다.
> $ bosh -e micro-bosh stemcells
```
Using environment '10.0.1.6' as client 'admin'

Name                                     Version  OS             CPI  CID
bosh-aws-xen-hvm-ubuntu-xenial-go_agent  621.94   ubuntu-xenial  -    ami-0694eb07c57faca73

(*) Currently deployed

1 stemcells

Succeeded
```

### <div id='2.3'>2.3. Deployment 다운로드
서비스 설치에 필요한 Deployment를 Git Repository에서 받아 서비스 설치 작업 경로로 위치시킨다.   
- 컨테이너 플랫폼 Deployment Git Repository URL : <br> https://github.com/PaaS-TA/paas-ta-container-platform-deployment

```
# Deployment 다운로드 파일 위치 경로 생성 및 이동
$ mkdir -p ~/workspace/paasta-5.5.1/deployment/
$ cd ~/workspace/paasta-5.5.1/deployment/

# Deployment 다운로드
$ git clone https://github.com/PaaS-TA/paas-ta-container-platform-deployment.git -b v1.0.4
```

### <div id='2.4'>2.4. Deployment 파일 수정
BOSH Deployment manifest는 Components 요소 및 배포의 속성을 정의한 YAML 파일이다. Deployment 파일에서 사용하는 network, vm_type, disk_type 등은 Cloud config를 활용하고, 활용 방법은 BOSH 2.0 가이드를 참고한다.
- Cloud config 설정 내용을 확인한다.
> $ bosh -e micro-bosh cloud-config
```
Using environment '10.0.1.6' as client 'admin'

azs:
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z1
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z2

... ((생략)) ...

disk_types:
- disk_size: 1024
  name: default
- disk_size: 1024
  name: 1GB

... ((생략)) ...

networks:
- name: default
  subnets:
  - az: z1
    cloud_properties:
      security_groups: paasta-security-group
      subnet: subnet-00000000000000000
    dns:
    - 8.8.8.8
    gateway: 10.0.1.1
    range: 10.0.1.0/24
    reserved:
    - 10.0.1.2 - 10.0.1.9
    static:
    - 10.0.1.10 - 10.0.1.120

... ((생략)) ...

vm_types:
- cloud_properties:
    ephemeral_disk:
      size: 3000
      type: gp2
    instance_type: t2.small
  name: minimal
- cloud_properties:
    ephemeral_disk:
      size: 10000
      type: gp2
    instance_type: t2.small
  name: small

... ((생략)) ...

Succeeded
```

> 일부 application의 경우 이중화를 위한 조치는 되어 있지 않으며 인스턴스 수 조정 시 신규로 생성되는 인스턴스에는 데이터의 반영이 안될 수 있으니, 1개의 인스턴스로 유지한다.

- Deployment YAML에서 사용하는 변수 파일을 서버 환경에 맞게 수정한다.
> $ vi ~/workspace/paasta-5.5.1/deployment/paas-ta-container-platform-deployment/bosh/manifests/paasta-container-service-vars-{IAAS}.yml
(e.g. {IAAS} :: aws)

> IPS - k8s_api_server_ip : Kubernetes Master Node Public IP<br>
  IPS - k8s_auth_bearer : [[5.1. Cluster Role 운영자 생성 및 Token 획득]](#5.1) 참고하여 Token 값 입력

#### paasta-container-service-vars-{IAAS}.yml
```
# INCEPTION OS USER NAME
inception_os_user_name: "ubuntu"

# REQUIRED FILE PATH VARIABLE
paasta_version: "5.5"

# RELEASE
container_platform_release_name: "paasta-container-platform"
container_platform_release_version: "1.0"

# IAAS
kubernetes_cluster_tag: 'kubernetes'                                                # Do not update!

# STEMCELL
stemcell_os: "ubuntu-xenial"                                                        # stemcell os
stemcell_version: "621.94"                                                          # stemcell version
stemcell_alias: "xenial"                                                            # stemcell alias

# CREDHUB
credhub_server_url: "10.0.1.6:8844"
credhub_admin_client_secret: "eft2zkfaerzyt8g6eonj"

# VM_TYPE
vm_type_small: "small"                                                              # vm type small
vm_type_small_highmem_16GB: "small-highmem-16GB"                                    # vm type small highmem
vm_type_small_highmem_16GB_100GB: "small-highmem-16GB"                              # vm type small highmem_100GB
vm_type_container_small: "small"                                                    # vm type small for caas's etc
vm_type_container_small_api: "small"                                                # vm type small for caas's api

# NETWORK
service_private_nat_networks_name: "default"                                        # private network name
service_private_networks_name: "default"
service_public_networks_name: "vip"                                                 # public network name

# IPS
haproxy_public_url: "<HAPROXY_IP>"                                                  # haproxy's public IP
k8s_api_server_ip: "<KUBERNETES_API_SERVER_IP>"                                     # kubernetes master node IP
k8s_api_server_port: "6443"                                                      
k8s_auth_bearer: "<KUBERNETES_AUTH_KEY>"                                            # kubernetes bearer token

# HAPROXY
haproxy_http_port: 8080                                                             # haproxy port
haproxy_azs: [z7]                                                                   # haproxy azs

# MARIADB
mariadb_port: "13306"                                                               # mariadb port (e.g. 13306)-- Do Not Use "3306"
mariadb_azs: [z5]                                                                   # mariadb azs
mariadb_persistent_disk_type: "10GB"                                                # mariadb persistent disk type
mariadb_admin_user_id: "cp-admin"                                                   # mariadb admin user name (e.g. cp-admin)
mariadb_admin_user_password: "Paasta!2021"                                          # mariadb admin user password (e.g. Paasta!2021)
mariadb_role_set_administrator_code_name: "Administrator"                           # administrator role's code name (e.g. Administrator)
mariadb_role_set_administrator_code: "RS0001"                                       # administrator role's code (e.g. RS0001)
mariadb_role_set_regular_user_code_name: "Regular User"                             # regular user role's code name (e.g. Regular User)
mariadb_role_set_regular_user_code: "RS0002"                                        # regular user role's code (e.g. RS0002)
mariadb_role_set_init_user_code_name: "Init User"                                   # init user role's code name (e.g. Init User)
mariadb_role_set_init_user_code: "RS0003"                                           # init user role's code (e.g. RS0003)

#PRIVATE IMAGE REPOSITORY
private_image_repository_azs: [z7]                                                   # private image repository azs
private_image_repository_port: 5001                                                  # private image repository port (e.g. 5001)-- Do Not Use "5000"
private_image_repository_root_directory: "/var/vcap/data/private-image-repository"   # private image repository root directory
private_image_repository_persistent_disk_type: "10GB"                                # private image repository's persistent disk type
```

### <div id='2.5'>2.5. 릴리즈 설치  
- 서버 환경에 맞추어 Deploy 스크립트 파일의 VARIABLES 설정을 수정한다.    
> $ vi ~/workspace/paasta-5.5.1/deployment/paas-ta-container-platform-deployment/bosh/deploy-{IAAS}.sh

```    
#!/bin/bash

# SET VARIABLES
export CONTAINER_DEPLOYMENT_NAME='paasta-container-platform'   # deployment name
export CONTAINER_BOSH2_NAME="<BOSH_NAME>"                     # bosh name (e.g. micro-bosh)
export CONTAINER_BOSH2_UUID=`bosh int <(bosh -e ${CONTAINER_BOSH2_NAME} environment --json) --path=/Tables/0/Rows/0/uuid`

# DEPLOY
bosh -e ${CONTAINER_BOSH2_NAME} -n -d ${CONTAINER_DEPLOYMENT_NAME} deploy --no-redact manifests/paasta-container-service-deployment-{IAAS}.yml \
    -l manifests/paasta-container-service-vars-{IAAS}.yml \    
    -o manifests/ops-files/paasta-container-service/network-{IAAS}.yml \
    -o manifests/ops-files/misc/first-time-deploy.yml \
    -v deployment_name=${CONTAINER_DEPLOYMENT_NAME} \
    -v director_name=${CONTAINER_BOSH2_NAME} \
    -v director_uuid=${CONTAINER_BOSH2_UUID}
```
- 릴리즈를 설치한다.
```
$ cd ~/workspace/paasta-5.5.1/deployment/paas-ta-container-platform-deployment/bosh
$ chmod +x *.sh
$ ./deploy-{IAAS}.sh
```

### <div id='2.6'>2.6. 릴리즈 설치 - 다운로드 된 릴리즈 파일 이용 방식
- 릴리즈 설치에 필요한 릴리즈 파일을 다운로드 받아 Local machine의 릴리즈 설치 작업 경로로 위치시킨다.  
  + 설치 릴리즈 파일 다운로드 :  
   [paasta-container-platform-1.0.tgz](https://nextcloud.k-paas.org/index.php/s/ggdZyEKejPSszFj/download)     
```
# 릴리즈 다운로드 파일 위치 경로 생성
$ mkdir -p ~/workspace/paasta-5.5.1/release/service
$ cd ~/workspace/paasta-5.5.1/release/service

# 릴리즈 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.k-paas.org/index.php/s/ggdZyEKejPSszFj/download
$ ls ~/workspace/paasta-5.5.1/release/service
  paasta-container-platform-1.0.tgz
```
- 서버 환경에 맞추어 Deploy 스크립트 파일의 VARIABLES 설정을 수정한다.  
  (추가) -o manifests/ops-files/use-offline-releases.yml \
> $ vi ~/workspace/paasta-5.5.1/deployment/paas-ta-container-platform-deployment/bosh/deploy-{IAAS}.sh

```    
#!/bin/bash

# SET VARIABLES
export CONTAINER_DEPLOYMENT_NAME='paasta-container-platform'   # deployment name
export CONTAINER_BOSH2_NAME="<BOSH_NAME>"                     # bosh name (e.g. micro-bosh)
export CONTAINER_BOSH2_UUID=`bosh int <(bosh -e ${CONTAINER_BOSH2_NAME} environment --json) --path=/Tables/0/Rows/0/uuid`

# DEPLOY
bosh -e ${CONTAINER_BOSH2_NAME} -n -d ${CONTAINER_DEPLOYMENT_NAME} deploy --no-redact manifests/paasta-container-service-deployment-{IAAS}.yml \
    -l manifests/paasta-container-service-vars-{IAAS}.yml \
    -o manifests/ops-files/use-offline-releases.yml \
    -o manifests/ops-files/paasta-container-service/network-{IAAS}.yml \
    -o manifests/ops-files/misc/first-time-deploy.yml \
    -v deployment_name=${CONTAINER_DEPLOYMENT_NAME} \
    -v director_name=${CONTAINER_BOSH2_NAME} \
    -v director_uuid=${CONTAINER_BOSH2_UUID}
```
- 릴리즈를 설치한다.
```
$ cd ~/workspace/paasta-5.5.1/deployment/paas-ta-container-platform-deployment/bosh
$ chmod +x *.sh
$ ./deploy-{IAAS}.sh
```

### <div id='2.7'>2.7. 릴리즈 설치 확인
설치 완료된 릴리즈를 확인한다.
> $ bosh -e micro-bosh -d paasta-container-platform vms
```
Using environment '10.0.1.6' as client 'admin'

Task 2983. Done

Deployment 'paasta-container-platform'

Instance                                                       Process State  AZ  IPs           VM CID               VM Type  Active
haproxy/32d1ff4e-1007-4e9a-8ebd-ffb33ba37348                   running        z7  10.0.0.121    i-0e6c374f2377ecf12  small    true
                                                                                  3.35.95.75
mariadb/42657509-69b6-4b4e-a006-20690e5ce2ea                   running        z5  10.0.161.121  i-0a8c71fb43ba3f34a  small    true
private-image-repository/2803b9a6-d797-4afb-9a34-65ce15853a9e  running        z7  10.0.0.122    i-0d5e4c451075e446b  small    true

3 vms
Succeeded
```

### <div id='2.8'>2.8. CVE/CCE 진단항목 적용
배포된 Kubernetes Cluster, BOSH Inception 환경에 아래 가이드를 참고하여 해당 CVE/CCE 진단항목을 필수적으로 적용시켜야 한다.    
- [CVE/CCE 진단 가이드](https://github.com/PaaS-TA/paas-ta-container-platform/blob/master/check-guide/paas-ta-container-platform-check-guide.md)

<br>

## <div id='3'>3. 컨테이너 플랫폼 배포
해당 항목부터는 배포된 Kubernetes Cluster 환경의 **Master Node**에서 진행한다. kubernetes에 K-PaaS용 컨테이너 플랫폼을 배포하기 위해서는 Bosh 릴리즈를 통해 배포된 Private Repository에 이미지를 업로드하는 작업이 필요하다.

### <div id='3.1'>3.1. Docker insecure-registry 설정
Kubernetes **Master Node, Worker Node** 내 docker daemon.json 파일에 'insecure-registries' 설정을 추가한다. <br>
Bosh 릴리즈를 통해 배포된 Private Repository를 'insecure-registries'로 설정 후 Docker를 재시작한다.<br>
>  - {HAProxy_IP} 값은 BOSH Inception에 배포된 Deployment 'paasta-container-platform' 의 haproxy public ip를 입력한다.
```
# Master Node, Worker Node 모두 'insecure-registries' 설정 추가 필요
$ sudo vi /etc/docker/daemon.json
{
        "insecure-registries": ["{HAProxy_IP}:5001"]
}

# docker 재시작
$ sudo systemctl restart docker
```

### <div id='3.2'>3.2. 컨테이너 플랫폼 이미지 업로드
Private Repository에 이미지 업로드를 위해 컨테이너 플랫폼 이미지 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
해당 내용은 Kubernetes **Master Node**에서 실행한다.

+ 컨테이너 플랫폼 이미지 파일 다운로드 :  
   [container-platform-standalone-image.tar](https://nextcloud.k-paas.org/index.php/s/PPCttKyiNcqYnJ9/download)  

```
# 이미지 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/paasta-5.5.1
$ cd ~/workspace/paasta-5.5.1

# 이미지 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.k-paas.org/index.php/s/PPCttKyiNcqYnJ9/download

$ ls ~/workspace/paasta-5.5.1
  container-platform-standalone-image.tar

# 이미지 파일 압축 해제
$ tar -xvf container-platform-standalone-image.tar
```

- 이미지 파일 디렉토리 구성

```
$ cd ~/workspace/paasta-5.5.1/container-platform

├── container-platform-image                            # 컨테이너 플랫폼 이미지 파일 위치
│   ├── container-platform-api.tar.gz
│   ├── container-platform-common-api.tar.gz
│   ├── container-platform-webadmin.tar.gz
│   └── container-platform-webuser.tar.gz
├── container-platform-script                           # 컨테이너 플랫폼 배포 관련 스크립트 파일 위치
│   ├── container-platform-deploy.sh
│   ├── container-platform-vars.sh
│   ├── image-upload-standalone.sh
│   └── remove-container-platform-resource.sh
├── container-platform-yaml                             # 컨테이너 플랫폼 배포 YAML 파일 위치
│   ├── paas-ta-container-platform-api.yaml
│   ├── paas-ta-container-platform-common-api.yaml
│   ├── paas-ta-container-platform-secret.yaml
│   ├── paas-ta-container-platform-temp-namespace.yaml
│   ├── paas-ta-container-platform-webadmin.yaml
│   └── paas-ta-container-platform-webuser.yaml
└── redis                                               # 컨테이너 플랫폼 사용자 세션관리를 위한 Redis 배포 YAML 파일 위치
    ├── kustomization.yaml
    ├── redis-config
    └── redis-deployment.yaml
```

#### Private Repository 이미지 업로드

 + Private Repository에 이미지를 업로드하는 스크립트를 실행한다. <br>
   스크립트 실행 후 Private Repository에 이미지가 정상적으로 업로드 되었는지 확인한다.
 ```
 $ cd ~/workspace/paasta-5.5.1/container-platform/container-platform-script
 $ chmod +x *.sh  
 $ ./image-upload-standalone.sh {HAProxy_IP}:5001

'''

*** List of uploaded images in the repository! ***

{"repositories":["container-platform-api","container-platform-common-api","container-platform-webadmin","container-platform-webuser"]}
 ```

 > * Private Repository에 업로드 된 이미지 목록 확인 명령어
 ```
 $ curl -H 'Authorization:Basic YWRtaW46YWRtaW4=' http://{HAProxy_IP}:5001/v2/_catalog
 ```


### <div id='3.3'>3.3. 컨테이너 플랫폼 배포

#### <div id='3.3.1'>3.3.1. 컨테이너 플랫폼 배포 YAML 내 환경변수 정의
컨테이너 플랫폼을 배포하기 전 배포 Yaml 내 환경변수 값 정의가 필요하다. 배포에 필요한 정보를 확인하여 변수를 설정한다.

```
$ vi container-platform-vars.sh
 ```

```                                                     
# COMMON ENV VARIABLE
HAPROXY_IP="xxx.xx.xx.xxx"                              # deployment 'paasta-container-platform' haproxy public ip
K8S_MASTER_NODE_IP="xxx.xx.xx.xxx"                      # kubernetes master node public ip
K8S_WORKER_NODE_IP="xxx.xx.xx.xxx"                      # kubernetes worker node public ip
K8S_WORKER_NODE_HOSTNAME="{k8s worker node hostname}"   # kubernetes worker node host name (run 'hostname' command in worker node)
CP_CLUSTER_NAME="{cluster name}"                        # cluster name to use on the container platform portal
MARIADB_USER_ID="{mariadb admin user id}"               # mariadb admin user id (e.g. cp-admin)
MARIADB_USER_PASSWORD="{mariadb admin user password}"   # mariadb admin user password (e.g. Paasta!2021)
```

> - HAPROXY_IP :<br>BOSH Inception에 배포된 Deployment 'paasta-container-platform' 의 haproxy public ip 입력 <br><br>
> - K8S_MASTER_NODE_IP :<br>Kubernetes master node public ip 입력 <br><br>
> - K8S_WORKER_NODE_IP :<br>Kubernetes worker node public ip 입력 <br>
>   + worker node가 2개 이상인 경우, 그 중 한 worker node의 public ip를 입력 &nbsp; :: ex)첫 번째 woker node의 public ip <br><br>
> - K8S_WORKER_NODE_HOSTNAME :<br>위 'K8S_WORKER_NODE_IP'에 입력한 woker node의 hostname 입력
>   + 해당 worker node 접속 후 명령어 'hostname'으로 확인 <br><br>
> - CP_CLUSTER_NAME :<br>컨테이너 플랫폼에서 사용할 클러스터 명으로 원하는 값 입력<br>
>   + 배포 후 운영자 포털 접속 및 회원가입 시 해당 클러스터 명 입력 필요 <br><br>
> - MARIADB_USER_ID :<br>배포된 Deployment 'paasta-container-platform' 의 mariadb admin user id 입력 <br>
>   + [paasta-container-service-vars-{IAAS}.yml](#paasta-container-service-vars-iaasyml) 내 MARIADB - 'mariadb_admin_user_id' 값 입력 <br><br>
> - MARIADB_USER_PASSWORD :<br>배포된 Deployment 'paasta-container-platform' 의 mariadb admin password 입력 <br>
>   + [paasta-container-service-vars-{IAAS}.yml](#paasta-container-service-vars-iaasyml) 내 MARIADB - 'mariadb_admin_user_password' 값 입력 <br><br>


#### <div id='3.3.2'>3.3.2. 컨테이너 플랫폼 리소스 배포
컨테이너 플랫폼 배포를 위한 배포 스크립트를 실행한다.

```
$ ./container-platform-deploy.sh
```

컨테이너 플랫폼 리소스가 정상적으로 배포되었는지 확인한다.<br>
리소스 Pod의 경우 Node에 바인딩 및 컨테이너 생성 후 Running 상태로 전환되기까지 몇 초가 소요된다.

```
*** Deployed Container Platform Resource List ***

NAME                                         READY   STATUS    RESTARTS   AGE
pod/api-deployment-657d86878f-8lrvb          1/1     Running   0          4m8s
pod/common-api-deployment-5579496679-27pj4   1/1     Running   0          4m8s
pod/redis-deployment-5bdcc468f4-9v4lp        1/1     Running   0          4m7s
pod/webadmin-deployment-d8d559b6c-ntthd      1/1     Running   0          4m7s
pod/webuser-deployment-64fdd9dd56-ch496      1/1     Running   0          4m7s

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/api-deployment          NodePort    10.233.21.136   <none>        3333:30333/TCP   4m8s
service/common-api-deployment   NodePort    10.233.37.90    <none>        3334:30334/TCP   4m8s
service/kubernetes              ClusterIP   10.233.0.1      <none>        443/TCP          5d23h
service/redis-deployment        NodePort    10.233.27.120   <none>        6379:32079/TCP   4m7s
service/webadmin-deployment     NodePort    10.233.36.158   <none>        8080:32080/TCP   4m7s
service/webuser-deployment      NodePort    10.233.12.9     <none>        8091:32091/TCP   4m7s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/api-deployment          1/1     1            1           4m8s
deployment.apps/common-api-deployment   1/1     1            1           4m8s
deployment.apps/redis-deployment        1/1     1            1           4m7s
deployment.apps/webadmin-deployment     1/1     1            1           4m7s
deployment.apps/webuser-deployment      1/1     1            1           4m7s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/api-deployment-657d86878f          1         1         1       4m8s
replicaset.apps/common-api-deployment-5579496679   1         1         1       4m8s
replicaset.apps/redis-deployment-5bdcc468f4        1         1         1       4m7s
replicaset.apps/webadmin-deployment-d8d559b6c      1         1         1       4m7s
replicaset.apps/webuser-deployment-64fdd9dd56      1         1         1       4m7s

NAME                                        STATUS   AGE
paas-ta-container-platform-temp-namespace   Active   4m8s

NAME        TYPE                             DATA   AGE
cp-secret   kubernetes.io/dockerconfigjson   1      4m8s

NAME                         DATA   AGE
cp-redis-config-bbdk5h2k6b   1      4m7s
```
##### 배포된 리소스 조회 명령어

 >  + Deployment, ReplicaSet, Pod, Service 조회
 ```
 $ kubectl get all
 ```
 > + Namespace 조회
 ```
 $ kubectl get namespace paas-ta-container-platform-temp-namespace
 ```
 > +  Secret 조회
 ```
 $ kubectl get secret cp-secret
 ```
 > + ConfigMap 조회
 ```
 $ kubectl get configmap -l app=cp-redis-config
 ```

<br>

#### <div id='3.3.3'>3.3.3. (참고) 컨테이너 플랫폼 리소스 삭제
배포된 컨테이너 플랫폼 리소스의 삭제를 원하는 경우 아래 스크립트를 실행한다.<br>
:loudspeaker: (주의) 컨테이너 플랫폼이 운영되는 상태에서 해당 스크립트 실행 시, 운영에 필요한 리소스가 모두 삭제되므로 주의가 필요하다. <br>
또한 Kubernetes Cluster 환경에 배포된 컨테이너 플랫폼 리소스가 삭제되며, BOSH Inception에 배포된 Deployment 'paasta-container-platform'에는 영향이 가지 않는다. <br>

```
$ cd ~/workspace/paasta-5.5.1/container-platform/container-platform-script
$ ./remove-container-platform-resource.sh
Are you sure you want to delete the deployed container platform resource? <y/n> y

deployment.apps "api-deployment" deleted
service "api-deployment" deleted
deployment.apps "common-api-deployment" deleted
service "common-api-deployment" deleted
secret "cp-secret" deleted
namespace "paas-ta-container-platform-temp-namespace" deleted
deployment.apps "webadmin-deployment" deleted
service "webadmin-deployment" deleted
deployment.apps "webuser-deployment" deleted
service "webuser-deployment" deleted
configmap "cp-redis-config-bbdk5h2k6b" deleted
service "redis-deployment" deleted
deployment.apps "redis-deployment" deleted
```

<br>

## <div id='4'>4. 컨테이너 플랫폼 운영자/사용자 포털 회원가입

컨테이너 플랫폼 최초 배포의 경우 운영자 포털 회원가입을 통해 Kubernetes Cluster 정보 등록이 선 진행되어야 한다. 따라서 운영자포털 회원가입 완료 후 사용자 포털 회원가입을 진행하도록 한다.

> - 컨테이너 플랫폼 운영자포털 접속 URI :: http://{K8S_WORKER_NODE_IP}:32080 <br>
> - 컨테이너 플랫폼 사용자포털 접속 URI :: http://{K8S_WORKER_NODE_IP}:32091 <br>
>   + {K8S_WORKER_NODE_IP} : [container-platform-vars.sh](#3.3.1)에 설정한 변수 'K8S_WORKER_NODE_IP' 값을 대입한다.

### <div id='4.1'/>4.1. 컨테이너 플랫폼 운영자 포털 회원가입
운영자 포털을 접속하기 전 네임스페이스 'paas-ta-container-platform-temp-namespace' 가 정상적으로 생성되어 있는지 확인한다.<br>
해당 Temp Namespace는 컨테이너 플랫폼 내 사용자 계정 관리를 위해 사용된다.

> $ kubectl get namespace paas-ta-container-platform-temp-namespace
```
NAME                                        STATUS   AGE
paas-ta-container-platform-temp-namespace   Active   74m
```

Kubernetes Cluster 정보, 생성할 Namespace 명, User 정보를 입력 후 [회원가입] 버튼을 클릭하여 컨테이너 플랫폼 운영자포털에 회원가입을 진행한다.

![image 005]
> - Kubernetes Cluster Name : <br> [container-platform-vars.sh](#3.3.1)에 설정한 변수 'CP_CLUSTER_NAME' 값을 입력 <br><br>
> - Kubernetes Cluster API URL : <br> <b>https://{K8S_MASTER_NODE_IP}:6443</b> 입력 <br>
>    + {K8S_MASTER_NODE_IP}는 [container-platform-vars.sh](#3.3.1)에 설정한 변수 'K8S_MASTER_NODE_IP' 값을 입력한다. <br><br>
> - Kubernetes Cluster Token : <br> [paasta-container-service-vars-{IAAS}.yml](#paasta-container-service-vars-iaasyml) 내 IPS - 'k8s_auth_bearer' 값 입력 <br>
>    + (참고) [[5.1. Cluster Role 운영자 생성 및 Token 획득]](#5.1) <br><br>
> - Namespace : <br> 신규로 생성할 Namespace 명을 입력<br>
>    + 회원가입 완료 후 해당 Namespace에 kubernetes에서 제공하는 ClusterRole 'cluster-admin' 과 운영자 계정이 바인딩된다.

<br>

```
# ex) 이해를 돕기 위한 예시 정보
# {Kubernetes Cluster Name} : cp-cluster
# {Kubernetes Cluster API URL} : https://xxx.xxx.xxx.xxx:6443
# {Kubernetes Cluster Token} : qY3k2xaZpNbw3AJxxxxx......
```
### <div id='4.2'/>4.2. 컨테이너 플랫폼 운영자 포털 로그인
- 사용자 ID와 비밀번호를 입력 후 [로그인] 버튼을 클릭하여 컨테이너 플랫폼 운영자 포털에 로그인 한다.

![image 006]

### <div id='4.3'/>4.3. 컨테이너 플랫폼 사용자 포털 회원가입
- 등록할 사용자 계정정보(사용자 ID, Password, E-mail)를 입력 후 [Register] 버튼을 클릭하여  컨테이너 플랫폼 사용자 포털에 회원가입한다. <br> 사용자 포털은 회원가입 후 즉시 이용이 불가하며 Cluster 관리자 혹은 Namespace 관리자로부터 해당 사용자가 이용할 Namespace와 Role을 할당 받은 후 포털 이용이 가능하다.

![image 007]

### <div id='4.4'/>4.4. 컨테이너 플랫폼 사용자 Namespace/Role 할당
## 1) Namespace 관리자 지정
- Clusters 메뉴 > Namespaces 선택 > 할당 하고자하는 Namespace 명 선택 > 하단 [수정]버튼 클릭

![image 008]
![image 009]

- 해당 Namespace의 관리자로 지정할 사용자 ID 선택 후 저장버튼 클릭
- 해당 Namespace의 Resource Quotas, Limit Ranges 수정 가능

![image 010]

- [참고] Namespace 생성시에도 Namespace 관리자를 지정할 수 있다.
![image 011]

## 2) Namespace 사용자 지정

### 운영자 포털
- Managements 메뉴 > Users 선택 > User 탭 선택 > 사용자 ID 선택 > 하단 [수정]버튼 클릭
![image 015]
![image 016]

- Namespaces/Roles 선택 > [수정]버튼 클릭

해당 사용자가 이용할 Namespace와 Role을 지정할 수 있다.
![image 017]
![image 018]

<hr>

### 사용자 포털
Namespace 관리자는 해당 Namespace를 이용중인 사용자의 Role 변경 및 해당 Namespace를 미사용하는 사용자에게 접근 권한을 할당할 수 있다.

![image 012]
![image 013]
![image 014]


### <div id='4.5'/>4.5. 컨테이너 플랫폼 사용자 포털 로그인
- 사용자 ID와 비밀번호를 입력후 [로그인] 버튼을 클릭하여 컨테이너 플랫폼 사용자 포털에 로그인 한다.

![image 019]

### <div id='4.6'/>4.6. 컨테이너 플랫폼 사용자/운영자 포털 사용 가이드
- 컨테이너 플랫폼 포털 사용방법은 아래 사용가이드를 참고한다.  
  + [컨테이너 플랫폼 운영자 포털  사용 가이드](https://github.com/PaaS-TA/paas-ta-container-platform/blob/master/use-guide/portal/paas-ta-container-platform-admin-guide-v1.0.md)    
  + [컨테이너 플랫폼 사용자 포털  사용 가이드](https://github.com/PaaS-TA/paas-ta-container-platform/blob/master/use-guide/portal/paas-ta-container-platform-user-guide-v1.0.md)


<br>

## <div id='5'>5. 컨네이너 플랫폼 참고

### <div id='5.1'>5.1. Cluster Role 운영자 생성 및 Token 획득
Cluster Role을 가진 운영자의 Service Account를 생성하고 해당 Service Account의 Token 값을 획득한다.<br>
획득한 Token 값은 컨테이너 플랫폼 배포 및 컨테이너 플랫폼 운영자 포털 회원가입에 사용된다.

- Service Account를 생성한다.
```
## {SERVICE_ACCOUNT} : 생성할 Service Account 명

$ kubectl create serviceaccount {SERVICE_ACCOUNT} -n kube-system
(ex. kubectl create serviceaccount k8sadmin -n kube-system)
```

- 생성한 Service Account와 kubernetes에서 제공하는 ClusterRole 'cluster-admin'을 바인딩한다.
```
$ kubectl create clusterrolebinding {SERVICE_ACCOUNT} --clusterrole=cluster-admin --serviceaccount=kube-system:{SERVICE_ACCOUNT}
(ex. kubectl create clusterrolebinding k8sadmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8sadmin)
```

- Service Account의 Mountable secrets 값을 확인한다.
```
$ kubectl describe serviceaccount {SERVICE_ACCOUNT} -n kube-system
(ex. kubectl describe serviceaccount k8sadmin -n kube-system)

...

Mountable secrets:   k8sadmin-token-xxxx
```

- Service Account의 Token을 획득한다.

```
## {SECRET_NAME} : Mountable secrets 값 입력

$ kubectl describe secret {SECRET_NAME} -n kube-system | grep -E '^token' | cut -f2 -d':' | tr -d " "
```
<br>

### <div id='5.2'>5.2. kubernetes 리소스 생성 시 주의사항

컨테이너 플랫폼 이용 중 리소스 생성 시 다음과 같은 prefix를 사용하지 않도록 주의한다.

|Resource 명|생성 시 제외해야 할 prefix|
|---|---|
|전체 Resource|kube*|
|Namespace|all|
||kubernetes-dashboard|
||paas-ta-container-platform-temp-namespace|
|Role|paas-ta-container-platform-init-role|
||paas-ta-container-platform-admin-role|
|ResourceQuota|paas-ta-container-platform-low-rq|
||paas-ta-container-platform-medium-rq|
||paas-ta-container-platform-high-rq|
|LimitRanges|paas-ta-container-platform-low-limit-range|
||paas-ta-container-platform-medium-limit-range|
||paas-ta-container-platform-high-limit-range|
|Pod|nodes|
||resources|

----
[image 001]:images/cp-001.png
[image 005]:images/cp-005.png
[image 006]:images/cp-006.png
[image 007]:images/cp-007.png
[image 008]:images/cp-008.png
[image 009]:images/cp-009.png
[image 010]:images/cp-010.png
[image 011]:images/cp-011.png
[image 012]:images/cp-012.png
[image 013]:images/cp-013.png
[image 014]:images/cp-014.png
[image 015]:images/cp-015.png
[image 016]:images/cp-016.png
[image 017]:images/cp-017.png
[image 018]:images/cp-018.png
[image 019]:images/cp-019.png
