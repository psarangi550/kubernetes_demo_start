# introduction to configMap and Secrets in kubernetes

- here we will be looking at `configMap and Secrets in kubernetes`

- one of the great thing of `working with images` is that `we can just download the image and use it`

- the `image will come as pre-packed` `with all the environment it need to run`

- here we know that `queue image` came `pre-packed by somebody` at `apache` and we can just work with that `image` by referencing it inside the `POD` configuration

    ```yaml
        workloads.yml
        =============
        apiVersion: apps/v1 # defining the apiVersion as apps/v1 in here as it belong to the core group
        kind: Deployment # here the type of object will be as Deployment
        metadata: # defining the name for the Deployment in here
            name: queueapp
        spec: # specification for the Deployment being defined in here
            replicas: 1 # setting up the replicas as 1 in this case
            selector: # defining the selector as baeed on the POD label
                matchLabels:
                    app: queueapp
            template: # defining the template for the POD being run here
                metadata: # deefining the POD labels inside the metadata section of the POD
                    labels: # POD label been defined here
                        app: queueapp
                spec: # specification for the POD container
                    containers: # container being defined in this case
                    - name: queueapp # name of the container
                        image: richardchesterwood/k8s-fleetman-queue:release2 #image of the container


    ```

- we don't know `how that image been build on which image` , such as the `activeMQ image` which been build on `JAVA` , but in  order to run the `activeMQ image` we  don't have to `install JAVA or JVM` or `setup` the `environment variable`

- but `for that convinience` we do pay a `little bit of price` i.e  `inflexibility`

- we will get `what the image been providing` , if we want to `tailor the specific image` i.e if  we want to `change it in any way` then `its not possible as image is immutable`

- for example if we want to add an `additional file` onto the `docker image` then we can't `as the image is immutable`

- what we can is do, is derive a `new image` from the `existing image` , `tailor or customize` it , we can `certainly` do that 

- but `by default` the `image` which came is `pre-build and finished`

- it will be `quite hard sometimes with the image` in order to `pass in your own data into the container of the pre-build image`

- The most common way , we can pass `variables` as `env variable` into the `container of the docker image`  like in the `position simulator`

- we have `contacted the developer of the image ` or `got the reference document from Dockerhub` to set the `env variable` and we can set the `env variable` as `SPRING_PROFILE_ACTIVE` as `prodcution-microservice` for the `position-tracker` docker image

- we can `pass variable` to the `image` which will later become the `POD container` as `environment variable`

- but sooner we will realises that `there is a bit of a problem here` , we are `duplicating` the `environment variable` in here 

- by accident that `already happening in the workloads.yml` file , we realizes that `we are duplicating the environement variable` accross `multiple PODs`

- we can see that `api-gateway` , `position-simulator` , `position-tracker` , `webapp`  `Deployment POD container we are` copying the `environement` pretty much on every place , which is `bad smell`

