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
- Opaque 타입은 시크릿 구성 파일에서 시크릿 타입을 지정하지 않았을 경우의 기본 시크릿 타입이다.  
  ![IMG_6_2_3]

  <table>
    <thead>
      <tr>
        <th>항목</th>
        <th>설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Storage Backend</td>
        <td>암호화된 Secret 데이터가 저장될 스토리지를 선택한다.</td>
      </tr>     
      <tr>
        <td>Namespace</td>
        <td>Secret 리소스가 만들어지는 Namespace를 선택한다.</td>
      </tr>
       <tr>
        <td>Name</td>
        <td>Secret의 Name을 입력한다.</td>
      </tr>
      <tr>
        <td>Specifying Data Types</td>
        <td>Secret 데이터의 유형을 지정할 수 있다.</td>
      </tr>
      <tr>
        <td>Key</td>
        <td>Secret 데이터로 저장할 Key 값을 입력한다.</td>
      </tr>
      <tr>
        <td>Value</td>
        <td>Secret 데이터로 저장할 Value 값을 입력한다.</td>
      </tr>
      <tr>
        <td>파일 업로드</td>
        <td>Value 값으로 파일 업로드를 할 수 있다.</td>
      </tr>
      <tr>
        <td>데이터 추가</td>
        <td>Key, Value 쌍으로 입력할 데이터를 추가할 수 있다.</td>
      </tr>
    </tbody>
  </table>

<br>

##### <div id='1-2-3-2'/> 1.2.3.2. kubernetes.io/tls 타입
- kubernetes.io/tls 타입은 tls를 위해 사용되는 인증서 및 관련된 키를 저장하기 위한 시크릿타입이며 tls.key 값과 tls.crt 값이 필요하다.  
  ![IMG_6_2_4]    

  <table>
    <thead>
      <tr>
        <th>항목</th>
        <th>설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Storage Backend</td>
        <td>암호화된 Secret 데이터가 저장될 스토리지를 선택한다.</td>
      </tr>     
      <tr>
        <td>Namespace</td>
        <td>Secret 리소스가 만들어지는 Namespace를 선택한다.</td>
      </tr>
       <tr>
        <td>Name</td>
        <td>Secret의 Name을 입력한다.</td>
      </tr>
      <tr>
        <td>Specifying Data Types</td>
        <td>Secret 데이터의 유형을 지정할 수 있다.</td>
      </tr>
      <tr>
        <td>Key</td>
        <td>Secret 데이터로 저장할 Key 값을 입력한다. tls.key 또는 tls.crt를 입력하면 자동으로 Data Type이 kubernetes.io/tls 타입으로 변경된다.</td>
      </tr>
      <tr>
        <td>Value</td>
        <td>Secret 데이터로 저장할 Value 값을 입력한다.</td>
      </tr>
      <tr>
        <td>파일 업로드</td>
        <td>Value 값으로 파일 업로드를 할 수 있다.</td>
      </tr>
      <tr>
        <td>데이터 추가</td>
        <td>Key, Value 쌍으로 입력할 데이터를 추가할 수 있다. Key 값에 tls.key가 입력된 상태로 데이터 추가 버튼을 클릭하면 그 다음 데이터 Key 값으로 tls.crt가 자동으로 입력되고 Key 값에 tls.crt가 입력된 상태로 데이터 추가 버튼을 클릭하면 그 다음 데이터 Key 값으로 tls.key가 자동으로 입력된다.</td>
      </tr>
    </tbody>
  </table>

<br>

##### <div id='1-2-3-3'/> 1.2.3.3. kubernetes.io/ssh-auth 타입
- kubernetes.io/ssh-auth 타입은 ssh 인증에 사용되는 데이터를 저장하기 위한 시크릿타입이며 ssh-privatekey 값이 필요하다.  
  ![IMG_6_2_5]

  <table>
    <thead>
      <tr>
        <th>항목</th>
        <th>설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Storage Backend</td>
        <td>암호화된 Secret 데이터가 저장될 스토리지를 선택한다.</td>
      </tr>     
      <tr>
        <td>Namespace</td>
        <td>Secret 리소스가 만들어지는 Namespace를 선택한다.</td>
      </tr>
       <tr>
        <td>Name</td>
        <td>Secret의 Name을 입력한다.</td>
      </tr>
      <tr>
        <td>Specifying Data Types</td>
        <td>Secret 데이터의 유형을 지정할 수 있다.</td>
      </tr>
      <tr>
        <td>Key</td>
        <td>Secret 데이터로 저장할 Key 값을 입력한다. ssh-privatekey를 입력하면 자동으로 Data Type이 kubernetes.io/ssh-auth 타입으로 변경된다.</td>
      </tr>
      <tr>
        <td>Value</td>
        <td>Secret 데이터로 저장할 Value 값을 입력한다.</td>
      </tr>
      <tr>
        <td>파일 업로드</td>
        <td>Value 값으로 파일 업로드를 할 수 있다.</td>
      </tr>
      <tr>
        <td>데이터 추가</td>
        <td>Key, Value 쌍으로 입력할 데이터를 추가할 수 있다.</td>
      </tr>
    </tbody>
  </table>

