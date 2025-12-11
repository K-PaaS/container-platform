### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) > [포털 사용 가이드](./cp-portal-use-guide.md) > Global 메뉴

<br>

## Table of Contents

1. [Global 메뉴](#1)  
   1.1. [Overview](#1-1)  
   1.1.1. [Overview 정보 조회](#1-1-1)  
   1.2. [Clusters](#1-2)  
   1.2.1. [Clusters 목록 조회](#1-2-1)  
   1.2.2. [Clusters 상세 조회](#1-2-2)  
   1.2.3. [Clusters 생성](#1-2-3)  
   1.2.4. [Clusters 등록](#1-2-4)  
   1.2.5. [Clusters 수정](#1-2-5)  
   1.3. [Cloud Accounts](#1-3)  
   1.3.1. [Cloud Accounts 목록 조회](#1-3-1)  
   1.3.2. [Cloud Accounts 상세 조회](#1-3-2)  
   1.3.3. [Cloud Accounts 생성](#1-3-3)  
   1.3.4. [Cloud Accounts 수정](#1-3-4)  
   1.3.5. [Cloud Accounts 삭제](#1-3-5)  
   1.4. [Instance Code Template](#1-4)  
   1.4.1. [Instance Code Template 목록 조회](#1-4-1)  
   1.4.2. [Instance Code Template 상세 조회](#1-4-2)  
   1.4.3. [Instance Code Template 생성](#1-4-3)  
   1.4.4. [Instance Code Template 수정](#1-4-4)  
   1.4.5. [Instance Code Template 삭제](#1-4-5)  
   1.5. [SSH Keys](#1-5)  
   1.5.1. [SSH Keys 목록 조회](#1-5-1)  
   1.5.2. [SSH Keys 상세 조회](#1-5-2)  
   1.5.3. [SSH Keys 생성](#1-5-3)  
   1.5.4. [SSH Keys 수정](#1-5-4)  
   1.5.5. [SSH Keys 삭제](#1-5-5)  
   1.6. [Federation](#1-6)  
   1.6.1. [Overview 정보 조회](#1-6-1)
   1.6.2. [Clusters 목록 조회](#1-6-2)
   1.6.3. [Clusters 결합](#1-6-3)
   1.6.4. [Clusters 상세 조회](#1-6-4)
   1.6.5. [Clusters 제외](#1-6-5)
   1.6.6. [Clusters Sync](#1-6-6)
   1.6.7. [Polices 목록 조회](#1-6-7)
   1.6.8. [Polices 생성](#1-6-8)
   1.6.9. [Polices 상세 조회](#1-6-9)
   1.6.10. [Polices 수정](#1-6-10)
   1.6.11. [Polices 삭제](#1-6-11)
   1.6.12. [Namespaces 목록 조회](#1-6-12)
   1.6.13. [Namespaces 생성](#1-6-13)
   1.6.14. [Namespaces 상세 조회](#1-6-14)
   1.6.15. [Namespaces 삭제](#1-6-15)  
   1.6.16. [Workloads 목록 조회](#1-6-16)
   1.6.17. [Workloads 생성](#1-6-17)
   1.6.18. [Workloads 상세 조회](#1-6-18)
   1.6.19. [Workloads 수정](#1-6-19)
   1.6.20. [Workloads 삭제](#1-6-20)
   1.6.21. [ConfigMaps&Secrets 목록 조회](#1-6-21)
   1.6.22. [ConfigMaps&Secrets 생성](#1-6-22)
   1.6.23. [ConfigMaps&Secrets 상세 조회](#1-6-23)
   1.6.24. [ConfigMaps&Secrets 수정](#1-6-24)
   1.6.25. [ConfigMaps&Secrets 삭제](#1-6-25)

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
| :-------: | :-----------: | :-------: | :-------: |
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

### <div id='1-6'/> 1.6. Federation

여러 클러스터를 단일 컨트롤 플레인에서 일관되게 운영·배포할 수 있도록 해주는 서비스이다.
Federation을 통해 멀티클러스터 통합 관리, 애플리케이션 멀티클러스터 배포, 정책 기반 운영 자동화, 멀티클러스터의 상태, 자원 사용량, 워크로드 현황을 통합해서 모니터링할 수 있다.

#### <div id='1-6-1'/> 1.6.1. Overview 정보 조회

- Federation 메뉴의 Overview 탭에서 Federation 리소스 정보, 호스트 클러스터와 멤버 클러스터의 CPU, Memory Requests 및 사용량 , Node 갯수, 상태를 조회 할 수 있다.
- 멤버 클러스터에 Metric이 없을 경우 해당 클러스터의 CPU, Memory 사용량이 나오지 않을 수 있다.
  ![IMG_1_6_1]

<br>

#### <div id='1-6-2'/> 1.6.2. Clusters 목록 조회

- Clusters 탭에서 Federation에 결합된 멤버 Cluster 목록을 볼 수 있다.
- 멤버 클러스터에 Metric이 없을 경우 해당 클러스터의 CPU, Memory 사용량이 나오지 않을 수 있다.
  ![IMG_1_6_2]

<br>

#### <div id='1-6-3'/> 1.6.3. Clusters 결합

- Clusters 탭에서 + Join 버튼을 클릭한다.
- Federation에 멤버 클러스터로 결합할 수 있는 Cluster 목록이 보여지며 복수 선택이 가능하다.
- 멤버 클러스터로 결합 가능한 클러스터 목록은 Global > Clusters에 생성되어 있는 Cluster 목록 중에 Federation 멤버 클러스터로 등록되지 않은 클러스터이다.
  ![IMG_1_6_3]

<br>

#### <div id='1-6-4'/> 1.6.4. Clusters 상세 조회

- 상세 조회 할 클러스터의 View 버튼을 클릭한다.
- yaml 에디터 형태로 멤버 클러스터 정보를 볼 수 있다.
  ![IMG_1_6_4]

<br>

#### <div id='1-6-5'/> 1.6.5. Clusters 제외

- 멤버 클러스터에서 제외할 클러스터의 Exclude 버튼을 클릭한다.
- Exclude 버튼은 클러스터 자체를 삭제하는 것이 아닌 Federation에 영향을 받는 멤버 클러스터에서 제외되는 것이다.
- 클러스터를 K-PaaS에서 완전히 해제하려면 Global > Clusters 메뉴에서 해제하는 것을 추가로 진행하면 된다.
  ![IMG_1_6_5]

<br>

#### <div id='1-6-6'/> 1.6.6. Clusters Sync

- Federation에 결합된 멤버 클러스터에 있는 리소스를 Federation으로 sync 할 수 있다.
- Federation으로 sync 할 멤버 클러스터의 Sync 버튼을 클릭한다.
- 멤버 클러스터에 있는 리소스 중 Federation에 sync 하고 싶은 리소스를 선택한 후 Apply 버튼을 누른다.
- 이미 Federation에 있는 리소스는 sync 할 필요가 없으므로 체크박스가 비활성화 된다.
- 'kube-', 'karamada-' 로 시작하는 네임스페이스의 리소스들은 sync 할 수 없다.
  ![IMG_1_6_6]

<br>

#### <div id='1-6-7'/> 1.6.7. Polices 목록 조회

- Policies 탭에서 Policy 목록을 조회할 수 있다.
- Policy는 Namespace 레벨과 Cluster 레벨이 있다.
- Namespace 레벨에서는 네임스페이스 별로 목록을 조회할 수 있다.
  ![IMG_1_6_7]

<br>

#### <div id='1-6-8'/> 1.6.8. Polices 생성

- Policies 탭에서 +Add 버튼을 눌러 Policy를 생성할 수 있다.
- 총 세개의 단계가 있으며, 첫 번째 단계는 Metadata이다.
- Level, Namespace, Name은 필수 입력이다.
- Preserve Resource On Deletion은 리소스 템플릿이 삭제될 때 멤버 클러스터들에 있는 리소스가 보존되어야 하는지를 제어한다.
  기본값은 false이며, true로 설정할 경우 다음 단계(Resource Selector)에서 선택한 리소스를(Federation에 있는 리소스) 삭제할 경우 멤버 클러스터에 전파된 리소스들이 삭제되지 않고 보존되어진다. 이후 멤버 클러스터에서 다시 리소스 삭제 시도를 할 경우 리소스가 삭제된 후 재생성되지 않는다.
  ![IMG_1_6_8_1]

<br>

- Next 버튼을 눌러 Resource Selectors 단계로 넘어온다.
- Resource Selectors 옆 + 버튼을 눌러 전파할 리소스를 추가한다. 최대 20개까지 추가 가능하다.
  ![IMG_1_6_8_2]

<br>

- 전파할 리소스를 추가할 때 Kind만 선택할 경우 Federation에 존재하는 해당 Kind의 모든 리소스가 전파되니 주의해야한다.
- LabelSelectors는 최대 20개까지 선택 가능하다.
  ![IMG_1_6_8_3]

<br>

- Next 버튼을 눌러 Placement 단계로 넘어온다.
- ClusterNames에서 리소스를 전파할 후보 멤버 클러스터를 필수로 1개이상 선택하여야 한다.
  ![IMG_1_6_8_4]

<br>

- 옵션 영역은 아래와 같다.

- `Replica Scheduling`: 리소스의 복제본에 대한 스케줄링 방법으로 디폴트는 Duplicated이다.
  - `Duplicated`: 리소스를 전파할 때 전 단계에서 추가한 리소스의 복제본 수를 각각 후보 멤버 클러스터에 똑같이 적용한다. 예를 들어, Resource Selectors 단계에서 replicas가 5개인 A deployment를 추가하고, member1, member2 라는 멤버 클러스터를 후보로 지정한 경우 member1 과 member2 클러스터에 replica가 5개인 A deployment가 전파된다.
  - `Divided`: 리소스를 전파할 때 Resource Selectors 단계에서 추가한 리소스의 복제본 수를 아래 Division Preference의 선택 값에 따라 후보 멤버 클러스터에 나눠서 전파된다.
- `Division Preference`: 복제본을 어떻게 나눠서 전파할지 선택할 수 있는 옵션
  - `Aggregated`: 복제본을 가능한 한 적은 수의 클러스터에 분할하여 전파한다.
  - `Weighted`: 복제본을 Weight Preference에 따라 가중치를 정한다.
    ![IMG_1_6_8_5]

<br>

#### <div id='1-6-9'/> 1.6.9. Polices 상세 조회

- Policy의 view 버튼을 누르면 해당 Policy의 정보를 yaml 형식으로 볼 수 있다.
  ![IMG_1_6_9]

<br>

#### <div id='1-6-10'/> 1.6.10. Polices 수정

- Policy의 view 버튼을 누르고 해당 Policy의 정보의 yaml을 수정한 후 하단의 Edit 버튼을 누르면 수정된다.
  ![IMG_1_6_10]

<br>

#### <div id='1-6-11'/> 1.6.11. Polices 삭제

- Policy의 Delete 버튼을 누르면 삭제할 수 있다.
  ![IMG_1_6_11]

<br>

#### <div id='1-6-12'/> 1.6.12. Namespaces 목록 조회

- Federation 메뉴의 Namespaces 탭을 클릭하면 Namespace 목록을 조회할 수 있다.
  ![IMG_1_6_12]

<br>

#### <div id='1-6-13'/> 1.6.13. Namespaces 생성

- +Add 버튼을 눌러 Namespace를 생성할 수 있다.
- Name은 필수로 입력해야한다.
  ![IMG_1_6_13]

<br>

#### <div id='1-6-14'/> 1.6.14. Namespaces 상세 조회

- Namespace의 view 버튼을 누르면 해당 Namespace의 정보를 yaml 형식으로 볼 수 있다.
  ![IMG_1_6_14]

<br>

#### <div id='1-6-15'/> 1.6.15. Namespaces 삭제

- Namespace의 Delete 버튼을 누르면 삭제할 수 있다.
  ![IMG_1_6_15]

<br>

#### <div id='1-6-16'/> 1.6.12. Workloads 목록 조회

- Federation 메뉴의 Workloads 탭을 클릭하면 Workload 목록을 조회할 수 있다.
  ![IMG_1_6_16]

<br>

#### <div id='1-6-17'/> 1.6.17. Workloads 생성

- +Add 버튼을 눌러 Workload를 생성할 수 있다.
  ![IMG_1_6_17]

<br>

#### <div id='1-6-18'/> 1.6.18. Workloads 상세 조회

- Workload의 view 버튼을 누르면 해당 Workload의 정보를 yaml 형식으로 볼 수 있다.
  ![IMG_1_6_18]

<br>

#### <div id='1-6-19'/> 1.6.19. Workloads 수정

- Workload의 view 버튼을 누르고 해당 Workload의 정보의 yaml을 수정한 후 하단의 Edit 버튼을 누르면 수정된다.
  ![IMG_1_6_19]

<br>

#### <div id='1-6-20'/> 1.6.20. Workloads 삭제

- Workload의 Delete 버튼을 누르면 삭제할 수 있다.
  ![IMG_1_6_20]

<br>

#### <div id='1-6-21'/> 1.6.21. ConfigMaps&Secrets 목록 조회

- Federation 메뉴의 ConfigMaps&Secrets 탭을 클릭하면 ConfigMap 과 Secret 목록을 각각 조회할 수 있다.
  ![IMG_1_6_21]

<br>

#### <div id='1-6-22'/> 1.6.22. ConfigMaps&Secrets 생성

- +Add 버튼을 눌러 ConfigMap 또는 Secret을 생성할 수 있다.
  ![IMG_1_6_22]

<br>

#### <div id='1-6-23'/> 1.6.23. ConfigMaps&Secrets 상세 조회

- Workload의 view 버튼을 누르면 해당 Workload의 정보를 yaml 형식으로 볼 수 있다.
  ![IMG_1_6_23]

<br>

#### <div id='1-6-24'/> 1.6.24. ConfigMaps&Secrets 수정

- Workload의 view 버튼을 누르고 해당 Workload의 정보의 yaml을 수정한 후 하단의 Edit 버튼을 누르면 수정된다.
  ![IMG_1_6_24]

<br>

#### <div id='1-6-25'/> 1.6.25. ConfigMaps&Secrets 삭제

- Workload의 Delete 버튼을 누르면 삭제할 수 있다.
  ![IMG_1_6_25]

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) > [포털 사용 가이드](./cp-portal-use-guide.md) > Global 메뉴

[IMG_1_1_1]: ../images/portal/IMG_1_1_1.png
[IMG_1_2_1]: ../images/portal/IMG_1_2_1.png
[IMG_1_2_2]: ../images/portal/IMG_1_2_2.png
[IMG_1_2_3]: ../images/portal/IMG_1_2_3.png
[IMG_1_2_4]: ../images/portal/IMG_1_2_4.png
[IMG_1_2_5]: ../images/portal/IMG_1_2_5.png
[IMG_1_3_1]: ../images/portal/IMG_1_3_1.png
[IMG_1_3_2]: ../images/portal/IMG_1_3_2.png
[IMG_1_3_3_1]: ../images/portal/IMG_1_3_3_1.png
[IMG_1_3_3_2]: ../images/portal/IMG_1_3_3_2.png
[IMG_1_3_4]: ../images/portal/IMG_1_3_4.png
[IMG_1_3_5]: ../images/portal/IMG_1_3_5.png
[IMG_1_4_1]: ../images/portal/IMG_1_4_1.png
[IMG_1_4_2]: ../images/portal/IMG_1_4_2.png
[IMG_1_4_3]: ../images/portal/IMG_1_4_3.png
[IMG_1_4_4]: ../images/portal/IMG_1_4_4.png
[IMG_1_4_5]: ../images/portal/IMG_1_4_5.png
[IMG_1_5_1]: ../images/portal/IMG_1_5_1.png
[IMG_1_5_2]: ../images/portal/IMG_1_5_2.png
[IMG_1_5_3]: ../images/portal/IMG_1_5_3.png
[IMG_1_5_4]: ../images/portal/IMG_1_5_4.png
[IMG_1_5_5]: ../images/portal/IMG_1_5_5.png
[IMG_1_6_1]: ../images/portal/IMG_1_6_1.png
[IMG_1_6_2]: ../images/portal/IMG_1_6_2.png
[IMG_1_6_3]: ../images/portal/IMG_1_6_3.png
[IMG_1_6_4]: ../images/portal/IMG_1_6_4.png
[IMG_1_6_5]: ../images/portal/IMG_1_6_5.png
[IMG_1_6_6]: ../images/portal/IMG_1_6_6.png
[IMG_1_6_7]: ../images/portal/IMG_1_6_7.png
[IMG_1_6_8_1]: ../images/portal/IMG_1_6_8_1.png
[IMG_1_6_8_2]: ../images/portal/IMG_1_6_8_2.png
[IMG_1_6_8_3]: ../images/portal/IMG_1_6_8_3.png
[IMG_1_6_8_4]: ../images/portal/IMG_1_6_8_4.png
[IMG_1_6_8_5]: ../images/portal/IMG_1_6_8_5.png
[IMG_1_6_9]: ../images/portal/IMG_1_6_9.png
[IMG_1_6_10]: ../images/portal/IMG_1_6_10.png
[IMG_1_6_11]: ../images/portal/IMG_1_6_11.png
[IMG_1_6_12]: ../images/portal/IMG_1_6_12.png
[IMG_1_6_13]: ../images/portal/IMG_1_6_13.png
[IMG_1_6_14]: ../images/portal/IMG_1_6_14.png
[IMG_1_6_15]: ../images/portal/IMG_1_6_15.png
[IMG_1_6_16]: ../images/portal/IMG_1_6_16.png
[IMG_1_6_17]: ../images/portal/IMG_1_6_17.png
[IMG_1_6_18]: ../images/portal/IMG_1_6_18.png
[IMG_1_6_19]: ../images/portal/IMG_1_6_19.png
[IMG_1_6_20]: ../images/portal/IMG_1_6_20.png
[IMG_1_6_21]: ../images/portal/IMG_1_6_21.png
[IMG_1_6_22]: ../images/portal/IMG_1_6_22.png
[IMG_1_6_23]: ../images/portal/IMG_1_6_23.png
[IMG_1_6_24]: ../images/portal/IMG_1_6_24.png
[IMG_1_6_25]: ../images/portal/IMG_1_6_25.png
