### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > Ingress HSTS 적용

<br>

# Ingress HSTS(HTTP Strict Transport Security) 적용

## Table of Contents

1. [문서 개요](#1)<br>
   1.1. [목적](#1.1)<br>
   1.2. [범위](#1.2)

2. [Ingress HSTS 적용](#2)<br>
   2.1. [Ingress Annotations 추가](#2.1)<br>
   2.2. [HTTP Header 확인 ](#2.2)


<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
**HSTS(HTTP Strict Transport Security)** 는 웹 애플리케이션이 보안 전송을 지원하기 위해 TLS만 사용하도록 보장하는 정책이다.
본 문서(Ingress HSTS(HTTP Strict Transport Security) 적용)는 Kubernetes 리소스 Ingress에 HSTS를 적용하여 HTTP를 통해 사이트에 접근하려는 모든 시도를 자동으로 HTTPS 요청으로 변환해야 함을 브라우저에 알리는 방법을 기술한다.

<br>

### <div id='1.2'> 1.2. 범위
컨테이너 플랫폼 포털은 Kubernetes 리소스 Ingress를 통해 각 서비스를 라우팅한다.<br>
> <b>[Ingress NGINX Controller](https://kubernetes.github.io/ingress-nginx/)</b> 사용 (Kubernetes용 Ingress 컨트롤러) <br>


<br>

## <div id='2'> 2. Ingress HSTS 적용

### <div id='2.1'> 2.1. Ingress Annotations 추가

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sample-ingress
  annotations:
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"     # 추가
    nginx.ingress.kubernetes.io/ssl-redirect: "true"           # 추가
    nginx.ingress.kubernetes.io/configuration-snippet: |       # 추가
      more_set_headers "Strict-Transport-Security: max-age=31536000; includeSubDomains; preload";   
spec:
  ingressClassName: nginx


```

#### hsts 항목
> [Nginx Configuration](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#hsts)

* **hsts-max-age** <br>
  브라우저에 해당 도메인의 HSTS 적용 시간(초) 설정

* **hsts-include-subdomains** <br>
  해당 도메인의 하위 도메인까지 HSTS 사용 활성화 여부

* **hsts-preload** <br>
  HSTS 기능에서 사전 로드 속성 활성화 여부

<br>

### <div id='2.2'> 2.2. HTTP Header 확인
> 브라우저 개발자 도구 (F12) 접속 -> [Network] 항목 > [Headers] 항목 > [Response Headers] 값 확인

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```
![image 001]

[image 001]:images/IMG_INGRESS_HSTS.png

<br>

### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > Ingress HSTS 적용