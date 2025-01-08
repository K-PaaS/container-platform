### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Chaos 메뉴

<br>

## Table of Contents

1. [Chaos 메뉴](#1)  
  1.1. [Experiments](#1-1)  
  1.1.1. [Experiment 목록 조회](#1-1-1)  
  1.1.2. [Experiment 생성](#1-1-2)  
  1.1.3. [Experiment 상세 조회](#1-1-3)  
  1.1.4. [Experiment 삭제](#1-1-4)  
  1.2. [Events](#1-2)   
  1.2.1. [Event 조회](#1-2-1)  
<br>

## <div id='1'/> 1. Chaos 메뉴
Chaos는 다양한 유형의 실험을 제공하여 운영 환경에서 실제 발생할 수 있는 이상(파드 종료, 네트워크 지연 등)을 시뮬레이션하고 실험 상태를 모니터링 할 수 있다.
실험의 종류로는 'Pod Fault', 'Network Attack', 'Stress Test'가 있다.

### <div id='1-1'/> 1.1. Experiments
#### <div id='1-1-1'/> 1.1.1. Experiment 목록 조회
- Chaos Experiment 목록을 조회한다.
  ![IMG_10_1_1_1]

<br>

#### <div id='1-1-2'/> 1.1.2. Experiment 생성
- Experiment 목록에서 생성 버튼을 클릭하여 Chaos Experiment 생성 페이지로 이동한다.
- Chaos Experiment 생성 페이지에서 'Pod Fault', 'Network Attack', 'Stress Test' 탭에 따라 각각 다른 종류의 Chaos Experiment을 생성할 수 있다.

<br>

- Pod Fault
  - Pod가 삭제된 후 복구가 되는지 확인하는 실험이다.
    ![IMG_10_1_1_2_1]

    <table>
    <thead>
      <tr>
        <th>항목</th>
        <th>설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Name</td>
        <td>Pod Fault 실험의 이름. 소문자와 숫자 조합 또는 소문자만 입력 가능하다.</td>
      </tr>
      <tr>
        <td>Namespace</td>
        <td>Pod Fault 실험을 위한 카오스 리소스가 만들어지는 Namespace를 선택한다.</td>
      </tr>
      <tr>
        <td>Grace period</td>
        <td>선택 사항이며 Pod가 정상적으로 종료될 때까지 주어진 기간(초)이다. 기본값은 0이며, 0이상 숫자만 입력 가능하다.</td>
      </tr>
      <tr>
        <td>Namespace Selector</td>
        <td>Pod Fault 실험 대상 Pod가 속한 Namespace를 지정한다.</td>
      </tr>
      <tr>
        <td>Label Selector</td>
        <td>Pod Fault 실험 대상 Pod가 가진 Label Selector를 지정한다.</td>
      </tr>
      <tr>
        <td>Preview of Pods to be injected</td>
        <td>Pod Fault 실험 대상 Pod를 최대 6개까지 선택할 수 있다.</td>
      </tr>
    </tbody>
    </table>

<br>

- Network Attack
  - 네트워크 지연을 발생시키는 실험이다.
    ![IMG_10_1_1_2_2]

    <table>
    <thead>
      <tr>
        <th>항목</th>
        <th>설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Name</td>
        <td>Network Attack 실험 이름. 소문자와 숫자 조합 또는 소문자만 입력 가능하다.</td>
      </tr>
      <tr>
        <td>Namespace</td>
        <td>Network Attack 실험을 위한 카오스 리소스가 만들어지는 Namespace를 선택한다.</td>
      </tr>
      <tr>
        <td>Latency</td>
        <td>네트워크 지연을 얼마나 발생시킬지 숫자를 입력 후 시간 단위를 선택한다.</td>
      </tr>
      <tr>
        <td>Duration</td>
        <td>카오스 실험이 얼마동안 진행할지 숫자를 입력 후 시간 단위를 선택한다. 최대 12시간까지 실험이 가능하다.</td>
      </tr>
      <tr>
        <td>Namespace Selector</td>
        <td>Network Attack 실험 대상 Pod가 속한 Namespace를 지정한다.</td>
      </tr>
      <tr>
        <td>Label Selector</td>
        <td>Network Attack 실험 대상 Pod가 가진 Label Selector를 지정한다.</td>
      </tr>
      <tr>
        <td>Preview of Pods to be injected</td>
        <td>Network Attack 실험 대상 Pod를 최대 6개까지 선택할 수 있다.</td>
      </tr>
    </tbody>
    </table>

<br>

-  Stress Test
  - 선택한 Pod에 CPU, Memory를 할당하여 부하를 주는 실험이다.
    ![IMG_10_1_1_2_3]
- CPU와 Memory 둘 다 부하를 주거나, 둘 중 하나를 선택해서 부하를 줄 수 있다.
- Duration이 30초 이상일 경우 메트릭이 수집되어 차트로 볼 수 있다.
- 리소스 상태가 정상적이지 않을 경우 메트릭이 원활하게 수집되지 않을 수 있다.

    <table>
    <thead>
      <tr>
        <th>항목</th>
        <th>설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Name</td>
        <td>Stress Test 실험 이름. 소문자와 숫자 조합 또는 소문자만 입력 가능하다.</td>
      </tr>
      <tr>
        <td>Namespace</td>
        <td>Stress Test 실험을 위한 카오스 리소스가 만들어지는 Namespace를 선택한다.</td>
      </tr>
      <tr>
        <td>CPU Workers</td>
        <td>CPU 부하를 지원할 스레드 수를 지정한다.</td>
      </tr>
      <tr>
        <td>CPU Load</td>
        <td>CPU가 차지할 비율 지정한다. CPU 부하 최종합계 = workers * load</td>
      </tr>
      <tr>
        <td>Memory Workers</td>
        <td>Memory 부하를 지원할 스레드 수를 지정한다.</td>
      </tr>
      <tr>
        <td>Memory Size</td>
        <td>1 Worker당 소비될 메모리 크기. 기본값은 총 이용가능한 Memory 크기.</td>
      </tr>
      <tr>
        <td>Duration</td>
        <td>카오스 실험이 얼마동안 진행할지 숫자를 입력 후 시간 단위를 선택한다. 최대 12시간까지 실험이 가능하다. 30초미만일 경우에도 카오스 실험은 진행되지만, 메트릭 수집이 불가하다.</td>
      </tr>
      <tr>
        <td>Namespace Selector</td>
        <td>Stress Test 실험 대상 Pod가 속한 Namespace를 지정한다.</td>
      </tr>
      <tr>
        <td>Label Selector</td>
        <td>Stress Test 실험 대상 Pod가 가진 Label Selector를 지정한다.</td>
      </tr>
      <tr>
        <td>Preview of Pods to be injected</td>
        <td>Stress Test 실험 대상 Pod를 최대 6개까지 선택할 수 있다.</td>
      </tr>
    </tbody>
    </table>

<br>

#### <div id='1-1-3'/> 1.1.3. Experiment 상세 조회
- Experiment 목록에서 Experiment명을 클릭하여 Experiment 상세 페이지로 이동한다.
  ![IMG_10_1_1_3_1]
  
<br>

- Stress Test Experiment일 경우 모니터링할 수 있는 차트를 추가로 볼 수 있다.
  - 차트의 리소스는 최대 6개까지 보여진다. 
  - 대상 리소스 중 HPA 설정이 되어 있는 리소스가 있을 경우, 첫 번째 Metric인 'Resource usage by selected Pods during chaos'에서 HPA에 의해 Metric이 수집되는 파드 수가 증가하여 Chaos Experiment에서 선택한 모든 파드의 차트가 보이지 않을 수 있다.(차트의 리소스는 최대 6개까지 보여진다.)
  - Chaos Experiment에 의해 부하를 받은 Pod에 접근이 정상적이지 않을 경우 APP Status에서 빈칸으로 표시되고, 정상적으로 접근이 될 경우 색칠한 칸으로 표시된다.
  ![IMG_10_1_1_3_2]

<br>

#### <div id='1-1-4'/> 1.1.4. Experiment 삭제
- Experiment 상세 페이지 하단의 삭제 버튼을 클릭하여 Experiment를 삭제한다.
  ![IMG_10_1_1_4_1]

<br>

### <div id='1-2'/> 1.2. Events
#### <div id='1-2-1'/> 1.2.1. Event 조회
- Chaos Event를 조회한다.
  ![IMG_10_1_2_1]

<br>


### [Index](https://github.com/K-PaaS/container-platform/blob/master/README.md) > [CP Use](../Readme.md) >  [포털 사용 가이드](./cp-portal-use-guide.md) > Chaos 메뉴

[IMG_10_1_1_1]:../images/portal/IMG_10_1_1_1.png
[IMG_10_1_1_2_1]:../images/portal/IMG_10_1_1_2_1.png
[IMG_10_1_1_2_2]:../images/portal/IMG_10_1_1_2_2.png
[IMG_10_1_1_2_3]:../images/portal/IMG_10_1_1_2_3.png
[IMG_10_1_1_3_1]:../images/portal/IMG_10_1_1_3_1.png
[IMG_10_1_1_3_2]:../images/portal/IMG_10_1_1_3_2.png
[IMG_10_1_1_4_1]:../images/portal/IMG_10_1_1_4_1.png
[IMG_10_1_2_1]:../images/portal/IMG_10_1_2_1.png

