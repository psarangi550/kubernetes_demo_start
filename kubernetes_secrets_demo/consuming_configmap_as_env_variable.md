# consuming configMaps as Environment variable 

- creating  the `configMaps` is `trivial and simple` , but interesting  thing here actually `how to use the configMaps ?`

- here we will show the `how to read in the database URL` that we set as the `configMaps` , in the `workloads.yml` file in `one or more POD container`

- we need to `feed` the values of the `configMaps` that we have created into the `POD container` at `runtime` , here we are considering the `position-simulator POD container`

- there are actually `3 ways` that we can do that 

  - <ins>1st approach or worst approach OR hardest approach</ins>
    
    - we can `change the values inside the ConfigMaps , into regular environment variables` then the `code inside the image` will work with the `image` as `environment variable`
    
    - here we can provide the `env variable` with the `named` as `DATABASE_URI` and instead of using `value` we can use the `valueFrom`
    
    - here we are telling `kubernetes` that we will not be `hardcoding the value` but rather we want `kubernetes to look for the value` , when we `apply changes` using the `yaml`
    
    - then we can point `kubernetes` to get the `values` from the `valueFrom` key inside it mentioning as `configMapKeyRef` and inside that we can mention the `name of the configMap we will be reading from i.e defined inside the configMap yml definition file`
    
    - along with the `name of the configMap` we also have to define the `key of the configMap` from where we want to `read the values`
    
    - we can define `workloads.yml` it as below for the `one of the POD` such as `position-simulator` POD  as below 

        
        ```yaml
            workloads.yml
            =============
            apiVersion: apps/v1 # defining the apiVersion as apps/v1 as the Deployment falls under the apps group
            kind: Deployment # here defining the kind of kubernetes object as Deployment
            metadata: # defining the name of the >Deployment as position-simulator
                name: position-simulator
            spec: #specification for the position-simulator deployment
                selector: # here the selector for the Deployment which will take the POD based on the POD label
                    matchLabels:
                        app: position-simulator
                replicas: 1 # defining the replicas as 1 in this case
                template: # definign the POD container template in this case 
                    metadata: # describing the POD labels so that Deployument can select the same
                        labels:
                            app: position-simulator
                    spec: # definign the specification of the POD container here
                        containers: # here we will mention the position-simulator
                            - name: position-simulator #name of container
                              image: richardchesterwood/k8s-fleetman-position-simulator:release2 # image for the container
                              env: # defining the environement variable for the POD container
                                - name: SPRING_PROFILE_ACTIVE # name of the environment variable
                                  value: production-microservice # value of the environment variable
                                - name: DATABASE_URI # name of the environment variable
                                  valueFrom: # here we are instructuing kubernetes to find the value Dynamically on runtime
                                    configMapKeyRef: # here we are providing the configMapKeyRef in thiss case
                                        name: database-uri-info # here we will be using the name as database-uri-info which is for the configMap name 
                                        key: database.url # here we are using the key defined inside the configMap those value we want to use as values


        ```  
  
  
  - if we have `multiple env variable` then we need to define the `name and valueFrom` for those  as well as below 
  
  - we can define that as below 

    ```yaml
        workloads.yml
        =============
        apiVersion: apps/v1 # defining the apiVersion as apps/v1 as the Deployment falls under the apps group
        kind: Deployment # here defining the kind of kubernetes object as Deployment
        metadata: # defining the name of the >Deployment as position-simulator
            name: position-simulator
        spec: #specification for the position-simulator deployment
            selector: # here the selector for the Deployment which will take the POD based on the POD label
                matchLabels:
                    app: position-simulator
            replicas: 1 # defining the replicas as 1 in this case
            template: # definign the POD container template in this case 
                metadata: # describing the POD labels so that Deployument can select the same
                    labels:
                        app: position-simulator
                spec: # definign the specification of the POD container here
                    containers: # here we will mention the position-simulator
                        - name: position-simulator #name of container
                            image: richardchesterwood/k8s-fleetman-position-simulator:release2 # image for the container
                            env: # defining the environement variable for the POD container
                            
                            - name: SPRING_PROFILE_ACTIVE # name of the environment variable
                                value: production-microservice # value of the environment variable
                            
                            - name: DATABASE_URI # name of the environment variable
                                valueFrom: # here we are instructuing kubernetes to find the value Dynamically on runtime
                                configMapKeyRef: # here we are providing the configMapKeyRef in thiss case
                                    name: database-uri-info # here we will be using the name as database-uri-info which is for the configMap name 
                                    key: database.url # here we are using the key defined inside the configMap those value we want to use as values
                            
                            - name: DATABASE_PASSWORD # name of the environment variable
                                valueFrom:  # here we are instructuing kubernetes to find the value Dynamically on runtime
                                configMapKeyRef: # here we are providing the configMapKeyRef in thiss case
                                    name: database-uri-info # here we will be using the name as database-uri-info which is for the configMap name 
                                    key: database.password # here we are using the key defined inside the configMap those value we want to use as values



    ```
  
  - if we want these inside `multiple PODs` then we can use it as below , here we can use the `position-tracker` POD container as well then we can define as below

    
    ```yaml
        workloads.yml
        =============
        apiVersion: apps/v1 # defining the apiVersion as apps/v1 as the Deployment falls under the apps group
        kind: Deployment # here defining the kind of kubernetes object as Deployment
        metadata: # defining the name of the >Deployment as position-simulator
            name: position-simulator
        spec: #specification for the position-simulator deployment
            selector: # here the selector for the Deployment which will take the POD based on the POD label
                matchLabels:
                    app: position-simulator
            replicas: 1 # defining the replicas as 1 in this case
            template: # definign the POD container template in this case 
                metadata: # describing the POD labels so that Deployument can select the same
                    labels:
                        app: position-simulator
                spec: # definign the specification of the POD container here
                    containers: # here we will mention the position-simulator
                        - name: position-simulator #name of container
                            image: richardchesterwood/k8s-fleetman-position-simulator:release2 # image for the container
                            env: # defining the environement variable for the POD container
                            
                            - name: SPRING_PROFILE_ACTIVE # name of the environment variable
                                value: production-microservice # value of the environment variable
                            
                            - name: DATABASE_URI # name of the environment variable
                                valueFrom: # here we are instructuing kubernetes to find the value Dynamically on runtime
                                configMapKeyRef: # here we are providing the configMapKeyRef in thiss case
                                    name: database-uri-info # here we will be using the name as database-uri-info which is for the configMap name 
                                    key: database.url # here we are using the key defined inside the configMap those value we want to use as values
                            
                            - name: DATABASE_PASSWORD # name of the environment variable
                                valueFrom:  # here we are instructuing kubernetes to find the value Dynamically on runtime
                                configMapKeyRef: # here we are providing the configMapKeyRef in thiss case
                                    name: database-uri-info # here we will be using the name as database-uri-info which is for the configMap name 
                                    key: database.password # here we are using the key defined inside the configMap those value we want to use as values


            ---

        
        apiVersion: apps/v1 # defining the apiVersion as apps/v1 as the Deployment falls under the apps group
        kind: Deployment # here defining the kind of kubernetes object as Deployment
        metadata: # defining the name of the >Deployment as position-tracker
            name: position-tracker
        spec: #specification for the position-simulator deployment
            selector: # here the selector for the Deployment which will take the POD based on the POD label
                matchLabels:
                    app: position-tracker
            replicas: 1 # defining the replicas as 1 in this case
            template: # definign the POD container template in this case 
                metadata: # describing the POD labels so that Deployument can select the same
                    labels:
                        app: position-tracker
                spec: # definign the specification of the POD container here
                    containers: # here we will mention the position-simulator
                        - name: position-tracker #name of container
                            image: richardchesterwood/k8s-fleetman-position-tracker:release3 # image for the container
                            env: # defining the environement variable for the POD container
                            
                            - name: SPRING_PROFILE_ACTIVE # name of the environment variable
                                value: production-microservice # value of the environment variable
                            
                            - name: DATABASE_URI # name of the environment variable
                                valueFrom: # here we are instructuing kubernetes to find the value Dynamically on runtime
                                configMapKeyRef: # here we are providing the configMapKeyRef in thiss case
                                    name: database-uri-info # here we will be using the name as database-uri-info which is for the configMap name 
                                    key: database.url # here we are using the key defined inside the configMap those value we want to use as values
                            
                            - name: DATABASE_PASSWORD # name of the environment variable
                                valueFrom:  # here we are instructuing kubernetes to find the value Dynamically on runtime
                                configMapKeyRef: # here we are providing the configMapKeyRef in thiss case
                                    name: database-uri-info # here we will be using the name as database-uri-info which is for the configMap name 
                                    key: database.password # here we are using the key defined inside the configMap those value we want to use as values


    ```

    - here even though we have to `copy` and `pasted` , but here we have `externalised the env variable value` which can be managed from the `configMaps`
    
    - if we change the value inside the `configMaps` that will be `dynamically reflected` in the `workload env variable value` at the `runtime` for all the `POD container`
    
    - as the `configMaps` values will be `stored into kubernetes` and `when we are deploying the changes of the workload.yml by appling the changes` those value will be reflect in `workloads` as well
    
    - as the `developer of the image` we need to understand that `whatever the env variable` we are providing the `image while ontime become container` can read those `env variable` , as the `Deployer` we don't have to bother about `how that been implemented inside the image`
    
    - but we can go to the `POD container` have a look over the `environment variable` that we have set in here
    
    - now we can just change the value of the `key and value ` inside the `configMap` file as below 
  
        ```yaml

            database-uri-data.yml
            =====================
            apiVersion: v1 # here we are using the apiVersion as v1 as it belong to the core group
            kind: ConfigMap # here the kubernetes object is of type ConfigMap
            metadata: # defining the name for the ConfigMap which will be used inside the configMapKeyRef name in workloads.yml
                name: database-uri-info
            data: # heredefining the data key where we will be saving the key and value pair
                database.url: https://mysqlserver.<something>.<something>:3306 # key and value for database.url and value
                database.password: P@SSW0RD1 # key and value for database.password and value

        ``` 

    - we can `deploy the changes` onto the `cluster` by `applying the changes` as below 
    
        ```bash
            kubectl apply -f database-uri-data.yml
            # here `deploy the changes` onto the `cluster` by `applying the changes`
            # we can see the below output in this case 
            configmap/database-uri-info configured

        ```  
    
    - we can `deploy the changes of the wokloads.yml` onto the `cluster` by `applying the changes` as below 

        ```bash
            kubectl apply -f workloads.yml
            # here `deploy the changes` onto the `cluster` by `applying the changes`
            # we will be getting the response as below
            deployment.apps/queueapp unchanged
            deployment.apps/position-simulator configured
            deployment.apps/position-tracker configured
            deployment.apps/api-gateway unchanged
            deployment.apps/webapp unchanged

        ```
    

    - now if we goto `one of the position-tracker/position-simulator` POD container where we used the `configMap value as env variable` then we can see those updated value 

        ```bash
            kubectl get all
            # here we are seeing the all the kubernetes object inside the default namespace
            # the output will be as below
            NAME                                      READY   STATUS    RESTARTS      AGE
            pod/api-gateway-56c46fbcdb-mckrd          1/1     Running   1 (90m ago)   10h
            pod/mongodb-578b98fbd4-6m8h8              1/1     Running   1 (90m ago)   10h
            pod/position-simulator-67d9f49f5b-2gkmm   1/1     Running   0             6s
            pod/position-tracker-9cbf78bd9-29h5t      1/1     Running   0             6s
            pod/queueapp-f55dcb97d-8z6xg              1/1     Running   1 (90m ago)   10h
            pod/webapp-66765b68df-7298p               1/1     Running   3 (19m ago)   10h

            NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
            service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   10h
            service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        10h
            service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         10h
            service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   10h
            service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     10h
            service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          11h

            NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
            deployment.apps/api-gateway          1/1     1            1           10h
            deployment.apps/mongodb              1/1     1            1           10h
            deployment.apps/position-simulator   1/1     1            1           10h
            deployment.apps/position-tracker     1/1     1            1           10h
            deployment.apps/queueapp             1/1     1            1           10h
            deployment.apps/webapp               1/1     1            1           10h

            NAME                                            DESIRED   CURRENT   READY   AGE
            replicaset.apps/api-gateway-56c46fbcdb          1         1         1       10h
            replicaset.apps/mongodb-578b98fbd4              1         1         1       10h
            replicaset.apps/position-simulator-5f4dc8c48b   0         0         0       19m
            replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       10h
            replicaset.apps/position-simulator-67d9f49f5b   1         1         1       6s
            replicaset.apps/position-simulator-84f66bc755   0         0         0       42s
            replicaset.apps/position-tracker-59fdfd8cf4     0         0         0       10h
            replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       42s
            replicaset.apps/position-tracker-9cbf78bd9      1         1         1       19m
            replicaset.apps/queueapp-f55dcb97d              1         1         1       10h
            replicaset.apps/webapp-66765b68df               1         1         1       10h
            
            # here we can check for one of the POD i.e position-simulator or position-tracker where we use the configMap value as env variable
            # here i am using the position-simulator as below
            kubectl exec -it pod/position-simulator-5f4dc8c48b-6jj2t -- bash
            # using the kubectl exec command accesssing position-simulator terminal access
            # here the outcome is as below with new terminal
            root@position-simulator-67d9f49f5b-2gkmm:/# echo $DATABASE_URI 3 here we are echoing the result in here
            # the outcome in here as below
            https://mysqlserver.<something>.<something>:3306

            #similarly if we check for the DATABASE_PASSWORD then we will get as below 
            root@position-simulator-67d9f49f5b-2gkmm:/# echo $DATABASE_PASSWORD
            # the output in here will be as below
            P@SSW0RD1
            
        ```

- the `configMap` allow us to `specify variable` which can be `deployed to multiple PODs`
