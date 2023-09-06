### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Architecture](../README.md) > Pipeline

## 목적
본 문서는 K-PaaS Container Platform Pipeline의 Architecture를 제공한다.
<br><br>

## 컨테이너 구성도
<img src="https://user-images.githubusercontent.com/33216551/209299431-af201419-9220-425a-8552-d15e379f8ee7.png" width="730" height="440" />



| 컨테이너명  | 역할 |
|-------|-----|
| Pipeline-API | 파이프라인 제어에 필요한 REST API 제공 |
| Common-API | 서비스 관리에 필요한 메타데이터 제어를 위한 Database API 제공 |
| Pipeline-UI | 파이프라인 서비스 Web UI |
| Inspection -API | 품질 관리 및 결과에 필요한 REST API 제공 |
| Pipeline-Broker | K-PaaS와 배포 파이프라인 서비스 간의 중계 역할 어플리케이션 |
| Inspection-Svr | SonarQube Server, 품질 관리 및 정적분석 서비스 제공 |
| Ci-Server | Jenkins Server 빌드 프로세스 관리 |
| Config-Server | Spring Cloud Config Server 어플리케이션 환경설정 관리 |
| PostgresSQL | 품질 관리 데이터 관리용 Database |



## 설명
K-PaaS 컨테이너플랫폼 파이프라인 서비스는 개발하는 어플리케이션의 빌드, 테스트, 정적분석 및 배포, 파이프라이닝하는 기능을 제공하고, 배포되는 애플리케이션의 환경정보를 관리할 수 있는 UI를 제공한다.   


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Architecture](../README.md) > Pipeline
