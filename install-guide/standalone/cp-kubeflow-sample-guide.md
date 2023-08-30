### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > Kubeflow 파이프라인 튜토리얼 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [시스템 구성도](#1.3)  
  1.4. [참고자료](#1.4)  

2. [Kubeflow 파이프라인 튜토리얼](#2)  
  2.1. [DSL - Control structures 튜토리얼](#2.1)  
  2.2. [Data passing in python components 튜토리얼](#2.2)  

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (Kubeflow 파이프라인 튜토리얼 가이드) 는 Kubeflow Central Dashboard 내 튜토리얼을 이용하여 파이프라인을 실행 및 확인하는 방법을 기술하였다.

<br>

### <div id='1.2'> 1.2. 범위
본 문서는 Kubeflow 파이프라인 튜토리얼을 구성 및 검증하기 위한 Kubeflow 파이프라인 튜토리얼 가이드를 기준으로 작성하였다.

<br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 Kubernetes Cluster(Master, Worker) 환경으로 구성되어 있다.
Kubespray를 통해 Kubernetes Cluster(Master, Worker)를 설치한다.
총 필요한 VM 환경으로는 **Master VM: 1개, Worker VM: 3개 이상**이 필요하다.

![image 001]

<br>

### <div id='1.4'> 1.4. 참고자료
> https://www.kubeflow.org/  
> https://github.com/kubeflow/kubeflow  

<br>

## <div id='2'> 2. Kubeflow 파이프라인 튜토리얼
본 가이드에서는 두가지의 튜토리얼 파이프라인을 이용하여 Kubeflow 파이프라인을 실행 및 확인한다.

- Kubeflow Central Dashboard의 Port 정보를 확인한다.
```
$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                      AGE
istio-ingressgateway   NodePort   10.233.43.143   <none>        15021:31169/TCP,80:30856/TCP,443:32596/TCP,31400:31165/TCP,15443:30988/TCP   25d
```

- Kubeflow Central Dashboard에 로그인한다. 최초 로그인 시 email 정보는 **user@example.com**, 패스워드는 **12341234**로 설정되어있다.

![image 002]

- Kubeflow Central Dashboard에 접속하여 **Pipelines** 메뉴로 이동한다.

<br>

### <div id='2.1'> 2.1. DSL - Control structures 튜토리얼
DSL 제어 구조 튜토리얼로 조건부 실행 및 종료 핸들러를 사용하는 방법을 보여주는 튜토리얼 파이프라인이다.

- Pipeline name 항목에서 "[Tutorial] DSL - Control structures"를 클릭한다.

![image 003]

- 우측 상단의 **Create experiment** 버튼을 클릭한다.

![image 004]

- **Experiment name**을 입력 후 Next 버튼을 클릭한다.

![image 005]

- **Start a run** 화면 하단의 Start 버튼을 클릭한다.

![image 006]

- **Runs** 메뉴에서 **Run name** 항목을 클릭하여 파이프라인 실행 상태를 확인한다.

![image 007]

- 파이프라인 실행 시 Pod 목록을 확인한다.
```
$ kubectl get pods -n kubeflow-user-example-com
NAME                                                                READY   STATUS             RESTARTS   AGE
conditional-execution-pipeline-with-exit-handler-g2gml-1108718281   0/2     Completed          0          8m36s
conditional-execution-pipeline-with-exit-handler-g2gml-2196565435   0/1     Completed          0          5m1s
conditional-execution-pipeline-with-exit-handler-g2gml-2475178693   0/2     Completed          0          5m23s
conditional-execution-pipeline-with-exit-handler-g2gml-585293224    0/2     Completed          0          10m
conditional-execution-pipeline-with-exit-handler-g2gml-893365874    0/2     Completed          0          8m10s
```

<br>


### <div id='2.2'> 2.2. Data passing in python components 튜토리얼
Python 구성 요소 간 데이터를 전달하는 방법을 보여주는 튜토리얼 파이프라인이다.

- Pipeline name 항목에서 "[Tutorial] Data passing in python components"를 클릭한다.

![image 003]

- 우측 상단의 **Create experiment** 버튼을 클릭한다.

![image 008]

- **Experiment name**을 입력 후 Next 버튼을 클릭한다.

![image 005]

- **Start a run** 화면 하단의 Start 버튼을 클릭한다.

![image 009]

- **Runs** 메뉴에서 **Run name** 항목을 클릭하여 파이프라인 실행 상태를 확인한다.

![image 010]

- 파이프라인 실행 시 Pod 목록을 확인한다.
```
$ kubectl get pods -n kubeflow-user-example-com
NAME                                                                READY   STATUS             RESTARTS   AGE
file-passing-pipelines-xnnp6-1960832278                             0/2     Completed          0          20m   10.233.70.252   cp-dev-cluster-3   <none>           <none>
file-passing-pipelines-xnnp6-2194305122                             0/2     Completed          0          19m   10.233.70.207   cp-dev-cluster-3   <none>           <none>
file-passing-pipelines-xnnp6-2419874560                             0/2     Completed          0          20m   10.233.70.250   cp-dev-cluster-3   <none>           <none>
file-passing-pipelines-xnnp6-2436652179                             0/2     Completed          0          19m   10.233.70.242   cp-dev-cluster-3   <none>           <none>
file-passing-pipelines-xnnp6-2520540274                             0/2     Completed          0          19m   10.233.70.219   cp-dev-cluster-3   <none>           <none>
file-passing-pipelines-xnnp6-3053816171                             0/2     Completed          0          20m   10.233.70.241   cp-dev-cluster-3   <none>           <none>
file-passing-pipelines-xnnp6-3667305124                             0/2     Completed          0          20m   10.233.70.253   cp-dev-cluster-3   <none>           <none>
file-passing-pipelines-xnnp6-595339866                              0/2     Completed          0          20m   10.233.70.240   cp-dev-cluster-3   <none>           <none>
```

<br>

[image 001]:images/standalone-v1.2.png

[image 002]:images/kubeflow-login.png
[image 003]:images/kubeflow-pipelines.png

[image 004]:images/kubeflow-pipelines-dsl-001.png
[image 005]:images/kubeflow-pipelines-dsl-002.png
[image 006]:images/kubeflow-pipelines-dsl-003.png
[image 007]:images/kubeflow-pipelines-dsl-004.png

[image 008]:images/kubeflow-pipelines-python-001.png
[image 009]:images/kubeflow-pipelines-python-002.png
[image 010]:images/kubeflow-pipelines-python-003.png


### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > Kubeflow 파이프라인 튜토리얼 가이드
