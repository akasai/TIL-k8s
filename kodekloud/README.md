# Practice TEST

## Core Concepts

### Pod

1. How many `pods` exist on the system?

    ```shell
    $ kubectl get pods
    ```

    클러스터의 존재하는 파드 전체 검색.


2. Create a new pod with the `nginx` image.

    ```shell
    $ kubectl run <pod_name> --image=<image_name>
    ```

    파드를 생성하는 명령적 방법.


3. How many pods are created now?

    ```shell
    $ kubectl get pods
    ```
   
4. What is the image used to create the new pods?

   ```shell
   $ kubectl describe pod <pod-name> | grep -i image
   ```

    pod 상세정보에서 Image 섹션 확인.
 

5. Which nodes are these pods placed on?

   ```shell
   $ kubectl get pod -o wide
   ---
   NAME                            READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
   frontend-fflmg                  1/1     Running   0          37d   172.17.0.6   minikube   <none>           <none>
   ``` 

    파드 목톡의 보기옵션 중 `wide`를 설정하여 node 정보까지 확인

   ```shell
   $ kubectl describe pod <pod-name> | grep -i node
   ```

   pod 상세정보에서 node 섹션 확인.


6. How many containers are part of the pod `webapp`?

    ```shell
    $ kubectl get pod
    ```
   
    pod 목록에서 READY 상태확인.


7. What images are used in the new `webapp` pod?

    ```shell
    $ kubectl describe pod <pod-name> | grep -i image
    ```

   pod 상세정보에서 Image 섹션 확인.

8. What is the state of the container `agentx` in the pod `webapp`?

    ```shell
    $ kubectl describe pod <pod-name>
    ```

   pod 상세정보에서 container 상태 확인.

9. Why do you think the container `agentx` in pod `webapp` is in error?

    ```shell
    $ kubectl describe pod <pod-name> # check Events section
    ```

   pod 상세정보에서 Event 섹션 확인.


10. What does the `READY` column in the output of the `kubectl get pods` command indicate?

    ```shell
    NAME                            READY   STATUS    RESTARTS   AGE
    frontend-fflmg                  1/1     Running   0          37d
    ```
    
    동작중인 파드내 컨테이너 / 파드내 전체 컨테이너 


11. Delete the `webapp` Pod.

    ```shell
    $ kubectl delete pod <pod-name>
    ```

12. Create a new pod with the name `redis` and with the image `redis123`.

    ```shell
    $ kubectl run <pod_name> --image=<image_name> --dry-run=client -o yaml > manifest.yaml # extract manifest file
   
    $ kubectl create -f manifest.yaml # create resource via manifest file
   
    ---
    $ kubectl run redis --image=redis123
    ```


13. Now change the image on this pod to `redis`.
   
    ```shell
    $ kubectl apply -f manifest.yaml # edit image section in manifest file 

    $ kubectl edit pod redis # edit image section
    ```
    
### ReplicaSet

1. How many PODs exist on the system?

   ```shell
   $ kubectl get pod
   ```

2. How many ReplicaSets exist on the system?

   ```shell
   $ kubectl get replicaset # or rs
   ```

3. How about now? How many ReplicaSets do you see?

   ```shell
   $ kubectl get replicaset # or rs
   ```

4. How many PODs are DESIRED in the new-replica-set?

   ```shell
   $ kubectl get replicaset # or rs
   ---
   NAME              DESIRED   CURRENT   READY   AGE
   new-replica-set   4         4         0       12s
   ```

5. What is the image used to create the pods in the `new-replica-set`?

   ```shell
   $ kubectl describe rs <rs_name> | grep -i image # check image section in rs spec
   ```

6. How many PODs are READY in the `new-replica-set`?

   ```shell
   $ kubectl get replicaset # or rs
   ---
   NAME              DESIRED   CURRENT   READY   AGE
   new-replica-set   4         4         0       12s # check the READY column
   ```
   
7. Why do you think the PODs are not ready?

   ```shell
   $ kubectl describe pod <pod-name> # check Events section
   ```

8. Delete any one of the 4 PODs.

   ```shell
   $ kubectl delete pod <pod-name>
   ```
   
9. How many PODs exist now?

   ```shell
   $ kubectl get pod
   ```

