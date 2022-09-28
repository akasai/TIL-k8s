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
