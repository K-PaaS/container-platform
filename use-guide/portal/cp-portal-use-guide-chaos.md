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
        <td>Pod Fault 실험 이름. 소문자와 숫자 조합 또는 소문자만 입력 가능하다.</td>
      </tr>
      <tr>
        <td>Namespace</td>
        <td>Pod Fault 실험이 진행될 Namespace를 선택한다.</td>
      </tr>
      <tr>
        <td>Grace period</td>
        <td>선택 사항이며 입력하지 않을 경우 기본값은 0이다. Pod가 정상적으로 종료될 때까지 주어진 기간(초)으로 0이상 숫자만 입력 가능하다.</td>
      </tr>
      <tr>
        <td>Namespace Selector</td>
        <td>Pod Fault 실험을 할 Pod가 속한 Namespace를 지정한다.</td>
      </tr>
      <tr>
        <td>Label Selector</td>
        <td>Pod Fault 실험을 할 Pod가 속한 Label Selector 지정한다.</td>
      </tr>
      <tr>
        <td>Preview of Pods to be injected</td>
        <td>Pod Fault 실험을 할 Pod를 최대 6개까지 선택할 수 있다.</td>
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
        <td>Network Attack 실험이 진행될 Namespace를 선택한다.</td>
      </tr>
      <tr>
        <td>Latency</td>
        <td>네트워크 지연시간을 입력한다.</td>
      </tr>
      <tr>
        <td>Duration</td>
        <td>카오스가 지속될 시간을 입력한다. 최대 12시간까지 실험이 가능하다.</td>
      </tr>
      <tr>
        <td>Namespace Selector</td>
        <td>Network Attack 실험을 할 Pod가 속한 Namespace를 지정한다.</td>
      </tr>
      <tr>
        <td>Label Selector</td>
        <td>Network Attack 실험을 할 Pod가 속한 Label Selector 지정한다.</td>
      </tr>
      <tr>
        <td>Preview of Pods to be injected</td>
        <td>Network Attack 실험을 할 Pod를 최대 6개까지 선택할 수 있다.</td>
      </tr>
    </tbody>
    </table>

<br>

-  Stress Test
  - 선택한 Pod에 CPU, Memory를 할당하여 부하를 주는 실험이다.
    ![IMG_10_1_1_2_3]
- CPU와 Memory 둘 다 부하를 주거나, 둘 중 하나에 부하를 줄 수 있다.
- Stress Test는 Duration이 30초 이상일 경우 메트릭이 수집되어 차트로 볼 수 있다.
- 리소스 상태가 정상적이지 않을 경우 메트릭이 정상적으로 수집이 되지 않을 수 있다.

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
        <td>Stress Test 실험이 진행될 Namespace를 선택한다.</td>
      </tr>
      <tr>
        <td>CPU Workers</td>
        <td>CPU 부하를 지원하는 스레드 수를 지정한다.</td>
      </tr>
      <tr>
        <td>CPU Load</td>
        <td>CPU가 차지하는 비율 지정. CPU 부하 최종합계 = workers * load</td>
      </tr>
      <tr>
        <td>Memory Workers</td>
        <td>Memory 부하를 지원하는 스레드 수를 지정한다.</td>
      </tr>
      <tr>
        <td>Memory Size</td>
        <td>1 Worker당 소비되는 메모리 크기. 기본값은 총 이용가능한 Memory.</td>
      </tr>
      <tr>
        <td>Duration</td>
        <td>카오스가 지속될 시간을 입력한다. 최대 12시간까지 실험이 가능하다. 30초미만일 경우에도 카오스 실험이 생성되지만, 메트릭 수집이 불가하다.</td>
      </tr>
      <tr>
        <td>Namespace Selector</td>
        <td>Stress Test 실험을 할 Pod가 속한 Namespace를 지정한다.</td>
      </tr>
      <tr>
        <td>Label Selector</td>
        <td>Stress Test 실험을 할 Pod가 속한 Label Selector 지정한다.</td>
      </tr>
      <tr>
        <td>Preview of Pods to be injected</td>
        <td>Stress Test 실험을 할 Pod를 최대 6개까지 선택할 수 있다.</td>
      </tr>
    </tbody>
    </table>

<br>

#### <div id='1-1-3'/> 1.1.3. Experiment 상세 조회
- Experiment 목록에서 Experiment명을 클릭하여 Experiment 상세 페이지로 이동한다.
  ![IMG_10_1_1_3_1]
  
<br>

- Stress Test Experiment일 경우 모니터링할 수 있는 차트를 추가로 볼 수 있다.
- 대상 리소스가 HPA 설정이 되어있을 경우
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

