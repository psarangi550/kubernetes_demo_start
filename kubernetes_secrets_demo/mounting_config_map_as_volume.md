# Mounting ConfigMaps as Volumes

- using the `environement variables` is `one way of injecting the variables data` to the `pre-defined images` , which will `inject the values` from the `configMap` into the `POD container` when the `image been turned to the container`

- but there is another way `which been conviently working`

- lets suppose the `application image` we are working on `does not support environment variable`

- we can `inject the content of the configMap into the image` in the `form of a file`  which been `mounted to a directory` inside the `POD container`

- we can do that using the `volumeMounts` for the `workloads.yml` file as below , where we will be pointing to the `configMaps`

- the `path` we have declared as the `mountPath` inside the `volumeMounts` , if not present then `that directory mentioned against the mountPath will automatically created` 

- here inside the `volumes` section where we have defined the `hostPath` or `amazonEBSStorage` we can define the `configMap`

- inside the `configMap` we can define the `name of the configMap` that we have used  

- here we will be working with the `workloads.yml` for the `Position Simulator` POD container 
    
    
    ```yaml
        workloads.yaml
        ==============
        apiVersion: apps/v1 # here defining the apiVersion as apps/v1 as the Deployment belong to the apps group
        kind: Deployment # here the kubernetes object is of type Deployment
        metadata: # defining the name for the Deployment in here
            name: position-simulator
        spec: # here the specification with respect to the Deployment been described here
            selector: # selector for the POD label so that Deployment can fetch the corresponding POD based on POD label
                matchlabels:
                    app: position-simulator
            replicas: 1 # replicas for the POD being as 1 here
            template: #template for the POD definition
                metadata: # POD labels been described inside the metadata section
                    labels:
                        app: position-simulator
                spec: # specification for the POD container been defined here 
                    containers: # container details been defined here
                        - name: position-simulator # defining the position simulator as container name in here
                          image: richardchesterwood/k8s-fleetman-position-simulator:release2 # image for the container
                          env: # envVariable for the POD container
                            - name: SPRING_PROFILE_ACTIVE # name fo the env variable
                              value: position-tracker # value of the env Variable
                          volumeMounts: # defining the volumenMount which we create a directory if not exist inside the POD container 
                            - name: database-config-volume # name of the volume mount defined here which will be referenced inside volumes section
                              mountPath: /etc/mydata # defining the mountPath over here , apth inside the container will be created if not exist
                    
                    volumes: # defining the volume which need to be associated with the volumeMounts
                        name: database-config-volume # referencing the volumeMounts name in here
                        configMap: # here defining the configName in here
                            name: database-uri-info-v5 # referencing name of the configMap that we will define inside the database-uri-data.yml as configMap

    ```

- here we can use the `configMap` configuration as below having name as `database-uri-data.yml`

- which can be defined as below 

    ```yaml
        database-uri-data.yml
        =====================
        apiVersion: v1 # here using the apiVersion as v1 in this case as it belong to the core-group
        kind: ConfigMap # defining the kind of Kubernetes object as ConfigMap
        # here sticking to the old approaches
        # as we are changing the configMaps we need to provide a New Version for the same
        # hence we can define that as below 
        metadata: # defining the name of the configMaps in here
            name: atabase-uri-info-v5 
        data: # here defining the data Key which will be used for the storing the key value pair
            DATABASE_URL : http://dbserver.<something>.<something>:3306 # first key-value pair
            DATABASE_PASSWORD: P@SSW0rd1 # 2nd key-value pair


    ```

- if we `deploy` both the `workloads.yml` and `database-uri-data.yml` then we can see that `the key of the configMap data` will be `saved as file` on the `mounthPath location` as `files` containing the `values of the configMap` as the `content of those key which is stored as files`

- now if we `deploy both the changes` into the `kubernetes cluster` by `applying the changes` then we can see the below result in that case

