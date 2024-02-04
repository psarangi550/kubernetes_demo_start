# Do changes made in configMaps propagate in kubernetes or Not

- what happen the if we `change the value` inside the `configMap` file 

- as of `9th of May 2019` , if we change the `value of the configMap` those changes will not be `propagated to the POD container` using the `value of the configMap as env variable`

- we relaised the `database config` that we stored inside the `configMap` having name as `database-uri-data.yml` been changed 

- we can rewrite the `database-uri-data.yml` as below 

    ```yaml
        database-uri-data.yml
        ======================
        apiVersion: v1 # defining the apiVersion as v1 as it belong to the core group in here
        kind: ConfigMap # here the type of kubernetes object in thsi case as ConfigMap
        metadata: # name of the configMaps are defined in here
            name: database-uri-info
        data: # defining the data section where we will define the key value pair
            database.url: https://changed.<something>.<something>:3306
            database.password: P@SSWOrd1
            # here we are chaning/updating the value of the configMaps

    ```

- we can `deploy the changes` onto the `cluster` by `applying the changes` as below 
    
    ```bash
        kubectl apply -f database-uri-data.yml
        # here `deploy the changes` onto the `cluster` by `applying the changes`
        # we can see the below output in this case 
        configmap/database-uri-info configured
    ```

- we can then see the details of the `configMap` using the  command as below 

    ```bash
        kubectl get cm
        # fetching the configMaps present in the default namespace in here
        # the output will be as below
        NAME                DATA   AGE
        database-uri-info   2      5h57m

        # now we can see the details about the configMap using the sdescribe command as below 
        # hence here we can perform the action such as 
        kubectl describe cm database-uri-info
        # here it will display the info about the configMaps
        Name:         database-uri-info
        Namespace:    default
        Labels:       <none>
        Annotations:  <none>

        Data
        ====
        database.password:
        ----
        P@SSW0RD
        database.url:
        ----
        https://changed.<something>.<something>:3306 # here we can see that the value been changed in the configMap 

        BinaryData
        ====

        Events:  <none>

        
    ```

- but at the `current time` , those `change of values of the configMap` been picked up by the `existing POD container` which been `alread read from the configMap file`

- if we are going to the `exec` command on the `existing POD` then we can see the below detials 

    
    ```bash
        kubectl get all
        # fetching all the kubernetes object in the default namespace
        # the output will be as below 
        NAME                                      READY   STATUS    RESTARTS        AGE
        pod/api-gateway-56c46fbcdb-mckrd          1/1     Running   1 (3h48m ago)   13h
        pod/mongodb-578b98fbd4-6m8h8              1/1     Running   1 (3h48m ago)   13h
        pod/position-simulator-5f4dc8c48b-ll66m   1/1     Running   0               134m
        pod/position-tracker-9cbf78bd9-29h5t      1/1     Running   0               138m
        pod/queueapp-f55dcb97d-8z6xg              1/1     Running   1 (3h48m ago)   13h
        pod/webapp-66765b68df-7298p               1/1     Running   3 (158m ago)    13h

        NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
        service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   13h
        service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        13h
        service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         13h
        service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   13h
        service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     13h
        service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          13h

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           13h
        deployment.apps/mongodb              1/1     1            1           13h
        deployment.apps/position-simulator   1/1     1            1           13h
        deployment.apps/position-tracker     1/1     1            1           13h
        deployment.apps/queueapp             1/1     1            1           13h
        deployment.apps/webapp               1/1     1            1           13h

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-56c46fbcdb          1         1         1       13h
        replicaset.apps/mongodb-578b98fbd4              1         1         1       13h
        replicaset.apps/position-simulator-5f4dc8c48b   1         1         1       158m
        replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       13h
        replicaset.apps/position-simulator-67d9f49f5b   0         0         0       138m
        replicaset.apps/position-simulator-84f66bc755   0         0         0       139m
        replicaset.apps/position-tracker-59fdfd8cf4     0         0         0       13h
        replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       139m
        replicaset.apps/position-tracker-9cbf78bd9      1         1         1       158m
        replicaset.apps/queueapp-f55dcb97d              1         1         1       13h
        replicaset.apps/webapp-66765b68df               1         1         1       13h


        # now as the position-simulator and position-tracker POD container been using it hence we can exec into the POD container
        # we can do that using the exec command as below 
        kubectl exec -it pod/position-simulator-5f4dc8c48b-ll66m -- bash
        # going to the terminal of the position-simulator POD container
        root@position-simulator-5f4dc8c48b-ll66m:/# echo $DATABASE_URI
        # here we are checking the DATABASE_URI env variable which been pointing to the old configMap values
        # here its still be pointing to the OLD value of the configMaps
        # the updated info not showing 
        https://mysqlserver.<something>.<something>:3306



    ```

