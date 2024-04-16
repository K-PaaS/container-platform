### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](https://github.com/K-PaaS/container-platform/blob/master/use-guide/Readme.md) > [Terraman 사용 가이드](../cp-terraman-guide.md) > Terraman 배포 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
   1.1. [목적](#1.1)  
   1.2. [범위](#1.2)  
   1.3. [참고자료](#1.3)
2. [Prerequisite](#2)  
   2.1. [방화벽 정보](#2.1)
3. [클러스터 배포 프로세스](#3)
4. [Cloud Accounts](#4)  
   4.1. [Cloud Accounts 등록](#4.1)  
　4.1.1. [NAVER](#4.1.1)
5. [Instance Code Template](#5)  
   5.1. [Instance Code Template 예제](#5.1)  
  　5.1.1. [NAVER](#5.1.1)   
   5.2. [Instance Code Template 등록](#5.2)
6. [Clusters](#6)  
   6.1. [Clusters 생성](#6.1)

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서(Terraman 배포 가이드)는 **OpenTofu**를 이용하여 서브 클러스터를 생성할 수 있도록 각 IaaS별 **HCL(Hashicorp Configuration Language)** 구문을 설명하고, 사용자가 **IaC(Infrastructure as Code)** 코드를 작성하여 서브 클러스터를 배포하는데 도움을 주기 위한 목적으로 작성되었다.

|클러스터|설명|
|---|---|
|호스트 클러스터(Host Cluster)|컨테이너플랫폼의 Kubernetes 메인 클러스터|
|서브 클러스터(Sub Cluster) |컨테이너 플랫폼 포털을 통해 신규 생성되거나, 등록된 관리 클러스터|

### <div id='1.2'> 1.2. 범위
Kubernetes Cluster를 배포하는 것을 기준으로 작성되었다.

### <div id='1.3'> 1.3. 참고자료
**Naver Cloud Platform**
> [Ncloud Provider](https://registry.terraform.io/providers/NaverCloudPlatform/ncloud/latest/docs)

<br>

## <div id='2'> 2. Prerequisite
K-PaaS 컨테이너 플랫폼 클러스터, 포털 배포가 사전에 진행되어야한다.
> [[K-PaaS 컨테이너 플랫폼 클러스터 설치 가이드]](../../../install-guide/standalone/cp-cluster-install-single.md) <br>
> [[K-PaaS 컨테이너 플랫폼 포털 배포 가이드]](../../../install-guide/portal/cp-portal-standalone-guide.md)

### <div id='2.1'> 2.1 방화벽 정보
- Master Node

|프로토콜|포트|비고|
|---|---|---|
|TCP|22|SSH|
|TCP|6443|Kubernetes API Server|
- IaaS 별 API 방화벽 오픈  
  (예시: OpenStack API - 8000, 8774, 5000, 9292, 9876, 9696, 8004, 8780, 8776)
- 각 IaaS에서 생성되는 Instance는 원격 접속을 위한 포트가 열려 있어야 한다.

  <br>
## <div id='3'> 3. 클러스터 배포 프로세스
Terraman을 통한 `Root Password` 방식의 클러스터 배포는 **Cloud Accounts** 설정을 통해 **Instance Code Template**을 활용하여 서브 클러스터를 배포한다.
1. [Cloud Accounts 등록](#4)
2. [Instance Code Template 등록 또는 기본 IaC 코드 활용](#5)
3. [Clusters 배포](#6)

<img src="../../images/terraman/IMG_3_NAVER.png">

<br>

## <div id='4'> 4. Cloud Accounts
### <div id='4.1'> 4.1 Cloud Accounts 등록
컨테이너 플랫폼 포털 메뉴 Global > Cloud Accounts에서 Cloud Accounts 정보를 등록한다.

#### <div id='4.1.1'> 4.1.1 Naver
아래 항목의 Ncloud 정보를 입력한다.
- 네이버 클라우드 플랫폼 콘솔의 마이페이지 > 계정 관리 > 인증키 관리에서 인증키 생성, 관리 정보를 확인할 수 있다.
- Ncloud Site 값은 기본적으로 'public'이며, 사용하고 있는 Ncloud 도메인에 따라 'public', 'gov', 'fin' 중 선택하여 입력한다.

|도메인|입력 값|
|:------:|:------:|
|www.ncloud.com|public|
|www.gov-ncloud.com|gov|
|www.fin-ncloud.com|fin|

<br>

|항목|Ncloud 정보|정보 위치|     
|:------:|:------:|:------:|
|accessKey 필드|Access Key|마이페이지 > 계정 관리 > 인증키 관리|
|secretKey 필드|Secret Key|마이페이지 > 계정 관리 > 인증키 관리|
|site 필드|Site|[가이드 참고](https://registry.terraform.io/providers/NaverCloudPlatform/ncloud/latest/docs#argument-reference)|
|region 필드|Region|대시보드 확인[(리전 이름 가이드 참고)](https://guide.ncloud-docs.com/docs/ko/environment-environment-1-1#%EC%A1%B4zone%EC%9D%98-%EC%A2%85%EB%A5%98%EC%99%80-%ED%8A%B9%EC%A7%95)|

<img src="../../images/terraman/IMG_4_1_1_NAVER.png">

<br>

## <div id='5'> 5. Instance Code Template
> 기본적으로 인스턴스 생성에 필요한 Code Template은 각 CSP당 한 종류씩 등록되어 있다. 추가로 필요한 Code Template은 신규로 등록하여 클러스터 생성 시 사용이 가능하다.

### <div id='5.1'> 5.1 Instance Code Template 예제
#### <div id='5.1.1'> 5.1.1 Naver
Terraman을 사용하여 Ncloud에서 인스턴스를 생성하는 방법을 설명한다. 기본 IaC 코드는 인스턴스 생성에 초점을 맞추고 있다.
> [Ncloud IaC 코드 작성시 변수 참고](https://registry.terraform.io/providers/NaverCloudPlatform/ncloud/latest/docs#argument-reference)
- 인스턴스 생성시 "master"와 "worker" 명칭을 반드시 표기해야 한다.  
  *예시*
  + 인스턴스 1개 생성시
    - resource "ncloud_server" "master" {...}
  + 인스턴스 n개 생성시
    - resource "ncloud_server" "master" {...}
    - resource "ncloud_server" "worker1" {...}
    - resource "ncloud_server" "worker2" {...}
    - resource "ncloud_server" "worker3" {...} ...

<details>
<summary> :page_facing_up: <b>Ncloud Instance Code Template</b> </summary>
<div markdown="1">

```
variable server_name01 {
  default = "cp-master"
}

variable server_name02 {
  default = "cp-worker"
}

variable client_ip {
  default = "x.x.x.x"                                                                               #client ip 지정
}

## Provides a Login key resource.
resource "ncloud_login_key" "key_scn_01" {
  key_name = var.server_name01
}

## Provides a VPC resource.
resource "ncloud_vpc" "vpc_scn_01" {
  name            = var.server_name01
  ipv4_cidr_block = "10.0.0.0/16"
}

## Provides a Subnet resource.
resource "ncloud_subnet" "subnet_scn_01" {
  name           = var.server_name01
  vpc_no         = ncloud_vpc.vpc_scn_01.id
  subnet         = cidrsubnet(ncloud_vpc.vpc_scn_01.ipv4_cidr_block, 8, 1)                          # 10.0.1.0/24
  zone           = "KR-1"                                                                           # Available zone where the subnet will be placed physically.
  network_acl_no = ncloud_vpc.vpc_scn_01.default_network_acl_no                                     # The ID of Network ACL.
  subnet_type    = "PUBLIC"                                                                         # PUBLIC(Public) | PRIVATE(Private)
}

## Provides a Network Interface resource
resource "ncloud_network_interface" "nic01" {
  name                  = "server-nic1"
  description           = "for server-nic"
  subnet_no             = ncloud_subnet.subnet_scn_01.id                                            # The ID of the associated Subnet.
  access_control_groups = [ncloud_access_control_group.acg_scn_01.id]                               # List of ACG ID to apply to network interfaces. A maximum of three ACGs can be applied.
}

 resource "ncloud_network_interface" "nic02" {
 name                  = "server-nic2"
 description           = "for server-nic"
 subnet_no             = ncloud_subnet.subnet_scn_01.id
 access_control_groups = [ncloud_access_control_group.acg_scn_01.id]
 }

## Provides a Server instance resource.
resource "ncloud_server" "server_01_master" {                                                       # 인스턴스 생성시 반드시 "master"와 "worker" 명칭으로 구분
  subnet_no                 = ncloud_subnet.subnet_scn_01.id
  name                      = var.server_name01
  server_image_product_code = "SW.VSVR.OS.LNX64.UBNTU.SVR2004.B050"                                 # Server image product code to determine which server image to create. It can be obtained through data.ncloud_server_image(s)
  login_key_name            = ncloud_login_key.key_scn_01.key_name
  description = "master"
  network_interface {                                                                               # List of Network Interface. You can assign up to three network interfaces.
    network_interface_no = ncloud_network_interface.nic01.id
    order                = 0
  }
}

resource "ncloud_server" "server_02" {
  subnet_no                 = ncloud_subnet.subnet_scn_01.id
  name                      = var.server_name02
  server_image_product_code = "SW.VSVR.OS.LNX64.UBNTU.SVR2004.B050"
  login_key_name            = ncloud_login_key.key_scn_01.key_name
  network_interface {
    network_interface_no = ncloud_network_interface.nic02.id
    order                = 0
  }
}

## Provides a Public IP instance resource.
resource "ncloud_public_ip" "public_ip_01" {
  server_instance_no = ncloud_server.server_01_master.id
  description        = "for ${var.server_name01}"
}

resource "ncloud_public_ip" "public_ip_02" {
  server_instance_no = ncloud_server.server_02.id
  description        = "for ${var.server_name02}"
}

## priority, protocol, ip_block, port_range, rule_action for networkACL
locals {
  scn01_inbound = [
    [1, "TCP", "0.0.0.0/0", "80", "ALLOW"],
    [2, "TCP", "0.0.0.0/0", "443", "ALLOW"],
    [3, "TCP", "${var.client_ip}/32", "22", "ALLOW"],
    [4, "TCP", "0.0.0.0/0", "111", "ALLOW"],
    [5, "TCP", "0.0.0.0/0", "2049", "ALLOW"],
    [6, "TCP", "0.0.0.0/0", "6443", "ALLOW"],
    [7, "TCP", "0.0.0.0/0", "2379-2380", "ALLOW"],
    [8, "TCP", "0.0.0.0/0", "10250-10255", "ALLOW"],
    [9, "UDP", "0.0.0.0/0", "4789", "ALLOW"],
    [10, "TCP", "0.0.0.0/0", "30000-32767", "ALLOW"],
    [197, "TCP", "0.0.0.0/0", "1-65535", "ALLOW"],
    [198, "UDP", "0.0.0.0/0", "1-65535", "ALLOW"],
    [199, "ICMP", "0.0.0.0/0", null, "ALLOW"],
  ]

  scn01_outbound = [
    [1, "TCP", "0.0.0.0/0", "80", "ALLOW"],
    [2, "TCP", "0.0.0.0/0", "443", "ALLOW"],
    [3, "TCP", "${var.client_ip}/32", "1000-65535", "ALLOW"],
    [4, "TCP", "0.0.0.0/0", "30000-32767", "ALLOW"],
    [5, "UDP", "0.0.0.0/0", "30000-32767", "ALLOW"],
    [197, "TCP", "0.0.0.0/0", "1-65535", "ALLOW"],
    [198, "UDP", "0.0.0.0/0", "1-65535", "ALLOW"],
    [199, "ICMP", "0.0.0.0/0", null, "ALLOW"]
  ]
}

## Provides a rule of Network ACL resource. 
resource "ncloud_network_acl_rule" "network_acl_01_rule" {
  network_acl_no = ncloud_vpc.vpc_scn_01.default_network_acl_no
  dynamic "inbound" {
    for_each = local.scn01_inbound
    content {
      priority    = inbound.value[0]
      protocol    = inbound.value[1]
      ip_block    = inbound.value[2]
      port_range  = inbound.value[3]
      rule_action = inbound.value[4]
      description = "for ${var.server_name01}"
    }
  }

  dynamic "outbound" {
    for_each = local.scn01_outbound
    content {
      priority    = outbound.value[0]
      protocol    = outbound.value[1]
      ip_block    = outbound.value[2]
      port_range  = outbound.value[3]
      rule_action = outbound.value[4]
      description = "for ${var.server_name01}"
    }
  }
}

## protocol, ip_loack, port_range for ACG
locals {
  default_acg_rules_inbound = [
    ["TCP", "0.0.0.0/0", "80"],
    ["TCP", "0.0.0.0/0", "443"],
    ["TCP", "0.0.0.0/0", "111"],
    ["TCP", "0.0.0.0/0", "2049"],
    ["TCP", "0.0.0.0/0", "2379-2380"],
    ["TCP", "0.0.0.0/0", "6443"],
    ["TCP", "0.0.0.0/0", "10250-10255"],
    ["TCP", "0.0.0.0/0", "30000-32767"],
    ["UDP", "0.0.0.0/0", "4789"],
    ["TCP", "10.0.1.0/24", "22"],
    ["TCP", "${var.client_ip}/32", "22"],
    ["TCP", "0.0.0.0/0", "1-65535"],
    ["UDP", "0.0.0.0/0", "1-65534"],
    ["ICMP", "0.0.0.0/0", null]
  ]

  default_acg_rules_outbound = [
    ["TCP", "0.0.0.0/0", "1-65535"],
    ["UDP", "0.0.0.0/0", "1-65534"],
    ["ICMP", "0.0.0.0/0", null]
  ]
}

## Provides an ACG(Access Control Group) resource.
resource "ncloud_access_control_group" "acg_scn_01" {
  vpc_no      = ncloud_vpc.vpc_scn_01.id
}

## Provides an rule of ACG(Access Control Group) resource.
resource "ncloud_access_control_group_rule" "acg_rule_scn_01" {
  access_control_group_no = ncloud_access_control_group.acg_scn_01.id

  dynamic "inbound" {
    for_each = local.default_acg_rules_inbound
    content {
      protocol    = inbound.value[0]
      ip_block    = inbound.value[1]
      port_range  = inbound.value[2]
    }
  }

  dynamic "outbound" {
    for_each = local.default_acg_rules_outbound
    content {
      protocol    = outbound.value[0]
      ip_block    = outbound.value[1]
      port_range  = outbound.value[2]
    }
  }
}
```
</div>
</details>

<br>

### <div id='5.2'> 5.2 Instance Code Template 등록
컨테이너 플랫폼 포털 메뉴 Global > Instance Code Template 에서 IaC 코드를 등록한다.
<img src="../../images/terraman/IMG_5_2_NAVER.png">

<br>

## <div id='6'> 6. Clusters
### <div id='6.1'> 6.1 Clusters 생성
컨테이너 플랫폼 포털 메뉴  Global > Clusters 에서 Cluster 생성을 위한 아래 항목을 입력한다.

|항목|설명|
|---|---|
|Cluster Name|생성할 클러스터의 이름|
|Provider|CSP 선택(AWS, OpenStack, NHN Cloud, Naver Cloud)|
|Cloud Account|CSP 별 등록한 계정정보를 선택| 
|Template|등록된 Code Template 선택| 
|Description|생성할 클러스터의 세부내용 기입(Option)|
|Template Detail|선택한 Code Template이 표시되고, 수정 가능함|

<img src="../../images/terraman/IMG_7_1_1.png">

##### Cluster 생성시 Terraman API에 의해서 서브 클러스터 생성이 진행되며 우측 status 항목을 통해 클러스터 상태 확인이 가능한다.
<img src="../../images/terraman/IMG_6_1_2_NAVER.png">

##### Cluster Logs 목록 페이지(status 아이콘 클릭)에서 클러스터 생성의 진행 상황을 로그로 확인 가능하다.
<img src="../../images/terraman/IMG_6_1_3_NAVER.png">

##### 클러스터 배포가 완료되면 화면과 같이 status의 상태 아이콘이 녹색으로 변경된다.
<img src="../../images/terraman/IMG_6_1_4_NAVER.png">

##### 클러스터 배포가 완료되면 아래와 같이 Overview 페이지에 서브 클러스터로 등록 된 것을 확인할 수 있다.
<img src="../../images/terraman/IMG_6_1_5_NAVER.png">

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](https://github.com/K-PaaS/container-platform/blob/master/use-guide/Readme.md) > [Terraman 사용 가이드](../cp-terraman-guide.md) > Terraman 배포 가이드