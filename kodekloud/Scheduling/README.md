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

