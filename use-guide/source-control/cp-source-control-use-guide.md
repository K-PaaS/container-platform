### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Use](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/use-guide/Readme.md) > Source Control 서비스 사용 가이드

<br>

# [PaaS-TA Container Platform 소스컨트롤 서비스 사용자 가이드]

<br>

## 목차
1. [문서 개요](#1)
     * [1.1. 목적](#1.1)
     * [1.2. 범위](#1.2)
2. [컨테이너 플랫폼 소스컨트롤 접속](#2)
     * [2.1. 서비스형 컨테이너 플랫폼 소스컨트롤 접속](#2.1)
     * [2.2. 단독 배포형 컨테이너 플랫폼 소스컨트롤 접속](#2.2)
3. [컨테이너 플랫폼 소스컨트롤 사용자 매뉴얼](#3)
     * [3.1. 컨테이너 플랫폼 소스컨트롤 사용자 메뉴 구성](#3.1)
     * [3.2. 컨테이너 플랫폼 소스컨트롤 사용자 메뉴 설명](#3.2)
     * [3.2.1 내정보 관리](#3.2.1)
     * [3.2.1.1 내정보 상세 조회/수정](#3.2.1.1)
     * [3.2.2 사용자 관리](#3.2.2)
     * [3.2.2.1 사용자 목록 조회](#3.2.2.1)
     * [3.2.2.2 사용자 상세 조회/삭제](#3.2.2.2)
     * [3.2.3 레파지토리 관리](#3.2.3)
     * [3.2.3.1 레파지토리 생성](#3.2.3.1)
     * [3.2.3.2 레파지토리 목록 조회](#3.2.3.2)
     * [3.2.3.3 레파지토리 상세 조회](#3.2.3.3)
     * [3.2.3.4 레파지토리 수정/삭제](#3.2.3.4)
     * [3.2.3.5 레파지토리 파일 목록 조회](#3.2.3.5)
     * [3.2.3.6 레파지토리 커밋 목록 조회](#3.2.3.6)
     * [3.2.3.7 레파지토리 참여자 목록 조회](#3.2.3.7)

<br>

# <div id='1'/> 1. 문서 개요

### <div id='1.1'/> 1.1. 목적
본 문서는 컨테이너 플랫폼 소스컨트롤 서비스를 사용할 사용자의 사용 방법에 대해 기술하였다.

<br>

### <div id='1.2'/> 1.2. 범위
본 문서는 Windows 환경을 기준으로 컨테이너 플랫폼 소스컨트롤 서비스를 사용할 사용자의 사용 방법에 대해 작성되었다.

<br>

# <div id='2'/> 2. 컨테이너 플랫폼 소스컨트롤 접속

### <div id='2.1'/> 2.1. 서비스형 컨테이너 플랫폼 소스컨트롤 접속
1. PaaS-TA 포탈의 스페이스 페이지에서 신청된 소스컨트롤의 “대시보드” 버튼을 클릭하여 접속을 진행한다.

<br>

2. 컨테이너 플랫폼 소스컨트롤 접속을 확인한다.
![image](https://user-images.githubusercontent.com/67407365/147020207-16929bd7-4011-4643-8eb9-958a9ccc4611.png)

<br>

3. 이 때 ***컨테이너 플랫폼 배포 소스컨트롤 서비스를 처음 생성한 사용자가 관리자가 된다.***

<br>

### <div id='2.2'/> 2.2. 단독 배포형 컨테이너 플랫폼 소스컨트롤 접속
1. 배포한 클러스터의 공인 IP로 웹 브라우저에서 http://{K8S_MASTER_NODE_IP}:30094 로 접속하여 진행한다.

<br>

2. 컨테이너 플랫폼 소스 컨트롤 접속 초기 정보를 확인한 후 소스 컨트롤에 로그인한다.

> 초기 계정정보는 아래 명령어를 통해 확인
```
$ kubectl get configmap -n cp-portal cp-portal-configmap -o yaml | grep KEYCLOAK_ADMIN
```
![image](https://user-images.githubusercontent.com/67407365/147020597-632f5a59-5054-49a4-b420-36d8385eadf7.png)

<br>

3. 컨테이너 플랫폼 소스컨트롤 접속을 확인한다.
![image](https://user-images.githubusercontent.com/67407365/147020207-16929bd7-4011-4643-8eb9-958a9ccc4611.png)

<br>

# <div id='3'/> 3. 컨테이너 플랫폼 소스컨트롤 사용자 매뉴얼
본 장에서는 컨테이너 플랫폼 파이프라인의 메뉴 구성 및 화면 설명에 대해서 기술하였다.

<br>

### <div id='3.1'/> 3.1. 컨테이너 플랫폼 소스컨트롤 사용자 메뉴 구성
컨테이너 플랫폼 소스컨트롤 서비스는 내정보 관리, 사용자 관리, 레파지토리 목록으로 구성되어 있다.

***※ 기본적으로 사용자는 사용자 관리 메뉴가 보이지 않는다.***
<table>
  <tr>
    <td>메뉴</td>
    <td>설명</td>
  </tr>
  <tr>
    <td>내정보 관리</td>
    <td>컨테이너 플랫폼 소스컨트롤을 사용하는 현재 사용자의 정보 조회 및 관리</td>
  </tr>
  <tr>
    <td>사용자 관리</td>
    <td>컨테이너 플랫폼 소스컨트롤을 사용하는 사용자들의 정보 조회 및 관리</td>
  </tr>
  <tr>
    <td>레파지토리</td>
    <td>레파지토리 등록, 목록 및 상세 조회, 삭제, 파일, 커밋, 참여자 권한 등의 기능을 관리</td>
  </tr>
</table>

<br>

### <div id='3.2'/> 3.2. 컨테이너 플랫폼 소스컨트롤 사용자 메뉴 설명
본 장에서는 컨테이너 플랫폼 소스컨트롤 3개 메뉴에 대한 설명을 기술한다.

<br>

### <div id='3.2.1'/> 3.2.1. 내정보 관리
컨테이너 플랫폼 소스컨트롤 서비스를 사용하는 현재 사용자의 정보 조회와 관리에 대한 설명을 기술한다.

<br>

### <div id='3.2.1.1'/> 3.2.1.1. 내정보 상세 조회/수정
1. 상단의 우측 메뉴에 현재 사용자 계정명 클릭 후 보이는 목록 첫번째의 "내정보 관리”를 클릭한다.
![image](https://user-images.githubusercontent.com/67407365/147021223-a17382b4-a754-425f-8f3b-b8601bab1a7f.png)

<br>

2. 현재 사용자의 정보를 상세 조회한다.
![image](https://user-images.githubusercontent.com/67407365/147020801-800f6321-a40c-4a18-8a1e-28ab0dd49bbb.png)

<br>

3. 변경할 비밀번호를 입력 후 수정버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/67407365/147021304-56f1437f-4a73-4247-bf83-f6095263aba1.png)


***※변경된 비밀번호는 소스 컨트롤 repository를 접근할 때 사용되는 비밀번호로, 
<br>로그인 정보(단독 배포의 경우 Keycloak 비밀번호, 서비스 배포의 경우 PaaS-TA 비밀번호)와 무관하다.*** <br>
***※최초 계정 로그인 시 비밀번호를 반드시 변경해야 repository pull, push 등의 접근이 가능하다.*** <br>
<br>

### <div id='3.2.2'/> 3.2.2. 사용자 관리
사용자 관리 메뉴는 관리자에게만 보이는 메뉴로써 본 장에서는 컨테이너 플랫폼 소스컨트롤 서비스를 사용하는 사용자들의 권한 관리 및 정보 조회에 대한 설명을 기술한다.

<br>

### <div id='3.2.2.1'/> 3.2.2.1. 사용자 목록 조회
1. 상단의 우측 메뉴에 현재 사용자 계정명 클릭 후 보이는 목록 두번째의 "사용자 관리”를 클릭한다.
![image](https://user-images.githubusercontent.com/67407365/147021373-8dbc3bb8-32d2-4679-951d-dc7e92a6258d.png)

<br>

2. 이동된 화면 목록에서 상단 관리자 기준으로 모든 사용자 아이디, 이름, 생성일, 수정일을 확인할 수 있다. 또한 목록 우측에서는 관리자 및 사용자의 사용 여부 확인이 가능하다.
![image](https://user-images.githubusercontent.com/67407365/147021423-54819e22-10d6-405d-abfd-386227331333.png)

<br>

### <div id='3.2.2.2'/> 3.2.2.2. 사용자 상세 조회/삭제
1. 상단의 우측 메뉴에 현재 사용자 계정명 클릭 후 보이는 목록 두번째의 "사용자 관리”를 클릭한다.
![image](https://user-images.githubusercontent.com/67407365/147021373-8dbc3bb8-32d2-4679-951d-dc7e92a6258d.png)

<br>

2. 사용자 정보 클릭 후 사용자 상세 조회/삭제 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/67407365/147021518-0de19286-55ab-41c8-b567-40e5f8ec5d07.png)

<br>

3. 사용자 상세 조회/삭제 페이지에서 아이디, 이름, 사용여부, 설명 정보를 조회한다.
![image](https://user-images.githubusercontent.com/67407365/147021556-0fcd21bb-1439-4875-b5db-ec4fa5c43269.png)

<br>

4. 사용자 상세 조회/삭제 페이지에서 왼쪽 하단의 “사용자 삭제” 버튼을 클릭하여 삭제한다.
![image](https://user-images.githubusercontent.com/67407365/147021579-9ffe96db-3b73-40e7-a8f5-d4dab899fd49.png)

<br>

### <div id='3.2.3'/> 3.2.3. 레파지토리 관리
컨테이너 플랫폼 소스컨트롤 서비스의 레파지토리 정보 조회와 관리에 대한 설명을 기술한다.

<br>

### <div id='3.2.3.1'/> 3.2.3.1. 레파지토리 생성
1. 상단 메뉴의 "레파지토리 목록"을 클릭한다.
![image](https://user-images.githubusercontent.com/67407365/147021750-a73371f4-4298-41e1-ab1a-4d736cfad2a0.png)

<br>

2. 레파지토리 명 입력, 유형 선택 후 등록 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/67407365/147021844-894f4bf7-9d5d-4536-a1f5-64aa9d269c17.png)

<br>

3. 추가적인 방법으로 레파지토리 목록 화면 우측 상단의 "신규생성" 버튼을 클릭하여 레파지토리 신규생성 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/67407365/147021886-c84aea6b-01ad-41be-867e-ef170d9ccb0a.png)

<br>

4. 레파지토리 신규생성 페이지에서 레파지토리 명은 영문, 숫자, 하이픈 조합으로 입력하고 유형(Git, SVN)을 선택 후, “생성” 버튼을 클릭한다. 알림 메시지 “레파지토리가 신규 생성되었습니다.” 나오면 “확인” 버튼을 눌러주면 레파지토리가 생성된다.
![image](https://user-images.githubusercontent.com/67407365/147021952-8231e434-1b5c-49af-8739-83d7c8dcb748.png)
![image](https://user-images.githubusercontent.com/67407365/147021986-3ece3a29-c114-4bbb-8227-cb859702d1f7.png)

<br>

### <div id='3.2.3.2'/> 3.2.3.2. 레파지토리 목록 조회
1. 레파지토리 목록 페이지에서 레파지토리 검색, 콤보 박스 형상관리 유형, 전체 레파지토리 및 업데이트 순 검색을 통해 레파지토리 목록을 확인한다.
![image](https://user-images.githubusercontent.com/67407365/147022269-be8fa1f4-4314-4981-b6d7-0d053f3660e9.png)

<br>

### <div id='3.2.3.3'/> 3.2.3.3. 레파지토리 상세 조회
1. 레파지토리 상세보기 페이지에서 파일(file), 커밋(Commit), 참여자(Contributor) 정보를 확인할 수 있다.
![image](https://user-images.githubusercontent.com/67407365/147022467-52235927-8c92-4b70-b924-dd465190537b.png)

<br>

### <div id='3.2.3.4'/> 3.2.3.4. 레파지토리 수정/삭제
1. 레파지토리 목록에서 레파지토리 명을 클릭 후, 레파지토리 상세보기 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/67407365/147022536-7fc3b6a5-65d0-4aba-a203-1a9f1dfcd171.png)

<br>

2. 레파지토리 상세보기 페이지에서 오른쪽 “정보보기/수정” 버튼을 클릭 후, 정보보기/수정 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/67407365/147022494-fa331bf1-1d11-43d7-86c6-072dbf244f9a.png)
<br>

3. 정보보기/수정 페이지에서 "수정" 버튼을 클릭한다. 알림 메시지 “수정 되었습니다.” 나오면 레파지토리 수정이 완료된다.
![image](https://user-images.githubusercontent.com/67407365/147022601-a22ac31d-79aa-4644-b39f-0d67f10a18cb.png)
![image](https://user-images.githubusercontent.com/67407365/147022639-2db1a1e2-820f-46de-8d82-807fdeb486e3.png)

<br>

4. 정보보기/수정 페이지에서 왼쪽 아래의 “레파지토리 삭제” 버튼을 클릭한다. 알림 메시지 "삭제 되었습니다." 나오면 레파지토리 삭제가 완료된다.
![image](https://user-images.githubusercontent.com/67407365/147022660-97dba604-783d-4713-a8ee-6bf3155477eb.png)
![image](https://user-images.githubusercontent.com/67407365/147022714-39b13490-c0f4-4e12-966d-a65334911a21.png)
![image](https://user-images.githubusercontent.com/67407365/147022730-9214894f-e3b9-48e5-8383-312a41e1645b.png)

<br>

### <div id='3.2.3.5'/> 3.2.3.5. 레파지토리 파일 목록 조회
1. 레파지토리 상세보기 오른쪽의 “레파지토리 클론”을 통해 URL 복사가 가능하다.
![image](https://user-images.githubusercontent.com/67407365/147022954-8f48fc82-b827-4dcc-a9a2-e4f0462eb68a.png)

<br>

2. 복사한 레파지토리 클론 URL을 기준으로 git remote add 및 push를 진행한다.

***※복사한 레포지토리에 대하여 push, pull 등의 접근을 위해서는 계정 정보(소스 컨트롤 아이디/비밀번호) 입력이 필요하다.*** <br>
***※아이디: 접속한 소스 컨트롤 아이디(e.g: admin)*** <br>
***※비밀번호: 3.2.1.1.1에서 변경한 비밀번호*** <br>

<br>

3. 레파지토리 상세보기 페이지의 탭 첫 번째 왼쪽의 “파일(file)”에서 브랜치, 태그에 대한 정보 확인이 가능하다.
![image](https://user-images.githubusercontent.com/67407365/147023724-71f271cf-0b64-44fa-a12d-77c91766b6da.png)

<br>

4. “파일(file)” 목록에서는 기본적으로 폴더/파일명, 최종 변경사항, 파일 크기, 마지막 업데이트를 확인할 수 있다.
![image](https://user-images.githubusercontent.com/67407365/147023747-456091b1-c1c6-4b2c-881b-565942d8a2b3.png)

<br>

### <div id='3.2.3.6'/> 3.2.3.6. 레파지토리 커밋 목록 조회
1. 레파지토리 상세보기 페이지의 탭 중간의 “커밋(Commit)”을 클릭하면 변경된 목록을 조회할 수 있다.
![image](https://user-images.githubusercontent.com/67407365/147023842-f43bf125-b9b0-4a06-a749-b2998eab63ad.png)

<br>

### <div id='3.2.3.7'/> 3.2.3.7. 레파지토리 참여자 목록 조회
1. 레파지토리 상세보기 페이지의 탭 마지막 오른쪽의 "참여자(Contributor)"를 클릭한다. 해당 레파지토리의 참여자에 대한 정보 확인이 가능하다.
![image](https://user-images.githubusercontent.com/67407365/147024402-2af350b6-de7a-42c4-b15a-b1a33be47114.png)

<br>

2. 참여자 추가/삭제 버튼을 클릭하여 소스컨트롤러 사용자 검색 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/67407365/147024585-c073f37c-6762-4ea2-b77b-bcdbfcc3c2f3.png)

<br>

3. 소스컨트롤러 사용자 검색 페이지에서 참여자 권한 추가할 사용자를 입력한다. 사용자 검색 목록에서 사용자를 확인 후, “+” 이미지를 클릭한다.
![image](https://user-images.githubusercontent.com/67407365/147024844-7b78393a-9073-451a-b5a7-8011c1125028.png)
![image](https://user-images.githubusercontent.com/67407365/147024953-de195989-a08c-4eeb-b50c-cad31170b2c8.png)

<br>

4. 소스컨트롤러 사용자 검색 하단에 참여자 초대 정보와 권한, 설명에 대한 확인 및 변경이 가능하다. 참여자 초대에 추가된 사용자의 아이디, 이름, 이메일에 대한 정보 확인 후, 필수 조건인 권한(쓰기 권한/보기 권한)을 설정을 완료하면 “추가” 버튼을 클릭한다. 알림 메시지 “[ 참여자가 추가되었습니다. ]”가 나오면 레파지토리 참여자 권한이 추가된다.
![image](https://user-images.githubusercontent.com/67407365/147025127-5eaeb80e-55b8-4176-b288-c6259e5d9431.png)
![image](https://user-images.githubusercontent.com/67407365/147025160-28a4081e-9cda-4380-8406-9836c3138db8.png)

<br>

5. 삭제할 레파지토리 참여 권한을 클릭한다.
![image](https://user-images.githubusercontent.com/67407365/147026049-c87df481-814b-426c-84de-1a8199288b0c.png)

<br>

6. 참여자 정보 페이지 좌측 하단의 참여자 삭제 버튼을 클릭한다. 알림 메시지 “참여자가 삭제가 완료되었습니다.”가 나오면 레파지토리 참여자 권한이 삭제된다.
![image](https://user-images.githubusercontent.com/67407365/147025867-8cd6c5a3-2584-4763-bbc2-7c2402f7c896.png)
![image](https://user-images.githubusercontent.com/67407365/147025936-cc53bcb4-b2ad-459f-a3dc-d2c60d1107ce.png)
![image](https://user-images.githubusercontent.com/67407365/147025962-b0eb4865-bc84-4230-8b4f-98ef6f3ec5fb.png)

<br>

7. 또는, 레파지토리 참여자 권한 추가와 동일하게 소스 컨트롤러 사용자 검색 안에서 참여자 권한 삭제 할 사용자를 입력한다. 사용자 검색 목록에서 사용자를 확인 후, “-” 이미지를 클릭한다. 알림 메시지 “참여자가 삭제되었습니다.”가 나오면 레파지토리 참여자 권한이 삭제된다.
![image](https://user-images.githubusercontent.com/67407365/147026107-61164c48-b938-490f-917b-8863d44eaee7.png)
![image](https://user-images.githubusercontent.com/67407365/147026141-c65dd6de-8c90-48d7-8d52-530140a82513.png)

<br>

### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Use](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/use-guide/Readme.md) > Source Control 서비스 사용 가이드
