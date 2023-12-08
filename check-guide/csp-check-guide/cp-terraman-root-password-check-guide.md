### [Index](https://github.com/K-PaaS/Guide/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > [Terraman IaC Index](../cp-terraman-check-index-guide.md) > Terraman IaC 스크립트 가이드

## Table of Contents

1. [문서 개요](#1)  
	1.1. [목적](#1.1)  
	1.2. [범위](#1.2)  
	1.3. [참고자료](#1.3)
2. [Prerequisite](#2)  
   	2.1. [방화벽 정보](#2.1)
3. [SSH Key 설정](#3)  
	3.1 [SSH Key 구성](#3.1)  
	3.2 [SSH Key 복사](#3.2)  
4. [Template 생성](#4)  
  4.1 [NAVER](#4.1)  
	

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
 Terraman IaC 스크립트 가이드는 Terraform을 이용하여 Sub Cluster를 생성하기 위한 각 IaaS별 HCL(Hashicorp Configuration Language) 구문을 설명하여, 사용자가 실제 배포할 스크립트를 작성하는 데 도움을 주기 위한 목적으로 제작되었다.
### <div id='1.2'> 1.2. 범위
Kubernetes Cluster를 배포하는 것을 기준으로 작성되었다.
### <div id='1.3'> 1.3. 참고자료
- Terraform NAVER
> https://registry.terraform.io/providers/NaverCloudPlatform/ncloud/latest/docs

## <div id='2'> 2. Prerequisite
- Terraman을 실행하기 전 필요한 사전 작업에 대한 설명이다.
### <div id='2.1'> 2.1 방화벽 정보
- Master Node

|프로토콜|포트|비고|
|---|---|---|
|TCP|22|SSH|
|TCP|6443|Kubernetes API Server|
- IaaS 별 API 방화벽 오픈  
    (예: OpenStack API - 8000, 8774, 5000, 9292, 9876, 9696, 8004, 8780, 8776)
- 각 IaaS에서 생성되는 Instance는 원격 접속을 위한 포트가 열려 있어야 한다.

## <div id='3'> 3. SSH Key 설정
#### 들어가기 전
- **Host Cluster**: Terraform을 생성하는 데 사용되는 클러스터
- **Sub Cluster**: Terraman을 통해 배포되는 클러스터

### <div id='3.1'> 3.1 SSH Key 구성
- **id_rsa**, **id_rsa.pub**
- Host Cluster 구성 시 생성된 RSA Key
>NAVER인 경우 Sub Cluster 접속시 Root Password 방식이며 Terraman API에서 Sub Cluster 접속 정보를 자동 조회 및 생성한다.

### <div id='3.2'>3.2 SSH Key 복사
- SSH Key 목적
	- Terraman 실행 중에 Host Cluster 및 Sub Cluster의 각 Master 노드에 액세스하기 위함이다.
- 각 공개키를 Terraman이 실행되는 Terraman Pod로 복사한다.
	- Host Cluster의 Master 키 이름은 *반드시 'master-key'* 로 복사 되어야 한다.
```bash
## Host Cluster Key
$ kubectl cp /home/ubuntu/.ssh/id_rsa {TERRAMAN_PDO_NAME}:/home/1000/.ssh/master-key -n cp-portal
```
*예시* 
```bash
## Terraman Pod Name 조회
$ kubectl get pods -n cp-portal
NAME                                               READY   STATUS    RESTARTS     AGE
cp-portal-api-deployment-6485fc78d9-48cdl          1/1     Running   0            9d
cp-portal-common-api-deployment-58b57cd8fb-txtdn   1/1     Running   0            9d
cp-portal-metric-api-deployment-757b98d55c-v4psb   1/1     Running   3 (9d ago)   9d
cp-portal-terraman-deployment-689585bc94-pq4bt     1/1     Running   0            9d
cp-portal-ui-deployment-74d744cff4-qh7g2           1/1     Running   0            9d

## Host Cluster Key Name: id_rsa
$ kubectl cp /home/ubuntu/.ssh/id_rsa cp-portal-terraman-deployment-689585bc94-pq4bt:/home/1000/.ssh/master-key -n cp-portal
```

## <div id='4'> 4. Template 생성
### <div id='4.1'> 4.1 NAVER
- 이 Template는 Terraman을 사용하여 NAVER에서 인스턴스를 생성하는 방법을 설명한다. 기본 Template는 인스턴스 생성에 초점을 맞추고 있다.

```
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
  subnet         = cidrsubnet(ncloud_vpc.vpc_scn_01.ipv4_cidr_block, 8, 1)  # 10.0.1.0/24
  zone           = "KR-1"     # Available zone where the subnet will be placed physically.
  network_acl_no = ncloud_vpc.vpc_scn_01.default_network_acl_no   # The ID of Network ACL.
  subnet_type    = "PUBLIC"   # PUBLIC(Public) | PRIVATE(Private)
}

## Provides a Network Interface resource
resource "ncloud_network_interface" "nic01" {
  name                  = "server-nic1"
  description           = "for server-nic"
  subnet_no             = ncloud_subnet.subnet_scn_01.id     # The ID of the associated Subnet.
  access_control_groups = [ncloud_access_control_group.acg_scn_01.id]    # List of ACG ID to apply to network interfaces. A maximum of three ACGs can be applied.
}

resource "ncloud_network_interface" "nic02" {
  name                  = "server-nic2"
  description           = "for server-nic"
  subnet_no             = ncloud_subnet.subnet_scn_01.id
  access_control_groups = [ncloud_access_control_group.acg_scn_01.id]
}

resource "ncloud_network_interface" "nic03" {
  name                  = "server-nic3"
  description           = "for server-nic"
  subnet_no             = ncloud_subnet.subnet_scn_01.id
  access_control_groups = [ncloud_access_control_group.acg_scn_01.id]
}

resource "ncloud_network_interface" "nic04" {
  name                  = "server-nic4"
  description           = "for server-nic"
  subnet_no             = ncloud_subnet.subnet_scn_01.id
  access_control_groups = [ncloud_access_control_group.acg_scn_01.id]
}

## Provides a Server instance resource.
resource "ncloud_server" "server_01" {
  subnet_no                 = ncloud_subnet.subnet_scn_01.id
  name                      = var.server_name01
  server_image_product_code = "SW.VSVR.OS.LNX64.UBNTU.SVR2004.B050"  # Server image product code to determine which server image to create. It can be obtained through data.ncloud_server_image(s)
  login_key_name            = ncloud_login_key.key_scn_01.key_name
  description = "master"
  network_interface {                 # List of Network Interface. You can assign up to three network interfaces.
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

resource "ncloud_server" "server_03" {
  subnet_no                 = ncloud_subnet.subnet_scn_01.id
  name                      = var.server_name03
  server_image_product_code = "SW.VSVR.OS.LNX64.UBNTU.SVR2004.B050"
  login_key_name            = ncloud_login_key.key_scn_01.key_name
  network_interface {
    network_interface_no = ncloud_network_interface.nic03.id
    order                = 0
  }
}

resource "ncloud_server" "server_04" {
  subnet_no                 = ncloud_subnet.subnet_scn_01.id
  name                      = var.server_name04
  server_image_product_code = "SW.VSVR.OS.LNX64.UBNTU.SVR2004.B050"
  login_key_name            = ncloud_login_key.key_scn_01.key_name
  description = "nfs"
  network_interface {
    network_interface_no = ncloud_network_interface.nic04.id
    order                = 0
  }
}

## Provides a Public IP instance resource.
resource "ncloud_public_ip" "public_ip_01" {
  server_instance_no = ncloud_server.server_01.id
  description        = "for ${var.server_name01}"
}

resource "ncloud_public_ip" "public_ip_02" {
  server_instance_no = ncloud_server.server_02.id
  description        = "for ${var.server_name02}"
}

resource "ncloud_public_ip" "public_ip_03" {
  server_instance_no = ncloud_server.server_03.id
  description        = "for ${var.server_name03}"
}

resource "ncloud_public_ip" "public_ip_04" {
  server_instance_no = ncloud_server.server_04.id
  description        = "for ${var.server_name04}"
}

## priority, protocol, ip_block, port_range, rule_action for networkACL
locals {
  scn01_inbound = [
    [1, "TCP", "0.0.0.0/0", "80", "ALLOW"],
    [2, "TCP", "0.0.0.0/0", "443", "ALLOW"],
    [3, "TCP", "${var.client_ip}/32", "22", "ALLOW"],
    [4, "TCP", "${var.client_ip2}/32", "22", "ALLOW"],
    [5, "TCP", "${var.client_ip3}/32", "22", "ALLOW"],
    [6, "TCP", "0.0.0.0/0", "111", "ALLOW"],
    [7, "TCP", "0.0.0.0/0", "2049", "ALLOW"],
    [8, "TCP", "0.0.0.0/0", "6443", "ALLOW"],
    [9, "TCP", "0.0.0.0/0", "2379-2380", "ALLOW"],
    [10, "TCP", "0.0.0.0/0", "10250-10255", "ALLOW"],
    [11, "UDP", "0.0.0.0/0", "4789", "ALLOW"],
    [12, "TCP", "0.0.0.0/0", "30000-32767", "ALLOW"],
    [197, "TCP", "0.0.0.0/0", "1-65535", "ALLOW"],
    [198, "UDP", "0.0.0.0/0", "1-65535", "ALLOW"],
    [199, "ICMP", "0.0.0.0/0", null, "ALLOW"],
  ]

  scn01_outbound = [
    [1, "TCP", "0.0.0.0/0", "80", "ALLOW"],
    [2, "TCP", "0.0.0.0/0", "443", "ALLOW"],
    [3, "TCP", "${var.client_ip}/32", "1000-65535", "ALLOW"],
    [4, "TCP", "${var.client_ip2}/32", "1000-65535", "ALLOW"],
    [5, "TCP", "${var.client_ip3}/32", "1000-65535", "ALLOW"],
    [6, "TCP", "0.0.0.0/0", "30000-32767", "ALLOW"],
    [7, "UDP", "0.0.0.0/0", "30000-32767", "ALLOW"],
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
   ["TCP", "${var.client_ip2}/32", "22"],
   ["TCP", "${var.client_ip3}/32", "22"],
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
  description = "Access to Cloud2"
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