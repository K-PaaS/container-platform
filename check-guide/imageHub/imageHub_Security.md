
## Trivy를 활용한 컨테이너 이미지 보안

### 1. 컨테이너 이미지 개요

컨테이너 이미지는 애플리케이션 실행에 필요한 모든 요소 (코드, 런타임, 시스템 도구, 시스템 라이브러리, 설정)를 포함하는 실행 가능한 패키지입니다. 컨테이너 이미지는 이식성이 뛰어나고 일관된 실행 환경을 제공하여 소프트웨어 개발 및 배포에 널리 사용됩니다.

### 2. 컨테이너 이미지 보안의 중요성

컨테이너 이미지는 소프트웨어 공급망의 핵심 구성 요소가 되었으며, 이로 인해 보안 위협에 대한 주요 공격 대상이 되었습니다. 컨테이너 이미지에 존재하는 취약점은 시스템 손상, 데이터 유출, 서비스 중단 등 심각한 보안 문제로 이어질 수 있습니다. 따라서 컨테이너 이미지 보안은 애플리케이션 및 시스템의 안전성과 신뢰성을 보장하기 위한 필수적인 요소입니다.

### 3. Trivy: 컨테이너 이미지 취약점 스캐너

Trivy는 CoreOS에서 개발한 오픈소스 취약점 스캐너입니다. 컨테이너 이미지, 파일 시스템, Git 저장소 등을 스캔하여 알려진 취약점을 식별하고 보고합니다. Trivy는 사용이 간편하고 다양한 취약점 데이터베이스를 지원하며, CI/CD 파이프라인에 통합하여 자동화된 보안 검사를 수행할 수 있습니다.

### 4. Trivy를 이용한 이미지 보안 점검 및 조치

#### 4.1. Trivy 설치

다음 명령어를 터미널에서 실행하여 Trivy를 설치합니다.

Bash

```
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
```

#### 4.2. 이미지 스캔

`trivy image [이미지 이름]` 명령어를 사용하여 이미지를 스캔합니다. 예를 들어, `zim-iot-data-api` 저장소의 이미지를 스캔하려면 다음 명령어를 순차적으로 실행합니다.

Bash

```
git clone https://github.com/suslmk-lee/zim-iot-data-api.git
cd zim-iot-data-api
docker build -t zim-iot-data-api .
trivy image zim-iot-data-api
```

![image](https://github.com/user-attachments/assets/1ca7368e-8432-49c1-be16-50bf2c151f42)

#### 4.3. 결과 분석 및 조치

Trivy는 스캔 결과를 통해 취약점의 심각도, 패키지 이름, 취약점 ID (CVE), 설명 등을 제공합니다.

- **심각도:** CRITICAL, HIGH, MEDIUM, LOW 네 단계로 구분됩니다.
- **패키지 이름:** 취약점이 발견된 패키지의 이름입니다.
- **취약점 ID:** CVE는 공통 취약점 및 노출(Common Vulnerabilities and Exposures)의 약자로, 취약점에 대한 표준 식별자입니다.
- **설명:** 취약점에 대한 간략한 설명과 해결 방법 등을 제공합니다.

스캔 결과를 분석하여 취약점을 식별하고, 다음과 같은 조치를 취해야 합니다.

- **베이스 이미지 업데이트:** 가능하다면 최신 버전의 베이스 이미지를 사용합니다.
- **패키지 업데이트:** 취약한 패키지를 최신 버전으로 업데이트합니다.
- **Dockerfile 수정:** 불필요한 패키지를 제거하고, 최소 권한 원칙을 적용하며, 보안 설정을 강화합니다.
- **보안 패치 적용:** OS 또는 애플리케이션 제조사에서 제공하는 보안 패치를 적용합니다.

#### 4.4. 재스캔 및 검증

취약점을 해결한 후 이미지를 다시 스캔하여 문제가 해결되었는지 확인합니다.

### 5. CI/CD 파이프라인에 Trivy 통합

CI/CD 파이프라인에 Trivy를 통합하면 이미지 빌드 시 자동으로 보안 검사를 수행하고 보고서를 생성할 수 있습니다.

#### 5.1. GitHub Actions workflow 파일 생성

`.github/workflows/trivy-scan.yml` 파일을 생성하고 다음 내용을 입력합니다.

YAML

```
name: Trivy Vulnerability Scan

on:
  push:
    branches:
      - main  # 스캔을 실행할 브랜치 지정

jobs:
  trivy_scan:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Build the Docker image
      run: docker build -t zim-iot-data-api .

    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'zim-iot-data-api:latest'  # 스캔할 이미지 이름 지정
        format: 'template'
        template: '.github/workflows/html.tpl'  # 템플릿 파일 경로 지정
        output: 'trivy-result.html'
        severity: 'CRITICAL,HIGH,MEDIUM'

    - name: Upload Trivy scan report
      uses: actions/upload-artifact@v3
      with:
        name: trivy-scan-report
        path: trivy-result.html
```

#### 5.2. HTML 템플릿 파일 생성

`.github/workflows/html.tpl` 파일을 생성하고 원하는 HTML 템플릿을 작성합니다. 예를 들어, 다음과 같이 간단한 템플릿을 사용할 수 있습니다.

HTML

```
<h1>Trivy Scan Report</h1>
<table>
  <thead>
    <tr>
      <th>Severity</th>
      <th>Package</th>
      <th>Vulnerability ID</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    {{ range .Vulnerabilities }}
    <tr>
      <td>{{ .Severity }}</td>
      <td>{{ .PkgName }}</td>
      <td>{{ .VulnerabilityID }}</td>
      <td>{{ .Description }}</td>
    </tr>
    {{ end }}
  </tbody>
</table>
```

#### 5.3. 코드 push 및 보고서 확인

코드를 GitHub 저장소에 push하면 GitHub Actions가 자동으로 실행되어 Trivy 스캔을 수행하고 HTML 보고서를 생성합니다. Actions 탭에서 스캔 작업이 완료되면 "Artifacts" 섹션에서 `trivy-scan-report`를 다운로드하여 HTML 보고서를 확인할 수 있습니다.
![image](https://github.com/user-attachments/assets/f78ee944-96bf-4e11-9249-01ccdafafd4b)

### 6. 참고 자료

- Trivy 공식 문서: [https://github.com/aquasecurity/trivy](https://www.google.com/url?sa=E&source=gmail&q=https://github.com/aquasecurity/trivy)
- CVE (Common Vulnerabilities and Exposures): [https://cve.mitre.org/](https://www.google.com/url?sa=E&source=gmail&q=https://cve.mitre.org/)
- GitHub Actions 공식 문서: [https://docs.github.com/en/actions](https://www.google.com/url?sa=E&source=gmail&q=https://docs.github.com/en/actions)
- 테스트완료한 GitHub Repo: https://github.com/suslmk-lee/zim-kafka-consum