- currently there is `no mechanism` exists in `kubernetes` to `automatically pick up the updated value of the configMap` in the `existing POD which been using those configMap values as the env variable`

- we can make the `updated values of the configMaps config inside the existing POD container using it` by `bouncing the existing POD container` i.e `killing the POD container` , as its a part of the `Deployment` , hence the `POD container` will be `recreated` , on the newly created `POD` we can see the `updated configMap Values`

- we can do that by using the command as below 

    ```bash
        kubectl get all
        # fetching all the kubernetes workload in the default namespace
        # the output will be as below 
        NAME                                      READY   STATUS    RESTARTS        AGE
        pod/api-gateway-56c46fbcdb-mckrd          1/1     Running   1 (4h30m ago)   13h
        pod/mongodb-578b98fbd4-6m8h8              1/1     Running   1 (4h30m ago)   13h
        pod/position-simulator-5f4dc8c48b-ll66m   1/1     Running   0               176m
        pod/position-tracker-9cbf78bd9-29h5t      1/1     Running   0               3h
        pod/queueapp-f55dcb97d-8z6xg              1/1     Running   1 (4h30m ago)   13h
        pod/webapp-66765b68df-7298p               1/1     Running   3 (3h19m ago)   13h

        NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
        service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   13h
        service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        13h
        service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         13h
        service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   13h
        service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     13h
        service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          14h

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           13h
        deployment.apps/mongodb              1/1     1            1           13h
        deployment.apps/position-simulator   1/1     1            1           13h
        deployment.apps/position-tracker     1/1     1            1           13h
        deployment.apps/queueapp             1/1     1            1           13h
        deployment.apps/webapp               1/1     1            1           13h

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-56c46fbcdb          1         1         1       13h
        replicaset.apps/mongodb-578b98fbd4              1         1         1       13h
        replicaset.apps/position-simulator-5f4dc8c48b   1         1         1       3h19m
        replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       13h
        replicaset.apps/position-simulator-67d9f49f5b   0         0         0       3h
        replicaset.apps/position-simulator-84f66bc755   0         0         0       3h
        replicaset.apps/position-tracker-59fdfd8cf4     0         0         0       13h
        replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       3h
        replicaset.apps/position-tracker-9cbf78bd9      1         1         1       3h19m
        replicaset.apps/queueapp-f55dcb97d              1         1         1       13h
        replicaset.apps/webapp-66765b68df               1         1         1       13h

        # here we can see the info about the position-simulator and position-tracker POD , which we can try to bounce back i.e kill and as its a Deployment a New POD will be spin on its place
        kubectl delete pod position-simulator-5f4dc8c48b-ll66m position-tracker-9cbf78bd9-29h5t
        # here we are removing the position-simulator and position-tracker POD in this case
        # the output will be as below for the file
        pod "position-simulator-5f4dc8c48b-ll66m" deleted
        pod "position-tracker-9cbf78bd9-29h5t" deleted



        # now as its a Deployment with replicas as 1 , hence New POD will be spunned for the same
        kubectl get all
        # fetching all the kubernetes workload in the default namespace
        # the output will be as below 
        NAME                                      READY   STATUS    RESTARTS        AGE
        pod/api-gateway-56c46fbcdb-mckrd          1/1     Running   1 (4h34m ago)   13h
        pod/mongodb-578b98fbd4-6m8h8              1/1     Running   1 (4h34m ago)   13h
        pod/position-simulator-5f4dc8c48b-nsj7b   1/1     Running   0               67s
        pod/position-tracker-9cbf78bd9-dx9bk      1/1     Running   0               67s
        pod/queueapp-f55dcb97d-8z6xg              1/1     Running   1 (4h34m ago)   13h
        pod/webapp-66765b68df-7298p               1/1     Running   3 (3h23m ago)   13h

        NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
        service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   13h
        service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        13h
        service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         13h
        service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   13h
        service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     13h
        service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          14h

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           13h
        deployment.apps/mongodb              1/1     1            1           13h
        deployment.apps/position-simulator   1/1     1            1           13h
            1/1     1            1           13h
        deployment.apps/queueapp             1/1     1            1           13h
        deployment.apps/webapp               1/1     1            1           13h

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-56c46fbcdb          1         1         1       13h
        replicaset.apps/mongodb-578b98fbd4              1         1         1       13h
        replicaset.apps/position-simulator-5f4dc8c48b   1         1         1       3h23m
        replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       13h
        replicaset.apps/position-simulator-67d9f49f5b   0         0         0       3h4m
        replicaset.apps/position-simulator-84f66bc755   0         0         0       3h4m
        replicaset.apps/position-tracker-59fdfd8cf4     0         0         0       13h
        replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       3h4m
        replicaset.apps/position-tracker-9cbf78bd9      1         1         1       3h23m
        replicaset.apps/queueapp-f55dcb97d              1         1         1       13h
        replicaset.apps/webapp-66765b68df               1         1         1       13h


        # now if we access the position-simulator or position-tracker POD then we can see the changes as below 
        kubectl exec -it pod/position-simulator-5f4dc8c48b-nsj7b -- bash
        # accessing the terminal to access the position-simulator POD container terminal
        root@position-simulator-5f4dc8c48b-nsj7b:/# echo $DATABASE_URI
        # here we can see that we are using the DATABASE_URI which we can see as changed as per the values updated in configMap
        https://changed.<something>.<something>:3306

    ```

