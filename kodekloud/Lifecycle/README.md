# Practice TEST

## Rolling Update & Rollback

1. We have deployed a simple web application. Inspect the PODs and the Services

2. What is the current color of the web application?

    - check the web page

3. Run the script named `curl-test.sh` to send multiple requests to test the web application. Take a note of the output.
   Execute the script at `/root/curl-test.sh`.

   ```shell
   $ ./curl-test.sh 
   ```
   
4. Inspect the deployment and identify the number of PODs deployed by it

   ```shell
   $ kubectl get deploy # check the replica
   ```
   
5. What container image is used to deploy the applications?

   ```shell
   $ kubectl describe deploy frontend | grep -i image
   ```
   
6. Inspect the deployment and identify the current strategy

   ```shell
   $ kubectl describe deployment # check the StrategyType section
   ```
   
7. If you were to upgrade the application now what would happen?

8. Let us try that. Upgrade the application by setting the image on the deployment to `kodekloud/webapp-color:v2`
   Do not delete and re-create the deployment. Only set the new image name for the existing deployment.

   ```shell
   $ kubectl edit deploy frontend # edit image section 
   ```
   
9. Run the script `curl-test.sh` again. Notice the requests now hit both the old and newer versions. However none of them fail.

10. Up to how many PODs can be down for upgrade at a time

   ```shell
   $ kubectl describe deployment # check the max unavailable section
   ```
   
11. Change the deployment strategy to `Recreate`

   ```shell
   $ kubectl edit deploy frontend # edit strategy.type section 
   ```
   
12. Upgrade the application by setting the image on the deployment to `kodekloud/webapp-color:v3`. 
Do not delete and re-create the deployment. Only set the new image name for the existing deployment. 

   ```shell
   $ kubectl edit deploy frontend # edit image section 
   ```

13. Run the script `curl-test.sh` again. Notice the failures. Wait for the new application to be ready. Notice that the requests now do not hit both the versions

## Commands and Arguments

1. How many PODs exist on the system?

   ```shell
   $ kubectl get po
   ```
   
2. kubectl describe po ubuntu-sleeper

   ```shell
   $ kubectl describe po ubuntu-sleeper # check the command section
   ```
   
3. Create a pod with the ubuntu image to run a container to sleep for 5000 seconds. Modify the file `ubuntu-sleeper-2.yaml`.

   ```shell
   $ kubectl run ubuntu-sleeper-2 --image=ubuntu --command sleep 5000
   ```

4. Create a pod using the file named `ubuntu-sleeper-3.yaml`. There is something wrong with it. Try to fix it!

   ```shell
   $ Both `sleep` and `1200` should be defined as a string.
   ```
   
5. Update pod `ubuntu-sleeper-3` to sleep for 2000 seconds.

   ```shell
   # edit yaml file
   ```
   
6. Inspect the file `Dockerfile` given at `/root/webapp-color` directory. What command is run at container startup?

   ```Dockerfile
   FROM python:3.6-alpine

   RUN pip install flask

   COPY . /opt/

   EXPOSE 8080

   WORKDIR /opt

   ENTRYPOINT ["python", "app.py"] 
   ```
   
7. Inspect the file `Dockerfile2` given at `/root/webapp-color` directory. What command is run at container startup?

8. Inspect the two files under directory `webapp-color-2`. What command is run at container startup?

9. Inspect the two files under directory `webapp-color-3`. What command is run at container startup?

10. Create a pod with the given specifications. By default it displays a `blue` background. Set the given command line arguments to change it to `green`.

## 

1. How many PODs exist on the system?

   ```shell
   $ kubectl get po
   ```

2. What is the environment variable name set on the container in the pod?

   ```shell
   $ kubectl describe po <pod_name> # check the environment
   ```

3. What is the value set on the environment variable `APP_COLOR` on the container in the pod?

   ```shell
   $ kubectl describe po <pod_name> # check the environment
   ```
   
4. View the web application UI by clicking on the `Webapp Color` Tab above your terminal.

5. Update the environment variable on the POD to display a `green` background

   ```shell
   $ kubectl get po <pod_name> -o yaml > pod.yaml 
   $ kubectl delete po <pod_name>
   $ vi pod.yaml # change env
   $ kubectl apply -f pod.yaml
   ```
   
6. View the changes to the web application UI by clicking on the `Webapp Color` Tab above your terminal.

7. How many `ConfigMaps` exists in the `default` namespace?

   ```shell
   $ kubectl get cm
   ```
   
