# Applying Liveness and Readiness Probes

- we can see the `if the software inside the POD container` is might not `ready for the Service` even though the `POD container running` , but `by default kubernetes will consider the POD in service as soon as the POD container been running` in that case `we need to set the readiness or liveness probe`

- we can add the `readinessProbe section` to the POD container definition yaml

- `kubernetes` provide support for `Liveness and Readiness  Probe` 

- the `Readiness and Liveness Probe` are `separate conecpt altogether` in fact , `but they configured in the similar way` and `pretty much identical`

- here if we want to `avoid the kubernetes Service` to send the `Request to the New Kubernetes POD which might not be ready for Service yet because the software inside the POD container has not come up running` then we can `use the readiness probe in this case`

- the `idea` of the `readiness probe` is to `give some configuration to the POD` using which `kubernetes can Test whether the POD is ready to receive and Serve request or Not`

- the `benifit` is that `if we have setup the Readiness Probe for a POD microservcie` then `kubernetes Service` `will not send request traffic` to the `New Kubernetes POD microservice` untill the `kubernetes POD microservice Readiness Probe been stating as ready`

- here we need to set the `readinessProbe` in the `same level` as `POD container name level` in this case 

- here we will `request kubernetes to send the Http request to the Corresponding POD container underwhich the readinessProbe been defined readinesssProbe`

- we will ask it to `reach to the /api Test endpoint` like how we are performing the `curl http://<minikube IP>:<Node Port>/api` if the `Http Request been successful return code` then the `POD can be marked as ready to receive request from the kubernetes Services`

- here we need to provide the `httpGet` inside the `readinessProbe` , to make the `http request to the  container iff it is a Web Server container or Web container` which can respond to the `http request`

- inside the `httpGet`  we need to provide the `Path`, on `which URL path` we will be sending the `HTTP GET request to the POD container`

- here we when we send the `curl http://<minikube IP>:<Node Port>/api` ti will goto the `nginx web server i.e nginx frontend` and `nginx configured such a way that` when the `request starts with /api` it will forward that to the `API Gateway End Point` but while sending to `Api Gateway` it will remove the `/api` from the `URL`

- hence we are making the `call to the API Gateway Service which will goto API Gateway POD` with the endpont as `/`  

- we also need to specify the `container target port which been exposed` so that we can make the `http request` which is normally `8080` for the `web Server or web container`  which the `developer of the POD can ensure`

