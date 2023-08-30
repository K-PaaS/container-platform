### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > Keycloak TLS 설정 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
    1.1. [목적](#1.1)  

2. [Keycloak TLS 설정](#2)  
    2.1. [TLS 인증서 파일 준비](#2.1)  
    2.2. [Dockerfile 내 인증서 파일 경로 추가](#2.2)   
    2.3. [Keycloak values.yaml 파일 수정](#2.3)   
    2.4. [컨테이너 플랫폼 포털 변수 파일 수정](#2.4)   

3. [(서비스형 배포) 사용자 인증 서비스 구성 변경](#3)  
    3.1. [사용자 인증 구성 변수 값 변경 ](#3.1)  

<br>

## <div id='1'>1. 문서 개요
### <div id='1.1'>1.1. 목적
본 문서(Keycloak TLS 설정 가이드)는 Kubernetes Cluster를 설치하고 컨테이너 플랫폼 포털 배포 시 Keycloak TLS 설정 방법을 기술하였다.
<br><br>

## <div id='2'>2. Keycloak TLS 설정
본 가이드는 컨테이너 플랫폼 포털 배포 전 설정이 진행되어야 한다.
컨테이너 플랫폼 포털 단독형 배포 설치 가이드, 서비스형 배포 설치 가이드 본문의 **[3.2.2. 컨테이너 플랫폼 포털 변수 정의]** 실행 전 작업한다.

### <div id='2.1'>2.1. TLS 인증서 파일 준비
컨테이너 플랫폼 포털 배포 전 TLS 인증서 파일 (ex: tls.key, tls.crt)이 사전에 준비되어야 한다.<br>
- 컨테이너 플랫폼 포털 Deployment 파일 **keycloak_orig** 디렉토리 하위에 위치 필요
- 인증서 파일 명은 **tls.key**, **tls.crt** 로 변경 필요
- 인증서 파일 권한 변경 필요

> `Example`
```
# 인증서 파일 keycloak_orig 디렉토리 하위에 위치
ls ~/workspace/container-platform/paas-ta-container-platform-portal-deployment/keycloak_orig/tls-key
tls.crt  tls.key

# 인증서 파일 권한 변경
chmod ug+r ~/workspace/container-platform/paas-ta-container-platform-portal-deployment/keycloak_orig/tls-key/*
```


<br>
    
### <div id='2.2'>2.2. Dockerfile 내 인증서 파일 경로 추가 
Keycloak Dockerfile 내 TLS 인증서 파일 경로를 추가한다.
```
$ cd ~/workspace/container-platform/paas-ta-container-platform-portal-deployment/keycloak_orig
$ vi Dockerfile
```
    
```
# TLS_FILE_PATH : TLS 인증서 파일이 위치한 Deployment 파일 keycloak_orig 디렉토리 내 인증서 파일 경로
    
...
COPY {TLS_FILE_PATH}/* /etc/x509/https/
...
```
    
> `Example`
```
...
COPY tls-key/* /etc/x509/https/
COPY container-platform/ /opt/jboss/keycloak/themes/container-platform/
...
```    
    
<br>
    
### <div id='2.3'>2.3. Keycloak values.yaml 파일 수정    
Keycloak values.yaml 파일 내 아래 내용을 수정한다.

```
$ cd ~/workspace/container-platform/paas-ta-container-platform-portal-deployment/values_orig
$ vi paas-ta-container-platform-keycloak-values.yaml
```

```
# service.targetPort 값을 8443으로 변경 (https로 접속)

...
service:
  type: {SERVICE_TYPE}
  protocol: {SERVICE_PROTOCOL}
  port: 8080
  https:
    port: 8443
  targetPort: 8443 (수정)
  nodePort: 32710
...
```

<br>
    
### <div id='2.4'>2.4. 컨테이너 플랫폼 포털 변수 파일 수정
컨테이너 플랫폼 포털 변수 파일 내 아래 내용을 수정한다.
```
$ cd ~/workspace/container-platform/paas-ta-container-platform-portal-deployment/script
$ vi container-platform-portal-vars.sh    
```    
```
# KEYCLOAK_URL 값 http -> https 로 변경 
# Domain으로 nip.io를 사용하는 경우 아래와 같이 변경
    
....  
# KEYCLOAK    
KEYCLOAK_URL="https:\/\/${K8S_MASTER_NODE_IP}.nip.io:32710"   # Keycloak url (include http:\/\/, if apply TLS, https:\/\/)
....     
```
<br>
    
위 항목들의 **Keycloak TLS 설정**이 완료되면, 컨테이너 플랫폼 포털 단독형 배포 설치 혹은 서비스형 배포 설치 가이드 본문의 **[3.2.2. 컨테이너 플랫폼 포털 변수 정의]** 부터 진행을 시작한다.
<br>


<br><br> 
    
## <div id='3'>3. (서비스형 배포) 사용자 인증 서비스 구성 변경 
컨테이너 플랫폼 포털 서비스형 배포 시, Keycloak TLS 설정이 적용된 경우 사용자 인증 구성 변수 값 변경이 필요하다.
    
### <div id='3.1'>3.1. 사용자 인증 구성 변수 값 변경 
 UAA 서비스와 Keycloak 서비스 인증 구성 변수 파일 내 **Keycloak URL** 값을 아래와 같이 변경한다.

```
$ cd ~/workspace/container-platform/paas-ta-container-platform-saml-deployment
$ vi container-platform-saml-vars.sh
```    
```
# KEYCLOAK_URL 값 http -> https 로 변경 
# Domain으로 nip.io를 사용하는 경우 아래와 같이 변경   
    
....     
# KEYCLOAK
KEYCLOAK_URL="https://${K8S_MASTER_NODE_IP}.nip.io:32710"   # Keycloak url (include http://, if apply TLS, https://)  
.... 
```
<br>
    
### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > Keycloak TLS 설정 가이드