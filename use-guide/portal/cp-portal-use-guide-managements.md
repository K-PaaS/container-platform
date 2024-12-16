### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Managements 메뉴

<br>

## Table of Contents

1. [Managements 메뉴](#1)  
  1.1. [Users](#1-1)  
  1.1.1. [클러스터 관리자 조회](#1-1-1)  
  1.1.2. [클러스터 관리자 상세 조회](#1-1-2)  
  1.1.3. [일반 사용자 목록 조회](#1-1-3)  
  1.1.4. [일반 사용자 상세 조회](#1-1-4)  
  1.1.5. [User 수정](#1-1-5)  
  1.2. [Roles](#1-2)  
  1.2.1. [Role 목록 조회](#1-2-1)  
  1.2.2. [Role 상세 조회](#1-2-2)  
  1.2.3. [Role 생성](#1-2-3)  
  1.2.4. [Role 수정](#1-2-4)  
  1.2.5. [Role 삭제](#1-2-5)  
  1.3. [Resource Quotas](#1-3)  
  1.3.1. [Resource Quota 목록 조회](#1-3-1)  
  1.3.2. [Resource Quota 상세 조회](#1-3-2)  
  1.3.3. [Resource Quota 생성](#1-3-3)  
  1.3.4. [Resource Quota 수정](#1-3-4)  
  1.3.5. [Resource Quota 삭제](#1-3-5)  
  1.4. [Limit Ranges](#1-4)  
  1.4.1. [Limit Range 목록 조회](#1-4-1)  
  1.4.2. [Limit Range 상세 조회](#1-4-2)  
  1.4.3. [Limit Range 생성](#1-4-3)  
  1.4.4. [Limit Range 수정](#1-4-4)  
  1.4.5. [Limit Range 삭제](#1-4-5)  

<br>


## <div id='1'/> 1. Managements 메뉴
### <div id='1-1'/> 1.1. Users
#### <div id='1-1-1'/> 1.1.1. 클러스터 관리자 조회
- Managements 메뉴의 Users를 선택하고 Administrator탭을 클릭하여 클러스터 관리자를 조회한다.
- 클러스터 관리자 권한은 한 명 이상 가능하다.
  ![IMG_7_1_1]

<br>

#### <div id='1-1-2'/> 1.1.2. 클러스터 관리자 상세 조회
- 클러스터 관리자 User ID를 클릭하여 클러스터 관리자 상세 조회 페이지로 이동한다.
  ![IMG_7_1_2]

<br>

#### <div id='1-1-3'/> 1.1.3. 일반 사용자 목록 조회
- Managements 메뉴의 Users를 선택하고 User탭을 클릭하여 사용자 목록을 조회한다.
  + 활성(Active) 탭 : 해당 클러스터 내 Namespace/Role 이 할당된 사용자 목록
  + 비활성(Inactive) 탭 : 해당 클러스터 내 Namespace/Role 이 비할당된 사용자 목록
    ![IMG_7_1_3_1]
    ![IMG_7_1_3_2]

<br>

#### <div id='1-1-4'/> 1.1.4. 일반 사용자 상세 조회
- 일반 사용자 User ID를 클릭하여 일반 사용자 상세 조회 페이지로 이동한다.
  ![IMG_7_1_4]

<br>

#### <div id='1-1-5'/> 1.1.5. User 수정
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

### <div id='1-2'/> 1.2. Roles
#### <div id='1-2-1'/> 1.2.1. Role 목록 조회
- Managements 메뉴의 Roles를 클릭하여 Role 목록 페이지로 이동한다.
  ![IMG_7_2_1]

<br>

#### <div id='1-2-2'/> 1.2.2. Role 상세 조회
- Role 목록에서 Role명을 클릭하여 Role 상세 페이지로 이동한다.
  ![IMG_7_2_2]

<br>

#### <div id='1-2-3'/> 1.2.3. Role 생성
- Role 목록에서 생성 버튼을 클릭할 시 Role 생성 팝업창이 뜬다.
  ![IMG_7_2_3]

<br>

#### <div id='1-2-4'/> 1.2.4. Role 수정
- Role 상세에서 수정 버튼을 클릭할 시 Role 수정 팝업창이 뜬다.
  ![IMG_7_2_4]

<br>

#### <div id='1-2-5'/> 1.2.5. Role 삭제
- Role 상세에서 삭제 버튼을 클릭할 시 Role이 삭제된다.
  ![IMG_7_2_5]

<br>

### <div id='1-3'/> 1.3. Resource Quotas
#### <div id='1-3-1'/> 1.3.1. Resource Quota 목록 조회
- Managements 메뉴의 Resource Quotas를 클릭하여 Resource Quota 목록 페이지로 이동한다.
  ![IMG_7_3_1]

<br>

#### <div id='1-3-2'/> 1.3.2. Resource Quota 상세 조회
- Resource Quota 목록에서 Resource Quota명을 클릭하여 Resource Quota 상세 페이지로 이동한다.
  ![IMG_7_3_2]

<br>

#### <div id='1-3-3'/> 1.3.3. Resource Quota 생성
- Resource Quota 목록에서 생성 버튼을 클릭할 시 Resource Quota 생성 팝업창이 뜬다.
  ![IMG_7_3_3]

<br>

#### <div id='1-3-4'/> 1.3.4. Resource Quota 수정
- Resource Quota 상세에서 수정 버튼을 클릭할 시 Resource Quota 수정 팝업창이 뜬다.
  ![IMG_7_3_4]

<br>

#### <div id='1-3-5'/> 1.3.5. Resource Quota 삭제
- Resource Quota 상세에서 삭제 버튼을 클릭할 시 Resource Quota가 삭제된다.
  ![IMG_7_3_5]

<br>

### <div id='1-4'/> 1.4. Limit Ranges
#### <div id='1-4-1'/> 1.4.1. Limit Range 목록 조회
- Managements 메뉴의 Limit Ranges를 클릭하여 Limit Range 목록 페이지로 이동한다.
  ![IMG_7_4_1]

<br>

#### <div id='1-4-2'/> 1.4.2. Limit Range 상세 조회
- Limit Range 목록에서 Limit Range명을 클릭하여 Limit Range 상세 페이지로 이동한다.
  ![IMG_7_4_2]

<br>

#### <div id='1-4-3'/> 1.4.3. Limit Range 생성
- Limit Range 목록에서 생성 버튼을 클릭할 시 Limit Range 생성 팝업창이 뜬다.
  ![IMG_7_4_3]

<br>

#### <div id='1-4-4'/> 1.4.4. Limit Range 수정
- Limit Range 상세에서 수정 버튼을 클릭할 시 Limit Range 수정 팝업창이 뜬다.
  ![IMG_7_4_4]

<br>

#### <div id='1-4-5'/> 1.4.5. Limit Range 삭제
- Limit Range 상세에서 삭제 버튼을 클릭할 시 Limit Range가 삭제된다.
  ![IMG_7_4_5]

<br>


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Managements 메뉴

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