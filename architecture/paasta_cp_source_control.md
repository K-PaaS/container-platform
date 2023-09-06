### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Architecture](../README.md) > SourceControl

## 목적
본 문서는 K-PaaS Container Platform SourceControl의 Architecture를 제공한다.
<br><br>

## 컨테이너 구성도
<img src="https://user-images.githubusercontent.com/33216551/209299431-af201419-9220-425a-8552-d15e379f8ee7.png" width="730" height="440" />



| 컨테이너명  | 역할 |
|-------|-----|
| SC-API | 소스 컨트롤 서비스 제어에 필요한 REST API 제공 |
| SC-UI | 소스 컨트롤 서비스 Web UI |
| SC-Broker | K-PaaS와 소스컨트롤 서비스 간의 중계 역할용 어플리케이션 |
| SCM-Server | SCM Server |



## 설명
K-PaaS 컨테이너플랫폼 소스 컨트롤 서비스는 Git과 Svn Repository를 관리할 수 있는 UI를 제공한다.   


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Architecture](../README.md) > SourceControl