- here we can see that result as below 

    ```bash
        kubectl apply -f .
        # applying the changes onto both the `workloads.yml` and `database-uri-data.yml`file to deploy it onto the kubernetes cluster
        configmap/database-uri-info-v5 created
        deployment.apps/queueapp unchanged
        deployment.apps/position-simulator configured
        deployment.apps/position-tracker unchanged
        deployment.apps/api-gateway unchanged
        deployment.apps/webapp unchanged

        # now we can see all the kubernetes object inside the default namespace 
        # if we want to see position-tracker then we cansee the info as below 
        kubectl get all
        # fetching all the kubernetes object inside the default namespace in here
        NAME                                      READY   STATUS    RESTARTS        AGE
        pod/api-gateway-56c46fbcdb-mckrd          1/1     Running   2 (4h15m ago)   30h
        pod/mongodb-578b98fbd4-6m8h8              1/1     Running   2 (4h15m ago)   30h
        pod/position-simulator-55cd9487fb-8rwl5   1/1     Running   0               5s
        pod/position-tracker-59fdfd8cf4-xvc9s     1/1     Running   0               171m
        pod/queueapp-f55dcb97d-8z6xg              1/1     Running   2 (4h15m ago)   30h
        pod/webapp-66765b68df-hcb7r               1/1     Running   0               28m

        NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
        service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   30h
        service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        30h
        service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         30h
        service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   30h
        service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     30h
        service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          31h

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           30h
        deployment.apps/mongodb              1/1     1            1           30h
        deployment.apps/position-simulator   1/1     1            1           30h
        deployment.apps/position-tracker     1/1     1            1           30h
        deployment.apps/queueapp             1/1     1            1           30h
        deployment.apps/webapp               1/1     1            1           30h

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-56c46fbcdb          1         1         1       30h
        replicaset.apps/mongodb-578b98fbd4              1         1         1       30h
        replicaset.apps/position-simulator-554dc5f486   0         0         0       171m
        replicaset.apps/position-simulator-55cd9487fb   1         1         1       5s
        replicaset.apps/position-simulator-5f4dc8c48b   0         0         0       20h
        replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       30h
        replicaset.apps/position-simulator-67d9f49f5b   0         0         0       19h
        replicaset.apps/position-simulator-6896898cb4   0         0         0       3h56m
        replicaset.apps/position-simulator-68b545b847   0         0         0       4h8m
        replicaset.apps/position-simulator-6db984b5f7   0         0         0       4m53s
        replicaset.apps/position-simulator-84f66bc755   0         0         0       19h
        replicaset.apps/position-simulator-bf87cb7f6    0         0         0       16h
        replicaset.apps/position-tracker-59fdfd8cf4     1         1         1       30h
        replicaset.apps/position-tracker-5dd5688987     0         0         0       16h
        replicaset.apps/position-tracker-67499f494d     0         0         0       4h8m
        replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       19h
        replicaset.apps/position-tracker-9cbf78bd9      0         0         0       20h
        replicaset.apps/position-tracker-f874f7467      0         0         0       3h56m
        replicaset.apps/queueapp-f55dcb97d              1         1         1       30h
        replicaset.apps/webapp-66765b68df               1         1         1       30h


        # now if we access the terminal of the position-simulator POD and accessing the terminal to see the file getting created or not 
        kubectl exec -it pod/position-simulator-55cd9487fb-8rwl5 -- bash
        # here defining the checking the terminal of the position-tracker
        root@position-simulator-55cd9487fb-8rwl5:/# cd /etc/mydata/
        # going to the specific folder
        root@position-simulator-55cd9487fb-8rwl5:/etc/mydata# ls
        # checking the content of the folder here
        DATABASE_PASSWORD  DATABASE_URI # here we can find the keys of the configMaps as file here containing the value of the corresponding configMap
        root@position-simulator-55cd9487fb-8rwl5:/etc/mydata# cat DATABASE_URI
        # THE OUTPUT WILL BE AS BELOW 
        https://mydatabase.<something>.<something>:3306


    ```

- hence the `code running inside the image` can r`ead this file` and `perform action on them` , this is another way to `manipulate the data inside the image dynamically`

- here we ended up getting `2 files` as there are `2 keys defined inside the configMaps with corresponding value`