- here is the example of the `workloads.yml` file 

    ```yaml
        
        workloads.yml
        ==============
        
        apiVersion: apps/v1 # defining the apiVersion as apps/v1 in here as it belong to the apps group
        kind: Deployment # here the type of object will be as Deployment
        metadata: # here we are setting up the name for the Deployment in here
            name: queueapp
        spec: # defining the specification for the Deployment
            replicas: 1 # defining the number of replicas in this case is 1
            selector: # using the selectors for the POD labels in this case
                matchLabels:
                    app: queueapp
            template: # defining the template for the POD
                metadata: # setting up the POD label for the selector
                    labels:
                        app: queueapp
                spec: # specification for the POD container
                    containers: # here we will define the container details
                        - name: queueapp # name of the POD container
                          image: richardchesterwood/k8s-fleetman-queue:release2 # image of the POD container

        ---

        apiVersion: apps/v1 # defining the apiVersion as apps/v1 in here as it belong to the apps group
        kind: Deployment # here the type of object will be as Deployment
        metadata: # here we are setting up the name for the Deployment in here
            name: position-simulator
        spec: # defining the specification for the Deployment
            replicas: 1 # defining the number of replicas in this case is 1
            selector: # using the selectors for the POD labels in this case
                matchLabels:
                    app: position-simulator

            template: # defining the template for the POD
                metadata: # setting up the POD label for the selector
                    labels:
                        app: position-simulator
                spec: # defining the specification for the POD container
                    containers: # defining the POD container details in here
                        - image: richardchesterwood/k8s-fleetman-position-simulator:release2 # name of the image
                          name: position-simulator # name of the container
                          env: # environment variable for the POD container
                            - name: SPRING_PROFILES_ACTIVE # name of rhe env variable
                              value: production-microservice # value of the env variable


        ---

        apiVersion: apps/v1  # defining the apiVersion as apps/v1 in here as it belong to the apps group
        kind: Deployment # here the type of object will be as Deployment
        metadata: # here we are setting up the name for the Deployment in here
            name: position-tracker
        spec: # specification for the Deployment descreibed here
            replicas: 1 # defining the number of replicas in this case is 1
            selector: # selector for the POD container defined in here
                matchLabels:
                    app: position-tracker
            template: #template for the POD container defined here
                metadata: # POD label so that selector can fetch it
                    labels:
                        app: position-tracker
                spec: # specification for the POD container
                    containers: # here are the container details
                    - name: position-tracker # name of the POD container
                        image: richardchesterwood/k8s-fleetman-position-tracker:release3 # image of the POD container
                        env: # environement for the POD container
                        - name: SPRING_PROFILES_ACTIVE # env variable name for the POD container 
                          value: production-microservice # value for the POD container

        ---

        apiVersion: apps/v1  # defining the apiVersion as apps/v1 in here as it belong to the apps group
        kind: Deployment # here the type of object will be as Deployment
        metadata: # here we are setting up the name for the Deployment in here
            name: api-gateway
            namespace: default # here using the default namespace in here
        spec:
            selector: #selector so that based on POD label the changes Deployment can detect the same 
                matchLabels:
                    app: api-gateway
            replicas: 1 # defining the number of replicas in this case is 1
            template: # defining the template for the POD described in here
                metadata: # defining the POD label in this case
                    labels:
                        app: api-gateway
                spec: # specification for the POD described here
                    containers: #container details given here
                        - name: api-gateway
                          image: richardchesterwood/k8s-fleetman-api-gateway:release2
                          env: # environment variable for the POD container
                            - name: SPRING_PROFILES_ACTIVE # name of the env variable
                              value: production-microservice # value of the env variable

        ---

        apiVersion: apps/v1 # defining the apiVersion as apps/v1 in here as it belong to the apps group
        kind: Deployment # here the type of object will be as Deployment
        metadata: # here we are setting up the name for the Deployment in here
            name: webapp 
            namespace: default # here using the default namespace in here
        spec: # defining the specification for the POD container 
            selector: # selector for the Deployment based on POD label
                matchLabels:
                    app: webapp
            replicas: 1 # defining the number of replicas in this case is 1
            template: # template for the POD been describe in here
                metadata: # metadata for the POD been described in here as key value pair
                    labels:
                        app: webapp
                spec: # specification for the container provided here
                    containers: # container details provided here
                        - name: webapp # name of the container
                          image: richardchesterwood/k8s-fleetman-webapp-angular:release2 # image of the container
                          env: # defining the environment variable in this case
                            - name: SPRING_PROFILES_ACTIVE # name of the env variable
                              value: production-microservice # value of the env variable


    ```


- it will be really great , if we can `gather together` `any data` we want share `among multiple POD container`

- if `we can gather those data and store it in kubernetes then it would be more helpful` which be shared among all the `POD container` , which `configMap` allows us to do


- For example 
  
  - lets suppose we will be using `external database` on some of the `POD container`
  
  - we have the `DATABASE_URI` for that `external database` such as `https://dbserver.<something>.<something>:3306` that we want to use it inside some of the `POD container`
  
  - if we want to do it then we need declare the `variable as env variable` as we want to pass the `DATABASE_URI` to the `POD container`
  
  - again we need to `mention multiple time in each POD` as `env variable` the `name of the Variable` as `DATABASE_URI` and the `value` as `https://dbserver.<something>.<something>:3306` which will again be a repetation in multiple `POD container` definition
  
  - if the `DATABASE_URI` changes in future then we have to `search every POD` and change the `value` for the Same , which is not good
  
  - we can store the `DATABASE_URI` with the `Value` in a `kubernetes ConfigMap` as below 


- A `configMap` is a simple `object in kubernetes` for which we can define the `config yaml` as below 

    
    ```yaml
        database-uri-data.yml
        ======================
        apiVersion: v1 # here the configMap belong to the core group hence we can mention as below 
        kind: ConfigMap # defining the kubernetes object opf type as ConfigMap
        metadata: # providing the name for the ConfigMap in here
            name: database-uri-info
            namespace: default # specifying the namespace as default in here
        data: # here we need to define a key called data where we will define the value as key value pair
            # defining the value as the key value pair
            # here this is not a list bit rather individual dict with key value pair
            database.url: https://dbserver.<something>.<something>:3306
            database.password: P@SSW0RD 


    ```

- we can `deploy these changes` to the `cluster` by `applying the changes as below` 

    ```bash
        kubectl apply -f database-uri-data.yml
        # providing the database uri in this case
        configmap/database-uri-info created

    ```

- here we can see the `details` about the `configmap` by using it as below 

    ```bash
        kubectl get cm/configmap 
        # fetching all the configmap in the default namespace
        # the output will be as below 
        NAME                DATA   AGE
        database-uri-info   2      2m2s
        
        # we dcan see furthermore details about the config by using the command as below 
        kubectl describe cm/configmap <name of the configmaps>
        # here we can see the details as below
        kubectl describe cm/configmap database-uri-info
        # below will be response for the same 
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
        https://dbserver.<something>.<something>:3306

        BinaryData
        ====

        Events:  <none>

    ```