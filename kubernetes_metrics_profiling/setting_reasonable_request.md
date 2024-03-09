# Setting reasonable Requests

- `kubectl top pod` command will not be a `replacement` for the `proper profiling` but does provide `some proper info`

- but after `tuning the java application` we can set some `reasonable good request` for the `java application POD container will be`

- here if we do a `kubectl top pod` now then we can see the `java application POD` such as `Api-gateway` , `position-tracker` & `position-simulator` POD container we can see that thats `not be consuming more than 350Mi i.e 350Mb of RAM memory size`

    ```bash
        kubectl top pod
        # here we are executing the kubectl top command in here
        # this will provide the resource utilization inside the cluster in this case
        # this will provide the number of CPU and RAM Memory we are usage inside the POD container
        NAME                                  CPU(cores)   MEMORY(bytes)   
        api-gateway-57f4df64d6-c8ltm          19m          243Mi           
        mongodb-659cb5db54-v67ts              989m         331Mi           
        position-simulator-7cb7567b8c-cnn8b   33m          198Mi           
        position-tracker-7d4c8b5965-n8wmr     37m          320Mi           
        queueapp-7fff87bc5d-xtt2r             48m          262Mi           
        webapp-66765b68df-8cfbr               0m           3Mi             

    ```

- with the `same token` we can mark those `java application POD container` such as `Api-gateway` , `position-tracker` & `position-simulator` POD container will be taking `50 millicore CPU or 100 muillicore CPU` in this range which will be the  `good values for the Java Application POD container`

- here for the `queueapp` also we can set the `RAM Memopry request` as `300Mi or 300Mb` in this case and `CPU Utilization resource` as `50 millicore` 

- for the `mongodb POD container` we can see that `its been consistent` around the `300-350Mi or Megabyte Range for RAM Memory` hence the `request of 400Mi or 400Mb will be suffice for the mongodb POD container`

- we can also define the `50Mi of RAM Memory` and `50m of CPU` in case of the `webapp POD container`

- hence in here we can set the `request`, which is not `limits` , the `resources requests` will help in `schedule` the `POD` that `which node` will be `prefered to run which POD` in this case  `schedule it accordingly`

- `resources` will also have an `effect over the eviction of the POD on the kubernetes Node if the Nodes starts running the out of Memory` which we will learn soon

- we can set the `reasonable requests` on individual `POD container` inside the `workloads.yml` 

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
                      image: richardchesterwood/k8s-fleetman-api-gateway:resources # image of the container changing the tags to resources
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
                    resources: # swetting up the resource for the webapp POD container in this case
                        requests:
                            memory: 50Mi # setting as 50Mi i.e 50 MegaByte in this case
                            cpu: 50m # setting up the 50millicore in this requests


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

- we can also setup the `resources requests` for the `mongo POD` inside the `mongo-stack.yml` file as below 

    
    ```yaml
        mongo-stack.yml
        ===============
        apiVersion: apps/v1 # defining the apiVersion as apps/v1 for the Deployment as it exist in apps group
        kind: Deployment # defining the type of object in this case as Deployment
        metadata: # defining the name for the Deployment out in here
            name: mongodb
        spec: # defining the specification for the Deployment
            replicas: 1 # using 1 replica-set which will responsible to manage the POD 
            selector: # defining the selector key value pair based on the POD label
                matchLabels:
                    app: mongodb
            template: # defining the POD definition inside the template derivatiove
                metadata: # defining the POD label inside the metadata section
                    labels: # defining the POD label as key and value pair
                        app: mongodb
                spec: # defining the specification for the POD definition
                    containers: # defining the containers inside the POD
                    - name: mongodb # name of the container as mongodb
                      image: mongo:3.6.5-jessie # image  we will beusing for the container
                      volumeMounts: # using the volumeMount which will define the lis of folder inside the container we want to map outside 
                        - name: mongo-persist-volume # using the name of the VolumeMount which will be used in the volumes section to refer which folder inside container we want map as there can be multiple folder inside the container map outside of the container 
                          mountPath: /data/db # defining the folder we want to map inside the container
                      resources: # setting up the resource requests in this case in here
                        requests:
                            memory: 400Mi # here setting up the resource request for 400MegaByte for the Memory
                            cpu: 1000m  # here setting up 1000 millicore for the CPU resource

                    volumes: # defining the volume which will tell how we will implement the volume mapping to a hostPath/awsEbsStorage or anything 
                    - name: mongo-persist-volume # referencing the volumeMount based on their name provided
                        persistentVolumeClaim: # defining the persistanceVolumeClaim which will be a separate confifuration file to define how to implement the volume mapping or mount in hostPath/awsEBSStorage
                            claimName: mongo-pvc # defining the claimName which need to be defined inside  the separate confifuration file with the same name in order to claim some storage for the volume
                            # here this mongo-pvc claim name should be defined in the separate configuration file which will define the how to implement volume mapping and also it will claim the spaces for the Persistant Volume



        ---
        # defining the documentSeparator over here 

        apiVersion: v1 # defining the apiVersion as v1 in here as it belong to the core group 
        kind: Service # defining the kubernetes object type as Service in here
        metadata: # defining the name for the Service whioch should exactly name as fleetman-mongodb
        name: fleetman-mongodb
        spec: # defining the spec for the Service defined here
        selector: # this will select the POD based on label same as the key-value pair
            app: mongodb
        ports: # defining the ports that need to exposed for the Service
        - name: mongoport # name for the port defined as mongoport
            port: 27017 # defining the port number as 27017 in here
            protocol: TCP # defining the protocol as TCP

        type: ClusterIP # defining it as ClusterIp service which will not avaialble outside of the minikube kubernetes local cluster


    ```

