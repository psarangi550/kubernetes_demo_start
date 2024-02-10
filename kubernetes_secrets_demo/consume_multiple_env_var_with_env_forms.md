# consuming multiple Environment varaible using the envFrom in kubernetes

- the `downside` of `preveious approach of declaring the values inside the configMap as ernv variable for the  POD` being we have to `manually decalre` the `each single env variable referencing the key of configMaps which value we want to refer` for the `POD container` which is a bit `labourious`

- if we have to declare `100 of the environment variable` in that case then `that will be very hard to declare all those 100 env variable referencing the key of the confgiMap whose value we want to refer`

- but there is a better way to declare , we can use the `envFrom` rather than `env` which is on the `same level` inside the `containers section` 

- this `envFrom` takes a `list as value ` same as the `env` , we can define the `list of configMapRef` inside the `envFrom`

- inside the `list of configMapRef` have the `name of the configMapRef defined inside of it`

- which will `goto the configMapRef` and fetch all the `variables declared as key value pair` and define them as the `env Variable`

- but we need to keep in mind that `key of the value defined in the configMaps` should be `valid key` , hence we can't `declare key `  as `database.url` or `database.password`

- hence we can rewrite the `database-uti-data.yaml` as below in this case

    ```yaml
        database-uti-data.yaml
        ======================
        apiVersion: v1
        kind: ConfigMap
        # here we are sticking to the old approiach as we are changing the key over here hence we will be making another version out of it
        # here we can declare the name of the ConfigMap as database-uri-info-v3 in this case
        metadata:
            name: database-uri-info-v3
        data: # defining the data key of the ConfiogMap where we will define the key value pair
            # here the key value pair will be valid variable which can be set as env variable in this case
            DATABASE_URI: https://mydatabase.<something>.<something>:3306
            DATABASE_PASSWORD: P@SSWOrd1
            # here defining the key and value as the valid variable which can be used as env variable with value

    ```

- we can declare the `workloads.yml` `position-simulator and position-tracker POD container` to reference this using the `envFrom` and `configMapRef` as below 

    
    ```yaml
        workloads.yml
        =============
        apiVersion: apps/v1 # here the Deployment belongs to the apps group hyence defined as below 
        kind: Deployment # type of the kubernetes object is deployment
        metadata: # name of the Deployment been described here
            name: position-simulator
        spec: # specification for the Deployment been defined here
            selector: # selector to select the POD based on the POD label
                matchlabels:
                    app: position-simulator
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: position-simulator
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                        - name: position-simulator # name of the container
                          image: richardchesterwood/k8s-fleetman-position-simulator:release2 # image for the container
                          env: # env Variable for the container
                            - name: SPRING_PROFILE_ACTIVE # name 0f the env Variable 
                              value: production-microservice # value of the env Variable
                          envFrom: # defining the envFrom in here
                            - configMapRef: # referencing the configMapRef in here
                                name: database-uri-info-v3 # name of the configMapRef to reference all the key and value pair become env variable with the name and value
        
        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hyence defined as below 
        kind: Deployment # type of the kubernetes object is deployment
        metadata:  # name of the Deployment been described here
            name: position-tracker
        spec: # specification for the Deployment been defined here
            selector: # selector to select the POD based on the POD label
                matchlabels:
                    app: position-tracker
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: position-tracker
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                        - name: position-tracker # name of the container
                          image: richardchesterwood/k8s-fleetman-position-tracker:release3 # image of the container
                          env: # env Variable for the container
                            - name: SPRING_PROFILE_ACTIVE # name 0f the env Variable 
                              value: production-microservice # value of the env Variable
                          envFrom: # defining the envFrom in here
                            - configMapRef:  # referencing the configMapRef in here
                                name: database-uri-info-v3 # name of the configMapRef to reference all the key and value pair become env variable with the name and value

    ```

- the `good thing about this approach` being `even if there were 1000 key value pair defined` all wilol become the `env variable in the POD container` we don't have to `manually declare that for each single env variable with the key of the configMap file like the last approch to reference the values of the configMap`

- if we `apply the changes` and `deploy the changes to the kubernetes cluster` for the `database-uri-data.yml` and fetch the `configmaps` then we can see the `older version as well as the newer version as well in this case`

