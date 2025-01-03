### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  포털 사용 가이드

<br>

## Table of Contents

1. [문서 개요](#1)  
   1.1. [목적](#1-1)  
   1.2. [범위](#1-2)  

2. [Prerequisite](#2)  
   2.1  [컨테이너 플랫폼 포털 접속 방법](#2-1)  
   2.2. [(참고) Terraman IaC 설정](#2-2)    

3. [컨테이너 플랫폼 포털 구성](#3)   
   3.1. [컨테이너 플랫폼 포털 사용자 권한 유형](#3-1)  
   3.2. [컨테이너 플랫폼 포털 메뉴 구성](#3-2)    

<br>


## <div id='1'/> 1. 문서 개요

### <div id='1-1'/> 1.1. 목적
본 문서는 컨테이너 플랫폼 포털 사용 방법에 대해 기술하였다.

<br>

### <div id='1-2'/> 1.2. 범위
본 문서는 Chrome 브라우저 환경을 기준으로 컨테이너 플랫폼 포털의 사용 방법에 대해 기술하였다.

<br>

## <div id='2'>2. Prerequisite

### <div id='2-1'>2.1. 컨테이너 플랫폼 포털 접속 방법
> [[컨테이너 플랫폼 포털 접속 방법]](/install-guide/portal/cp-portal-standalone-guide.md#4)

<br>

### <div id='2-2'>2.2. (참고) Terraman IaC 설정
컨테이너 플랫폼 포털을 통해 **서브 클러스터를 생성할 경우** 대상 클라우드(IaaS)의 추가 설정이 필요하다. <br>
아래 가이드를 참조하여 설정 완료 후 서브 클러스터 생성을 진행한다.
> [[Terraman IaC 스크립트 가이드]](/use-guide/terraman/cp-terraman-guide.md)



<br>    

## <div id='3'/> 3. 컨테이너 플랫폼 포털 구성
### <div id='3-1'/> 3.1. 컨테이너 플랫폼 포털 사용자 권한 유형
| <center>사용자 유형</center> | <center>기능</center> |
| :--- | :--- |
| Super Admin (전체 관리자) | 전체 클러스터 관리 권한  |
| Cluster Admin (클러스터 관리자) | 할당된 클러스터 관리 권한  |
| User (일반 사용자) | 할당된 클러스터 내 네임스페이스 관리 권한|

<br>

### <div id='3-2'/> 3.2. 컨테이너 플랫폼 포털 메뉴 구성
<table>
<thead>
  <tr>
    <th>Menu</th>
    <th>Category</th>
    <th>Description</th>
    <th>Access Permissions</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="5">🔗<a href="./cp-portal-use-guide-global.md"><b>Global</b></a></td>
    <td>Overview</td>
    <td>클러스터 정보 대시보드</td>
    <td rowspan="5">Super Admin <br> Cluster Admin (조회 권한)</td>
  </tr>
  <tr>
    <td>Clusters</td>
    <td>클러스터 정보 관리</td>
  </tr>
  <tr>
    <td>Cloud Accounts</td>
    <td>Cloud Accounts 정보 관리</td>
  </tr>
  <tr>
    <td>Instance Code Template</td>
    <td>Instance Code Template 정보 관리</td>
  </tr>
  <tr>
    <td>SSH Keys</td>
    <td>SSH Keys 정보 관리</td>
  </tr>   
  <tr>
    <td rowspan="3">🔗<a href="./cp-portal-use-guide-clusters.md"><b>Clusters</b></a></td>
    <td>Overview</td>
    <td>클러스터 내 리소스 대시보드</td>
    <td>All</td>
  </tr>
  <tr>
    <td>Nodes</td>
    <td>Nodes 정보 관리</td>
    <td>Super Admin, Cluster Admin</td>
  </tr>
  <tr>
    <td>Namespaces</td>
    <td>Namespaces 정보 관리</td>
    <td>All</td>
  </tr>
  <tr>
    <td rowspan="2">🔗<a href="./cp-portal-use-guide-catalogs.md"><b>Catalogs</b></a></td>
    <td>Repositories</td>
    <td>Helm Chart Repository 정보 관리</td>
    <td rowspan="2">All</td>
  </tr>
  <tr>
    <td>Releases</td>
    <td>Releases 정보 관리</td>
  </tr>  
  <tr>
    <td rowspan="3">🔗<a href="./cp-portal-use-guide-workloads.md"><b>Workloads</b></a></td>
    <td>Deployments</td>
    <td>Deployments 정보 관리</td>
    <td rowspan="3">All</td>
  </tr>
  <tr>
    <td>Pods</td>
    <td>Pods 정보 관리</td>
  </tr>
  <tr>
    <td>ReplicaSets</td>
    <td>ReplicaSets 정보 관리</td>
  </tr>
  <tr>
    <td rowspan="2">🔗<a href="./cp-portal-use-guide-services.md"><b>Services</b></a></td>
    <td>Services</td>
    <td>Services 정보 관리</td>
    <td rowspan="2">All</td>
  </tr>
  <tr>
    <td>Ingresses</td>
    <td> Ingresses 정보 관리</td>
  </tr>
  <tr>
    <td rowspan="3">🔗<a href="./cp-portal-use-guide-storages.md"><b>Storages</b></a></td>
    <td>Persistent Volumes</td>
    <td>Persistent Volumes 정보 관리</td>
    <td>Super Admin, Cluster Admin</td>
  </tr>
  <tr>
    <td>Persistent Volume Claims</td>
    <td>Persistent Volume Claims 정보 관리</td>
    <td>All</td>
  </tr>
  <tr>
    <td>Storage Classes</td>
    <td>Storage Classes 정보 관리</td>
    <td>Super Admin, Cluster Admin</td>
  </tr>
 <tr>
    <td rowspan="2">🔗<a href="./cp-portal-use-guide-configs.md"><b>Configs</b></a></td>
    <td>ConfigMaps</td>
    <td>ConfigMaps 정보 관리</td>
    <td rowspan="2">All</td>
  </tr>
  <tr>
    <td>Secrets</td>
    <td>Secrets 정보 관리</td>
  </tr>
  <tr>
    <td rowspan="4">🔗<a href="./cp-portal-use-guide-managements.md"><b>Managements</b></a></td>
    <td>Users</td>
    <td>사용자 관리</td>
    <td>Super Admin, Cluster Admin</td>
  </tr>
  <tr>
    <td>Roles</td>
    <td>Roles 관리</td>
    <td rowspan="3">All</td>
  </tr>
  <tr>
    <td>Resource Quotas</td>
    <td>Resource Quotas 정보 관리</td>
  </tr>
  <tr>
    <td>Limit Ranges</td>
    <td>Limit Ranges 정보 관리</td>
  </tr>
  <tr>
    <td rowspan="2">🔗<a href="./cp-portal-use-guide-chaos.md"><b>Chaos</b></a></td>
    <td>Experiments</td>
    <td>Chaos Experiments 정보 관리</td>
    <td rowspan="2">Super Admin</td>
  </tr>
  <tr>
    <td>Events</td>
    <td>Chaos Events 정보 관리</td>
  </tr>   
  <tr>
    <td rowspan="2">🔗<a href="./cp-portal-use-guide-info.md"><b>Info</b></a></td>
    <td>Access</td>
    <td>컨테이너 플랫폼 포털 <br>CLI 사용을 위한 환경 설정 정보 관리</td>
    <td rowspan="2">All</td>
  </tr>
  <tr>
    <td>Private Repository</td>
    <td>Private Repository 정보 관리</td>
  </tr>
</tbody>
</table>

<br>


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  포털 사용 가이드







