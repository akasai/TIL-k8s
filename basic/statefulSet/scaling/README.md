# Scaling

### 명령적 방법

일반적인 scale 명령과 동일하게 스케일링 가능하다.

```bash
kubectl scale statefulesets <statefulset_name> --replicas=<N>
```

### 선언적 방법

스테이터스풀셋을 메니페스트 파일로 생성(kubectl apply)했을 경우 `replica`필드를 수정하여 스케일링 가능하다.

```shell
kubectl apply -f <new_menifest_file>
```

또는 **edit**를 사용하여 `replicas`필드를 수정한다.

```shell
kubectl edit statefulsets <statefulset_name>
```

**patch**명령을 사용하여 수정도 가능하다.

## 트러블 슈팅

스테이트풀셋은 확실한 특징을 가지고 있는 리소스이다.

즉, 스케일링은 스테이트풀셋 파드의 상태가 모두 정상일 경우에만 이루어진다.

비정상 파드의 존재로 인해 스케일링에 문제가 발생했을때 오류처리가 늦어질 경우

최소동작파드의 갯수 이하로 떨어지는 상황이 발생할 수 있다.

**따라서 스테이트풀셋의 스케일링은 반드시 스테이트풀셋 어플리케이션이 모두 정상이라고 판단된 이후에 
진행하는 것이 옳다.**