- we can `configure the readiness probe` inside the `API Gateway POD container` inside the `workloads.yml` as below 

    ```yaml
        workload.yaml
        =============
        apiVersion: apps/v1 # here the Deployment belongs to the apps group hyence defined as below 
        kind: Deployment # type of the kubernetes object is deployment
        metadata: # name of the Deployment been described here
            name: position-simulator
        spec: # specification for the Deployment been defined here
            selector: # selector to select the POD based on the POD label
                matchLabels:
                    app: position-simulator
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: position-simulator
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                    - name: position-simulator # name of the container
                      image: richardchesterwood/k8s-fleetman-position-simulator:resources # image of the container changing the tags to resources
                      env: # env Variable for the container
                        - name: SPRING_PROFILES_ACTIVE # name 0f the env Variable 
                          value: production-microservice # value of the env Variable
                      resources: # setting up the resource request for the position-simulator POD container
                        requests:
                            memory: 350Mi # setting up the resources in the range as 350Mi i.e 350 Megabyte 
                            cpu: 100m # setting up the resources in this request as  100millicore

        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hence defined as below 
        kind: Deployment # type of the kubernetes object is deployment
        metadata: # name of the Deployment been described here
            name: position-tracker
        spec: # specification for the Deployment been defined here
            selector: # selector to select the POD based on the POD label
                matchLabels:
                    app: position-tracker
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container

                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: position-tracker
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                    - name: position-tracker # name of the container
                      image: richardchesterwood/k8s-fleetman-position-tracker:resources # image of the container changing the tags to resources
                      env: # env Variable for the container
                        - name: SPRING_PROFILES_ACTIVE # name 0f the env Variable 
                          value: production-microservice # value of the env Variable
                      resources: # setting up the resource request for the position-tracker POD container
                        requests:
                            memory: 350Mi # setting up the resources in the range as 350Mi i.e 350 Megabyte 
                            cpu: 100m # setting up the resources in this request as  100millicore


        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hence defined as below 
        kind: Deployment # type of the kubernetes object is Deployment
        metadata: # name of the Deployment been described here
            name: api-gateway
            namespace: default # here the namespace for the POD being as default
        spec: # defining the specification for the Deployment in this case
            selector: # selector for the deployment based on POD label being defined in here
                matchLabels:
                    app: api-gateway
            replicas: 1 #replicas of the POD defined as 1 at any instance in order to scale down # imp
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: api-gateway
            spec: # Specification of the POD container been defined in here
                containers: # container details provided here
                - name: api-gateway # name of the container
                  image: richardchesterwood/k8s-fleetman-api-gateway:performance # image of the container changing the tags to resources
                  readinessProbe: # here we are setting up the readioness probe for the api-gateway POD created from the Deployment
                    httpGet: # here we will be sending the http Get request and the POD will be only marked as `running` if the httpGet request is successful
                        path: / # here the path for the request as / as the nginx web frontend hit the api-gateway as /api and when it hit the Api Gateway then /api will be removed hence we are providing as / in this case
                        port: 8080 # here the port in which it will try to access the apiGateway Service Port which is 8080 to get the 200 respinse to make the POD as running and ready to receive request from kubernetes Service
                  env: # env Variable for the container
                    - name: SPRING_PROFILES_ACTIVE # name 0f the env Variable 
                      value: production-microservice # value of the env Variable
                  resources: # setting up the resource request for the api-gateway POD container
                    requests:
                        memory: 350Mi # setting up the resources in the range as 350Mi i.e 350 Megabyte 
                        cpu: 100m # setting up the resources in this request as  100millicore

        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hence defined as below 
        kind: Deployment # type of the kubernetes object is Deployment
        metadata: # name of the Deployment been described here
            name: webapp
            namespace: default # here the namespace it belong to is default
        spec: # defining the specification for the Deployment in this case
            selector: # selector for the deployment based on POD label being defined in here
                matchLabels:
                    app: webapp
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container

                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: webapp
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                    - name: webapp # name of the container
                      image: richardchesterwood/k8s-fleetman-webapp-angular:release2 # image of the container changing the tags to resources
                      env: # env variable defined for the container
                        - name: SPRING_PROFILES_ACTIVE # name of the env Variable
                          value: production-microservice # value of the env Variable
                      resources:
                        requests:
                            memory: 50Mi
                            cpu: 50m


        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hence defined as below
        kind: Deployment # type of the kubernetes object is Deployment
        metadata: # name of the Deployment been described here
            name: queueapp
        spec: # here the spoecification for the Deployment been defined in here
            replicas: 1 # here we are spinning 1 replica in this case
            selector: # selector for the deployment based on POD label being defined in here
                matchLabels:
                    app: queueapp
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: queueapp
                spec: # specification for the POC container
                    containers: # container details been provided here
                    - name: queueapp # name of the container
                    image: richardchesterwood/k8s-fleetman-queue:resources # image of the container changing the tags to resources
                    resources: # reouces for the container
                        requests: # here requesting for the resources in the POD container
                            memory: 300Mi # we need the Memory of 300 megabyte
                            cpu: 100m # we need the cpu of 100millicore


    ```

- there are `few paramter` that `we can set` to the `Readiness and Livenss Probe` which is as below 