10. Why are there still 4 PODs, even after you deleted one?

   ReplicaSet ensures that desired number of PODs always run.

11. Create a ReplicaSet using the `replicaset-definition-1.yaml` file located at `/root/`.
    There is an issue with the file, so try to fix it.

   ```shell
   $ kubectl explain replicaset | grep VERSION # check support version of replicaSet
   ```
   
12. Fix the issue in the `replicaset-definition-2.yaml` file and create a `ReplicaSet` using it.

   ```yaml
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
      name: replicaset-2
   spec:
      replicas: 2
      selector:
         matchLabels:
            tier: frontend # selector section has to equal the pod's label
      template:
         metadata:
            labels:
               tier: frontend
         spec:
            containers:
            - name: nginx
              image: nginx
   ```

13. Delete the two newly created ReplicaSets - `replicaset-1` and `replicaset-2`

   ```shell
   $ kubectl delete replicaset <rs_name>
   ```

14. Fix the original replica set `new-replica-set` to use the correct `busybox` image.

   ```shell
   $ kubectl edit replicaset <rs_name> # edit image section
   ---
   $ kubectl delete po <pod_name> # delete all pod that created via replicaset
   ```

15. Scale the ReplicaSet to 5 PODs.

   ```shell
   $ kubectl scale rs <rs_name> --replicas=5 # scale command that can do only scaling 
   ---
   $ kubectl edit rs <rs_name> # edit replicas section
   ```

16. Now scale the ReplicaSet down to 2 PODs.
   
   ```shell
   $ kubectl scale rs <rs_name> --replicas=5 # scale command that can do only scaling 
   ---
   $ kubectl edit rs <rs_name> # edit replicas section
   ```

### Deployment

1. How many PODs exist on the system?

   ```shell
   $ kubectl get pod
   ``` 

2. How many ReplicaSets exist on the system?

   ```shell
   $ kubectl get replicaset # or rs
   ```

3. How many Deployments exist on the system?

   ```shell
   $ kubectl get deployment # or deploy
   ```

4. How many Deployments exist on the system now?

   ```shell
   $ kubectl get deployment # or deploy
   ```

5. How many ReplicaSets exist on the system now?

   ```shell
   $ kubectl get replicaset # or rs
   ```

6. How many PODs exist on the system now?

   ```shell
   $ kubectl get pod
   ``` 

7. Out of all the existing PODs, how many are ready?

   ```shell
   $ kubectl get pod # check the READY column
   ``` 

8. What is the image used to create the pods in the new deployment?

   ```shell
   $ kubectl describe deployment <deployment_name> | grep -i image # check image name
   ``` 

9. 

   ```shell
   $ kubectl describe pod <pod-name> # check event section
   ```
   
10. Create a new Deployment using the deployment-definition-1.yaml file located at /root/.
There is an issue with the file, so try to fix it.

   ```yaml
   apiVersion: apps/v1
   kind: deployment => Deployment # Capital D
   metadata:
      name: deployment-1
   spec:
      replicas: 2
      selector:
         matchLabels:
            name: busybox-pod
      template:
         metadata:
            labels:
               name: busybox-pod
         spec:
            containers:
             - name: busybox-container
               image: busybox888
               command:
                 - sh
                 - "-c"
                 - echo Hello Kubernetes! && sleep 3600
   ```

11. Create a new Deployment with the below attributes using your own deployment definition file.
    * Name: httpd-frontend;
    * Replicas: 3;
    * Image: httpd:2.4-alpine

   ```shell
   $ kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3 --dry-run=client -o yaml > deployment.yaml
   $ kubectl apply -f deployment.yaml
   ```

   ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: httpd-frontend
      name: httpd-frontend
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: httpd-frontend
      template:
        metadata:
          labels:
            app: httpd-frontend
        spec:
          containers:
          - image: httpd:2.4-alpine
            name: httpd
   ```

### Namespace

1. How many Namespaces exist on the system?

   ```shell
   $ kubectl get ns # or namespaces
   ---
   $ kubectl get ns --no-headers | wc -l # counting
   ```

2. How many pods exist in the `research` namespace?

   ```shell
   $ kubectl get pod -n=research
   ```

3. Create a POD in the `finance` namespace. Use the spec given below.
   - Name: redis
   - Image Name: redis

   ```shell
   $ kubectl run redis --image=redis --namespace=finance --dry-run=client -o yaml > pod.yaml
   ```
   
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     labels:
       run: redis
     name: redis
     namespace: finance
   spec:
     containers:
      - image: redis
        name: redis
   ```
   
