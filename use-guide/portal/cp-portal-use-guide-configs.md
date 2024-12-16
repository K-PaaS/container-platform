### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Configs 메뉴

<br>

## Table of Contents

1. [Configs 메뉴](#1)  
  1.1. [ConfigMaps](#1-1)  
    1.1.1. [ConfigMap 목록 조회](#1-1-1)  
    1.1.2. [ConfigMap 상세 조회](#1-1-2)  
    1.1.3. [ConfigMap 생성](#1-1-3)  
    1.1.4. [ConfigMap 수정](#1-1-4)  
    1.1.5. [ConfigMap 삭제](#1-1-5)  
  1.2. [Secrets](#1-2)  
    1.2.1. [Secret 목록 조회](#1-2-1)  
    1.2.2. [Secret 상세 조회](#1-2-2)  
    1.2.3. [Secret 생성](#1-2-3)  
    1.2.3.1. [Opaque 타입](#1-2-3-1)  
    1.2.3.2. [kubernetes.io/tls 타입](#1-2-3-2)  
    1.2.3.3. [kubernetes.io/ssh-auth 타입](#1-2-3-3)  
    1.2.3.4. [kubernetes.io/dockercfg 타입](#1-2-3-4)  
    1.2.3.5. [bootstrap.kubernetes.io/token 타입](#1-2-3-5)  
    1.2.3.6. [kubernetes.io/service-account-token 타입](#1-2-3-6)  
    1.2.4. [Secret 수정](#1-2-4)  
    1.2.5. [Secret 삭제](#1-2-5)  
    1.2.6. [Vault Secret 목록 조회](#1-2-6)  
    1.2.7. [Vault Secret 상세 조회](#1-2-7)  
    1.2.8. [Vault Secret 생성](#1-2-8)  
    1.2.9. [Vault Secret 적용](#1-2-9)  
    1.2.10. [Vault Secret 수정](#1-2-10)  
    1.2.11. [Vault Secret 삭제](#1-2-11)    
 
<br>

## <div id='1'/> 1. Configs 메뉴
### <div id='1-1'/> 1.1. ConfigMaps
#### <div id='1-1-1'/> 1.1.1. ConfigMap 목록 조회
- Configs의 ConfigMaps를 클릭하여 ConfigMap 목록 페이지로 이동한다.
  ![IMG_6_1_1]

<br>

#### <div id='1-1-2'/> 1.1.2. ConfigMap 상세 조회
- ConfigMap 목록에서 ConfigMap명을 클릭하여 ConfigMap 상세 페이지로 이동한다.
  ![IMG_6_1_2]

<br>

#### <div id='1-1-3'/> 1.1.3. ConfigMap 생성
- ConfigMap 목록에서 생성 버튼을 클릭할 시 ConfigMap 생성 팝업창이 뜬다.
  ![IMG_6_1_3]

<br>

#### <div id='1-1-4'/> 1.1.4. ConfigMap 수정
- ConfigMap 상세에서 수정 버튼을 클릭할 시 ConfigMap 수정 팝업창이 뜬다.
  ![IMG_6_1_4]

<br>

#### <div id='1-1-5'/> 1.1.5. ConfigMap 삭제
- ConfigMap 상세에서 삭제 버튼을 클릭할 시 ConfigMap 삭제가 완료된다.
  ![IMG_6_1_5]

<br>

### <div id='1-2'/> 1.2. Secrets
#### <div id='1-2-1'/> 1.2.1. Secret 목록 조회
- Configs의 Secrets를 클릭하여 Kubernetes 탭을 선택하여 Secret 목록 페이지로 이동한다.
  ![IMG_6_2_1]

<br>

#### <div id='1-2-2'/> 1.2.2. Secret 상세 조회
- Secret 목록에서 Secret명을 클릭하여 Secret 상세 페이지로 이동한다.
  ![IMG_6_2_2]

<br>

#### <div id='1-2-3'/> 1.2.3. Secret 생성
##### <div id='1-2-3-1'/> 1.2.3.1. Opaque 타입
- Secret 목록에서 생성 버튼을 클릭할 시 Secret 생성 페이지로 이동한다.
- Storage Backend 셀렉트박스에서 Kubernetes를 선택한다.
  ![IMG_6_2_3]

- 파일 업로드 버튼을 클릭하여 파일을 Value 값으로 업로드할 수 있다.
  ![IMG_6_2_4]

- 데이터 추가 버튼을 클릭하여 Key, Value 데이터를 추가 입력할 수 있다.
  ![IMG_6_2_5]

<br>

##### <div id='1-2-3-2'/> 1.2.3.2. kubernetes.io/tls 타입
- Key 값에 tls.key 또는 tls.crt를 입력하면 자동으로 Data Type이 kubernetes.io/tls 타입으로 변경되고 데이터 추가 버튼을 누르게 되면 자동으로 tls.key 또는 tls.crt가 Key 값에 입력되어 추가가 된다.
  ![IMG_6_2_6]  
  ![IMG_6_2_7]

<br>

##### <div id='1-2-3-3'/> 1.2.3.3. kubernetes.io/ssh-auth 타입
- Key 값에 ssh-privatekey를 입력하면 자동으로 Data Type이 kubernetes.io/ssh-auth 타입으로 변경된다.
  ![IMG_6_2_8]

<br>

##### <div id='1-2-3-4'/> 1.2.3.4. kubernetes.io/dockercfg 타입
- Specifying Data Types 체크 박스를 클릭하여 Data Type 셀렉트 박스가 활성화가 되면 Docker Config를 선택할 수 있다. Docker Config 타입은 Key 값으로 docker-server, docker-username, docker-password 필수 입력 값이 있고 docker-email 옵션 입력 값이 있다.
  ![IMG_6_2_9]

<br>

##### <div id='1-2-3-5'/> 1.2.3.5. bootstrap.kubernetes.io/token 타입
- Specifying Data Types 체크 박스를 클릭하여 Data Type 셀렉트 박스가 활성화가 되면 Boot Strap Token을 선택할 수 있다. Boot Strap Token 타입은 Key 값으로 token-id, token-secret, usage-bootstrap-authentication, usage-bootstrap-signing 필수 입력 값이 있고 auth-extra-group, expiration, description 옵션 입력 값이 있다.
  ![IMG_6_2_10]

<br>

##### <div id='1-2-3-6'/> 1.2.3.6. kubernetes.io/service-account-token 타입
- Specifying Data Types 체크 박스를 클릭하여 Data Type 셀렉트 박스가 활성화가 되면 Service Account Token을 선택할 수 있다. Service Account Token 타입은 동일 Namespace에 Service Account가 사전에 생성되어 있어야하고 Service Account Name 셀렉트 박스에서 선택할 수 있다.
  ![IMG_6_2_11]

<br>

#### <div id='1-2-4'/> 1.2.4. Secret 수정
- Secret 상세에서 Data 목록의 Edit 버튼을 눌러 Value 값을 수정하고 수정 버튼을 클릭하여 수정한다.
  ![IMG_6_2_12]

<br>

#### <div id='1-2-5'/> 1.2.5. Secret 삭제
- Secret 상세에서 삭제 버튼을 클릭할 시 Secret 삭제가 완료된다.
  ![IMG_6_2_13]

<br>

#### <div id='1-2-6'/> 1.2.6. Vault Secret 목록 조회
- Configs의 Secrets를 클릭하여 Vault 탭을 선택하여 Vault Secret 목록 페이지로 이동한다.
  ![IMG_6_2_14]

<br>

#### <div id='1-2-7'/> 1.2.7. Vault Secret 상세 조회
- Vault Secret 목록에서 Vault Secret명을 클릭하여 Vault Secret 상세 페이지로 이동한다.
  ![IMG_6_2_15]

<br>

#### <div id='1-2-8'/> 1.2.8. Vault Secret 생성
- Vault Secret 목록에서 생성 버튼을 클릭할 시 Secret 생성 페이지로 이동한다.
- Storage Backend 셀렉트박스에서 Vault를 선택한다.
- Vault Secret을 생성하기 위해서는 사전에 Postgresql DB가 설치되어 있어야한다. DB Types 셀렉트 박스는 Postgres SQL을 선택하고 DB Service는 생성할 Postgre SQL 서비스를 선택한다. Default TTL은 Vault Secret을 사용할 유효한 시간을 정하고 Max TTL은 Default TTL시간 동안 최대한으로 유효 가능 시간을 정한다.
  ![IMG_6_2_16]

<br>

#### <div id='1-2-9'/> 1.2.9. Vault Secret 적용
- Vault Secret 상세에서 적용 버튼을 클릭할 시 Apply 팝업창에 선택한 Namespace에 대한 Deployment 목록이 나온다. Vault Secret Data를 적용할 Deployment를 선택하여 저장 버튼을 클릭한다.
  ![IMG_6_2_17]

<br>

#### <div id='1-2-10'/> 1.2.10. Vault Secret 수정
- Vault Secret 상세에서 Data 목록의 Edit 아이콘을 눌러 Default TTL, Max TTL 값을 수정하고 수정 버튼을 클릭하여 수정한다.
  ![IMG_6_2_18]

<br>

#### <div id='1-2-11'/> 1.2.11. Vault Secret 삭제
- Vault Secret 상세에서 삭제 버튼을 클릭할 시 Vault Secret 삭제가 완료된다.
  ![IMG_6_2_19]

<br>


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Configs 메뉴

[IMG_6_1_1]:../images/portal/IMG_6_1_1.png
[IMG_6_1_2]:../images/portal/IMG_6_1_2.png
[IMG_6_1_3]:../images/portal/IMG_6_1_3.png
[IMG_6_1_4]:../images/portal/IMG_6_1_4.png
[IMG_6_1_5]:../images/portal/IMG_6_1_5.png
[IMG_6_2_1]:../images/portal/IMG_6_2_1.png
[IMG_6_2_2]:../images/portal/IMG_6_2_2.png
[IMG_6_2_3]:../images/portal/IMG_6_2_3.png
[IMG_6_2_4]:../images/portal/IMG_6_2_4.png
[IMG_6_2_5]:../images/portal/IMG_6_2_5.png
[IMG_6_2_6]:../images/portal/IMG_6_2_6.png
[IMG_6_2_7]:../images/portal/IMG_6_2_7.png
[IMG_6_2_8]:../images/portal/IMG_6_2_8.png
[IMG_6_2_9]:../images/portal/IMG_6_2_9.png
[IMG_6_2_10]:../images/portal/IMG_6_2_10.png
[IMG_6_2_11]:../images/portal/IMG_6_2_11.png
[IMG_6_2_12]:../images/portal/IMG_6_2_12.png
[IMG_6_2_13]:../images/portal/IMG_6_2_13.png
[IMG_6_2_14]:../images/portal/IMG_6_2_14.png
[IMG_6_2_15]:../images/portal/IMG_6_2_15.png
[IMG_6_2_16]:../images/portal/IMG_6_2_16.png
[IMG_6_2_17]:../images/portal/IMG_6_2_17.png
[IMG_6_2_18]:../images/portal/IMG_6_2_18.png
[IMG_6_2_19]:../images/portal/IMG_6_2_19.png