# K-PaaS 컨테이너 플랫폼

## Related Repositories

<table>
  <tr>
    <td colspan=2 align=center>플랫폼</td>
    <td colspan=2 align=center><a href="https://github.com/K-PaaS/cp-deployment">🚩 컨테이너 플랫폼</a></td>
    <td colspan=2 align=center><a href="https://github.com/K-PaaS/sidecar-deployment">사이드카</a></td>
    <td colspan=2 align=center><a href="https://github.com/K-PaaS/ap-deployment">어플리케이션 플랫폼</a></td>
  </tr>
  <tr>
    <td colspan=2 align=center>포털</td>
    <td colspan=2 align=center><a href="https://github.com/K-PaaS/cp-portal-release">CP 포털</a></td>
    <td colspan=2 align=center>-</td>
    <td colspan=2 align=center><a href="https://github.com/K-PaaS/portal-deployment">AP 포털</a></td>
  </tr>
  <tr align=center>
    <td colspan=2 rowspan=9>Component<br>/ 서비스</td>
    <td colspan=2><a href="https://github.com/K-PaaS/cp-portal-common-api">Common API</a></td>
    <td colspan=2>-</td>
    <td colspan=2><a href="https://github.com/K-PaaS/ap-mongodb-shard-release">MongoDB</a></td>
  </tr>
  <tr align=center>
    <td colspan=2><a href="https://github.com/K-PaaS/cp-metrics-api">Metric API</a></td>
    <td colspan=2>  </td>
    <td colspan=2><a href="https://github.com/K-PaaS/ap-mysql-release">MySQL</a></td>
  </tr>
  <tr align=center>
    <td colspan=2><a href="https://github.com/K-PaaS/cp-portal-api">Portal API</a></td>
    <td colspan=2>  </td>
    <td colspan=2><a href="https://github.com/K-PaaS/ap-pipeline-release">Pipeline</a></td>
  </tr>
  <tr align=center>
    <td colspan=2><a href="https://github.com/K-PaaS/cp-portal-ui">Portal UI</a></td>
    <td colspan=2>  </td>
    <td colspan=2><a href="https://github.com/K-PaaS/ap-rabbitmq-release">RabbintMQ</a></td>
  </tr>
  <tr align=center>
    <td colspan=2><a href="https://github.com/K-PaaS/cp-portal-service-broker">Service Broker</a></td>
    <td colspan=2>  </td>
    <td colspan=2><a href="https://github.com/K-PaaS/ap-on-demand-redis-release">Redis</a></td>
  </tr>
  <tr align=center>
    <td colspan=2><a href="https://github.com/K-PaaS/cp-metrics-api">Terraman API</a></td>
    <td colspan=2>  </td>
    <td colspan=2><a href="https://github.com/K-PaaS/ap-source-control-release">SoureceControl</a></td>
  </tr>
</table>
<i>🚩 You are here.</i>

## Notice
#### 릴리즈의 경로가 https://nextcloud.paas-ta.org/ 에서 https://nextcloud.k-paas.org/ 로 변경되었습니다

<br>

## 소개
네이티브 쿠버네티스 설치 가이드(Kubespray 설치, KubeEdge 설치) 및 쿠버네티스에 컨테이너 플랫폼을 배포하여 사용할 수 있는 방법에 대한 다양한 설치 방법에 대한 가이드 및 활용 가이드를 다루고 있다.

<br>

## Install

### 단독형 배포

#### Single Cloud
- 클러스터 설치
  + [클러스터 설치 가이드](install-guide/standalone/cp-cluster-install-single.md)