8. Identify the database host from the config map `db-config`

   ```shell
   $ kubectl describe configmaps <configmap_name>
   ```
   
9. Create a new ConfigMap for the `webapp-color` POD. Use the spec given below.

   ```shell
   $ kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
   ```
   
10. Update the environment variable on the POD to use the newly created ConfigMap

   ```yaml
   ...
   - envFrom:
     - configMapRef:
         name: webapp-config-map
   ...
   ```
   
11. View the changes to the web application UI by clicking on the `Webapp Color` Tab above your terminal.

## Secret

1. How many `Secrets` exist on the system?

   ```shell
   $ kubectl get secret
   ```
   
2. How many secrets are defined in the `dashboard-token` secret?

   ```shell
   $ kubectl describe secret dashboard-token # counting data section
   $ kubectl get secret # check the data column
   ```
   
3. What is the type of the `dashboard-token` secret?

   ```shell
   $ kubectl describe secret dashboard-token # counting type section
   ```
   
4. Which of the following is not a secret data defined in `dashboard-token` secret?

   ```shell
   $ kubectl describe secret dashboard-token # counting data section
   ```
   
5. We are going to deploy an application with the below architecture

6. The reason the application is failed is because we have not created the secrets yet. Create a new secret named `db-secret` with the data given below.

   ```shell
   $ kubectl create secret generic db-secret --from-literal=DB_HOST=c3FsMDEK --from-literal=DB_USER=cm9vdAo= --from-literal=DB_Password=cGFzc3dvcmQxMjMK
   ```

7. Configure `webapp-pod` to load environment variables from the newly created secret.

   ```yaml
   ...
   envFrom:
    - secretRef:
        name: db-secret
   ...
   ```
   
8. View the web application to verify it can successfully connect to the database

## Multi Container

1. Identify the number of containers created in the `red` pod.

   ```shell
   $ kubectl get po # check the READY column
   ```
   
2. Identify the name of the containers running in the `blue` pod.

   ```shell
   $ kubectl describe po blue # check the container name
   ```
   
3. Create a multi-container pod with 2 containers. Use the spec given below.
   * Name: yellow
   * Container 1 Name: lemon
   * Container 1 Image: busybox
   * Container 2 Name: gold
   * Container 2 Image: redis
   
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     labels:
       run: yellow
     name: yellow
   spec:
     containers:
       - command:
         - sleep
         - "1000"
         image: busybox
         name: lemon
       - name: gold
         image: redis
   ```
   
4. We have deployed an application logging stack in the `elastic-stack` namespace. Inspect it.

5. Once the pod is in a ready state, inspect the Kibana UI using the link above your terminal. There shouldn't be any logs for now.

6. Inspect the `app` pod and identify the number of containers in it.

   ```shell
   $ kubectl get po -n elastic-stack
   ```
 
7.The application outputs logs to the file `/log/app.log`. View the logs and try to identify the user having issues with Login.

   ```shell
   $ kubectl exec app -- cat /log/app.log
   ```

8. Edit the pod to add a sidecar container to send logs to Elastic Search. Mount the log volume to the sidecar container.
Only add a new container. Do not modify anything else. Use the spec provided below.
 
## Init Container

1. Identify the pod that has an `initContainer` configured.

   ```shell
   $ kubectl describe po <pod_name> # check the initContainer
   ```
   
2. What is the image used by the `initContainer` on the `blue` pod?

   ```shell
   $ kubectl describe po <pod_name> # check the initContainer.image
   ```

3. What is the state of the `initContainer` on pod `blue`?

   ```shell
   $ kubectl describe po <pod_name> # check the initContainer.State
   ```
   
4. Why is the `initContainer` terminated? What is the reason?

   ```shell
   $ kubectl describe po <pod_name> # check the initContainer.State.Reason
   ```
   
5. We just created a new app named `purple`. How many `initContainers` does it have?

   ```shell
   $ kubectl describe po <pod_name> # check the initContainer
   ```

6. What is the state of the POD?

   ```shell
   $ kubectl get po
   ```

7. How long after the creation of the POD will the application come up and be available to users?

   ```shell
   $ kubectl describe po <pod_name> # check the initContainer.command
   ```

8. Update the pod `red` to use an `initContainer` that uses the `busybox` image and `sleeps for 20` seconds

   ```yaml
   ...
     initContainers:
     - command:
       - sh
       - -c
       - sleep 20
      image: busybox
      name: init-red
   ...
   ```
   
9. A new application `orange` is deployed. There is something wrong with it. Identify and fix the issue.



