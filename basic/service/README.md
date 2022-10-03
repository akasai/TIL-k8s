# Service

파드에서 실행중인 애플리케이션을 네트워크 서비스로 노출시키는 추상화 방법이다.

즉, 파드 내부의 앱에 네트워크를 연결해주는 Gateway역할을 하는 리소스이다.

파드에게 주어진 고유 IP와 파드 집합에 대한 단일 DNS 명을 부여하고, 그것들 간에 로드밸런스를 수행한다.

서비스는 파드의 논리적 집합과 그것들에 접근할 수 있는 정책을 정의하는 추상적 개념이다.

서비스가 대상으로 하는 파드 집합은 일반적으로 `셀렉터`가 결정한다.

백엔드로 사용되는 파드의 배포로 인한 변경이 일어날지라도 프론트엔드는 이를 인지할 필요가 없다.

이러한 디커플링을 가능하게 해주는 리소스가 서비스이다.

## 서비스 정의

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

`app=MyApp`이라는 셀럭터를 갖는 파드들의 네트워크를 정의하는 메니페스트이다.

각 파드들은 TCP 포트 9376로 수신하며 포트 80으로 아운바운드된다.

k8s는 위 서비스에 서비스 프록시가 사용하는 IP 주소 ("Cluster IP"라고도 함) 를 할당한다.

> 서비스는 모든 수신 port를 targetPort에 매핑할 수 있다. 기본적으로 그리고 편의상, targetPort는 port 필드와 같은 값으로 설정된다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
    - name: nginx
      image: nginx:stable
      ports:
        - containerPort: 80
          name: http-web-svc

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
    - name: name-of-service-port
      protocol: TCP
      port: 80
      targetPort: http-web-svc
```

파드의 `ports`필드의 `name`필드를 이용하여 `targetPort` 필드에서 이를 참조할 수 있다.

### 셀렉터가 없는 서비스

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

기존에 생성했던 메니페스트파일과 달리 리소스를 지정하는 `selector`필드가 존재하지 않는다.

따라서 엔드포인트 오브젝트가 자동으로 생성되지 않는다.

엔드포인트 오브젝트를 수동으로 추가하여, 서비스를 실행 중인 네트워크 주소 및 포트에 서비스를 수동으로 매핑할 수 있다.

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service # 여기서의 이름은 서비스의 이름과 일치해야 한다.
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

위처럼 엔드포인트를 설정한 뒤 미리 생성된 서비스에 접근하 셀렉터를 설정한 것과 같이 동작한다.

즉, `192.0.2.42:9376`으로 라우팅된다.

### 초과 용량 엔드포인트 

만약 관리되는 엔드포인트의 수가 1000개를 초과하는 경우 클러스터는 `endpoints.kubernetes.io/over-capacity: truncated` 어노테이션을 추가한다.

## 멀티-포트 서비스 

둘 이상의 포트를 노출해야 하는 멀티 포트 서비스의 경우, 모든 포트 이름을 명확하게 지정해야 한다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

## 헤드리스(Headless) 서비스 

로드밸런싱이나 단일 서비스 IP는 필요치 않는 경우, **헤드리스 서비스**를 생성할 수 있다.

명시적으로 클러스터 IP (`.spec.clusterIP`)에 "None"을 지정한다.

클러스터 IP가 할당되지 않고, kube-proxy가 이러한 서비스를 처리하지 않으며, 플랫폼에 의해 로드 밸런싱 또는 프록시를 하지 않는다. 

DNS가 자동으로 구성되는 방법은 서비스에 셀렉터가 정의되어 있는지 여부에 달려있다.

### 셀렉터가 있는 경우 

엔드포인트 컨트롤러는 API에서 엔드포인트 레코드를 생성하고, DNS 구성을 수정하여 서비스를 지원하는 파드를 직접 가리키는 A 레코드(IP 주소)를 반환한다.

### 셀렉터가 없는 경우 

엔드포인트 컨트롤러는 엔드포인트 레코드를 생성하지 않는다. 그러나 DNS 시스템은 다음 중 하나를 찾고 구성한다.

1. ExternalName-유형 서비스에 대한 CNAME 레코드

2. 다른 모든 유형에 대해, 서비스의 이름을 공유하는 모든 엔드포인트에 대한 레코드

## 서비스 퍼블리싱 (ServiceTypes) 

ServiceTypes는 원하는 서비스 종류를 지정할 수 있도록 해준다. 기본 값은 **ClusterIP**이다.

1. **ClusterIP**: 서비스를 클러스터-내부 IP에 노출시킨다. 이 값을 선택하면 클러스터 내에서만 서비스에 도달할 수 있다. 기본값.

2. **NodePort**: 고정 포트(NodePort)로 각 노드의 IP에 서비스를 노출시킨다. NodePort 서비스가 라우팅되는 ClusterIP 서비스가 자동으로 생성된다. <NodeIP>:<NodePort>를 요청하여, 클러스터 외부에서 NodePort 서비스에 접속할 수 있다.

3. **LoadBalancer**: 클라우드 공급자의 로드 밸런서를 사용하여 서비스를 외부에 노출시킨다. 외부 로드 밸런서가 라우팅되는 NodePort와 ClusterIP 서비스가 자동으로 생성된다.

4. **ExternalName**: 값과 함께 CNAME 레코드를 리턴하여, 서비스를 externalName 필드의 콘텐츠 (예:foo.bar.example.com)에 매핑한다. 어떤 종류의 프록시도 설정되어 있지 않다.

인그레스(Ingress)를 사용하여 서비스를 노출시킬 수도 있다. 인그레스는 서비스 유형이 아니지만, 클러스터의 진입점 역할을 한다.

### NodePort 유형 

`type` 필드를 **NodePort**로 설정하면, k8s 컨트롤 플레인은 --service-node-port-range 플래그로 지정된 범위에서 포트를 할당한다 (기본값 : 30000-32767).

서비스는 할당된 포트를 `.spec.ports[*].nodePort` 필드에 나타낸다.
