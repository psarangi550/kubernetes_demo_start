# QoS labels

- every `POD` inside the `kubernetes cluster` will be allocated a `Qos Label class` by the `kubernetes Scheduler` based on the `kuberentes requests and limits set on the POD`

- there are `3 possible values` for the `quality of Service class label for the POD`

  - `QoS:Guaranteed` :- when the `POD` have both the `CPU or Memory requests and limits set` and `both the CPU and RAM Memory requests and limits` having the `same value`
  
  - `Qos:Burstable` :- when the `POD` have the `CPU and/or Memory Request set but does not have the limits set`
  
  - `QoS:BestEffort` :- when the `POD` does the specify the `CPU or Memory requests and limits`  at all
  
- these `Qos class label` has big impact over the `kubernetes scheduler` if the `Kubernetes Nodes start running out of resources i.e CPU or Memory`

- if we want to see the `QoS class label` then we can perform as below in this case

- we can set the `workloads.yml` file accordingly 

    
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
            replicas: 1 #replicas of the POD defined as 1 at any instance
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: api-gateway
            spec: # Specification of the POD container been defined in here
                containers: # container details provided here
                - name: api-gateway # name of the container
                  image: richardchesterwood/k8s-fleetman-api-gateway:performance # image of the container changing the tags to resources
                  env: # env Variable for the container
                    - name: SPRING_PROFILES_ACTIVE # name 0f the env Variable 
                      value: production-microservice # value of the env Variable
                  resources: # setting up the resource request for the api-gateway POD container
                    # imp
                    # here we will be setting up the requests and limit for both CPU and RAM Memory
                    # here the requests and limits having the same value in here to enage the Qos as  Guaranteed
                    requests:
                        memory: 350Mi # setting up the resources in the range as 350Mi i.e 350 Megabyte 
                        cpu: 100m # setting up the resources in this request as  100millicore
                    limit: # setting up the kubernetes resources limits for the api-gateway POD
                        memory: 350Mi # here setting the limit as 350Mi i.e 350Mb which is same as requests
                        cpu: 100m # here setting the CPU as 100m which is same as the CPU requested i.e 100millicore

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

