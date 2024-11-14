# 표준 컨테이너 이미지 허브 구축 교육
 본 문서는 표준 컨테이너 이미지 구축에 대한 교육을 위해 K-PaaS에서 제공하는 기능을 중심으로 설명합니다.


 ## K-PaaS 표준모델을 통한 컨테이너 이미지 관리 방안
  - K-PaaS는 Kubernetes를 기반의 PaaS 플랫폼으로, 다음과 같은 기술 스택을 사용하여 안정적이고 효율적인 PaaS 환경 및 컨테이너 이미지 환경을 제공한다.
  
|역할|기능도구|
|:------|:---|
|컨테이너 오케스트레이션|Kubernetes|
|컨테이너 런타임|CRI-O|
|컨테이너 도구|Podman|
|이미지 레지스트리|Harbor|
|이미지 보안검사|Trivy|

### 컨테이너 런타임

#### 1. CRI-O 

Kubernetes의 Container Runtime Interface(CRI)를 구현한 경량 컨테이너 런타임으로, CRI-O는 Kubernetes와 통합되어 운영되며, 아래와 같은 장점을 제공

- 경량화 : 불필요한 기능을 제거하여 경량화된 런타임 환경 제공
- 보안 : 최소 권한 원칙을 적용하여 보안 취약점 최소화 
- Kubernetes 최적화 : Kubernetes의 CRI 표준을 완벽하게 구현
#### 2. containerd

Docker의 핵심 컨테이너 런타임 컴포넌트로 시작된 containerd는 현재 Kubernetes와 독립적으로 운영 가능한 컨테이너 런타임으로 발전. CRI 표준을 준수하여 Kubernetes와의 호환성을 제공
- 모듈화 : 모듈형 아키텍쳐로, 다양한 컨테이너 기술과 쉽게 통합
- 안정성 : 널리 사용되며 검증된 안정성을 제공
- 크로스 플랫폼 : 리눅스, 윈도우 등 다양한 플랫폼에서 운영 가능

### 컨테이너 도구
#### 1. Podman

Podman은 컨테이너를 관리하기 위한 도구로, Docker와 유사한 인터페이스를 제공하나 시스템 데몬을 필요로 하지 않음
- No데몬 구조: Docker와 달리 시스템 데몬 없이 개별 컨테이너를 독립적으로 실행/관리
- Rootless 모드: 비루트 사용자로 컨테이너를 실행하여 보안성 높임.
- Docker 호환성: Docker CLI와 유사한 명령어를 사용하여 기존 Docker사용자들이 쉽게 적응
#### 2. crictl

주로 Kubernetes 환경에서 containerd와 같은 CRI(컨테이너 런타임 인터페이스) 호환 컨테이너 런타임을 관리하기 위해 사용
- `crictl ps`, `crictl logs`, `crictl exec` 등을 통해 Kubernetes에서 생성된 파드와 컨테이너를 확인하고 디버깅
- Kubernetes 클러스터에서 컨테이너를 관리하는 데 최적화되어 있으며, Kubernetes와 직접 연동하여 컨테이너를 관리
#### 3. ctr
containerd의 기본 CLI 도구로, containerd와 직접 상호작용
- `ctr containers`, `ctr images`, `ctr tasks` 등을 통해 컨테이너와 이미지 관리 및 기타 containerd 작업을 수행
- 디버깅이나 로우레벨(containerd의 내부 작업)에 대한 제어가 필요할 때 주로 사용. Kubernetes 환경에서 쉽게 사용할 수는 있지만 Kubernetes와 연동되어 있지 않아, 직접 Kubernetes CRI와 호환되지는 않음
#### 4. nerdctl
containerd를 Docker와 유사한 방식으로 사용자 친화적인 Docker 스타일의 CLI를 제공하여 관리하는 도구
- `nerdctl run`, `nerdctl build`, `nerdctl pull` 등을 통해 Docker와 같은 방식으로 컨테이너 실행, 이미지 관리 등을 수행할 수 있습니다. Compose 파일을 통한 멀티 컨테이너 관리도 가능
- Docker CLI와 호환되는 명령어 체계를 가지고 있어 Docker 사용자들이 containerd로 쉽게 전환할 수 있습니다. Kubernetes 없이도 사용할 수 있으며, 로컬 개발 환경에서 특히 유용
- Rootless 모드: 비루트 사용자로 컨테이너를 실행하여 보안성 높임.

