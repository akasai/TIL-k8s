# Practice TEST

## Scheduling

1. A pod definition file `nginx.yaml` is given. Create a pod using the file.

    ```shell
    $ kubectl apply -f nginx.yaml
    ```

2. What is the status of the created POD?

    ```shell
    $ kubectl get pod # check STATUS column
    ```
   
3. Why is the POD in a pending state? Inspect the environment for various kubernetes control plane components.

    ```shell
    $ kubectl describe po nginx # check the state section & node section
    ``` 

    ```shell
    $ kubectl get pods --namespace=kube-system # check the scheduler
    ```

    first, check the detail information of target pod. state section shows pod's current state.

    Nodes section shows the nodes with the pod assigned to them.

    Scheduler is not normal if node section is NONE.

    Next, check the scheduler's state.

4. Manually schedule the pod on `node01`. Delete and recreate the POD if necessary.

    ```shell
    $ kubectl delete pod nginx # delete pod
    ```
   
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      nodeName: node01 # add nodeName section
      containers:
      - image: nginx
        name: nginx
    ```
    
    ```shell
    $ kubectl get po -o wide # check NODE column
    ```
   
    or
    
    ```shell
    $ kubectl replace --force -f nginx.yaml
    ---
    pod "nginx" deleted
    pod/nginx replaced
    ```

   **Import things this Cannot move a running pod from one node to another.**

5. Now schedule the same pod on the `controlplane` node. Delete and recreate the POD if necessary.

    ```shell
    $ kubectl delete pod nginx # delete pod
    ```

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      nodeName: controlplane # change nodeName to controlplane
      containers:
      - image: nginx
        name: nginx
    ```

    ```shell
    $ kubectl get po -o wide # check NODE column
    ```

    or

    ```shell
    $ kubectl replace --force -f nginx.yaml
    ---
    pod "nginx" deleted
    pod/nginx replaced
    ```

   **Import things this Cannot move a running pod from one node to another.**

## Labels & Selectors

1. We have deployed a number of PODs. They are labelled with `tier`, `env` and `bu`. How many PODs exist in the `dev` environment (`env`)?

   ```shell
   $ kubectl get po --selector='env=dev' --no-headers | wc -l
   ```

2. How many PODs are in the `finance` business unit (`bu`)?

   ```shell
   $ kubectl get po --selector='bu=finance' --no-headers | wc -l
   ```
   
3. How many objects are in the `prod` environment including PODs, ReplicaSets and any other objects?

   ```shell
   $ kubectl get all --selector='env=prod' --no-headers | wc -l
   ```

4. Identify the POD which is part of the `prod` environment, the `finance` BU and of `frontend` tier?

   ```shell
   $ kubectl get po --selector='env=prod,bu=finance,tier=frontend'
   ```

5. A ReplicaSet definition file is given `replicaset-definition-1.yaml`. Try to create the replicaset. There is an issue with the file. Try to fix it.

   ```yaml
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
    name: replicaset-1
   spec:
    replicas: 2
    selector:
      matchLabels:
        tier: front-end
    template:
      metadata:
        labels:
          tier: nginx # it must same selector section => front-end
      spec:
        containers:
        - name: nginx
          image: nginx
   ```
  
   ```shell
   $ kubectl apply -f replicaset-definition-1.yaml
   ```

## Taints & Tolerations

1. How many `nodes` exist on the system? Including the `controlplane` node.

   ```shell
   $ kubectl get node --no-headers | wc -l
   ```
   
2. Do any taints exist on `node01` node?

   ```shell
   $ kubectl describe node node01 | grep -i taints
   ```
   
3. Create a taint on `node01` with key of `spray`, value of `mortein` and effect of `NoSchedule`

   ```shell
   $ kubectl taint node node01 spray=mortein:NoSchedule
   ```

4. Create a new pod with the nginx image and `pod` name as `mosquito`.

   ```shell
   $ kubectl run mosquito --image=nginx
   ```
   
5. What is the state of the POD?

   ```shell
   $ kubectl get po # check the STATE column
   ```
   
6. Why do you think the pod is in a pending state?

   ```shell
   $ kubectl describe po node01 # check Event section
   ```
   
