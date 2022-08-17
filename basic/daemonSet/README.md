# DaemonSet

모든 노드가 파드를 실행할 수 있도록 보장하는 리소스.

### 용도

1. 모든 노드에서 클러스터 스토리지 데몬 실행
2. 모든 노드에서 로그 수집 데몬 실행
3. 모든 노드에서 노드 모니터링 데몬 실행

## Menifest 

[Damonset.yaml](./daemonset.yaml)

### 필수 필드

`apiVersion`, `kind`, `metadata`는 다른 메니페스트와 동일하게 필수적으로 필요하다.

데몬셋에서 관리할 리소스를 정의하기 위해 `spec`필드를 반드시 정의해야 한다.

### 파드 템플릿

`spec.template`에 관리할 파드를 정의한다.

Pod의 메니페스트와 동일한 스키마를 갖는다.

단 `restartPolicy`는 항상 **Always**를 가져야한다. (항상 실행을 보장하기 위해)

### 파드 셀렉터

`spec.selector`에 데몬셋이 관리할 파드와 일치되는 label을 정의한다.

따라서 위의 값은 반드시 `spec.template.metadata.labels`와 일치해야 한다.

### 특정 노드 지정

`spec.template.spec.nodeSelector`를 정의하면 데몬셋 컨트롤러는 일치하는 노드에만 파드를 생성한다.

`spec.template.spec.affinity` 역시 위와 동일한 동작을 기대하며, 둘 다 정의되지 않았다면 모든 노드에 파드를 실행한다.

## 스케줄링

데몬셋을 통해 생성되는 파드는 데몬셋 컨트롤러를 통해 스케줄링된다.

* 데몬셋 파드는 **Pending**상태로 생성되지 않는다.
* 파드는 스케줄러에 의해 선점되고 스케줄링되지만, 데몬셋 파드는 이를 고려하지 않고 스케줄링된다.
하지만, `nodeAffinity` 스키마를 이용하여 데몬셋 컨트롤러가 아닌 기본 스케줄러를 통해 스케줄링할 수 있다.
* 데몬셋 파드에는 `node.kubernetes.io/unschedulable:NoSchedule` 톨러레이션이 자동으로 추가된다.
위 과정을 통해 기본 스케줄러가 스케줄링하게되면 **unschedulable**인 노드를 무시한다.

### taint & toleration

데몬셋 파드는 자동적으로 톨러레이션이 추가된다.

1. `node.kubernetes.io/not-ready:NoExecute`

    네트워크 문제등의 노드 문제가 발생해도 데몬셋 파드는 축출되지 않는다.

2. `node.kubernetes.io/unreachable:NoExecute`

   네트워크 문제등의 노드 문제가 발생해도 데몬셋 파드는 축출되지 않는다.

3. `node.kubernetes.io/disk-pressure:NoSchedule`

    기본 스케줄러에서 디스크압박 속성을 허용한다.

4. `node.kubernetes.io/pid-pressure:NoSchedule`

    기본 스케줄러에서 메모리압박 속성을 허용한다.

5. `node.kubernetes.io/unschedulable:NoExecute`

    기본 스케줄러의 **스케줄할 수 없는(unschedulable) 속성**을 극복한다.

6. `node.kubernetes.io/network-unavailable:NoSchedule`

    호스트 네트워크를 사용하는 데몬셋 파드는 기본 스케줄러에 의해 **이용할 수 없는 네트워크 속성**을 극복한다.

## 데몬셋 파드와 통신

1. push

2. node IP와 known PORT

    호스트 포트를 사용하며 노드 IP를 이용하여 접근가능.

3. DNS

    동일한 파드 셀렉터로 **헤드리스 서비스**를 만들고 `A RECORD`를 통해 접근한다.

4. 서비스

    **서비스**를 사용하여 임의의 노드의 데몬에 도달한다.

## 데몬셋 업데이트

만약 노드 레이블이 변경되면(데몬셋이 대상으로 하는 노드의 조건이 변경되면), 새로운 노드에 즉시 파드를 추가하고
기존 노드에서 파드를 삭제한다.

## 대안

데몬셋을 사용하지 않을 경우 대체제로 사용할 수 있는 리소스가 존재한다.

### 초기화 스크립트

데몬 프로세스를 직접 노드에서 실행시킬 수 있다. 하지만 데몬셋을 이용하면 몇가지 이점이 있다.

1. 기존 APP(파드 등)과 동일한 방법으로 모니터링, 로그관리를 할 수 있다.
2. 기존 APP(파드 등)과 동일한 구성언어와 도구를 사용할 수 있다.
3. 기존 APP(파드 등)과 동일하게 컨테이너에서 데몬을 실행하여 격리도를 낮출 수 있다.
 
### Bare 파드

직접 파드 매니페스트를 작성하여 각 노드마다 수동으로 파드를 생성할 수 있다.

하지만, 이 방법은 유지보수등의 부작용이 많다.

### 스태틱(Static) 파드

kubelet이 감시하는 디렉토리에 매니페스트를 생성하여 파드를 생성할 수 있다(컨트롤러, etcd 등).

하지만 kubectl등의 관리도구로 리소스를 관리할 수 없다.

### Deployment

레플리카를 관리한다는 점에선 두 리소스는 매우 유사하다. 하지만 파드 사본이 노드 호스트들에 종속적이라면

데몬셋을 사용하는 것이 좋다. deployment를 통한 생성은 node에 종속적이지 않다(임의 생성).
