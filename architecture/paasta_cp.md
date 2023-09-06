### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Architecture](../README.md) > K-PaaS CP

## 목적
본 문서는 K-PaaS Container Platform (CP)의 Architecture를 제공한다.
<br><br>

## 시스템 구성도
<img src="https://user-images.githubusercontent.com/33216551/209304557-bda8531f-19de-4e35-989d-05e4053efe6c.png" width="700" height="150" />


| 구분  | 인스턴스 수| 스펙 |
|-------|----|-----|
| master | 1 or 3 | 2vCPU / 4GB RAM |
| worker | N | 2vCPU / 4GB RAM |
| nfs-server | N | 1vCPU / 2GB RAM / 100GB 추가 디스크 |


## 설명
K-PaaS CP는 컨테이너 오케스트레이션을 지원하는 Kubernetes를 이용해 컨테이너기반으로 개발 및 배포를 관리하는 플랫폼이다. 
K-PaaS CP를 사용하면 컨테이너 기반으로 어플리케이션을 더 빠르고 쉽게 구축, 테스트 배포 및 확장할 수 있다.


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Architecture](../README.md) > K-PaaS CP