- this is `one of the most frequently requested features in kubernetes` 

- there is a `github` issue reference as `22368` having link as [Kubernetes GitHub Issue tracker](https://github.com/kubernetes/kubernetes/issues/22368) which is `currently open`

- if we are a `JAVA spring developer` there is a `solution available for this`

- the `most common way of handling this` is by making the `configMap` as `immutable in nature`

- **How many Project handle the configMap value update**

  - if we want to update the `database url` to `http://mydatabase.<something>.<something>:3306` then 
  
  - here `instead of hacking in` and changing/updating the `value of the configMap` `most of the project abandon the existing configMap name`  

  - we need to define the `new version name for the configMap` and update the `reference of the configMap name` inside the `workloads POD container` which been using it 
  
  -  we can do that using it as below `database-uri-data.yml` and `workloads.yml` as below 
    
    
    
    ```yaml
        database-uri-data.yml
        ======================
        apiVersion: v1 # defining the apiVersion as v1 as it belong to the core group in here
        kind: ConfigMap # here the type of kubernetes object in thsi case as ConfigMap
        metadata: # name of the configMaps are defined in here
            name: database-uri-info-v2 # updating the name of the configMap in here
        data: # defining the data section where we will define the key value pair
            database.url: https://mydatabase.<something>.<something>:3306
            database.password: P@SSWOrd1
            # here we are chaning/updating the value of the configMaps

    ```

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
                                    name: database-uri-info-v2 # here we will be using the name as database-uri-info-v2 which is for the configMap name 
                                    key: database.url # here we are using the key defined inside the configMap those value we want to use as values
                            
                            - name: DATABASE_PASSWORD # name of the environment variable
                                valueFrom:  # here we are instructuing kubernetes to find the value Dynamically on runtime
                                configMapKeyRef: # here we are providing the configMapKeyRef in thiss case
                                    name: database-uri-info-v2 # here we will be using the name as database-uri-info-v2 which is for the configMap name 
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
                                    name: database-uri-info-v2 # here we will be using the name as database-uri-info-v2 which is for the configMap name 
                                    key: database.url # here we are using the key defined inside the configMap those value we want to use as values
                            
                            - name: DATABASE_PASSWORD # name of the environment variable
                                valueFrom:  # here we are instructuing kubernetes to find the value Dynamically on runtime
                                configMapKeyRef: # here we are providing the configMapKeyRef in thiss case
                                    name: database-uri-info-v2 # here we will be using the name as database-uri-info-v2 which is for the configMap name 
                                    key: database.password # here we are using the key defined inside the configMap those value we want to use as values


    ```

- we can `deploy the changes` onto the `cluster` by `applying the changes` as below 
    
    ```bash
        kubectl apply -f .
        # here `deploy the changes` onto the `cluster` by `applying the changes`
        # we can see the below output in this case 
        configmap/database-uri-info-v2 created
        deployment.apps/queueapp unchanged
        deployment.apps/position-simulator configured
        deployment.apps/position-tracker configured
        deployment.apps/api-gateway unchanged
        deployment.apps/webapp unchanged

    ```


- now if we goto the `position-tracker or position-simulator POD container` then we can see the below info 

    ```bash
        kubectl get all
        # fetching all kubernetes workloads inside the default namespace here
        NAME                                     READY   STATUS    RESTARTS        AGE
        pod/api-gateway-56c46fbcdb-mckrd         1/1     Running   1 (4h53m ago)   14h
        pod/mongodb-578b98fbd4-6m8h8             1/1     Running   1 (4h54m ago)   14h
        pod/position-simulator-bf87cb7f6-wtmhg   1/1     Running   0               2m8s
        pod/position-tracker-5dd5688987-whmff    1/1     Running   0               2m8s
        pod/queueapp-f55dcb97d-8z6xg             1/1     Running   1 (4h53m ago)   14h
        pod/webapp-66765b68df-7298p              1/1     Running   3 (3h43m ago)   14h

        NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
        service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   14h
        service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        14h
        service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         14h
        service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   14h
        service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     14h
        service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          14h

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           14h
        deployment.apps/mongodb              1/1     1            1           14h
        deployment.apps/position-simulator   1/1     1            1           14h
        deployment.apps/position-tracker     1/1     1            1           14h
        deployment.apps/queueapp             1/1     1            1           14h
        deployment.apps/webapp               1/1     1            1           14h

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-56c46fbcdb          1         1         1       14h
        replicaset.apps/mongodb-578b98fbd4              1         1         1       14h
        replicaset.apps/position-simulator-5f4dc8c48b   0         0         0       3h43m
        replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       14h
        replicaset.apps/position-simulator-67d9f49f5b   0         0         0       3h23m
        replicaset.apps/position-simulator-84f66bc755   0         0         0       3h24m
        replicaset.apps/position-simulator-bf87cb7f6    1         1         1       2m8s
        replicaset.apps/position-tracker-59fdfd8cf4     0         0         0       14h
        replicaset.apps/position-tracker-5dd5688987     1         1         1       2m8s
        replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       3h24m
        replicaset.apps/position-tracker-9cbf78bd9      0         0         0       3h43m
        replicaset.apps/queueapp-f55dcb97d              1         1         1       14h
        replicaset.apps/webapp-66765b68df               1         1         1       14h

        # if we now access the configMaps to see the updated changes we can see those changes
        kubectl get cm 
        # fetching all the configMaps inside the defaulot namespace
        # here the output will be as below 
        NAME                   DATA   AGE
        database-uri-info      2      7h10m
        database-uri-info-v2   2      3m7s

        # if we want to see the details of the new configMap then we can see that as below 
        kubectl describe cm database-uri-info-v2
        # fetching the info about the configMaps in this case here
        Name:         database-uri-info-v2
        Namespace:    default
        Labels:       <none>
        Annotations:  <none>

        Data
        ====
        database.password:
        ----
        P@SSW0RD
        database.url:
        ----
        https://mydatabase.<something>.<something>:3306  # here we can see the updated change

        BinaryData
        ====

        Events:  <none>

        # now if we access the position-simulator or position-tracker POD container then we can see the below details as well
        kubectl exec -it pod/position-simulator-bf87cb7f6-wtmhg -- bash
        # here we can see the info as below in this case
        # the output will be as below 
        root@position-simulator-bf87cb7f6-wtmhg:/# echo $DATABASE_URI
        # while echoign we can see the updated result in this case
        https://mydatabase.<something>.<something>:3306

    ```

- the `most common way of handling this` is by making the `configMap` as `immutable in nature`

- if we want to change the value of the `configMap` then we can change the `name and value of the configMap` to `newvalue`  and also update the corresponding `references`
  