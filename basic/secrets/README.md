# Secret

## Secret 오브젝트 생성

### 명령적 생성법

cli 명령을 통한 오브젝트 생성.

1. 기본 생성 명령

   ```shell
   $ kubectl create secret generic <secret-name>
   ```
   
2. Data를 포함한 생성 명령

   1. `--from-literal`
   
      ```shell
      $ kubectl create secret generic <secret-name> \
         --from-literal=<KEY>=<DATA> \
         --from-literal=<KEY1>=<DATA1> # 각 키마다 조건 추가.
      ```
   2. `--from-file`

      ```shell
      $ kubectl create secret generic <secret-name> \
         --from-file=<filePath> \
         --from-file=<filePath1> # 각 키마다 파일 추가.
      ```
      
      ```shell
      $ cat USERNAME
      ZGV2dXNlcg== # Base64 encoded
      ```
      
      File Path를 이용하여 폴더 내의 파일들을 자동으로 generate할 수 있다.
   
      ```shell
      $ ls ./env
      PASSWORD USERNAME
      
      $ kubectl create secret generic <secret-name> --from-file=./env
      ```

### 선언적 생성법

`yaml 파일`을 통한 오브젝트 생성.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-demo
data: # spec 대신 data섹션이 있다.
  PASSWORD: <string>
  USERNAME: <string>
```

## 조회

secret을 조회합니다.

### secret 목록 조회

```shell
$ kubectl get secret

NAME                  TYPE           DATA   AGE
secret-demo-1         Opaque         0      26m
secret-demo-2         Opaque         2      7s
secret-demo-3         Opaque         2      7s
secret-demo-4         Opaque         2      7s
```

### secret 상세 정보

```shell
$ kubectl describe secret secret-demo
```

```shell
Name:         secret-demo
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  12 bytes
username:  12 bytes
```

위 처럼 조회하면 secret데이터가 숨겨있다.

### yaml 형태로 조회

```shell
$ kubectl get secret secret-demo -o yaml
```

```shell
apiVersion: v1
kind: Secret
metadata:
  name: secret-demo
  namespace: default
data:
  password: <string>
  username: <string>
type: Opaque
```

## Pod에 Secret 적용

secret에 세팅된 데이터를 pod에 적용하는 방법.

### secret 설정

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
  labels:
    run: pod-secret
spec:
  containers:
    - image: nginx
      name: pod-secret
      envFrom: # container에 동작하는 app에 환경변수 [List]
        - secretRef:
            name: secret-demo # 사전에 세팅된 secret이름 
```

```shell
$ kubectl exec pod-secret -it env

...
USERNAME=ZGV2dXNlcg==
PASSWORD=cGFzc3dvcmQ=
...
```

```shell
$ kubectl describe po pod-secret

# description
    ...
    Environment Variables from:
      secret-demo  Secret  Optional: false
    ...
# Event
Variables from:  Normal  Scheduled  4m6s   default-scheduler  Successfully assigned default/pod-secret to minikube
```

### 단일 키 설정

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret2
  labels:
    run: pod-secret2
spec:
  containers:
    - image: nginx
      name: pod-secret2
      env:
        - name: USER_NAME  # container에서 사용할 환경변수 이름
          valueFrom:
             secretKeyRef:
              name: secret-demo-2
              key: USERNAME # secret에 세팅된 키 이름
```

```shell
$ kubectl exec pod-secret2 -it env

...
USERNAME=ZGV2dXNlcg==
...
```

```shell
$ kubectl describe po pod-secret2

# description
    ...
    Environment:
      USER_NAME:  <set to the key 'USERNAME' in secret 'secret-demo-2'>  Optional: false
    ...
# Event
  Normal  Scheduled  33s   default-scheduler  Successfully assigned default/pod-secret2 to minikube
```

### 볼륨 마운트

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret3
  labels:
    run: pod-secret3
spec:
  containers:
  - image: nginx
    name: pod-secret3
    volumeMounts: # 컨테이너에 마운트할 볼륨
      - name: secret-vol # 마운트할 볼륨 이름
        mountPath: "/etc/foo" # 볼륨 경로
  volumes:
    - name: secret-vol # volume 이름
      secret:
        secretName: secret-demo-3 # 마운트할 secret 이름
```

```shell
$ kubectl exec pod-secret3 -- ls /etc/foo
PASSWORD  USERNAME

$ kubectl exec pod-secret3 -- env
# 파일로 마운트 했기 때문에 환경변수에는 없다.
```

```shell
$ kubectl describe po pod-configmap3

# description
  ...
  Mounts:
    /etc/foo from secret-vol (rw)
  ...
   Volumes:
     secret-vol:
       Type:        Secret (a volume populated by a Secret)
       SecretName:  secret-demo-3
       Optional:    false
  ...
```

## 응용

### 단일 volumeMount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret4
  labels:
    run: pod-secret4
spec:
  containers:
    - image: nginx
      name: pod-secret4
      volumeMounts:
        - name: secret-demo
          mountPath: "/config"  # 마운트한 볼륨 경로
  volumes:  # 파드 레벨에서 볼륨을 설정한 후, 컨테이너에 마운트.
    - name: secret-demo
      secret:
        secretName: secret-demo-4
        items:  # secret에서 파일로 생성할 키
          - key: USERNAME # secret key 이름
            path: "USER_NAME" # 마운트된 볼륨에 생성될 파일이름
```

```shell
$ kubectl exec pod-secret4 -- ls config
USER_NAME
```

### 숨김파일 (dotfile)

키 이름에 `.(도트)`를 선행하면 컨테이너에 생성되는 파일은 숨김처리 할 수 있다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-demo-1
data:
  .key: dmFsdWUtMg0KDQo=
```

```shell
$ kubectl exec pod-secret5 -- ls config
# 숨겨져서 안보인다.

$ kubectl exec pod-secret5 -- ls -la config
drwxrwxrwt 3 root root  100 Jul 25 15:45 .
drwxr-xr-x 1 root root 4096 Jul 25 15:45 ..
drwxr-xr-x 2 root root   60 Jul 25 15:45 ..2022_07_25_15_45_31.1760309582
lrwxrwxrwx 1 root root   32 Jul 25 15:45 ..data -> ..2022_07_25_15_45_31.1760309582
lrwxrwxrwx 1 root root   11 Jul 25 15:45 .key -> ..data/.key
```

### immutable

configMap과 동일하게 immutable세팅을 할 수 있다.

마찬가지로 `edit`명령을 통한 수정이 불가능하다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-demo-2
data:
  password: Y0dGemMzZHZjbVE9
  username: WkdWMmRYTmxjZz09
immutable: true
```

## 타입

### 불투명 (Opaque)

기본 타입. cli명령을 통한 생성시 `generic` 옵션을 통해 생성가능

```shell
$ kubectl create secret generic <secret-name>
```

### 서비스 어카운트 토큰 

`kubernetes.io/service-account-token` 타입

```shell
Name:         <name>
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: <uuid>

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1111 bytes
namespace:  7 bytes
token: <string>
```