- now we can `deploy the changes to the kubernetes cluster` by `applying the changes` as below 

    ```bash
        kubectl apply -f workloads.yml
        # deploying the kubernetes resources into the kubernetes cluster in this case
        # here we will be getting the output as below 
        deployment.apps/position-simulator unchanged
        deployment.apps/position-tracker unchanged
        deployment.apps/api-gateway configured
        deployment.apps/webapp unchanged
        deployment.apps/queueapp unchanged

        kubectl get pods
        # here we are checking the PODs that been spunned as below in the default namespace
        # here we can see the info as below 
        NAME                                 READY   STATUS             RESTARTS         AGE
        api-gateway-787fc6d8fc-75kp9         0/1     Running            0                2m
        mongodb-5558cf99d9-l28qp             1/1     Running            53 (5m20s ago)   4d3h
        position-simulator-f9c8794ff-qtwgn   1/1     Running            6 (5m20s ago)    4d16h
        position-tracker-bb49fdd8f-z79ld     1/1     Running            6 (5m20s ago)    4d16h
        queueapp-7fff87bc5d-htbxr            1/1     Running            6 (5m20s ago)    4d17h
        webapp-587c5cbd6d-v6sjt              0/1     CrashLoopBackOff   16 (97s ago)     4d3h


        # now if we want to check the api-gateway POD for which we have set the CPU and Memory requests and limits then we can see as below 
        # here we can see the info as below in that case by using the kubectl describe command
        kubectl  describe pod api-gateway-787fc6d8fc-75kp9
        # checking the description of the POD to fetch the QoS service class level
        Name:             api-gateway-787fc6d8fc-75kp9
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Mon, 11 Mar 2024 03:08:46 +0530
        Labels:           app=api-gateway
                        pod-template-hash=787fc6d8fc
        Annotations:      <none>
        Status:           Running
        IP:               10.244.1.22
        IPs:
        IP:           10.244.1.22
        Controlled By:  ReplicaSet/api-gateway-787fc6d8fc
        Containers:
        api-gateway:
            Container ID:   docker://87b2b9a536eaa275d0c283697519aef22b3c98e4a80e580f2ad0334606347644
            Image:          richardchesterwood/k8s-fleetman-api-gateway:performance
            Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-api-gateway@sha256:19ea0588b3127f45420120de171da151bca5efacebcbc963fcb05419c12cdbe0
            Port:           <none>
            Host Port:      <none>
            State:          Running
            Started:      Mon, 11 Mar 2024 03:08:47 +0530
            Ready:          False
            Restart Count:  0
            Limits:
            cpu:     100m
            memory:  350Mi
            Requests:
            cpu:      100m
            memory:   350Mi
            Readiness:  http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
            Environment:
            SPRING_PROFILES_ACTIVE:  production-microservice
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-frn9x (ro)
        Conditions:
        Type              Status
        Initialized       True 
        Ready             False 
        ContainersReady   False 
        PodScheduled      True 
        Volumes:
        kube-api-access-frn9x:
            Type:                    Projected (a volume that contains injected data from multiple sources)
            TokenExpirationSeconds:  3607
            ConfigMapName:           kube-root-ca.crt
            ConfigMapOptional:       <nil>
            DownwardAPI:             true
        QoS Class:                   Guaranteed # here we can see the Service class as Guaranteed
        Node-Selectors:              <none>
        Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                    node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
        Events:
        Type     Reason     Age                   From               Message
        ----     ------     ----                  ----               -------
        Normal   Scheduled  3m21s                 default-scheduler  Successfully assigned default/api-gateway-787fc6d8fc-75kp9 to minikube
        Normal   Pulled     3m21s                 kubelet            Container image "richardchesterwood/k8s-fleetman-api-gateway:performance" already present on machine
        Normal   Created    3m21s                 kubelet            Created container api-gateway
        Normal   Started    3m20s                 kubelet            Started container api-gateway
        Warning  Unhealthy  21s (x22 over 3m20s)  kubelet            Readiness probe failed: Get "http://10.244.1.22:8080/": dial tcp 10.244.1.22:8080: connect: connection refused

        kubectl describe pod position-tracker-bb49fdd8f-z79ld
        # here we can see the other POD such as the position-tracker have only set the CPU and Memory Request but not the limits
        # if we want to check that POD for the QoS service class label then we can see as below 
        # the output in that case as below 
        Name:             position-tracker-bb49fdd8f-z79ld
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Wed, 06 Mar 2024 10:14:23 +0530
        Labels:           app=position-tracker
                        pod-template-hash=bb49fdd8f
        Annotations:      <none>
        Status:           Running
        IP:               10.244.1.15
        IPs:
        IP:           10.244.1.15
        Controlled By:  ReplicaSet/position-tracker-bb49fdd8f
        Containers:
        position-tracker:
            Container ID:   docker://6e693d96b552d5a34fb162e7ca2b34db1f9027560b612ca5c55bea3bfa838a1d
            Image:          richardchesterwood/k8s-fleetman-position-tracker:resources
            Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-position-tracker@sha256:d887a3241bde3c19a580392441bc07fbc3dc138c3414217bc747b6486be48ec5
            Port:           <none>
            Host Port:      <none>
            State:          Running
            Started:      Mon, 11 Mar 2024 03:05:52 +0530
            Last State:     Terminated
            Reason:       Error
            Exit Code:    255
            Started:      Sat, 09 Mar 2024 12:52:53 +0530
            Finished:     Mon, 11 Mar 2024 03:05:26 +0530
            Ready:          True
            Restart Count:  6
            Requests:
            cpu:     100m
            memory:  350Mi
            Environment:
            SPRING_PROFILES_ACTIVE:  production-microservice
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cdpqh (ro)
        Conditions:
        Type              Status
        Initialized       True 
        Ready             True 
        ContainersReady   True 
        PodScheduled      True 
        Volumes:
        kube-api-access-cdpqh:
            Type:                    Projected (a volume that contains injected data from multiple sources)
            TokenExpirationSeconds:  3607
            ConfigMapName:           kube-root-ca.crt
            ConfigMapOptional:       <nil>
            DownwardAPI:             true
        QoS Class:                   Burstable # here we can see the Qos class label for the POD being as Burstable
        Node-Selectors:              <none>
        Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                    node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
        Events:
        Type    Reason          Age    From     Message
        ----    ------          ----   ----     -------
        Normal  SandboxChanged  8m55s  kubelet  Pod sandbox changed, it will be killed and re-created.
        Normal  Pulled          8m54s  kubelet  Container image "richardchesterwood/k8s-fleetman-position-tracker:resources" already present on machine
        Normal  Created         8m54s  kubelet  Created container position-tracker
        Normal  Started         8m54s  kubelet  Started container position-tracker


    ```

- now we can modify the `workloads.yml` and `remove the requests and limits` for `one of the POD container` to check the `Qos Level for that`

