## Table Contents
1. [문서 개요](#1)    
 1.1 [목적](#1.1)    
 1.2 [유의사항](#1.2)  

2. [CCE 진단항목](#2)  
  2.1 [root 계정 원격 접속 제한](#2.1)    
  2.2 [패스워드 복잡성 설정](#2.2)    
  2.3 [계정 잠금 임계값 설정](#2.3)    
  2.4 [파일 및 디렉터리 소유자 설정](#2.4)    
  2.5 [/etc/shadow 파일 소유자 및 권한 설정](#2.5)    
  2.6 [SUID, SGID, Sticky bit 설정 파일 점검](#2.6)    
  2.7 [world writable 파일 점검](#2.7)    
  2.8 [Docker daemon audit 설정](#2.8)    
  2.9 [/usr/lib/docker audit 설정](#2.9)    
  2.10 [/etc/docker audit 설정](#2.10)    
  2.11 [docker.service audit 설정](#2.11)    
  2.12 [docker.socket audit 설정](#2.12)    
  2.13 [/etc/default/docker audit 설정](#2.13)    
  2.14 [default bridge를 통한 컨테이너 간 네티워크 트래픽 제한](#2.14)         
  2.15 [DOCKER_CONTENT_TRUST 값 설정](#2.15)     
  2.16 [패스워드 최대 사용 기간 설정](#2.16)   
  2.17 [도커의 default bridge docker0 사용 제한](#2.17)  

3. [CCE 진단항목(Docker 취약사항 대체용 Kubernetes 취약점 조치)](#3)      
  3.1 [API서버 인증제어](#3.1)   
  3.2 [API서버 권한제어](#3.2)   
  3.3 [Controller Manager 인증제어](#3.3)   
  3.4 [Kubelet 인증 제어](#3.4)   
  3.5 [Kubelet 권한 제어](#3.5)   
  3.6 [Container에 대한 보안 프로필 적용](#3.6)      

4. [CVE 진단항목](#4)  
  4.1 [TCP timestamp responses 비활성화 설정](#4.1)           
  4.2 [X.509 인증서의 Subject CN필드가 Entity Name과 불일치](#4.2)         
  4.3 [신뢰할 수 없는 TLS/SSL server X.509 인증서](#4.3)         
  4.4 [자체 서명된 TLS/SSL 인증서](#4.4)           

## <div id='1'/>1. 문서 개요
### <div id='1.1'/>1.1. 목적
본 문서는 CVE, CCE 취약점에 대한 기술적 보안 가이드를 제공한다. 각각은 항목설명, 조치대상, 진단방법, 조치방법으로 구성되어있다. 본 가이드를 활용하여 취약점 내용에 관련해 보안조치를 취할 수 있다.

### <div id='1.2'/>1.2. 유의사항
본 문서의 수록된 판단 기준은 클라우드 인증 평가 시 사용되고 있는 사항이며, 양호 혹은 취약을 가르는 실제 판단 기준은 각 클라우드 서비스 운영에 따라 다양한 환경, 정책 등 고려하여
심사원이 최종적으로 결정한다.

<br>

##  <div id='2'/>2. CCE 진단항목
### <div id='2.1'/>2.1. root 계정 원격 접속 제한

- 항목 설명
  + 각종 공격(무작위 대입 공격, 사전 대입 공격 등)을 통해 root 원격 접속 차단이 적용되지 않은 시스템의 root 계정 정보를 비인가자가 획득할 경우 시스템 계정 정보 유출, 파일 및 디렉터리 변조 등의 행위 침해사고가 발생할 수 있다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + /etc/ssh/sshd_config 파일에서 Root 로그인 설정 확인
```
# cat /etc/ssh/sshd_config | grep PermitRootLogin
```

- 조치방법
  + /etc/ssh/sshd_config 설정 값 변경
```
# vi /etc/ssh/sshd_config

[현황]
#PermitRootLogin prohibit-password

[조치]
PermitRootLogin no
```
---

### <div id='2.2'/>2.2. 패스워드 복잡성 설정

- 항목 설명
  + 패스워드 복잡성 설정이 되어 있지 않은 사용자 계정 패스워드 존재 시 비인가자가 각종 공격(무작위 대입 공격, 사전 대입 공격 등)을 통해 취약한 패스워드가 설정된 사용자 계정의 패스워드를 획득하여 획득한 사용자 계정 정보를 통해 해당 사용자 계정의 시스템에 접근할 수 있는 위험이 존재한다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + /etc/pam.d/common-password 파일 설정 내용 확인
```
# cat /etc/pam.d/common-password
```

- 조치방법
  + /etc/pam.d/common-password 파일 편집
```
# vi /etc/pam.d/common-password

[현황]
password  requisite     pam_deny.so

[조치]
password  requisite     pam_pwquality.so enforce_for_root retry=3 minlen=8 dcredit=-1 ucredit=-1 lcredit=-1 ocredit=-1
```
---

### <div id='2.3'/>2.3. 계정 잠금 임계값 설정

- 항목 설명
  + 로그인 실패 임계값이 설정되어 있지 않을 경우 반복되는 로그인 시도에 대한 차단이 이루어지지 않아 각종 공격(무작위 대입 공격, 사전 대입 공격, 추측 공격 등)에 취약하여 비인가자에게 사용자 계정 패스워드를 유출 당할 수 있다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |  

- 진단방법
  + /etc/pam.d/common-auth 파일에서 임계값 설정 확인
```
# cat /etc/pam.d/common-auth
```

- 조치방법
  + /etc/pam.d/common-auth 파일 내 설정 값을 변경
```
# vi /etc/pam.d/common-auth

[현황]
auth    requisite                       pam_deny.so

[조치]
auth    required                        pam_tally2.so deny=5 no_magic_root        
```
---

### <div id='2.4'/>2.4. 파일 및 디렉터리 소유자 설정

- 항목 설명
  + 삭제된 소유자의 UID와 동일한 사용자가 해당 파일, 디렉터리에 접근 가능하여 사용자 정보 등 중요 정보가 노출될 위험이 있다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + 시스템에서 소유자나 그룹이 존재하지 않는 파일 및 디렉터리를 검색
```
# find / -nouser -o -nogroup
```

- 조치방법
  + 소유자가 존재하지 않는 파일이나 디렉터리가 불필요한 경우 rm 명령으로 삭제
  + 필요한 경우 chown 명령으로 소유자 및 그룹 변경 
```
## 양호: 소유자나 그룹이 존재하지 않는 파일 및 디렉터리가 없는 경우
## 취약: 소유자나 그룹이 존재하지 않는 파일 및 디렉터리가 있는 경우

소유자가 존재하지 않는 파일이나 디렉터리가 불필요한 경우 rm 명령으로 삭제
# rm <file name>
# rm -rf <directory name>

필요한 경우 chown 명령으로 소유자 및 그룹 변경 
# chown <user name> <file name>
```
---

### <div id='2.5'/>2.5. /etc/shadow 파일 소유자 및 권한 설정

- 항목 설명
  + 해당 파일에 대한 권한 관리가 이루어지지 않을 시 ID 및 패스워드 정보가 외부로 노출될 수 있다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + /etc/shadow 파일의 퍼미션과 소유자를 확인
```
# ls -l /etc/shadow
```

- 조치방법
  + /etc/shadow 파일의 소유자 및 권한 변경 (소유자 root, 권한 400)
```
[현황]
ex) 검색시 나온 파일
-rw-r----- 1 root shadow 863 Jan 28 06:18 /etc/shadow


[조치]
ex) 검색시 나온 파일의 소유자 및 권한 변경
# chown root /etc/shadow
# chmod 400 /etc/shadow
```
---

### <div id='2.6'/>2.6. SUID, SGID, Sticky bit 설정 파일 점검

- 항목 설명
  + SUID, SGID 파일의 접근권한이 적절하지 않을 경우 SUID, SGID 설정된 파일로 특정 명령어를 실행하여 root 권한 획득 및 정상 서비스 장애를 발생시킬 수 있다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |   

- 진단방법
  + 아래 명령어를 통해 SUID와 SGID 파일을 검색하여 주요 파일의 권한을 확인
```
# find / -user root -type f \( -perm -04000 -o -perm -02000 \) -xdev -exec ls -al {} \;
```

- 조치방법
  + SUID, SGID 설정 제거
```
# chmod -s /usr/bin/newgrp
# chmod -s /sbin/unix_chkpwd
# chmod -s /usr/bin/at
```
---

### <div id='2.7'/>2.7. world writable 파일 점검

- 항목 설명
  + 시스템 파일과 같은 중요 파일에 world writable 설정이 될 경우, 악의적인 사용자가 해당 파일을 마음대로 파일을 덧붙이거나 지울 수 있게 되어 시스템의 무단 접근 및 시스템 장애를 유발할 수 있다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | O |
|| HAProxy | O |
|| Private-image-repository | O |

- 진단방법
  + world writable 파일 존재 여부 확인
```
# 참고: 아래 명령어로 진단시 너무 많은 항목이 나오기 때문에 cce진단 적용시 문제가 되는 항목들만 조치를 취하는 방법을 적용한다. 
# find / -type f -perm -2 -exec ls -l {} \;
```

- 조치방법
  + 일반 사용자 쓰기 권한 제거 방법
```
# chmod o-w <file name>

1) kubernetes master, kubernetes worker에서 cce진단 적용시 문제가 되는 항목들
[현황]
  4039   4 drwxrwxrwt 10 root root 4096 Jan 5 07:46 /tmp
  256009 4 drwxrwxrwt 2 root root 4096 Jan 2 12:25  /tmp/systemd-private-aae1da4e104642589da1b983608b0bee-systemd-timesyncd.service-0nl8Q4/tmp
  256007 4 drwxrwxrwt 2 root root 4096 Jan 2 12:25  /tmp/.Test-unix
  256005 4 drwxrwxrwt 2 root root 4096 Jan 2 12:25  /tmp/.XIM-unix
  256003 4 drwxrwxrwt 2 root root 4096 Jan 2 12:25  /tmp/.X11-unix
  256004 4 drwxrwxrwt 2 root root 4096 Jan 2 12:25  /tmp/.ICE-unix
  256006 4 drwxrwxrwt 2 root root 4096 Jan 2 12:25  /tmp/.font-unix
  67692  4 drwxrwxrwt 2 root root 4096 Oct 26 17:27 /var/crash
  67674  4 drwxrwxrwt 5 root root 4096 Jan 2 13:01  /var/tmp
  256061 4 drwxrwxrwt 2 root root 4096 Jan 2 12:25  /var/tmp/systemd-private-aae1da4e104642589da1b983608b0bee-systemd-timesyncd.service-jeqO9z/tmp
  256028 4 drwxrwxrwt 2 root root 4096 Jan 2 12:26 /var/tmp/cloud-init

[조치]
# chmod o-w /tmp
# chmod o-w /tmp/.Test-unix
# chmod o-w /tmp/.XIM-unix
# chmod o-w /tmp/.X11-unix
# chmod o-w /tmp/.ICE-unix
# chmod o-w /tmp/.font-unix
# chmod o-w /tmp/systemd*/tmp
# chmod o-w /var/crash
# chmod o-w /var/tmp
# chmod o-w /var/tmp/cloud-init
# chmod o-w /var/tmp/systemd*/tmp
-------------------------------------------------------------------------------------------

2) mariadb, haproxy, private-image-repository에서 cce진단 적용시 문제가 되는 항목들
[현황]
 4039      4 drwxrwxrwt  10 root     root         4096 Jan 29 06:15 /tmp
67674      4 drwxrwxrwt   5 root     root         4096 Jan 28 04:52 /var/tmp

[조치]
# BOSH Iception환경 Instance VM 접근 방법
ex) {Instance VM Name} : mariadb (위 세가지 vm 입력)
$ bosh -e <bosh_name> -d paasta-container-platform ssh {Instance VM Name}

# chmod o-w /tmp
# chmod o-w /var/tmp
```
---

### <div id='2.8'/>2.8. Docker daemon audit 설정
- 2.8. Docker daemon audit 설정부터 2.13. /etc/default/docker audit 설정 항목까지는 동일한 파일 audit.rules를 수정하므로 ## Add at the bottom 부분의 설정을 하위에 계속 추가 적용하면 된다. 

- 항목 설명
  + Docker 데몬은 root 권한으로 실행 되기 때문에 그 활동과 용도를 감사하여야한다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + 명령어를 통해 /usr/bin/docker 감사 설정 확인
```
# auditctl -l | grep /usr/bin/docker
```
 + 명령어를 통해 /etc/audit/audit.rules 파일의 내용 확인
```
# cat /etc/audit/audit.rules | grep /usr/bin/docker
```

- 조치방법
  + audit 설정 적용
```
다음과 같은 절차로 설정 적용
# apt install auditd -y
# vi /etc/audit/rules.d/audit.rules

## Add at the bottom
-w /usr/bin/docker -k docker

# systemctl restart auditd.service
```
---

### <div id='2.9'/>2.9. /usr/lib/docker audit 설정

- 항목 설명
  + /var/lib/docker 디렉터리는 컨테이너에 대한 모든 정보를 보유하고 있는 디렉터리이므로 감사 설정을 하여야 한다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + 명령어를 통해 /var/lib/docker 감사 설정 확인
```
# auditctl -l | grep /var/lib/docker
```
 + 명령어를 통해 /etc/audit/audit.rules 파일의 내용 확인
```
# cat /etc/audit/audit.rules | grep /var/lib/docker
```

- 조치방법
  + audit 설정 적용
```
다음과 같은 절차로 설정 적용
# apt install auditd -y
# vi /etc/audit/rules.d/audit.rules

## Add at the bottom
-w /var/lib/docker -k docker

# systemctl restart auditd.service
```
---

### <div id='2.10'/>2.10. /etc/docker audit 설정

- 항목 설명
  + /etc/docker 디렉터리는 Docker 데몬과 Docker 클라이언트 간의 TLS 통신에 사용되는 다양한 인증서와 키를 보유하고 있으므로 감사 설정을 하여야 한다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + 명령어를 통해 /etc/docker 감사 설정 확인
```
# auditctl -l | grep /etc/docker
```
 + 명령어를 통해 /etc/audit/audit.rules 파일의 내용 확인
```
# cat /etc/audit/audit.rules | grep /etc/docker
```

- 조치방법
  + audit 설정 적용
```
다음과 같은 절차로 설정 적용
# apt install auditd -y
# vi /etc/audit/rules.d/audit.rules

## Add at the bottom
-w /etc/docker -k docker

# systemctl restart auditd.service
```
---

### <div id='2.11'/>2.11. docker.service audit 설정

- 항목 설명
  + 데몬 매개변수가 관리자에 의해 변경된 경우 docker.service 파일이 존재한다. docker.service 파일은 Docker 데몬을 위한 다양한 파라미터를 보유하고 있으므로 감사 설정을 하여야 한다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + docker.service 파일의 경로 확인
```
# systemctl show -p FragmentPath docker.service
```
  + 명령어를 통해 docker.service 감사 설정 확인
```
# auditctl -l | grep /lib/systemd/system/docker.service
```
 + 명령어를 통해 /etc/audit/audit.rules 파일의 내용 확인
```
# cat /etc/audit/audit.rules | grep docker.service
```

- 조치방법
  + audit 설정 적용
```
다음과 같은 절차로 설정 적용
# apt install auditd -y
# vi /etc/audit/rules.d/audit.rules

## Add at the bottom
-w /lib/systemd/system/docker.service -k docker

# systemctl restart auditd.service
```
---

### <div id='2.12'/>2.12. docker.socket audit 설정

- 항목 설명
  + docker.socket 파일은 Docker 데몬 소켓을 위한 다양한 파라미터를 보유하고 있으므로 감사 설정을 하여야 한다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |  

- 진단방법
  + docker.socket 파일의 경로 확인
```
# systemctl show -p FragmentPath docker.socket
```
  + 명령어를 통해 docker.socket  감사 설정 확인
```
# auditctl -l | grep /lib/systemd/system/docker.socket
```
 + 명령어를 통해 /etc/audit/audit.rules 파일의 내용 확인
```
# cat /etc/audit/audit.rules | grep docker.socket
```

- 조치방법
  + audit 설정 적용
```
다음과 같은 절차로 설정 적용
# apt install auditd -y
# vi /etc/audit/rules.d/audit.rules

## Add at the bottom
-w /lib/systemd/system/docker.socket -k docker

# systemctl restart auditd.service
```
---

### <div id='2.13'/>2.13. /etc/default/docker audit 설정

- 항목 설명
  + /etc/default/docker 파일은 Docekr 데몬을 위한 다양한 파라미터를 보유하고 있으므로 감사 설정을 하여야 한다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + 명령어를 통해 /etc/default/docker 감사 설정 확인
```
# auditctl -l | grep /etc/default/docker
```
 + 명령어를 통해 /etc/audit/audit.rules 파일의 내용 확인
```
# cat /etc/audit/audit.rules | /etc/default/docker
```

- 조치방법
  + audit 설정 적용
```
다음과 같은 절차로 설정 적용
# apt install auditd -y
# vi /etc/audit/rules.d/audit.rules

## Add at the bottom
-w /etc/default/docker -k docker

# systemctl restart auditd.service
```
---

### <div id='2.14'/>2.14. default bridge를 통한 컨테이너 간 네티워크 트래픽 제한

- 항목 설명
  + Default Network Bridge의 동일한 호스트에서 컨테이너 간의 네트워크 통신은 제한되지 않는다. 따라서 각 컨테이너는 호스트의 네트워크를 통해 다른 컨테이너 네트워크 패킷을 모두 볼 수 있다. 이로 인해 의도하지 않거나 원하지 않는 정보가 다른 컨테이너에 공개될 수 있으므로 Default Network Bridge에서 컨테이너 간 통신을 제한하여야 한다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master(Cloudcore) | O |
|| Worker(EdgeCore) | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |   

- 진단방법
  + 아래 명령을 통해 컨테이너 간 제한 옵션이 적용되어 있는지 확인
```
$ docker network ls --quiet | xargs docker network inspect --format "{{ .Name}}: {{.Options }}"
```

- 조치방법 (단독 배포)
  + --icc=false 옵션 추가
```
# vi /etc/systemd/system/docker.service.d/docker-options.conf
(변경)    --iptables=true
(추가)    --icc=false

[Service]
Environment="DOCKER_OPTS= --iptables=true \
--icc=false \
--exec-opt native.cgroupdriver=systemd \
 \
--data-root=/var/lib/docker \
--log-opt max-size=50m --log-opt max-file=5 \

# 도커 daemon reload 및 재시작
# systemctl daemon-reload
# systemctl restart docker
```

- 조치방법 (Edge 배포)
  + "icc":false 추가
```
# vi /etc/docker/daemon.json
{
        "icc":false
}

# 도커 daemon reload 및 재시작
# systemctl daemon-reload
# systemctl restart docker
```
---

### <div id='2.15'/>2.15. DOCKER_CONTENT_TRUST 값 설정

- 항목 설명
  + Docker의 이미지 변조 제어를 한다. 

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 조치방법
  + /etc/bash.bashrc 파일 설정 변경 및 적용
  + DOCKER_CONTENT_TRUST항목을 1로 지정한다.
```
# vi /etc/bash.bashrc
----------------------------------------
## Add at the bottom
export DOCKER_CONTENT_TRUST=1
----------------------------------------
# source /etc/bash.bashrc
```

### <div id='2.16'/>2.16. 패스워드 최대 사용 기간 설정

- 항목 설명
  + 패스워드 최대 사용기간을 설정하지 않은 경우 비인가자의 각종 공격(무작위 대입 공격, 사전 대입 공격 등)을 시도할 수 있는 기간 제한이 없으므로 공격자 입장에서는 장기적인 공격을 시행할 수 있어 시행한 기간에 비례하여 사용자 패스워드가 유출될 수 있는 확률이 증가한다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 진단방법
  + /etc/login.defs 파일에서 패스워드 최대 사용 기간의 설정 값 확인  
  + 각 계정별 설정 값 확인
```
# cat /etc/login.defs | grep PASS_MAX_DAYS

## <계정명> : ex) root, ubuntu
# chage -l <계정명>
```

- 조치방법  
  + /etc/login.defs 파일 설정 변경 
  + 각 계정별 설정 변경
```
1) /etc/login.defs 파일 설정 변경 
# vim /etc/login.defs

[현황]
PASS_MAX_DAYS     99999

[조치]
PASS_MAX_DAYS     90
--------------------------
2) 각 계정별 설정 변경
## <계정명> : ex) root, ubuntu
# chage -M 90 <계정명> 

[현황]
root
Maximum number of days between password change          : 99999

ubuntu
Maximum number of days between password change          : 99999

[조치]
root
Maximum number of days between password change          : 90

ubuntu
Maximum number of days between password change          : 90
```

#### <div id='2.17'/>2.17. 도커의 default bridge docker0 사용 제한
- 항목 설명
  + Docker는 브리지 모드에서 생성된 가상 인터페이스를 docker0라는 공통 브리지에 연결
  + 이 네트워크 모델은 필터링이 적용되지 않기 때문에 ARP Spoofing 및 MAC Flooding 등의 공격에 취약

- 조치방법  
  + kubernetes는 CNI를 이용한 network ovelay를 통해 pod간 네트워킹을 구성하기에 docker0 bridge는 사용되지 않음. (1번 사항 참조) 
  + 운영사항을 고려한 brigde제거 방법 (2번 사항 참조)  

1. Docker0 Bridge Network가 사용 되고 있는 상태(삭제 전 예시)
```
## bridge 사용 확인을 위해 bridge-utils 패키지 설치
$ sudo apt install bridge-utils

## Network Brdige 확인 
$ brctl show
bridge name            bridge id                STP enabled        interfaces
docker0                8000.024282e2a076        no

## Pod 배포
$ kubectl apply -f {POD_YAML_FILE}
$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
nginx-559d75d76b-d6p84                 1/1     Running   0          11m

## Pod를 배포 하여도 interfaces를 보면 docker0 bridge에 아무것도 붙지 않는 것을 확인 할 수 있다.
$ brctl show
bridge name            bridge id                STP enabled        interfaces
docker0                8000.024282e2a076        no

## Calico CNI Plugin 사용, Pod 배포 시 calixxxxxxxxxx 로 interface가 할당된다.
## docker0 network bridge가 사용 되고 있는 것을 확인할 수 있다.
$ ifconfig
cali57a63d8e86c: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::ecee:eeff:feee:eeee  prefixlen 64  scopeid 0x20<link>
        ether ee:ee:ee:ee:ee:ee  txqueuelen 0  (Ethernet)
        RX packets 960  bytes 81389 (81.3 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1035  bytes 536155 (536.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

cali76e9dbd11dd: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::ecee:eeff:feee:eeee  prefixlen 64  scopeid 0x20<link>
        ether ee:ee:ee:ee:ee:ee  txqueuelen 0  (Ethernet)
        RX packets 1858  bytes 159032 (159.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1899  bytes 291632 (291.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...생략...
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:25:66:af:b5  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...생략...

## 도커 network 확인
$ sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
095c47839397   bridge    bridge    local
ec24cb1c3ae4   host      host      local
58219ab61fba   none      null      local
```
2. Docker0 Bridge Network 삭제 (삭제 후 예시)
```
##  Docker0 Bridge Network 를 삭제하기 위한 네트워크 설정
$ sudo vi /etc/docker/daemon.json
{
       "bridge": "none"
}

## 도커 daemon reload 및 재시작
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker

## Network Brdige 확인 
$ brctl show
bridge name            bridge id                STP enabled        interfaces

## Pod 배포
$ kubectl apply -f {POD_YAML_FILE}
$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
nginx-559d75d76b-d6p84                 1/1     Running   0          13m
nginx2-559d75d76b-t24tg                1/1     Running   0          15s

## Pod 배포 후 Network Brdige 재확인 
$ brctl show
bridge name            bridge id                STP enabled        interfaces

## docker0 bridge가 사라진 것을 볼 수 있다.
$ ifconfig
cali57a63d8e86c: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::ecee:eeff:feee:eeee  prefixlen 64  scopeid 0x20<link>
        ether ee:ee:ee:ee:ee:ee  txqueuelen 0  (Ethernet)
        RX packets 203  bytes 19527 (19.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 244  bytes 245060 (245.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

cali76e9dbd11dd: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::ecee:eeff:feee:eeee  prefixlen 64  scopeid 0x20<link>
        ether ee:ee:ee:ee:ee:ee  txqueuelen 0  (Ethernet)
        RX packets 263  bytes 26006 (26.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 264  bytes 92832 (92.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
...생략...

## 도커 network 확인
$ sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
ec24cb1c3ae4   host      host      local
58219ab61fba   none      null      local
```

<br>

### <div id='3'/> 3. CCE 진단항목(Docker 취약사항 대체용 Kubernetes 취약점 조치)
- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> | <center>적용 범위</center> |
| :--- | :--- | :---: | :---: |
| Cluster | Master | O | O |
|| Worker | X | O |

#### <div id='3.1'/>3.1. API서버 인증제어
- 항목 설명
  + API Server는 Kubernetes 요소들의 허브로 HTTP, HTTPS 기반의 REST API를 제공 한다. 다른 모든 구성요소를 상호 작용할 수 있도록 연결하는 역할을 한다.
  + API Server에 대한 인증제어를 검증한다. 

- docker 점검 대상 항목
  + 도커 클라이언트 인증 활성화
  + 추가 권한 획득으로부터 컨테이너 제한
  + root가 아닌 user로 컨테이너 실행

- 조치방법
  + kube-apiserver.yaml 파일은 Master에만 있기 때문에 Master에 CCE진단 적용을 하면 전체 Cluster에 보안점검이 적용 된다.  
  + /etc/kubernetes/manifests/kube-apiserver.yaml 파일 수정  
  + 파일 내에 아래 항목이 존재하면 설정 값 변경, 존재하지 않으면 설정 값 추가
```
 $ sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

 ## 비인증 접근 차단
 ----------------------------------------
 --anonymous-auth=false
 --insecure-allow-any-token=true 제거
 --insecure-bind-address=X.X.X.X 제거
 --insecure-port=0
 ----------------------------------------

 ## 취약한 방식의 인증 방식 사용 제거
 ----------------------------------------
 --basic-auth-file={filename} 제거
 --token-auth-file={filename} 제거
 ----------------------------------------
```
![image](https://user-images.githubusercontent.com/67575226/106992759-966a6600-67bc-11eb-8d7a-e8a218613d70.png)

#### <div id='3.2'/>3.2. API서버 권한제어
- 항목 설명
  + Kubernetes는 API Server를 통해 정책에 따라 요청에 대한 승인을 수행한다.
  + API서버 권한제어를 검증한다.

- docker 점검 대상 항목
  + 도커 클라이언트 인증 활성화
  + 추가 권한 획득으로부터 컨테이너 제한
  + root가 아닌 user로 컨테이너 실행

- 조치방법
  + kube-apiserver.yaml 파일은 Master에만 있기 때문에 Master에 CCE진단 적용을 하면 전체 Cluster에 보안점검이 적용 된다.
  + /etc/kubernetes/manifests/kube-apiserver.yaml 파일 수정
  + 파일 내에 아래 항목이 존재하면 설정 값 변경, 존재하지 않으면 설정 값 추가
```
 $ sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

 ## Node 권한, 역할 기반 엑세스 제어 (RBAC) 사용
 ----------------------------------------
 --authorization-mode=Node,RBAC
 ----------------------------------------
```
![image](https://user-images.githubusercontent.com/67575226/106992884-d03b6c80-67bc-11eb-9a76-b09b17fe57c9.png)

#### <div id='3.3'/>3.3. Controller Manager 인증제어
- 항목 설명
  + Kubernetes에서 Controller는 API Server를 통해 클러스터의 공유 상태를 감시한다.(현재 상태를 원하는 상태로 이동하려고 변경하는 제어 루프)
  + Controller Manger 인증제어를 검증한다.

- docker 점검 대상 항목
  + 도커 클라이언트 인증 활성화
  + 추가 권한 획득으로부터 컨테이너 제한
  + root가 아닌 user로 컨테이너 실행

- 조치방법
  + kube-controller-manager.yaml 파일은 Master에만 있기 때문에 Master에 CCE진단 적용을 하면 전체 Cluster에 보안점검이 적용 된다.
  + /etc/kubernetes/manifests/kube-controller-manager.yaml 파일 수정
  + 파일 내에 아래 항목이 존재하면 설정 값 변경, 존재하지 않으면 설정 값 추가
```
 $ sudo vi /etc/kubernetes/manifests/kube-controller-manager.yaml

 ## 각 컨트롤러에 대한 개별 서비스 계정 자격 증명
 ----------------------------------------
 --use-service-account-credentials=true
 ----------------------------------------
 
 ## 컨트롤러 계정 자격증명에 사용되는 인증서 관리
 ----------------------------------------
 --service-account-private-key-file=/etc/kubernetes/ssl/sa.key
 ----------------------------------------
```
![image](https://user-images.githubusercontent.com/67575226/106992932-f103c200-67bc-11eb-9b19-8b2ae91bacec.png)

#### <div id='3.4'/>3.4. Kubelet 인증 제어
- 항목 설명
  + Kubelet은 Kubernetes 각 노드에서 실행되는 에이전트이다. Pod에 대해 정의된 YAML 또는 JSON 현태의 PodSpec에 따라 컨테이너를 실행하고 관리한다.
  + Kubelet 인증 제어를 검증한다.

- docker 점검 대상 항목
  + 도커 클라이언트 인증 활성화
  + 추가 권한 획득으로부터 컨테이너 제한
  + root가 아닌 user로 컨테이너 실행

- 조치방법
  + config.yaml 파일은 Master에만 있기 때문에 Master에 CCE진단 적용을 하면 전체 Cluster에 보안점검이 적용 된다.
  + /var/lib/kubelet/config.yaml 파일 수정
  + 파일 내에 아래 항목이 존재하면 설정 값 변경, 존재하지 않으면 설정 값 추가
```
 $ sudo vi /var/lib/kubelet/config.yaml

 ## Kubelet 인증 제어
 ----------------------------------------
 authentication:
    anonymous:
        enabled: false
 
 readOnlyPort: 0
 ----------------------------------------
```
![image](https://user-images.githubusercontent.com/67575226/106992965-05e05580-67bd-11eb-8749-acb298c60a78.png)

#### <div id='3.5'/>3.5. Kubelet 권한 제어
- 항목 설명
  + Kubelet은 기본적으로 Kubernetes Master의 API Server에서 전달되는 요청에 대해 권한 검사 없이 모두 허용한다. 
  + 설정 변경을 통해 권한 검증을 수행한다.

- docker 점검 대상 항목
  + 도커 클라이언트 인증 활성화
  + 추가 권한 획득으로부터 컨테이너 제한
  + root가 아닌 user로 컨테이너 실행


- 조치방법
  + config.yaml 파일은 Master에만 있기 때문에 Master에 CCE진단 적용을 하면 전체 Cluster에 보안점검이 적용 된다.
  + /var/lib/kubelet/config.yaml 파일 수정
  + 파일 내에 아래 항목이 존재하면 설정 값 변경, 존재하지 않으면 설정 값 추가
```
 $ sudo vi /var/lib/kubelet/config.yaml

 ## Kubelet 인증 제어
 ----------------------------------------
 authorization:
     mode: Webhook
 ----------------------------------------
```
![image](https://user-images.githubusercontent.com/67575226/106993012-1d1f4300-67bd-11eb-8beb-a0ee6f53579f.png)

#### <div id='3.6'/>3.6. Container에 대한 보안 프로필 적용
- 항목 설명
  + AppArmor는 LSM(Linux Security Module)을 사용해 만든 SELinux 대안 프레임워크
  + 호스트 운영체제에서 실행되는 프로세스의 기능을 제한하는데 사용할 수 있는 Linux 커널 보안 모듈
  + 각 프로세스는 자체 보안 프로필을 가질 수 있으며 네트워크 액세스, 파일 읽기/쓰기/실행 권한과 같은 특정 기능을 허용하거나 허용하지 않음
  + Ubuntu 7.10 이후 기본적으로 Ubuntu에 포함된 중요한 보안 기능이며 컨테이너 실행 시 docker-default AppArmor 보안 프로필을 자동으로 적용

- docker 점검 대상 항목
  + 컨테이너 SELinux 보안 옵션 설정

```
 ## AppArmor 상태 확인
 $ sudo apparmor_status

 ## docker-default 보안 프로필 테스트
 $ kubectl  exec –it nginx-xxxxx -- /bin/bash
 $ cat proc/sysrq-trigger
```
![image](https://user-images.githubusercontent.com/67575226/106980702-03bdcd00-67a4-11eb-89d8-376e4a5a1bd1.png)
![image](https://user-images.githubusercontent.com/67575226/106980792-2a7c0380-67a4-11eb-8aca-bb79604a18de.png)

<br>

##  <div id='4'/>4. CVE 진단항목
### <div id='4.1'/> 4.1. TCP timestamp responses 비활성화 설정

- 항목 설명
  + TCP 타임스탬프 응답을 사용하면 원격 호스트 가동 시간의 근사치를 계산하고 향후 공격 시 도움을 줄 수 있다. 또한, 일부 운영 체제의 경우 해당 TCP 타임스탬프의 동작을 바탕으로 핑거프린팅될 수 있다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |
| Bosh | MariaDB | X |
|| HAProxy | X |
|| Private-image-repository | X |

- 조치방법
  + /etc/sysctl.conf 파일 설정 변경, iptables에 정책 추가
```
 $ sudo vi /etc/sysctl.conf
 ----------------------------------------
 ## Add at the bottom
 net.ipv4.tcp_timestamps=0
 ----------------------------------------
 $ sudo reboot

 #  iptables에 정책 추가
 $ sudo iptables -A INPUT -p icmp --icmp-type timestamp-request -j DROP
 $ sudo iptables -A OUTPUT -p icmp --icmp-type timestamp-reply -j DROP
```

### <div id='4.2'/> 4.2. X.509 인증서의 Subject CN필드가 Entity Name과 불일치

- 항목설명
  + 인증서를 발급하기 전에 인증 기관은 CA의 CPS (Certification Practice Statement)에 지정된 대로 인증서를 요청하는 엔터티의 ID를 확인해야 한다. 따라서 표준 인증서 유효성 검사 절차에서는 인증서를 제시하는 엔티티의 실제 이름과 일치하는 인증서의 제목 CN 필드가 필요하다.

- 조치대상

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |

- 조치방법
  + 인증서 구매 후 Kubernetes에서 secret tls를 추가하여 Ingress에 인증서 적용여부만 결정
  + Ingress에 적용하는 방법 이외에 인증서를 기존과 같이 WebServer에 포함시키고, 이를 Ingress 뒷 단인 Pod 내부에 두는 방식으로 사용 가능하며, L4Switch&CDN 등 Kubernetes 앞단에 인증서를 적용하여 활용도 가능
  + 운영환경에 맞춰 인증서 처리 진행

### <div id='4.3'/> 4.3. 신뢰할 수 없는 TLS/SSL server X.509 인증서

- 항목설명
  + 서버의 TLS/SSL 인증서는 신뢰할 수 없는 인증 기관에서 서명 되었다. 자체 서명된 인증서의 사용은 TLS/SSL 중간자 공격이 발생하고 있음을 나타낼 수 있으므로 권장되지 않는다.

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |

- 조치방법
  + 인증서 구매 후 Kubernetes에서 secret tls를 추가하여 Ingress에 인증서 적용여부만 결정
  + Ingress에 적용하는 방법 이외에 인증서를 기존과 같이 WebServer에 포함시키고, 이를 Ingress 뒷 단인 Pod 내부에 두는 방식으로 사용 가능하며, L4Switch&CDN 등 Kubernetes 앞단에 인증서를 적용하여 활용도 가능
  + 운영환경에 맞춰 인증서 처리 진행

### <div id='4.4'/> 4.4. 자체 서명된 TLS/SSL 인증서

- 항목설명
  + 서버의 TLS/SSL 인증서는 자체 서명된다. 특히 TLS/SSL man-in-the-middle 공격은 일반적으로 자체 서명 된 인증서를 사용하여 TLS/SSL 연결을 도청하기 때문에 자체 서명 된 인증서는 기본적으로 신뢰할 수 없다.

| <center>대상 환경</center> | <center>분류</center> | <center>조치 대상</center> |
| :--- | :--- | :---: |
| Cluster | Master | O |
|| Worker | O |

- 조치방법
  + 인증서 구매 후 Kubernetes에서 secret tls를 추가하여 Ingress에 인증서 적용여부만 결정
  + Ingress에 적용하는 방법 이외에 인증서를 기존과 같이 WebServer에 포함시키고, 이를 Ingress 뒷 단인 Pod 내부에 두는 방식으로 사용 가능하며, L4Switch&CDN 등 Kubernetes 앞단에 인증서를 적용하여 활용도 가능
  + 운영환경에 맞춰 인증서 처리 진행

<br>