- we can do as below for the changes

    ```bash
        kubectl apply -f database-uri-data.yml
        # here we are deploying the changes to the kubernetes cluster by applying the change
        # the output will be as below 
        configmap/database-uri-info-v3 created

        # now if we want to see the configmaps inside the cluster inside the default namespace as below
        kubectl get cm
        # fetching the configmaps from the default cluster as below 
        # here we can see all 3 version of the configMaps in this case
        NAME                   DATA   AGE
        database-uri-info      2      19h
        database-uri-info-v2   2      12h
        database-uri-info-v3   2      91s


        # now if we want to see the described details o9f the new configMap i.e database-uri-info-v3
        kubectl describe cm database-uri-info-v3
        # here we can see the dtails as below 
        Name:         database-uri-info-v3
        Namespace:    default
        Labels:       <none>
        Annotations:  <none>

        Data
        ====
        DATABASE_PASSWORD:
        ----
        P@SSWOrd1
        DATABASE_URI:
        ----
        https://mydatabase.<something>.<something>:3306

        BinaryData
        ====

        Events:  <none>


    ```

- if we `apply the changes` and `deploy the changes to the kubernetes cluster` for the `workloads.yml` then we can see the below details 

    ```bash
        kubectl apply -f workloads.yml
        # here we are deploying the changes to the kubernetes cluster by applying the change
        # the output will be as below 
        deployment.apps/queueapp unchanged
        deployment.apps/position-simulator configured
        deployment.apps/position-tracker configured
        deployment.apps/api-gateway unchanged
        deployment.apps/webapp unchanged

        # now if we are accessing the position-tracker/position-simulator POD container then we can see the details as below 
        # in order to access the position-simulator then we can do the below thing 
        kubectl get all 
        # fetching all the kubernetes workloads inside the default namespace
        NAME                                      READY   STATUS             RESTARTS        AGE
        pod/api-gateway-56c46fbcdb-mckrd          1/1     Running            2 (8m25s ago)   26h
        pod/mongodb-578b98fbd4-6m8h8              1/1     Running            2 (8m25s ago)   26h
        pod/position-simulator-68b545b847-cqlgz   1/1     Running            0               112s
        pod/position-tracker-67499f494d-qlvmn     1/1     Running            0               112s
        pod/queueapp-f55dcb97d-8z6xg              1/1     Running            2 (8m25s ago)   26h
        pod/webapp-66765b68df-7298p               0/1     CrashLoopBackOff   9 (95s ago)     26h

        NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
        service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   26h
        service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        26h
        service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         26h
        service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   26h
        service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     26h
        service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          27h

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           26h
        deployment.apps/mongodb              1/1     1            1           26h
        deployment.apps/position-simulator   1/1     1            1           26h
        deployment.apps/position-tracker     1/1     1            1           26h
        deployment.apps/queueapp             1/1     1            1           26h
        deployment.apps/webapp               0/1     1            0           26h

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-56c46fbcdb          1         1         1       26h
        replicaset.apps/mongodb-578b98fbd4              1         1         1       26h
        replicaset.apps/position-simulator-5f4dc8c48b   0         0         0       16h
        replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       26h
        replicaset.apps/position-simulator-67d9f49f5b   0         0         0       15h
        replicaset.apps/position-simulator-68b545b847   1         1         1       112s
        replicaset.apps/position-simulator-84f66bc755   0         0         0       15h
        replicaset.apps/position-simulator-bf87cb7f6    0         0         0       12h
        replicaset.apps/position-tracker-59fdfd8cf4     0         0         0       26h
        replicaset.apps/position-tracker-5dd5688987     0         0         0       12h
        replicaset.apps/position-tracker-67499f494d     1         1         1       112s
        replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       15h
        replicaset.apps/position-tracker-9cbf78bd9      0         0         0       16h
        replicaset.apps/queueapp-f55dcb97d              1         1         1       26h
        replicaset.apps/webapp-66765b68df               1         1         0       26h

        # now we can access the Position-tracker POD container as below 
        kubectl exec -it pod/position-simulator-68b545b847-cqlgz -- bash
        # fetching the kubernetes exec value in here 
        root@position-simulator-68b545b847-cqlgz:/# echo $DATABASE_URI
        # echoing the DATABASE_URI which will provide the value in this case
        https://mydatabase.<something>.<something>:3306


    ```

- but if we apply the changes to the `workloads` files first then `it will look for the refernece` of the `database-uri-info-v3` , but if we haave not deployed the `database-uri-data.yml` then those `configMap names` were not available hence we will be getting the error as `container creation failed` and old `POD` are still running

- but as soon as `we deploy the database-uri-data.yml` and `database-uri-info-v3` will be avialable hence the `POD container` will fetch the `value` and come to running 