- rather here `i want the single file` such as `database.properties` and all the `value defined as key value as <key>=<value>` as its a `.properties file`

- if we are using the `mounting a configMap as volume` then we can define the below syntax for the `configMap`

- instead off having the `key-value pair` , here we can define the `key` as the `name of the file we want to give` , because that will become the `file name inside the POD container where it been mount`

- we can define the `<key>=<value> as per the *.properties file as value` inside the `key i.e name of that dile that we have provided`

- hence we can declare that as below for the `database-uri-data.yml file`

    ```yaml
        database-uri-data.yml
        =====================
        apiVersion: v1 # here using the apiVersion as v1 in this case as it belong to the core-group
        kind: ConfigMap # defining the kind of Kubernetes object as ConfigMap
        # here sticking to the old approaches
        # as we are changing the configMaps we need to provide a New Version for the same
        # hence we can define that as below 
        metadata: # defining the name of the configMaps in here
            name: atabase-uri-info-v6 
        data: # here defining the data Key which will be used for the storing the key value pair
            database.properties: | # here we are defining the name of the file as key in this case
                database.url=http://dbserver.<something>.<something>:3306 # first key-value pair
                database.password=P@SSW0rd1 # 2nd key-value pair
                # here using the | defining the multiple line as above key=value because in standard we need to provide the key=value form
                # here is the stack overflow link for reference 
                # https://stackoverflow.com/questions/3790454/how-do-i-break-a-string-in-yaml-over-multiple-lines

    ```

- here we can provide the `workloads.yml` to refer the same by declatring as below 

    ```yaml
        workloads.yaml
        ==============
        apiVersion: apps/v1 # here defining the apiVersion as apps/v1 as the Deployment belong to the apps group
        kind: Deployment # here the kubernetes object is of type Deployment
        metadata: # defining the name for the Deployment in here
            name: position-simulator
        spec: # here the specification with respect to the Deployment been described here
            selector: # selector for the POD label so that Deployment can fetch the corresponding POD based on POD label
                matchlabels:
                    app: position-simulator
            replicas: 1 # replicas for the POD being as 1 here
            template: #template for the POD definition
                metadata: # POD labels been described inside the metadata section
                    labels:
                        app: position-simulator
                spec: # specification for the POD container been defined here 
                    containers: # container details been defined here
                        - name: position-simulator # defining the position simulator as container name in here
                          image: richardchesterwood/k8s-fleetman-position-simulator:release2 # image for the container
                          env: # envVariable for the POD container
                            - name: SPRING_PROFILE_ACTIVE # name fo the env variable
                              value: position-tracker # value of the env Variable
                          volumeMounts: # defining the volumenMount which we create a directory if not exist inside the POD container 
                            - name: database-config-volume # name of the volume mount defined here which will be referenced inside volumes section
                              mountPath: /etc/mydata # defining the mountPath over here , apth inside the container will be created if not exist
                    
                    volumes: # defining the volume which need to be associated with the volumeMounts
                        name: database-config-volume # referencing the volumeMounts name in here
                        configMap: # here defining the configName in here
                            name: database-uri-info-v6 # referencing new name of the configMap that we will define inside the database-uri-data.yml as configMap


    ```

- now if we `deploy both the changes` into the `kubernetes cluster` by `applying the changes` then we can see the below result in that case

