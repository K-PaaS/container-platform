### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Workloads 메뉴

<br>

## Table of Contents

1. [Workloads 메뉴](#1)  
  1.1. [Deployments](#1-1)   
  1.1.1. [Deployment 목록 조회](#1-1-1)    
  1.1.2. [Deployment 상세 조회](#1-1-2)  
  1.1.3. [Deployment 생성](#1-1-3)  
  1.1.4. [Deployment 수정](#1-1-4)  
  1.1.5. [Deployment 삭제](#1-1-5)   
  1.2. [Pods](#1-2)      
  1.2.1. [Pod 목록 조회](#1-2-1)  
  1.2.2. [Pod 상세 조회](#1-2-2)  
  1.2.3. [Pod 생성](#1-2-3)  
  1.2.4. [Pod 수정](#1-2-4)  
  1.2.5. [Pod 삭제](#1-2-5)  
  1.3. [ReplicaSets](#1-3)  
  1.3.1. [ReplicaSet 목록 조회](#1-3-1)  
  1.3.2. [ReplicaSet 상세 조회](#1-3-2)  
  1.3.3. [ReplicaSet 생성](#1-3-3)  
  1.3.4. [ReplicaSet 수정](#1-3-4)  
  1.3.5. [ReplicaSet 삭제](#1-3-5)  
       
<br>


## <div id='1'/> 1. Workloads 메뉴
K-PaaS 표준 모델로 설치한 Single 클라우드 배포(기본설치)의 경우, Kyverno 기반의 네트워크 격리 정책이 기본으로 적용되어 있다.
이 정책에 따라 네임스페이스 간 Pod 트래픽 통신은 기본적으로 차단된다. 다만 특정 서비스 간의 상호 통신이 필요한 경우, 이를 허용하기 위한 별도의 설정이 필요하다.

#### 네임스페이스 간 통신 허용 방법
현재 네트워크 정책은 네임스페이스 간 통신을 기본적으로 차단하지만, Pod에 특정 라벨이 있을 경우 통신이 가능하다.
-  `cp-role=shared`
   + 해당 라벨이 설정되어 있어야 타 네임스페이스와의 통신 가능
   + 라벨이 없으면, 정책에 따라 통신 차단

#### 포털 UI에서 설정하는 방법
Workloads 메뉴에서 Deployment, ReplicaSet, Pod를 생성할 때, 아래 기능을 통해 라벨을 자동으로 추가할 수 있다.

##### 설정 방법
1. 리소스 목록 화면에서 [생성] 버튼 클릭
2. "네임스페이스 간 트래픽 허용" 체크박스 선택
3. 리소스를 생성하면 Pod에 `cp-role=shared` 라벨이 자동으로 추가

![IMG_3_4_1]

##### 확인 방법
리소스 생성 후, Pod 상세 화면의 Labels 항목에서 라벨이 정상적으로 추가되었는지 확인할 수 있다.

![IMG_3_4_2]

<br>

### <div id='1-1'/> 1.1. Deployments
#### <div id='1-1-1'/> 1.1.1. Deployment 목록 조회
- Workloads의 Deployments를 클릭하여 Deployment 목록 페이지로 이동한다.
  ![IMG_3_1_1]

<br>

#### <div id='1-1-2'/> 1.1.2. Deployment 상세 조회
- Deployment 목록에서 Deployment명을 클릭하여 Deployment 상세 페이지로 이동한다.
  ![IMG_3_1_2]

<br>

#### <div id='1-1-3'/> 1.1.3. Deployment 생성
- Deployment 목록에서 생성 버튼을 클릭할 시 Deployment 생성 팝업창이 뜬다.
  ![IMG_3_1_3]

<br>

#### <div id='1-1-4'/> 1.1.4. Deployment 수정
- Deployment 상세에서 수정 버튼을 클릭할 시 Deployment 수정 팝업창이 뜬다.
  ![IMG_3_1_4]

<br>

#### <div id='1-1-5'/> 1.1.5. Deployment 삭제
- Deployment 상세에서 삭제 버튼을 클릭할 시 Deployment가 삭제된다.
  ![IMG_3_1_5]

<br>


### <div id='1-2'/> 1.2. Pods
#### <div id='1-2-1'/> 1.2.1. Pod 목록 조회
- Workloads의 Pods를 클릭하여 Pod 목록 페이지로 이동한다.
  ![IMG_3_2_1]

<br>

#### <div id='1-2-2'/> 1.2.2. Pod 상세 조회
- Pod 목록에서 Pod명을 클릭하여 Pod 상세 페이지로 이동한다.
  ![IMG_3_2_2]

<br>

#### <div id='1-2-3'/> 1.2.3. Pod 생성
- Pod 목록에서 생성 버튼을 클릭할 시 Pod 생성 팝업창이 뜬다.
  ![IMG_3_2_3]

<br>

#### <div id='1-2-4'/> 1.2.4. Pod 수정
- Pod 상세에서 수정 버튼을 클릭할 시 Pod 수정 팝업창이 뜬다.
  ![IMG_3_2_4]

<br>

#### <div id='1-2-5'/> 1.2.5. Pod 삭제
- Pod 상세에서 삭제 버튼을 클릭할 시 Pod 삭제가 완료된다.
  ![IMG_3_2_5]

<br>

### <div id='1-3'/> 1.3. ReplicaSets
#### <div id='1-3-1'/> 1.3.1. ReplicaSet 목록 조회
- Workloads의 ReplicaSets을 클릭하여 ReplicaSet 목록 페이지로 이동한다.
  ![IMG_3_3_1]

<br>

#### <div id='1-3-2'/> 1.3.2. ReplicaSet 상세 조회
- ReplicaSet 목록에서 ReplicaSet명을 클릭하여 ReplicaSet 상세 페이지로 이동한다.
  ![IMG_3_3_2]

<br>

#### <div id='1-3-3'/> 1.3.3. ReplicaSet 생성
- ReplicaSet 목록에서 생성 버튼을 클릭할 시 ReplicaSet 생성 팝업창이 뜬다.
  ![IMG_3_3_3]

<br>

#### <div id='1-3-4'/> 1.3.4. ReplicaSet 수정
- ReplicaSet 상세에서 수정 버튼을 클릭할 시 ReplicaSet 수정 팝업창이 뜬다.
  ![IMG_3_3_4]

<br>

#### <div id='1-3-5'/> 1.3.5. ReplicaSet 삭제
- ReplicaSet 상세에서 삭제 버튼을 클릭할 시 ReplicaSet 삭제가 완료된다.
  ![IMG_3_3_5]

<br>


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Workloads 메뉴

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
[IMG_3_4_1]:../images/portal/IMG_3_4_1.png
[IMG_3_4_2]:../images/portal/IMG_3_4_2.png
