### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](/install-guide/Readme.md) > GlusterFS 서버 설치


# GlusterFS 서버 설치

## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  

2. [NFS Server 설치](#2)  
  2.1. [구성](#2.1)  
  2.2. [Prerequisite](#2.2)  
  2.3. [glusterFS Volume 구성](#2.3)  
  2.4. [Heketi 서버 설정](#2.4)  
  2.5. [Volume Resouce 테스트](#2.5)

<br>

## <div id='1'> 1. 문서 개요

### <div id='1.1'> 1.1. 목적
본 문서 (Kubespray 설치 가이드) 는 개방형 PaaS 플랫폼 고도화 및 개발자 지원 환경 기반의 Open PaaS에 배포되는 컨테이터 플랫폼을 위한 glusterFS 클러스터의 구성과 Heketi를 통한 REST API를 이용한 컨테이너플랫폼에서의 활용 방법을 기술하였다.

PaaS-TA 6.0 버전부터는 Kuberspray에서 배포되는 기본클러스터에 컨테이너플랫폼용 서비스를 설치하고자 할 경우에는 Persistence Volume으로 사용할 스토리지가 필수다.

<br>

### <div id='1.2'> 1.2. 범위
설치 범위는 Kubernetes Native를 검증하기 위한 Kubespray 기본 설치를 기준으로 작성하였다.

<br>

## <div id='2'> 2. glusterFS Server 설치

<br>

### <div id='2.1'> 2.1. 구성
![image](https://user-images.githubusercontent.com/67575226/204468030-8ab2c67b-8cda-40f1-8246-636e2c601d2c.png)


### <div id='2.2'> 2.2. Prerequisite
본 설치 가이드는 **Ubuntu 20.04** 환경에서 설치하는 것을 기준으로 하였다.


### <div id='2.3'> 2.3. glusterFS Volume 구성
- 각 서버의 도메인 설정(모든서버)
```
$ sudo vi /etc/hosts
10.100.1.78 gluster-1
10.100.1.169 gluster-2
10.100.1.133 gluster-3
```

- 도메인 통신 확인

```
$ ping gluster-2 
PING gluster-2 (10.100.1.169) 56(84) bytes of data. 
64 bytes from gluster-2 (10.100.1.169): icmp_seq=1 ttl=64 time=1.43 ms 
64 bytes from gluster-2 (10.100.1.169): icmp_seq=2 ttl=64 time=0.279 ms
```

- install GlusterFS Server(모든 서버)
```
$ sudo apt install software-properties-common -y  
$ wget -O- https://download.gluster.org/pub/gluster/glusterfs/6/rsa.pub | apt-key add -  
$ sudo add-apt-repository ppa:gluster/glusterfs-6  
$ sudo apt install glusterfs-server -y  
$ sudo systemctl start glusterd  
$ sudo systemctl enable glusterd
```

- Peer 설정
```
$ sudo gluster peer probe gluster-2 
peer probe: success.  
$ sudo gluster peer probe gluster-3 
peer probe: success.
```


- peer 연결상태 확인
```
$ sudo gluster peer status  
UUID                                    Hostname        State 
b221334c-b8cc-44d3-b462-043411bef4c8    gluster-2       Connected  
7f187bda-b084-4fe0-995e-515fe8d894ab    gluster-3       Connected  
297b9046-51ca-47ca-9e5c-dcfcdc1efeb4    localhost       Connected
```

- peer 연결상태 확인2
```
$ sudo gluster peer status  
Number of Peers: 2 
Hostname: gluster-2 
Uuid: b221334c-b8cc-44d3-b462-043411bef4c8 
State: Peer in Cluster (Connected) 
Hostname: gluster-3 
Uuid: 7f187bda-b084-4fe0-995e-515fe8d894ab 
State: Peer in Cluster (Connected)
```


### <div id='2.4'> 2.4. Heketi 서버 설정

- Download Heketi
```
$ wget https://github.com/heketi/heketi/releases/download/v9.0.0/heketi-v9.0.0.linux.amd64.tar.gz 
$ tar -zxvf heketi-v9.0.0.linux.amd64.tar.gz
```

- Copy Bin
```
$ chmod +x heketi/heketi
$ chmod +x heketi/heketi-cli
$ sudo cp heketi/heketi /usr/local/bin
$ sudo cp heketi/heketi-cli /usr/local/bin
```

- Create dir for heketi, Make to heketi.json
```
$ mkdir -p /var/lib/heketi /etc/heketi /var/log/heketi
$ vi /etc/heketi/heketi.json
{ 
  "_port_comment": "Heketi Server Port Number", 
  "port": "8080", 
	"_enable_tls_comment": "Enable TLS in Heketi Server", 
	"enable_tls": false, 
	"_cert_file_comment": "Path to a valid certificate file", 
	"cert_file": "", 
	"_key_file_comment": "Path to a valid private key file", 
	"key_file": "", 
  "_use_auth": "Enable JWT authorization. Please enable for deployment", 
  "use_auth": false, 
  "_jwt": "Private keys for access", 
  "jwt": { 
    "_admin": "Admin has access to all APIs", 
    "admin": { 
      "key": "password" 
    }, 
    "_user": "User only has access to /volumes endpoint", 
    "user": { 
      "key": "paasword" 
    } 
  }, 
  "_backup_db_to_kube_secret": "Backup the heketi database to a Kubernetes secret when running in Kubernetes. Default is off.", 
  "backup_db_to_kube_secret": false, 
  "_profiling": "Enable go/pprof profiling on the /debug/pprof endpoints.", 
  "profiling": false, 
  "_glusterfs_comment": "GlusterFS Configuration", 
  "glusterfs": { 
    "_executor_comment": [ 
      "Execute plugin. Possible choices: mock, ssh", 
      "mock: This setting is used for testing and development.", 
      "      It will not send commands to any node.", 
      "ssh:  This setting will notify Heketi to ssh to the nodes.", 
      "      It will need the values in sshexec to be configured.", 
      "kubernetes: Communicate with GlusterFS containers over", 
      "            Kubernetes exec api." 
    ], 
    "executor": "ssh", 
    "_sshexec_comment": "SSH username and private key file information", 
    "sshexec": { 
      "keyfile": "/etc/heketi/heketi_key", 
      "user": "root", 
      "port": "22", 
      "fstab": "/etc/fstab" 
    }, 
    "_db_comment": "Database file name", 
    "db": "/var/lib/heketi/heketi.db", 
     "_refresh_time_monitor_gluster_nodes": "Refresh time in seconds to monitor Gluster nodes", 
    "refresh_time_monitor_gluster_nodes": 120, 
    "_start_time_monitor_gluster_nodes": "Start time in seconds to monitor Gluster nodes when the heketi comes up", 
    "start_time_monitor_gluster_nodes": 10, 
    "_loglevel_comment": [ 
      "Set log level. Choices are:", 
      "  none, critical, error, warning, info, debug", 
      "Default is warning" 
    ], 
    "loglevel" : "debug", 
    "_auto_create_block_hosting_volume": "Creates Block Hosting volumes automatically if not found or exsisting volume exhausted", 
    "auto_create_block_hosting_volume": true, 
    "_block_hosting_volume_size": "New block hosting volume will be created in size mentioned, This is considered only if auto-create is enabled.", 
    "block_hosting_volume_size": 500, 
    "_block_hosting_volume_options": "New block hosting volume will be created with the following set of options. Removing the group gluster-block option is NOT recommended. Additional options can be added next to it separated by a comma.", 
    "block_hosting_volume_options": "group gluster-block", 
    "_pre_request_volume_options": "Volume options that will be applied for all volumes created. Can be overridden by volume options in volume create request.", 
    "pre_request_volume_options": "", 
    "_post_request_volume_options": "Volume options that will be applied for all volumes created. To be used to override volume options in volume create request.", 
    "post_request_volume_options": "" 
  } 
}
```

- Load all Kernel modules that wiill be required by Heketi.
```
$ sudo modprobe dm_snapshot
$ sudo modprobe dm_mirror
$ sudo modprobe dm_thin_pool
```

- Create ssh key for the API to connect to the other hosts
```
$ sudo ssh-keygen -m PEM -t rsa -b 4096 -q -f /etc/heketi/heketi_key -N ''
$ sudo chown heketi:heketi /etc/heketi/heketi_key*
```

- Copy Key to all hosts
```
$ cat /etc/heketi/heketi_key.pub  
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6yN+AWN/nQ/jkUyKCL/dP2cDZ0zuA5ATtrfxEzXeWqVD38eBIdrz0Q0BlUYLTQjeJQQqA16eXwPAtvbFdID4OGdUYsjJy0+2LtVRUoimLSqZKXcBaAQd9dpN.....

## glusterfs Node
$ sudo su
$ vi $HOME/.ssh/authorized_keys
```

- Allow heketi user perms on folders
```
$ chown heketi:heketi /var/lib/heketi /var/log/heketi /etc/heketi
```

- Create a systemd file
```
$ vi /etc/systemd/system/heketi.service

[Unit] 
Description=Heketi Server 
[Service] 
Type=simple 
WorkingDirectory=/var/lib/heketi 
EnvironmentFile=-/etc/heketi/heketi.env 
User=heketi 
ExecStart=/usr/local/bin/heketi --config=/etc/heketi/heketi.json 
Restart=on-failure 
StandardOutput=syslog 
StandardError=syslog 
[Install] 
WantedBy=multi-user.target
```

- Reload systemd and enable new heketi service
```
$ systemctl daemon-reload 
$ systemctl enable --now heketi
$ systemctl start heketi

## curl을 이용하여 web 서비스 확인
$ curl http://localhost:8080/hello
Hello from Heketi
```

- Create topology
```
$ vi /etc/heketi/topology.json

{ 
  "clusters": [ 
    { 
      "nodes": [ 
                    {          
          "node": { 
            "hostnames": { 
              "manage": [ 
                "gluster-1" 
              ], 
              "storage": [ 
                "10.100.1.78" 
              ] 
            }, 
            "zone": 1 
          }, 
          "devices": [ 
            "/dev/vdb" 
          ] 
        },            { 
          "node": { 
            "hostnames": { 
              "manage": [ 
                "gluster-2" 
              ], 
              "storage": [ 
                "10.100.1.169" 
              ] 
            }, 
            "zone": 1 
          }, 
          "devices": [ 
            "/dev/vdb" 
          ] 
        },            { 
          "node": { 
            "hostnames": { 
              "manage": [ 
                "gluster-3" 
              ], 
              "storage": [ 
                "10.100.1.133" 
              ] 
            }, 
            "zone": 1 
          }, 
          "devices": [ 
            "/dev/vdb" 
          ] 
        } 
      ] 
    } 
  ] 
}
```

- Load topology
```
$ heketi-cli topology load --user admin --secret password --json=/etc/heketi/topology.json -s http://localhost:8080

Creating cluster ... ID: a67b22a8a9efc3c358190b0d7c3e5436 
        Allowing file volumes on cluster. 
        Allowing block volumes on cluster. 
        Creating node gluster-1 ... ID: c2a0e2d68a0802018002665e1ed829d0 
                Adding device /dev/vdb ... OK 
        Creating node gluster-2 ... ID: 662c52eb313b60e9127432d76e67f861 
                Adding device /dev/vdb ... OK 
        Creating node gluster-3 ... ID: fa1e01b117db5db28d9ba02e4b256334 
                Adding device /dev/vdb ... OK
```

- Check connection to other devices work
```
$ heketi-cli cluster list
Clusters: 
Id:a67b22a8a9efc3c358190b0d7c3e5436 [file][block]
```

### <div id='2.5'> 2.5. Volume Resouce 테스트


- Prerequisite<br>
  Kubernetes 각 노드에 glusterfs client 가 설치되어야 한다.<br>
  미설치 시 컨테이너에 Volumen 할당이 안됨.
```
$ sudo apt-get -y install thin-provisioning-tools
$ sudo apt-get install software-properties-common   
$ sudo add-apt-repository ppa:gluster/glusterfs-6   
$ sudo apt-get update   
$ sudo apt install glusterfs-client
```

- Secret
```
apiVersion: V1 
kind: Secret 
metadata: 
  name: heketi-secret 
  namespace: default 
type: "kubernetes.io/glusterfs" 
data: 
  # echo -n "password" | base64 
  # key: PASSWORD_BASE64_ENCODED 
  key: "a2V5cGFzc3dvcmQ="
```


- StorageClass
```
apiVersion: storage.k8s.io/v1 
kind: StorageClass 
metadata: 
  name: "gluster-heketi" 
provisioner: kubernetes.io/glusterfs 
parameters: 
  resturl: "http://10.100.1.78:8080" 
  restuser: "admin" 
  # restuserkey: "password" 
  secretName: "heketi-secret" 
  secretNamespace: "default" 
  gidMin: "40000" 
  gidMax: "50000"
```


- PVC
```
apiVersion: v1 
kind: PersistentVolumeClaim 
metadata: 
 name: pvc-gluster-1 
spec: 
 storageClassName: gluster-heketi 
 accessModes: 
  - ReadWriteMany 
 resources: 
   requests: 
     storage: 1Gi
```


- 배포 후 결과 확인
```
$ kubectl get sc,pvc -n default 
NAME                                              PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE 
storageclass.storage.k8s.io/gluster-heketi        kubernetes.io/glusterfs      Delete          Immediate           false                  36m  

NAME                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE 
persistentvolumeclaim/pvc-gluster-1   Bound    pvc-a6bd0246-22bb-4cc7-94ba-fe8794b42ca8   1Gi        RWX            gluster-heketi   32m
```


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Install](https://github.com/PaaS-TA/paas-ta-container-platform/tree/master/install-guide) > > GlusterFS 서버 설치
