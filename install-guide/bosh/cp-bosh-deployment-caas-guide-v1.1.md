## Table of Contents

1. [문서 개요](#1)  
 1.1. [목적](#1.1)  
 1.2. [범위](#1.2)  
 1.3. [시스템 구성도](#1.3)  
 1.4. [참고자료](#1.4)  

2. [컨테이너 서비스 설치](#2)  
 2.1. [Prerequisite](#2.1)  
 2.2. [Stemcell 확인](#2.2)  
 2.3. [Deployment 다운로드](#2.3)  
 2.4. [Deployment 파일 수정](#2.4)    
 2.5. [서비스 설치](#2.5)    
 2.6. [서비스 설치 - 다운로드 된 릴리즈 파일 이용 방식](#2.6)   
 2.7. [서비스 설치 확인](#2.7)    
 2.8. [CVE/CCE 진단항목 적용](#2.8)    

3. [컨테이너 서비스 배포](#3)  
 3.1. [CRI-O insecure-registry 설정](#3.1)  
 3.2. [컨테이너 서비스 이미지 업로드](#3.2)  
 3.3. [컨테이너 서비스 배포](#3.3)   
 3.3.1. [컨테이너 서비스 배포 YAML 내 환경변수 정의](#3.3.1)  
 3.3.2. [컨테이너 서비스 리소스 배포](#3.3.2)    

4. [컨테이너 서비스 브로커](#4)  
 4.1. [컨테이너 서비스 브로커 등록](#4.1)  
 4.2. [컨테이너 서비스 UAA Client 등록](#4.2)   
 4.3. [K-PaaS 포털에서 컨테이너 서비스 조회 설정](#4.3)      

5. [Jenkins 서비스 브로커(Optional)](#5)   
 5.1. [Jenkins 서비스 브로커 배포](#5.1)   
 5.2. [Jenkins 서비스 브로커 등록](#5.2)  
 5.3. [K-PaaS 포털에서 Jenkins 서비스 조회 설정](#5.3)   

6. [참고](#6)  
 6.1. [Cluster Role 사용자 생성 및 Token 획득](#6.1)   

<br>

## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(컨테이너 서비스 설치 가이드)는 Kubernetes를 사용하기 위해 Bosh 기반 릴리즈의 설치 및 서비스 등록 방법을 기술하였다.
<br>K-PaaS 3.5 버전부터는 Bosh 2.0 기반으로 배포(deploy)를 진행한다.

<br>

### <div id='1.2'>1.2. 범위
설치 범위는 Kubernetes 서비스 배포를 기준으로 작성하였다.

<br>

### <div id='1.3'>1.3. 시스템 구성도
시스템 구성은 Kubernetes Cluster(Master, Worker)와 BOSH Inception(DBMS, HAProxy, Private Repository)환경으로 구성되어 있다. <br>
Kubespary를 통해 Kubernetes Cluster를 설치하고 BOSH 릴리즈로 Database, Private Repository 등 미들웨어 환경을 제공하여 Docker Image로 Kubernetes Cluster에 컨테이너 서비스 포털 환경을 배포한다. <br>
총 필요한 VM 환경으로는 **Master Node VM: 1개, Worker Node VM: 1개 이상, BOSH Inception VM: 1개**가 필요하고 본 문서는 BOSH Inception 환경을 구성하기 위한 VM 설치와 컨테이너 서비스를 설치하는 내용이다.

![image 001]

<br>

### <div id='1.4'>1.4. 참고 자료
> http://bosh.io/docs  
> https://docs.cloudfoundry.org

<br>

## <div id='2'>2. 컨테이너 서비스 설치
### <div id='2.1'>2.1. Prerequisite
본 설치 가이드는 **Ubuntu 18.04** 환경에서 설치하는 것을 기준으로 작성하였다. 서비스 설치를 위해서는 BOSH 2.0과 K-PaaS 5.5, K-PaaS 포털 API, K-PaaS 포털 UI가 설치 되어 있어야 한다.
> [BOSH 2.0 설치 가이드](https://github.com/PaaS-TA/application-platform-guide/blob/master/install/application_platform/bosh.md) <br>
> [K-PaaS 5.5 설치 가이드](https://github.com/PaaS-TA/application-platform-guide/blob/master/install/application_platform/paasta_ap.md) <br>
> [K-PaaS 포털 API 설치 가이드](https://github.com/PaaS-TA/application-platform-guide/blob/master/install/portal/vm_type_api.md) <br>
> [K-PaaS 포털 UI 설치 가이드](https://github.com/PaaS-TA/application-platform-guide/blob/master/install/portal/vm_type_ui.md) <br>

#### 방화벽 정보
IaaS Security Group의 열어줘야할 Port를 설정한다.

- Master Node

| <center>프로토콜</center> | <center>포트</center> | <center>비고</center> |  
| :---: | :---: | :--- |  
| TCP | 179 | Calio BGP Network |  
| TCP | 2379-2380 | etcd server client API |  
| TCP | 6443 | Kubernetes API Server |  
| TCP | 10250 | Kubelet API |  
| TCP | 10251 | kube-scheduler |  
| TCP | 10252 | kube-controller-manager |  
| TCP | 10255 | Read-Only Kubelet API |  
| IP-in-IP (Protocol Num 4) || Calico Overlay Network |  

- Worker Node

| <center>프로토콜</center> | <center>포트</center> | <center>비고</center> |  
| :---: | :---: | :--- |  
| TCP | 179 | Calio BGP network |
| TCP | 10250 | Kubelet API |  
| TCP | 10255 | Read-Only Kubelet API |  
| TCP | 30000-32767 | NodePort Services |  
| IP-in-IP (Protocol Num 4) || Calico Overlay Network |  

<br>

### <div id='2.2'>2.2. Stemcell 확인
Stemcell 목록을 확인하여 서비스 설치에 필요한 Stemcell이 업로드 되어 있는 것을 확인한다. (K-PaaS 5.5 와 동일 Stemcell 사용)
- Stemcell 업로드 및 Cloud Config, Runtime Config 설정 부분은 [K-PaaS 5.5 설치가이드](https://github.com/PaaS-TA/application-platform-guide/blob/master/install/application_platform/paasta_ap.md)를 참고 한다.  

```
$ bosh -e micro-bosh stemcells
```

```
Using environment '10.0.1.6' as client 'admin'

Name                                     Version  OS             CPI  CID
bosh-aws-xen-hvm-ubuntu-xenial-go_agent  621.94   ubuntu-xenial  -    ami-0694eb07c57faca73

(*) Currently deployed

1 stemcells

Succeeded
```

<br>

### <div id='2.3'>2.3. Deployment 다운로드
서비스 설치에 필요한 Deployment를 Git Repository에서 받아 서비스 설치 작업 경로로 위치시킨다.   
- 컨테이너 플랫폼 Deployment Git Repository URL : <br> https://github.com/PaaS-TA/paas-ta-container-platform-deployment
```
# Deployment 다운로드 파일 위치 경로 생성 및 이동
$ mkdir -p ~/workspace/paasta-5.5/deployment/
$ cd ~/workspace/paasta-5.5/deployment/

# Deployment 다운로드
$ git clone https://github.com/PaaS-TA/paas-ta-container-platform-deployment.git -b v1.0.4
```

<br>

### <div id='2.4'>2.4. Deployment 파일 수정
BOSH Deployment manifest는 Components 요소 및 배포의 속성을 정의한 YAML 파일이다.<br> Deployment 파일에서 사용하는 network, vm_type, disk_type 등은 Cloud config를 활용하고, 활용 방법은 BOSH 2.0 가이드를 참고한다.<br>
일부 application의 경우 이중화를 위한 조치는 되어 있지 않으며 인스턴스 수 조정 시 신규로 생성되는 인스턴스에는 데이터의 반영이 안될 수 있으니, 1개의 인스턴스로 유지한다.

- Cloud config 설정 내용을 확인한다.
```
$ bosh -e micro-bosh cloud-config
```
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

<br>

- Deployment YAML에서 사용하는 변수 파일을 서버 환경에 맞게 수정한다.
  + IPS - k8s_api_server_ip : Kubernetes Master Node IP<br>
  + IPS - k8s_auth_bearer : [[6.1. Cluster Role 사용자 생성 및 Token 획득]](#6.1) 참고하여 Token 값 입력

#### paasta-container-service-vars-{IAAS}.yml
```
$ vi ~/workspace/paasta-5.5/deployment/paas-ta-container-platform-deployment/bosh/manifests/paasta-container-service-vars-{IAAS}.yml
(e.g. {IAAS} :: aws)
```
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

<br>

### <div id='2.5'>2.5. 서비스 설치
- 서버 환경에 맞추어 Deploy 스크립트 파일의 VARIABLES 설정을 수정한다.    
```
$ vi ~/workspace/paasta-5.5/deployment/paas-ta-container-platform-deployment/bosh/deploy-{IAAS}.sh  
(e.g. {IAAS} :: aws)
```
```    
#!/bin/bash

# SET VARIABLES
export CONTAINER_DEPLOYMENT_NAME='paasta-container-platform'   # deployment name
export CONTAINER_BOSH2_NAME='micro-bosh'                       # bosh name (e.g. micro-bosh)
export CONTAINER_BOSH2_UUID=`bosh int <(bosh -e ${CONTAINER_BOSH2_NAME} environment --json) --path=/Tables/0/Rows/0/uuid`

# DEPLOY
bosh -e ${CONTAINER_BOSH2_NAME} -n -d ${CONTAINER_DEPLOYMENT_NAME} deploy --no-redact manifests/paasta-container-service-deployment-aws.yml \
    -l manifests/paasta-container-service-vars-aws.yml \    
    -o manifests/ops-files/paasta-container-service/network-aws.yml \
    -o manifests/ops-files/misc/first-time-deploy.yml \
    -v deployment_name=${CONTAINER_DEPLOYMENT_NAME} \
    -v director_name=${CONTAINER_BOSH2_NAME} \
    -v director_uuid=${CONTAINER_BOSH2_UUID}
```
- 서비스를 설치한다.
```
$ cd ~/workspace/paasta-5.5/deployment/paas-ta-container-platform-deployment/bosh
$ chmod +x *.sh
$ ./deploy-{IAAS}.sh
```

<br>                                             

### <div id='2.6'>2.6. 서비스 설치 - 다운로드 된 릴리즈 파일 이용 방식

- 서비스 설치에 필요한 릴리즈 파일을 다운로드 받아 Local machine의 서비스 설치 작업 경로로 위치시킨다.  
  + 설치 릴리즈 파일 다운로드 :  
  [paasta-container-platform-1.0.tgz](https://nextcloud.k-paas.org/index.php/s/ggdZyEKejPSszFj/download)  

```
# 릴리즈 다운로드 파일 위치 경로 생성
$ mkdir -p ~/workspace/paasta-5.5/release/service
$ cd ~/workspace/paasta-5.5/release/service

# 릴리즈 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.k-paas.org/index.php/s/ggdZyEKejPSszFj/download
$ ls ~/workspace/paasta-5.5/release/service
  paasta-container-platform-1.0.tgz  
```
- 서버 환경에 맞추어 Deploy 스크립트 파일의 VARIABLES 설정을 수정한다.  
  (추가) -o manifests/ops-files/use-offline-releases.yml \
```
$ vi ~/workspace/paasta-5.5/deployment/paas-ta-container-platform-deployment/bosh/deploy-{IAAS}.sh  
(e.g. {IAAS} :: aws)
```
```    
#!/bin/bash

# SET VARIABLES
export CONTAINER_DEPLOYMENT_NAME='paasta-container-platform'   # deployment name
export CONTAINER_BOSH2_NAME='micro-bosh'                       # bosh name (e.g. micro-bosh)
export CONTAINER_BOSH2_UUID=`bosh int <(bosh -e ${CONTAINER_BOSH2_NAME} environment --json) --path=/Tables/0/Rows/0/uuid`

# DEPLOY
bosh -e ${CONTAINER_BOSH2_NAME} -n -d ${CONTAINER_DEPLOYMENT_NAME} deploy --no-redact manifests/paasta-container-service-deployment-aws.yml \
    -l manifests/paasta-container-service-vars-aws.yml \  
    -o manifests/ops-files/use-offline-releases.yml \  
    -o manifests/ops-files/paasta-container-service/network-aws.yml \
    -o manifests/ops-files/misc/first-time-deploy.yml \
    -v deployment_name=${CONTAINER_DEPLOYMENT_NAME} \
    -v director_name=${CONTAINER_BOSH2_NAME} \
    -v director_uuid=${CONTAINER_BOSH2_UUID}
```
- 서비스를 설치한다.
```
$ cd ~/workspace/paasta-5.5/deployment/paas-ta-container-platform-deployment/bosh
$ chmod +x *.sh
$ ./deploy-{IAAS}.sh
```

<br>

### <div id='2.7'>2.7. 서비스 설치 확인
설치 완료된 서비스를 확인한다.
```
$ bosh -e micro-bosh -d paasta-container-platform vms
```
```
Using environment '10.0.1.6' as client 'admin'

Task 2983. Done

Deployment 'paasta-container-platform'

Instance                                                       Process State  AZ  IPs           VM CID               VM Type  Active
haproxy/cbd5d103-765d-47e0-ac5b-233a21108c77                   running        z7  10.0.0.122    i-0f49ce7431aaa2901  small    true
                                                                                  15.164.15.53
mariadb/448be54d-f2ff-4fc9-8bf1-621eda8e2577                   running        z5  10.0.161.121  i-09b27b184b7aea066  small    true
private-image-repository/561550fb-95de-4c12-95bf-94ac5fde53cc  running        z7  10.0.0.123    i-02ff1da176d1d0a16  small    true

3 vms

Succeeded
```

<br>

### <div id='2.8'>2.8. CVE/CCE 진단항목 적용
배포된 Kubernetes Cluster, BOSH Inception 환경에 아래 가이드를 참고하여 해당 CVE/CCE 진단항목을 필수적으로 적용시켜야 한다.  
- [CVE/CCE 진단 가이드](/check-guide/cp-check-guide.md)

<br>

## <div id='3'>3. 컨테이너 서비스 배포
해당 [[3.컨테이너 서비스 배포]](#3) 항목은 배포된 Kubernetes Cluster 환경의 **Master Node**에서 진행한다. kubernetes에 PaaS-TA용 컨테이너 서비스를 배포하기 위해서는 Bosh 릴리즈를 통해 배포된 Private Repository에 이미지를 업로드하는 작업이 필요하다.

### <div id='3.1'>3.1. CRI-O insecure-registry 설정
컨테이너 플랫폼 Image Push, Pull 작업을 진행하기 위해 Kubernetes **Master Node, Worker Node** 내 podman 설치 및 config 파일에 'insecure-registries' 설정을 추가한다. <br>
Bosh 릴리즈를 통해 배포된 Private Repository를 'insecure-registries'로 설정 후 crio, podman을 재시작한다.<br>
 - {HAProxy_IP} 값은 BOSH Inception에 배포된 Deployment **'paasta-container-platform'** 의 haproxy public ip 입력
```
# podman 설치
$ sudo apt-get update
$ sudo apt-get install -y podman
```

```
# Master Node, Worker Node 모두 'insecure-registries' 설정 추가 필요
$ sudo vi /etc/crio/crio.conf
```

```
...
## 아래 항목을 추가한다
insecure_registries = [
    "{HAProxy_IP}:5001"
  ]
...
```

```
# crio 재시작
$ sudo systemctl restart crio
```

```
# Master Node, Worker Node 모두 'insecure-registries' 설정 추가 필요
$ sudo vi /etc/containers/registries.conf
```

```
...
## 아래 항목을 추가한다
[[registry]]
insecure = true
location = "{HAProxy_IP}:5001"
...
```

```
# podman 재시작
$ sudo systemctl restart podman
```

<br>

### <div id='3.2'>3.2. 컨테이너 서비스 이미지 업로드
Private Repository에 이미지 업로드를 위해 컨테이너 서비스 이미지 파일을 다운로드 받아 아래 경로로 위치시킨다.<br>
해당 내용은 Kubernetes **Master Node**에서 실행한다.

+ 컨테이너 서비스 이미지 파일 다운로드 :  
   [container-service-image-1.1.tar](https://nextcloud.k-paas.org/index.php/s/Fz3N5odb3yzoMFW/download)  

```
# 이미지 파일 다운로드 경로 생성
$ mkdir -p ~/workspace/paasta-5.5
$ cd ~/workspace/paasta-5.5

# 이미지 파일 다운로드 및 파일 경로 확인
$ wget --content-disposition https://nextcloud.k-paas.org/index.php/s/Fz3N5odb3yzoMFW/download

$ ls ~/workspace/paasta-5.5
  container-service-image-1.1.tar

# 이미지 파일 압축 해제
$ tar -xvf container-service-image-1.1.tar
```

- 이미지 파일 디렉토리 구성

```
$ cd ~/workspace/paasta-5.5/container-service

├── container-service-yaml                         # 컨테이너 서비스 배포 YAML 파일 위치
│   ├── container-service-api.yaml
│   ├── container-service-broker.yaml
│   ├── container-service-common-api.yaml
│   └── container-service-dashboard.yaml
├── jenkins-service-yaml                           # Jenkins 서비스 브로커 배포 YAML 파일 위치
│   └── container-jenkins-broker.yaml
├── service-image                                  # 컨테이너 및 Jenkins 서비스 관련 이미지 파일 위치
│   ├── container-jenkins-broker.tar.gz
│   ├── container-service-api.tar.gz
│   ├── container-service-broker.tar.gz
│   ├── container-service-common-api.tar.gz
│   ├── container-service-dashboard.tar.gz
│   └── paasta-jenkins.tar.gz
└── service-script                                 # 컨테이너 및 Jenkins 서비스 배포 관련 스크립트 파일 위치
    ├── container-service-deploy.sh
    ├── container-service-vars.sh
    ├── image-upload-caas.sh
    └── jenkins-service-deploy.sh
```

#### Private Repository 이미지 업로드

 + Private Repository에 이미지를 업로드하는 스크립트를 실행한다. <br>
   스크립트 실행 후 Private Repository에 이미지 업로드가 정상적으로 되었는지 확인한다.
 ```
 $ cd ~/workspace/paasta-5.5/container-service/service-script
 $ chmod +x *.sh  
 $ ./image-upload-caas.sh {HAProxy_IP}:5001

'''

*** List of uploaded images in the repository! ***

{"repositories":["container-jenkins-broker","container-service-api","container-service-broker","container-service-common-api","container-service-dashboard","paasta_jenkins"]}
 ```

 * Private Repository에 업로드 된 이미지 목록 확인 명령어
 ```
 $ curl -H 'Authorization:Basic YWRtaW46YWRtaW4=' http://{HAProxy_IP}:5001/v2/_catalog
 ```

<br>

### <div id='3.3'>3.3. 컨테이너 서비스 배포

#### <div id='3.3.1'>3.3.1. 컨테이너 서비스 배포 YAML 내 환경변수 정의
컨테이너 서비스를 배포하기 전 배포 Yaml 내 환경변수 값 정의가 필요하다. 배포에 필요한 정보를 확인하여 변수를 설정한다.

```
$ vi container-service-vars.sh
 ```

```                                                     
# COMMON ENV VARIABLE

PAASTA_SYSTEM_DOMAIN="xxx.xx.xx.xxx.nip.io"             # PaaS-TA System Domain
HAPROXY_IP="xxx.xx.xx.xxx"                              # Deployment 'paasta-container-platform' haproxy public ip
K8S_MASTER_NODE_IP="xxx.xx.xx.xxx"                      # Kubernetes master node public ip
K8S_WORKER_NODE_IP="xxx.xx.xx.xxx"                      # Kubernetes worker node public ip
K8S_WORKER_NODE_HOSTNAME="{k8s worker node hostname}"   # Kubernetes worker node host name (run 'hostname' command in worker node)
K8S_AUTH_BEARER_TOKEN="{k8s bearer token}"              # Kubernetes bearer token
MARIADB_USER_ID="{mariadb admin user id}"               # Mariadb admin user id (e.g. cp-admin)
MARIADB_USER_PASSWORD="{mariadb admin user password}"   # Mariadb admin user password (e.g. Paasta!2021)
```
- **PAASTA_SYSTEM_DOMAIN** <br> PaaS-TA System Domain 입력<br><br>
- **HAPROXY_IP** <br>BOSH Inception에 배포된 Deployment 'paasta-container-platform' 의 haproxy public ip 입력 <br><br>
- **K8S_MASTER_NODE_IP** <br>Kubernetes master node public ip 입력 <br><br>
- **K8S_WORKER_NODE_IP** <br>Kubernetes worker node public ip 입력 <br>
  + worker node가 2개 이상인 경우, 그 중 한 worker node의 public ip를 입력 &nbsp; :: ex)첫 번째 woker node의 public ip <br><br>
- **K8S_WORKER_NODE_HOSTNAME** <br>위 'K8S_WORKER_NODE_IP'에 입력한 woker node의 hostname 입력
   + 해당 worker node 접속 후 명령어 'hostname'으로 확인 <br><br>
- **K8S_AUTH_BEARER_TOKEN** <br>kubernetes bearer token 입력 <br>
   + [paasta-container-service-vars-{IAAS}.yml](#paasta-container-service-vars-iaasyml) 내 IPS - 'k8s_auth_bearer' 값 입력 <br><br>
- **MARIADB_USER_ID** <br>배포된 Deployment 'paasta-container-platform' 의 mariadb admin user id 입력 <br>
   + [paasta-container-service-vars-{IAAS}.yml](#paasta-container-service-vars-iaasyml) 내 MARIADB - 'mariadb_admin_user_id' 값 입력 <br><br>
- **MARIADB_USER_PASSWORD** <br>배포된 Deployment 'paasta-container-platform' 의 mariadb admin password 입력 <br>
   + [paasta-container-service-vars-{IAAS}.yml](#paasta-container-service-vars-iaasyml) 내 MARIADB - 'mariadb_admin_user_password' 값 입력 <br><br>

<br>

#### <div id='3.3.2'>3.3.2. 컨테이너 서비스 리소스 배포
컨테이너 서비스 배포를 위한 배포 스크립트를 실행한다.

```
$ ./container-service-deploy.sh
```

리소스 Pod의 경우 Node에 바인딩 및 컨테이너 생성 후 Running 상태로 전환되기까지 몇 초가 소요된다. <br>
스크립트 실행 후 아래 명령어를 통해 리소스가 정상적으로 배포되었는지 다시 한번 확인한다.

```
*** Create secret to pull an image from a repository! ***

secret/cp-secret created

NAME        TYPE                             DATA   AGE
cp-secret   kubernetes.io/dockerconfigjson   1      0s

*** Deployed Container Service Resource List ***

NAME                                                READY   STATUS    RESTARTS   AGE
pod/service-api-deployment-6d98766d5c-gh5kl         1/1     Running   0          30s
pod/service-broker-deployment-5dd6c6bbcb-v9d6n      1/1     Running   0          30s
pod/service-common-api-deployment-5d786869b-89zmn   1/1     Running   0          30s
pod/service-dashboard-deployment-974c87585-vmjf4    1/1     Running   0          30s

NAME                                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes                      ClusterIP   10.233.0.1      <none>        443/TCP          10d
service/service-api-deployment          NodePort    10.233.9.15     <none>        3333:30333/TCP   33s
service/service-broker-deployment       NodePort    10.233.60.232   <none>        8888:31888/TCP   34s
service/service-common-api-deployment   NodePort    10.233.4.60     <none>        3334:30334/TCP   33s
service/service-dashboard-deployment    NodePort    10.233.26.118   <none>        8091:32091/TCP   32s

NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/service-api-deployment          1/1     1            1           33s
deployment.apps/service-broker-deployment       1/1     1            1           34s
deployment.apps/service-common-api-deployment   1/1     1            1           33s
deployment.apps/service-dashboard-deployment    1/1     1            1           32s

NAME                                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/service-api-deployment-6d98766d5c         1         1         1       30s
replicaset.apps/service-broker-deployment-5dd6c6bbcb      1         1         1       30s
replicaset.apps/service-common-api-deployment-5d786869b   1         1         1       30s
replicaset.apps/service-dashboard-deployment-974c87585    1         1         1       30s
```
##### 배포된 리소스 조회 명령어

* Secret 조회
 ```
 $ kubectl get secret cp-secret
 ```

+ Deployment, ReplicaSet, Pod, Service 조회
 ```
 $ kubectl get all
 ```

<br>

## <div id='4'>4. 컨테이너 서비스 브로커
컨테이너 서비스 형태로 설치하는 경우에 CF와 배포된 Kubernetes와의 연동을 위해서는 Bosh Inception 환경에서 컨테이너 서비스 브로커를 등록해 주어야 한다.<br>K-PaaS 운영자 포털을 통해 서비스를 등록하고 공개하면, K-PaaS 사용자 포털을 통해 서비스를 신청하여 사용할 수 있다.

### <div id='4.1'>4.1. 컨테이너 서비스 브로커 등록

서비스 브로커 등록 시 개방형 클러스터 플랫폼에서 서비스 브로커를 등록할 수 있는 사용자로 로그인이 되어 있어야 한다.


- 서비스 브로커 목록을 확인한다.

```
$ cf service-brokers
Getting service brokers as admin...
No service brokers found
```

- 컨테이너 서비스 브로커를 등록한다.
```
$ cf create-service-broker container-service-broker admin cloudfoundry http://xxx.xxx.xxx.xxx:31888
```
> $ create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{Worker Node IP}:31888
> - 서비스팩 이름 : 서비스 팩 관리를 위해 개방형 클라우드 플랫폼에서 보여지는 명칭
> - 서비스팩 사용자 ID/비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID/비밀번호
> - 서비스팩 URL : Kubernetes Worker Node IP 와 배포된 컨테이너 서비스 브로커 NodePort
>   + Worker Node IP : [container-service-vars.sh](#3.3.1)에서 입력한 'K8S_WORKER_NODE_IP' 값을 입력한다.


- 등록된 컨테이너 서비스 브로커를 확인한다.
```
$ cf service-brokers
Getting service brokers as admin...

name                       url
container-service-broker   http://xxx.xxx.xxx.xxx:31888
```

- 접근 가능한 서비스 목록을 확인한다.

```
$ cf service-access
Getting service access as admin...

broker: container-service-broker
   offering            plan       access   orgs
   container-service   Advenced   none
   container-service   Micro      none
   container-service   Small      none
```

- 특정 조직에 해당 서비스 접근 허용을 할당한다.

```
$ cf enable-service-access container-service
Enabling access to all plans of service offering container-service for all orgs as admin...
OK
```

- 접근 가능한 서비스 목록을 확인한다.

```
$ cf service-access
Getting service access as admin...

broker: container-service-broker
   offering            plan       access   orgs
   container-service   Advenced   all
   container-service   Micro      all
   container-service   Small      all
```

<br>

### <div id='4.2'> 4.2. 컨테이너 서비스 UAA Client 등록
UAA 포털 계정 등록 절차에 대한 순서를 확인한다.

- uaac의 endpoint를 설정하고 uaac 로그인을 실행한다.
```
# endpoint 설정
$ uaac target https://uaa.<DOMAIN> --skip-ssl-validation

# target 확인
$ uaac target
Target: https://uaa.<DOMAIN>
Context: uaa_admin, from client uaa_admin

# uaac 로그인
$ uaac token client get <UAA_ADMIN_CLIENT_ID> -s <UAA_ADMIN_CLIENT_SECRET>
Successfully fetched token via client credentials grant.
Target: https://uaa.<DOMAIN>
Context: admin, from client admin
```

- 컨테이너 서비스 계정 생성을 한다.

> $ uaac client add caasclient -s {클라이언트 비밀번호} --redirect_uri  "http://{Worker Node IP}:32091, http://{Worker Node IP}:32091/callback" --scope {퍼미션 범위} --authorized_grant_types {권한 타입} --authorities={권한 퍼미션} --autoapprove={자동승인권한}
  - <클라이언트 비밀번호> : uaac 클라이언트 secret
    + common-vars.yml의 uaa_client_portal_secret값을 입력
  - <컨테이너 서비스 DashBoard URI> : 성공적으로 리다이렉션 할 컨테이너 서비스 접근 URI  (Kubernetes   Worker Node IP 와 배포된 컨테이너 서비스 대시보드 NodePort)
    +  Worker Node IP : [container-service-vars.sh](#3.3.1)에서 입력한 'K8S_WORKER_NODE_IP' 값을 입력
  - <퍼미션 범위> : 클라이언트가 사용자를 대신하여 얻을 수있는 허용 범위 목록  
  - <권한 타입> : 서비스가 제공하는 API를 사용할 수 있는 권한 목록  
  - <권한 퍼미션> : 클라이언트에 부여 된 권한 목록  
  - <자동승인권한> : 사용자 승인이 필요하지 않은 권한 목록

```  
# e.g. 컨테이너 서비스 계정 생성
$ uaac client add caasclient -s clientsecret --redirect_uri "http://xxx.xxx.xxx.xxx:32091, http://xxx.xxx.xxx.xxx:32091/callback" \
--scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
--authorized_grant_types "authorization_code , client_credentials , refresh_token" \
--authorities="uaa.resource" \
--autoapprove="openid , cloud_controller_service_permissions.read"


# e.g. 컨테이너 서비스 계정 생성 확인
$ uaac clients

  caasclient
    scope: cloud_controller.read cloud_controller.write
    cloud_controller_service_permissions.read openid cloud_controller.admin
    resource_ids: none
    authorized_grant_types: refresh_token client_credentials authorization_code
    redirect_uri: http://xxx.xxx.xxx.xxx:32091/callback http://xxx.xxx.xxx.xxx:32091
    autoapprove: cloud_controller_service_permissions.read openid
    authorities: uaa.resource
    name: caasclient
    lastmodified: 1612173849506
```  

-  uaac caasclient의 redirect_uri가  잘못 등록되어있다면 uaac client update를 통해 uri를 수정해야한다.
> $ uaac client update caasclient --redirect_uri "{http://{Worker Node IP}:32091}, {http://{Worker Node IP}:32091}/callback"

```
# e.g. caasclient redirect_uri update
$ uaac client update caasclient --redirect_uri "http://xxx.xxx.xxx.xxx:32091, http://xxx.xxx.xxx.xxx:32091/callback"
```

<br>

### <div id='4.3'>4.3. K-PaaS 포털에서 컨테이너 서비스 조회 설정

해당 설정은 K-PaaS 포털에 컨테이너 서비스 상의 자원들을 간략하게 조회하기 위한 설정이다.

1.K-PaaS Admin 포털에 접속한다.
![image 002]

2.왼쪽 네비게이션 바에서 [설정]-[설정정보] 페이지를 접속, 인프라 설정을 클릭 후 CaaS서비스 정보를 입력한다.
![image 003]


- 해당 정보를 입력하기 위해 필요한 값을 찾는다.
> - CaaS_Api_Uri : <br> http://{Worker Node IP}:30333 <br>
                   -  Kubernetes Worker Node IP 와 배포된 컨테이너 서비스 Api NodePort <br>
                   -  Worker Node IP : [container-service-vars.sh](#3.3.1)에서 입력한 'K8S_WORKER_NODE_IP' 값을 입력한다.
                   <br><br>
> - CaaS_Authorization : <br> Basic YWRtaW46UGFhUy1UQQ==


```
ex)
- CaaS_Api_Uri : http://{Worker Node IP}:30333
- CaaS_Authorization : Basic YWRtaW46UGFhUy1UQQ==
```
![image 004]


3.[운영관리]-[카탈로그] 메뉴에서 앱서비스 탭 안에 CaaS서비스를 선택한다.
![image 005]

4.서비스 항목을 'container-service' 로 선택, 공개 항목을 'Y' 로 체크 후 저장한다.
![image 006]

<br>

## <div id='5'>5. Jenkins 서비스 브로커(Optional)
해당 내용은 jenkins 서비스를 이용하기 위한 설정이다.

### <div id='5.1'>5.1. Jenkins 서비스 브로커 배포
해당 [[5.1. Jenkins 서비스 브로커 배포]](#5.1) 항목은 배포된 Kubernetes Cluster 환경의 **Master Node**에서 진행한다.<br>
Jenkins 서비스 브로커 배포를 위한 배포 스크립트를 실행한다.

```
$ cd ~/workspace/paasta-5.5/container-service/service-script
$ ./jenkins-service-deploy.sh
```

리소스 Pod의 경우 Node에 바인딩 및 컨테이너 생성 후 Running 상태로 전환되기까지 몇 초가 소요된다. <br>
스크립트 실행 후 아래 명령어를 통해 리소스가 정상적으로 배포되었는지 다시 한번 확인한다.

```
*** Deployed jenkins Service Resource List ***

NAME                                            READY   STATUS    RESTARTS   AGE
pod/jenkins-broker-deployment-94555d79f-rb2dl   1/1     Running   0          60s

NAME                                 TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/jenkins-broker-deployment    NodePort   10.233.22.46   <none>        8787:31787/TCP   60s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jenkins-broker-deployment   1/1     1            1           60s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/jenkins-broker-deployment-94555d79f   1         1         1       60s
```
##### 배포된 리소스 조회 명령어

+ Jenkins 서비스 브로커 관련 Deployment, ReplicaSet, Pod, Service 조회
 ```
 $ kubectl get all -l app=jenkins-broker
 ```

<br>

### <div id='5.2'>5.2. Jenkins 서비스 브로커 등록

- 브로커 목록을 확인한다.

```
$ cf service-brokers
Getting service brokers as admin...

name                       url
container-service-broker   http://xxx.xxx.xxx.xxx:31888
```
- Jenkins 서비스 브로커를 등록한다.

```
$ cf create-service-broker jenkins-service-broker admin cloudfoundry http://xxx.xxx.xxx.xxx:31787
Creating service broker jenkins-service-broker as admin...
OK
```

> $ create-service-broker {서비스팩 이름} {서비스팩 사용자ID} {서비스팩 사용자비밀번호} http://{Worker Node IP}:31787
> - 서비스팩 이름 : 서비스 팩 관리를 위해 개방형 클라우드 플랫폼에서 보여지는 명칭
> - 서비스팩 사용자 ID/비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID/비밀번호
> - 서비스팩 URL : Kubernetes Worker Node IP 와 배포된 Jenkins 서비스 브로커 NodePort
>   + Worker Node IP : [container-service-vars.sh](#3.3.1)에서 입력한 'K8S_WORKER_NODE_IP' 값을 입력한다.


- 등록된 Jenkins 서비스 브로커를 확인한다.
```
$ cf service-brokers
Getting service brokers as admin...

name                       url
container-service-broker   http://xxx.xxx.xxx.xxx:31888
jenkins-service-broker     http://xxx.xxx.xxx.xxx:31787
```

- 접근 가능한 서비스 목록을 확인한다.
```
$ cf service-access
Getting service access as admin...

broker: container-service-broker
   offering            plan       access   orgs
   container-service   Advenced   all
   container-service   Micro      all
   container-service   Small      all

broker: jenkins-service-broker
   offering                    plan           access   orgs
   container-jenkins-service   jenkins_20GB   none
```

- 특정 조직에 해당 서비스 접근 허용을 할당한다.

```
$ cf enable-service-access container-jenkins-service
Enabling access to all plans of service offering container-jenkins-service for all orgs as admin...
OK
```

- 접근 가능한 서비스 목록을 확인한다.
```
$ cf service-access
Getting service access as admin...

broker: container-service-broker
   offering            plan       access   orgs
   container-service   Advenced   all
   container-service   Micro      all
   container-service   Small      all

broker: jenkins-service-broker
   offering                    plan           access   orgs
   container-jenkins-service   jenkins_20GB   all
```

<br>

### <div id='5.3'>5.3. K-PaaS 포털에서 Jenkins 서비스 조회 설정

1.K-PaaS Admin 포털에 접속한다.
![image 002]

2.[운영관리]-[카탈로그] 메뉴에서 앱서비스 탭 안에 CaaS Jenkins 서비스를 선택한다.
![image 007]

3.서비스 항목을 'container-jenkins-service' 로 선택, 공개 항목을 'Y' 로 체크 후 저장한다.
![image 008]

<br>


## <div id='6'>6. 참고

### <div id='6.1'>6.1. Cluster Role 사용자 생성 및 Token 획득
Kubernetes에서 Cluster Role을 가진 사용자의 Service Account를 생성하고 해당 Service Account의 Token 값을 획득한다.<br>
획득한 Token 값은 컨테이너 서비스 설치에 사용된다.

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

----
[image 001]:images/cp-001.png
[image 002]:images/cp-002.png
[image 003]:images/cp-020.png
[image 004]:images/cp-021.png
[image 005]:images/cp-022.png
[image 006]:images/cp-023.png
[image 007]:images/cp-024.png
[image 008]:images/cp-025.png