- we can get the `documentation` of the `Readiness and LiveNess Probe` at [Readiness and Liveness Docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

- some of the `additional paramter` are as below 

    - `initialDelaySeconds`  :-
      
      - `Number of seconds` `after the container has started before startup, liveness or readiness probes are initiated`
      
      - `If a startup probe is defined`, `liveness and readiness probe delays do not begin until the startup probe has succeeded`.  



    - `periodSeconds` :-
      
      - `How often (in seconds) to perform the probe`
      
      - `Default to 10 seconds`
      
      - `The minimum value is 1`



    - `timeoutSeconds` :-
      
      - `Number of seconds after which the probe times out`. 
      
      - `Defaults to 1 second. Minimum value is 1.` 



    - `successThreshold` :-
      
      - `Minimum consecutive successes for the probe to be considered successful after having failed`. 
      
      - `Defaults to 1`. 
      
      - `Must be 1 for liveness and startup Probes`. 
      
      - `Minimum value is 1`. 


    - `failureThreshold` :- 
      
      - `After a probe fails failureThreshold times in a row, Kubernetes considers that the overall check has failed: the container is not ready/healthy/live`



- we need to rest the `Deployment back` to the `1 replica` in order to `perform the action back again`

- hence we can set the `set the replicas for the api-gateway Deployment` as `1` in the `workloads.yml` file in this case

- now we can `Deploy the changes to the cluster by apply the changes` as below 

    ```bash
        kubectl apply -f workloads.yml
        # Deploy the changes to the cluster by apply the changes
        # the output will be as below 
        deployment.apps/position-simulator unchanged
        deployment.apps/position-tracker unchanged
        deployment.apps/api-gateway configured
        deployment.apps/webapp unchanged
        deployment.apps/queueapp unchanged

    ```

- again here we need to `increase the replicas of the Deployment as 3` inside the `workloads.yml` and poerform the action as below 

    ```yaml
        workloads.yml
        ==============
        apiVersion: apps/v1 # here the Deployment belongs to the apps group hyence defined as below 
        kind: Deployment # type of the kubernetes object is deployment
        metadata: # name of the Deployment been described here
            name: position-simulator
        spec: # specification for the Deployment been defined here
            selector: # selector to select the POD based on the POD label
                matchLabels:
                    app: position-simulator
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: position-simulator
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                    - name: position-simulator # name of the container
                      image: richardchesterwood/k8s-fleetman-position-simulator:resources # image of the container changing the tags to resources
                      env: # env Variable for the container
                        - name: SPRING_PROFILES_ACTIVE # name 0f the env Variable 
                          value: production-microservice # value of the env Variable
                      resources: # setting up the resource request for the position-simulator POD container
                        requests:
                            memory: 350Mi # setting up the resources in the range as 350Mi i.e 350 Megabyte 
                            cpu: 100m # setting up the resources in this request as  100millicore

        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hence defined as below 
        kind: Deployment # type of the kubernetes object is deployment
        metadata: # name of the Deployment been described here
            name: position-tracker
        spec: # specification for the Deployment been defined here
            selector: # selector to select the POD based on the POD label
                matchLabels:
                    app: position-tracker
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container

                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: position-tracker
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                    - name: position-tracker # name of the container
                      image: richardchesterwood/k8s-fleetman-position-tracker:resources # image of the container changing the tags to resources
                      env: # env Variable for the container
                        - name: SPRING_PROFILES_ACTIVE # name 0f the env Variable 
                          value: production-microservice # value of the env Variable
                      resources: # setting up the resource request for the position-tracker POD container
                        requests:
                            memory: 350Mi # setting up the resources in the range as 350Mi i.e 350 Megabyte 
                            cpu: 100m # setting up the resources in this request as  100millicore


        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hence defined as below 
        kind: Deployment # type of the kubernetes object is Deployment
        metadata: # name of the Deployment been described here
            name: api-gateway
            namespace: default # here the namespace for the POD being as default
        spec: # defining the specification for the Deployment in this case
            selector: # selector for the deployment based on POD label being defined in here
                matchLabels:
                    app: api-gateway
            replicas: 3 #replicas of the POD defined as 3 at any instance in order to scale up # imp
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: api-gateway
            spec: # Specification of the POD container been defined in here
                containers: # container details provided here
                - name: api-gateway # name of the container
                  image: richardchesterwood/k8s-fleetman-api-gateway:performance # image of the container changing the tags to resources
                  readinessProbe: # here we are setting up the readioness probe for the api-gateway POD created from the Deployment
                    httpGet: # here we will be sending the http Get request and the POD will be only marked as `running` if the httpGet request is successful
                        path: / # here the path for the request as / as the nginx web frontend hit the api-gateway as /api and when it hit the Api Gateway then /api will be removed hence we are providing as / in this case
                        port: 8080 # here the port in which it will try to access the apiGateway Service Port which is 8080 to get the 200 respinse to make the POD as running and ready to receive request from kubernetes Service
                  env: # env Variable for the container
                    - name: SPRING_PROFILES_ACTIVE # name 0f the env Variable 
                      value: production-microservice # value of the env Variable
                  resources: # setting up the resource request for the api-gateway POD container
                    requests:
                        memory: 350Mi # setting up the resources in the range as 350Mi i.e 350 Megabyte 
                        cpu: 100m # setting up the resources in this request as  100millicore

        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hence defined as below 
        kind: Deployment # type of the kubernetes object is Deployment
        metadata: # name of the Deployment been described here
            name: webapp
            namespace: default # here the namespace it belong to is default
        spec: # defining the specification for the Deployment in this case
            selector: # selector for the deployment based on POD label being defined in here
                matchLabels:
                    app: webapp
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container

                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: webapp
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                    - name: webapp # name of the container
                      image: richardchesterwood/k8s-fleetman-webapp-angular:release2 # image of the container changing the tags to resources
                      env: # env variable defined for the container
                        - name: SPRING_PROFILES_ACTIVE # name of the env Variable
                          value: production-microservice # value of the env Variable
                      resources:
                        requests:
                            memory: 50Mi
                            cpu: 50m


        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hence defined as below
        kind: Deployment # type of the kubernetes object is Deployment
        metadata: # name of the Deployment been described here
            name: queueapp
        spec: # here the spoecification for the Deployment been defined in here
            replicas: 1 # here we are spinning 1 replica in this case
            selector: # selector for the deployment based on POD label being defined in here
                matchLabels:
                    app: queueapp
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: queueapp
                spec: # specification for the POC container
                    containers: # container details been provided here
                    - name: queueapp # name of the container
                    image: richardchesterwood/k8s-fleetman-queue:resources # image of the container changing the tags to resources
                    resources: # reouces for the container
                        requests: # here requesting for the resources in the POD container
                            memory: 300Mi # we need the Memory of 300 megabyte
                            cpu: 100m # we need the cpu of 100millicore




    ```

- now we can `Deploy the changes to the cluster by apply the changes` as below 

    ```bash
        kubectl apply -f workloads.yml
        # Deploy the changes to the cluster by apply the changes
        # the output will be as below 
        deployment.apps/position-simulator unchanged
        deployment.apps/position-tracker unchanged
        deployment.apps/api-gateway configured
        deployment.apps/webapp unchanged
        deployment.apps/queueapp unchanged

    ```

- but before that we will making a `curl request to the API Gateway` using the `for loop infiniutely` in a `seprate terminal`

    ```bash
        while true; do
        curl http://192.168.49.2:30030/api ; echo ;
        done
        # here we can get the minikube IP by using the command as `minikube ip` command
        # here we can also get the minikube Nodeport using the command as minikube service --url <Service name which is fleetman-webapp> 
        # echpo command wil make it readable by new line
        # the output will be as below 
        # here we aqwill be performing the ionfinity loop


        # imp
        # here as we have set the ReadinessProbe we will not be getting the 502 Bad Gateway Error
        # here the API Gateway POD microservicce will marked as ready only if the ReadinessProbe is successful
        # kubernetes service will not send the traffic request to the New POD untill the POD readiness probe being ready
        # here we can see that even though we have increased the replica of the Deployment and POD are getting ready but not serviceable but we will not see error in this case
        <p>Fleetman API Gateway at Sat Mar 09 08:16:07 GMT 2024</p>
        <p>Fleetman API Gateway at Sat Mar 09 08:16:07 GMT 2024</p>
        <p>Fleetman API Gateway at Sat Mar 09 08:16:07 GMT 2024</p>
        <p>Fleetman API Gateway at Sat Mar 09 08:16:07 GMT 2024</p> 

        # when we peform the automatic Autoscaling using the HPA rule this kind of scale up will happen and which can put system in instability and there will be doenntime for the Users
    
    ```

- now when we check the `kubernetes POD container` after `bumping the request` we can see the details as below 

    ```bash
        kubectl get pods                                                                              (minikube/default)
        # here we are performing the kubectl getpod to fetch all the PODs inside the default namespace
        # the result will be as below 
        NAME                                 READY   STATUS    RESTARTS         AGE
        api-gateway-7b48df45dc-kbrvn         0/1     ContainerCreating   0      28s  
        api-gateway-7b48df45dc-mmkg2         0/1     Running   0                28s ## here we can see that its taking time to see the make the POD avialable to server request after the POD container been runnign
        api-gateway-7b48df45dc-mzpzx         1/1     Running   0                6m37s
        mongodb-5558cf99d9-l28qp             1/1     Running   52 (3h50m ago)   2d16h
        position-simulator-f9c8794ff-qtwgn   1/1     Running   5 (3h50m ago)    3d6h
        position-tracker-bb49fdd8f-z79ld     1/1     Running   5 (3h50m ago)    3d6h
        queueapp-7fff87bc5d-htbxr            1/1     Running   5 (3h50m ago)    3d7h
        webapp-587c5cbd6d-v6sjt              1/1     Running   11 (3h49m ago)   2d16h

        # we can see the rediness Probe info about the POd if we are performing the kubectl describe pod <pod name>
        # here we will be getting the response as below in that case
        kubectl describe pod api-gateway-7b48df45dc-mmkg2 
        # here we are describing the POD to see the readiness info here
        # the output will be as below 
        Name:             api-gateway-7b48df45dc-mmkg2
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Sat, 09 Mar 2024 16:42:28 +0530
        Labels:           app=api-gateway
                        pod-template-hash=7b48df45dc
        Annotations:      <none>
        Status:           Running
        IP:               10.244.1.6
        IPs:
        IP:           10.244.1.6
        Controlled By:  ReplicaSet/api-gateway-7b48df45dc
        Containers:
        api-gateway:
            Container ID:   docker://ecaa1f77c658dc38903f25dc1605389295e4e4b30cc367a838dc14bcbf0cf455
            Image:          richardchesterwood/k8s-fleetman-api-gateway:performance
            Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-api-gateway@sha256:19ea0588b3127f45420120de171da151bca5efacebcbc963fcb05419c12cdbe0
            Port:           <none>
            Host Port:      <none>
            State:          Running
            Started:      Sat, 09 Mar 2024 16:42:29 +0530
            Ready:          True
            Restart Count:  0
            Requests:
            cpu:      100m
            memory:   350Mi
            Readiness:  http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3  # here we can see the Rediness Probe info in here
            Environment:
            SPRING_PROFILES_ACTIVE:  production-microservice
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-28xwh (ro)
        Conditions:
        Type              Status
        Initialized       True 
        Ready             True 
        ContainersReady   True 
        PodScheduled      True 
        Volumes:
        kube-api-access-28xwh:
            Type:                    Projected (a volume that contains injected data from multiple sources)
            TokenExpirationSeconds:  3607
            ConfigMapName:           kube-root-ca.crt
            ConfigMapOptional:       <nil>
            DownwardAPI:             true
        QoS Class:                   Burstable
        Node-Selectors:              <none>
        Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                    node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
        Events:
        Type     Reason     Age                    From               Message
        ----     ------     ----                   ----               -------
        Normal   Scheduled  6m58s                  default-scheduler  Successfully assigned default/api-gateway-7b48df45dc-mmkg2 to minikube
        Normal   Pulled     6m59s                  kubelet            Container image "richardchesterwood/k8s-fleetman-api-gateway:performance" already present on machine
        Normal   Created    6m58s                  kubelet            Created container api-gateway
        Normal   Started    6m58s                  kubelet            Started container api-gateway
        Warning  Unhealthy  6m49s (x3 over 6m58s)  kubelet            Readiness probe failed: Get "http://10.244.1.6:8080/": dial tcp 10.244.1.6:8080: connect: connection refused


    ```

- we do need the `Readiness Probe for all the Deployment POD container definition` if we are performing the `HPA autoscaling for the Deployment` and we want to `avoid the system instability and downtime error`

- **How to confiugure the liveness Probe in kubernetes**

- `Liveness Probe` is `different` in case to the `Readiness Probe` 

- A `Liveness Probe` would be a `http request` exactly same as we set for the `Readiness Probe` , but `Liveness Probe` will continually run `for the lifetime pof the POD container lifetime` , if any case the `Liveness Probe` fails then kubernetes will mark the `POD as failed` and `effectively restart the POD` i.e it will restart the `POD container` 

- if there any `container` which got `freeze for a reason` then we can utilize the `Liveness Probe` give the `POD container a restart once it got froze` , which can be `useful in some situation`

- with the current discussion , all this `Readiness and Liveness probe` make sense for the `web  server or web container` , if we have a `non web container` then  we can to follow the below request  

- **How to make the Liveness And Readiness Probe for a non webserver or non web container**

- here we can configure the `readiness or liveness command` apart from the `http request`

- we can configure this as below `liveness-probe.yml` as below 

    ```yaml
        liveness-probe.yml
        ==================
        apiVersion: v1 # here the apiVersion for the POD is v1 as it belong to the core group
        kind: Pod # here the kubernetes object will be as POD
        metadata: # here providing the name of the POD as liveness-exec 
            name: liveness-exec 
            labels: # providing the labels for the POD to be selected by the kubernetes Deployment or Service Selector based on POD labels
                test: liveness
        spec: # definign the specification for the POD container
            containers: # here are the contianer details in this case
                - name: liveness # name of the POD in this case
                  image: k8s.gcr.io/busybox # here this is the image inside the kubernetes registry
                  args: # here providing the command args which will be executed as soon as the container starts which will create the file called healthy & remove file healthy later after 30 sec and then gors for sleep of 600sec
                    - /bin/sh
                    - -c
                    - touch /tmp/healthy ; sleep 30 ; rm -rf /tmp/healthy ; sleep 600
                  livenessProbe: # here setting up the livenessProbe in here
                    exec: # providing the exec ommand in here
                        command: # here command been provided which will be tried after every 5 minute
                            - cat
                            - /tmp/healthy
                    initialDelaySeconds: 5 # here it will wait for 5 seconds after the container start and probe begin
                    periodSeconds: 5 # here we are probe request in 5 sec interval

    ```

- now we can `Deploy the changes to the cluster by apply the changes` as below 

    ```bash
        kubectl applu -f liveness-probe.yml
        # Deploy the changes to the cluster by apply the changes
        # the output will be as below 
        pod/liveness-exec created

        kubectl get pods
        # fetching all the PODs inside the default namespace
        # the output will be as below in this case
        NAME                                 READY   STATUS    RESTARTS         AGE
        api-gateway-7b48df45dc-kbrvn         1/1     Running   0                37m
        api-gateway-7b48df45dc-mmkg2         1/1     Running   0                37m
        api-gateway-7b48df45dc-mzpzx         1/1     Running   0                43m
        liveness-exec                        1/1     Running   0                6s
        mongodb-5558cf99d9-l28qp             1/1     Running   52 (4h27m ago)   2d17h
        position-simulator-f9c8794ff-qtwgn   1/1     Running   5 (4h27m ago)    3d7h
        position-tracker-bb49fdd8f-z79ld     1/1     Running   5 (4h27m ago)    3d7h
        queueapp-7fff87bc5d-htbxr            1/1     Running   5 (4h27m ago)    3d7h
        webapp-587c5cbd6d-v6sjt              1/1     Running   11 (4h26m ago)   2d17h

        kubectl describe pod liveness-exec
        # here fetching the liveness-exec POD that we created describing that info 
        # the below will be the output 
        Name:             liveness-exec
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Sat, 09 Mar 2024 17:20:12 +0530
        Labels:           test=liveness
        Annotations:      <none>
        Status:           Running
        IP:               10.244.1.8
        IPs:
        IP:  10.244.1.8
        Containers:
        liveness:
            Container ID:  docker://58279c559d3b2c8a1c37bc179a691020d0544a61c39a004d4054fd1811dc04f8
            Image:         k8s.gcr.io/busybox
            Image ID:      docker-pullable://k8s.gcr.io/busybox@sha256:d8d3bc2c183ed2f9f10e7258f84971202325ee6011ba137112e01e30f206de67
            Port:          <none>
            Host Port:     <none>
            Args:
            /bin/sh
            -c
            touch /tmp/healthy ; sleep 30 ; rm -rf /tmp/healthy ; sleep 600
            State:          Running
            Started:      Sat, 09 Mar 2024 17:22:45 +0530
            Last State:     Terminated
            Reason:       Error
            Exit Code:    137
            Started:      Sat, 09 Mar 2024 17:21:30 +0530
            Finished:     Sat, 09 Mar 2024 17:22:43 +0530
            Ready:          True
            Restart Count:  2
            Liveness:       exec [cat /tmp/healthy] delay=5s timeout=1s period=5s #success=1 #failure=3 # here we can see the info about the liveness probe here
            Environment:    <none>
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-x5j5r (ro)
        Conditions:
        Type              Status
        Initialized       True 
        Ready             True 
        ContainersReady   True 
        PodScheduled      True 
        Volumes:
        kube-api-access-x5j5r:
            Type:                    Projected (a volume that contains injected data from multiple sources)
            TokenExpirationSeconds:  3607
            ConfigMapName:           kube-root-ca.crt
            ConfigMapOptional:       <nil>
            DownwardAPI:             true
        QoS Class:                   BestEffort
        Node-Selectors:              <none>
        Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                    node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
        Events:
        Type     Reason     Age                  From               Message
        ----     ------     ----                 ----               -------
        Normal   Scheduled  3m28s                default-scheduler  Successfully assigned default/liveness-exec to minikube
        Normal   Pulled     3m24s                kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 3.697s (3.697s including waiting)
        Normal   Pulled     2m11s                kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 2.18s (2.18s including waiting)
        Normal   Pulling    58s (x3 over 3m28s)  kubelet            Pulling image "k8s.gcr.io/busybox"
        Normal   Created    56s (x3 over 3m24s)  kubelet            Created container liveness
        Normal   Started    56s (x3 over 3m24s)  kubelet            Started container liveness
        Normal   Pulled     56s                  kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 1.997s (1.997s including waiting)
        Warning  Unhealthy  13s (x9 over 2m53s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
        Normal   Killing    13s (x3 over 2m43s)  kubelet            Container liveness failed liveness probe, will be restarted
 

    ```

- we can also set up the `TCP liveness or readiness Probe` ,like how we have set for the `httpGet and exec command`