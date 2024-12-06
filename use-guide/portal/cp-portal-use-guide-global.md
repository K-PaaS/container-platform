### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Global 메뉴

<br>

## Table of Contents

1. [Global 메뉴](#1)  
  1.1. [Overview](#1-1)  
  1.2. [Clusters](#1-2)      
  1.3. [Cloud Accounts](#1-3)  
  1.4. [Instance Code Template](#1-4)    
  1.5. [SSH Keys](#1-5)           
     
<br>


## <div id='1'/> 1. Global 메뉴
### <div id='1-1'/> 1.1. Overview
#### <div id='1-1-1'/> 1.1.1. Overview 정보 조회
- 클러스터 정보 및 TOP Node(CPU, Memory)를 조회한다.
  ![IMG_1_1_1]


<br>

### <div id='1-2'/> 1.2. Clusters
#### <div id='1-2-1'/> 1.2.1. Clusters 목록 조회
- 클러스터 목록을 조회한다.
  ![IMG_1_2_1]

<br>

#### <div id='1-2-2'/> 1.2.2. Clusters 상세 조회
- 클러스터 상세 정보를 조회한다.
  ![IMG_1_2_2]

<br>

#### <div id='1-2-3'/> 1.2.3. Clusters 생성
- 클러스터를 생성하기 위한 과정으로 아래 내역을 입력 후 저장 버튼을 클릭하면 클러스터 환경이 구성된다.
1. **Cluster Name** : 생성할 클러스터 명을 입력한다.
2. **Provider** : 생성할 클러스터의 Provider을 선택한다.([1] AWS [2] OPENSTACK [3] NAVER [4] NHN)
3. **Cloud Accounts** : 선택한 Provider에 맞는 Account 정보를 선택한다.
4. **Template** : VM 배포를 위한 HCL Template을 선택한다.
5. **Description** : 부가정보를 입력한다.(선택)
6. **Template Detail** : 선택된 HCL Template를 기준으로 자기만의 환경정보를 기입한다.
   ![IMG_1_2_3]

<br>

#### <div id='1-2-4'/> 1.2.4. Clusters 등록
- 클러스터를 등록한다.
1. **Cluster Name** : 등록할 클러스터 명을 입력한다.
2. **Provider** : 등록할 클러스터의 Provider을 선택한다.([1] AWS [2] OPENSTACK [3] NAVER [4] NHN [5] KT)
3. **Cluster API URL** : 대상 클러스터 Kubernetes API URL을 입력한다. (e.g: https[]()://xxx.xxx.xxx.xxx:6443)
4. **Description** : 부가정보를 입력한다.(선택)
5. **Cluster Service Token** : 대상 클러스터 접근을 위한 'cluster-admin' 권한의 인증 토큰 정보를 입력한다.
   ![IMG_1_2_4]

<br>

#### <div id='1-2-5'/> 1.2.5. Clusters 수정
- 클러스터 정보를 수정한다.
- 수정이 가능한 항목일 경우 항목 옆의 Edit 버튼을 클릭하여 값 수정이 가능하다.
- 내용 변경 후 하단 수정 버튼을 클릭하면 클러스터 정보가 변경된다.
  ![IMG_1_2_5]

<br>

### <div id='1-3'/> 1.3. Cloud Accounts
#### <div id='1-3-1'/> 1.3.1. Cloud Accounts 목록 조회
- Cloud Accounts 목록을 조회한다.
  ![IMG_1_3_1]

<br>

#### <div id='1-3-2'/> 1.3.2. Cloud Accounts 상세 조회
- Cloud Accounts 정보를 상세 조회한다.
  ![IMG_1_3_2]

<br>

#### <div id='1-3-3'/> 1.3.3. Cloud Accounts 생성
- Cloud Accounts를 생성한다.
- IaC를 통해 자동화된 VM 생성을 위해 필요한 자격 증명에 사용되며, Vault를 통해 안전하게 저장된다.
- Provider 항목에 따라 입력항목이 달라진다.

|  **AWS**  | **OPENSTACK** | **NAVER** |  **NHN**  |
|:---------:|:-------------:|:---------:|:---------:|
| accessKey |   auth_url    | accessKey | auth_url  |
| secretKey |   password    | secretKey | password  |
|  region   |   user_name   |   site    | user_name |
|           |    project    | accessKey |  project  |
|           |    region     |  region   |  region   |

![IMG_1_3_3_1]

<br>

#### <div id='1-3-4'/> 1.3.4. Cloud Accounts 수정
- Cloud Accounts를 수정한다.
- 수정이 가능한 항목일 경우 항목 옆의 Edit 버튼을 클릭하여 값 수정이 가능하다.
- 내용 변경 후 하단 수정 버튼을 클릭하면 Cloud Accounts 정보가 변경된다.
  ![IMG_1_3_4]

<br>

#### <div id='1-3-5'/> 1.3.5. Cloud Accounts 삭제
- Cloud Accounts를 삭제한다.
- 상세화면의 하단 삭제 버튼을 클릭하면 해당 Cloud Account가 삭제된다.
  ![IMG_1_3_5]

<br>

### <div id='1-4'/> 1.4. Instance Code Template
Instance Code Template은 IaC를 통해 자동화된 VM 배포를 수행하는 코드의 템플릿으로 K-PaaS 컨테이너플랫폼을 통해 서브 클러스터를 쉽게 배포할 수 있도록 미리 템플릿을 등록할 수 있으며, 기본적으로 AWS, OPENSTACK, NAVER, NHN을 위한 Template를 제공한다.
#### <div id='1-4-1'/> 1.4.1. Instance Code Template 목록 조회
- Instance Code Template 목록을 조회한다.
  ![IMG_1_4_1]

<br>

#### <div id='1-4-2'/> 1.4.2. Instance Code Template 상세 조회
- Instance Code Template 정보를 상세 조회한다.
  ![IMG_1_4_2]

<br>

#### <div id='1-4-3'/> 1.4.3. Instance Code Template 생성
- Instance Code Template를 생성한다.
  ![IMG_1_4_3]

<br>

#### <div id='1-4-4'/> 1.4.4. Instance Code Template 수정
- Instance Code Template를 수정한다.
- 수정이 가능한 항목일 경우 항목 옆의 Edit 버튼을 클릭하여 값 수정이 가능하다.
- 내용 변경 후 하단 수정 버튼을 클릭하면 Template 정보가 변경된다.
  ![IMG_1_4_4]

<br>

#### <div id='1-4-5'/> 1.4.5. Instance Code Template 삭제
- Instance Code Template를 삭제한다.
- 상세화면의 하단 삭제 버튼을 클릭하면 해당 Instance Code Template가 삭제된다.
  ![IMG_1_4_5]

<br>

### <div id='1-5'/> 1.5. SSH Keys
멀티클러스터 생성을 위한 SSH Key로 K-PaaS 컨테이너플랫폼을 통해 서브 클러스터 Instance에 접속할 수 있는 Key를 등록할 수 있다.
#### <div id='1-5-1'/> 1.5.1. SSH Keys 목록 조회
- SSH Keys 목록을 조회한다.
  ![IMG_1_5_1]

<br>

#### <div id='1-5-2'/> 1.5.2. SSH Keys 상세 조회
- SSH Keys 정보를 상세 조회한다.
  ![IMG_1_5_2]

<br>

#### <div id='1-5-3'/> 1.5.3. SSH Keys 생성
- SSH Keys를 생성한다.
  ![IMG_1_5_3]

<br>

#### <div id='1-5-4'/> 1.5.4. SSH Keys 수정
- SSH Keys를 수정한다.
- 수정이 가능한 항목일 경우 항목 옆의 Edit 버튼을 클릭하여 값 수정이 가능하다.
- 내용 변경 후 하단 수정 버튼을 클릭하면 SSH Key 정보가 변경된다.
  ![IMG_1_5_4]

<br>

#### <div id='1-5-5'/> 1.5.5. SSH Keys 삭제
- SSH Keys를 삭제한다.
- 상세화면의 하단 삭제 버튼을 클릭하면 해당 SSH Keys가 삭제된다.
  ![IMG_1_5_5]

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Global 메뉴

[IMG_1_1_1]:../images/portal/IMG_1_1_1.png
[IMG_1_2_1]:../images/portal/IMG_1_2_1.png
[IMG_1_2_2]:../images/portal/IMG_1_2_2.png
[IMG_1_2_3]:../images/portal/IMG_1_2_3.png
[IMG_1_2_4]:../images/portal/IMG_1_2_4.png
[IMG_1_2_5]:../images/portal/IMG_1_2_5.png
[IMG_1_3_1]:../images/portal/IMG_1_3_1.png
[IMG_1_3_2]:../images/portal/IMG_1_3_2.png
[IMG_1_3_3_1]:../images/portal/IMG_1_3_3_1.png
[IMG_1_3_3_2]:../images/portal/IMG_1_3_3_2.png
[IMG_1_3_4]:../images/portal/IMG_1_3_4.png
[IMG_1_3_5]:../images/portal/IMG_1_3_5.png
[IMG_1_4_1]:../images/portal/IMG_1_4_1.png
[IMG_1_4_2]:../images/portal/IMG_1_4_2.png
[IMG_1_4_3]:../images/portal/IMG_1_4_3.png
[IMG_1_4_4]:../images/portal/IMG_1_4_4.png
[IMG_1_4_5]:../images/portal/IMG_1_4_5.png
[IMG_1_5_1]:../images/portal/IMG_1_5_1.png
[IMG_1_5_2]:../images/portal/IMG_1_5_2.png
[IMG_1_5_3]:../images/portal/IMG_1_5_3.png
[IMG_1_5_4]:../images/portal/IMG_1_5_4.png
[IMG_1_5_5]:../images/portal/IMG_1_5_5.png