- we can do that as below

    ```yaml
        database-uti-data.yaml
        ======================
        apiVersion: v1
        kind: ConfigMap
        # here we are sticking to the old approiach as we are changing the key over here hence we will be making another version out of it
        # here we can declare the name of the ConfigMap as database-uri-info-v3 in this case
        metadata: # here defining the metadata as database-uri-info-v4
            name: database-uri-info-v4
        data: # defining the data key of the ConfiogMap where we will define the key value pair
            # here the key value pair will be valid variable which can be set as env variable in this case
            DATABASE_URI: https://mydatabase.<something>.<something>:3306
            DATABASE_PASSWORD: P@SSWOrd1
            # here defining the key and value as the valid variable which can be used as env variable with value

    ```

- we can define the `workloads.yml` file as below referencing the new name of the `configMap` as below

    ```yaml
        workloads.yml
        =============
        apiVersion: apps/v1 # here the Deployment belongs to the apps group hyence defined as below 
        kind: Deployment # type of the kubernetes object is deployment
        metadata: # name of the Deployment been described here
            name: position-simulator
        spec: # specification for the Deployment been defined here
            selector: # selector to select the POD based on the POD label
                matchlabels:
                    app: position-simulator
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: position-simulator
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                        - name: position-simulator # name of the container
                          image: richardchesterwood/k8s-fleetman-position-simulator:release2 # image for the container
                          env: # env Variable for the container
                            - name: SPRING_PROFILE_ACTIVE # name 0f the env Variable 
                              value: production-microservice # value of the env Variable
                          envFrom: # defining the envFrom in here
                            - configMapRef: # referencing the configMapRef in here
                                name: database-uri-info-v4 # name of the configMapRef to reference all the key and value pair become env variable with the name and value
        
        ---

        apiVersion: apps/v1 # here the Deployment belongs to the apps group hyence defined as below 
        kind: Deployment # type of the kubernetes object is deployment
        metadata:  # name of the Deployment been described here
            name: position-tracker
        spec: # specification for the Deployment been defined here
            selector: # selector to select the POD based on the POD label
                matchlabels:
                    app: position-tracker
            replicas: 1 # replicas of the POD defined as 1 at any instance
            template: # template for the POD container
                metadata: # POD labels been defined inside the metadata section
                    labels:
                        app: position-tracker
                spec: # Specification of the POD container been defined in here
                    containers: # container details provided here
                        - name: position-tracker # name of the container
                          image: richardchesterwood/k8s-fleetman-position-tracker:release3 # image of the container
                          env: # env Variable for the container
                            - name: SPRING_PROFILE_ACTIVE # name 0f the env Variable 
                              value: production-microservice # value of the env Variable
                          envFrom: # defining the envFrom in here
                            - configMapRef:  # referencing the configMapRef in here
                                name: database-uri-info-v4 # name of the configMapRef to reference all the key and value pair become env variable with the name and value

    ```

- now if we deploy the `workloads.yml` by making the change then we can see the `container failing` as the reference `database-uri-info-v4` reference can't be found

- but as soon as we deploy the `database-uri-data.yml` file `database-uri-info-v4` wuill be avialable and `container starts working fine`

- henece there is an `order` for executing the `yml definition` , until the `database-uri-data.yml` been deployed the `POD which using the configMap name database-uri-info-v4` will not run fine , but once deployed then `container will restart and fetch the database-uri-info-v4 configMap`  

