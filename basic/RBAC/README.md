# RBAC (Role-Based Access Control)

RBAC는 조직이나 개인에게 네트워크 리소스나 컴퓨터에 접근할 때, 역할(Role)기반의 접근 권한을 부여하는 방법이다.

RBAC는 `rbac.authorization.k8s.io` API 그룹을 사용하며, k8s api를 이용하여 다양한 권한제어, 정책 등을 관리한다.

RBAC를 사용하기 위해서는 **kube-apiserver**를 시작할 때 `--authorization-mode`에 추가해주면 된다.

```shell
kube-apiserver --authorization-mode=Example,RBAC # comma separated
```

## API Object

RBAC API는 Role, ClusterRole, RoleBinding, ClusterRoleBinding등 4가지 리소스를 선언할 수 있다.

다른 리소스들과 마찬가지로 kubectl을 이용하여 이를 제어할 수 있다.

### Role & RoleBinding

Role은 권한을 정의하는 오브젝트이며, RoleBinding은 정의된 Role을 특정 유저, 그룹에 연결하는 역할을 하는 오브젝트이다.

### Role and ClusterRole

**Role, ClusterRole**은 권한에 대한 내용들을 정의하며, 추가하는 방식이다. (즉, '**거부**' 설정은 없다.; 목록에 없으면 거부)

Role은 특정 namespace내의 리소스에 대한 권한을 부여하는 반면, **ClusterRole**은 namespace에 영향을 받지 않는다.

**ClusterRole**은 아래와 같은 용도가 있다.

1. 네임스페이스 리소스에 대한 권한을 정의하고, 개별 네임스페이스별 권한을 부여한다.
2. 네임스페이스 리소스에 대한 권한을 정의하고, 모든 네임스페이스에 권한을 부여한다.
3. 클러스터 리소스 전체에 권한을 정의한다.

만약, 네임스페이스에 한정된 role을 정의하려면 Role, 클러스터 전역에 role을 정의하려면 ClusterRole을 사용하면 된다.

#### Role example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default # default namespace
  name: pod-reader
rules:
  - apiGroups: [ "" ] # "" indicates the core API group
    resources: [ "pods" ]
    verbs: [ "get", "watch", "list" ]
```

위 처럼 **default** 네임스페이스의 파드들에 읽기권한을 부여할 수 있다.

#### ClusterRole example

ClusterRole을 사용하여 Role과 동일한 권한을 부여할 수 있다.

1. 클러스터 리소스 전역에 할당되는 권한
2. 리소스가 아닌 endpoints (eg: /healthz)
3. 전체 네임스페이스에 걸친 네임스페이스 귀속 리소스 (eg: Pods)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # ClusterRole은 네임스페이스 한정 오브젝트가 아니므로 namespace 필드는 생략
  name: secret-reader
rules:
  - apiGroups: [ "" ]
    resources: [ "secrets" ]
    verbs: [ "get", "watch", "list" ]
```

### RoleBinding and ClusterRoleBinding

정의된 권한을 특정 유저나 유저그룹에 부여하는 역할을 하는 오브젝트이다.

Role과 마찬가지로 RoleBinding은 특정 네임스페이스 안의 권한을 부여하는 반면,

ClusterRoleBinding은 클러스터 전역에 해당되는 권한을 부여한다.

RoleBinding은 동일 네임스페이스 내의 Role을 바인딩할 수 있을 뿐만 아니라, ClusterRole 역시 바인딩할 수 있지만

ClusterRoleBinding에 바인딩하기 위해선 반드시 ClusterRoleBinding을 참조해야한다.

#### RoleBinding examples

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects: # 여러 그룹에 권한을 바인딩할 수 있다.
  - kind: User
    name: jane # "name" 필드는 대소문자 구분.
    apiGroup: rbac.authorization.k8s.io
roleRef: # "roleRef" 바인드할 Role / ClusterRole을 정의힌다.
  kind: Role # 반드시 Role 또는 ClusterRole
  name: pod-reader # 바인드할 대상의 이름. 같은 네임스페이스에 미리 선언이 되어있어야한다.
  apiGroup: rbac.authorization.k8s.io