<br>

##### <div id='1-2-3-4'/> 1.2.3.4. kubernetes.io/dockercfg 타입
- kubernetes.io/dockerconfigjson 타입은 이미지에 대한 도커 레지스트리 접속 자격 증명을 저장하기 위한 시크릿타입이며 docker-server(필수), docker-username(필수), docker-password(필수), docker-email(옵션) 값이 필요하다.  
  ![IMG_6_2_6]

  <table>
    <thead>
      <tr>
        <th>항목</th>
        <th>설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Storage Backend</td>
        <td>암호화된 Secret 데이터가 저장될 스토리지를 선택한다.</td>
      </tr>     
      <tr>
        <td>Namespace</td>
        <td>Secret 리소스가 만들어지는 Namespace를 선택한다.</td>
      </tr>
       <tr>
        <td>Name</td>
        <td>Secret의 Name을 입력한다.</td>
      </tr>
      <tr>
        <td>Specifying Data Types</td>
        <td>Secret 데이터의 유형을 지정할 수 있다.</td>
      </tr>
      <tr>
        <td>Key</td>
        <td>Secret 데이터로 저장할 Key 값을 입력한다.</td>
      </tr>
      <tr>
        <td>Value</td>
        <td>Secret 데이터로 저장할 Value 값을 입력한다.</td>
      </tr>
      <tr>
        <td>파일 업로드</td>
        <td>Value 값으로 파일 업로드를 할 수 있다.</td>
      </tr>      
    </tbody>
  </table>

<br>

##### <div id='1-2-3-5'/> 1.2.3.5. bootstrap.kubernetes.io/token 타입
- bootstrap.kubernetes.io/token 타입은 컨피그맵에 서명하는 데 사용되는 토큰을 저장하기 위한 시크릿타입이며 token-id(필수), token-secret(필수), usage-bootstrap-authentication(필수), usage-bootstrap-signing(필수), auth-extra-group(옵션), expiration(옵션), description(옵션) 값이 필요하다.  

  ![IMG_6_2_7]

  <table>
    <thead>
      <tr>
        <th>항목</th>
        <th>설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Storage Backend</td>
        <td>암호화된 Secret 데이터가 저장될 스토리지를 선택한다.</td>
      </tr>     
      <tr>
        <td>Namespace</td>
        <td>Secret 리소스가 만들어지는 Namespace를 선택한다.</td>
      </tr>
       <tr>
        <td>Name</td>
        <td>Secret의 Name을 입력한다.</td>
      </tr>
      <tr>
        <td>Specifying Data Types</td>
        <td>Secret 데이터의 유형을 지정할 수 있다.</td>
      </tr>
      <tr>
        <td>Key</td>
        <td>Secret 데이터로 저장할 Key 값을 입력한다.</td>
      </tr>
      <tr>
        <td>Value</td>
        <td>Secret 데이터로 저장할 Value 값을 입력한다.</td>
      </tr>
      <tr>
        <td>파일 업로드</td>
        <td>Value 값으로 파일 업로드를 할 수 있다.</td>
      </tr>      
    </tbody>
  </table>

<br>

##### <div id='1-2-3-6'/> 1.2.3.6. kubernetes.io/service-account-token 타입
- kubernetes.io/service-account-token 타입은 서비스 어카운트를 확인하는 토큰 자격증명을 저장하기 위한 시크릿타입이며 Service Account Name값이 필요하다. Service Account Token 타입은 동일 Namespace에 Service Account가 사전에 생성되어 있어야한다.   

  ![IMG_6_2_8]

  <table>
    <thead>
      <tr>
        <th>항목</th>
        <th>설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Storage Backend</td>
        <td>암호화된 Secret 데이터가 저장될 스토리지를 선택한다.</td>
      </tr>     
      <tr>
        <td>Namespace</td>
        <td>Secret 리소스가 만들어지는 Namespace를 선택한다.</td>
      </tr>
       <tr>
        <td>Name</td>
        <td>Secret의 Name을 입력한다.</td>
      </tr>
      <tr>
        <td>Specifying Data Types</td>
        <td>Secret 데이터의 유형을 지정할 수 있다.</td>
      </tr>
      <tr>
        <td>Service Account Name</td>
        <td>Service Account Name 값을 선택한다.</td>
      </tr>      
    </tbody>
  </table>

<br>

#### <div id='1-2-4'/> 1.2.4. Secret 수정
- Secret 상세에서 Data 목록의 Edit 버튼을 눌러 Value 값을 수정하고 수정 버튼을 클릭하여 수정한다.  
  ![IMG_6_2_9]

<br>

#### <div id='1-2-5'/> 1.2.5. Secret 삭제
- Secret 상세에서 삭제 버튼을 클릭할 시 Secret 삭제가 완료된다.  
  ![IMG_6_2_10]

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