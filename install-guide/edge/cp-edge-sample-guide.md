### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > Edge 샘플 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [시스템 구성도](#1.3)  
  1.4. [참고자료](#1.4)  

2. [KubeEdge Sample 배포](#2)  
  2.1. [Web 기반 KubeEdge Counter Sample](#2.1)  
  2.2. [Edge 기반 KubeEdge 온도 수집 Sample](#2.2)  

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (KubeEdge Sample 배포 가이드) 는 Raspberry Pi 및 온습도 센서 (DHT11)을 이용하여 실제 Edge 환경과 Device를 구성하여 Sample을 배포 및 확인하는 방법을 기술하였다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 KubeEdge 실제 Edge 환경을 구성 및 검증하기 위한 KubeEdge Sample 배포 가이드를 기준으로 작성하였다.

<br>

### <div id='1.3'> 1.3. 시스템 구성도
시스템 구성은 Kubernetes Cluster(Master, Worker)와 Raspberry Pi(Edge), DHT11 센서(Device) 환경으로 구성되어 있다.
Kubespray를 통해 Kubernetes Cluster(Master, Worker)를 설치하고 Kubernetes 환경에 KubeEdge를 설치한다.
총 필요한 VM 환경으로는 **Master VM: 1개, Worker VM: 3개 이상, Raspberry Pi: 1개 이상**이 필요하다.
본 문서는 실제 Edge 환경을 구성하기 위한 Edge 환경 구성 및 검증 내용이다.

![image 001]

<br>

### <div id='1.4'> 1.4. 참고자료
> https://kubeedge.io/en/docs/developer/device_crd/
> https://github.com/kubeedge/examples
> https://github.com/kubeedge/examples/blob/master/kubeedge-counter-demo/README.md
> https://github.com/kubeedge/examples/blob/master/temperature-demo/README.md

<br>

## <div id='2'> 2. KubeEdge Sample 환경 설정
본 가이드에서는 두가지의 Sample을 이용하여 Cloud, Edge 간 데이터 통신을 확인한다.
Cloud 환경에 Kubernetes Cluster를 구성하였으며 Raspberry Pi를 이용하여 Edge Node를 추가하였다.
Raspberry Pi에는 온습도 센서 (DHT11)를 연결, 구성하였다.

Container Platform 포털 설치 진행 전 KubeEdge Sample을 배포하려면 별도로 Podman 설치를 진행해야한다.

- Ubuntu 20.04 arm64 Podman 설치
```
$ echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /" | sudo tee etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list

$ curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/Release.key | sudo apt-key add -

$ sudo apt-get update
$ sudo apt-get -y upgrade 
$ sudo apt-get -y install podman
```

<br>

### <div id='2.1'> 2.1. Web 기반 KubeEdge Counter Sample
**Master Node**에서 Web Application을 배포한 후 **Edge Node**에서 Counter Application 배포를 진행한다.

- **Master Node**와 **Edge Node**에서 Sample 배포에 필요한 파일을 다운로드한다.
```
$ wget --content-disposition https://nextcloud.paas-ta.org/index.php/s/acaatJ77zgYsEYb/download

$ tar zxvf kubeedge-sample-v1.4.tar.gz
```

<br>

- **Master Node**에서 Container Image 파일을 Load한다.
```
$ cd ~/kubeedge-sample/kubeedge-counter/amd64

$ sudo podman load -i kubeedge-counter-app.tar
```

<br>

- **Edge Node**에서 Container Image 파일을 Load한다.
```
$ cd ~/kubeedge-sample/kubeedge-counter/arm64

$ sudo podman load -i kubeedge-pi-counter.tar
```

<br>

- **Master Node**에서 DeviceModel, DeviceInstance를 배포한다.
```
$ cd ~/kubeedge-sample/kubeedge-counter/

## DeviceModel 배포
$ kubectl apply -f kubeedge-counter-model.yaml

## DeviceInstance 내 호스트명 변경 ()
$ sed -i "s/{EDGE_NODE_NAME}/{{실제 엣지노드 호스트명}}/g" kubeedge-counter-instance.yaml

ex) sed -i "s/{EDGE_NODE_NAME}/paasta-cp-edge-1/g" kubeedge-counter-instance.yaml

## DeviceInstance 배포
$ kubectl apply -f kubeedge-counter-instance.yaml
```

<br>

- **Master Node**에서 Web Application, Counter Application을 배포한다.
```
$ kubectl apply -f kubeedge-web-controller-app.yaml

$ kubectl apply -f kubeedge-pi-counter-app.yaml
```

<br>

- 브라우저에서 배포된 Web에 접근하여 Counter 기능을 제어한다. Cloud 영역에 배포된 Web을 통해 Edge 영역에 배포된 Counter Application을 제어하여 Counter 값을 얻을 수 있다.
![image 002]

<br>

- **Master Node**에서 Device의 정보를 확인하여 수집중인 Counter 정보를 확인한다. 하단의 value 값 업데이트가 확인된다.
```
$ kubectl get device counter -oyaml -w -n kubeedge
```
```
...
status:
  twins:
  - desired:
      metadata:
        timestamp: "1640"
        type: string
      value: "ON"
    propertyName: status
    reported:
      metadata:
        timestamp: "1640253571427"
        type: string
      value: "99"
```

<br>

- **Edge Node**에서 '$hw/events/device/counter/twin/update' 토픽을 구독하여 전달되는 데이터를 확인한다.
```
## mosquitto_sub 명령어 사용을 위해서는 다음 패키지 설치를 진행한다.
$ sudo apt install mosquitto-clients

$ mosquitto_sub -h 127.0.0.1 -t '$hw/events/device/counter/twin/update' -p 1883
```
```
{"event_id":"","timestamp":0,"twin":{"status":{"actual":{"value":"1643"},"metadata":{"type":"Updated"}}}}
{"event_id":"","timestamp":0,"twin":{"status":{"actual":{"value":"1644"},"metadata":{"type":"Updated"}}}}
{"event_id":"","timestamp":0,"twin":{"status":{"actual":{"value":"1645"},"metadata":{"type":"Updated"}}}}
{"event_id":"","timestamp":0,"twin":{"status":{"actual":{"value":"1646"},"metadata":{"type":"Updated"}}}}
{"event_id":"","timestamp":0,"twin":{"status":{"actual":{"value":"1647"},"metadata":{"type":"Updated"}}}}
```
<br>


### <div id='2.2'> 2.2. Edge 기반 KubeEdge 온도 수집 Sample
**Edge Node (Raspberry Pi)** 에서 GPIO를 이용하여 DHT11 온도센서를 구성하였으며, Temperature Application을 배포하였다.

- **Master Node**와 **Edge Node**에서 Sample 배포에 필요한 파일을 다운로드한다.
```
$ wget --content-disposition https://nextcloud.paas-ta.org/index.php/s/acaatJ77zgYsEYb/download

$ tar zxvf kubeedge-sample-v1.4.tar.gz
```

<br>

- **Edge Node**에서 Container Image 파일을 Load한다.
```
$ cd ~/kubeedge-sample/kubeedge-temperature/arm64

$ sudo podman load -i kubeedge-temperature.tar
```

<br>

- **Master Node**에서 DeviceModel, DeviceInstance를 배포한다.
```
$ cd ~/kubeedge-sample/kubeedge-temperature/

## DeviceModel 배포
$ kubectl apply -f model.yaml

## DeviceInstance 내 호스트명 변경 ()
$ sed -i "s/{EDGE_NODE_NAME}/{{실제 엣지노드 호스트명}}/g" instance.yaml

ex) sed -i "s/{EDGE_NODE_NAME}/paasta-cp-edge-1/g" instance.yaml

## DeviceInstance 배포
$ kubectl apply -f instance.yaml
```

<br>

- **Master Node**에서 Web Application, Counter Application을 배포한다.
```
## deployment 내 nodeSelector 변경
$ vi deployment.yaml

...
nodeSelector:
  kubernetes.io/hostname: {{EDGE_NODE_NAME}} (변경)
...

$ kubectl apply -f deployment.yaml
```

<br>

- **Master Node**에서 Device의 정보를 확인하여 수집중인 온도 정보를 확인한다.
```
$ kubectl get device temperature -oyaml -w -n kubeedge
```
```
...
status:
  twins:
  - desired:
      metadata:
        type: string
      value: ""
    propertyName: temperature-status
    reported:
      metadata:
        timestamp: "1640253571427"
        type: string
      value: "24C"
```

<br>

- **Edge Node**에서 '$hw/events/device/temperature/twin/update' 토픽을 구독하여 전달되는 데이터를 확인한다.
```
## mosquitto_sub 명령어 사용을 위해서는 다음 패키지 설치를 진행한다.
$ sudo apt install mosquitto-clients

$ mosquitto_sub -h 127.0.0.1 -t '$hw/events/device/temperature/twin/update' -p 1883
```
```
{"event_id":"","timestamp":0,"twin":{"temperature-status":{"actual":{"value":"24C"},"metadata":{"type":"Updated"}}}}
{"event_id":"","timestamp":0,"twin":{"temperature-status":{"actual":{"value":"24C"},"metadata":{"type":"Updated"}}}}
{"event_id":"","timestamp":0,"twin":{"temperature-status":{"actual":{"value":"23C"},"metadata":{"type":"Updated"}}}}
{"event_id":"","timestamp":0,"twin":{"temperature-status":{"actual":{"value":"24C"},"metadata":{"type":"Updated"}}}}
{"event_id":"","timestamp":0,"twin":{"temperature-status":{"actual":{"value":"24C"},"metadata":{"type":"Updated"}}}}
```

<br>

[image 001]:images/edge-v1.2.png
[image 002]: images/kubeedge-counter-web.png

### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide/Readme.md) > Edge 샘플 가이드