```

RoleBinding은 ClusterRole도 바인딩할 수 있다.

클러스터 전체에서 공통 Role집합을 정의한 다음 여러 네임스페이스 내에서 해당 Role을 재사용할 수 있다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-secrets
  # namespace 필드를 통해 RoleBinding이 어떤 네임스페이스에 권한을 부여할지 결정한다.
  namespace: development # "development" namespace내의 리소스에 한정한다.
subjects:
  - kind: User
    name: dave
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole # ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

기존에 생성되어 있는 **secret-reader**라는 ClusterRole을 참조하여 **development** namespace의 **dave**라는 User에게 권한을 바인딩한다.

#### ClusterRoleBinding example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
  - kind: Group
    name: manager
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

위의 ClusterRoleBinding은 **manager** Group에 모든 유저에게 미리 생성된 **secret-reader** 권한을 바인딩한다.

바인딩 생성 후에는 바인딩이 참조하는 Role 또는 ClusterRole을 변경할 수 없다. 만약 바인딩의 `roleRef` 필드를 변경하려고 하면 오류가 발생한다.

`roleRef`필드를 변경하려면 바인딩 오브젝트를 제거하고 대체할 수 있는 오브젝트를 생성해야한다.

1. Role이나 RoleBinding의 변경으로 기존 바인딩에 참조된 대상들이 필요이상의 권한을 부여받을 수 있게된다.
2. 서로 다 역할에 대한 바인딩은 기본적으로 다른 바인딩이다. 따라서 새로운 Role에대한 권한을 부여할 경우 새로운 바인딩을 생성(삭제/재작성)하여 권한을 부여하는 것이 의도된 동작이다.

`kubectl auth reconcile` 명령을 통해 RBAC 오브젝트나 관련 메니페스트파일을 생성/수정할 수 있다.

### 리소스 참조

RBAC를 이용하여 기본 resource뿐만 아니라, 세부 resource까지 권한을 부여할 수 있다.

```text
GET /api/v1/namespaces/{namespace}/pods/{name}/log
```

위 처럼 `pods`는 네임스페이스 영역에 있는 오브젝트이다. 위에서 설명했던 것처럼 resource를 이용하여 User에게 권한을 부여할 수 있다.

추가적으로 특정 `pod`내의 로그를 확인하는 부분에 권한을 부여하고 싶다면, resource를 추가하여 가능하다.

즉, 위에서 `log`는 `pod`라는 resource의 subresource이다. 이 관계를 확인하고 메니페스트에 추가하여 권한을 부여할 수 있다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-and-pod-logs-reader
rules:
  - apiGroups: [ "" ]
    resources: [ "pods", "pods/log" ] # pods와 log를 '/'(슬래쉬)로 구분한다.
    verbs: [ "get", "list" ]
```

뿐만 아니라, 리소소의 이름을 이용하여 특정 오브젝트만 권한을 부여할 수 있다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: configmap-updater
rules:
  - apiGroups: [ "" ]
    resources: [ "configmaps" ]
    resourceNames: [ "my-configmap" ] # configmap중 my-configmap만 권한을 부여한다.
    verbs: [ "update", "get" ]
``` 

만약 위처럼 선언하기 위해서는 반드시 생성되어 있는 오브젝트, configMap의 `metadata.name`필드가 my-configmap으로 선언되어 있어야 한다.

또한, `create` 또는 `deletecollection` 액션은 권한부여가 불가능하다. 당연히 오브젝트 생성에 있어서 리소스이름을 판단할 수 없기 때문이다.

### 통합 ClusterRole

몇몇개의 ClusterRole을 통합한 ClusterRole을 생성할 수 있다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring
aggregationRule:
  clusterRoleSelectors:
    - matchLabels:
        rbac.example.com/aggregate-to-monitoring: "true"
rules: [ ] # The control plane automatically fills in the rules
```

`aggregationRule`필드를 이용하여 조건에 맞는 ClusterRole을 통합할 수 있다.

일반적인 오브젝트들과 동일하게 이 작업이 성립하기 위해선 label을 사용한다.

사전에 통합대상 ClusterRole을 생성하거나 수정을 통해 label을 세팅해두면 이를 감지하여 통합 ClusterRole을 생성할 수 있다.

위에서는 `rbac.example.com/aggregate-to-monitoring: true` label을 추가해준다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-endpoints
  labels:
    rbac.example.com/aggregate-to-monitoring: "true" # 통합 ClusterRole을 위한 값
rules:
  - apiGroups: [ "" ]
    resources: [ "services", "endpoints", "pods" ]
    verbs: [ "get", "list", "watch" ]
```

label을 이용하면 같은 리소스라도 다양한 룰을 적용시킬 수 있다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: aggregate-cron-tabs-edit
  labels:
    # Add these permissions to the "admin" and "edit" default roles.
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
rules:
  - apiGroups: [ "stable.example.com" ]
    resources: [ "crontabs" ]
    verbs: [ "get", "list", "watch", "create", "update", "patch", "delete" ]
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: aggregate-cron-tabs-view
  labels:
    # Add these permissions to the "view" default role.
    rbac.authorization.k8s.io/aggregate-to-view: "true"
rules:
  - apiGroups: [ "stable.example.com" ]
    resources: [ "crontabs" ]
    verbs: [ "get", "list", "watch" ]
```

