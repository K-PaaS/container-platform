### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > Terraman IaC 스크립트 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [참고자료](#1.3)  

2. [Terraman 실행](#2)  
  2.1. [Prerequisite](#2.1)  
  2.2. [SSH Key 생성 및 사전작업](#2.2)  
  2.3. [Terraman Template OpenStack](#2.3)  
  2.4. [Terraman Template AWS](#2.4)  

3. [SubCluster 생성 구동 시 주의사항](#3)  

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (Terraman IaC 스크립트 가이드)는 Terraform을 실행시켜 Sub Cluster를 생성하기 위한 IaaS별 HCL(Hashicorp Configuration Language)의 구문별 설명을 통해 실제 배포할 스크립트 완성을 도와주기 위한 문서이다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 Kubernetes Cluster 배포를 기준으로 작성하였다.

<br>

### <div id='1.3'> 1.3. 참고자료
- Terraform OpenStack
> https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs  
- Terraform AWS
> https://registry.terraform.io/providers/hashicorp/aws/latest/docs
<br>

## <div id='2'> 2. Terraman 실행

### <div id='2.1'> 2.1. Prerequisite
본 가이드는 Terraman을 실행하기 전 필수 선행 작업에 대한 내용을 기술하였다.
<br>SubCluster 생성을 위한 Terraform은 컨테이너플랫폼 HostCluster 설치 시 자동으로 배포되기에 아래 방화벽 설정에 대한 내용만 확인하면 된다.


#### 방화벽 정보
- Master Node

| <center>프로토콜</center> | <center>포트</center> | <center>비고</center> |  
| :---: | :---: | :--- |  
| TCP | 22 | SSH | 
| TCP | 6443 | Kubernetes API Server |  
-  IaaS 별 API 방화벽 오픈  
ex) OpenStack API (8000, 8774, 5000, 9292, 9876, 9696, 8004, 8780, 8776)
- 각 Iaas에서 생성되는 Instance는 remote 접속을 위한 포트가 열려있어야 한다.
<br>

### <div id='2.2'> 2.2. SSH Key 생성 및 사전작업
- Terraman이 동작하기 위해서는 Terraman Pod에서 **HostCluster의 Master Node**와 **생성되는 SubCluster의 Master Node**에 접속 할 수 있어야 한다.
- cluster 접속을 위한 key 생성 전 기존 master cluster ansible 구성시 사용되었던 **개인키와 공개키(id_rsa, id_rsa.pub)** 는 백업이 필요하다.

- **Master Node**에서 RSA 공개키를 생성한다. 
<br>**(cluster 접속을 위한 key 생성 - 반드시 RSA 공개키 생성 필요!! OPENSSH는 접속이 되지 않는다. 3.주의사항 참조)**
```
$ ssh-keygen -t rsa -m PEM -f /home/ubuntu/.ssh/{{ clusterName }}-key
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): [엔터키 입력]
Enter passphrase (empty for no passphrase): [엔터키 입력]
Enter same passphrase again: [엔터키 입력]
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:pIG4/G309Dof305mWjdNz1OORx9nQgQ3b8yUP5DzC3w ubuntu@paasta-cp-master
The key's randomart image is:
+---[RSA 2048]----+
|            ..= o|
|   . .       * B |
|  . . . .   . = *|
| . .   +     + E.|
|  o   o S     +.O|
|   . o o .     XB|
|    . o . o   *oO|
|     .  .. o B oo|
|        .o. o.o  |
+----[SHA256]-----+
```

- 생성된 공개키는 해당 Iaas에 keypair로 등록한다.

- 백업해 놓은 **Master Node 개인키**와 생성된 **Cluster 접속을 위한 개인키**를 Terraman Pod에 복사한다.
```
## 개인키 복사
$ kubectl cp /home/ubuntu/.ssh/{{백업해 놓은 Master Node 개인키 명}} {{Terraman Pod 명}}:/home/1000/.ssh/paasta-master-key -n cp-portal  (Master Node 접속용 개인키)
( ex. kubectl cp /home/ubuntu/.ssh/master-host-node-key terraman-pod:/home/1000/.ssh/paasta-master-key -n cp-portal )

$ kubectl cp /home/ubuntu/.ssh/{{Cluster 접속을 위한 개인키 명}} {{Terraman Pod 명}}:/home/1000/.ssh/{{ clusterName }}-key -n cp-portal  (Cluster 접속용 개인키)
( ex. kubectl cp /home/ubuntu/.ssh/cluster-name-key terraman-pod:/home/1000/.ssh/cluster-name-key -n cp-portal )
```

<br>

### <div id='2.3'> 2.3. Terraman Template OpenStack
Terraman을 이용하여 OpenStack Instance를 생성 시키는 Template 설명이다. 
현 기본 템플릿은 Instance 생성에 맞추어져 있으므로 네트워크, keypair, 보안그룹 등은 사전에 Iaas에 생성되어 있는 정보를 사용한다. 따라서 Instance 생성 이외의 리소스는 사전에 생성해야 한다.

```
# 1. data로 시작되는 영역은 Openstack에서 사용가능한 자원을 가져와 보관한다. 
#   - name으로 가져온 자원의 id는 사용가능하다.
#	ex)	data "테라폼 데이터소스 함수명" "사용할 변수명" {
#		  name = "ubuntu-20.04"
#		}
#		...
#	    block_device {
#		  uuid = data.테라폼 데이터소스 함수명.변수명.id
#		}
#
# 2. resource로 시작되는 영역은 자원 생성 및 보관에 관여하며 data로 선언한 값을 사용 할 수 있다.
# 3. openstack의 경우 각 instance별로 floating_ip자원을 가져와 수동으로 붙여야한다.

# 사용 가능한 OpenStack 이미지
data "openstack_images_image_v2" "ubuntu" {
  name = "ubuntu-20.04"                                     # 이미지 명
}

# 사용 가능한 OpenStack keyPair
data "openstack_compute_keypair_v2" "paasta-cp-keypair" {
  name = "passta-cp-terraform-keypair"	                    # keyPair 명
}

# 사용 가능한 OpenStack floating_ip 주소 ( instance 별 각각 필요하다. )
data "openstack_networking_floatingip_v2" "paasta-cp-floatingip-master" {
  address = "203.255.255.115"			                    # 유동 IP 주소
}

# 사용 가능한 OpenStack floating_ip 주소 ( instance 별 각각 필요하다. )
data "openstack_networking_floatingip_v2" "paasta-cp-floatingip-worker" {
  address = "203.255.255.118"			                    # 유동 IP 주소
}

# 사용 가능한 OpenStack network
data "openstack_networking_network_v2" "paasta-cp-network" {
  name = "paasta-cp-network"			                    # network 명
}

# 사용 가능한 OpenStack subnet
data "openstack_networking_subnet_v2" "paasta-cp-subnet" {
  name = "paasta-cp-subnet"				                    # subnet 명
}

# 사용 가능한 OpenStack router
data "openstack_networking_router_v2" "ext_route" {
  name = "ext_route"					                    # router 명
}

# 사용 가능한 OpenStack security_group
data "openstack_networking_secgroup_v2" "paasta-cp-secgroup" {
  name = "paasta-cp-secgroup"			                    # security group 명
}

# OpenStack 내에서 V2 라우터 인터페이스 리소스를 관리한다.
resource "openstack_networking_router_interface_v2" "paasta-cp-router-interface" {
  router_id = data.openstack_networking_router_v2.ext_route.id				            # 이 인터페이스가 속한 라우터의 ID
  subnet_id = data.openstack_networking_subnet_v2.paasta-cp-subnet.id		            # 이 인터페이스가 연결되는 서브넷의 ID
}

# OpenStack 내에서 V2 VM 인스턴스 리소스를 관리
resource "openstack_compute_instance_v2" "paasta-terraform-master-node" {
  name              = "paasta-terraform-master-node"				                    # 리소스의 고유한 이름
  flavor_id         = "m1.large"														# 서버에 대해 원하는 플레이버의 플레이버 ID
  key_pair          = data.openstack_compute_keypair_v2.paasta-cp-keypair.id			# 서버에 넣을 키 쌍의 이름
  security_groups   = [data.openstack_networking_secgroup_v2.paasta-cp-secgroup.id]		# 서버와 연결할 하나 이상의 보안 그룹 이름 배열
  availability_zone = "octavia"															# 서버를 생성할 가용성 영역
  region            = "RegionOne"														# 서버 인스턴스를 생성할 지역

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
    uuid = data.openstack_networking_network_v2.paasta-cp-network.id			        # 서버에 연결할 네트워크 UUID
  }
}

resource "openstack_compute_instance_v2" "paasta-terraform-worker-node" {
  name              = "paasta-terraform-worker-node"
  flavor_id         = "m1.large"
  key_pair          = data.openstack_compute_keypair_v2.paasta-cp-keypair.id
  security_groups   = [data.openstack_networking_secgroup_v2.paasta-cp-secgroup.id]
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
    uuid = data.openstack_networking_network_v2.paasta-cp-network.id
  }
}

# floating IP 영역
resource "openstack_compute_floatingip_associate_v2" "fip_1" {
  floating_ip = data.openstack_networking_floatingip_v2.paasta-cp-floatingip-master.address		# 연결할 유동 IP
  instance_id = "${openstack_compute_instance_v2.paasta-terraform-master-node.id}"				# 유동 IP를 연결할 인스턴스
  wait_until_associated = true					                                                # OpenStack 환경이 연결이 완료될 때까지 자동으로 기다리지 않는 경우 유동 IP가 연결될 때까지 Terraform이 인스턴스를 폴링하도록 이 옵션을 설정, 기본값 false
}

resource "openstack_compute_floatingip_associate_v2" "fip_2" {
  floating_ip = data.openstack_networking_floatingip_v2.paasta-cp-floatingip-worker.address
  instance_id = "${openstack_compute_instance_v2.paasta-terraform-worker-node.id}"
  wait_until_associated = true
}
```

<br>

### <div id='2.4'> 2.4. Terraman Template AWS
2.3.Terraman을 이용하여 AWS Instance를 생성 시키는 Template 설명이다.
현 기본 템플릿은 keypair, 이미지 리소스는 사전에 Iaas에 생성되어 있는 정보를 사용한다. 따라서 keypair, 이미지 리소스는 사전에 생성해야 한다.

```
# 1. data로 시작되는 영역은 Openstack에서 사용가능한 자원을 가져와 보관한다. 
#   - name으로 가져온 자원의 id는 사용가능하다.
#	ex)	data "테라폼 데이터소스 함수명" "사용할 변수명" {
#		  name = "ubuntu-20.04"
#		}
#		...
#	    block_device {
#		  uuid = data.테라폼 데이터소스 함수명.변수명.id
#		}
#
# 2. resource로 시작되는 영역은 자원 생성 및 보관에 관여하며 data로 선언한 값을 사용 할 수 있다.
# 3. variable로 시작되는 영역은 변수 선언영역이다.
#   ex)	variable "route_table_name" {
#			default = "paasta-cp-routing-public"
#		}
#		...
#		filter {
#			name = "tag-value"
#			values = ["${var.route_table_name}"]
#		}

# 키 페어에 대한 정보 제공
data "aws_key_pair" "default_key" {
  key_name = "cluster-name-key"		# 키 쌍 이름
}

# 리소스에서 사용할 등록된 AMI의 ID 제공
data "aws_ami" "ubuntu" {
	most_recent = true							       # 둘 이상의 결과가 반환되는 경우 가장 최근의 AMI를 사용
	filter {										   # 필터링할 하나 이상의 이름/값 쌍
		name = "name"								   # 기본 AWS API에서 정의한 필터링 기준 필드의 이름
		values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]	# 지정된 필드에 대해 허용되는 값 집합
	}
	filter {
		name = "virtualization-type"
		values = ["hvm"]
	}
	owners = ["099720109477"]				           # 검색을 제한할 AMI 소유자 목록
}

# VPC 리소스 영역
resource "aws_vpc" "paasta-cp-terraform-vpc" {
	cidr_block = "172.10.0.0/20"					   # VPC에 대한 IPv4 CIDR 블록
	tags = { Name = "paasta-cp-terraform-vpc" }		   # 리소스에 할당할 태그 맵
}

# VPC 서브넷 리소스 영역
resource "aws_subnet" "paasta-cp-terraform-subnet01" {
  vpc_id = "${aws_vpc.aws-vpc.id}"					   # VPC ID
  cidr_block = "172.10.0.0/24"						   # 서브넷의 IPv4 CIDR 블록
  availability_zone = "ap-northeast-2a"				   # 서브넷의 AZ
  tags = { Name = "paasta-cp-terraform-subnet01" }	   # 리소스에 할당할 태그 맵
}

resource "aws_subnet" "paasta-cp-terraform-subnet02" {
  vpc_id = "${aws_vpc.aws-vpc.id}"
  cidr_block = "172.10.1.0/24"
  availability_zone = "ap-northeast-2a"
  tags = {
    Name = "paasta-cp-terraform-subnet02"
  }
}

# 보안 그룹 리소스 영역
resource "aws_security_group" "paasta-cp-terraform-sg-all" {
  name = "paasta-cp-terraform-sg-all"					# 보안 그룹의 이름
  description = "Allow all inbound traffic"			    # 보안 그룹 설명
  vpc_id = "${aws_vpc.aws-vpc.id}"	# VPC ID

  ingress {							                    # 수신 규칙에 대한 구성 블록
	from_port = 0					                    # 시작 포트
	to_port = 0						                    # 끝 범위 포트
	protocol = "-1"					                    # 프로토콜
	cidr_blocks = ["0.0.0.0/0"]		                    # CIDR 블록 집합
  }

  egress {							                    # 송신 규칙에 대한 구성 블록
	from_port = 0
	to_port = 0
	protocol = "-1"
	cidr_blocks = ["0.0.0.0/0"]
  }
}

# 인스턴스 리소스 영역
resource "aws_instance" "master" {
  ami = "${data.aws_ami.ubuntu.id}"									# 인스턴스에 사용할 AMI
  instance_type = "t3.medium"										# 인스턴스에 사용할 인스턴스 유형
  key_name = data.aws_key_pair.default_key.key_name					# 인스턴스에 사용할 키 쌍의 키 이름
  subnet_id = "${aws_subnet.paasta-cp-terraform-subnet01.id}"		# 시작할 VPC 서브넷 ID
  vpc_security_group_ids = [										# 연결할 보안 그룹 ID
	"${aws_security_group.paasta-cp-terraform-sg-all.id}"
  ]
  associate_public_ip_address = true			                    # 퍼블릭 IP 주소를 VPC의 인스턴스와 연결할지 여부
  tags = {													        # 리소스에 할당할 태그의 맵
	Name = "paasta-cp-terraform-m"
  }
  provisioner "remote-exec" {				                        # Terraform으로 리소스를 생성하거나 제거할 때 로컬이나 원격에서 스크립트를 실행할 수 있는 기능
	connection {							                        # 연결 블록
	  type = "ssh"						                            # 연결 유형
	  host = "${self.public_ip}"			                        # 연결할 리소스의 주소
	  user = "ubuntu"									            # 연결에 사용할 사용자
	  private_key = file("~/.ssh/cluster-name-key.pem")		        # 연결에 사용할 SSH 키의 내용
	  timeout = "1m"									            # 연결을 사용할 수 있을 때까지 기다리는 시간 초과
	}
	inline = [
	  "cat .ssh/authorized_keys"							        # 실행 커맨드
	]
  }
}

resource "aws_instance" "worker" {
  ami = "${data.aws_ami.ubuntu.id}"
  instance_type = "t3.medium"
  key_name = data.aws_key_pair.default_key.key_name
  subnet_id = "${aws_subnet.paasta-cp-terraform-subnet01.id}"
  vpc_security_group_ids = [
    "${aws_security_group.paasta-cp-terraform-sg-all.id}"
    #data.aws_security_group.default.id
  ]
  associate_public_ip_address = true
  tags = {
    Name = "paasta-cp-terraform-w"
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
  default_route_table_id = "${aws_vpc.aws-vpc.default_route_table_id}"	# 기본 라우팅 테이블의 ID
  tags = { Name = "New Public Route Table" }							# 리소스에 할당할 태그의 맵
}

# VPC 라우팅 테이블을 생성하기 위한 리소스 영역
resource "aws_route_table" "nc-private" {
  vpc_id = "${aws_vpc.aws-vpc.id}"				                        # VPC ID
  tags = { Name = "New Route Private Table" }	                        # 리소스에 할당할 태그의 맵
}

# 라우팅 테이블과 서브넷 또는 라우팅 테이블과 인터넷 게이트웨이 또는 가상 프라이빗 게이트웨이 간의 연결을 생성하기 위한 리소스 영역
resource "aws_route_table_association" "aws_public_2a" {
  subnet_id = "${aws_subnet.paasta-cp-terraform-subnet01.id}"		    # 연결을 생성하기 위한 서브넷 ID
  route_table_id = "${aws_vpc.aws-vpc.default_route_table_id}"		    # 연결할 라우팅 테이블의 ID
}

resource "aws_route_table_association" "aws_private_2a" {
  subnet_id = "${aws_subnet.paasta-cp-terraform-subnet02.id}"
  route_table_id = "${aws_vpc.aws-vpc.default_route_table_id}"
}

# VPC 인터넷 게이트웨이를 생성하기 위한 리소스
resource "aws_internet_gateway" "aws-igw" {
  vpc_id = "${aws_vpc.aws-vpc.id}"					                    # 생성할 VPC ID
  tags = { Name = "aws-vpc Internet Gateway" }		                    # 리소스에 할당할 태그 맵
}

# VPC 라우팅 테이블에서 라우팅 테이블 항목(경로)을 생성하기 위한 리소스
resource "aws_route" "aws_public" {
  route_table_id         = "${aws_vpc.aws-vpc.default_route_table_id}"	# 라우팅 테이블의 ID
  destination_cidr_block = "0.0.0.0/0"								    # 대상 CIDR 블록
  gateway_id             = "${aws_internet_gateway.aws-igw.id}"			# VPC 인터넷 게이트웨이 또는 가상 프라이빗 게이트웨이의 식별자
}

resource "aws_route" "aws_private" {
  route_table_id         = "${aws_route_table.nc-private.id}"
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = "${aws_nat_gateway.aws-nat.id}"
}

# 탄력적 IP 리소스를 제공
resource "aws_eip" "aws_nat" {
  vpc = true		# EIP가 VPC에 있는 경우, 리전이 EC2-Classic을 지원하지 않는 한 기본값은 true
}

# VPC NAT 게이트웨이를 생성하기 위한 리소스 영역
resource "aws_nat_gateway" "aws-nat" {
  allocation_id = "${aws_eip.aws_nat.id}"								# 게이트웨이에 대한 탄력적 IP 주소의 할당 ID
  subnet_id     = "${aws_subnet.paasta-cp-terraform-subnet01.id}"		# 게이트웨이를 배치할 서브넷의 서브넷 ID
}
```

<br>

## <div id='3'> 3. SubCluster 생성 구동 시 주의사항
- **ssh 접속 키**는 RSA기반으로 만들어져야 한다.
	<br>( OPENSSH 기반으로 생성했을 경우 변환 필요!! $ssh-keygen -p -m PEM -f /home/ubuntu/.ssh/id_rsa )

- **Master Node 개인키**와 **Cluster 개인키**를 혼동하지 않도록 주의하고 각각의 명칭에 맞게 복사한다.
	<br>( 복사한 파일의 권한은 600 )

- **Master Node 개인키**와 **Cluster 개인키**는 반드시 Terraman Pod 내에 **/home/1000/.ssh** 경로 내에 존재하여야 한다.

- **Master Node**는 Template 작성시 Instance 명에 **master**가 포함되어야 한다. 
  <br>( master node의 ip를 특정하기 위한 구분 )

- SubCluster의 API 호출을 Domain으로 할 경우 Terraform이 설치되어 있는 **Master Node**의 Hosts 파일에 Domain을 추가해야 한다.
  <br>**( $ echo -e "\n{{ API IP }} {{ Domain }}" >> /etc/hosts )**
  <br>( ex. echo -e "\n192.168.10.111 terraman-ex >> /etc/hosts )

<br>

<!-- [image 001]:images/standalone-v1.2.png -->

<!-- ### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > 클러스터 설치 가이드 -->
