# ConfigMap

## ConfigMap 오브젝트 생성

### 명령적 생성법

cli 명령을 통한 오브젝트 생성.

1. 기본 생성 명령

   ```shell
   $ kubectl create configmap <configmap1>
   ```
   
2. Data를 포함한 생성 명령

    1. `--from-literal`
       ```shell
       $ kubectl create configmap <configmap2> --from-literal=<KEY>=<VALUE> \
          --from-literal=<KEY2>=<VALUE2> # 각 키마다 조건 추가.
       ```
    2. `--from-file`
       ```shell
       $ kubectl create configmap <configmap3> --from-file=<filePath> \
          --from-file=<filePath> # 각 키마다 파일 추가.
       ```
       ```shell
       $ cat COLOR
       RED
       ```

       File Path를 이용하여 폴더 내에 파일들을 자동으로 generate할 수 있다.
       ```shell
       $ ls ./env
       BG COLOR
       
       $ kubectl create configmap <configmap4> --from-file=./env
       ```

### 선언적 생성법

`yaml 파일`을 통한 오브젝트 생성.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-demo-5
data: # 일반 spec섹션 대신 data섹션이 있다.
  BG: BLACK
  COLOR: BLUE
```

## Pod에 configMap 적용

configMap에 세팅된 환경변수 등의 데이터를 pod에 적용하는 방법.

### configMap 설정

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap1
  labels:
    run: pod-configmap1
spec:
  containers:
    - image: nginx
      name: pod-configmap1
      envFrom: # container에 동작하는 app에 환경변수 [List]
        - configMapRef:
            name: configmap-demo-4 # 사전에 세팅된 configMap이름
```
```shell
$ kubectl describe po pod-configmap1

# description
    ...
    Environment Variables from:
      configmap-demo-4  ConfigMap  Optional: false
    ...
```
### 단일 키 설정

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap2
  labels:
    run: pod-configmap2
spec:
  containers:
    - image: nginx
      name: pod-configmap2
      env: # 환경변수
        - name: FONT_COLOR # container에서 사용할 환경변수 이름
          valueFrom:
            configMapKeyRef:
              name: configmap-demo-4
              key: COLOR # configMap에 세팅된 환경변수 이름
```
```shell
$ kubectl describe po pod-configmap2

# description
    ...
    Environment:
      FONT_COLOR:  <set to the key 'COLOR' of config map 'configmap-demo-4'>  Optional: false
    ...
```

### 볼륨 마운트

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap3
  labels:
    run: pod-configmap3
spec:
  containers:
  - image: nginx
    name: pod-configmap3
    volumeMounts: # 컨테이너에 마운트할 볼륨
      - name: config-map-vol # 마운트할 볼륨 이름
        mountPath: "/etc/foo" # 볼륨 경로
  volumes:
    - name: config-map-vol # volume 이름
      configMap:
        name: configmap-demo-4 # 마운트할 configMap이름
```
```shell
$ kubectl describe po pod-configmap3

# description
    ...
    Mounts:
      /etc/foo from config-map-vol (rw)
    ...
    Volumes:
      config-map-vol:
        Type:      ConfigMap (a volume populated by a ConfigMap)
        Name:      configmap-demo-4
    ...
```

## 조회

configMap을 조회합니다.

```shell
$ kubectl describe configMap <configmap-demo-4>
```

```shell
Name:         configmap-demo-4
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
BG:
----
BLACK

COLOR:
----
BLUE


BinaryData
====

Events:  <none>
```

## 응용

### 단일 volumeMount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap4
  labels:
    run: pod-configmap4
spec:
  containers:
    - image: nginx
      name: pod-configmap4
      volumeMounts:
        - name: config-map-demo
          mountPath: "/config" # 마운트한 볼륨 경로
  volumes: # 파드 레벨에서 볼륨을 설정한 후, 컨테이너에 마운트.
    - name: config-map-demo # volume 이름
      configMap:
        name: configmap-demo-4 # 마운트할 configMap
        items: # configMap에서 파일로 생성할 키
          - key: BG
            path: "BACKGROUND" # 마운트된 볼륨에 생성될 파일이름
```
```shell
$ cd config
drwxrwxrwx 3 root root 4096 Jul 11 15:48 .
drwxr-xr-x 1 root root 4096 Jul 11 15:48 ..
drwxr-xr-x 2 root root 4096 Jul 11 15:48 ..2022_07_11_15_48_07.3623223474
lrwxrwxrwx 1 root root   32 Jul 11 15:48 ..data -> ..2022_07_11_15_48_07.3623223474
lrwxrwxrwx 1 root root   17 Jul 11 15:48 BACKGROUND -> ..data/BACKGROUND
```

### immutable

edit를 통해 object 수정이 불가능하다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-demo-5
data: # 일반 spec섹션 대신 data섹션이 있다.
  BG: BLACK
  COLOR: BLUE
immutable: true
```