- 포털 설치
  + [포털 설치 가이드](install-guide/portal/cp-portal-standalone-guide.md)
  + [설치 및 배포 파일](https://github.com/K-PaaS/cp-helm-chart)
  + [릴리즈 파일](https://github.com/K-PaaS/cp-portal-release)
- 서비스 설치
  + [Pipeline 설치 가이드](install-guide/pipeline/cp-pipeline-standalone-guide.md)
  + [SourceControl 설치 가이드](install-guide/source-control/cp-source-control-standalone-guide.md)
- Kubeflow 튜토리얼
  + [Kubeflow 파이프라인 튜토리얼 가이드](install-guide/standalone/cp-kubeflow-sample-guide.md)

#### Multi Cloud
- 클러스터 설치
  + [클러스터 설치 가이드](install-guide/standalone/cp-cluster-install-multi.md)
- 포털 설치
  + [포털 설치 가이드](install-guide/portal/cp-portal-standalone-guide-mc.md)
  + [설치 및 배포 파일](https://github.com/K-PaaS/cp-helm-chart)
  + [릴리즈 파일](https://github.com/K-PaaS/cp-portal-release)
- 컨테이너플랫폼 Kubernetes 활용
  + [Linkerd 설치 가이드](install-guide/multicluster/cp-linkerd-install.md)
  + [Karmada 설치 가이드](install-guide/multicluster/cp-karmada-install.md)
- CSP Kubernetes Service 활용
  + [istio 설치 및 샘플](install-guide/csp/container-platform-csp-istio-guide.md)
  + [linkerd 설치 및 샘플](install-guide/csp/container-platform-csp-linkerd-guide.md)

### 서비스형 배포 

#### Single Cloud

- 클러스터 설치
  + [클러스터 설치 가이드](install-guide/standalone/cp-cluster-install.md)
- 포털 설치
  + [포털 설치 가이드](install-guide/portal/cp-portal-service-guide.md)
  + [설치 및 배포 파일](https://github.com/K-PaaS/cp-helm-chart/tree/master)
  + [릴리즈 파일](https://github.com/K-PaaS/cp-portal-release/tree/master)
- 서비스 설치
  + [Pipeline 설치 가이드](install-guide/pipeline/cp-pipeline-service-guide.md)
  + [SourceControl 설치 가이드](install-guide/source-control/cp-source-control-service-guide.md)

#### Multi Cloud

- 클러스터 설치
  + [클러스터 설치 가이드](install-guide/standalone/cp-cluster-install-multi.md)
- 포털 설치
  + [포털 설치 가이드](install-guide/portal/cp-portal-service-guide-mc.md)
- 서비스 매시
  + [Linkerd 설치 가이드](install-guide/multicluster/cp-linkerd-install.md)
- 페더레이션
  + [Karmada 설치 가이드](install-guide/multicluster/cp-karmada-install.md)

### Edge 배포
- Edge 설치
  + [Edge 설치 가이드](install-guide/edge/cp-edge-install.md)
- 포털 설치
  + [포털 설치 가이드](install-guide/portal/cp-portal-standalone-guide.md)
  + [설치 및 배포 파일](https://github.com/K-PaaS/cp-helm-chart/tree/master)
  + [릴리즈 파일](https://github.com/K-PaaS/cp-portal-release/tree/master)
- 서비스 설치
  + [Pipeline 설치 가이드](install-guide/pipeline/cp-pipeline-standalone-guide.md)
  + [SourceControl 설치 가이드](install-guide/source-control/cp-source-control-standalone-guide.md)
- 샘플 모델
  + [웹 카운팅 / 실시간 온도수집](install-guide/edge/cp-edge-sample-guide.md)


<br>

## Use

### 포털 이용 가이드
  + [포털 사용 가이드](use-guide/portal/container-platform-portal-guide.md)
  + [Terraman IaC 스크립트 가이드](check-guide/cp-terraman-check-index-guide.md)
  
### 서비스 이용 가이드
- Pipeline 서비스
  + [Pipeline 서비스 사용 가이드](use-guide/pipeline/cp-pipeline-use-guide.md)
- Source Control 서비스
  + [Source Control 서비스 사용 가이드](use-guide/source-control/cp-source-control-use-guide.md)


<br>

## Project

### 포털 프로젝트 
- [cp-portal-api](https://github.com/K-PaaS/cp-portal-api)
- [cp-portal-common-api](https://github.com/K-PaaS/cp-portal-common-api)
- [cp-portal-ui](https://github.com/K-PaaS/cp-portal-ui)
- [cp-portal-service-broker](https://github.com/K-PaaS/cp-portal-service-broker)
- [cp-metrics-api](https://github.com/K-PaaS/cp-metrics-api)
- [cp-terraman](https://github.com/K-PaaS/cp-terraman)

<br>

## Contributors ✨

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):
<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="https://github.com/jinyung0101java2"><img src="https://avatars.githubusercontent.com/u/67574725?v=4?s=100" width="100px;" alt=""/><br /><sub><b>JinYoung Jang</b></sub></a><br /><a href="https://github.com/PaaS-TA/paas-ta-container-platform/commits?author=jinyung0101java2" title="Code">💻</a> <a href="https://github.com/PaaS-TA/paas-ta-container-platform/pulls?q=is&Apr+reviewed-by&jinyung0101java2" title="Reviewed Pull Requests">👀</a></td>
    <td align="center"><a href="https://github.com/hoon77"><img src="https://avatars.githubusercontent.com/u/33216551?v=4?s=100" width="100px;" alt=""/><br /><sub><b>JiHoon Kang</b></sub></a><br /><a href="https://github.com/PaaS-TA/paas-ta-container-platform/commits?author=hoon77" title="Code">💻</a> <a href="https://github.com/PaaS-TA/paas-ta-container-platform/pulls?q=is&Apr+reviewed-by&hoon77" title="Reviewed Pull Requests">👀</a></td>
    <td align="center"><a href="https://github.com/suslmk-lee"><img src="https://avatars.githubusercontent.com/u/67575226?v=4?s=100" width="100px;" alt=""/><br /><sub><b>suslmk</b></sub></a><br /><a href="#maintenance-suslmk" title="Maintenance">🚧</a></td>
    <td align="center"><a href="https://github.com/dev-taewoo"><img src="https://avatars.githubusercontent.com/u/67407365?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Taewoo Kim</b></sub></a><br /><a href="https://github.com/PaaS-TA/paas-ta-container-platform/commits?author=dev-taewoo" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/rexx4314"><img src="https://avatars.githubusercontent.com/u/26153262?v=4?s=100" width="100px;" alt=""/><br /><sub><b>rexx4314</b></sub></a><br /><a href="#ideas-rexx4314" title="Ideas, Planning, & Feedback">🤔</a></td>
    <td align="center"><a href="https://github.com/opdc-minsu"><img src="https://avatars.githubusercontent.com/u/67140002?v=4?s=100" width="100px;" alt=""/><br /><sub><b>MinSu Kang</b></sub></a><br /><a href="https://github.com/PaaS-TA/paas-ta-container-platform/issues?q=author&opdc-minsu" title="Bug reports">🐛</a></td>
    <td align="center"><a href="https://github.com/jhuhm135"><img src="https://avatars.githubusercontent.com/u/70005316?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Juhyun Um</b></sub></a><br /><a href="#ideas-jhuhm135" title="Ideas, Planning, & Feedback">🤔</a></td>
  </tr>
  <tr>
    <td align="center"><a href="https://github.com/kyuminhan"><img src="https://avatars.githubusercontent.com/u/80228983?v=4?s=100" width="100px;" alt=""/><br /><sub><b>KyuMin Han</b></sub></a><br /><a href="#ideas-kyuminhan" title="Ideas, Planning, & Feedback">🤔</a></td>
  </tr>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->
 
<br>

## 라이선스
[Apache-2.0 License](http://www.apache.org/licenses/LICENSE-2.0)를 사용한다. 