위처럼 `crontabs`오브젝트를 크게 두가지의 권한으로 나눌 수 있다.

상단에서는 거의 전체 권한을 부여하되 label을 통해 통합 ClusterRole을 세팅할 수 있도록 admin, edit에 값을 주었고,

하단에서는 읽기권한만을 부여하며 view라는 label을 적용하였다.

이를 통해 좀 더 수월한 통합 ClusterRole세팅이 가능하다. (단, label은 metadata일뿐 강력한 권한은 없다.)

#### example

```yaml
rules:
  - apiGroups: [ "" ]
    resources: [ "pods" ]
    verbs: [ "get", "list", "watch" ]
```

전체 파드에 읽기권한을 주는 Role

```yaml
rules:
  - apiGroups: [ "apps" ]
    resources: [ "deployments" ]
    verbs: [ "get", "list", "watch", "create", "update", "patch", "delete" ]
```

deployment에 읽기/쓰기 권한을 주는 Role

```yaml
rules:
  - apiGroups: [ "" ]
    resources: [ "pods" ]
    verbs: [ "get", "list", "watch" ]
  - apiGroups: [ "batch" ]
    resources: [ "jobs" ]
    verbs: [ "get", "list", "watch", "create", "update", "patch", "delete" ]
```

pods에 읽기권한을 부여하며, jobs에 읽기/쓰기 권한을 주는 Role

```yaml
rules:
  - apiGroups: [ "" ]
    resources: [ "configmaps" ]
    resourceNames: [ "my-config" ]
    verbs: [ "get" ]
```

my-config라는 ConfigMap에 읽기권한만을 주는 Role

```yaml
rules:
  - apiGroups: [ "" ]
    resources: [ "nodes" ]
    verbs: [ "get", "list", "watch" ]
```

node에 읽기권한만 부여하는 ClusterRole (node는 non-namespaced)

```yaml
rules:
  - nonResourceURLs: [ "/healthz", "/healthz/*" ]
    verbs: [ "get", "post" ]
```

nonResource인 /healthz및 이하 도메인에 get, post권한만 부여하는 ClusterRole (api는 non-namespaced)

### subjects 참조

생성된 Role이나 ClusterRole은 Binding Resource를 통해 특정유저, 그룹, SA에 바인딩된다.

k8s에서 username이나 그룹이름은 단순 문자열, email형식, 숫자형식등의 문자열로 구성된다. (eg. "alice", "alice@example.com", "12341")

단, prefix `system:`은 예약어이다.

SA(ServiceAccount)는 단일계정일 경우 `system:serviceaccount:`(단수)를 그룹일 경우 `system:serviceaccounts:`(복수)를 prefix로 사용한다.

#### example

```yaml
subjects:
  - kind: User
    name: "alice@example.com"
    apiGroup: rbac.authorization.k8s.io
```

`alice@example.com`유저에게 바인딩한다.

```yaml
subjects:
  - kind: Group
    name: "frontend-admins"
    apiGroup: rbac.authorization.k8s.io
```

`frontend-admins` 그룹에게 바인딩한다.

```yaml
subjects:
  - kind: ServiceAccount
    name: default
    namespace: kube-system
```

`kube-system` 네임스페이스의 `default` SA에 바인딩한다.

```yaml
subjects:
  - kind: Group
    name: system:serviceaccounts:qa
    apiGroup: rbac.authorization.k8s.io
```

`system:serviceaccounts:qa` SA에 바인딩한다.

```yaml
subjects:
  - kind: Group
    name: system:serviceaccounts
    apiGroup: rbac.authorization.k8s.io
```

모든 SA그룹에 바인딩한다.

```yaml
subjects:
  - kind: Group
    name: system:authenticated
    apiGroup: rbac.authorization.k8s.io
```

모든 인증된 유저(`system:authenticated`)에게 바인딩한다.

```yaml
subjects:
  - kind: Group
    name: system:unauthenticated
    apiGroup: rbac.authorization.k8s.io
```

모든 인증되지 않은 유저(`system:unauthenticated`)에게 바인딩한다.

```yaml
subjects:
  - kind: Group
    name: system:authenticated
    apiGroup: rbac.authorization.k8s.io
  - kind: Group
    name: system:unauthenticated
    apiGroup: rbac.authorization.k8s.io
```

모든 저에게 바인딩한다.