4. Which namespace has the `blue` pod in it?

   ```shell
   $ kubectl get po --all-namespaces | grep blue
   ```

5. Access the Blue web application using the link above your terminal!!

6. What DNS name should the `Blue` application use to access the database `db-service` in its own namespace - `marketing`?

   ```shell
   $ kubectl get po --all-namespaces | grep blue # check namespace of blue
   ```
   
7. What DNS name should the `Blue` application use to access the database `db-service` in the `dev` namespace?

   =>  Since the `blue` application and the `db-service` are in different namespaces.

   `<application_name>.<namespace>.svc.cluster.local`

### Service

1. How many Services exist on the system?

```shell
$ kubectl get svc # or service
```

3. What is the type of the default `kubernetes` service?

```shell
$ kubectl get svc # check Type column
```

4. What is the `targetPort` configured on the `kubernetes` service?

```shell
$ kubectl describe svc kubernetes | grep -i targetport
```

5. How many labels are configured on the `kubernetes` service?

```shell
$ kubectl describe svc kubernetes # check Labels section
```

6. How many Endpoints are attached on the `kubernetes` service?

```shell
$ kubectl describe svc kubernetes # check Endpoints section
```

7. How many Deployments exist on the system now?

```shell
$ kubectl get deploy
```

8. What is the image used to create the pods in the deployment?

```shell
$ kubectl describe deploy simple-webapp-deployment | grep -i image
```

9. Are you able to accesss the Web App UI?

10. Create a new service to access the web application using the `service-definition-1.yaml` file.
    - Name: webapp-service
    - Type: NodePort
    - targetPort: 8080
    - port: 8080
    - nodePort: 30080
    - selector: name: simple-webapp

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
      name: webapp-service
   spec:
      type: NodePort
      ports:
       - targetPort: 8080
         port: 8080
         nodePort: 30080
      selector:
        name: simple-webapp
   ```
   ```shell
   $ kubectl apply -f service.yaml 
   ```

### Imperative command

1. In this lab, you will get hands-on practice with creating Kubernetes objects imperatively.

2. Deploy a pod named `nginx-pod` using the `nginx:alpine` image.

   ```shell
   $ kubectl run nginx-pod --image=nginx:alpine
   ```

3. Deploy a `redis` pod using the `redis:alpine` image with the labels set to `tier=db`.

   ```shell
   $ kubectl run redis --image=redis:alpine --labels='tier=db'
   ```

4. Create a service `redis-service` to expose the `redis` application within the cluster on port `6379`.

   ```shell
   $ kubectl expose pod redis --port=6379 --name=redis-service
   ```

5. Create a deployment named `webapp` using the image `kodekloud/webapp-color` with `3` replicas.

   ```shell
   $ kubectl create deploy webapp --image=kodekloud/webapp-color --replicas=3
   ```

6. Create a new pod called `custom-nginx` using the `nginx` image and expose it on `container port 8080`.

   ```shell
   $ kubectl run custom-nginx --image=nginx --port=8080
   ```
   
7. Create a new namespace called `dev-ns`.

   ```shell
   $ kubectl create ns dev-ns
   ```
   
8. Create a new deployment called `redis-deploy` in the `dev-ns` namespace with the `redis` image. It should have `2` replicas.

   ```shell
   $ kubectl create deploy redis-deploy --image=redis --replicas=2 --namespace=dev-ns
   ```
   
9. Create a pod called `httpd` using the image `httpd:alpine` in the default namespace. Next, create a service of type `ClusterIP` by the same name (`httpd`). The target port for the service should be `80`.

   ```shell
   $ kubectl run httpd --image=httpd:alpine --port=80 --expose # expose option will create svc(clusterIp)
   ```

   --expose=false:
   If true, create a ClusterIP service associated with the pod. Requires `--port`.
