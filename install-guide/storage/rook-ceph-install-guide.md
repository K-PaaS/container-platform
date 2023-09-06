### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > Rook Ceph 설치


# Rook Ceph 설치

## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  

2. [Rook Ceph 설치](#2)  
  2.1. [구성](#2.1)  
  2.2. [Prerequisite](#2.2)  
  2.3. [Rook ceph 구성](#2.3)  

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (Kubespray 설치 가이드) 는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에 배포되는 컨테이터 플랫폼을 위해 Ceph를 Rook을 통해 배포하는 방법을 알아본다.

PaaS-TA 6.0 버전부터는 Kuberspray에서 배포되는 기본클러스터에 컨테이너플랫폼용 서비스를 설치하고자 할 경우에는 Persistence Volume으로 사용할 스토리지가 필수다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 Kubernetes Native를 검증하기 위한 Kubespray 기본 설치를 기준으로 작성하였다.

<br>

## <div id='2'> 2. Rook ceph 설치

<br>

### <div id='2.1'> 2.1. 구성
![image](https://user-images.githubusercontent.com/67575226/204473474-519db488-2f05-4d3e-926e-b711cdd73f0f.png)


### <div id='2.2'> 2.2. Prerequisite
본 설치 가이드는 **Ubuntu 20.04** OS환경에 , 컨테이너플랫폼 단독배포 설치 기준으로 K8s클러스터가 설치된 환경을 기준으로 진행한다.
WorkerNode는 3개이상으로 구성해야하며 각각 10Gi 이상의 External Volume을 붙여야 한다.


### <div id='2.3'> 2.3. Rook ceph 구성
- rook ceph 설치 (마스터 노드)
```
$ git clone --single-branch --branch v1.9.3 https://github.com/rook/rook.git
$ cd rook/deploy/examples
$ kubectl create -f crds.yaml
$ kubectl create -f common.yaml
$ kubectl create -f operator.yaml
$ kubectl create -f cluster.yaml
```

- rook ceph 설치확인

```
$ kubectl get ns rook-ceph 
NAME        STATUS   AGE 
rook-ceph   Active   30m 
$ kubectl get pod -n rook-ceph 

```

- 추가 pod 구성
```
$ kubectl create -f toolbox.yaml
$ kubectl create -f csi/rbd/storageclass.yaml
```

- ceph 설치 확인
```
$ kubectl get cephcluster -A 
NAMESPACE   NAME        DATADIRHOSTPATH   MONCOUNT   AGE   PHASE   MESSAGE                        HEALTH      EXTERNAL 
rook-ceph   rook-ceph   /var/lib/rook     3          34m   Ready   Cluster created successfully   HEALTH_OK
```


- ceph 상태 확인을 위한 tookbox pod 접속
```
$ kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') -- bash
```

- ceph 상태 확인
```
[rook@rook-ceph-tools-656c694bcd-dwr9c /]$ ceph status 
  cluster: 
    id:     5fa189a6-7473-4778-9f97-fb9a411f2876 
    health: HEALTH_OK 
  services: 
    mon: 3 daemons, quorum a,b,c (age 26m) 
    mgr: a(active, since 24m), standbys: b 
    osd: 4 osds: 4 up (since 23m), 4 in (since 24m) 
  data: 
    pools:   2 pools, 33 pgs 
    objects: 6 objects, 35 B 
    usage:   21 MiB used, 80 GiB / 80 GiB avail 
    pgs:     33 active+clean

[rook@rook-ceph-tools-656c694bcd-dwr9c /]$ ceph osd pool stats 
pool device_health_metrics id 1 
  nothing is going on 
pool replicapool id 2 
  nothing is going on
```


- StorageClass 확인
```
$ kubectl get sc 
NAME              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE 
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   30m
```

- PVC 배포 
```
$ cat <<EOF | kubectl apply -f - 
apiVersion: v1 
kind: PersistentVolumeClaim 
metadata: 
  name: mongo-pvc 
spec: 
  storageClassName: rook-ceph-block 
  accessModes: 
  - ReadWriteOnce 
  resources: 
    requests: 
      storage: 2Gi 
EOF 
```

- PV,PVC 확인
```
$ kubectl get pv,pvc 
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS      REASON   AGE 
persistentvolume/pvc-97fb61a4-07c2-4155-8124-b6d1bafc566b   2Gi        RWO            Delete           Bound    default/mongo-pvc   rook-ceph-block            24m 
NAME                              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE 
persistentvolumeclaim/mongo-pvc   Bound    pvc-97fb61a4-07c2-4155-8124-b6d1bafc566b   2Gi        RWO            rook-ceph-block   24m
```

- NodePort 배포 Dashboard확인
```
apiVersion: v1 
kind: Service 
metadata: 
  name: rook-ceph-mgr-dashboard-nodeport 
  namespace: rook-ceph 
  labels: 
    app: rook-ceph-mgr 
    rook_cluster: rook-ceph 
spec: 
  ports: 
  - name: dashboard 
    port: 8443 
    protocol: TCP 
    targetPort: 8443 
  selector: 
    app: rook-ceph-mgr 
    rook_cluster: rook-ceph 
  sessionAffinity: None 
  type: NodePort
```

- Service 배포 확인
```
$ kubectl get svc -n rook-ceph 
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE 
csi-cephfsplugin-metrics           ClusterIP   10.233.22.157   <none>        8080/TCP,8081/TCP   129m 
csi-rbdplugin-metrics              ClusterIP   10.233.23.196   <none>        8080/TCP,8081/TCP   129m 
rook-ceph-mgr                      ClusterIP   10.233.14.163   <none>        9283/TCP            22m 
rook-ceph-mgr-dashboard            ClusterIP   10.233.50.109   <none>        7000/TCP            22m 
rook-ceph-mgr-dashboard-nodeport   NodePort    10.233.20.171   <none>        7000:31683/TCP      5m40s 
rook-ceph-mon-a                    ClusterIP   10.233.52.20    <none>        6789/TCP,3300/TCP   138m 
rook-ceph-mon-b                    ClusterIP   10.233.45.1     <none>        6789/TCP,3300/TCP   100m 
rook-ceph-mon-c                    ClusterIP   10.233.40.207   <none>        6789/TCP,3300/TCP   74m
```

- Ceph Dashboard 초기 패스워드 확인 (admin)
```
$ kubectl get secret rook-ceph-dashboard-password -o yaml -n rook-ceph | grep "password:" | awk '{print $2}' | base64 --decode
```

![image](https://user-images.githubusercontent.com/67575226/204476235-3c3ed360-1576-4451-b090-3cfcc4e6ba44.png)


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide) > > Rook ceph 설치