- here we can see that result as below 

    ```bash
        kubectl apply -f .
        # applying the changes onto both the `workloads.yml` and `database-uri-data.yml`file to deploy it onto the kubernetes cluster
        configmap/database-uri-info-v6 created
        deployment.apps/queueapp unchanged
        deployment.apps/position-simulator configured
        deployment.apps/position-tracker unchanged
        deployment.apps/api-gateway unchanged
        deployment.apps/webapp unchanged

        # now we can see all the kubernetes object inside the default namespace 
        # if we want to see position-tracker then we cansee the info as below 
        kubectl get all
        # fetching all the kubernetes object inside the default namespace in here
        NAME                                      READY   STATUS    RESTARTS        AGE
        pod/api-gateway-56c46fbcdb-mckrd          1/1     Running   2 (4h44m ago)   31h
        pod/mongodb-578b98fbd4-6m8h8              1/1     Running   2 (4h44m ago)   31h
        pod/position-simulator-7859f68975-xlxn9   1/1     Running   0               16s
        pod/position-tracker-59fdfd8cf4-xvc9s     1/1     Running   0               3h20m
        pod/queueapp-f55dcb97d-8z6xg              1/1     Running   2 (4h44m ago)   31h
        pod/webapp-66765b68df-hcb7r               1/1     Running   0               57m

        NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                          AGE
        service/fleetman-api-gateway        NodePort    10.106.34.66    <none>        8080:30030/TCP                   31h
        service/fleetman-mongodb            ClusterIP   10.99.142.227   <none>        27017/TCP                        31h
        service/fleetman-position-tracker   ClusterIP   10.96.157.227   <none>        8080/TCP                         31h
        service/fleetman-queue              NodePort    10.96.88.35     <none>        8161:30010/TCP,61616:31321/TCP   31h
        service/fleetman-webapp             NodePort    10.100.148.34   <none>        80:30080/TCP                     31h
        service/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                          31h

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           31h
        deployment.apps/mongodb              1/1     1            1           31h
        deployment.apps/position-simulator   1/1     1            1           31h
        deployment.apps/position-tracker     1/1     1            1           31h
        deployment.apps/queueapp             1/1     1            1           31h
        deployment.apps/webapp               1/1     1            1           31h

        NAME                                            DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-56c46fbcdb          1         1         1       31h
        replicaset.apps/mongodb-578b98fbd4              1         1         1       31h
        replicaset.apps/position-simulator-554dc5f486   0         0         0       3h20m
        replicaset.apps/position-simulator-55cd9487fb   0         0         0       29m
        replicaset.apps/position-simulator-5f4dc8c48b   0         0         0       20h
        replicaset.apps/position-simulator-5fdb4ddbd5   0         0         0       31h
        replicaset.apps/position-simulator-67d9f49f5b   0         0         0       20h
        replicaset.apps/position-simulator-6896898cb4   0         0         0       4h25m
        replicaset.apps/position-simulator-68b545b847   0         0         0       4h37m
        replicaset.apps/position-simulator-6db984b5f7   0         0         0       34m
        replicaset.apps/position-simulator-7859f68975   1         1         1       16s
        replicaset.apps/position-simulator-84f66bc755   0         0         0       20h
        replicaset.apps/position-simulator-bf87cb7f6    0         0         0       17h
        replicaset.apps/position-tracker-59fdfd8cf4     1         1         1       31h
        replicaset.apps/position-tracker-5dd5688987     0         0         0       17h
        replicaset.apps/position-tracker-67499f494d     0         0         0       4h37m
        replicaset.apps/position-tracker-7dbf5fcbb4     0         0         0       20h
        replicaset.apps/position-tracker-9cbf78bd9      0         0         0       20h
        replicaset.apps/position-tracker-f874f7467      0         0         0       4h25m
        replicaset.apps/queueapp-f55dcb97d              1         1         1       31h
        replicaset.apps/webapp-66765b68df               1         1         1       31h

        # now if we access the terminal of the position-simulator POD and accessing the terminal to see the file getting created or not 
        kubectl exec -it pod/position-simulator-7859f68975-xlxn9 -- bash
        # here defining the checking the terminal of the position-tracker
        root@position-simulator-7859f68975-xlxn9:/# cd /etc/mydata/
        # going to the specific folder
        root@position-simulator-7859f68975-xlxn9:/etc/mydata# ls
        # checking the content of the folder here
        database.properties # here we can see only one file in this case
        root@position-simulator-55cd9487fb-8rwl5:/etc/mydata# cat database.properties
        # THE OUTPUT WILL BE AS BELOW 
        # here the key value pair will be valid variable which can be set as env variable in this case
        database.url=https://mydatabase.<something>.<something>:3306
        database.password=P@SSWOrd1
        # here defining the key and value as the valid variable which can be used as env variable with value


    ```