7. Create another pod named `bee` with the `nginx` image, which has a toleration set to the taint `mortein`. 
   
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata: 
      name: bee
   spec:
      containers:
        - image: nginx
          name: bee
      tolerations: # Add toleration section
        - key: spray
          value: mortein
          effect: NoSchedule
          operator: Equal
   ```

   ```shell
   $ kubectl create -f pod.yaml
   ```

8. Notice the `bee` pod was scheduled on node `node01` despite the taint.

   ```shell
   $ kubectl get po -o wide # check NODE column
   ```
   
9. Do you see any taints on `controlplane` node?

   ```shell
   $ kubectl describe node controlplane # check the taints section
   ```
   
10. Remove the taint on `controlplane`, which currently has the taint effect of `NoSchedule`.

   ```shell
   $ kubectl taint node controlplane node-role.kubernetes.io/master:NoSchedule- # minus mean untaint
   $ kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
   ```
   
11. What is the state of the pod `mosquito` now?

   ```shell
   $ kubectl get po # check STATE column
   ```
   
12. Which node is the POD mosquito on now?

   ```shell
   $ kubectl get po -o wide # check NODE column
   ```

## Node Affinity

1. How many Labels exist on node node01?

   ```shell
   $ kubectl get node node01 --show-labels
   ---
   $ kubectl describe node node01 # counting labels
   ```

2. What is the value set to the label key `beta.kubernetes.io/arch` on `node01`?

   ```shell
   $ kubectl get node node01 --show-labels
   ---
   $ kubectl describe node node01 # check the labels
   ```

3. Apply a label `color=blue` to node `node01`

   ```shell
   $ kubectl label node node01 color=blue
   ```
   
4. Create a new deployment named `blue` with the `nginx` image and 3 replicas.

   ```shell
   $ kubectl create deployment blue --image=nginx --replicas=3
   ```

5. Which nodes `can` the pods for the `blue` deployment be placed on?
   Make sure to check taints on both nodes!

   ```shell
   $ kubectl describe node controlplane | grep -i taints
   $ kubectl describe node node01 | grep -i taints
   # check all node's taints. all pods allocated at node that taints is none.
   ```
   
6. Set Node Affinity to the deployment to place the pods on `node01` only.

   - Name: blue

   - Replicas: 3

   - Image: nginx

   - NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution

   - Key: color

   - value: blue

   ```shell
   $ kubectl create deploy blue --image=nginx --replicas=3 --dry-run=client -o yaml > deploy.yaml
   ```

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
      labels:
         app: blue
      name: blue
   spec:
      replicas: 3
      selector:
         matchLabels:
           app: blue
      template:
        metadata:
          labels:
            app: blue
        spec:
          containers:
            - image: nginx
              name: nginx
          affinity: # Add node Affinity
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: color
                        values:
                        - blue
                        operator: In
   ```
   
7. Which nodes are the pods placed on now?

   ```shell
   $ kubectl get pods -o wide # check the NODE column
   ```
   
8. Create a new deployment named `red` with the `nginx` image and `2` replicas, and ensure it gets placed on the controlplane node only.
Use the label key - `node-role.kubernetes.io/control-plane` - which is already set on the `controlplane` node.

   - Name: red

   - Replicas: 2

   - Image: nginx

   - NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution

   - Key: node-role.kubernetes.io/control-plane

   - Use the right operator

   ```shell
   $ kubectl create deploy red --image=nginx --replicas=2 --dry-run=client -o yaml > deploy.yaml
   ```

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
      labels:
         app: red 
      name: red
   spec:
      replicas: 2
      selector:
         matchLabels:
           app: red
      template:
        metadata:
          labels:
            app: red
        spec:
          containers:
            - image: nginx
              name: nginx
          affinity: # Add node Affinity
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: node-role.kubernetes.io/control-plane
                        operator: Exists
   ```

## Resource Limits

1. A pod called `rabbit` is deployed. Identify the CPU requirements set on the Pod

   ```shell
   $ kubectl describe pod rabbit # check the CONTAINER.RESOURCES.REQUEST.CPU section
   ```

2. Delete the `rabbit` Pod.

   ```shell
   $ kubectl delete pod rabbit
   ```

3. Another pod called `elephant` has been deployed in the default namespace. It fails to get to a running state. Inspect this pod and identify the `Reason` why it is not running.

   ```shell
   $ kubectl describe pod elephant # check the Reason section
   ```

4. The `elephant` pod runs a process that consume 15Mi of memory. Increase the limit of the `elephant` pod to 20Mi.
   Delete and recreate the pod if required. Do not modify anything other than the required fields.

   * Pod Name: elephant
   * Image Name: polinux/stress
   * Memory Limit: 20Mi

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
      run: elephant
      name: elephant
   spec:
      containers:
       - image: polinux/stress
         name: elephant
         resources:
            requests:
               memory: 15Mi
            limits:
               memory: 20Mi 
   ```

## Daemonsets

1. How many `DaemonSets` are created in the cluster in all namespaces?

   ```shell
   $ kubectl get ds --all-namespaces
   ```

2. Which namespace are the `DaemonSets` created in?

   ```shell
   $ kubectl get ds --all-namespaces
   ```
   
3. Which of the below is a `DaemonSet`?

   ```shell
   $ kubectl get ds --all-namespaces
   ```

4. On how many nodes are the pods scheduled by the DaemonSet `kube-proxy`

   ```shell
   $ kubectl describe daemonset kube-proxy --namespace=kube-system
   # Desired Number of Nodes Scheduled: 1
   # Current Number of Nodes Scheduled: 1
   ```

5. What is the image used by the POD deployed by the `kube-flannel-ds` DaemonSet?

   ```shell
   $ kubectl create deploy elasticsearch --image=k8s.gcr.io/fluentd-elasticsearch:1.20 --namespace=kube-system -o yaml --dry-run=client > ds.yaml
   # change kind deployment to daemonset
   # remove replica section
   ```


