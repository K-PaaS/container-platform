### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  포털 사용 가이드

<br>

## Table of Contents

1. [문서 개요](#1)
   * [1.1. 목적](#1-1)
   * [1.2. 범위](#1-2)
2. [Prerequisite](#2)
   * [2.1. Terraman IaC 설정](#2-1)
3. [컨테이너 플랫폼 포털 접속](#3)
   * [3.1. 컨테이너 플랫폼 포털 관리자 계정 로그인](#3-1)
   * [3.2. 컨테이너 플랫폼 포털 사용자 계정 로그인](#3-2)
4. [컨테이너 플랫폼 포털 구성](#4)
   * [4.1. 컨테이너 플랫폼 포털 사용자 권한 유형](#4-1)
   * [4.2. 컨테이너 플랫폼 포털 메뉴 구성](#4-2)
5. [컨테이너 플랫폼 포털 메뉴 설명](#5)
   * [5.1. Global 메뉴](#5-1)
   * [5.1.1. Overview](#5-1-1)
   * [5.1.2. Clusters](#5-1-2)
   * [5.1.3. Cloud Accounts](#5-1-3)
   * [5.1.4. Instance Code Template](#5-1-4)
   * [5.1.5. SSH Keys](#5-1-5)
   * [5.2. Clusters 메뉴](#5-2)
   * [5.2.1. Overview](#5-2-1)
   * [5.2.2. Nodes](#5-2-2)
   * [5.2.3. Namespaces](#5-2-3)
   * [5.3. Workloads 메뉴](#5-3)
   * [5.3.1. Deployment](#5-3-1)
   * [5.3.2. Pods](#5-3-2)
   * [5.3.3. ReplicaSets](#5-3-3)
   * [5.4. Services 메뉴](#5-4)
   * [5.4.1. Services](#5-4-1)
   * [5.4.2. Ingresses](#5-4-2)
   * [5.5. Storages 메뉴](#5-5)
   * [5.5.1. Persistent Volumes](#5-5-1)
   * [5.5.2. Persistent Volume Claims](#5-5-2)
   * [5.5.3. Storage Classes](#5-5-3)
   * [5.6. ConfigMaps 메뉴](#5-6)
   * [5.6.1. ConfigMaps](#5-6-1)
   * [5.7. Managements 메뉴](#5-7)
   * [5.7.1. Users](#5-7-1)
   * [5.7.2. Roles](#5-7-2)
   * [5.7.3. Resource Quotas](#5-7-3)
   * [5.7.4. Limit Ranges](#5-7-4)
   * [5.8. Info 메뉴](#5-8)
   * [5.8.1. Access](#5-8-1)
   * [5.8.2. Private Repository](#5-8-2)


<br>


# <div id='1'/> 1. 문서 개요

## <div id='1-1'/> 1.1. 목적
본 문서는 컨테이너 플랫폼 포털 사용 방법에 대해 기술하였다.

<br>

## <div id='1-2'/> 1.2. 범위
본 문서는 Windows 환경을 기준으로 컨테이너 플랫폼 포털의 사용 방법에 대해 기술하였다.

<br>

# <div id='2'>2. Prerequisite

## <div id='2-1'>2.1. Terraman IaC 설정
컨테이너 플랫폼 포털을 통해 서브 클러스터를 생성하기 위해서는 대상 클라우드(IaaS)의 추가 설정이 필요하다. <br>
아래 가이드를 참조하여 설정 완료 후 서브 클러스터 생성을 진행한다.
> [Terraman IaC 스크립트 가이드](../../check-guide/cp-terraman-check-index-guide.md)

<br>

## <div id='3'>3. 컨테이너 플랫폼 포털 접속
컨테이너 플랫폼 포털에 접속한다.<br><br>
**컨테이너 플랫폼 포털 URL** : `http://portal.${HOST_DOMAIN}`
+ [[3.1.2. 컨테이너 플랫폼 포털 변수 정의]](../../install-guide/portal/cp-portal-standalone-guide.md#3.1.2) 에서 정의한 `HOST_DOMAIN` 값 입력

<br>

### <div id='3-1'/>3.1. 컨테이너 플랫폼 포털 관리자 계정 로그인
관리자 계정은 패스워드 초기화 설정이 필요하므로 아래 내용을 참조하여 선처리한다.
<details>
<summary><h4> :key: 컨테이너 플랫폼 포털 관리자 계정 패스워드 설정 </h4></summary>

<h1></h1>  

### 1. Keycloak Admin 계정 정보 조회
Keycloak Admin 계정 정보는 아래 명령어를 통해 확인한다.
```bash
# Keycloak Admin 계정 조회
$ kubectl get cm cp-portal-configmap -n cp-portal -o yaml | grep KEYCLOAK_ADMIN
KEYCLOAK_ADMIN_USERNAME: ********* (Username)
KEYCLOAK_ADMIN_PASSWORD: ********* (Password)
```

<br>

### 2. Keycloak Admin Console 접속 및 로그인
Keycloak Admin Console에 접속 후 조회한 Keycloak Admin 계정으로 로그인한다.<br><br>

**Keycloak Admin Console URL** : `http://keycloak.${HOST_DOMAIN}/auth/admin`
+ Keycloak TLS 적용 시 `https` 로 접속
+ [[3.1.2. 컨테이너 플랫폼 포털 변수 정의]](../../install-guide/portal/cp-portal-standalone-guide.md#3.1.2) 에서 정의한 `HOST_DOMAIN` 값 입력

![image 011]

<br>

### 3. 컨테이너 플랫폼 관리자 계정 패스워드 초기화
- 왼쪽 상단의 Realm 정보를 `Cp-realm` 으로 변경한다.

![image 012]

- 메뉴 [Users] 선택 후 Username 이 `admin`인 계정을 클릭한다.

![image 013]

- 메뉴 [Credentials] 선택 후 버튼 [Reset password]을 클릭하여 패스워드 초기화를 진행한다.
   + **Password** : `초기화할 패스워드 값 입력`
   + **Temporary** : `Off` 선택
      - 'On' 으로 선택할 경우 사용자는 다음 로그인 시 비밀번호 재변경 필요

![image 014]
![image 015]

<h1></h1>

</details>

- 관리자 계정으로 컨테이너 플랫폼 포털에 로그인한다.
   + **Username** : `admin`
   + **Password** : `초기화한 패스워드 값`

![image 002]

<br>

### <div id='3-2'/>3.2. 컨테이너 플랫폼 포털 사용자 계정 로그인
#### 사용자 회원가입
- 하단의 'Register' 버튼을 클릭한다.

![image 003]

- 등록할 사용자 계정정보를 입력 후 'Register' 버튼을 클릭하여 컨테이너 플랫폼 포털에 회원가입한다.

![image 004]

- 회원가입 후 바로 포털 접속이 불가하며 관리자로부터 해당 사용자가 이용할 Namespace와 Role을 할당 받은 후 포털 이용이 가능하다.
  사용자에게 Namespace 및 Role 할당은 [[5.7.1.5. User 수정]](#5-7-1-5) 를 참고한다.

![image 005]

#### 사용자 로그인
- 회원가입을 통해 등록된 계정으로 컨테이너 플랫폼 포털에 로그인한다.

![image 006]

<br>    

# <div id='4'/> 4. 컨테이너 플랫폼 포털 구성
## <div id='4-1'/> 4.1. 컨테이너 플랫폼 포털 사용자 권한 유형
| <center>사용자 유형</center> | <center>기능</center> |
| :--- | :--- |
| Super Admin (전체 관리자) | 전체 클러스터 관리 권한  |
| Cluster Admin (클러스터 관리자) | 할당된 클러스터 관리 권한  |
| User (일반 사용자) | 할당된 클러스터 내 네임스페이스 관리 권한|

<br>

## <div id='4-2'/> 4.2. 컨테이너 플랫폼 포털 메뉴 구성
<table style="table-layout: fixed; width: 816px;>
<colgroup>
<col style="width: 126px">
<col style="width: 174px">
<col style="width: 307px">
<col style="width: 209px">
</colgroup>
<thead>
  <tr>
    <th>메뉴</th>
    <th>분류</th>
    <th>설명</th>
    <th>접근 권한</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="4">Global</td>
    <td>Overview</td>
    <td>클러스터 정보 대시보드</td>
    <td rowspan="4">Super Admin <br> Cluster Admin (조회 권한)</td>
  </tr>
  <tr>
    <td>Clusters</td>
    <td>클러스터 정보 관리</td>
  </tr>
  <tr>
    <td>Cloud Accounts</td>
    <td>Cloud Accounts 정보 관리</td>
  </tr>
  <tr>
    <td>Instance Code Template</td>
    <td>Instance Code Template 정보 관리</td>
  </tr>
  <tr>
    <td rowspan="3">Clusters</td>
    <td>Overview</td>
    <td>클러스터 내 리소스 대시보드</td>
    <td>All</td>
  </tr>
  <tr>
    <td>Nodes</td>
    <td>Nodes 정보 관리</td>
    <td>Super Admin, Cluster Admin</td>
  </tr>
  <tr>
    <td>Namespaces</td>
    <td>Namespaces 정보 관리</td>
    <td>All</td>
  </tr>
  <tr>
    <td rowspan="3">Workloads</td>
    <td>Deployments</td>
    <td>Deployments 정보 관리</td>
    <td rowspan="3">All</td>
  </tr>
  <tr>
    <td>Pods</td>
    <td>Pods 정보 관리</td>
  </tr>
  <tr>
    <td>ReplicaSets</td>
    <td>ReplicaSets 정보 관리</td>
  </tr>
  <tr>
    <td rowspan="2">Services</td>
    <td>Services</td>
    <td>Services 정보 관리</td>
    <td rowspan="2">All</td>
  </tr>
  <tr>
    <td>Ingresses</td>
    <td> Ingresses 정보 관리</td>
  </tr>
  <tr>
    <td rowspan="3">Storages</td>
    <td>Persistent Volumes</td>
    <td>Persistent Volumes 정보 관리</td>
    <td>Super Admin, Cluster Admin</td>
  </tr>
  <tr>
    <td>Persistent Volume Claims</td>
    <td>Persistent Volume Claims 정보 관리</td>
    <td>All</td>
  </tr>
  <tr>
    <td>Storage Classes</td>
    <td>Storage Classes 정보 관리</td>
    <td>Super Admin, Cluster Admin</td>
  </tr>
  <tr>
    <td>ConfigMaps</td>
    <td>ConfigMaps</td>
    <td> ConfigMaps 정보 관리</td>
    <td>All</td>
  </tr>
  <tr>
    <td rowspan="4">Managements</td>
    <td>Users</td>
    <td>사용자 관리</td>
    <td>Super Admin, Cluster Admin</td>
  </tr>
  <tr>
    <td>Roles</td>
    <td>Roles 관리</td>
    <td rowspan="3">All</td>
  </tr>
  <tr>
    <td>Resource Quotas</td>
    <td>Resource Quotas 정보 관리</td>
  </tr>
  <tr>
    <td>Limit Ranges</td>
    <td>Limit Ranges 정보 관리</td>
  </tr>
  <tr>
    <td rowspan="2">Info</td>
    <td>Access</td>
    <td>컨테이너 플랫폼 포털  <br>CLI 사용을 위한 환경 설정 정보 관리</td>
    <td rowspan="2">All</td>
  </tr>
  <tr>
    <td>Private Repository</td>
    <td>Private Repository 정보 관리</td>
  </tr>
</tbody>
</table>

<br>

# <div id='5'/> 5. 컨테이너 플랫폼 포털 메뉴 설명
본 장에서는 컨테이너 플랫폼 포털의 메뉴에 대한 설명을 기술하였다.

<br>

## <div id='5-1'/> 5.1. Global 메뉴
### <div id='5-1-1'/> 5.1.1. Overview
#### <div id='5-1-1-1'/> 5.1.1.1. Overview 정보 조회
- 클러스터 정보 및 TOP Node(CPU, Memory)를 조회한다.
  ![IMG_1_1_1]


<br>

### <div id='5-1-2'/> 5.1.2. Clusters
#### <div id='5-1-2-1'/> 5.1.2.1. Clusters 목록 조회
- 클러스터 목록을 조회한다.
  ![IMG_1_2_1]

<br>

#### <div id='5-1-2-2'/> 5.1.2.2. Clusters 상세 조회
- 클러스터 상세 정보를 조회한다.
  ![IMG_1_2_2]

<br>

#### <div id='5-1-2-3'/> 5.1.2.3. Clusters 생성
- 클러스터를 생성하기 위한 과정으로 아래 내역을 입력 후 저장 버튼을 클릭하면 클러스터 환경이 구성된다.
1. **Cluster Name** : 생성할 클러스터 명을 입력한다.
2. **Provider** : 생성할 클러스터의 Provider을 선택한다.([1] AWS [2] OPENSTACK [3] NAVER [4] NHN)
3. **Cloud Accounts** : 선택한 Provider에 맞는 Account 정보를 선택한다.
4. **Template** : VM 배포를 위한 HCL Template을 선택한다.
5. **Description** : 부가정보를 입력한다.(선택)
6. **Template Detail** : 선택된 HCL Template를 기준으로 자기만의 환경정보를 기입한다.
   ![IMG_1_2_3]

<br>

#### <div id='5-1-2-4'/> 5.1.2.4. Clusters 등록
- 클러스터를 등록한다.
1. **Cluster Name** : 등록할 클러스터 명을 입력한다.
2. **Provider** : 등록할 클러스터의 Provider을 선택한다.([1] AWS [2] OPENSTACK [3] NAVER [4] NHN [5] KT)
3. **Cluster API URL** : 대상 클러스터 Kubernetes API URL을 입력한다. (e.g: https[]()://xxx.xxx.xxx.xxx:6443)
4. **Description** : 부가정보를 입력한다.(선택)
5. **Cluster Service Token** : 대상 클러스터 접근을 위한 'cluster-admin' 권한의 인증 토큰 정보를 입력한다.
   ![IMG_1_2_4]

<br>

#### <div id='5-1-2-5'/> 5.1.2.5. Clusters 수정
- 클러스터 정보를 수정한다.
- 수정이 가능한 항목일 경우 항목 옆의 Edit 버튼을 클릭하여 값 수정이 가능하다.
- 내용 변경 후 하단 수정 버튼을 클릭하면 클러스터 정보가 변경된다.
  ![IMG_1_2_5]

<br>

### <div id='5-1-3'/> 5.1.3. Cloud Accounts
#### <div id='5-1-3-1'/> 5.1.3.1. Cloud Accounts 목록 조회
- Cloud Accounts 목록을 조회한다.
  ![IMG_1_3_1]

<br>

#### <div id='5-1-3-2'/> 5.1.3.2. Cloud Accounts 상세 조회
- Cloud Accounts 정보를 상세 조회한다.
  ![IMG_1_3_2]

<br>

#### <div id='5-1-3-3'/> 5.1.3.3. Cloud Accounts 생성
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

#### <div id='5-1-3-4'/> 5.1.3.4. Cloud Accounts 수정
- Cloud Accounts를 수정한다.
- 수정이 가능한 항목일 경우 항목 옆의 Edit 버튼을 클릭하여 값 수정이 가능하다.
- 내용 변경 후 하단 수정 버튼을 클릭하면 Cloud Accounts 정보가 변경된다.
  ![IMG_1_3_4]

<br>

#### <div id='5-1-3-5'/> 5.1.3.5. Cloud Accounts 삭제
- Cloud Accounts를 삭제한다.
- 상세화면의 하단 삭제 버튼을 클릭하면 해당 Cloud Account가 삭제된다.
  ![IMG_1_3_5]

<br>

### <div id='5-1-4'/> 5.1.4. Instance Code Template
Instance Code Template은 IaC를 통해 자동화된 VM 배포를 수행하는 코드의 템플릿으로 K-PaaS 컨테이너플랫폼을 통해 서브 클러스터를 쉽게 배포할 수 있도록 미리 템플릿을 등록할 수 있으며, 기본적으로 AWS, OPENSTACK, NAVER, NHN을 위한 Template를 제공한다.
#### <div id='5-1-4-1'/> 5.1.4.1. Instance Code Template 목록 조회
- Instance Code Template 목록을 조회한다.
  ![IMG_1_4_1]

<br>

#### <div id='5-1-4-2'/> 5.1.4.2. Instance Code Template 상세 조회
- Instance Code Template 정보를 상세 조회한다.
  ![IMG_1_4_2]

<br>

#### <div id='5-1-4-3'/> 5.1.4.3. Instance Code Template 생성
- Instance Code Template를 생성한다.
  ![IMG_1_4_3]

<br>

#### <div id='5-1-4-4'/> 5.1.4.4. Instance Code Template 수정
- Instance Code Template를 수정한다.
- 수정이 가능한 항목일 경우 항목 옆의 Edit 버튼을 클릭하여 값 수정이 가능하다.
- 내용 변경 후 하단 수정 버튼을 클릭하면 Template 정보가 변경된다.
  ![IMG_1_4_4]

<br>

#### <div id='5-1-4-5'/> 5.1.4.5. Instance Code Template 삭제
- Instance Code Template를 삭제한다.
- 상세화면의 하단 삭제 버튼을 클릭하면 해당 Instance Code Template가 삭제된다.
  ![IMG_1_4_5]

<br>

### <div id='5-1-5'/> 5.1.5. SSH Keys
멀티클러스터 생성을 위한 SSH Key로 K-PaaS 컨테이너플랫폼을 통해 서브 클러스터 Instance에 접속할 수 있는 Key를 등록할 수 있다.
#### <div id='5-1-5-1'/> 5.1.5.1. SSH Keys 목록 조회
- SSH Keys 목록을 조회한다.
  ![IMG_1_5_1]

<br>

#### <div id='5-1-5-2'/> 5.1.5.2. SSH Keys 상세 조회
- SSH Keys 정보를 상세 조회한다.
  ![IMG_1_5_2]

<br>

#### <div id='5-1-5-3'/> 5.1.5.3. SSH Keys 생성
- SSH Keys를 생성한다.
  ![IMG_1_5_3]

<br>

#### <div id='5-1-5-4'/> 5.1.5.4. SSH Keys 수정
- SSH Keys를 수정한다.
- 수정이 가능한 항목일 경우 항목 옆의 Edit 버튼을 클릭하여 값 수정이 가능하다.
- 내용 변경 후 하단 수정 버튼을 클릭하면 SSH Key 정보가 변경된다.
  ![IMG_1_5_4]

<br>

#### <div id='5-1-5-5'/> 5.1.5.5. SSH Keys 삭제
- SSH Keys를 삭제한다.
- 상세화면의 하단 삭제 버튼을 클릭하면 해당 SSH Keys가 삭제된다.
  ![IMG_1_5_5]

<br>  

## <div id='5-2'/> 5.2. Clusters 메뉴
### <div id='5-2-1'/> 5.2.1. Overview
#### <div id='5-2-1-1'/> 5.2.1.1. Overview 정보 조회
- Namespace, Deployment, Pod, User의 개수와 Deployment, Pod, ReplicaSet의 차트를 조회한다.
  ![IMG_2_1_1]

<br>

#### <div id='5-2-1-2'/> 5.2.1.2. Overview 클러스터 변경
- Select Box에서 클러스터를 선택하면 해당 클러스터에 대한 정보가 조회된다.
- 클러스터 선택 후 Select Box에서 Namespace를 선택하면 해당 Namespace에 대한 정보가 조회된다.
- Namespace **'ALL'** 의 경우 전체 Namespace에 대한 정보가 조회된다.
  ![IMG_2_1_2_1]
  ![IMG_2_1_2_2]

<br>

### <div id='5-2-2'/> 5.2.2. Nodes
#### <div id='5-2-2-1'/> 5.2.2.1. Node 목록 조회
- Clusters 메뉴의 Nodes를 클릭하여 Node 목록 페이지로 이동한다.
  ![IMG_2_2_1]

<br>

#### <div id='5-2-2-2'/> 5.2.2.2. Node 상세 조회
- Node 목록에서 Node명을 클릭하여 Node 상세 페이지로 이동한다.
  ![IMG_2_2_2]

<br>

### <div id='5-2-3'/> 5.2.3. Namespaces
#### <div id='5-2-3-1'/> 5.2.3.1. Namespace 목록 조회
- Clusters 메뉴의 Namespaces 클릭하여 Namespace 목록 페이지로 이동한다.
  ![IMG_2_3_1]

<br>

#### <div id='5-2-3-2'/> 5.2.3.2. Namespace 상세 조회
- Namespace 목록에서 Namespace명을 클릭하여 Namespace 상세 페이지로 이동한다.
  ![IMG_2_3_2]

<br>

#### <div id='5-2-3-3'/> 5.2.3.3. Namespace 생성
- Namespace 목록에서 생성 버튼을 클릭하여 Namespace 생성 페이지로 이동한다.
- Namespace 생성 페이지에서 **Resource Quotas**, **Limit Ranges** 를 지정할 수 있다.
- Namespace 생성 페이지 하단의 저장 버튼을 클릭하여 Namespace를 생성한다.
  ![IMG_2_3_3_1]
  ![IMG_2_3_3_2]
  ![IMG_2_3_3_3]

<br>

#### <div id='5-2-3-4'/> 5.2.3.4. Namespace 수정
- Namespace 상세에서 수정 버튼을 클릭하여 Namespace 수정 페이지로 이동한다.
- Namespace 수정 페이지에서 **Resource Quotas**, **Limit Ranges**를 수정할 수 있다.
- Namespace 수정 페이지 하단의 저장 버튼을 클릭하여 Namespace를 수정한다.
  ![IMG_2_3_4]

<br>

#### <div id='5-2-3-5'/> 5.2.3.5. Namespace 삭제
- Namespace 상세에서 삭제 버튼을 클릭하여 Namespace를 삭제한다.
  ![IMG_2_3_5]

<br>

## <div id='5-3'/> 5.3. Workloads 메뉴
### <div id='5-3-1'/> 5.3.1. Deployments
#### <div id='5-3-1-1'/> 5.3.1.1. Deployment 목록 조회
- Workloads의 Deployments를 클릭하여 Deployment 목록 페이지로 이동한다.
  ![IMG_3_1_1]

<br>

#### <div id='5-3-1-2'/> 5.3.1.2. Deployment 상세 조회
- Deployment 목록에서 Deployment명을 클릭하여 Deployment 상세 페이지로 이동한다.
  ![IMG_3_1_2]

<br>

#### <div id='5-3-1-3'/> 5.3.1.3. Deployment 생성
- Deployment 목록에서 생성 버튼을 클릭할 시 Deployment 생성 팝업창이 뜬다.
  ![IMG_3_1_3]

<br>

#### <div id='5-3-1-4'/> 5.3.1.4. Deployment 수정
- Deployment 상세에서 수정 버튼을 클릭할 시 Deployment 수정 팝업창이 뜬다.
  ![IMG_3_1_4]

<br>

#### <div id='5-3-1-5'/> 5.3.1.5. Deployment 삭제
- Deployment 상세에서 삭제 버튼을 클릭할 시 Deployment가 삭제된다.
  ![IMG_3_1_5]

<br>


### <div id='5-3-2'/> 5.3.2. Pods
#### <div id='5-3-2-1'/> 5.3.2.1. Pod 목록 조회
- Workloads의 Pods를 클릭하여 Pod 목록 페이지로 이동한다.
  ![IMG_3_2_1]

<br>

#### <div id='5-3-2-2'/> 5.3.2.2. Pod 상세 조회
- Pod 목록에서 Pod명을 클릭하여 Pod 상세 페이지로 이동한다.
  ![IMG_3_2_2]

<br>

#### <div id='5-3-2-3'/> 5.3.2.3. Pod 생성
- Pod 목록에서 생성 버튼을 클릭할 시 Pod 생성 팝업창이 뜬다.
  ![IMG_3_2_3]

<br>

#### <div id='5-3-2-4'/> 5.3.2.4. Pod 수정
- Pod 상세에서 수정 버튼을 클릭할 시 Pod 수정 팝업창이 뜬다.
  ![IMG_3_2_4]

<br>

#### <div id='5-3-2-5'/> 5.3.2.5. Pod
- Pod 상세에서 삭제 버튼을 클릭할 시 Pod 삭제가 완료된다.
  ![IMG_3_2_5]

<br>

### <div id='5-3-3'/> 5.3.3. ReplicaSets
#### <div id='5-3-3-1'/> 5.3.3.1. ReplicaSet 목록 조회
- Workloads의 ReplicaSets을 클릭하여 ReplicaSet 목록 페이지로 이동한다.
  ![IMG_3_3_1]

<br>

#### <div id='5-3-3-2'/> 5.3.3.2. ReplicaSet 상세 조회
- ReplicaSet 목록에서 ReplicaSet명을 클릭하여 ReplicaSet 상세 페이지로 이동한다.
  ![IMG_3_3_2]

<br>

#### <div id='5-3-3-3'/> 5.3.3.3. ReplicaSet 생성
- ReplicaSet 목록에서 생성 버튼을 클릭할 시 ReplicaSet 생성 팝업창이 뜬다.
  ![IMG_3_3_3]

<br>

#### <div id='5-3-3-4'/> 5.3.3.4. ReplicaSet 수정
- ReplicaSet 상세에서 수정 버튼을 클릭할 시 ReplicaSet 수정 팝업창이 뜬다.
  ![IMG_3_3_4]

<br>

#### <div id='5-3-3-5'/> 5.3.3.5. ReplicaSet 삭제
- ReplicaSet 상세에서 삭제 버튼을 클릭할 시 ReplicaSet 삭제가 완료된다.
  ![IMG_3_3_5]

<br>

## <div id='5-4'/> 5.4. Services 메뉴
### <div id='5-4-1'/> 5.4.1. Services
#### <div id='5-4-1-1'/> 5.4.1.1. Service 목록 조회
- Services의 Services을 클릭하여 Service 목록 페이지로 이동한다.
  ![IMG_4_1_1]

<br>

#### <div id='5-4-1-2'/> 5.4.1.2. Service 상세 조회
- Service 목록에서 Service명을 클릭하여 Service 상세 페이지로 이동한다.
  ![IMG_4_1_2]

<br>

#### <div id='5-4-1-3'/> 5.4.1.3. Service 생성
- Service 목록에서 생성 버튼을 클릭할 시 Service 생성 팝업창이 뜬다.
  ![IMG_4_1_3]

<br>

#### <div id='5-4-1-4'/> 5.4.1.4. Service 수정
- Service 상세에서 수정 버튼을 클릭할 시 Service 수정 팝업창이 뜬다.
  ![IMG_4_1_4]

<br>

#### <div id='5-4-1-5'/> 5.4.1.5. Service 삭제
- Service 상세에서 삭제 버튼을 클릭할 시 Service 삭제가 완료된다.
  ![IMG_4_1_5]

<br>

### <div id='5-4-2'/> 5.4.2. Ingresses
#### <div id='5-4-2-1'/> 5.4.2.1. Ingress 목록 조회
- Services의 Ingresses을 클릭하여 Ingress 목록 페이지로 이동한다.
  ![IMG_4_2_1]

<br>

#### <div id='5-4-2-2'/> 5.4.2.2. Ingress 상세 조회
- Ingress 목록에서 Ingress명을 클릭하여 Ingress 상세 페이지로 이동한다.
  ![IMG_4_2_2]

<br>

#### <div id='5-4-2-3'/> 5.4.2.3. Ingress 생성
- Ingress 목록에서 생성 버튼을 클릭할 시 Ingress 생성 페이지로 이동한다.
- **Rule** (Host, Path, Target) 정보를 입력한다.
  ![IMG_4_2_3]

<br>

#### <div id='5-4-2-4'/> 5.4.2.4. Ingress 수정
- 수정이 가능한 항목일 경우 항목 옆의 아이콘을 클릭하여 값 수정이 가능하다.
- 내용 변경 후 하단 수정 버튼을 클릭하면 Ingress 정보가 변경된다.
  ![IMG_4_2_4]

<br>

#### <div id='5-4-2-5'/> 5.4.2.5. Ingress 삭제
- Ingress 상세에서 삭제 버튼을 클릭할 시 Ingress 삭제가 완료된다.
  ![IMG_4_2_5]

<br>

## <div id='5-5'/> 5.5. Storages 메뉴
### <div id='5-5-1'/> 5.5.1. Persistent Volumes
#### <div id='5-5-1-1'/> 5.5.1.1. Persistent Volume 목록 조회
- Storages의 Persistent Volumes를 클릭하여 Persistent Volume 목록 페이지로 이동한다.
  ![IMG_5_1_1]

<br>

#### <div id='5-5-1-2'/> 5.5.1.2. Persistent Volume 상세 조회
- Persistent Volume 목록에서 Persistent Volume명을 클릭하여 Persistent Volume 상세 페이지로 이동한다.
  ![IMG_5_1_2]

<br>

#### <div id='5-5-1-3'/> 5.5.1.3. Persistent Volume 생성
- Persistent Volume 목록에서 생성 버튼을 클릭할 시 Persistent Volume 생성 팝업창이 뜬다.
  ![IMG_5_1_3]

<br>

#### <div id='5-5-1-4'/> 5.5.1.4. Persistent Volume 수정
- Persistent Volume 상세에서 수정 버튼을 클릭할 시 Persistent Volume 수정 팝업창이 뜬다.
  ![IMG_5_1_4]

<br>

#### <div id='5-5-1-5'/> 5.5.1.5. Persistent Volume 삭제
- Persistent Volume 상세에서 삭제 버튼을 클릭할 시 Persistent Volume 삭제가 완료된다.
  ![IMG_5_1_5]

<br>

### <div id='5-5-2'/> 5.5.2. Persistent Volume Claims
#### <div id='5-5-2-1'/> 5.5.2.1. Persistent Volume Claim 목록 조회
- Storages의 Persistent Volume Claims를 클릭하여 Persistent Volume Claim 목록 페이지로 이동한다.
  ![IMG_5_2_1]

<br>

#### <div id='5-5-2-2'/> 5.5.2.2. Persistent Volume Claim 상세 조회
- Persistent Volume Claim 목록에서 Persistent Volume Claim명을 클릭하여 Persistent Volume Claim 상세 페이지로 이동한다.
  ![IMG_5_2_2]

<br>

#### <div id='5-5-2-3'/> 5.5.2.3. Persistent Volume Claim 생성
- Persistent Volume Claim 목록에서 생성 버튼을 클릭할 시 Persistent Volume Claim 생성 팝업창이 뜬다.
  ![IMG_5_2_3]

<br>

#### <div id='5-5-2-4'/> 5.5.2.4. Persistent Volume Claim 수정
- Persistent Volume Claim 상세에서 수정 버튼을 클릭할 시 Persistent Volume Claim 수정 팝업창이 뜬다.
  ![IMG_5_2_4]

<br>

#### <div id='5-5-2-5'/> 5.5.2.5. Persistent Volume Claim 삭제
- Persistent Volume Claim 상세에서 삭제 버튼을 클릭할 시 Persistent Volume Claim 삭제가 완료된다.
  ![IMG_5_2_5]

<br>

### <div id='5-5-3'/> 5.5.3. Storage Classes
#### <div id='5-5-3-1'/> 5.5.3.1. Storage Class 목록 조회
- Storages의 Storage Classes를 클릭하여 Storage Class 목록 페이지로 이동한다.
  ![IMG_5_3_1]

<br>

#### <div id='5-5-3-2'/> 5.5.3.2. Storage Class 상세 조회
- Storage Class 목록에서 Storage Class명을 클릭하여 Storage Class 상세 페이지로 이동한다.
  ![IMG_5_3_2]

<br>

#### <div id='5-5-3-3'/> 5.5.3.3. Storage Class 생성
- Storage Class 목록에서 생성 버튼을 클릭할 시 Storage Class 생성 팝업창이 뜬다.
  ![IMG_5_3_3]

<br>

#### <div id='5-5-3-4'/> 5.5.3.4. Storage Class 수정
- Storage Class 상세에서 수정 버튼을 클릭할 시 Storage Class 수정 팝업창이 뜬다.
  ![IMG_5_3_4]

<br>

#### <div id='5-5-3-5'/> 5.5.3.5. Storage Class 삭제
- Storage Class 상세에서 삭제 버튼을 클릭할 시 Storage Class 삭제가 완료된다.
  ![IMG_5_3_5]

<br>

## <div id='5-6'/> 5.6. ConfigMaps 메뉴
### <div id='5-6-1'/> 5.6.1. ConfigMaps
#### <div id='5-6-1-1'/> 5.6.1.1. ConfigMap 목록 조회
- ConfigMaps의 ConfigMaps를 클릭하여 ConfigMap 목록 페이지로 이동한다.
  ![IMG_6_1_1]

<br>

#### <div id='5-6-1-2'/> 5.6.1.2. ConfigMap 상세 조회
- ConfigMap 목록에서 ConfigMap명을 클릭하여 ConfigMap 상세 페이지로 이동한다.
  ![IMG_6_1_2]

<br>

#### <div id='5-6-1-3'/> 5.6.1.3. ConfigMap 생성
- ConfigMap 목록에서 생성 버튼을 클릭할 시 ConfigMap 생성 팝업창이 뜬다.
  ![IMG_6_1_3]

<br>

#### <div id='5-6-1-4'/> 5.6.1.4. ConfigMap 수정
- ConfigMap 상세에서 수정 버튼을 클릭할 시 ConfigMap 수정 팝업창이 뜬다.
  ![IMG_6_1_4]

<br>

#### <div id='5-6-1-5'/> 5.6.1.5. ConfigMap 삭제
- ConfigMap 상세에서 삭제 버튼을 클릭할 시 ConfigMap 삭제가 완료된다.
  ![IMG_6_1_5]

<br>

## <div id='5-7'/> 5.7. Managements
### <div id='5-7-1'/> 5.7.1. Users
#### <div id='5-7-1-1'/> 5.7.1.1. 클러스터 관리자 조회
- Managements 메뉴의 Users를 선택하고 Administrator탭을 클릭하여 클러스터 관리자를 조회한다.
- 클러스터 관리자 권한은 한 명 이상 가능하다.
  ![IMG_7_1_1]

<br>

#### <div id='5-7-1-2'/> 5.7.1.2. 클러스터 관리자 상세 조회
- 클러스터 관리자 User ID를 클릭하여 클러스터 관리자 상세 조회 페이지로 이동한다.
  ![IMG_7_1_2]

<br>

#### <div id='5-7-1-3'/> 5.7.1.3. 일반 사용자 목록 조회
- Managements 메뉴의 Users를 선택하고 User탭을 클릭하여 사용자 목록을 조회한다.
   + 활성(Active) 탭 : 해당 클러스터 내 Namespace/Role 이 할당된 사용자 목록
   + 비활성(Inactive) 탭 : 해당 클러스터 내 Namespace/Role 이 비할당된 사용자 목록
     ![IMG_7_1_3_1]
     ![IMG_7_1_3_2]

<br>

#### <div id='5-7-1-4'/> 5.7.1.4. 일반 사용자 상세 조회
- 일반 사용자 User ID를 클릭하여 일반 사용자 상세 조회 페이지로 이동한다.
  ![IMG_7_1_4]

<br>

#### <div id='5-7-1-5'/> 5.7.1.5. User 수정
- User 상세에서 수정 버튼을 클릭하여 User 수정 페이지로 이동한다.
- User 수정 페이지에서 해당 사용자에게 **클러스터 관리자(Cluster Admin)** 또는 **일반 사용자(User)** 권한 할당이 가능하다.

<br>

**클러스터 관리자(Cluster Admin) 권한 설정**
>`'Authority' 항목 : 'Cluster Admin'으로 선택`

![IMG_7_1_5_1]

<br>

**일반 사용자(User) 권한 설정**
>`'Authority' 항목 : 'User'으로 선택`<br>
>`'Namespaces ⁄ Roles' 항목 : 선택 버튼 클릭하여 해당 사용자에게 할당할 Namespace/Role 선택`

![IMG_7_1_5_2]
![IMG_7_1_5_3]
![IMG_7_1_5_4]

<br>

##### 하단 수정 버튼 클릭하여 변경된 User 권한을 반영한다.
![IMG_7_1_5_5]

<br>

### <div id='5-7-2'/> 5.7.2. Roles
#### <div id='5-7-2-1'/> 5.7.2.1. Role 목록 조회
- Managements 메뉴의 Roles를 클릭하여 Role 목록 페이지로 이동한다.
  ![IMG_7_2_1]

<br>

#### <div id='5-7-2-2'/> 5.7.2.2. Role 상세 조회
- Role 목록에서 Role명을 클릭하여 Role 상세 페이지로 이동한다.
  ![IMG_7_2_2]

<br>

#### <div id='5-7-2-3'/> 5.7.2.3. Role 생성
- Role 목록에서 생성 버튼을 클릭할 시 Role 생성 팝업창이 뜬다.
  ![IMG_7_2_3]

<br>

#### <div id='5-7-2-4'/> 5.7.2.4. Role 수정
- Role 상세에서 수정 버튼을 클릭할 시 Role 수정 팝업창이 뜬다.
  ![IMG_7_2_4]

<br>

#### <div id='5-7-2-5'/> 5.7.2.5. Role 삭제
- Role 상세에서 삭제 버튼을 클릭할 시 Role이 삭제된다.
  ![IMG_7_2_5]

<br>

### <div id='5-7-3'/> 5.7.3. Resource Quotas
#### <div id='5-7-3-1'/> 5.7.3.1. Resource Quota 목록 조회
- Managements 메뉴의 Resource Quotas를 클릭하여 Resource Quota 목록 페이지로 이동한다.
  ![IMG_7_3_1]

<br>

#### <div id='5-7-3-2'/> 5.7.3.2. Resource Quota 상세 조회
- Resource Quota 목록에서 Resource Quota명을 클릭하여 Resource Quota 상세 페이지로 이동한다.
  ![IMG_7_3_2]

<br>

#### <div id='5-7-3-3'/> 5.7.3.3. Resource Quota 생성
- Resource Quota 목록에서 생성 버튼을 클릭할 시 Resource Quota 생성 팝업창이 뜬다.
  ![IMG_7_3_3]

<br>

#### <div id='5-7-3-4'/> 5.7.3.4. Resource Quota 수정
- Resource Quota 상세에서 수정 버튼을 클릭할 시 Resource Quota 수정 팝업창이 뜬다.
  ![IMG_7_3_4]

<br>

#### <div id='5-7-3-5'/> 5.7.3.5. Resource Quota 삭제
- Resource Quota 상세에서 삭제 버튼을 클릭할 시 Resource Quota가 삭제된다.
  ![IMG_7_3_5]

<br>

### <div id='5-7-4'/> 5.7.4. Limit Ranges
#### <div id='5-7-4-1'/> 5.7.4.1. Limit Range 목록 조회
- Managements 메뉴의 Limit Ranges를 클릭하여 Limit Range 목록 페이지로 이동한다.
  ![IMG_7_4_1]

<br>

#### <div id='5-7-4-2'/> 5.7.4.2. Limit Range 상세 조회
- Limit Range 목록에서 Limit Range명을 클릭하여 Limit Range 상세 페이지로 이동한다.
  ![IMG_7_4_2]

<br>

#### <div id='5-7-4-3'/> 5.7.4.3. Limit Range 생성
- Limit Range 목록에서 생성 버튼을 클릭할 시 Limit Range 생성 팝업창이 뜬다.
  ![IMG_7_4_3]

<br>

#### <div id='5-7-4-4'/> 5.7.4.4. Limit Range 수정
- Limit Range 상세에서 수정 버튼을 클릭할 시 Limit Range 수정 팝업창이 뜬다.
  ![IMG_7_4_4]

<br>

#### <div id='5-7-4-5'/> 5.7.4.5. Limit Range 삭제
- Limit Range 상세에서 삭제 버튼을 클릭할 시 Limit Range가 삭제된다.
  ![IMG_7_4_5]

<br>

## <div id='5-8'/> 5.8. Info 메뉴
### <div id='5-8-1'/> 5.8.1. Access
#### <div id='5-8-1-1'/> 5.8.1.1. Access 정보 조회
- User Info 정보를 조회한다.
- 컨테이너 플랫폼의 CLI 사용을 위한 환경 및 사용 설정 정보를 조회한다.
  ![IMG_8_1_1]
  ![IMG_8_1_2]

<br>

### <div id='5-8-2'/> 5.8.2. Private Repository
#### <div id='5-8-2-1'/> 5.8.2.1. Private Repository 정보 조회
- Private Repository 정보를 조회한다.
  ![IMG_8_2_1]

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  포털 사용 가이드

----
[image 002]:../../install-guide/images/portal/cp-002.png
[image 003]:../../install-guide/images/portal/cp-003.png
[image 004]:../../install-guide/images/portal/cp-004.png
[image 005]:../../install-guide/images/portal/cp-005.png
[image 006]:../../install-guide/images/portal/cp-006.png
[image 011]:../../install-guide/images/portal/cp-011.png
[image 012]:../../install-guide/images/portal/cp-012.png
[image 013]:../../install-guide/images/portal/cp-013.png
[image 014]:../../install-guide/images/portal/cp-014.png
[image 015]:../../install-guide/images/portal/cp-015.png
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
[IMG_2_1_1]:../images/portal/IMG_2_1_1.png
[IMG_2_1_2_1]:../images/portal/IMG_2_1_2_1.png
[IMG_2_1_2_2]:../images/portal/IMG_2_1_2_2.png
[IMG_2_2_1]:../images/portal/IMG_2_2_1.png
[IMG_2_2_2]:../images/portal/IMG_2_2_2.png
[IMG_2_3_1]:../images/portal/IMG_2_3_1.png
[IMG_2_3_2]:../images/portal/IMG_2_3_2.png
[IMG_2_3_3_1]:../images/portal/IMG_2_3_3_1.png
[IMG_2_3_3_2]:../images/portal/IMG_2_3_3_2.png
[IMG_2_3_3_3]:../images/portal/IMG_2_3_3_3.png
[IMG_2_3_4]:../images/portal/IMG_2_3_4.png
[IMG_2_3_5]:../images/portal/IMG_2_3_5.png
[IMG_3_1_1]:../images/portal/IMG_3_1_1.png
[IMG_3_1_2]:../images/portal/IMG_3_1_2.png
[IMG_3_1_3]:../images/portal/IMG_3_1_3.png
[IMG_3_1_4]:../images/portal/IMG_3_1_4.png
[IMG_3_1_5]:../images/portal/IMG_3_1_5.png
[IMG_3_2_1]:../images/portal/IMG_3_2_1.png
[IMG_3_2_2]:../images/portal/IMG_3_2_2.png
[IMG_3_2_3]:../images/portal/IMG_3_2_3.png
[IMG_3_2_4]:../images/portal/IMG_3_2_4.png
[IMG_3_2_5]:../images/portal/IMG_3_2_5.png
[IMG_3_3_1]:../images/portal/IMG_3_3_1.png
[IMG_3_3_2]:../images/portal/IMG_3_3_2.png
[IMG_3_3_3]:../images/portal/IMG_3_3_3.png
[IMG_3_3_4]:../images/portal/IMG_3_3_4.png
[IMG_3_3_5]:../images/portal/IMG_3_3_5.png
[IMG_4_1_1]:../images/portal/IMG_4_1_1.png
[IMG_4_1_2]:../images/portal/IMG_4_1_2.png
[IMG_4_1_3]:../images/portal/IMG_4_1_3.png
[IMG_4_1_4]:../images/portal/IMG_4_1_4.png
[IMG_4_1_5]:../images/portal/IMG_4_1_5.png
[IMG_4_2_1]:../images/portal/IMG_4_2_1.png
[IMG_4_2_2]:../images/portal/IMG_4_2_2.png
[IMG_4_2_3]:../images/portal/IMG_4_2_3.png
[IMG_4_2_4]:../images/portal/IMG_4_2_4.png
[IMG_4_2_5]:../images/portal/IMG_4_2_5.png
[IMG_5_1_1]:../images/portal/IMG_5_1_1.png
[IMG_5_1_2]:../images/portal/IMG_5_1_2.png
[IMG_5_1_3]:../images/portal/IMG_5_1_3.png
[IMG_5_1_4]:../images/portal/IMG_5_1_4.png
[IMG_5_1_5]:../images/portal/IMG_5_1_5.png
[IMG_5_2_1]:../images/portal/IMG_5_2_1.png
[IMG_5_2_2]:../images/portal/IMG_5_2_2.png
[IMG_5_2_3]:../images/portal/IMG_5_2_3.png
[IMG_5_2_4]:../images/portal/IMG_5_2_4.png
[IMG_5_2_5]:../images/portal/IMG_5_2_5.png
[IMG_5_3_1]:../images/portal/IMG_5_3_1.png
[IMG_5_3_2]:../images/portal/IMG_5_3_2.png
[IMG_5_3_3]:../images/portal/IMG_5_3_3.png
[IMG_5_3_4]:../images/portal/IMG_5_3_4.png
[IMG_5_3_5]:../images/portal/IMG_5_3_5.png
[IMG_6_1_1]:../images/portal/IMG_6_1_1.png
[IMG_6_1_2]:../images/portal/IMG_6_1_2.png
[IMG_6_1_3]:../images/portal/IMG_6_1_3.png
[IMG_6_1_4]:../images/portal/IMG_6_1_4.png
[IMG_6_1_5]:../images/portal/IMG_6_1_5.png
[IMG_7_1_1]:../images/portal/IMG_7_1_1.png
[IMG_7_1_2]:../images/portal/IMG_7_1_2.png
[IMG_7_1_3_1]:../images/portal/IMG_7_1_3_1.png
[IMG_7_1_3_2]:../images/portal/IMG_7_1_3_2.png
[IMG_7_1_4]:../images/portal/IMG_7_1_4.png
[IMG_7_1_5_1]:../images/portal/IMG_7_1_5_1.png
[IMG_7_1_5_2]:../images/portal/IMG_7_1_5_2.png
[IMG_7_1_5_3]:../images/portal/IMG_7_1_5_3.png
[IMG_7_1_5_4]:../images/portal/IMG_7_1_5_4.png
[IMG_7_1_5_5]:../images/portal/IMG_7_1_5_5.png
[IMG_7_2_1]:../images/portal/IMG_7_2_1.png
[IMG_7_2_2]:../images/portal/IMG_7_2_2.png
[IMG_7_2_3]:../images/portal/IMG_7_2_3.png
[IMG_7_2_4]:../images/portal/IMG_7_2_4.png
[IMG_7_2_5]:../images/portal/IMG_7_2_5.png
[IMG_7_3_1]:../images/portal/IMG_7_3_1.png
[IMG_7_3_2]:../images/portal/IMG_7_3_2.png
[IMG_7_3_3]:../images/portal/IMG_7_3_3.png
[IMG_7_3_4]:../images/portal/IMG_7_3_4.png
[IMG_7_3_5]:../images/portal/IMG_7_3_5.png
[IMG_7_4_1]:../images/portal/IMG_7_4_1.png
[IMG_7_4_2]:../images/portal/IMG_7_4_2.png
[IMG_7_4_3]:../images/portal/IMG_7_4_3.png
[IMG_7_4_4]:../images/portal/IMG_7_4_4.png
[IMG_7_4_5]:../images/portal/IMG_7_4_5.png
[IMG_8_1_1]:../images/portal/IMG_8_1_1.png
[IMG_8_1_2]:../images/portal/IMG_8_1_2.png
[IMG_8_2_1]:../images/portal/IMG_8_2_1.png