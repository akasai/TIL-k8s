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

