### [Index](https://github.com/K-PaaS/Guide/blob/master/README.md) > [CP Install](https://github.com/K-PaaS/container-platform/blob/master/install-guide/Readme.md) > [Terraman IaC Index](../cp-terraman-check-index-guide.md) > Terraman IaC 스크립트 가이드

## Table of Contents

1. [문서 개요](#1)  
 1.1. [목적](#1.1)  
 1.2. [범위](#1.2)  
 1.3. [참고자료](#1.3)
2. [Prerequisite](#2)  
 2.1. [방화벽 정보](#2.1)
3. [SSH Key 설정](#3)  
 3.1. [SSH Key 생성](#3.1)  
　3.1.1. [Terraman을 통한 SubCluster 배포를 위한 SSH Key 생성](#3.1.1)   
　3.1.2. [생성된 공개키는 해당 IaaS에 keypair로 등록](#3.1.2)   
 3.2 [SSH Key 구성](#3.2)  
 3.3 [SSH Key 복사](#3.3)  
4. [Template 생성](#4)  
 4.1 [Openstack](#4.1)  
 4.2 [AWS](#4.2)  
 4.3 [NHN](#4.3)

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
Terraman IaC 스크립트 가이드는 OpenTofu를 이용하여 Sub Cluster를 생성하기 위한 각 IaaS별 HCL(Hashicorp Configuration Language) 구문을 설명하여, 사용자가 실제 배포할 스크립트를 작성하는 데 도움을 주기 위한 목적으로 제작되었다.
### <div id='1.2'> 1.2. 범위
Kubernetes Cluster를 배포하는 것을 기준으로 작성되었다.
### <div id='1.3'> 1.3. 참고자료
- OpenStack
> https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs  
- AWS
> https://registry.terraform.io/providers/hashicorp/aws/latest/docs
- NHN
> https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs 

## <div id='2'> 2. Prerequisite
- Terraman을 실행하기 전 필요한 사전 작업에 대한 설명이다.
#### <div id='2.1'> 2.1 방화벽 정보
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
- **Host Cluster**: OpenTofu를 생성하는 데 사용되는 클러스터
- **Sub Cluster**: Terraman을 통해 배포되는 클러스터

### <div id='3.1'>  3.1. SSH Key 생성
#### <div id='3.1.1'> 3.1.1 Terraman을 통한 SubCluster 배포를 위한 SSH Key 생성
- 변수 {TERRAMAN_MASTER_NODE_NAME}은 Container Platform Portal Cluster 등록시 Cluster Name과 동일해야한다. 즉, Cluster 등록할 때 Cluster Name과 SSH Key 생성할 때 이름이 동일해야한다.  
<kbd>
  <img src="../images/IMG_1_1.png">
</kbd>

<br>

```bash
$ ssh-keygen -t rsa -m PEM -f /home/ubuntu/.ssh/{TERRAMAN_MASTER_NODE_NAME}-key
```
*예시*
- Master Node의 이름: terraman-master 
- SSH Key의 이름: terraman-master-key
```bash
$ ssh-keygen -t rsa -m PEM -f /home/ubuntu/.ssh/terraman-master-key
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/terraman-master-key
Your public key has been saved in /home/ubuntu/.ssh/terraman-master-key.pub
The key fingerprint is:
SHA256:YLTGX5o4P9IRvQbsSCFnicqQQ5SN9PKXBwd2KuWWaIY ubuntu@bami-dev-terraman-1
The key's randomart image is:
+---[RSA 3072]----+
|+=+ .==o         |
|+oo.***+ .       |
| E B *B.+ o      |
|  B o++* * .     |
|   . o+.S o      |
|    . .+ o       |
|      . +        |
|       . .       |
|                 |
+----[SHA256]-----+
```
####  <div id='3.1.2'> 3.1.2 생성된 공개키는 해당 IaaS에 keypair로 등록
- 공개키 복사
```bash
$ cat /home/ubuntu/.ssh/terraman-master-key.pub
ssh-rsa AAAAB3NzaC1yc2EAAA......ADAQABAAABgQCRfg1qOsA12PRCVE2GFNrsMyF+wA5J3H4eKpwsYfYV5ldAZOuC/n7vGYLIr+ykDFEAAC83lAxq.....b7rjoSVXqkYnn06kzjpKDt0WPnMaoRgdY8ZHiSNWnQAgyMzEZO5jPH6sfW6n......FJaPo7vyEn10Uy9Drd5+HNwkj7eYLoIry8kAiMfnWcsYC7f30JpW6ODSe........83fUu1B1aA7GRZTIRL0b55+MJNUwMN/L8ES/n7j.......syNykOtnF9tM= ubuntu@terramantest-1
```
- IaaS Dashboard 접속 - Openstack
- 공개 키 가져오기
### <div id='3.2'> 3.2 SSH Key 구성
- **id_rsa**, **id_rsa.pub**
	- Host Cluster 구성 시 생성된 RSA Key
- **{MASTER_NODE_NAME}-key**, **{MASTER_NODE_NAME}-key.pub**
	- Sub Cluster 구성을 위해 생성된 RSA Key

### <div id='3.3'>3.3 SSH Key 복사
- SSH Key 목적
	- Terraman 실행 중에 Host Cluster 및 Sub Cluster의 각 Master 노드에 액세스하기 위함이다.
- 각 공개키를 Terraman이 실행되는 Terraman Pod로 복사한다.
	- Host Cluster의 Master 키 이름은 *반드시 'master-key'* 로 복사 되어야 한다.
```bash
## Host Cluster Key
$ kubectl cp /home/ubuntu/.ssh/id_rsa {TERRAMAN_PDO_NAME}:/home/1000/.ssh/master-key -n cp-portal
## Sub Cluster Key
$ kubectl cp /home/ubuntu/.ssh/{TERRAMAN_MASTER_NODE_NAME}-key {TERRAMAN_PDO_NAME}:/home/1000/.ssh/{TERRAMAN_MASTER_NODE_NAME}-key -n cp-portal
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

## Sub Cluster Key Name: terraman-master-key
$ kubectl cp /home/ubuntu/.ssh/terraman-master-key cp-portal-terraman-deployment-689585bc94-pq4bt:/home/1000/.ssh/terraman-master-key -n cp-portal
```

## <div id='4'> 4. Template 생성
### <div id='4.1'> 4.1 OpenStack
- 이 Template는 Terraman을 사용하여 OpenStack에서 인스턴스를 생성하는 방법을 설명한다. 기본 Template는 인스턴스 생성에 집중되어 있으며, *네트워크, 키페어, 보안 그룹 등*은 이미 IaaS에 생성된 정보를 활용한다. 따라서 인스턴스 이외의 다른 리소스는 미리 생성되어 있어야 한다.
- [OpenStack Template 작성시 변수 참고](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs#configuration-reference)
- 인스턴스 생성시 "master"와 "worker" 명칭을 반드시 표기해야 한다.  
*예시*
  + 인스턴스 1개 생성시 
    - resource "openstack_compute_instance_v2" "master" {...}
  + 인스턴스 n개 생성시 
    - resource "openstack_compute_instance_v2" "master" {...}
    - resource "openstack_compute_instance_v2" "worker1" {...}
    - resource "openstack_compute_instance_v2" "worker2" {...}
    - resource "openstack_compute_instance_v2" "worker3" {...} ...
```
# 사용 가능한 OpenStack 이미지
data "openstack_images_image_v2" "ubuntu" {
  name = "ubuntu-20.04"                                                                 # 이미지 명
}

# 사용 가능한 OpenStack keyPair
data "openstack_compute_keypair_v2" "cp-keypair" {
  name = "passta-cp-opentofu-keypair"	                                                # keyPair 명
}

# 사용 가능한 OpenStack floating_ip 주소 ( instance 별 각각 필요하다. )
data "openstack_networking_floatingip_v2" "cp-floatingip-master" {
  address = "203.255.255.115"			                                                # 유동 IP 주소
}

# 사용 가능한 OpenStack floating_ip 주소 ( instance 별 각각 필요하다. )
data "openstack_networking_floatingip_v2" "cp-floatingip-worker" {
  address = "203.255.255.118"			                                                # 유동 IP 주소
}

# 사용 가능한 OpenStack network
data "openstack_networking_network_v2" "cp-network" {
  name = "cp-network"			                                                        # network 명
}

# 사용 가능한 OpenStack subnet
data "openstack_networking_subnet_v2" "cp-subnet" {
  name = "cp-subnet"				                                                    # subnet 명
}

# 사용 가능한 OpenStack router
data "openstack_networking_router_v2" "ext_route" {
  name = "ext_route"					                                                # router 명
}

# 사용 가능한 OpenStack security_group
data "openstack_networking_secgroup_v2" "cp-secgroup" {
  name = "cp-secgroup"			                                                        # security group 명
}

# OpenStack 내에서 V2 라우터 인터페이스 리소스를 관리한다.
resource "openstack_networking_router_interface_v2" "cp-router-interface" {
  router_id = data.openstack_networking_router_v2.ext_route.id				            # 이 인터페이스가 속한 라우터의 ID
  subnet_id = data.openstack_networking_subnet_v2.cp-subnet.id		                    # 이 인터페이스가 연결되는 서브넷의 ID
}

# OpenStack 내에서 V2 VM 인스턴스 리소스를 관리
resource "openstack_compute_instance_v2" "opentofu-master-node" {                       # 인스턴스 생성시 반드시 "master"와 "worker"명칭으로 구분
  name              = "opentofu-master-node"				                            # 리소스의 고유한 이름
  flavor_id         = "m1.large"											           	# 서버에 대해 원하는 플레이버의 플레이버 ID
  key_pair          = data.openstack_compute_keypair_v2.cp-keypair.id		        	# 서버에 넣을 키 쌍의 이름
  security_groups   = [data.openstack_networking_secgroup_v2.cp-secgroup.id]   	        # 서버와 연결할 하나 이상의 보안 그룹 이름 배열
  availability_zone = "octavia"									                	    # 서버를 생성할 가용성 영역
  region            = "RegionOne"										                # 서버 인스턴스를 생성할 지역

  # 블록 영역
  block_device {
    uuid                  = data.openstack_images_image_v2.ubuntu.id		            # 이미지, 볼륨 또는 스냅샷의 UUID
    source_type           = "image"					                                    # 장치의 소스 유형입니다. "", "image", "volume", "snapshot" 중 하나
    volume_size           = 80							                                # 생성할 볼륨의 크기(GB)
    boot_index            = 0								                            # 볼륨의 부팅 인덱스
    destination_type      = "volume"				                                    # 생성되는 유형
    delete_on_termination = true						                                # 인스턴스 종료 시 볼륨/블록 디바이스를 삭제 여부, 기본값 false
  }

  # network 영역
  network {
    uuid = data.openstack_networking_network_v2.cp-network.id			                # 서버에 연결할 네트워크 UUID
  }
}

resource "openstack_compute_instance_v2" "opentofu-worker-node" {                       # 인스턴스 2개 이상 생성시 반드시 "master"와 "worker"명칭으로 구분
  name              = "opentofu-worker-node"
  flavor_id         = "m1.large"
  key_pair          = data.openstack_compute_keypair_v2.cp-keypair.id
  security_groups   = [data.openstack_networking_secgroup_v2.cp-secgroup.id]
  availability_zone = "octavia"
  region            = "RegionOne"

  block_device {
    uuid                  = data.openstack_images_image_v2.ubuntu.id
    source_type           = "image"
    volume_size           = 80
    boot_index            = 0
    destination_type      = "volume"
    delete_on_termination = true
  }

  network {
    uuid = data.openstack_networking_network_v2.cp-network.id
  }
}

# floating IP 영역
resource "openstack_compute_floatingip_associate_v2" "fip_1" {
  floating_ip = data.openstack_networking_floatingip_v2.cp-floatingip-master.address	# 연결할 유동 IP
  instance_id = "${openstack_compute_instance_v2.opentofu-master-node.id}"			    # 유동 IP를 연결할 인스턴스
  wait_until_associated = true					                                        # OpenStack 환경이 연결이 완료될 때까지 자동으로 기다리지 않는 경우 유동 IP가 연결될 때까지 OpenTofu가 인스턴스를 폴링하도록 이 옵션을 설정, 기본값 false
}

resource "openstack_compute_floatingip_associate_v2" "fip_2" {
  floating_ip = data.openstack_networking_floatingip_v2.cp-floatingip-worker.address
  instance_id = "${openstack_compute_instance_v2.opentofu-worker-node.id}"
  wait_until_associated = true
}
```


### <div id='4.2'> 4.2 AWS
- 이 템플릿은 Terraman을 사용하여 AWS에서 인스턴스를 생성하는 방법을 설명한다. 기본 Template는 키페어 및 이미지 리소스를 활용하는 데 초점을 맞추고 있다. 그러므로 키페어 및 이미지 리소스는 사전에 생성되어 있어야 한다.
- [AWS Template 작성시 변수 참고](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#aws-configuration-reference)
- 인스턴스 생성시 "master"와 "worker" 명칭을 반드시 표기해야 한다.  
*예시*
  + 인스턴스 1개 생성시 
    - resource "aws_instance" "master" {...}
  + 인스턴스 n개 생성시 
    - resource "aws_instance" "master" {...}
    - resource "aws_instance" "worker1" {...}
    - resource "aws_instance" "worker2" {...}
    - resource "aws_instance" "worker3" {...} ...
```
# 키 페어에 대한 정보 제공
data "aws_key_pair" "default_key" {
  key_name = "cluster-name-key"		# 키 쌍 이름
}

# 리소스에서 사용할 등록된 AMI의 ID 제공
data "aws_ami" "ubuntu" {
	most_recent = true							                                         # 둘 이상의 결과가 반환되는 경우 가장 최근의 AMI를 사용
	filter {										                                     # 필터링할 하나 이상의 이름/값 쌍
		name = "name"								                                     # 기본 AWS API에서 정의한 필터링 기준 필드의 이름
		values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]	         # 지정된 필드에 대해 허용되는 값 집합
	}
	filter {
		name = "virtualization-type"
		values = ["hvm"]
	}
	owners = ["099720109477"]				                                             # 검색을 제한할 AMI 소유자 목록
}

# VPC 리소스 영역
resource "aws_vpc" "cp-opentofu-vpc" {
	cidr_block = "172.10.0.0/20"					                                     # VPC에 대한 IPv4 CIDR 블록
	tags = { Name = "cp-opentofu-vpc" }		                                             # 리소스에 할당할 태그 맵
}

# VPC 서브넷 리소스 영역
resource "aws_subnet" "cp-opentofu-subnet01" {
  vpc_id = "${aws_vpc.aws-vpc.id}"					                                     # VPC ID
  cidr_block = "172.10.0.0/24"						                                     # 서브넷의 IPv4 CIDR 블록
  availability_zone = "ap-northeast-2a"				                                     # 서브넷의 AZ
  tags = { Name = "cp-opentofu-subnet01" }	                                             # 리소스에 할당할 태그 맵
}

resource "aws_subnet" "cp-opentofu-subnet02" {
  vpc_id = "${aws_vpc.aws-vpc.id}"
  cidr_block = "172.10.1.0/24"
  availability_zone = "ap-northeast-2a"
  tags = {
    Name = "cp-opentofu-subnet02"
  }
}

# 보안 그룹 리소스 영역
resource "aws_security_group" "cp-opentofu-sg-all" {
  name = "cp-opentofu-sg-all"				                                          	 # 보안 그룹의 이름
  description = "Allow all inbound traffic"			                                     # 보안 그룹 설명
  vpc_id = "${aws_vpc.aws-vpc.id}"	# VPC ID

  ingress {							                                                     # 수신 규칙에 대한 구성 블록
	from_port = 0					                                                     # 시작 포트
	to_port = 0						                                                     # 끝 범위 포트
	protocol = "-1"					                                                     # 프로토콜
	cidr_blocks = ["0.0.0.0/0"]		                                                     # CIDR 블록 집합
  }

  egress {							                                                     # 송신 규칙에 대한 구성 블록
	from_port = 0
	to_port = 0
	protocol = "-1"
	cidr_blocks = ["0.0.0.0/0"]
  }
}

# 인스턴스 리소스 영역
resource "aws_instance" "master" {                                                       # 인스턴스 2개 이상 생성시 반드시 "master"와 "worker"명칭으로 구분
  ami = "${data.aws_ami.ubuntu.id}"								                         # 인스턴스에 사용할 AMI
  instance_type = "t3.medium"										                     # 인스턴스에 사용할 인스턴스 유형
  key_name = data.aws_key_pair.default_key.key_name					                     # 인스턴스에 사용할 키 쌍의 키 이름
  subnet_id = "${aws_subnet.cp-opentofu-subnet01.id}"		                             # 시작할 VPC 서브넷 ID
  vpc_security_group_ids = [										                     # 연결할 보안 그룹 ID
	"${aws_security_group.cp-opentofu-sg-all.id}"
  ]
  associate_public_ip_address = true			                                         # 퍼블릭 IP 주소를 VPC의 인스턴스와 연결할지 여부
  tags = {													                             # 리소스에 할당할 태그의 맵
	Name = "cp-opentofu-m"
  }
  provisioner "remote-exec" {				                                             # OpenTofu로 리소스를 생성하거나 제거할 때 로컬이나 원격에서 스크립트를 실행할 수 있는 기능
	connection {							                                             # 연결 블록
	  type = "ssh"						                                                 # 연결 유형
	  host = "${self.public_ip}"			                                             # 연결할 리소스의 주소
	  user = "ubuntu"									                                 # 연결에 사용할 사용자
	  private_key = file("~/.ssh/cluster-name-key.pem")		                             # 연결에 사용할 SSH 키의 내용
	  timeout = "1m"									                                 # 연결을 사용할 수 있을 때까지 기다리는 시간 초과
	}
	inline = [
	  "cat .ssh/authorized_keys"							                             # 실행 커맨드
	]
  }
}

resource "aws_instance" "worker" {                                                       # 인스턴스 2개 이상 생성시 반드시 "master"와 "worker"명칭으로 구분                  
  ami = "${data.aws_ami.ubuntu.id}"
  instance_type = "t3.medium"
  key_name = data.aws_key_pair.default_key.key_name
  subnet_id = "${aws_subnet.cp-opentofu-subnet01.id}"
  vpc_security_group_ids = [
    "${aws_security_group.cp-opentofu-sg-all.id}"
    #data.aws_security_group.default.id
  ]
  associate_public_ip_address = true
  tags = {
    Name = "cp-opentofu-w"
  }
  provisioner "remote-exec" {
    connection {
        type = "ssh"
        host = "${self.public_ip}"
        user = "ubuntu"
        private_key = file("~/.ssh/cluster-name-key.pem")
        timeout = "1m"
    }
    inline = [
      "cat .ssh/authorized_keys"
    ]
  }
}

# VPC의 기본 라우팅 테이블을 관리하기 위한 리소스 영역
resource "aws_default_route_table" "nc-public" {
  default_route_table_id = "${aws_vpc.aws-vpc.default_route_table_id}"	                 # 기본 라우팅 테이블의 ID
  tags = { Name = "New Public Route Table" }						                     # 리소스에 할당할 태그의 맵
}

# VPC 라우팅 테이블을 생성하기 위한 리소스 영역
resource "aws_route_table" "nc-private" {
  vpc_id = "${aws_vpc.aws-vpc.id}"				                                         # VPC ID
  tags = { Name = "New Route Private Table" }	                                         # 리소스에 할당할 태그의 맵
}

# 라우팅 테이블과 서브넷 또는 라우팅 테이블과 인터넷 게이트웨이 또는 가상 프라이빗 게이트웨이 간의 연결을 생성하기 위한 리소스 영역
resource "aws_route_table_association" "aws_public_2a" {
  subnet_id = "${aws_subnet.cp-opentofu-subnet01.id}"		                             # 연결을 생성하기 위한 서브넷 ID
  route_table_id = "${aws_vpc.aws-vpc.default_route_table_id}"		                     # 연결할 라우팅 테이블의 ID
}

resource "aws_route_table_association" "aws_private_2a" {
  subnet_id = "${aws_subnet.cp-opentofu-subnet02.id}"
  route_table_id = "${aws_vpc.aws-vpc.default_route_table_id}"
}

# VPC 인터넷 게이트웨이를 생성하기 위한 리소스
resource "aws_internet_gateway" "aws-igw" {
  vpc_id = "${aws_vpc.aws-vpc.id}"					                                      # 생성할 VPC ID
  tags = { Name = "aws-vpc Internet Gateway" }		                                      # 리소스에 할당할 태그 맵
}

# VPC 라우팅 테이블에서 라우팅 테이블 항목(경로)을 생성하기 위한 리소스
resource "aws_route" "aws_public" {
  route_table_id         = "${aws_vpc.aws-vpc.default_route_table_id}"	                  # 라우팅 테이블의 ID
  destination_cidr_block = "0.0.0.0/0"								                      # 대상 CIDR 블록
  gateway_id             = "${aws_internet_gateway.aws-igw.id}"			                  # VPC 인터넷 게이트웨이 또는 가상 프라이빗 게이트웨이의 식별자
}

resource "aws_route" "aws_private" {
  route_table_id         = "${aws_route_table.nc-private.id}"
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = "${aws_nat_gateway.aws-nat.id}"
}

# 탄력적 IP 리소스를 제공
resource "aws_eip" "aws_nat" {
  vpc = true		                                                                      # EIP가 VPC에 있는 경우, 리전이 EC2-Classic을 지원하지 않는 한 기본값은 true
}

# VPC NAT 게이트웨이를 생성하기 위한 리소스 영역
resource "aws_nat_gateway" "aws-nat" {
  allocation_id = "${aws_eip.aws_nat.id}"					                     	      # 게이트웨이에 대한 탄력적 IP 주소의 할당 ID
  subnet_id     = "${aws_subnet.cp-opentofu-subnet01.id}"		                          # 게이트웨이를 배치할 서브넷의 서브넷 ID
}
```
### <div id='4.3'> 4.3 NHN
- 이 Template는 Terraman을 사용하여 NHN에서 인스턴스를 생성하는 방법을 설명한다. 기본 Template는 인스턴스 생성에 집중되어 있으며, *네트워크, 키페어, 보안 그룹 등*은 이미 IaaS에 생성된 정보를 활용한다. 따라서 인스턴스 이외의 다른 리소스는 미리 생성되어 있어야 한다.
- [OpenStack Template 작성시 변수 참고](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs#configuration-reference)
- 인스턴스 생성시 "master"와 "worker" 명칭을 반드시 표기해야 한다.  
*예시*
  + 인스턴스 1개 생성시 
    - resource "openstack_compute_instance_v2" "master" {...}
  + 인스턴스 n개 생성시 
    - resource "openstack_compute_instance_v2" "master" {...}
    - resource "openstack_compute_instance_v2" "worker1" {...}
    - resource "openstack_compute_instance_v2" "worker2" {...}
    - resource "openstack_compute_instance_v2" "worker3" {...} ...
```
## Use this data source to get the ID of an available OpenStack network.
data "openstack_networking_network_v2" "cp-network" {
  name = "Default Network"
}

## Use this data source to get the ID of an available OpenStack subnet.
data "openstack_networking_subnet_v2" "cp-subnet" {
  network_id = data.openstack_networking_network_v2.cp-network.id
  name       = "Default Network"
}

## Use this data source to get the ID of an available OpenStack security group.
data "openstack_networking_secgroup_v2" "cp-sg" {
  name = "default"
}

## Use this data source to get the ID of an available OpenStack image.
data "openstack_images_image_v2" "ubuntu_focal" {
  name        = "Ubuntu Server 20.04.6 LTS (2023.08.22)"
  most_recent = true
}

## Use this data source to get the ID of an available OpenStack network.
data "openstack_networking_network_v2" "ext_network" {
  name = "Public Network"
}

## Manages a V2 port resource within OpenStack.
resource "openstack_networking_port_v2" "nic" {

  name = "cp-nic01"
  network_id = data.openstack_networking_network_v2.cp-network.id
  fixed_ip {
    subnet_id = data.openstack_networking_subnet_v2.cp-subnet.id
  }
  security_group_ids = [
    data.openstack_networking_secgroup_v2.cp-sg.id
  ]
}

## Manages a V2 VM instance resource within OpenStack.
resource "openstack_compute_instance_v2" "vm-cp-master" {                                 # 인스턴스 2개 이상 생성시 반드시 "master"와 "worker"명칭으로 구분

  name              = "cp-cluster-master"
  availability_zone = "kr-pub-a"
  flavor_name       = "m2.c4m8"
  key_pair          = "cp-nhn-common-key"

  block_device {                                                                          # Configuration of block devices.
    uuid                  = data.openstack_images_image_v2.ubuntu_focal.id                # The UUID of the image, volume, or snapshot. Changing this creates a new server.
    source_type           = "image"                                                       # The source type of the device. Must be one of "blank", "image", "volume", or "snapshot". Changing this creates a new server.
    destination_type      = "volume"                                                      # The type that gets created. Possible values are "volume" and "local". Changing this creates a new server.
    delete_on_termination = true                                                          # Delete the volume / block device upon termination of the instance. Defaults to false. Changing this creates a new server.
    volume_size           = 40                                                            # The size of the volume to create (in gigabytes).
    volume_type           = "General HDD"                                                 # The volume type that will be used, for example SSD or HDD storage.
  }

  network {
    port = openstack_networking_port_v2.nic.id                                            # The port UUID of a network to attach to the server.
  }
}

## Manages a V2 floating IP resource within OpenStack Neutron (networking) that can be used for load balancers. These are similar to Nova (compute) floating IP resources, but only compute floating IPs can be used with compute instances.
resource "openstack_networking_floatingip_v2" "fip_1" {
  pool       = data.openstack_networking_network_v2.ext_network.name    # The name of the pool from which to obtain the floating IP. Changing this creates a new floating IP.
}

## Associate a floating IP to an instance.
resource "openstack_compute_floatingip_associate_v2" "fip_1" {
  floating_ip = openstack_networking_floatingip_v2.fip_1.address                          # The floating IP to associate.
  instance_id = openstack_compute_instance_v2.vm-cp-master.id                                       # The instance to associte the floating IP with.
  wait_until_associated = true                                                            # In cases where the OpenStack environment does not automatically wait until the association has finished, set this option to have OpenTofu poll the instance until the floating IP has been associated. Defaults to false.
}
```