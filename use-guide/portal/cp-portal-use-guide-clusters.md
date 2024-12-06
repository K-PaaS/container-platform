### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Clusters 메뉴

<br>

## Table of Contents

1. [Clusters 메뉴](#1)  
   1.1. [Overview](#1-1)  
   1.1.1. [Overview 정보 조회](#1-1-1)  
   1.1.2. [Overview 클러스터 변경](#1-1-2)  
   1.2. [Nodes](#1-2)  
   1.2.1. [Node 목록 조회](#1-2-1)  
   1.2.2. [Node 상세 조회](#1-2-2)  
   1.3. [Namespaces](#1-3)  
   1.3.1. [Namespace 목록 조회](#1-3-1)  
   1.3.2. [Namespace 상세 조회](#1-3-2)  
   1.3.3. [Namespace 생성](#1-3-3)  
   1.3.4. [Namespace 수정](#1-3-4)  
   1.3.5. [Namespace 삭제](#1-3-5)  

<br>

## <div id='1'/> 1. Clusters 메뉴
### <div id='1-1'/> 1.1. Overview
#### <div id='1-1-1'/> 1.1.1. Overview 정보 조회
- Namespace, Deployment, Pod, User의 개수와 Deployment, Pod, ReplicaSet의 차트를 조회한다.
  ![IMG_2_1_1]

<br>

#### <div id='1-1-2'/> 1.1.2. Overview 클러스터 변경
- Select Box에서 클러스터를 선택하면 해당 클러스터에 대한 정보가 조회된다.
- 클러스터 선택 후 Select Box에서 Namespace를 선택하면 해당 Namespace에 대한 정보가 조회된다.
- Namespace **'ALL'** 의 경우 전체 Namespace에 대한 정보가 조회된다.
  ![IMG_2_1_2_1]
  ![IMG_2_1_2_2]

<br>

### <div id='1-2'/> 1.2. Nodes
#### <div id='1-2-1'/> 1.2.1. Node 목록 조회
- Clusters 메뉴의 Nodes를 클릭하여 Node 목록 페이지로 이동한다.
  ![IMG_2_2_1]

<br>

#### <div id='1-2-2'/> 1.2.2. Node 상세 조회
- Node 목록에서 Node명을 클릭하여 Node 상세 페이지로 이동한다.
  ![IMG_2_2_2]

<br>

### <div id='1-3'/> 1.3. Namespaces
#### <div id='1-3-1'/> 1.3.1. Namespace 목록 조회
- Clusters 메뉴의 Namespaces 클릭하여 Namespace 목록 페이지로 이동한다.
  ![IMG_2_3_1]

<br>

#### <div id='1-3-2'/> 1.3.2. Namespace 상세 조회
- Namespace 목록에서 Namespace명을 클릭하여 Namespace 상세 페이지로 이동한다.
  ![IMG_2_3_2]

<br>

#### <div id='1-3-3'/> 1.3.3. Namespace 생성
- Namespace 목록에서 생성 버튼을 클릭하여 Namespace 생성 페이지로 이동한다.
- Namespace 생성 페이지에서 **Resource Quotas**, **Limit Ranges** 를 지정할 수 있다.
- Namespace 생성 페이지 하단의 저장 버튼을 클릭하여 Namespace를 생성한다.
  ![IMG_2_3_3_1]
  ![IMG_2_3_3_2]
  ![IMG_2_3_3_3]

<br>

#### <div id='1-3-4'/> 1.3.4. Namespace 수정
- Namespace 상세에서 수정 버튼을 클릭하여 Namespace 수정 페이지로 이동한다.
- Namespace 수정 페이지에서 **Resource Quotas**, **Limit Ranges**를 수정할 수 있다.
- Namespace 수정 페이지 하단의 저장 버튼을 클릭하여 Namespace를 수정한다.
  ![IMG_2_3_4]

<br>

#### <div id='1-3-5'/> 1.3.5. Namespace 삭제
- Namespace 상세에서 삭제 버튼을 클릭하여 Namespace를 삭제한다.
  ![IMG_2_3_5]

<br>


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Clusters 메뉴

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