- we can do that as below 

    ```bash
        kubectl apply -f workloads.yml
        # applying the changes to the workloads.yml then we can do that as below 
        deployment.apps/queueapp unchanged
        deployment.apps/position-simulator configured
        deployment.apps/position-tracker configured
        deployment.apps/api-gateway unchanged
        deployment.apps/webapp unchanged

        # now if we see the POD we can see the POD getting failed because the reference could not found in this case
        # hence in this case we will begetting the error as below 
        kubectl get all 
        # fetching all the kubernetes workloads inside the default namespace
        NAME                                      READY   STATUS                       RESTARTS         AGE
        pod/api-gateway-56c46fbcdb-mckrd          1/1     Running                      2 (20m ago)      26h
        pod/mongodb-578b98fbd4-6m8h8              1/1     Running                      2 (20m ago)      26h # new PODs been getting crashed
        pod/position-simulator-6896898cb4-wrgcv   0/1     CreateContainerConfigError   0                109s # here the old POD still running
        pod/position-simulator-68b545b847-cqlgz   1/1     Running                      0                14m
        pod/position-tracker-67499f494d-qlvmn     1/1     Running                      0                14m # here the old POD still running
        pod/position-tracker-f874f7467-drrh8      0/1     CreateContainerConfigError   0                109s # new PODs been getting crashed
        pod/queueapp-f55dcb97d-8z6xg              1/1     Running                      2 (20m ago)      26h
        pod/webapp-66765b68df-7298p               0/1     CrashLoopBackOff             11 (3m29s ago)   26h

        NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
        service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   26h
        service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        26h
        service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         26h
        service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   26h
        service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     26h
        service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          27h

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           26h
        deployment.apps/mongodb              1/1     1            1           26h
        deployment.apps/position-simulator   1/1     1            1           26h
        deployment.apps/position-tracker     1/1     1            1           26h
        deployment.apps/queueapp             1/1     1            1           26h
        deployment.apps/webapp               0/1     1            0           26h

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-56c46fbcdb          1         1         1       26h
        replicaset.apps/mongodb-578b98fbd4              1         1         1       26h
        replicaset.apps/position-simulator-5f4dc8c48b   0         0         0       16h
        replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       26h
        replicaset.apps/position-simulator-67d9f49f5b   0         0         0       15h
        replicaset.apps/position-simulator-6896898cb4   1         1         0       109s
        replicaset.apps/position-simulator-68b545b847   1         1         1       14m
        replicaset.apps/position-simulator-84f66bc755   0         0         0       15h
        replicaset.apps/position-simulator-bf87cb7f6    0         0         0       12h
        replicaset.apps/position-tracker-59fdfd8cf4     0         0         0       26h
        replicaset.apps/position-tracker-5dd5688987     0         0         0       12h
        replicaset.apps/position-tracker-67499f494d     1         1         1       14m
        replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       15h
        replicaset.apps/position-tracker-9cbf78bd9      0         0         0       16h
        replicaset.apps/position-tracker-f874f7467      1         1         0       109s
        replicaset.apps/queueapp-f55dcb97d              1         1         1       26h
        replicaset.apps/webapp-66765b68df               1         1         0       26h
 
        # if we check the position-simulator POD detials then we can see as below 
        kubectl describe pod/position-simulator-6896898cb4-wrgcv
        # here see the description of the logs in this case as below 
        Name:             position-simulator-6896898cb4-wrgcv
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Sun, 04 Feb 2024 11:45:18 +0530
        Labels:           app=position-simulator
                        pod-template-hash=6896898cb4
        Annotations:      <none>
        Status:           Pending
        IP:               10.244.0.74
        IPs:
        IP:           10.244.0.74
        Controlled By:  ReplicaSet/position-simulator-6896898cb4
        Containers:
        position-simulator:
            Container ID:   
            Image:          richardchesterwood/k8s-fleetman-position-simulator:release2
            Image ID:       
            Port:           <none>
            Host Port:      <none>
            State:          Waiting
            Reason:       CreateContainerConfigError
            Ready:          False
            Restart Count:  0
            Environment Variables from:
            database-uri-info-v4  ConfigMap  Optional: false
            Environment:
            SPRING_PROFILES_ACTIVE:  production-microservice
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n75ml (ro)
        Conditions:
        Type              Status
        Initialized       True 
        Ready             False 
        ContainersReady   False 
        PodScheduled      True 
        Volumes:
        kube-api-access-n75ml:
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
        Type     Reason     Age                   From               Message
        ----     ------     ----                  ----               -------
        Normal   Scheduled  3m11s                 default-scheduler  Successfully assigned default/position-simulator-6896898cb4-wrgcv to minikube
        Warning  Failed     71s (x12 over 3m11s)  kubelet            Error: configmap "database-uri-info-v4" not found # here we can see the error in this case
        Normal   Pulled     57s (x13 over 3m11s)  kubelet            Container image "richardchesterwood/k8s-fleetman-position-simulator:release2" already present on machine


        # now if we deploy the dat6abase-uri-data.yml npow to the cluster then we can see the container find the database-uri-info-v4 will work fine
        kubectl apply -f database-uri-info.yml
        # here deploying the chnages to the cluster by applying the changes
        configmap/database-uri-info-v4 created


        # now if we see all the kubernetes workloads inside the default namespace as below
        kubectl get all
        # the output will be as below
        # we can see the OLD PODs which are running being terminated as the new POD comes back up
        # the new POD comes backup as database-uri-ionfo-v4 been available 
        NAME                                      READY   STATUS             RESTARTS         AGE
        pod/api-gateway-56c46fbcdb-mckrd          1/1     Running            2 (26m ago)      26h
        pod/mongodb-578b98fbd4-6m8h8              1/1     Running            2 (26m ago)      26h
        pod/position-simulator-6896898cb4-wrgcv   1/1     Running            0                7m1s
        pod/position-tracker-f874f7467-drrh8      1/1     Running            0                7m1s
        pod/queueapp-f55dcb97d-8z6xg              1/1     Running            2 (26m ago)      26h
        pod/webapp-66765b68df-7298p               0/1     CrashLoopBackOff   12 (3m35s ago)   26h

        NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
        service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   26h
        service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        26h
        service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         26h
        service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   26h
        service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     26h
        service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          27h

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           26h
        deployment.apps/mongodb              1/1     1            1           26h
        deployment.apps/position-simulator   1/1     1            1           26h
        deployment.apps/position-tracker     1/1     1            1           26h
        deployment.apps/queueapp             1/1     1            1           26h
        deployment.apps/webapp               0/1     1            0           26h

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-56c46fbcdb          1         1         1       26h
        replicaset.apps/mongodb-578b98fbd4              1         1         1       26h
        replicaset.apps/position-simulator-5f4dc8c48b   0         0         0       16h
        replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       26h
        replicaset.apps/position-simulator-67d9f49f5b   0         0         0       16h
        replicaset.apps/position-simulator-6896898cb4   1         1         1       7m1s
        replicaset.apps/position-simulator-68b545b847   0         0         0       19m
        replicaset.apps/position-simulator-84f66bc755   0         0         0       16h
        replicaset.apps/position-simulator-bf87cb7f6    0         0         0       12h
        replicaset.apps/position-tracker-59fdfd8cf4     0         0         0       26h
        replicaset.apps/position-tracker-5dd5688987     0         0         0       12h
        replicaset.apps/position-tracker-67499f494d     0         0         0       19m
        replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       16h
        replicaset.apps/position-tracker-9cbf78bd9      0         0         0       16h
        replicaset.apps/position-tracker-f874f7467      1         1         1       7m1s
        replicaset.apps/queueapp-f55dcb97d              1         1         1       26h
        replicaset.apps/webapp-66765b68df               1         1         0       26h


        # here we will be now seeing the describe details of the new PODs then we can see the below details 
        kubectl describe pod/position-simulator-6896898cb4-wrgcv
        # then we can see the info as below 
        Name:             position-simulator-6896898cb4-wrgcv
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Sun, 04 Feb 2024 11:45:18 +0530
        Labels:           app=position-simulator
                        pod-template-hash=6896898cb4
        Annotations:      <none>
        Status:           Running # here we can see the status as running as database-uri-info-v2 configMap name being found
        IP:               10.244.0.74
        IPs:
        IP:           10.244.0.74
        Controlled By:  ReplicaSet/position-simulator-6896898cb4
        Containers:
        position-simulator:
            Container ID:   docker://7552c66236da479f5dae220697ec5f834ccda77ed18db5b1532831b5577e8249
            Image:          richardchesterwood/k8s-fleetman-position-simulator:release2
            Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-position-simulator@sha256:0b540a28f5a617b5d5643e82939018414a0af6a7ed0c0f4bb1e50cddb7fd5a8d
            Port:           <none>
            Host Port:      <none>
            State:          Running
            Started:      Sun, 04 Feb 2024 11:51:22 +0530
            Ready:          True
            Restart Count:  0
            Environment Variables from:
            database-uri-info-v4  ConfigMap  Optional: false
            Environment:
            SPRING_PROFILES_ACTIVE:  production-microservice
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n75ml (ro)
        Conditions:
        Type              Status
        Initialized       True 
        Ready             True 
        ContainersReady   True 
        PodScheduled      True 
        Volumes:
        kube-api-access-n75ml:
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
        Type     Reason     Age                     From               Message
        ----     ------     ----                    ----               -------
        Normal   Scheduled  8m58s                   default-scheduler  Successfully assigned default/position-simulator-6896898cb4-wrgcv to minikube
        Warning  Failed     6m57s (x12 over 8m57s)  kubelet            Error: configmap "database-uri-info-v4" not found # older Logs
        Normal   Pulled     3m49s (x27 over 8m57s)  kubelet            Container image "richardchesterwood/k8s-fleetman-position-simulator:release2" already present on machine

    ```