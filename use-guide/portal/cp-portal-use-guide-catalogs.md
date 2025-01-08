### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Catalogs 메뉴

<br>

## Table of Contents

1. [Catalogs 메뉴](#1)  
   1.1. [Repositories](#1-1)  
   1.1.1. [기본 차트 저장소 추가](#1-1-1)  
   1.1.2. [차트 저장소 추가](#1-1-2)  
   1.1.3. [차트 저장소 목록 조회](#1-1-3)  
   1.1.4. [차트 저장소 캐시 삭제](#1-1-4)  
   1.1.5. [차트 저장소 업데이트](#1-1-5)  
   1.1.6. [차트 저장소 삭제](#1-1-6)  
   1.1.7. [차트 목록 조회](#1-1-7)  
   1.1.8. [차트 설치](#1-1-8)  
   1.2. [Releases](#1-2)   
   1.2.1. [릴리즈 목록 조회](#1-2-1)  
   1.2.2. [릴리즈 상세 조회](#1-2-2)  
   1.2.3. [릴리즈 업그레이드](#1-2-3)  
   1.2.4. [릴리즈 롤백](#1-2-4)  
   1.2.5. [릴리즈 삭제](#1-2-5)

<br>

## <div id='1'/> 1. Catalogs 메뉴
### <div id='1-1'/> 1.1. Repositories
컨테이너 플랫폼 포털 메뉴 Catalogs > Repositories 에서 차트 저장소 정보를 관리한다. <br>

#### <div id='1-1-1'/> 1.1.1. 기본 차트 저장소 추가
> **(Super Admin 권한만 가능)** 메뉴 Catalogs > Repositories  <br>

**[ChartMuseum](https://github.com/helm/chartmuseum)** 은 Helm 차트를 저장하고 관리하는 오픈소스 Repository 서버로 Helm을 사용하는 쿠버네티스 환경에서 차트 저장소를 제공하는 역할을 한다.
컨테이너 플랫폼 포털 배포에 포함되어 컨테이너 플랫폼 환경의 기본 차트 저장소로 활용된다. <br>

|이름|URL|
|---|---|
|cp-chart-repository|`https://chartmuseum.${HOST_DOMAIN}`|

<br>

##### 아래 등록 버튼을 클릭하여 기본 차트 저장소를 추가한다.
![IMG_9_1_1]

<br>

##### 기본 차트 저장소에 차트 업로드
차트 업로드는 CLI를 통해 가능하다.
```bash
# Master Node 접속
# helm repo 목록 조회 (포털 배포 시 cp-chart-repository 자동 등록)
$ helm repo list
NAME                    URL
cp-chart-repository     https://chartmuseum.105.xxx.xxx.xxx.nip.io

# 업로드할 차트 
$ ls
nginx-18.2.6.tgz

# helm 플러그인 통한 저장소에 차트 업로드 
$ helm cm-push nginx-18.2.6.tgz cp-chart-repository
Pushing nginx-18.2.6.tgz to cp-chart-repository...
Done.

# 저장소 업데이트
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "cp-chart-repository" chart repository
Update Complete. ⎈Happy Helming!⎈

# 차트 업로드 확인
$ helm search repo nginx
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
cp-chart-repository/nginx       18.2.6          1.27.3          NGINX Open Source is a web server that can be a...
```

##### 포털 화면에서 저장소 업데이트
![IMG_9_1_2]
![IMG_9_1_3]
![IMG_9_1_4]


<br>


#### <div id='1-1-2'/> 1.1.2. 차트 저장소 추가
> 메뉴 Catalogs > Repositories > [추가] 버튼 클릭

차트 저장소를 등록한다.

##### 차트 저장소 추가 항목
|항목|설명|정보|
|---|---|---|
|Name|**(필수)** 차트 저장소 명 입력||
|URL |**(필수)** 차트 저장소 URL 입력||
|Username|차트 저장소 사용자 명 입력|Authentication 체크박스 클릭|
|Password|차트 저장소 비밀번호 입력|Authentication 체크박스 클릭|
|CA Certificate|사설 차트 저장소에서 HTTPS 사용의 경우 인증서 입력|TLS Verification 체크박스 클릭|

##### (예시) Bitnami 추가
![IMG_9_1_5]
![IMG_9_1_6]

##### (예시) 사설 차트 저장소 추가
![IMG_9_1_7]


<br>


#### <div id='1-1-3'/> 1.1.3. 차트 저장소 목록 조회
> 메뉴 Catalogs > Repositories

##### 등록된 차트 저장소 목록을 조회한다.
![IMG_9_1_8]


<br>

#### <div id='1-1-4'/> 1.1.4. 차트 저장소 캐시 삭제
> **(Super Admin 권한만 가능)** 메뉴 Catalogs > Repositories > [캐시 삭제] 버튼 클릭
#### 스토리지에 캐쉬된 차트 파일 삭제
스토리지에 캐시된 차트 파일을 삭제한 후, 모든 차트 저장소를 최신 정보로 업데이트한다. <br>
모든 차트 저장소의 인덱스 파일을 새로 받아오기 때문에, 등록된 저장소가 많을수록 시간이 더 소요될 수 있다. <br> 따라서 이 작업은 자주 수행하지 않는 것을 권장한다.


![IMG_9_1_9]
![IMG_9_1_10]

<br>

#### <div id='1-1-5'/> 1.1.5. 차트 저장소 업데이트
> 메뉴 Catalogs > Repositories > 차트 저장소 명 클릭 > [업데이트] 버튼 클릭
##### 해당 차트 저장소의 차트를 최신정보로 업데이트 한다.
![IMG_9_1_11]
![IMG_9_1_12]

<br>

#### <div id='1-1-6'/> 1.1.6. 차트 저장소 삭제
> 메뉴 Catalogs > Repositories > 차트 저장소 명 클릭 > [삭제] 버튼 클릭
##### 해당 차트 저장소를 삭제한다.
![IMG_9_1_13]
![IMG_9_1_14]


<br>

#### <div id='1-1-7'/> 1.1.7. 차트 목록 조회
> 메뉴 Catalogs > Repositories > 차트 저장소 명 클릭

##### 차트 저장소에 등록된 차트를 확인한다.
![IMG_9_1_15]

<br>

#### <div id='1-1-8'/> 1.1.8. 차트 설치
> 메뉴 Catalogs > Repositories > 차트 저장소 명 클릭 > 차트 명 클릭

해당 차트를 클러스터에 설치한다.

##### 차트 설치 항목
|항목|입력 정보|
|---|---|
|Chart Versions|설치할 차트 버전 선택|
|Release Name|설치할 릴리즈 명 입력|
|기본 Chart Values|(Read Only) 차트에서 제공하는 Original values.yaml|
|사용자 정의 Values|**(값 수정)** 기본 Chart Values 값을 기준으로 사용자가 값을 재정의|

##### KeyMap 사용법
|KeyMap|정보|
|---|---|
|Ctrl-F|키워드 검색 시작 (Start searching)|
|Ctrl-G|키워드 다음 찾기 (Find next)|
|Shift-Ctrl-G|키워드 이전 찾기 (Find previous)|
|Shift-Ctrl-F|문자 치환 (Replace)|
|Shift-Ctrl-R|문자 전체 치환 (Replace all)|
|Alt-G|라인 이동 (Jump to line)|

<br>

#### (예시) Bitnami Nginx 설치
- 차트를 설치할 네임스페이스 선택
- 설치할 차트버전(Chart Versions) 선택
- 설치할 릴리즈 명(Release Name) 입력

![IMG_9_1_16]

##### 사용자 정의 Values 재정의
- commonLabels 추가 : `"app": "mynginx"`
- Service Type 지정: `NodePort`
- Service NodePort 지정: `30000 (http)`

###### 사용자 정의 Values 창 확장
![IMG_9_1_17]

###### 사용자 정의 Values 창 축소
![IMG_9_1_18]


##### 차트 설치
![IMG_9_1_19]
![IMG_9_1_20]


<br>


### <div id='1-2'/> 1.2. Releases 메뉴
컨테이너 플랫폼 포털 메뉴 Catalogs > Releases 에서 릴리즈 정보를 관리한다.

#### <div id='1-2-1'/> 1.2.1. 릴리즈 목록 조회
> 메뉴 Catalogs > Releases

##### 배포된 릴리즈 목록을 조회한다.
![IMG_9_2_1]

<br>

#### <div id='1-2-2'/> 1.2.2. 릴리즈 상세 조회
> 메뉴 Catalogs > Releases > Releases 명 선택

##### 해당 릴리즈의 상세 정보를 조회한다.

|항목|설명|
|---|---|
|Details| 릴리즈 상세 정보|
|Histories| 릴리즈에 대한 Revision 기록 정보|

![IMG_9_2_2]

|항목|설명|
|---|---|
|Resources Status|릴리즈에 포함된 쿠버네티스 리소스 목록 정보|

![IMG_9_2_3]

|항목|설명|
|---|---|
|Manifests|릴리즈의 Manifest 정보<br>Manifest는 해당 릴리즈의 차트에서 생성된 쿠버네티스 리소스를 YAML로 인코딩한 값이다.|

![IMG_9_2_4]

|항목|설명|
|---|---|
|Values|릴리즈에 대한 Values 정보|

![IMG_9_2_5]


|항목|설명|
|---|---|
|Notes|릴리즈의 차트에서 제공하는 노트 정보|

![IMG_9_2_6]


<br>

#### <div id='1-2-3'/> 1.2.3. 릴리즈 업그레이드
> 메뉴 Catalogs > Releases > Releases 명 선택

##### 릴리즈를 업그레이드한다.
- 차트 버전 업그레이드
- Values 값 업데이트

|항목|입력 정보|
|---|---|
|Chart Versions|업그레이드할 차트 버전 선택|
|배포된 Values|(Read Only) 현재 배포된 릴리즈의 Values 값|
|사용자 정의 Values|**(값 수정)** 현재 배포된 릴리즈의 Values 값을 기준으로 사용자가 Values 재정의|
|배포된 Manifest|(Read Only) 현재 배포된 릴리즈의 Manifest|
|렌더링된 Manifest|(Read Only) 새로운 차트 버전 또는 재정의된 Values 값(사용자 정의 Values)이 렌더링 된 Manifest 미리보기|

##### KeyMap 사용법
|KeyMap|정보|
|---|---|
|Ctrl-F|키워드 검색 시작 (Start searching)|
|Ctrl-G|키워드 다음 찾기 (Find next)|
|Shift-Ctrl-G|키워드 이전 찾기 (Find previous)|
|Shift-Ctrl-F|문자 치환 (Replace)|
|Shift-Ctrl-R|문자 전체 치환 (Replace all)|
|Alt-G|라인 이동 (Jump to line)|

#### (예시) Bitnami Nginx 업그레이드
- 업그레이드 할 차트 버전 선택
- Service Type 변경: `ClusterIP`
- Ingress 활성화
- IngressClassName 지정
- Ingress hostname 지정

![IMG_9_2_7]


##### 사용자 정의 Values 재정의
![IMG_9_2_8]

##### [차트 템플릿 렌더링 비교] 버튼 클릭
업그레이드할 차트 버전 또는 변경된 사용자 정의 Values 값을 기준으로 Manifest를 미리보기용으로 새로 렌더링한다.

![IMG_9_2_9]

##### 렌더링된 Manifest 확인 (미리보기)
![IMG_9_2_10]

##### 릴리즈 업그레이드 진행
![IMG_9_2_11]
![IMG_9_2_12]

##### 릴리즈 업그레이드 후 릴리즈 상태 확인
![IMG_9_2_13]

<br>

#### <div id='1-2-4'/> 1.2.4. 릴리즈 롤백
> 메뉴 Catalogs > Releases > Releases 명 선택 > [롤백] 버튼 클릭<br>
- *Histories에서 Revision 버전이 최대 1인 경우 [롤백] 버튼은 비활성화이다.*

##### 되돌릴 Rollback Revision 번호를 선택한다.
![IMG_9_2_14]

##### 릴리즈 Revision 별 Manifest 비교 (축소 버튼 클릭)
![IMG_9_2_15]

##### 릴리즈 롤백 진행
![IMG_9_2_16]
![IMG_9_2_17]

##### 릴리즈 롤백 후 릴리즈 상태 확인
![IMG_9_2_18]


 <br>

#### <div id='1-2-5'/> 1.2.5. 릴리즈 삭제
> 메뉴 Catalogs > Releases > Releases 명 선택 > [삭제] 버튼 클릭

##### 릴리즈를 삭제한다.
![IMG_9_2_19]
![IMG_9_2_20]


<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Catalogs 메뉴

[IMG_9_1_1]:../images/portal/IMG_9_1_1.png
[IMG_9_1_2]:../images/portal/IMG_9_1_2.png
[IMG_9_1_3]:../images/portal/IMG_9_1_3.png
[IMG_9_1_4]:../images/portal/IMG_9_1_4.png
[IMG_9_1_5]:../images/portal/IMG_9_1_5.png
[IMG_9_1_6]:../images/portal/IMG_9_1_6.png
[IMG_9_1_7]:../images/portal/IMG_9_1_7.png
[IMG_9_1_8]:../images/portal/IMG_9_1_8.png
[IMG_9_1_9]:../images/portal/IMG_9_1_9.png
[IMG_9_1_10]:../images/portal/IMG_9_1_10.png
[IMG_9_1_11]:../images/portal/IMG_9_1_11.png
[IMG_9_1_12]:../images/portal/IMG_9_1_12.png
[IMG_9_1_13]:../images/portal/IMG_9_1_13.png
[IMG_9_1_14]:../images/portal/IMG_9_1_14.png
[IMG_9_1_15]:../images/portal/IMG_9_1_15.png
[IMG_9_1_16]:../images/portal/IMG_9_1_16.png
[IMG_9_1_17]:../images/portal/IMG_9_1_17.png
[IMG_9_1_18]:../images/portal/IMG_9_1_18.png
[IMG_9_1_19]:../images/portal/IMG_9_1_19.png
[IMG_9_1_20]:../images/portal/IMG_9_1_20.png
[IMG_9_2_1]:../images/portal/IMG_9_2_1.png
[IMG_9_2_2]:../images/portal/IMG_9_2_2.png
[IMG_9_2_3]:../images/portal/IMG_9_2_3.png
[IMG_9_2_4]:../images/portal/IMG_9_2_4.png
[IMG_9_2_5]:../images/portal/IMG_9_2_5.png
[IMG_9_2_6]:../images/portal/IMG_9_2_6.png
[IMG_9_2_7]:../images/portal/IMG_9_2_7.png
[IMG_9_2_8]:../images/portal/IMG_9_2_8.png
[IMG_9_2_9]:../images/portal/IMG_9_2_9.png
[IMG_9_2_10]:../images/portal/IMG_9_2_10.png
[IMG_9_2_11]:../images/portal/IMG_9_2_11.png
[IMG_9_2_12]:../images/portal/IMG_9_2_12.png
[IMG_9_2_13]:../images/portal/IMG_9_2_13.png
[IMG_9_2_14]:../images/portal/IMG_9_2_14.png
[IMG_9_2_15]:../images/portal/IMG_9_2_15.png
[IMG_9_2_16]:../images/portal/IMG_9_2_16.png
[IMG_9_2_17]:../images/portal/IMG_9_2_17.png
[IMG_9_2_18]:../images/portal/IMG_9_2_18.png
[IMG_9_2_19]:../images/portal/IMG_9_2_19.png
[IMG_9_2_20]:../images/portal/IMG_9_2_20.png