- we can `Deploy the Resources` into the `kubernetes cluster` by applying the `changes` as below 

    ```bash
        kubectl apply -f workloads.yml
        # here applying the changes to deploy the resources into the kubernetes cluster in this case
        # we will get the response as below 
        deployment.apps/position-simulator configured
        deployment.apps/position-tracker configured
        deployment.apps/api-gateway configured
        deployment.apps/webapp configured
        deployment.apps/queueapp configured

        kubectl apply -f mongo-stack.yml
        # here applying the changes to deploy the resources into the kubernetes cluster in this case
        # we will get the response as below 
        deployment.apps/mongodb configured
        service/fleetman-mongodb unchanged

    ```

- now when we perform `kubectl top pod` we can see the `usage` for the `POD` on the `default namespace` as below 

    ```bash
        kubectl top pod
        # here we are executing the kubectl top command in here
        # this will provide the resource utilization inside the cluster in this case
        # this will provide the number of CPU and RAM Memory we are usage inside the POD container
        # here there will no significant change as we have only set the requests
        NAME                                 CPU(cores)   MEMORY(bytes)   
        api-gateway-697d5d48bd-hzcrv         20m          234Mi           
        mongodb-659cb5db54-v67ts             993m         343Mi           
        position-simulator-f9c8794ff-qtwgn   30m          187Mi           
        position-tracker-bb49fdd8f-z79ld     25m          302Mi           
        queueapp-7fff87bc5d-htbxr            29m          345Mi           
        webapp-66765b68df-477x6              0m           3Mi    

        # but if we describe the minikube node in that case we can see the changes as below 
        kubectl describe node minikube 
        # here we are describing the minikube kubernetes nodes which is available in minikube kubernetes cluster we can see the below details 
        # here we can see the allocated CPU and Memory Usage for the entire minikube kubernetes Node
        # also we can see that CPU and Memory Usage of all POD running on the Node
        Name:               minikube
        Roles:              control-plane
        Labels:             beta.kubernetes.io/arch=amd64
                            beta.kubernetes.io/os=linux
                            kubernetes.io/arch=amd64
                            kubernetes.io/hostname=minikube
                            kubernetes.io/os=linux
                            minikube.k8s.io/commit=8220a6eb95f0a4d75f7f2d7b14cef975f050512d
                            minikube.k8s.io/name=minikube
                            minikube.k8s.io/primary=true
                            minikube.k8s.io/updated_at=2024_03_04T15_49_36_0700
                            minikube.k8s.io/version=v1.32.0
                            node-role.kubernetes.io/control-plane=
                            node.kubernetes.io/exclude-from-external-load-balancers=
        Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/cri-dockerd.sock
                            node.alpha.kubernetes.io/ttl: 0
                            volumes.kubernetes.io/controller-managed-attach-detach: true
        CreationTimestamp:  Mon, 04 Mar 2024 15:49:33 +0530
        Taints:             <none>
        Unschedulable:      false
        Lease:
        HolderIdentity:  minikube
        AcquireTime:     <unset>
        RenewTime:       Wed, 06 Mar 2024 10:23:17 +0530
        Conditions:
        Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
        ----             ------  -----------------                 ------------------                ------                       -------
        MemoryPressure   False   Wed, 06 Mar 2024 10:23:10 +0530   Tue, 05 Mar 2024 15:26:07 +0530   KubeletHasSufficientMemory   kubelet has sufficient memory available
        DiskPressure     False   Wed, 06 Mar 2024 10:23:10 +0530   Tue, 05 Mar 2024 15:26:07 +0530   KubeletHasNoDiskPressure     kubelet has no disk pressure
        PIDPressure      False   Wed, 06 Mar 2024 10:23:10 +0530   Tue, 05 Mar 2024 15:26:07 +0530   KubeletHasSufficientPID      kubelet has sufficient PID available
        Ready            True    Wed, 06 Mar 2024 10:23:10 +0530   Tue, 05 Mar 2024 15:26:07 +0530   KubeletReady                 kubelet is posting ready status
        Addresses:
        InternalIP:  192.168.49.2
        Hostname:    minikube
        Capacity:
        cpu:                16
        ephemeral-storage:  153187824Ki
        hugepages-1Gi:      0
        hugepages-2Mi:      0
        memory:             16331356Ki
        pods:               110
        Allocatable:
        cpu:                16
        ephemeral-storage:  153187824Ki
        hugepages-1Gi:      0
        hugepages-2Mi:      0
        memory:             16331356Ki
        pods:               110
        System Info:
        Machine ID:                 95ec6cb4fc594350b039919b0ad12d15
        System UUID:                eef55e80-d837-43a5-934f-46fb87bcbe6d
        Boot ID:                    1227659b-f6d6-4d7b-9491-1a33305473a1
        Kernel Version:             6.5.0-21-generic
        OS Image:                   Ubuntu 22.04.3 LTS
        Operating System:           linux
        Architecture:               amd64
        Container Runtime Version:  docker://24.0.7
        Kubelet Version:            v1.28.3
        Kube-Proxy Version:         v1.28.3
        PodCIDR:                      10.244.0.0/24
        PodCIDRs:                     10.244.0.0/24
        Non-terminated Pods:          (16 in total)
        # the below will show all the kiubernetes PODS CPU and memory usage as copmparred to the available CPU and Memory of the Node
        Namespace                   Name                                          CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
        ---------                   ----                                          ------------  ----------  ---------------  -------------  ---
        default                     api-gateway-697d5d48bd-hzcrv                  100m (0%)     0 (0%)      350Mi (2%)       0 (0%)         8m59s
        default                     mongodb-5558cf99d9-l28qp                      1 (6%)        0 (0%)      400Mi (2%)       0 (0%)         3s
        default                     position-simulator-f9c8794ff-qtwgn            100m (0%)     0 (0%)      350Mi (2%)       0 (0%)         6m15s
        default                     position-tracker-bb49fdd8f-z79ld              100m (0%)     0 (0%)      350Mi (2%)       0 (0%)         8m59s
        default                     queueapp-7fff87bc5d-htbxr                     100m (0%)     0 (0%)      300Mi (1%)       0 (0%)         45m
        default                     webapp-66765b68df-477x6                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         45m
        kube-system                 coredns-5dd5756b68-qx6jm                      100m (0%)     0 (0%)      70Mi (0%)        170Mi (1%)     42h
        kube-system                 etcd-minikube                                 100m (0%)     0 (0%)      100Mi (0%)       0 (0%)         42h
        kube-system                 kube-apiserver-minikube                       250m (1%)     0 (0%)      0 (0%)           0 (0%)         42h
        kube-system                 kube-controller-manager-minikube              200m (1%)     0 (0%)      0 (0%)           0 (0%)         42h
        kube-system                 kube-proxy-qdrps                              0 (0%)        0 (0%)      0 (0%)           0 (0%)         42h
        kube-system                 kube-scheduler-minikube                       100m (0%)     0 (0%)      0 (0%)           0 (0%)         42h
        kube-system                 metrics-server-7c66d45ddc-4m245               100m (0%)     0 (0%)      200Mi (1%)       0 (0%)         42h
        kube-system                 storage-provisioner                           0 (0%)        0 (0%)      0 (0%)           0 (0%)         42h
        kubernetes-dashboard        dashboard-metrics-scraper-7fd5cb4ddc-f24f5    0 (0%)        0 (0%)      0 (0%)           0 (0%)         42h
        kubernetes-dashboard        kubernetes-dashboard-8694d4445c-dndp8         0 (0%)        0 (0%)      0 (0%)           0 (0%)         42h
        Allocated resources:
        (Total limits may be over 100 percent, i.e., overcommitted.)
        Resource           Requests      Limits
        --------           --------      ------
        cpu                2300m (14%)   0 (0%) # here we are describing the for CPu we are utilizing 14% of the Node CPU that is available
        memory             2170Mi (13%)  170Mi (1%) # here we are requesting for the 13% of the overall kubernetes Node Memory
        ephemeral-storage  0 (0%)        0 (0%)
        hugepages-1Gi      0 (0%)        0 (0%)
        hugepages-2Mi      0 (0%)        0 (0%)

        
        Events:              <none>

        # one more thing to observe in this case as Few of the Other PODs in other namespace habve not define the resource request for the CPU and Memory
        # hence when we do the kubectl top node then we can see some discripancy in this case
        
        kubectl top node
        # checking the kubernetes Nodes CPU and Memory Resource Usage in this case
        NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
        minikube   1492m        9%     1954Mi          12%


    ```

- when we `now move to the production cluster` then the `kubernetes Scheduler` will `make best decesssion` that `which POD will placed and run on which node` in this case

- 