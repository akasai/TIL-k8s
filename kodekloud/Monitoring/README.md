# Practice TEST

## Monitor Cluster Component

1. We have deployed a few PODs running workloads. Inspect them.

2. Let us deploy metrics-server to monitor the PODs and Nodes. Pull the git repository for the deployment files.

   ```shell
   $ git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
   ```
   
3. Deploy the metrics-server by creating all the components downloaded.

   ```shell
   $ kubectl create -f ./kubernetes-metrics-server
   ```

4. It takes a few minutes for the metrics server to start gathering data.

   ```shell
   $ kubectl top node
   ```
   
5. Identify the node that consumes the `most` CPU.

   ```shell
   $ kubectl top node --sort-by='cpu' --no-headers | head -1 
   ```
   
6. Identify the node that consumes the most Memory.

   ```shell
   $ kubectl top node --sort-by='memory' --no-headers | head -1 
   ```
   
7. Identify the POD that consumes the most Memory.

   ```shell
   $ kubectl top pod --sort-by='memory' --no-headers | head -1 
   ```
   
8. Identify the POD that consumes the least CPU.

   ```shell
   $ kubectl top pod --sort-by='cpu' --no-headers | tail -1 
   ```

## Managing Application logs 

1. We have deployed a POD hosting an application. Inspect it. Wait for it to start.

2. A user - `USER5` - has expressed concerns accessing the application. Identify the cause of the issue.

   ```shell
   $ kubectl logs webapp-1 # check error message
   ```
   
3. We have deployed a new POD - `webapp-2` - hosting an application. Inspect it. Wait for it to start.

4. A user is reporting issues while trying to purchase an item. Identify the user and the cause of the issue.

   ```shell
   $ kubectl logs webapp-2
   ```

