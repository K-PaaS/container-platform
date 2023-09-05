### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Use](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/use-guide/Readme.md) > Pipeline 서비스 사용 가이드

<br>

# [PaaS-TA Container Platform 파이프라인 사용자 가이드]

## 목차
1. [문서 개요](#1)
	* [1.1. 목적](#1-1)
	* [1.2. 범위](#1-2)
2. [컨테이너 ](#2)
	* [2.1. 서비스형 컨테이너 플랫폼 파이프라인 접속](#2-1)
	* [2.2. 단독 배포형 컨테이너 플랫폼 파이프라인 ](#2-2)
3. [컨테이너 플랫폼 파이프라인 사용자 매뉴얼](#3)
	* [3.1. 컨테이너 플랫폼 파이프라인 사용자 메뉴 구성](#3-1)
	* [3.2. 컨테이너 플랫폼 파이프라인 사용자 메뉴 설명](#3-2)
	* [3.2.1. 사용자 관리](#3-2-1)
	* [3.2.1.1. 사용자 대시보드](#3-2-1-1)
	* [3.2.1.1.1. 사용자 목록 검색 조회](#3-2-1-1-1)
	* [3.2.1.1.2. 사용자 상세 조회/수정](#3-2-1-1-2)
	* [3.2.2. 파이프라인](#3-2-2)
	* [3.2.2.1. 나의 파이프라인](#3-2-2-1)
	* [3.2.2.2. 파이프라인 신규 생성](#3-2-2-2)
	* [3.2.2.2.1. 파이프라인 신규 생성(1)](#3-2-2-2-1)
	* [3.2.2.2.2. 파이프라인 신규 생성(2)](#3-2-2-2-2)
	* [3.2.2.2.3. 파이프라인 신규 생성(3)](#3-2-2-2-3)
	* [3.2.2.3.   파이프라인 대시보드](#3-2-2-3)
	* [3.2.2.3.1. 파이프라인 목록 검색 조회](#3-2-2-3-1)
	* [3.2.2.3.2. 파이프라인 상세 정보 조회](#3-2-2-3-2)
	* [3.2.2.3.3. 파이프라인 수정](#3-2-2-3-3)
	* [3.2.2.3.4. 파이프라인 삭제](#3-2-2-3-4)
	* [3.2.2.4.   파이프라인 상세](#3-2-2-4)
	* [3.2.2.4.1. 참여자 관리](#3-2-2-4-1)
	* [3.2.2.4.1.1. 참여자 추가](#3-2-2-4-1-1)
	* [3.2.2.4.1.2.    참여자 목록 검색 조회](#3-2-2-4-1-2)
	* [3.2.2.4.1.3.    참여자 상세 정보 조회/수정](#3-2-2-4-1-3)
	* [3.2.2.4.1.4.    참여자 삭제](#3-2-2-4-1-4)
	* [3.2.2.4.2. 빌드 Job](#3-2-2-4-2)
	* [3.2.2.4.2.1.    빌드 Job 생성](#3-2-2-4-2-1)
	* [3.2.2.4.2.2.    빌드 Job 구성 조회/수정](#3-2-2-4-2-2)
	* [3.2.2.4.2.3.    빌드 Job 실행](#3-2-2-4-2-3)
	* [3.2.2.4.2.4.    빌드 Job 정지](#3-2-2-4-2-4)
	* [3.2.2.4.2.5.    빌드 Job 로그/히스토리](#3-2-2-4-2-5)
	* [3.2.2.4.2.6.    빌드 Job 추가](#3-2-2-4-2-6)
	* [3.2.2.4.2.7.    빌드 Job 복제](#3-2-2-4-2-7)
	* [3.2.2.4.2.8.    빌드 Job 삭제](#3-2-2-4-2-8)
	* [3.2.2.4.3. 정적분석 Job](#3-2-2-4-3)
	* [3.2.2.4.3.1.    정적분석 Job 생성](#3-2-2-4-3-1)
	* [3.2.2.4.3.2.    정적분석 Job 구성 조회/수정](#3-2-2-4-3-2)
	* [3.2.2.4.3.3.    정적분석 Job 실행](#3-2-2-4-3-3)
	* [3.2.2.4.3.4.    정적분석 Job 정지](#3-2-2-4-3-4)
	* [3.2.2.4.3.5.    정적분석 Job 로그/히스토리](#3-2-2-4-3-5)
	* [3.2.2.4.3.6.    정적분석 Job 추가](#3-2-2-4-3-6)
	* [3.2.2.4.3.7.    정적분석 Job 품질 이슈 결과](#3-2-2-4-3-7)
	* [3.2.2.4.3.8.    정적분석 Job 복제](#3-2-2-4-3-8)
	* [3.2.2.4.3.9.    정적분석 Job 삭제](#3-2-2-4-3-9)
	* [3.2.2.4.4. 테스트 Job](#3-2-2-4-4)
	* [3.2.2.4.4.1.    테스트 Job 생성](#3-2-2-4-4-1)
	* [3.2.2.4.4.2.    테스트 Job 구성 조회/수정](#3-2-2-4-4-2)
	* [3.2.2.4.4.3.    테스트 Job 실행](#3-2-2-4-4-3)
	* [3.2.2.4.4.4.    테스트 Job 정지](#3-2-2-4-4-4)
	* [3.2.2.4.4.5.    테스트 Job 로그/히스토리](#3-2-2-4-4-5)
	* [3.2.2.4.4.6.    테스트 Job 추가](#3-2-2-4-4-6)
	* [3.2.2.4.4.7.    테스트 Job 결과](#3-2-2-4-4-7)
	* [3.2.2.4.4.8.    테스트 Job 복제](#3-2-2-4-4-8)
	* [3.2.2.4.4.9.    테스트 Job 삭제](#3-2-2-4-4-9)
	* [3.2.2.4.5. 배포 Job](#3-2-2-4-5)
	* [3.2.2.4.5.1.    배포 Job 생성](#3-2-2-4-5-1)
	* [3.2.2.4.5.2.    배포 Job 구성 조회/수정](#3-2-2-4-5-2)
	* [3.2.2.4.5.3.    배포 Job 실행](#3-2-2-4-5-3)
	* [3.2.2.4.5.4.    배포 Job 정지](#3-2-2-4-5-4)
	* [3.2.2.4.5.5.    배포 Job 로그/히스토리](#3-2-2-4-5-5)
	* [3.2.2.4.5.6.    배포 Job 현재 작업으로 롤백](#3-2-2-4-5-6)
	* [3.2.2.4.5.7.    배포 Job 추가](#3-2-2-4-5-7)
	* [3.2.2.4.5.8.    배포 Job 복제](#3-2-2-4-5-8)
	* [3.2.2.4.5.9.    배포 Job 삭제](#3-2-2-4-5-9)
	* [3.2.2.4.6. Job 작업 정렬](#3-2-2-4-6)
	* [3.2.2.4.7. 새 작업 그룹 추가](#3-2-2-4-7)
	* [3.2.3.     파이프라인 관리](#3-2-3)
	* [3.2.3.1.   Kubernetes 정보 관리](#3-2-3-1)
	* [3.2.3.1.1. Kubernetes 정보 등록](#3-2-3-1-1)
	* [3.2.3.1.2. Kubernetes 정보 수정](#3-2-3-1-2)
	* [3.2.3.2.   JOB 감사 추적](#3-2-3-2)
	* [3.2.3.2.1. JOB 감사 추적 조회](#3-2-3-2-1)
	* [3.2.3.2.2. JOB 감사 추적 검색 조회](#3-2-3-2-2)
	* [3.2.4.     품질 관리](#3-2-4)
	* [3.2.4.1.   품질 이슈](#3-2-4-1)
	* [3.2.4.2.   코딩 규칙](#3-2-4-2)
	* [3.2.4.3.   품질 프로파일](#3-2-4-3)
	* [3.2.4.3.1. 품질 프로파일 생성](#3-2-4-3-1)
	* [3.2.4.3.2. 품질 프로파일 복제](#3-2-4-3-2)
	* [3.2.4.3.3. 품질 프로파일 수정](#3-2-4-3-3)
	* [3.2.4.3.4. 품질 프로파일 프로젝트 연결](#3-2-4-3-4)
	* [3.2.4.3.5. 품질 프로파일 삭제](#3-2-4-3-5)
	* [3.2.4.4.   품질 게이트](#3-2-4-4)
	* [3.2.4.4.1. 품질 게이트 생성](#3-2-4-4-1)
	* [3.2.4.4.2. 품질 게이트 복제](#3-2-4-4-2)
	* [3.2.4.4.3. 품질 게이트 수정](#3-2-4-4-3)
	* [3.2.4.4.4. 품질 게이트 조건 추가](#3-2-4-4-4)
	* [3.2.4.4.5. 품질 게이트 프로젝트 연결](#3-2-4-4-5)
	* [3.2.4.4.6. 품질 게이트 삭제](#3-2-4-4-6)
	* [3.2.4.5.   스테이징 관리](#3-2-4-5)
	* [3.2.4.5.1. 환경 정보 관리](#3-2-4-5-1)
	* [3.2.4.5.1.1. 환경 정보 등록](#3-2-4-5-1-1)
	* [3.2.4.5.1.2. 환경 정보 목록 검색](#3-2-4-5-1-2)
	* [3.2.4.5.1.3. 환경 정보 삭제](#3-2-4-5-1-3)
	

# <div id='1'/> 1. 문서 개요

## <div id='1-1'/> 1.1 목적
본 문서는 컨테이너 플랫폼 파이프라인 서비스를 사용할 사용자를 위한 사용 방법에 대해 기술하였다.

## <div id='1-2'/> 1.2 범위
본 문서는 Windows 환경을 기준으로 컨테이너 플랫폼 파이프라인 서비스를 사용할 사용자를 위한 사용 방법에 대해 작성되었다.

# <div id='2'/> 2. 컨테이너 플랫폼 파이프라인 접속

## <div id='2-1'/> 2.1 서비스형 컨테이너 플랫폼 파이프라인 접속
1. PaaS-TA 포탈의 스페이스 페이지에서 신청된 배포 파이프라인의 “대시보드” 버튼을 클릭하여 접속을 진행한다.

2. 컨테이너 플랫폼 파이프라인 접속을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146734921-c3bb22aa-40b2-4d1a-91ad-4b6361d9e379.png)

3. 이 때 ***컨테이너 플랫폼 배포 파이프라인 서비스를 처음 생성한 사용자가 관리자가 된다.*** 우측 상단의 사용자 관리 메뉴를 클릭하여 사용자 대시보드로 이동한다.

## <div id='2-2'/> 2.2 단독 배포 컨테이너 플랫폼 파이프라인 접속
1. 배포한 클러스터의 공인 IP로 웹 브라우저에서 http://{K8S_MASTER_NODE_IP}:30084 로 접속하여 진행한다.

2. 컨테이너 플랫폼 파이프라인 접속 초기 정보를 확인한 후 파이프라인에 로그인한다.

> 초기 계정정보는 아래 명령어를 통해 확인
```
$ kubectl get configmap -n cp-portal cp-portal-configmap -o yaml | grep KEYCLOAK_ADMIN
```
![image](https://user-images.githubusercontent.com/80228983/146735412-c7b1e66b-c8a1-4f37-8d79-182a666a318d.png)

3. 컨테이너 플랫폼 파이프라인 접속을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146734921-c3bb22aa-40b2-4d1a-91ad-4b6361d9e379.png)


# <div id='3'/> 3. 컨테이너 플랫폼 파이프라인 사용자 매뉴얼
본 장에서는 컨테이너 플랫폼 파이프라인의 메뉴 구성 및 화면 설명에 대해서 기술하였다.

## <div id='3-1'/> 3.1 컨테이너 플랫폼 파이프라인 사용자 메뉴 구성
컨테이너 플랫폼 파이프라인 서비스는 파이프라인을 조회 및 관리하는 부분과 Kubernetes cluster 접속 정보를 관리하는 부분 및 Job 감사 추적하는 부분, 소스 코드의 품질을 관리하는 부분, 배포된 애플리케이션의 설정을 관리하는 부분, 부여된 권한에 따라 사용자를 관리할 수 있는 사용자 관리 부분으로 구성되어 있다.

***※ 기본적으로 사용자는 사용자 관리, 파이프라인 관리, 품질 관리 메뉴가 보이지 않는다.***
<table>
  <tr>
    <td>메뉴</td>
    <td>분류</td>
		<td>설명</td>
  </tr>
  <tr>
    <td>사용자 관리</td>
    <td>사용자 대시보드</td>
	  <td>컨테이너 플랫폼 파이프라인을 사용하는 사용자들의 정보 조회 및 관리</td>
  </tr>
  <tr>
		<td>파이프라인</td>
		<td>파이프라인 대시보드</td>
		<td>파이프라인 등록, 목록 및 상세 조회, 삭제, 참여자 권한 등의 기능을 관리</td>
  </tr>
	<tr>
		<th rowspan="2">파이프라인 관리</th>
		<td>Kubernetes 정보 관리</td>
		<td>Kubernetes 클러스터 접속 정보(kubeconfig)를 관리</td>
  </tr>
	<tr>
		<td>JOB 감사 추적</td>
		<td>컨테이너 플랫폼 배포 파이프라인에서 수행되는 job의 실행 상태를 추적 관리</td>
  </tr>
	<tr>
		<th rowspan="4">품질 관리</th>
		<td>품질 이슈</td>
		<td>수행된 소스 코드의 오류 해결 여부 및 오류의 수준, 활성화 상태를 관리</td>
  </tr>
	<tr>
		<td>코딩 규칙</td>
		<td>표준화된 소스 코드의 코딩 양식을 관리</td>
	</tr>
	<tr>
		<td>품질 프로파일</td>
		<td>소스 코드의 품질을 분석하기 위한 기준이 되는 코딩 규칙을 관리</td>
	</tr>
	<tr>
		<td>품질 게이트</td>
		<td>소스 코드의 품질 분석 결과 정해진 품질 기준 값을 초과하거나 미달하지 않았는지에 대한 기준을 관리</td>
	</tr>
	<tr>
		<td>스테이징 관리</td>
		<td>환경정보 관리</td>
		<td>배포된 애플리케이션의 환경 설정 정보를 관리</td>
  </tr>
</table>


## <div id='3-2'/> 3.2 컨테이너 플랫폼 파이프라인 사용자 메뉴 설명
본 장에서는 컨테이너 플랫폼 배포 파이프라인의 4개 메뉴에 대한 설명을 기술한다.

### <div id='3-2-1'/> 3.2.1. 사용자 관리
사용자 관리 메뉴는 관리자에게만 보이는 메뉴로써 본 장에서는 컨테이너 플랫폼 파이프라인 서비스를 사용하는 사용자들의 권한 관리 및 정보 조회와 관리에 대한 설명을 기술한다.

#### <div id='3-2-1-1'/> 3.2.1.1. 사용자 대시보드

##### <div id='3-2-1-1-1'/> 3.2.1.1.1. 사용자 목록 검색 조회
1. 상단의 우측 메뉴에 “사용자 관리” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146738210-5a2c3a2a-fa13-40af-8b5d-dd865671a2d4.png)


2. 사용자 대시보드로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146738266-394f5cb2-aa91-408b-a499-9d2bcfdbfccb.png)

3. 사용자 대시보드 목록에서 검색할 수 있는 조건에는 사용자 아이디를 검색어로 하는 검색과 검색 타입이 관리자/사용자인 검색 2개가 있다. 검색어를 입력 후 “ENTER”를 하거나 “돋보기” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146738410-2d7b3fdd-6319-407d-8515-d71bb76caba7.png)

![image](https://user-images.githubusercontent.com/80228983/146738490-72b48ccd-8258-45da-bd6c-064758daa5d1.png)

##### <div id='3-2-1-1-2'/> 3.2.1.1.2. 사용자 상세 조회/수정
1. 사용자 대시보드에서 아이디를 선택하여 사용자 상세 조회/수정 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146738595-22e7e2d3-c40e-46f8-816d-3d7fada97908.png)

2. 사용자의 정보를 상세 조회한다.
![image](https://user-images.githubusercontent.com/80228983/146738690-f1b61c35-06a9-466d-934c-0ed41c9b0f93.png)

3. 정보를 수정하고자 할 때 입력 값을 새롭게 입력한다. 권한은 관리자/사용자로 선택할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146738750-8089a489-5ed7-4c0d-a86b-c42286f5d156.png)

4. 입력 값을 수정 후 “수정” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146738791-baf6604a-b217-4e1d-9339-9df03dc5f6d5.png)

5. 수정된 정보를 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146738856-1c401eb3-2dd2-4257-90b4-5e8ac4bce32a.png)


### <div id='3-2-2'/> 3.2.2. 파이프라인
본 장에서는 파이프라인을 관리하는 2개 방법에 대한 설명을 기술한다.

***※ 기본적으로 사용자는 파이프라인을 보기만 할 수 있고, 생성, 조회, 수정, 삭제할 수 없다. 하지만 파이프라인 참여자 권한이 부여되었을 경우에는 조건에 따라 파이프라인 생성, 조회, 수정, 삭제가 가능해진다.***

#### <div id='3-2-2-1'/> 3.2.2.1. 나의 파이프라인
1.	어떤 페이지에 있든 상관없이 Drop down 메뉴의 나의 파이프라인 버튼을 클릭하면 대시보드로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146739051-75da86a8-7e74-4a2c-abe6-ba54a446f20c.png)

2.	나의 파이프라인에서 목록의 파이프라인 명을 클릭하면 해당 파이프라인의 상세 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146739192-4dedce55-467d-4674-b45a-e6fabe00091c.png)

#### <div id='3-2-2-2'/> 3.2.2.2. 파이프라인 신규 생성
##### <div id='3-2-2-2-1'/> 3.2.2.2.1. 파이프라인 신규 생성(1)
1. 상단 메뉴의 “파이프라인 목록” 을 클릭하면 파이프라인 명 입력만으로 즉시 파이프라인을 신규 생성할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146739350-09474dfa-085a-4147-88e5-5bcb986742a3.png)
2. 등록 버튼을 누른 후 대시보드에 추가된 파이프라인을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146739409-f403fa35-cb11-47d0-b2fa-db2b40870fa8.png)
3. Drop down 메뉴에도 등록된 파이프라인을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146739462-9b291eff-5939-4447-a9e5-aa3a879ed6d1.png)
##### <div id='3-2-2-2-2'/> 3.2.2.2.2. 파이프라인 신규 생성(2)
1.	오른쪽 상단의 “신규 생성” 버튼을 클릭한다.
***※ 사용자는 “신규 생성” 버튼이 활성화되지 않는다.***
![image](https://user-images.githubusercontent.com/80228983/146739665-3d0c2dc6-27ca-427e-9362-7f657af33ad0.png)
2.	파이프라인 신규 생성 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146739702-8e709946-6e2d-42e3-bb8e-b4abf8cf4a00.png)
3.	파이프라인 명은 필수로 입력받기 때문에 파이프라인 명을 반드시 입력한 후 “생성” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146739772-8fd5e45c-51a9-4b64-8f24-b6096fb4a297.png)
4.	생성된 후 파이프라인 대시보드에서 새로 추가된 파이프라인 명을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146739814-8c37e6e5-df08-43ad-9fb6-c4aaa34b26d0.png)

##### <div id='3-2-2-2-3'/> 3.2.2.2.3. 파이프라인 신규 생성(3)
***※ 사용자는 파이프라인에 대한 참여자 권한이 주어지지 않으면 파이프라인 상세 페이지로 이동할 수 없다.***
1. 오른쪽 상단의 “신규 생성” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146739909-be7e283a-3b56-4016-a686-33fdb0e750b3.png)
2. 파이프라인 신규 생성 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146740201-e2ede42c-2562-48b2-9a6b-15b15f70d80f.png)
3. 이후 생성 과정은 3.2.2.2.2. 파이프라인 신규 생성(2) 참고.

#### <div id='3-2-2-3'/> 3.2.2.3. 파이프라인 대시보드
##### <div id='3-2-2-3-1'/> 3.2.2.3.1. 파이프라인 목록 검색 조회
1. 목록에서 검색할 수 있는 조건은 파이프라인 명과 날짜/업데이트/이름순 필터 2개이다.
![image](https://user-images.githubusercontent.com/80228983/146740286-02497cca-7db5-4677-b02a-738cda839d84.png)
2.	검색어를 입력 후 “ENTER”를 하거나 “돋보기” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146740392-c9f289d1-b88c-4c3b-a5e9-9c31fa8cf2d6.png)
3.	목록 조회/검색 목록 조회일 때 모두 검색 타입 필터를 선택하면 즉시 반영된다.
![image](https://user-images.githubusercontent.com/80228983/146740529-7519a68b-afe5-48fe-af99-651003045b05.png)

##### <div id='3-2-2-3-2'/> 3.2.2.3.2. 파이프라인 상세 정보 조회
1.	파이프라인 대시보드에서 파이프라인 명을 눌러 파이프라인 상세 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146740755-66f7f2d0-b599-4f57-a743-38254d25d5a4.png)
2.	파이프라인 상세페이지에서 오른쪽 상단에 “정보보기/수정” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146740783-840e90ce-f18a-49e5-8b52-7c2dc1a4c368.png)
3.	파이프라인 정보보기/수정 페이지로 이동하여 해당 파이프라인의 정보를 상세 조회한다.
![image](https://user-images.githubusercontent.com/80228983/146741363-0d4f24f4-3438-4e8c-bf6d-348654939adc.png)


##### <div id='3-2-2-3-3'/> 3.2.2.3.3. 파이프라인 수정
1.	파이프라인 정보보기/수정 페이지에서 입력 값을 수정 후 “수정” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146741421-f9545a4a-d5c1-408f-b6d4-1a58cea22e69.png)

2.	수정하고 나면 파이프라인 상세 페이지로 이동한다. 다시 “정보보기/수정” 버튼을 클릭하여 변경된 값을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146744408-0e19583e-1ddd-4aa3-bd0b-c8d2345b1011.png)

##### <div id='3-2-2-3-4'/> 3.2.2.3.4. 파이프라인 삭제
1.	파이프라인 정보보기/수정 페이지에서 “파이프라인 삭제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146744457-34c04434-9812-4519-9931-580d11cc2587.png)
2.	삭제 후에 파이프라인 대시보드로 이동하여 파이프라인이 삭제되었는지 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146744503-b40ffdeb-f358-456f-a9d3-46feab5de4ea.png)


#### <div id='3-2-2-4'/> 3.2.2.4. 파이프라인 상세
본 장에서는 파이프라인에 참여하는 참여자 추가, 수정, 삭제 및 권한 부여 등 전반적인 참여자 관리와 하나의 파이프라인에 대한 Job들을 생성 및 실행, 삭제, 로그/히스토리 조회, 참여자 등을 관리하는 방법에 대해 기술한다.
##### <div id='3-2-2-4-1'/> 3.2.2.4.1. 참여자 관리
###### <div id='3-2-2-4-1-1'/> 3.2.2.4.1.1. 참여자 추가
***※	관리자는 파이프라인 생성자이므로 기본적으로 생성 권한을 가진 참여자이다.***
1.	파이프라인 상세페이지에서 참여자 탭을 선택한다.
![image](https://user-images.githubusercontent.com/80228983/146744564-3b3fbc77-dcf7-48fb-b538-a524358ed59f.png)
2.	우측 상단에 “참여자 추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146744613-c464dc22-5d97-437e-ba74-bbe26206cc84.png)
3.	참여자 추가 페이지로 이동하여 현재 배포 파이프라인에 초대된 사용자를 검색하고, 선택한다. 권한은 보기, 생성, 실행 권한 중 한 개를 선택하여 “추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146744679-ea86e8b6-821e-4889-ae05-2ae2cd8a0e42.png)
4.	참여자 탭에서 추가된 참여자를 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146744724-fd9e2e5e-f817-4dfb-a4b3-05ba962f8b81.png)

###### <div id='3-2-2-4-1-2'/> 3.2.2.4.1.2. 참여자 목록 검색 조회
1.	참여자 목록에서 검색할 수 있는 조건은 참여자 아이디 검색 1개이다.
![image](https://user-images.githubusercontent.com/80228983/146744773-9afd316f-4516-4c57-9a53-5bfff55f66ec.png)
2.	검색어를 입력 후 “ENTER”를 하거나 “돋보기” 버튼을 클릭한 후 검색 조회를 한다.
![image](https://user-images.githubusercontent.com/80228983/146744836-61c87dc7-3357-452a-b2e1-de1e29d291ae.png)

###### <div id='3-2-2-4-1-3'/> 3.2.2.4.1.3.	참여자 상세 정보 조회/수정
1.	참여자 목록에서 정보 조회할 참여자 아이디를 선택한다.
![image](https://user-images.githubusercontent.com/80228983/146745046-fe208791-1399-4f99-8e19-25da48c07f72.png)
2.	참여자 상세 정보 조회/수정 페이지로 이동하여 참여자의 상세 정보를 조회한다.
![image](https://user-images.githubusercontent.com/80228983/146745108-2a629019-8b4d-42c9-948d-4e2209c67500.png)
3.	수정하고자 하는 권한을 선택하여 “수정” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146745164-9863d2aa-bea7-4a28-9fa9-b3cd7f08681e.png)
4.	참여자 목록에서 수정된 참여자의 권한을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146745207-ad8372c4-9513-4ccd-a729-a21de9215758.png)


###### <div id='3-2-2-4-1-4'/> 3.2.2.4.1.4.	참여자 삭제
1.	참여자 목록에서 삭제할 참여자 아이디를 선택한다.
![image](https://user-images.githubusercontent.com/80228983/146745268-723a4048-1fc3-46d4-9e8a-9a447ddfbd0c.png)
2.	참여자 상세 정보 조회/수정 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146745316-68d3758b-ae46-4866-9055-e52e324c8886.png)
3.	“참여자 삭제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146745342-0dcb9159-bee3-4a99-b236-7fb8103b4347.png)
4.	참여자 목록에서 참여자가 삭제되었음을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146745391-cc7aa6b6-3b8b-4a92-bef8-997a0f270c1c.png)


##### <div id='3-2-2-4-2'/> 3.2.2.4.2. 빌드 Job
###### <div id='3-2-2-4-2-1'/> 3.2.2.4.2.1. 빌드 Job 생성
1.	파이프라인 상세 페이지에서 ‘이곳을 클릭하여 새 작업 추가’ 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146745481-b70f5459-54e4-4c6f-863a-5093ef546011.png)
2.	구성 상세페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146745559-1148971f-f12f-48e6-8a9a-7a67aedfa6f1.png)
3. 작업 유형을 선택한 후 빌드 유형으로 선택한다. 그 후 Harbor에 저장할 이미지 이름을 입력한다. <br>
![image](https://user-images.githubusercontent.com/80228983/146745847-993cd35d-5c9c-41f6-ba7f-c43a901b0c2a.png)

***※ Gradle을 선택한 경우 소스 빌드 및 테스트, 정적 분석의 경우 Gradle wrapper 설정을 반영한다.***

4. Docker File 생성하기 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146745889-fca0a935-c9fe-4745-b10f-132b7af0a775.png)

5. 스테이징 정보 조회 내 빌더유형에 맞는 이름을 클릭하면, Spring Config Client 설정 방법 예시 정보가 조회된다.
![image](https://user-images.githubusercontent.com/80228983/146876710-95db595c-2d71-41f6-bb1b-d62da91eb0f1.png)
![image](https://user-images.githubusercontent.com/80228983/146877968-5fc84abf-f882-46a2-8f28-77f846052031.png)

6. 형상관리 정보를 입력한 후 브랜치 조회 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146746751-8d825ad5-caa4-485f-91f9-72adc15fb9b8.png)

***※ Github을 선택했을 때, 공개 레파지토리 경로를 입력하는 경우 브랜치 조회 시 아이디와 비밀번호를 입력하지 않아도 된다.***

7. 원하는 브랜치를 선택하고, 저장 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146750384-d397253f-225d-4241-bbf1-72351a7f5486.png)


***※ 빌드 Job 생성은 관리자와 파이프라인 참여자 중 생성 권한을 가진 참여자만 생성이 가능하다.***


###### <div id='3-2-2-4-2-2'/> 3.2.2.4.2.2. 빌드 Job 구성 조회/수정
1. 생성된 빌드 Job 의 “구성” 아이콘을 클릭한다.<br>
![image](https://user-images.githubusercontent.com/80228983/146850081-a3331972-8b8a-4c41-9c8f-fe258d34bf8d.png)
2. 구성 상세페이지로 이동하여 생성 시 저장해 놓았던 구성 정보들을 조회한다.
![image](https://user-images.githubusercontent.com/80228983/146850120-cee00d5f-c539-4bbf-914d-d14bf559c5f2.png)
3. 수정 시에는 각 입력 폼에 수정할 정보들을 다시 입력한 후 “저장” 버튼을 클릭한다.(예제에서는  Dockerfile을 수정)
![image](https://user-images.githubusercontent.com/80228983/146850268-67058b4b-7c88-4911-a686-8d37aabeec61.png)

***※ 빌드 Job 수정 시, 형상관리 정보의 경우 브랜치 이외에는 수정할 수 없다.***<br>
4. 구성 상세페이지로 이동하여 수정된 정보들을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146850355-91325e6d-4b7f-4a07-9a7b-0d1c992981f8.png)

***※ 빌드 Job 구성 조회는 파이프라인 참여자이면 모두 조회가 가능하다. 하지만 수정은 관리자와 파이프라인 참여자 중 생성 권한을 가진 참여자만 수정이 가능하다.***

###### <div id='3-2-2-4-2-3'/> 3.2.2.4.2.3. 빌드 Job 실행
1. 파이프라인 상세페이지에서 “실행” 아이콘을 클릭한다.<br>
![image](https://user-images.githubusercontent.com/80228983/146850397-8c68b631-d8fe-4449-8d62-24ba86cc6cf2.png)
2. 실행이 될 때 파란색으로 바뀌며 깜빡이는 것을 확인할 수 있다. (실행 중에 “로그/히스토리” 아이콘을 클릭하여 실시간으로 로그를 조회할 수 있다.)
![image](https://user-images.githubusercontent.com/80228983/146850501-bf13c417-cf94-465c-8544-bf35beaece0f.png)
3. 실행이 완료되면 초록색으로 바뀌며 작업 부분에 Build(실행완료) 로 표시된다.
![image](https://user-images.githubusercontent.com/80228983/146850568-30b068e6-b168-4fd8-b9a5-ea3d4178cd81.png)

***※	빌드 Job 실행은 관리자와 파이프라인 참여자 중 생성 권한과 실행 권한을 가진 참여자만 가능하다.***


###### <div id='3-2-2-4-2-4'/> 3.2.2.4.2.4. 빌드 Job 정지
1.	실행 중인 빌드 Job을 정지 및 취소하고 싶을 때 “정지” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146850630-62dafc69-f867-4b1f-b334-bdac4fad8387.png)
2.	정지된 빌드 Job은 주황색으로 바뀌는 것을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146850652-8e95625f-e35b-48ef-bd19-8c1d9f3be335.png)

***※	빌드 Job 정지는 관리자와 파이프라인 참여자 중 생성 권한과 실행 권한을 가진 참여자만 가능하다.***


###### <div id='3-2-2-4-2-5'/> 3.2.2.4.2.5.	빌드 Job 로그/히스토리
1.	빌드 Job 실행이 진행 중일 때 “로그/히스토리” 아이콘을 클릭하여 실시간으로 로그를 조회할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146850705-e4479cf3-e470-4b07-883d-656855cd404a.png)
2.	로그 조회 페이지로 이동한다. 실시간으로 로그가 보이고 있는 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146850741-c9f337fc-aedc-443c-a2dd-12373d4ac867.png)
3.	빌드 Job 실행이 완료된 것을 확인하고, 히스토리를 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146850789-b26d9402-472b-4cc5-8cb1-be28a4b6897a.png)
4.	로그/히스토리 페이지에서 상단의 “취소” 버튼 또는 “실행” 버튼을 클릭하여 Job을 취소하고, 실행할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146850835-a94b0f16-a0a2-496c-9425-cca7e66cf84c.png)
![image](https://user-images.githubusercontent.com/80228983/146850845-0fecf61e-7799-4209-82e0-df735b1ad213.png)
5.	로그/히스토리 페이지에서 “구성” 버튼을 클릭하면 빌드 Job 구성 조회 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146850911-c6360a49-215c-438a-b075-46fb4123dcde.png)
6.	로그/히스토리 페이지에서 “목록” 버튼을 클릭하면 파이프라인 상세페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146851003-63628715-e4b0-406e-88a8-7e06aec2a8fe.png)

***※	빌드 Job 로그/히스토리는 관리자와 모든 파이프라인 참여자가 조회 가능하나 실행 및 정지 버튼은 생성 권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-2-6'/> 3.2.2.4.2.6.	빌드 Job 추가
1.	파이프라인 상세페이지에서 빌드 Job의 “추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146851143-3e0902c7-5ade-45f8-bdc6-f1ff35076f62.png)
2.	구성 상세페이지로 이동 후 빌드 Job 생성과 같이 입력 폼에 알맞은 값들을 입력한 후 ”저장” 버튼을 클릭한다.(작업 유형은 빌드 이외에도 테스트, 배포 유형 선택이 가능하다.).
![image](https://user-images.githubusercontent.com/80228983/146851274-16f0fcd6-6282-44e7-a09a-f51d015d35b1.png)
3.	추가된 빌드 Job 을 확인한다.<br>
![image](https://user-images.githubusercontent.com/80228983/146851391-d452fb78-32ac-4561-95ea-545e5194ccbf.png)
4.	작업 트리거에서 이 작업(Job)을 새 작업 그룹으로 구성을 체크하면 새로운 그룹으로 Job이 추가된다. (이후에 새 작업 그룹 추가 항목에서 설명하겠습니다.)

***※	빌드 Job 추가는 관리자와 생성 권한을 가진 파이프라인 참여자만 가능하다.***

###### <div id='3-2-2-4-2-7'/> 3.2.2.4.2.7.	빌드 Job 복제
1.	파이프라인 상세페이지에서 빌드 Job의 “복제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146851489-6c7f2590-cf73-4be1-9cbe-45cc694796c7.png)
2. 빌드 Job 이 복제된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146851531-7e56baf5-2a5b-4abb-a0f1-7defa5fccf0b.png)

***※	빌드 Job 복제는 관리자와 생성 권한을 가진 파이프라인 참여자만 가능하다.***

###### <div id='3-2-2-4-2-8'/> 3.2.2.4.2.8.	빌드 Job 삭제
1.	파이프라인 상세페이지에서 빌드 Job의 “삭제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146851560-96146948-a220-4dff-b59d-e122a89ebdd5.png)
2.	빌드 Job 이 삭제된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146851577-ba273df2-d952-42c0-9dad-4d44598371fc.png)

***※	빌드 Job 삭제는 관리자와 생성 권한을 가진 파이프라인 참여자만 가능하다.***


##### <div id='3-2-2-4-3'/> 3.2.2.4.3. 정적분석 Job
###### <div id='3-2-2-4-3-1'/> 3.2.2.4.3.1. 정적분석 Job 생성
1.	Job의 “추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146851671-71848dd9-7a21-4202-a5d3-d3f6cd37ced0.png)
2.	구성 페이지로 이동하여 작업 유형을 정적분석(Static-Analysis)으로 선택한 후 입력 유형에서 원하는 품질 프로파일과 품질 게이트, 작업 그룹을 선택한다. 그 후에 작업 트리거는 각자의 상황에 맞게 선택한다. (처음에는 품질 게이트가 존재하지 않는다. [품질 게이트 생성](#3-2-4-4-1) 을 참조하여 생성한다.)
![image](https://user-images.githubusercontent.com/80228983/146852468-9f9620b6-280a-4946-9c69-eaa75e647f4d.png) 
3.	JACOCO plugin script tab의 GRADLE, MAVEN을 클릭하면 jacoco plugin 적용방법을 확인할 수 있다.(예시이므로 상황에 맞게 플러그인을 적용한다.)
![image](https://user-images.githubusercontent.com/80228983/146852604-7539eefd-f3d0-435f-8248-f8ae534d071e.png)
4.	“저장” 버튼을 클릭하고, 파이프라인 상세페이지에서 테스트 Job 생성된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146852642-91708e7f-6fa9-4ec9-8f18-ba97bbfe6e6e.png)

***※	정적분석 Job 생성은 관리자와 파이프라인 참여자 중 생성 권한을 가진 참여자만 생성이 가능하다.***

###### <div id='3-2-2-4-3-2'/> 3.2.2.4.3.2. 정적분석 Job 구성 조회/수정
1.	생성된 정적분석 Job 의 “구성” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146852683-cae485a0-a992-43f8-9355-94aac7736b11.png)
2.	구성 상세페이지로 이동하여 생성 시 저장해 놓았던 구성 정보들을 조회한다.
![image](https://user-images.githubusercontent.com/80228983/146852728-f1824c6f-db5c-4265-9979-65bfaf632e05.png)
3.	수정 시에는 각 입력 폼에 수정할 정보들을 다시 입력한 후 “저장” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146853294-af315b80-4ce4-40ae-8d6a-da33f7e8ba45.png)
4.	구성 상세페이지로 이동하여 수정된 정보들을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146853332-42795e48-c5dc-40f6-bb95-b7bf15db53cb.png)
![image](https://user-images.githubusercontent.com/80228983/146853360-16b00830-b127-4cb7-87a2-aab4cde1602e.png)

***※	정적분석 Job 구성 조회는 파이프라인 참여자이면 모두 조회가 가능하다. 하지만 수정은 관리자와 파이프라인 참여자 중 생성 권한을 가진 참여자만 수정이 가능하다.***

###### <div id='3-2-2-4-3-3'/> 3.2.2.4.3.3.	정적분석 Job 실행
1.	파이프라인 상세페이지에서 정적분석 Job의 “실행” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146853405-4be9ac3d-a04a-4e77-994f-57826271ee1a.png)
2.	실행이 될 때 파란색으로 바뀌며 깜빡이는 것을 확인할 수 있다. (실행 중에 “로그/히스토리” 아이콘을 클릭하여 실시간으로 로그를 조회할 수 있다.)
![image](https://user-images.githubusercontent.com/80228983/146853428-0ec56965-6b93-4e7a-b2ed-7eb0391c1e5d.png)
3. 실행이 완료되면 초록색으로 바뀌며 작업 부분에 Test(실행완료) 로 표시된다.

***※	 Job 실행은 관리자와 파이프라인 참여자 중 생성 권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-3-4'/> 3.2.2.4.3.4.	정적분석 Job 정지
1.	실행 중인 정적분석 Job을 정지 및 취소하고 싶을 때 “정지” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146853481-0f5823a7-d29e-4e9c-acd8-4e0a89e0e2d7.png)
2.	정지된 정적분석 Job은 주황색으로 바뀌는 것을 확인할 수 있다
![image](https://user-images.githubusercontent.com/80228983/146853489-13d585dc-da3c-4eb4-b6a8-af4d3c72ccd4.png)

***※	 Job 정지는 관리자와 파이프라인 참여자 중 생성 권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-3-5'/> 3.2.2.4.3.5.	정적분석 Job 로그/히스토리
1.	정적분석 Job 실행이 진행 중일 때 “로그/히스토리” 아이콘을 클릭하여 실시간으로 로그를 조회할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146853578-293557f5-9e3a-4b57-b714-aee1d4012e8d.png)
2.	로그 조회 페이지로 이동한다. 실시간으로 로그가 보이고 있는 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146853606-1ee8a3ef-3ad7-47c8-9a94-b5f14baef299.png)
3.	정적분석 Job 실행이 완료된 것을 확인하고, 히스토리를 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146853695-5b4d7fc9-ca68-4b70-a5bd-b40f3fe7f709.png)
4.	“실행”, “취소”, “구성”, “목록” 버튼은 3.2.2.4.2.5. 빌드 Job 로그/히스토리 항목을 참고한다.

***※	 Job 로그/히스토리는 관리자와 모든 파이프라인 참여자가 조회 가능하나 실행 및 정지 버튼은 생성 권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-3-6'/> 3.2.2.4.3.6.	정적분석 Job 추가
1.	파이프라인 상세페이지에서 정적분석 Job의 “추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146854048-a807b104-c7fd-41e1-916d-3ae4bedf67ac.png)
2.	그 이후의 과정은 3.2.2.4.2.7. 빌드 Job 추가 항목을 참고한다.

***※	정적분석 Job 추가는 관리자와 생성 권한을 가진 파이프라인 참여자만 가능하다.***

###### <div id='3-2-2-4-3-7'/> 3.2.2.4.3.7.	정적분석 Job 품질 이슈 결과
1.	테스트 Job의 로그/히스토리 “품질 이슈 결과” 버튼을 누르면 수행된 소스 코드의 오류 해결 여부 및 오류의 수준, 활성화 상태를 관리하는 품질 관리 대시보드로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146853748-363d2706-87ba-4724-ac6f-015075c05740.png)
2.	품질 관리 대시보드에 대해서는 이후 3.2.4 품질 관리 부분을 참고한다.

***※	 Job 품질 이슈 결과는 관리자와 모든 파이프라인 참여자가 조회 가능하다.***


###### <div id='3-2-2-4-3-8'/> 3.2.2.4.3.8.	정적분석 Job 복제
1.	파이프라인 상세페이지에서 테스트 Job의 “복제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146854081-eb5c227c-9f06-46ab-9c09-18e11be46647.png)
2.	정적분석 Job 이 복제된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146854105-c13124bf-c7a3-43cb-a120-8be167d6715e.png)

***※	정적분석 Job 복제는 관리자와 생성 권한을 가진 파이프라인 참여자만 가능하다.***

###### <div id='3-2-2-4-3-9'/> 3.2.2.4.3.9.	정적분석 Job 삭제
1.	파이프라인 상세페이지에서 정적분석 Job의 “삭제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146854183-3f69d84d-9553-4870-a5dd-7cbb9cf2890f.png)
2.	정적분석 Job 이 삭제된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146854202-4204c295-514a-4a4e-bc97-d5e06db1665b.png)

***※	 Job 삭제는 관리자와 생성 권한을 가진 파이프라인 참여자만 가능하다.***

##### <div id='3-2-2-4-4'/> 3.2.2.4.4. 테스트 Job
###### <div id='3-2-2-4-4-1'/> 3.2.2.4.4.1. 테스트 Job 생성
1.	Job의 “추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146854258-a09a0a78-765d-4d40-acf8-856da17bdf33.png)
2.	구성 페이지로 이동하여 작업 유형을 테스트(JUnit-Test)로 선택한 후 테스트할 입력유형을 선택한다. 그 후에 작업 트리거는 각자의 상황에 맞게 선택한다. 
![image](https://user-images.githubusercontent.com/80228983/146854434-072106ab-6050-493f-917c-c93dd30bcf46.png)
3.	“저장” 버튼을 클릭하고, 파이프라인 상세페이지에서 테스트 Job 생성된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146854467-e84b2ab1-2f5d-4e5a-ad1b-735368c8fe3f.png)

***※	테스트 Job 생성은 관리자와 파이프라인 참여자 중 생성 권한을 가진 참여자만 생성이 가능하다.***

###### <div id='3-2-2-4-4-2'/> 3.2.2.4.4.2. 테스트 Job 구성 조회/수정
1.	생성된 테스트 Job 의 “구성” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146854497-54981906-68a4-43e3-aa0f-b7fe0d761084.png)
2.	구성 상세페이지로 이동하여 생성 시 저장해 놓았던 구성 정보들을 조회한다.
3.	수정 시에는 각 입력 폼에 수정할 정보들을 다시 입력한 후 “저장” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146854560-f4a6c6b3-53be-4fcb-8251-890bd517c47a.png)
4.	구성 상세페이지로 이동하여 수정된 정보들을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146854581-3054e420-18a9-4caa-8f26-1fdce8ba34ca.png)

***※	테스트 Job 구성 조회는 파이프라인 참여자이면 모두 조회가 가능하다. 하지만 수정은 관리자와 파이프라인 참여자 중 생성 권한을 가진 참여자만 수정이 가능하다.***

###### <div id='3-2-2-4-4-3'/> 3.2.2.4.4.3.	테스트 Job 실행
1.	파이프라인 상세페이지에서 테스트 Job의 “실행” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146857738-21d46e4c-4679-4373-8be8-31b4aa583439.png)
2.	실행이 될 때 파란색으로 바뀌며 깜빡이는 것을 확인할 수 있다. (실행 중에 “로그/히스토리” 아이콘을 클릭하여 실시간으로 로그를 조회할 수 있다.)
![image](https://user-images.githubusercontent.com/80228983/146857766-88d020e8-f41c-4f5e-aac2-e8564c2ed237.png)
3. 실행이 완료되면 초록색으로 바뀌며 작업 부분에 Test(실행완료) 로 표시된다.

***※	테스트 Job 실행은 관리자와 파이프라인 참여자 중 생성 권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-4-4'/> 3.2.2.4.4.4.	테스트 Job 정지
1.	실행 중인 테스트 Job을 정지 및 취소하고 싶을 때 “정지” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146857820-1b9d8493-aa68-4d5b-9659-590e642c392e.png)
2.	정지된 빌드 Job은 주황색으로 바뀌는 것을 확인할 수 있다
![image](https://user-images.githubusercontent.com/80228983/146857870-1e7a427d-50ad-41ce-9a17-d0c456048006.png)

***※	테스트 Job 정지는 관리자와 파이프라인 참여자 중 생성 권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-4-5'/> 3.2.2.4.4.5.	테스트 Job 로그/히스토리
1.	빌드 Job 실행이 진행 중일 때 “로그/히스토리” 아이콘을 클릭하여 실시간으로 로그를 조회할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146857902-4bba7e92-2da2-48b0-86eb-5fd4314262b4.png)
2.	로그 조회 페이지로 이동한다. 실시간으로 로그가 보이고 있는 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146857935-14f830cb-f40b-4692-8b5e-fe510d24453f.png)
3.	테스트 Job 실행이 완료된 것을 확인하고, 히스토리를 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146857996-3ceb188e-e440-4fb6-94c6-7a271a7fec45.png)
4.	“실행”, “취소”, “구성”, “목록” 버튼은 3.2.2.4.2.5. 빌드 Job 로그/히스토리 항목을 참고한다.

***※	테스트 Job 로그/히스토리는 관리자와 모든 파이프라인 참여자가 조회 가능하나 실행 및 정지 버튼은 생성 권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-4-6'/> 3.2.2.4.4.6.	테스트 Job 추가
1.	파이프라인 상세페이지에서 테스트 Job의 “추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146858210-30863f47-0e41-43ba-bd3b-4df06e93aa81.png)
2.	그 이후의 과정은 3.2.2.4.2.6. 빌드 Job 추가 항목을 참고한다.

***※	테스트 Job 추가는 관리자와 생성 권한을 가진 파이프라인 참여자만 가능하다.***

###### <div id='3-2-2-4-4-7'/> 3.2.2.4.4.7.	테스트 Job 결과
1.	테스트 Job의 로그/히스토리 “Test 결과” 버튼을 누르면 수행된 소스 코드의 JUnit Test 결과 팝업 창이 뜬다.
![image](https://user-images.githubusercontent.com/80228983/146858037-941cd966-1375-48ca-81a6-8d3d02e4244d.png)
![image](https://user-images.githubusercontent.com/80228983/146858111-9b25dab5-85e0-4e43-af33-3aee82095eb4.png)

***※	테스트 Job 품질 이슈 결과는 관리자와 모든 파이프라인 참여자가 조회 가능하다.***

###### <div id='3-2-2-4-4-8'/> 3.2.2.4.4.8.	테스트 Job 복제
1.	파이프라인 상세페이지에서 테스트 Job의 “복제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146858286-38b6cff2-79be-4908-a68d-9c107e18b32e.png)
2.	그 이후의 과정은 3.2.2.4.2.7 빌드 Job 복제 항목을 참고한다.


***※	테스트 Job 복제는 관리자와 생성 권한을 가진 파이프라인 참여자만 가능하다.***

###### <div id='3-2-2-4-4-9'/> 3.2.2.4.4.9.	테스트 Job 삭제
1.	파이프라인 상세페이지에서 테스트 Job의 “삭제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146858302-223245b6-dd0a-4012-a31b-46761c075b70.png)
2.	그 이후의 과정은 3.2.2.4.2.8 빌드 Job 삭제 항목을 참고한다.


***※	테스트 Job 삭제는 관리자와 생성 권한을 가진 파이프라인 참여자만 가능하다.***

##### <div id='3-2-2-4-5'/> 3.2.2.4.5.	배포 Job
###### <div id='3-2-2-4-5-1'/> 3.2.2.4.5.1.	배포 Job 생성
1. Job의 “추가” 버튼을 클릭한다.<br>
![image](https://user-images.githubusercontent.com/80228983/146858411-96203d4e-86db-45cb-b4eb-69d8839beaa6.png)
2.	구성 페이지로 이동하여 작업 유형을 배포(Deploy)로 선택한 후 유형에서 원하는 배포 유형 및 앱 유형을 선택, 파이프라인 관리에서 저장해 놓은 Kubernetes 정보를 선택한다. (Kubernetes 정보를 가져오기 위해서는 선행 과정이 필요하다. 과정은 3.2.3.1. Kubernetes 정보 관리 항목을 참고한다).
***※	선택한 유형(type), 앱 유형에 따라 DEPLOYMENT.YML, DEPLOYMENT_SERVICE.YML 서식이 달라진다.***
![image](https://user-images.githubusercontent.com/80228983/146858644-8c03afea-aa2f-4eca-8833-dbfd624990bc.png)
3.	DEPLOYMENT.YML, DEPLOYMENT_SERVICE.YML을 앱 설정에 맞게 변경한다.
![image](https://user-images.githubusercontent.com/80228983/146859081-15639ae3-80c8-4ad7-b642-2477d1262c03.png)
***※	YML 파일 내부에 들어 있는 %INTERNAL_SERVICE_PORT% 등의 %{VALUE}% 값은 반드시 변경해야 한다.***
4.	“저장” 버튼을 클릭하고, 파이프라인 상세페이지에서 배포 Job이 생성된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146859103-c8cd5ec4-a72b-4250-96e4-3ffb214baf55.png)

***※	배포 Job 생성은 관리자와 파이프라인 참여자 중 생성 권한을 가진 참여자만 생성이 가능하다.***

###### <div id='3-2-2-4-5-2'/> 3.2.2.4.5.2.	배포 Job 구성 조회/수정
1.	생성된 배포 Job 의 “구성” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146859159-6a32b287-ccdb-4278-b070-6db2d4900346.png)
2.	구성 상세페이지로 이동하여 생성 시 저장해 놓았던 구성 정보들을 조회한다.
![image](https://user-images.githubusercontent.com/80228983/146859182-f382647e-b7d2-4d6c-8c39-5fbcbfc1536a.png)
3.	수정 시에는 각 입력 YML에 수정할 정보들을 다시 입력한 후 “저장” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146859270-94d0e125-f790-485b-9515-710673d5bf66.png)
4.	구성 상세페이지로 이동하여 수정된 정보들을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146859308-6871dc46-49b1-423e-b3c5-a3f5956b6e15.png)


***※	 배포 Job 구성 조회는 파이프라인 참여자이면 모두 조회가 가능하다. 하지만 수정은 관리자와 파이프라인 참여자 중 생성 권한을 가진 참여자만 수정이 가능하다.***

###### <div id='3-2-2-4-5-3'/> 3.2.2.4.5.3.	배포 Job 실행
1.	파이프라인 상세페이지에서 배포 Job의 “실행” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146859343-b0ec964d-ba01-40e7-a73d-ce2703e3b2a1.png)
2.	실행이 될 때 파란색으로 바뀌며 깜빡이는 것을 확인할 수 있다. (실행 중에 “로그/히스토리” 아이콘을 클릭하여 실시간으로 로그를 조회할 수 있다.)
![image](https://user-images.githubusercontent.com/80228983/146859370-cacabc1a-1bfd-4fb0-bbfe-dfc355c1057c.png)
3. 실행이 완료되면 초록색으로 바뀌며 작업 부분에 Deploy(실행완료) 로 표시된다.


***※	배포 Job 실행은 관리자와 파이프라인 참여자 중 생성 권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-5-4'/> 3.2.2.4.5.4.	배포 Job 정지
1.	실행 중인 배포 Job을 정지 및 취소하고 싶을 때 “정지” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146859436-8034f833-8cc9-4b5d-956d-6a96bab44df1.png)
2.	정지된 배포 Job이 주황색으로 바뀌는 것을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146859444-3933f6c3-b707-43fa-80c2-84c8d4ad1213.png)

***※	배포 Job 정지는 관리자와 파이프라인 참여자 중 생성권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-5-5'/> 3.2.2.4.5.5.	배포 Job 로그/히스토리
1.	배포 Job 실행이 진행 중일 때 “로그/히스토리” 아이콘을 클릭하여 실시간으로 로그를 조회할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146859484-9ef181ed-5612-4206-87ca-6b6df4f2343a.png)
2.	로그 조회 페이지로 이동한다. 실시간으로 로그가 보이고 있는 것을 확인한다.
3.	배포 Job 실행이 완료된 것을 확인하고, 히스토리를 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146859513-36f03f0d-aeaa-428b-8ff7-5414488f7959.png)
4.	PaaS-TA 컨테이너 플랫폼 포탈로 조회한 결과 디플로이먼트 부분에 ‘spring-music-test’라는 애플리케이션이 배포되었음을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146859752-60cc0195-164d-4681-96a7-ee57c948bac8.png)

***※	배포 Job 로그/히스토리는 관리자와 모든 파이프라인 참여자가 조회 가능하나 실행 및 정지 버튼은 생성권한과 실행 권한을 가진 참여자만 가능하다.***

###### <div id='3-2-2-4-5-6'/> 3.2.2.4.5.6.	배포 Job 현재 작업으로 롤백
1.	배포 Job 의 로그/히스토리 페이지에서 “현재 작업으로 롤백” 버튼을 현재 지원하지 않는다.
![image](https://user-images.githubusercontent.com/80228983/146859913-eaf78822-3a4e-4bf0-9f3b-034e293ac273.png)

###### <div id='3-2-2-4-5-7'/> 3.2.2.4.5.7.	배포 Job 추가
1.	파이프라인 상세페이지에서 테스트 Job의 “추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146859973-75e5dc02-d29a-4b2e-8c72-1526c20025ff.png)
2.	그 이후의 과정은 3.2.2.4.2.6. 빌드 Job 추가 항목을 참고한다.

***※	배포 Job 추가는 관리자와 생성권한을 가진 파이프라인 참여자만 가능하다.***

###### <div id='3-2-2-4-5-8'/> 3.2.2.4.5.8.	배포 Job 복제
1.	파이프라인 상세페이지에서 배포 Job의 “복제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146860020-4ead6b2e-4313-4946-989e-2be6c75df8d2.png)
2.	그 이후의 과정은 3.2.2.4.2.7. 빌드 Job 복제 항목을 참고한다.

***※	배포 Job 복제는 관리자와 생성권한을 가진 파이프라인 참여자만 가능하다.***

###### <div id='3-2-2-4-5-9'/> 3.2.2.4.5.9.	배포 Job 삭제
1.	파이프라인 상세페이지에서 배포 Job의 “삭제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146860050-1d04e411-18b2-4412-a7a9-62970bdef9eb.png)
2.	그 이후의 과정은 3.2.2.4.2.8. 빌드 Job 삭제 항목을 참고한다.


***※	배포 Job 삭제는 관리자와 생성권한을 가진 파이프라인 참여자만 가능하다.***

##### <div id='3-2-2-4-6'/> 3.2.2.4.6. Job 작업 정렬
1.	파이프라인 상세페이지에서 각 Job의 “작업 정렬” 아이콘을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146860164-f447fbaa-1f6b-491d-a8b8-ad7a109125d2.png)
2.	현재 작업 그룹 내에서 정렬할 수 있는 나머지 Job의 번호의 목록이 drop down 메뉴로 보인다.
![image](https://user-images.githubusercontent.com/80228983/146860179-bfda5a8b-ca6d-48a6-b77d-dee8d022c543.png)
3.	그중 정렬하고자 하는 번호를 클릭 시 그 번호의 Job 과 위치가 서로 바뀌게 되며 정렬된다.
![image](https://user-images.githubusercontent.com/80228983/146867130-e39790ed-765c-4ec4-9668-04e356271c74.png)

***※	Job 작업 정렬은 관리자와 생성권한을 가진 파이프라인 참여자만 가능하다.***

##### <div id='3-2-2-4-7'/> 3.2.2.4.7. 새 작업 그룹 추가
1.	파이프라인 상세페이지에서 “새 작업 그룹 추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146867152-0477d45e-b5b5-4b64-8735-ee5ac4596880.png)
2.	Job 구성 상세페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146867201-e8622591-6e47-4f57-b3e9-162f47ee6529.png)
3.	새로운 Job 한 개를 생성한 뒤 파이프라인 상세페이지로 이동한다. 이전 작업 그룹 아래로 점선이 생기며 새로 생성한 Job이 새로운 그룹 내로 나눠진 것을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146867284-fdac0c1e-962b-49c4-a8ea-759b99b9cc32.png)

***※	Job 새 작업  그룹 추가는 관리자와 생성권한을 가진 파이프라인 참여자만 가능하다.***

### <div id='3-2-3'/> 3.2.3. 파이프라인 관리
#### <div id='3-2-3-1'/> 3.2.3.1. Kubernetes 정보 관리
###### <div id='3-2-3-1-1'/> 3.2.3.1.1. Kubernetes 정보 등록
1. 파이프라인 관리 > Kubernetes 정보 관리 메뉴를 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146867460-8d62380a-8c55-4b41-826c-0237c49aee56.png)
2. 대시보드에서 우측 상단에 “Kuber Config 등록” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146867477-b8c5ff71-29c9-4d91-9e1c-8f55b42222db.png)
3. kubernetes config 등록 페이지로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146867522-fec558d7-8caf-4639-aad0-dcabe9ad565e.png)
4. config 명과 kube config를 입력한 후 등록 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146867697-4ba6e049-35b1-4192-81a5-13e5d8511b12.png)
5. kubernetes config가 등록되었음을 확인한다.<br>
![image](https://user-images.githubusercontent.com/80228983/146867770-d5515ac3-8701-4c7d-a1f9-a8d37dbd24f9.png)

###### <div id='3-2-3-1-2'/> 3.2.3.1.2. Kubernetes 정보 수정
1.	kubernetes 정보 관리 대시보드에서 수정하고자 하는 정보를 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146868073-7f58c5bf-2cd2-4d77-9de6-ebcd77bfcb34.png)
2.	수정할 값들을 입력하고 “수정” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146868101-2e66c9c0-6aef-4425-9c68-79e88be3ab1f.png)
3.	다시 대시보드를 통해 정보 상세 페이지로 이동 후 수정되었는지 확인한다.

#### <div id='3-2-3-2'/> 3.2.3.2. JOB 감사 추적
###### <div id='3-2-3-2-1'/> 3.2.3.2.1. JOB 감사 추적 조회
1.      파이프라인 관리 > JOB 감사 추적 메뉴를 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146868232-07e6f6bc-c4d3-45c6-a16c-4f0297796724.png)
2.      JOB 감사 추적 대시보드에서 최근 등록/실행/수정/삭제된 JOB 목록을 조회할 수 있다.
![image](https://user-images.githubusercontent.com/80228983/146868328-1bdac61c-9e7d-4331-bea8-61ccb5e9657b.png)


###### <div id='3-2-3-2-2'/> 3.2.3.2.2. JOB 감사 추적 검색 조회
1.	JOB 감사 추적 대시보드에서 검색하고자 하는 메시지를 입력한다.
![image](https://user-images.githubusercontent.com/80228983/146868426-029fff3a-3eb2-46ab-9631-2cf8fd890fae.png)
2.	JOB 감사 추적 페이지 안에 검색한 메시지와 일치하는 추적 기록만 조회된다.
![image](https://user-images.githubusercontent.com/80228983/146868447-a51aa764-5e56-4902-83e7-538ba3f60cec.png)


### <div id='3-2-4'/> 3.2.4. 품질 관리
본 장에서는 정적분석 Job을 통해 검사한 소스 코드와 관련하여 품질 이슈와 코딩 규칙, 품질 프로파일, 품질 게이트에 대한 설명을 기술한다.
#### <div id='3-2-4-1'/> 3.2.4.1. 품질 이슈
1.	품질 관리 메뉴에서 품질 이슈를 클릭하여 품질 이슈 대시보드로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146868645-21adac37-a946-4dcb-8fce-b3d4e642b0f9.png)
2.      보고자하는 코딩 규칙의 “상세” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146868787-a3182e36-27f6-4925-9c24-4ca302195167.png)
3.	오른쪽 상단의 “목록” 버튼을 클릭하면 Job 테스트 목록이 있는 품질 이슈 대시보드로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146868854-58ede260-4324-4e70-8623-cb5378403a85.png)
<br>
![image](https://user-images.githubusercontent.com/80228983/146868893-612518cf-0f65-4b81-a184-0a1d2b4ac8e7.png)


#### <div id='3-2-4-2'/> 3.2.4.2. 코딩 규칙
1.	품질 관리 메뉴에서 코딩 규칙을 클릭하여 코딩 규칙 대시보드로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146869100-bf9ced1d-6ac5-43d5-9772-dacb723c71bb.png)
2.	“프로파일에 추가” 버튼을 클릭한 뒤 프로파일 추가 팝업 창에서 이슈 수준을 정한 후 “추가” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146869126-57b894b2-3bb4-4dfc-a5f8-945e4bcee0dc.png) 
![image](https://user-images.githubusercontent.com/80228983/146869144-225b9a5b-8795-4eee-98c0-d8aea5c5095a.png) 

3. 왼쪽 하단에 품질 프로파일 메뉴에서 이전 단계에서 선택한 default-paasta 을 클릭하여 추가한 코딩 규칙이 추가되었는지 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146869220-88df7154-a0e9-49a2-aed0-399ed89856d6.png)
4.	“프로파일에 제거” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146869258-5f0dd80d-4ca1-4edc-ab98-a168a2d04e0b.png)
5.	제거한 코딩 규칙이 품질 프로파일에서 삭제된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146869296-2399267c-6685-40ee-9e94-e66875b63841.png)

#### <div id='3-2-4-3'/> 3.2.4.3. 품질 프로파일
##### <div id='3-2-4-3-1'/> 3.2.4.3.1. 품질 프로파일 생성
1.	품질 관리 메뉴에서 품질 프로파일을 선택하여 품질 프로파일 대시보드로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146869326-75a4cefe-3359-4868-961d-c40cabff7471.png)
2.	우측 상단의 “생성” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146869368-150fd27b-c848-492a-9bc9-daf82bad9dd4.png)
3.	품질 프로파일 명을 입력하고, 개발언어를 선택하여 “생성” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146869418-4b14c549-055d-46d9-9645-2ecb39ed267d.png)
4.	품질 프로파일이 생성된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146869466-70d23957-9668-44e1-90ff-18f743060566.png)

##### <div id='3-2-4-3-2'/> 3.2.4.3.2. 품질 프로파일 복제
1.	품질 프로파일 대시보드에서 “복제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146869595-579a04e7-92ea-4f2f-b67d-7d4567b417a4.png)
2.	복제할 품질 프로파일의 이름을 입력하고, “복제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146869642-41573c30-9867-484c-b3db-d36c7c5396de.png)
3.	복제된 품질 프로파일을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146869693-2c0bced9-3618-4e4b-ba3d-a839706bb567.png)

##### <div id='3-2-4-3-3'/> 3.2.4.3.3. 품질 프로파일 수정
1.	품질 프로파일 대시보드에서 “수정” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146869730-5e78b1e5-70ca-4472-b2a4-fdeb45fe7c3a.png)
2.	수정할 품질 프로파일 팝업창이 뜨면 품질 프로파일 명을 수정하고 “수정” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146869753-e5a28251-de12-4331-b20e-7abcfd4e2cab.png)
3. 품질 프로파일 명이 수정되었음을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146869782-6fb49ca0-1a77-43b7-b497-12ef3a5a4728.png)

##### <div id='3-2-4-3-4'/> 3.2.4.3.4. 품질 프로파일 프로젝트 연결
1.	품질 프로파일 대시보드에서 연결된 프로젝트 항목을 확인한다. (첫 번째 사진은 테스트 Job 구성 조회 시 품질 프로파일을 Sonar way 로 설정한 것이다. 그러므로 두 번째 사진에서 예시로 새로 생성한 품질 프로파일[GuideModified]에는 연결된 프로젝트가 보이지 않는다.)
![image](https://user-images.githubusercontent.com/80228983/146869881-a5b19ce0-c43c-430f-bbc0-34e648c7816d.png)
2.	연결된 프로젝트가 없을 경우 “미연결” 탭을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146870059-5edd86ac-a917-4d68-9057-df6b095d0cef.png) 
3.	프로파일을 연결할 경우 "연결됨" 탭에 연결된 프로젝트가 확인된다.
![image](https://user-images.githubusercontent.com/80228983/146870218-7903313a-1acd-4c5d-bb8e-bf2150e3aa3f.png) 
![image](https://user-images.githubusercontent.com/80228983/146870235-e1d10c13-4001-47a9-b936-0684ded07a00.png) 

##### <div id='3-2-4-3-5'/> 3.2.4.3.5. 품질 프로파일 삭제
1.	품질 프로파일 대시보드에서 “삭제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146870360-07dfcc04-779c-4330-b46c-d7e3582b8ad8.png)

#### <div id='3-2-4-4'/> 3.2.4.4. 품질 게이트
##### <div id='3-2-4-4-1'/> 3.2.4.4.1. 품질 게이트 생성
1.	품질 관리 메뉴에서 품질 게이트를 선택하여 품질 게이트 대시보드로 이동한다.
![image](https://user-images.githubusercontent.com/80228983/146870977-2e815cd0-06a0-45d3-8b74-ae08b13ebb96.png)
2.	우측 상단의 “생성” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146870997-cb6f0416-54f2-4943-add9-f8afef3963ee.png)
3.	품질 게이트 명을 입력하고, “생성” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146871027-05bff6f6-dfc8-4be5-bf9c-12f6c5afcf8b.png)
4.	품질 게이트가 생성된 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146871082-9f7b5524-7a08-4a65-b32e-5451a51d8192.png)

##### <div id='3-2-4-4-2'/> 3.2.4.4.2. 품질 게이트 복제
1.	품질 게이트 대시보드에서 “복제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146871101-e7d27bb3-088e-4e7c-8f2f-94d4c2674d74.png)
2.	복제할 품질 게이트의 이름을 입력하고, “복제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146871144-b568c53d-0eb2-44b8-b497-ed41f27d7c63.png)


##### <div id='3-2-4-4-3'/> 3.2.4.4.3. 품질 게이트 수정
1.	품질 게이트 대시보드에서 “수정” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146871383-81744c10-8694-430d-a5e1-3a8f2ba53fda.png)
2.	수정할 품질 게이트 팝업창이 뜨면 품질 게이트 명을 수정하고 “수정” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146871410-9462890c-ce73-4824-9200-4e19681bd17d.png)


##### <div id='3-2-4-4-4'/> 3.2.4.4.4. 품질 게이트 조건추가
1.	품질 게이트 대시보드에서 Job 테스트 시 통과 기준이 되는 조건을 설정할 수 있는 조건 추가 부분을 확인한다. 사용자가 직접 조건을 추가하고 기준 설정이 가능하다. 저장/수정 열에서 저장을 클릭하여 저장한다.
![image](https://user-images.githubusercontent.com/80228983/146871550-aadfbdcc-d228-4838-89f7-50e77865b71f.png)
2.	조건에 따라 어느 기준 이상/이하가 될 시에 테스트를 통과시키도록 한다.
![image](https://user-images.githubusercontent.com/80228983/146871698-fd190b68-328f-46f3-813b-1923fe3171e9.png)

##### <div id='3-2-4-4-5'/> 3.2.4.4.5. 품질 게이트 프로젝트 연결
1. 품질 게이트 대시보드에서 연결된 프로젝트 항목을 확인한다. (첫 번째 사진은 정적분석 Job 구성 조회 시 품질 게이트를 fortest 로 설정한 것이다. )
![image](https://user-images.githubusercontent.com/80228983/146872137-88e8c4cd-bae9-4f19-bde7-513be375d303.png)
2. 품질 게이트도 품질 프로파일과 마찬가지로 품질 게이트 1개당 여러 프로젝트 연결이 가능하다.


##### <div id='3-2-4-4-6'/> 3.2.4.4.6. 품질 게이트 삭제
1.	품질 게이트 대시보드에서 “삭제” 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146872354-95a8dc91-4c18-43d5-9b5d-bbe76bd9226b.png)
2.	품질 게이트가 삭제되었음을 확인한다.


#### <div id='3-2-4-5'/> 3.2.4.5. 스테이징 관리
##### <div id='3-2-4-5-1'/> 3.2.4.5.1. 환경 정보 관리
배포된 애플리케이션의 환경 설정을 관리하는 페이지이다. 컨테이너 플랫폼 파이프라인에서는 spring cloud config server 를 제공하며, 환경 정보를 DB에 저장하여 관리한다.

###### <div id='3-2-4-5-1-1'/> 3.2.4.5.1.1. 환경 정보 등록
1. 환경 정보 관리 페이지에 접속한다.
![image](https://user-images.githubusercontent.com/80228983/146874425-487939dd-2877-45a6-ba78-075c1d096fd0.png) 
2. 환경 정보 관리 대시보드 우측의 추가 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146874528-b9274fa9-4cfc-43c6-874f-596847eb699f.png)
3. 환경 정보 등록 페이지 내 파일 선택 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146874591-9421c887-c653-4c24-ae81-9fad12bd44e1.png)
***※    .properties, .yaml', .yml 형식을 선택하며, '----' 등의 구분선이 포함되지 않아야한다.***
4. 올바르게 선택한 경우 정상이라는 팝업 알림창이 뜬다.
![image](https://user-images.githubusercontent.com/80228983/146874707-3efa4b94-2af7-437e-8004-efde7ae737a8.png)
5. 상황에 맞게 Application 명, Profile 명을 입력한다.
![image](https://user-images.githubusercontent.com/80228983/146874773-59b32463-844f-4d76-bc55-7316bdb777b8.png)
***※	Container Platform 파이프라인에서는 Label 명이 'standalone'으로 고정된다.***
6. 등록 버튼을 클릭하여 환경정보를 등록한다.
![image](https://user-images.githubusercontent.com/80228983/146874930-d4fd9c03-ef65-4d3d-8d04-a5de6c0f6d71.png)
7. 환경 정보가 제대로 등록되었음을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146874997-8f397560-05ff-4d01-9ed6-27e23983cc77.png)

###### <div id='3-2-4-5-1-2'/> 3.2.4.5.1.2. 환경 정보 목록 검색
환경정보 관리 대시보드에서는 'Application 명', 'profile 명', 'Key 검색' 3가지를 통해 검색할 수 있다.
1. Application 필터를 확장시켜 원하는 어플리케이션을 선택한다.
![image](https://user-images.githubusercontent.com/80228983/146875629-2c192489-d556-40ec-8769-53bc31abbf1f.png)
2. 선택된 어플리케이션으로 필터링되어 조회된다.
![image](https://user-images.githubusercontent.com/80228983/146875710-102c4b0d-d265-498f-a13d-78afbcb333ce.png)
3. 선택된 프로필명으로 필터링되어 조회되는 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146875761-2f72dff9-2346-4f84-9a21-952e735f4fe3.png)
3. 입력한 Key명으로 필터링되어 조회되는 것을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146875822-e2e85169-4cf7-417f-b80c-3a9ed13bf38b.png)

###### <div id='3-2-4-5-1-3'/> 3.2.4.5.1.3. 환경 정보 삭제
환경정보 관리 대시보드에서는 선택한 어플리케이션/프로파일 정보를 삭제할 수 있다.
1. Application 필터를 확장시켜 삭제하고자 하는 어플리케이션을 선택한다.
![image](https://user-images.githubusercontent.com/80228983/146875945-98cf9929-1c37-417d-b1e1-c13d9b077f03.png) 
2. Profile 필터를 확장시켜, 삭제하고자하는 프로필을 선택한다.
![image](https://user-images.githubusercontent.com/80228983/146876117-de125635-3612-43b1-9de4-1d1781b3c67e.png)
3. paasta-delivery-pipeline-inspection-api 어플리케이션, 'default' 프로필을 선택하고, 'Application 삭제' 버튼을 클릭한다.
![image](https://user-images.githubusercontent.com/80228983/146876164-ffb906d8-0a8a-4aef-ac1d-82469c87c0ee.png)
4. 선택한 환경정보가 삭제되었음을 확인한다.
![image](https://user-images.githubusercontent.com/80228983/146876207-8d3b8d4e-4fe3-4e33-8fbe-5a41fd8bc1f2.png)

<br>

### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Use](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/use-guide/Readme.md) > Pipeline 서비스 사용 가이드