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
　4.1.1. [OpenStack](#4.1.1)   
　4.1.2. [AWS](#4.1.2)   
　4.1.3. [NHN](#4.1.3)   
5. [SSH Keys](#5)  
 5.1. [SSH Keys 등록](#5.1)  
　5.1.1. [기존 SSH Key 등록](#5.1.1)   
　5.1.2. [SSH Key 신규 등록](#5.1.2)   
6. [Instance Code Template](#6)  
 6.1. [Instance Code Template 예제](#6.1)  
　6.1.1. [OpenStack](#6.1.1)   
　6.1.2. [AWS](#6.1.2)   
　6.1.3. [NHN](#6.1.3)   
 6.2. [Instance Code Template 등록](#6.2)  
7. [Clusters](#7)  
 7.1. [Clusters 생성](#7.1)  

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
**OpenStack** 
> [OpenStack Provider](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs) <br>

**AWS**
> [AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) <br>

**NHN**
> [NHN Terraform 사용 가이드](https://docs.nhncloud.com/ko/Compute/Instance/ko/terraform-guide/#tf)<br>
> [OpenStack Provider](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs) <br>


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
Terraman을 통한 `SSH Key` 방식의 클러스터 배포는 **Cloud Accounts**와 **SSH Keys** 설정을 통해 **Instance Code Template**을 활용하여 서브 클러스터를 배포한다.
1. [Cloud Accounts 등록](#4)
2. [SSH Keys 등록](#5)
3. [Instance Code Template 등록 또는 기본 IaC 코드 활용](#6) 
4. [Clusters 배포](#7)
 

<img src="../../images/terraman/IMG_3.png">


<br>
<br>

## <div id='4'> 4. Cloud Accounts
### <div id='4.1'> 4.1 Cloud Account 등록
컨테이너 플랫폼 포털 메뉴 Global > Cloud Accounts에서 Cloud Accounts 정보를 등록한다.

#### <div id='4.1.1'> 4.1.1 OpenStack
아래 항목의 OpenStack Cloud 정보를 입력한다.
- OpenStack은 RC 파일이라는 클라이언트 환경 스크립트를 지원하며, OpenStack RC 파일은 OpenStack 대시보드 우측 상단의 계정을 클릭하면 다운로드 받을 수 있다.

|항목|OpenStack Cloud 정보| 정보 위치|
|:------:|:------:|:------:|
|auth_url 필드|OS_AUTH_URL|OpenStack RC 파일 참고|
|password 필드|계정 비밀번호|계정 비밀번호|
|user_name 필드|계정 아이디|계정 아이디|
|project 필드|OS_PROJECT_ID|OpenStack RC 파일 참고|
|region 필드|OS_REGION_NAME|OpenStack RC 파일 참고|


<img src="../../images/terraman/IMG_4_1_1.png">


#### <div id='4.1.2'> 4.1.2 AWS
아래 항목의 AWS Cloud 정보를 입력한다.
- AWS 자격증명은 IAM 대시보드에서 발급 받을 수 있다. (분실시 재발급 필요)
  
|항목|AWS Cloud 정보|정보 위치|
|:------:|:------:|:------:|
|accessKey 필드|엑세스 키 ID|IAM 대시보드에서 발급|
|secretKey 필드|비밀 엑세스 키|IAM 대시보드에서 발급|
|region 필드|사용 리전|계정 정보 확인[(리전 이름 가이드 참고)](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions)|

<img src="../../images/terraman/IMG_4_1_2.png">

#### <div id='4.1.3'> 4.1.3 NHN
아래 항목의 NHN Cloud 정보를 입력한다.
- NHN Cloud는 Compute > Instance 페이지에서 인증 정보 확인이 가능하다.
  
|항목|NHN Cloud 정보|정보 위치|  
|:------:|:------:|:------:|
|auth_url 필드|신원 서비스(identity)|Instance 페이지 > API 엔드포인트 설정 > 신원 서비스(identity)|  
|password 필드|계정 비밀번호|계정 비밀번호|  
|user_name 필드|계정 아이디|계정 아이디|  
|project 필드|테넌트 ID|Instance 페이지 > API 엔드포인트 설정 > 테넌트 ID|    
|region 필드|사용 리전|계정 정보 확인[(리전 이름 가이드 참고)](https://docs.nhncloud.com/ko/Storage/Object%20Storage/ko/s3-api-guide/#signature)| 


<img src="../../images/terraman/IMG_4_1_3.png">


<br>
<br>


## <div id='5'> 5. SSH Keys
### <div id='5.1'>  5.1. SSH Keys 등록
컨테이너 플랫폼 Terraman을 통해 신규 생성되는 인스턴스에 접근하기 위해서 SSH Key가 필요하다. <br>
컨테이너 플랫폼 포털 메뉴 Global > SSH Keys 에서 SSH Key를 등록한다.

#### <div id='5.1.1'> 5.1.1. 기존 SSH Key 등록
해당 클라우드 서비스에서 발급 받은 개인키를 소유하고 있다면 해당 키를 SSH Keys 메뉴에 등록한다.
- SSH Key 명은 사용하고자 하는 이름으로 입력한다.

<img src="../../images/terraman/IMG_5_1_1_1.png">



#### <div id='5.1.2'> 5.1.2. SSH Key 신규 등록
신규로 SSH Key를 발급할 경우 해당 클라우드 서비스에 공개키를 등록하고 컨테이너 플랫폼 포털의 SSH Keys 메뉴에 개인키를 등록한다.

```bash
# ssh-keygen을 이용한 rsa key pair 생성
$ ssh-keygen -t rsa -m PEM -N '' -f $HOME/.ssh/id_rsa
Generating public/private rsa key pair.
Your identification has been saved in /home/ubuntu/.ssh/id_rsa
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:odWdv3PDIEpkPuoS53yM0hrsEQZL4mHvM0KwLK2uC57 ubuntu@cp-master
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|         . . .   |
|.+ o    = . o    |
|++= o  * .   .   |
|oo+o .. S . . .  |
|.+..o  o o . + . |
|o +o O. .     *  |
|=o.o=o*        o |
|E++o ++          |
+----[SHA256]-----+
```
```bash
# rsa key pair 생성 확인
$ ls .ssh/
id_rsa  id_rsa.pub

# 공개키: id_rsa.pub 조회
$ cat .ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCjlIHPM3Nnh+6CK3klnD0l4epC2RG...

# 개인키: id_rsa 조회
$ cat .ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAo5SBzzNzZ4fugit5JZw9JeHqQtkRhTJ8Zdl3CCVKdIWfOI79...
-----END RSA PRIVATE KEY-----
```


##### 예시) Cloud Platform(OpenStack) 공개키(id_rsa.pub) 등록
<img src="../../images/terraman/IMG_5_1_1_2.png">


##### 예시) 컨테이너 플랫폼 포털 SSH Keys 개인키(id_rsa) 등록
<img src="../../images/terraman/IMG_5_1_1_1.png">

<br>

## <div id='6'> 6. Instance Code Template
> 기본적으로 인스턴스 생성에 필요한 Code Template은 각 CSP당 한 종류씩 등록되어 있다. 추가로 필요한 Code Template은 신규로 등록하여 클러스터 생성 시 사용이 가능하다.

### <div id='6.1'> 6.1 Instance Code Template 예제
#### <div id='6.1.1'> 6.1.1 OpenStack
Terraman을 사용하여 OpenStack에서 인스턴스를 생성하는 방법을 설명한다. 기본 IaC 코드는 인스턴스 생성에 맞춰져 있으며, *네트워크, 키페어, 보안 그룹 등*은 이미 IaaS에 생성된 정보를 활용한다. 따라서 인스턴스 이외의 다른 리소스는 미리 생성되어 있어야 한다.
> [OpenStack IaC 코드 작성시 변수 참고](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs#configuration-reference)
- 인스턴스 생성시 "master"와 "worker" 명칭을 반드시 표기해야 한다.  
*예시*
  + 인스턴스 1개 생성시 
    - resource "openstack_compute_instance_v2" "master" {...}
  + 인스턴스 n개 생성시 
    - resource "openstack_compute_instance_v2" "master" {...}
    - resource "openstack_compute_instance_v2" "worker1" {...}
    - resource "openstack_compute_instance_v2" "worker2" {...}
    - resource "openstack_compute_instance_v2" "worker3" {...} ...

<details>
<summary> :page_facing_up: <b>OpenStack Instance Code Template</b> </summary>
<div markdown="1">

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
  address = "x.x.x.x"        			                                        # 유동 IP 주소
}

# 사용 가능한 OpenStack floating_ip 주소 ( instance 별 각각 필요하다. )
data "openstack_networking_floatingip_v2" "cp-floatingip-worker" {
  address = "x.x.x.x"        			                                        # 유동 IP 주소
}

# 사용 가능한 OpenStack network
data "openstack_networking_network_v2" "cp-network" {
  name = "cp-network"			                                                # network 명
}

# 사용 가능한 OpenStack subnet
data "openstack_networking_subnet_v2" "cp-subnet" {
  name = "cp-subnet"				                                        # subnet 명
}

# 사용 가능한 OpenStack router
data "openstack_networking_router_v2" "ext_route" {
  name = "ext_route"					                                # router 명
}

# 사용 가능한 OpenStack security_group
data "openstack_networking_secgroup_v2" "cp-secgroup" {
  name = "cp-secgroup"			                                                # security group 명
}

# OpenStack 내에서 V2 라우터 인터페이스 리소스를 관리한다.
resource "openstack_networking_router_interface_v2" "cp-router-interface" {
  router_id = data.openstack_networking_router_v2.ext_route.id				# 이 인터페이스가 속한 라우터의 ID
  subnet_id = data.openstack_networking_subnet_v2.cp-subnet.id		                # 이 인터페이스가 연결되는 서브넷의 ID
}

# OpenStack 내에서 V2 VM 인스턴스 리소스를 관리
resource "openstack_compute_instance_v2" "opentofu-master-node" {                       # 인스턴스 생성시 반드시 "master"와 "worker" 명칭으로 구분
  name              = "opentofu-master-node"				                # 리소스의 고유한 이름
  flavor_id         = "m1.large"							# 서버에 대해 원하는 플레이버의 플레이버 ID
  key_pair          = data.openstack_compute_keypair_v2.cp-keypair.id		        # 서버에 넣을 키 쌍의 이름
  security_groups   = [data.openstack_networking_secgroup_v2.cp-secgroup.id]   	        # 서버와 연결할 하나 이상의 보안 그룹 이름 배열
  availability_zone = "octavia"								# 서버를 생성할 가용성 영역
  region            = "RegionOne"							# 서버 인스턴스를 생성할 지역

  # 블록 영역
  block_device {
    uuid                  = data.openstack_images_image_v2.ubuntu.id		        # 이미지, 볼륨 또는 스냅샷의 UUID
    source_type           = "image"					                # 장치의 소스 유형입니다. "", "image", "volume", "snapshot" 중 하나
    volume_size           = 80							        # 생성할 볼륨의 크기(GB)
    boot_index            = 0								# 볼륨의 부팅 인덱스
    destination_type      = "volume"				                        # 생성되는 유형
    delete_on_termination = true						        # 인스턴스 종료 시 볼륨/블록 디바이스를 삭제 여부, 기본값 false
  }

  # network 영역
  network {
    uuid = data.openstack_networking_network_v2.cp-network.id			        # 서버에 연결할 네트워크 UUID
  }
}

resource "openstack_compute_instance_v2" "opentofu-worker-node" {                       # 인스턴스 2개 이상 생성시 반드시 "master"와 "worker" 명칭으로 구분
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
  instance_id = "${openstack_compute_instance_v2.opentofu-master-node.id}"		# 유동 IP를 연결할 인스턴스
  wait_until_associated = true					                        # OpenStack 환경이 연결이 완료될 때까지 자동으로 기다리지 않는 경우 유동 IP가 연결될 때까지 OpenTofu가 인스턴스를 폴링하도록 이 옵션을 설정, 기본값 false
}

resource "openstack_compute_floatingip_associate_v2" "fip_2" {
  floating_ip = data.openstack_networking_floatingip_v2.cp-floatingip-worker.address
  instance_id = "${openstack_compute_instance_v2.opentofu-worker-node.id}"
  wait_until_associated = true
}
```

</div>
</details>

#### <div id='6.1.2'> 6.1.2 AWS
Terraman을 사용하여 AWS에서 인스턴스를 생성하는 방법을 설명한다. 기본 IaC 코드는 이미 등록된 키페어 및 OS 이미지를 활용하는 템플릿에 맞춰져 있다. 그러므로 키페어 및 OS 이미지는 사전에 생성되어 있어야 한다.
> [AWS IaC 코드 작성시 변수 참고](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#aws-configuration-reference)
- 인스턴스 생성시 "master"와 "worker" 명칭을 반드시 표기해야 한다.  
*예시*
  + 인스턴스 1개 생성시 
    - resource "aws_instance" "master" {...}
  + 인스턴스 n개 생성시 
    - resource "aws_instance" "master" {...}
    - resource "aws_instance" "worker1" {...}
    - resource "aws_instance" "worker2" {...}
    - resource "aws_instance" "worker3" {...} ...

<details>
<summary> :page_facing_up: <b>AWS Instance Code Template</b></summary>
<div markdown="2">

```
# 키 페어에 대한 정보 제공
data "aws_key_pair" "default_key" {
  key_name = "cluster-name-key"		                                                # 키 쌍 이름
}

# 리소스에서 사용할 등록된 AMI의 ID 제공
data "aws_ami" "ubuntu" {
	most_recent = true							        # 둘 이상의 결과가 반환되는 경우 가장 최근의 AMI를 사용
	filter {									# 필터링할 하나 이상의 이름/값 쌍
		name = "name"								# 기본 AWS API에서 정의한 필터링 기준 필드의 이름
		values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]	# 지정된 필드에 대해 허용되는 값 집합
	}
	filter {
		name = "virtualization-type"
		values = ["hvm"]
	}
	owners = ["099720109477"]				                        # 검색을 제한할 AMI 소유자 목록
}

# VPC 리소스 영역
resource "aws_vpc" "cp-opentofu-vpc" {
	cidr_block = "172.10.0.0/20"					                # VPC에 대한 IPv4 CIDR 블록
	tags = { Name = "cp-opentofu-vpc" }		                                # 리소스에 할당할 태그 맵
}

# VPC 서브넷 리소스 영역
resource "aws_subnet" "cp-opentofu-subnet01" {
  vpc_id = "${aws_vpc.aws-vpc.id}"					                # VPC ID
  cidr_block = "172.10.0.0/24"						                # 서브넷의 IPv4 CIDR 블록
  availability_zone = "ap-northeast-2a"				                        # 서브넷의 AZ
  tags = { Name = "cp-opentofu-subnet01" }	                                        # 리소스에 할당할 태그 맵
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
  name = "cp-opentofu-sg-all"				                                # 보안 그룹의 이름
  description = "Allow all inbound traffic"			                        # 보안 그룹 설명
  vpc_id = "${aws_vpc.aws-vpc.id}"	                                                # VPC ID

  ingress {							                        # 수신 규칙에 대한 구성 블록
	from_port = 0					                                # 시작 포트
	to_port = 0						                        # 끝 범위 포트
	protocol = "-1"					                                # 프로토콜
	cidr_blocks = ["0.0.0.0/0"]		                                        # CIDR 블록 집합
  }

  egress {							                        # 송신 규칙에 대한 구성 블록
	from_port = 0
	to_port = 0
	protocol = "-1"
	cidr_blocks = ["0.0.0.0/0"]
  }
}

# 인스턴스 리소스 영역
resource "aws_instance" "master" {                                                      # 인스턴스 2개 이상 생성시 반드시 "master"와 "worker" 명칭으로 구분
  ami = "${data.aws_ami.ubuntu.id}"							# 인스턴스에 사용할 AMI
  instance_type = "t3.medium"								# 인스턴스에 사용할 인스턴스 유형
  key_name = data.aws_key_pair.default_key.key_name					# 인스턴스에 사용할 키 쌍의 키 이름
  subnet_id = "${aws_subnet.cp-opentofu-subnet01.id}"		                        # 시작할 VPC 서브넷 ID
  vpc_security_group_ids = [								# 연결할 보안 그룹 ID
	"${aws_security_group.cp-opentofu-sg-all.id}"
  ]
  associate_public_ip_address = true			                                # 퍼블릭 IP 주소를 VPC의 인스턴스와 연결할지 여부
  tags = {										# 리소스에 할당할 태그의 맵
	Name = "cp-opentofu-m"
  }
  provisioner "remote-exec" {				                                # OpenTofu로 리소스를 생성하거나 제거할 때 로컬이나 원격에서 스크립트를 실행할 수 있는 기능
	connection {							                # 연결 블록
	  type = "ssh"						                        # 연결 유형
	  host = "${self.public_ip}"			                                # 연결할 리소스의 주소
	  user = "ubuntu"								# 연결에 사용할 사용자
	  private_key = file("~/.ssh/cluster-name-key.pem")		                # 연결에 사용할 SSH 키의 내용
	  timeout = "1m"								# 연결을 사용할 수 있을 때까지 기다리는 시간 초과
	}
	inline = [
	  "cat .ssh/authorized_keys"							# 실행 커맨드
	]
  }
}

resource "aws_instance" "worker" {                                                      # 인스턴스 2개 이상 생성시 반드시 "master"와 "worker" 명칭으로 구분                  
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
  default_route_table_id = "${aws_vpc.aws-vpc.default_route_table_id}"	                # 기본 라우팅 테이블의 ID
  tags = { Name = "New Public Route Table" }						# 리소스에 할당할 태그의 맵
}

# VPC 라우팅 테이블을 생성하기 위한 리소스 영역
resource "aws_route_table" "nc-private" {
  vpc_id = "${aws_vpc.aws-vpc.id}"				                        # VPC ID
  tags = { Name = "New Route Private Table" }	                                        # 리소스에 할당할 태그의 맵
}

# 라우팅 테이블과 서브넷 또는 라우팅 테이블과 인터넷 게이트웨이 또는 가상 프라이빗 게이트웨이 간의 연결을 생성하기 위한 리소스 영역
resource "aws_route_table_association" "aws_public_2a" {
  subnet_id = "${aws_subnet.cp-opentofu-subnet01.id}"		                        # 연결을 생성하기 위한 서브넷 ID
  route_table_id = "${aws_vpc.aws-vpc.default_route_table_id}"		                # 연결할 라우팅 테이블의 ID
}

resource "aws_route_table_association" "aws_private_2a" {
  subnet_id = "${aws_subnet.cp-opentofu-subnet02.id}"
  route_table_id = "${aws_vpc.aws-vpc.default_route_table_id}"
}

# VPC 인터넷 게이트웨이를 생성하기 위한 리소스
resource "aws_internet_gateway" "aws-igw" {
  vpc_id = "${aws_vpc.aws-vpc.id}"					                # 생성할 VPC ID
  tags = { Name = "aws-vpc Internet Gateway" }		                                # 리소스에 할당할 태그 맵
}

# VPC 라우팅 테이블에서 라우팅 테이블 항목(경로)을 생성하기 위한 리소스
resource "aws_route" "aws_public" {
  route_table_id         = "${aws_vpc.aws-vpc.default_route_table_id}"	                # 라우팅 테이블의 ID
  destination_cidr_block = "0.0.0.0/0"							# 대상 CIDR 블록
  gateway_id             = "${aws_internet_gateway.aws-igw.id}"			        # VPC 인터넷 게이트웨이 또는 가상 프라이빗 게이트웨이의 식별자
}

resource "aws_route" "aws_private" {
  route_table_id         = "${aws_route_table.nc-private.id}"
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = "${aws_nat_gateway.aws-nat.id}"
}

# 탄력적 IP 리소스를 제공
resource "aws_eip" "aws_nat" {
  vpc = true		                                                                # EIP가 VPC에 있는 경우, 리전이 EC2-Classic을 지원하지 않는 한 기본값은 true
}

# VPC NAT 게이트웨이를 생성하기 위한 리소스 영역
resource "aws_nat_gateway" "aws-nat" {
  allocation_id = "${aws_eip.aws_nat.id}"					        # 게이트웨이에 대한 탄력적 IP 주소의 할당 ID
  subnet_id     = "${aws_subnet.cp-opentofu-subnet01.id}"		                # 게이트웨이를 배치할 서브넷의 서브넷 ID
}
```

</div>
</details>

#### <div id='6.1.3'> 6.1.3 NHN
Terraman을 사용하여 NHN에서 인스턴스를 생성하는 방법을 설명한다. 기본 IaC 코드는 인스턴스 생성에 맞춰져 있으며, *네트워크, 키페어, 보안 그룹 등*은 이미 IaaS에 생성된 정보를 활용한다. 따라서 인스턴스 이외의 다른 리소스는 미리 생성되어 있어야 한다.
> [NHN IaC 코드 작성시 변수 참고](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs#configuration-reference)
- 인스턴스 생성시 "master"와 "worker" 명칭을 반드시 표기해야 한다.  
*예시*
  + 인스턴스 1개 생성시 
    - resource "openstack_compute_instance_v2" "master" {...}
  + 인스턴스 n개 생성시 
    - resource "openstack_compute_instance_v2" "master" {...}
    - resource "openstack_compute_instance_v2" "worker1" {...}
    - resource "openstack_compute_instance_v2" "worker2" {...}
    - resource "openstack_compute_instance_v2" "worker3" {...} ...

<details>
<summary> :page_facing_up: <b>NHN Instance Code Template</b></summary>
<div markdown="3">

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
  name        = "Ubuntu Server 22.04.3 LTS (2023.11.21)"
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

resource "openstack_networking_port_v2" "nic2" {

  name = "cp-nic02"
  network_id = data.openstack_networking_network_v2.cp-network.id
  fixed_ip {
    subnet_id = data.openstack_networking_subnet_v2.cp-subnet.id
  }
  security_group_ids = [
    data.openstack_networking_secgroup_v2.cp-sg.id
  ]
}

## Manages a V2 VM instance resource within OpenStack.
resource "openstack_compute_instance_v2" "vm-cp-master" {                               # 인스턴스 2개 이상 생성시 반드시 "master"와 "worker" 명칭으로 구분

  name              = "cp-cluster-master"
  availability_zone = "kr-pub-a"
  flavor_name       = "m2.c4m8"
  key_pair          = "nhn-cluster-key"

  block_device {                                                                        # Configuration of block devices.
    uuid                  = data.openstack_images_image_v2.ubuntu_focal.id              # The UUID of the image, volume, or snapshot. Changing this creates a new server.
    source_type           = "image"                                                     # The source type of the device. Must be one of "blank", "image", "volume", or "snapshot". Changing this creates a new server.
    destination_type      = "volume"                                                    # The type that gets created. Possible values are "volume" and "local". Changing this creates a new server.
    delete_on_termination = true                                                        # Delete the volume / block device upon termination of the instance. Defaults to false. Changing this creates a new server.
    volume_size           = 40                                                          # The size of the volume to create (in gigabytes).
    volume_type           = "General HDD"                                               # The volume type that will be used, for example SSD or HDD storage.
  }

  network {
    port = openstack_networking_port_v2.nic.id                                          # The port UUID of a network to attach to the server.
  }
}

## Manages a V2 floating IP resource within OpenStack Neutron (networking) that can be used for load balancers. These are similar to Nova (compute) floating IP resources, but only compute floating IPs can be used with compute instances.
resource "openstack_networking_floatingip_v2" "fip_1" {
  pool       = data.openstack_networking_network_v2.ext_network.name                    # The name of the pool from which to obtain the floating IP. Changing this creates a new floating IP.
}

## Associate a floating IP to an instance.
resource "openstack_compute_floatingip_associate_v2" "fip_1" {
  floating_ip = openstack_networking_floatingip_v2.fip_1.address                        # The floating IP to associate.
  instance_id = openstack_compute_instance_v2.vm-cp-master.id                           # The instance to associte the floating IP with.
  wait_until_associated = true                                                          # In cases where the OpenStack environment does not automatically wait until the association has finished, set this option to have OpenTofu poll the instance until the floating IP has been associated. Defaults to false.
}

resource "openstack_compute_instance_v2" "vm-cp-worker" {
  name              = "cp-cluster-worker"
  availability_zone = "kr-pub-a"
  flavor_name       = "m2.c4m8"
  key_pair          = "nhn-cluster-key"

  block_device {
    uuid                  = data.openstack_images_image_v2.ubuntu_focal.id
    source_type           = "image"
    destination_type      = "volume"
    delete_on_termination = true
    volume_size           = 40
    volume_type           = "General HDD"
  }

  network {
    port = openstack_networking_port_v2.nic2.id
  }
}

resource "openstack_networking_floatingip_v2" "fip_2" {
  pool       = data.openstack_networking_network_v2.ext_network.name
}

resource "openstack_compute_floatingip_associate_v2" "fip_2" {
  floating_ip = openstack_networking_floatingip_v2.fip_2.address
  instance_id = openstack_compute_instance_v2.vm-cp-worker.id
  wait_until_associated = true
}
```

</div>
</details>

<br>

### <div id='6.2'> 6.2 Instance Code Template 등록
컨테이너 플랫폼 포털 메뉴 Global > Instance Code Template 에서 IaC 코드를 등록한다.

<img src="../../images/terraman/IMG_6_2.png">

<br>

## <div id='7'> 7. Clusters
### <div id='7.1'> 7.1 Cluster 생성
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
<img src="../../images/terraman/IMG_7_1_2.png">

##### Cluster Logs 목록 페이지(status 아이콘 클릭)에서 클러스터 생성의 진행 상황을 로그로 확인 가능하다.
<img src="../../images/terraman/IMG_7_1_3.png">

##### 클러스터 배포가 완료되면 화면과 같이 status의 상태 아이콘이 녹색으로 변경된다.
<img src="../../images/terraman/IMG_7_1_4.png">

##### 클러스터 배포가 완료되면 아래와 같이 Overview 페이지에 서브 클러스터로 등록 된 것을 확인할 수 있다.
<img src="../../images/terraman/IMG_7_1_5.png">

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](https://github.com/K-PaaS/container-platform/blob/master/use-guide/Readme.md) > [Terraman 사용 가이드](../cp-terraman-guide.md) > Terraman 배포 가이드
