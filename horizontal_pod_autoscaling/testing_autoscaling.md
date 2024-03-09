# Testing Autoscaling

- we have `setup` the `HPA autoscaling rule` that `for any reason if the APi Gateway POD microservice` `CPU request goes beyond the 400% of the Actual CPU request` i.e `400millicore` then we will presume that `there will be heavy traffic` and `HPA autoscaling rule` create the `replicas for the Deployment automatically`

- `HPA rule` will create the `propotionate amount of replica based on the overuse of CPU request thats the POD been utilizing`

- for example :- if the `APi Gateway POD microservice` usage is `1200m` then it will create `3 replicas for the APi Gateway POD microservice` as the `400 millicore` set as the `CPU request` which will bring the `CPU Usage of each APi Gateway POD microservice` back to `400m` which is the `maximum CPU usage for a POD based on the request`

- but if reuse the `curl command` multiple times in order to `perform the CPU intensive Task to make the CPU request go crazy` then we can see the `CPU request` will be much `higher than the CPU requested`

    ```bash
        curl http://192.168.49.2:30030/api/panic
        # here we are hitting the URL endpoint as /api/panic
        # this will perform some CPU intensive Task and make the CPU request go crazy
        # here we need to perform this curl command multiple time to increase the CPU request Usage
        # the result will be as below 
        # we need to execute the command multiple times in order to make the CPU request go crazy
        Results
        0.4980303947417202
        0.7605447090702907
        0.5307555181446534
        0.4850623162638466
        0.8823027734940408
        0.406893307816916
        0.21791576142316393
        0.2943334506508206
        0.3243915847530662
        0.008709842126961909
        0.30853959783554175
        0.4538077598192898
        0.6257812015183968
        0.5722180541253049
        0.7442600655363791
        0.6148140945545075
        0.6743229942355875
        0.998101334665764
        0.7867397112312472
        0.02212006658582011
        0.905029925424083
        0.3080710762337142
        0.2696481320635776
        0.3418999044957054
        0.2304367709043822
        0.05936162802562772
        ---

        # now when we check the kubectl top pod command in order to check the CPU and Memory Usage
        # here we can see the info as below 
        kubectl top pod
        # fetching the resource usage acrross the default namespace POD in this case
        # here as the kubernetes Metrics Server taking 1` minutes to gather the metrics we will wait for a Minute as the default sampling value to see the higher value
        # here we will be getting the below output in this case
        NAME                                 CPU(cores)   MEMORY(bytes)   
        api-gateway-597c9f8675-fqv7v         1428m        245Mi           
        mongodb-5558cf99d9-l28qp             295m         490Mi           
        position-simulator-f9c8794ff-qtwgn   24m          214Mi           
        position-tracker-bb49fdd8f-z79ld     7m           316Mi           
        queueapp-7fff87bc5d-htbxr            30m          347Mi           
        webapp-587c5cbd6d-v6sjt              1m           9Mi 

    ```

- here we can see that `kubectl top pod` command been `giving us CPU usage as 1428m` hence we can see that `there will be 3 and 1 API Gateway PODs microservice running now`

- we can see that info as below 

    ```bash
        kubectl get pod
        # fetching the POD in the default namespace in this case
        # the output will be as below 
        NAME                                 READY   STATUS    RESTARTS      AGE
        api-gateway-597c9f8675-25z8r         1/1     Running   0             5s
        api-gateway-597c9f8675-f2gj9         1/1     Running   0             2m5s
        api-gateway-597c9f8675-fqv7v         1/1     Running   1 (9h ago)    9h
        api-gateway-597c9f8675-vv89s         1/1     Running   0             5s
        mongodb-5558cf99d9-l28qp             1/1     Running   51 (9h ago)   2d2h
        position-simulator-f9c8794ff-qtwgn   1/1     Running   4 (9h ago)    2d15h
        position-tracker-bb49fdd8f-z79ld     1/1     Running   4 (9h ago)    2d16h
        queueapp-7fff87bc5d-htbxr            1/1     Running   4 (9h ago)    2d16h
        webapp-587c5cbd6d-v6sjt              1/1     Running   8 (9h ago)    2d2h

    ```

- when we see that `kubectl top pod` that `CPU request` been distributed accross the `all the 4 POD` which been created by the `HPA Rule`

- we can see that info as below 

    ```bash
        # now when we check the kubectl top pod command in order to check the CPU and Memory Usage
        # here we can see the info as below 
        kubectl top pod
        # fetching the resource usage acrross the default namespace POD in this case
        # here as the kubernetes Metrics Server taking 1` minutes to gather the metrics we will wait for a Minute as the default sampling value to see the higher value
        # here we will be getting the below output in this case
        NAME                                 CPU(cores)   MEMORY(bytes)   
        api-gateway-597c9f8675-25z8r         24m          232Mi           
        api-gateway-597c9f8675-f2gj9         7m           255Mi           
        api-gateway-597c9f8675-fqv7v         4m           245Mi           
        api-gateway-597c9f8675-vv89s         18m          254Mi           
        mongodb-5558cf99d9-l28qp             1453m        492Mi           
        position-simulator-f9c8794ff-qtwgn   26m          214Mi           
        position-tracker-bb49fdd8f-z79ld     11m          317Mi           
        queueapp-7fff87bc5d-htbxr            31m          317Mi           
        webapp-587c5cbd6d-v6sjt              1m           9Mi 

    ```

- now when we check the `kubeclt describe hpa api-gateway-another` then we can see that `how many instances that been replicated because of the High Volume of Traffic`

    ```bash
        kubectl describe hpa api-gateway-another
        # fetching the description of the HPA rule in this case
        Name:                     api-gateway-another
        Namespace:                default
        Labels:                   <none>
        Annotations:              autoscaling.alpha.kubernetes.io/conditions:
                                    [{"type":"AbleToScale","status":"True","lastTransitionTime":"2024-03-08T18:33:45Z","reason":"ScaleDownStabilized","message":"recent recomm...
                                autoscaling.alpha.kubernetes.io/current-metrics:
                                    [{"type":"Resource","resource":{"name":"cpu","currentAverageUtilization":7,"currentAverageValue":"7m"}}]
        CreationTimestamp:        Sat, 09 Mar 2024 00:03:30 +0530
        Reference:                Deployment/api-gateway
        Target CPU utilization:   400%
        Current CPU utilization:  7%
        Min replicas:             1
        Max replicas:             5
        Deployment pods:          3 current / 3 desired
        Events:
        Type    Reason             Age                   From                       Message
        ----    ------             ----                  ----                       -------
        Normal  SuccessfulRescale  6m59s (x4 over 135m)  horizontal-pod-autoscaler  New size: 4; reason: cpu resource utilization (percentage of request) above target


        # here actually we can see that 4 instances of the POD microservice including the number of request we can see in here

    ```

- here we can see the `CPU request` been `disributed across all the API Gateway POD microservice` , but the `POD microservices` not going to be `scaled down immediately`

- when the `API Gateway POD microservice` when under the `CPU request`  if we are not making the `special URL endpoint call` then also we can see the `POD still persist` and will not be `scale down immediately`

- this is because the `HPA will monitor the metrics coming from the metrics-server for several moment` and after `several moment` if it observe that `the CPU request not going high` then it will then be `Scaling Down the request`

- if we see the `Metrics` inside the `kubectl describe hpa api-gateway-another` reporting 
  
  - `Current CPU utilization`
  
  - `Target CPU utilization` 

- we will see the `Scale down eventually` after all the `CPU request for All the API Gateway POD container` are `consistent`

- when all the `API Gateway POD microservice CPU percentage` goes `below the percentage of CPU that we have specified in the HPA Rule for a several minute of duration` then it will `scale down the instances back to the original one`

- the `reasoning behind` this is the `CPU request` been getting `spikey` we don't want to `perfom` the `scale up` and `scale down` again and again , which is called `thrashing`

- frequent `scale up` and `scale down` again and again will add the `CPU load to the cluster`

- `HPA rule` is very `conservative` while doing the `Scale Down` , it will gpoing to `wait and wait` and `perform the ScaleDown after several minute of observation of Lower CPU request limit that we have specified` 

- we can see that `after several minutes` as below 

    ```bash
        kubectl describe hpa api-gateway-another
        # fetching the description of the HPA rule in this case
        Name:                     api-gateway-another
        Namespace:                default
        Labels:                   <none>
        Annotations:              autoscaling.alpha.kubernetes.io/conditions:
                                    [{"type":"AbleToScale","status":"True","lastTransitionTime":"2024-03-08T18:33:45Z","reason":"ReadyForNewScale","message":"recommended size...
                                autoscaling.alpha.kubernetes.io/current-metrics:
                                    [{"type":"Resource","resource":{"name":"cpu","currentAverageUtilization":5,"currentAverageValue":"5m"}}]
        CreationTimestamp:        Sat, 09 Mar 2024 00:03:30 +0530
        Reference:                Deployment/api-gateway
        Target CPU utilization:   400%
        Current CPU utilization:  5%
        Min replicas:             1
        Max replicas:             5
        Deployment pods:          1 current / 1 desired
        Events:
        Type    Reason             Age                   From                       Message
        ----    ------             ----                  ----                       -------
        Normal  SuccessfulRescale  26m (x2 over 154m)    horizontal-pod-autoscaler  New size: 2; reason: cpu resource utilization (percentage of request) above target
        Normal  SuccessfulRescale  24m                   horizontal-pod-autoscaler  New size: 4; reason: All metrics below target
        Normal  SuccessfulRescale  20m                   horizontal-pod-autoscaler  New size: 3; reason: All metrics below target
        Normal  SuccessfulRescale  10m                   horizontal-pod-autoscaler  New size: 4; reason: cpu resource utilization (percentage of request) above target
        Normal  SuccessfulRescale  9m46s                 horizontal-pod-autoscaler  New size: 5; reason: cpu resource utilization (percentage of request) above target
        Normal  SuccessfulRescale  4m15s (x3 over 148m)  horizontal-pod-autoscaler  New size: 1; reason: All metrics below target
        # here we can see that its been scaled down to 1 instance in this case


    ```