- here we can see the info as below in this case

    ```yaml
        workloads.yml
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
            replicas: 1 #replicas of the POD defined as 1 at any instance
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: api-gateway
            spec: # Specification of the POD container been defined in here
                containers: # container details provided here
                - name: api-gateway # name of the container
                  image: richardchesterwood/k8s-fleetman-api-gateway:performance # image of the container changing the tags to resources
                  env: # env Variable for the container
                    - name: SPRING_PROFILES_ACTIVE # name 0f the env Variable 
                      value: production-microservice # value of the env Variable
                    # imp
                    # here we will be not setting up the requests and limit for both CPU and RAM Memory for the api-gateway POD container
                    
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

- now we can `deploy the changes to the kubernetes cluster` by `applying the changes` as below 

    ```bash
        kubectl apply -f workloads.yml
        # deploying the kubernetes resources into the kubernetes cluster in this case
        # here we will be getting the output as below 
        deployment.apps/position-simulator unchanged
        deployment.apps/position-tracker unchanged
        deployment.apps/api-gateway configured
        deployment.apps/webapp unchanged
        deployment.apps/queueapp unchanged

        kubectl get pods
        # here we are checking the PODs that been spunned as below in the default namespace
        # here we can see the info as below 
        NAME                                 READY   STATUS             RESTARTS         AGE
        api-gateway-8545dcfc6c-7j77h         1/1     Running            0                31s
        mongodb-5558cf99d9-l28qp             1/1     Running            53 (15m ago)     4d3h
        position-simulator-f9c8794ff-qtwgn   1/1     Running            6 (15m ago)      4d17h
        position-tracker-bb49fdd8f-z79ld     1/1     Running            6 (15m ago)      4d17h
        queueapp-7fff87bc5d-htbxr            1/1     Running            6 (15m ago)      4d17h
        webapp-587c5cbd6d-v6sjt              0/1     CrashLoopBackOff   18 (3m29s ago)   4d3h

        # now if we want to check the api-gateway POD for which we have set the CPU and Memory requests and limits then we can see as below 
        # here we can see the info as below in that case by using the kubectl describe command
        kubectl  describe pod api-gateway-8545dcfc6c-7j77h
        # the output will be as below in that case
        Name:             api-gateway-8545dcfc6c-7j77h
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Mon, 11 Mar 2024 03:20:11 +0530
        Labels:           app=api-gateway
                        pod-template-hash=8545dcfc6c
        Annotations:      <none>
        Status:           Running
        IP:               10.244.1.26
        IPs:
        IP:           10.244.1.26
        Controlled By:  ReplicaSet/api-gateway-8545dcfc6c
        Containers:
        api-gateway:
            Container ID:   docker://54d18626b8d7ec129eaca7e7b430abc6f53fb6918c6c83c3859ebbcbf84eb3c0
            Image:          richardchesterwood/k8s-fleetman-api-gateway:performance
            Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-api-gateway@sha256:19ea0588b3127f45420120de171da151bca5efacebcbc963fcb05419c12cdbe0
            Port:           <none>
            Host Port:      <none>
            State:          Running
            Started:      Mon, 11 Mar 2024 03:20:12 +0530
            Ready:          True
            Restart Count:  0
            Readiness:      http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
            Environment:
            SPRING_PROFILES_ACTIVE:  production-microservice
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lzzq6 (ro)
        Conditions:
        Type              Status
        Initialized       True 
        Ready             True 
        ContainersReady   True 
        PodScheduled      True 
        Volumes:
        kube-api-access-lzzq6:
            Type:                    Projected (a volume that contains injected data from multiple sources)
            TokenExpirationSeconds:  3607
            ConfigMapName:           kube-root-ca.crt
            ConfigMapOptional:       <nil>
            DownwardAPI:             true
        QoS Class:                   BestEffort # here we can see the QoS service class as BestEffort in here
        Node-Selectors:              <none>
        Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                    node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
        Events:
        Type     Reason     Age                    From               Message
        ----     ------     ----                   ----               -------
        Normal   Scheduled  2m24s                  default-scheduler  Successfully assigned default/api-gateway-8545dcfc6c-7j77h to minikube
        Normal   Pulled     2m24s                  kubelet            Container image "richardchesterwood/k8s-fleetman-api-gateway:performance" already present on machine
        Normal   Created    2m24s                  kubelet            Created container api-gateway
        Normal   Started    2m23s                  kubelet            Started container api-gateway
        Warning  Unhealthy  2m22s (x2 over 2m23s)  kubelet            Readiness probe failed: Get "http://10.244.1.26:8080/": dial tcp 10.244.1.26:8080: connect: connection refused

    ```