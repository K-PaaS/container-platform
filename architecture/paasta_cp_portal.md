### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Architecture](../README.md) > Portal

## 목적
본 문서는 K-PaaS Container Platform (CP)의 포털 서비스에 대한 Architecture를 제공한다.
<br><br>

## 컨테이너 구성도
<img src="https://user-images.githubusercontent.com/33216551/209299431-af201419-9220-425a-8552-d15e379f8ee7.png" width="730" height="440" />



| 컨테이너 명  | 역할 |
|-------|----|
| Portal-API | 컨테이너플랫폼 포털 서비스 제어에 필요한 REST API를 제공 |
| Common-API | 컨테이너플랫폼 포털 메타데이터 제어를 위한 Database API를 제공 |
| Web-UI| 컨테이너플랫폼 포털 웹 애플리케이션 |
| Terraman | HCL 스크립트를 통한 VM 생성 및 Kubernetes 클러스터 설치를 REST API로 제공 |
| Metrics-API |  Kubernetes 클러스터 리소스의 메트릭 데이터를 REST API로 제공 |
| Portal-Broker | K-PaaS와 컨테이너플랫폼 포털 서비스 간의 중계 역할용 어플리케이션 |



## 설명
K-PaaS 컨테이너플랫폼 포털 서비스는 배포되는 쿠버네티스 클러스터의 워크로드를 관리하고, 테넌시별로 컨테이너의 배포 및 관리를 제어할 수 있는 UI를 제공한다.


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Architecture](../README.md) > Portal