![image](https://github.com/user-attachments/assets/9c4f5717-3188-4f57-89a3-e8c1563f347f)

※ Docker와 Podman은 모두 OCI(Open Container Initiative) 표준을 준수하기에 Docker로 만들어진 이미지는 Podman에서도 호환되어 사용 가능

### 컨테이너 이미지 관리
#### 1. Harbor의 보안 관리 기능
컨테이너 이미지 레지스트리로, 다음과 같은 보안 및 관리 기능을 제공함
 - 이미지 서명: 컨테이너 이미지에 서명하여 신뢰할 수 있는 소스에서 이미지 배포
 - 취약점 스캔: Trivy와 통합되어 저장된 이미지에 대해 자동으로 취약점 스캔 수행
 - 액세스 제어: 역할 기반 액세스제어(RBAC)를 통해 사용자의 접근 관리
#### 2. Trivy를 통한 취약점 스캔
컨테이너 이미지 취약점을 스캔하는 보안 도구로써, 다음과 같은 장점이 있음
 - 광범위한 취약점 데이터베이스: DB화된 데이터를 활용하여 최신 보안 취약점을 신속하게 탐지
 - 통합스캔: Podman과 통합되어 이미지 빌드과정에서 자동 스캔
 - 사용 편의성: CLI를 지원하여 여러 Pipeline 스크립트 상에서 쉽게 적용이 가능


### Dockerfile Best Practice
#### 1. 수명이 짧은 컨테이너 만들기

`Dockerfile`에 의해 정의된 이미지는 가능한 수명이 짧은 컨테이너를 생성해야 한다. 여기서 수명이 짧다 (ephemeral) 라는 것의 의미는, 컨테이너가 멈추고, 삭제되고 그리고 다시 빌드되고 재구축 되는 일련의 과정이 최소한의 구성과 설정으로 이루어져야 한다는 뜻이다.

#### 2. Build context에 대한 이해 

`docker build` 명령어를 실행했을 때, 현재 작업 디렉토리를 build context (이하 빌드 컨텍스트)라고 한다. 기본적으로 Dockerfile은 여기에 위치하는데, `-f` 플래그로 다른 곳에 위치한 파일도 지정할 수 있다. `Dockerfile`의 위치와 관계 없이, 현재 디렉토리에 있는 파일 및 디렉토리 내부의 재귀적으로 존재하는 모든 내용은 빌드 컨텍스트로 도커 데몬에 전송된다.

이미지를 만드는데 필요하지 않은 파일을 포함시키면 빌드 컨텍스트가 커지고, 이미지 크기도 커진다. 이미지 크기가 커지면 빌드에 걸리는 시간, push pull에 소요되는 시간, 컨테이너 런타임 크기 등 모든 것이 늘어난다. 이 빌드 컨텍스트의 크기를 보려면 `Dockerfile`을 작성할 때 다음과 같은 메시지를 확인해보자.
```sh
Sending build context to Docker daemon  187.8MB
```

#### 3. `stdin`을 활용한 `Dockerfile` pipe
도커는 원격 또는 리모트 빌드 컨텍스트를 `stdin` 명령어를 통해 이미지를 만들 수 있다. `Dockerfile`을 `stdin` 명령어로 파이핑 하는 것은 디스크에 `Dockerfile`을 쓰지 않고 일회성으로 일회성으로 빌드하거나, `Dockerfile`이 있지만 이후에 삭제될 수도 있는 상황에서 유용하다.

```sh
echo -e 'FROM busybox\nRUN echo "hello world"' | docker build -
```

또는

```sh
docker build -<<EOF
FROM busybox
RUN echo "hello world"
EOF
```

#### 4. `.dockerignore`로 파일 제외하기

원본 소스 저장소를 건들지 않고 빌드와 관련 없는 파일을 제거하기 위해서는 `.dockerignore` 파일을 사용하면 된다. 이 파일은 `.gitignore` 파일과 유사한 패턴을 지원한다.

#### 5. 멀티 스테이지 빌드 사용하기 

멀티 스테이지 빌드를 사용하면, 중간 레이어와 파일 수를 줄이는데 힘쓰지 않아도 최종 이미지 크기를 줄일 수 있다.

빌드 프로세스의 마지막 단계에서 실제로 사용되는 이미지가 작성되므로, 빌드 캐시를 활용하여 이미지 레이어를 최소화 할 수 있다.

예를 들어, 빌드에 여러 개의 레이어가 포함되어 있는 경우, 변경이 별로 없는 레이어 (빌드 캐시를 적극 활용할 수 있는 레이어)에서 자주 변경이 일어나는 레이어로 순서를 지정할 수 있다.

- 애플리케이션 빌드를 위해 필요한 툴 설치
- 라이브러리 의존성 설치 또는 업데이트
- 애플리케이션 생성

#### 6. 불필요한 패키지를 설치하지 않기
복잡성, 의존성, 파일크기, 빌드시간을 줄이려면 '있으면 좋다' 라는 이유만으로 불필요한 파일이나 패키지를 추가하는 것은 좋지 않다. 예를 들어 데이터 베이스 이미지에 텍스트 에디터는 필요 없다.

#### 7. 애플리케이션 디커플링
각 컨테이너에는 하나의 관심사만 존재해야 한다. 애플리케이션을 여러 컨테이너로 분리하면 수평 확장이 용이해지고, 컨테이너를 재사용하기 쉬워진다. 예를 들어, 웹 애플리케이션은 웹, 데이터 베이스, 인메모리 캐시 등으로 분리하여 관리할 수 있다.

각 컨테이너를 하나의 프로세스로 제한하는 것은 좋은 규칙이지만, 쉽게 적용할 수는 없다. 예를 들어 컨테이너 들만 프로세스를 생성할 수 있는 것은 아니고, 일부 프로그램 또한 프로세스를 자체적으로 생성할 수 도 있다.

컨테이너를 가능한 클린하고 모듈식으로 유지하기 위해 최선의 판단을 내리자. 컨테이너가 서로 종속적이라면, Docker Container Network를 통해 컨테이너 끼리 통신하도록 할 수 있다.

#### 8. 레이어 수를 최소화 하기

옛날 버전의 도커에서는, 이미지에서 레이어 수를 최소화 하여 레이어 성능을 보장하는 것이 중요했다. 이러한 제한을 줄이기 위해 다음과 같은 기능이 추가되었다.

 - RUN COPY ADD 만 레이어를 생성한다. 다른 명령어는 임시로 중간 이미지를 생성하며, 빌드 사이즈에 영향을 미치지 않는다.
 - 가능하다면, 멀티 스테이지 빌드를 사용하고 필요한 아티팩트만 마지막 이미지에 복사하는 것이 좋다. 이렇게 하면 최종 이미지의 크기를 늘리지 않고도 중간 빌드 단계에 각종 도구와 디버그 정보를 포함 시킬 수 있다.

 #### 9. 여러줄 인수를 정렬

가능하다면, 여러 줄 인수를 알파벳순서로 정렬하는 것이 좋다. 이렇게 하면 패키지 중복을 방지하고 목록을 쉽게 업데이트 할 수 있다. 또한 PR 검토 또한 용이해진다. \ 앞에 공백을 추가하는 것도 도움이 된다.
```sh
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion \
  && rm -rf /var/lib/apt/lists/*
```

출처 : https://docs.docker.com/build/building/best-practices/


### 컨테이너 이미지 생성
#### 1. 애플리케이션의 요구 사항에 맞게 Dockerfile 작성
샘플 link : https://github.com/suslmk-lee/zim-solar-inverter-sp-api
```sh
# 1. 빌드 스테이지: Go 애플리케이션을 빌드하기 위한 단계
FROM golang:1.22-alpine AS builder

# 2. 환경 변수 설정: 모듈 모드와 빌드 환경 설정
# - GO111MODULE=on: Go 모듈 사용
# - CGO_ENABLED=0: CGO 비활성화 (모든 패키지를 순수 Go 코드로 빌드)
# - GOOS=linux: 리눅스용으로 빌드
# - GOARCH=amd64: 64비트 아키텍처용으로 빌드
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# 3. 작업 디렉터리 설정: 소스 코드 빌드를 위한 작업 디렉터리
WORKDIR /build

# 4. 모듈 파일과 소스 코드 복사
# - go.mod와 go.sum을 먼저 복사하여 모듈 의존성을 캐시할 수 있도록 함
COPY go.mod go.sum ./
# - go mod download로 의존성 파일을 먼저 다운로드하고, 추가적인 변경이 없는 경우 캐시를 사용
RUN go mod download

# 5. 나머지 소스 코드 복사
COPY main.go ./
COPY config ./config
COPY data ./data
COPY handler ./handler

# 6. 빌드 의존성 최적화
RUN go mod tidy

# 7. 애플리케이션 빌드
RUN go build -o main .

# 8. 최종 스테이지: scratch 이미지를 사용하여 최종 컨테이너 크기 최소화
FROM scratch

# 9. 빌드된 애플리케이션 복사
# - scratch 이미지는 완전한 빈 이미지이므로, 필요한 파일만 복사
COPY --from=builder /build/main /main

# 10. 환경 변수 설정: 프로덕션 모드와 허용된 오리진 설정
ENV PROFILE=prod \
    ALLOWED_ORIGINS="http://localhost:3000"

# 11. 애플리케이션 시작 명령어
ENTRYPOINT ["/main"]

```

#### 2. Podman을 이용한 이미지 빌드
```sh
podman build -t container-platform-image:1.0 .
```
  
#### 3. 이미지 태그 및 푸시: 빌드된 이미지를 Harbor 레지스트리에 태그하고 푸시
```sh
podman tag container-platform-image:1.0 harbor.nip.io/cp-platform/go-sample:1.0
podmna push harbor.nip.io/cp-platform/go-sample:1.0
```

#### 4. 이미지 보안 스캔: Harbor에 푸시된 이미지를 Trivy를 통해 자동으로 스캔 
![image](https://github.com/user-attachments/assets/666bef59-a935-4393-8743-e51cdda4c258)
### 다양한 CI/CD를 활용한 이미지 생성 및 배포
- PaaS 구축 사업에서 애플리케이션에 필요한 컨테이너 이미지를 효율적으로 생성하고 관리하기 위해 다음과 같은 템플릿을 구현할 수 있다.  
- CI/CD 파이프라인 템플릿: GitHub Action, GitLab CI, Jenkins 등을 이용하여 Podman을 사용한 이미지 빌드 및 푸시, 이미지 보안 스캔을 자동화 할 수 있는 파이프라인 작성

> Github Action을 통한 배포 파이프라인 스크립트 템플릿 예제
```sh
name: Build and Deploy to Kubernetes

on:
  push:
    branches:
      - master

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    env:
      IMAGE_NAME: "zim-solar-inverter-sp-api"
      REGISTRY: "44ce789b-kr1-registry.container.nhncloud.com/container-platform-registry"
      NAMESPACE: "edge-test"
      DEPLOYMENT_NAME: "inverter-sp-deployment"
      CLUSTER_NAME: "dev-cluster"
      KUBECONFIG_DATA: ${{ secrets.KUBECONFIG_DATA }}

    steps:
      - name: Checkout source code
        uses: actions/checkout@v2

      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.22

      - name: Log in to Container Registry
        env:
          REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
        run: echo "${REGISTRY_PASSWORD}" | docker login $REGISTRY -u "${REGISTRY_USERNAME}" --password-stdin

      - name: Build and Push Docker Image
        env:
          GITHUB_SHA: ${{ github.sha }}
        run: |
          docker build -t $REGISTRY/$IMAGE_NAME:latest .
          docker push $REGISTRY/$IMAGE_NAME:latest

      - name: Set up Kubernetes
        uses: azure/k8s-set-context@v2
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBECONFIG_DATA }}

      - name: Apply Kubernetes Deployment
        run: |
          kubectl apply -f deployment/k8s.yaml -n $NAMESPACE
```