# DaemonSet in kubernetes 

- from the name `DaemonSet` might looks `biong and scary` but they are actually very `trivial`

- A `DaemonSet` `ask kubernetes to ensure` that `all nodes inside the kubernetes cluster` run a `copy of the POD`

- As `Nodes added to the Cluster` the `Corresponding POD` will also going to `added to them`

- if we `remove a Node from the cluster` then the `copy of the POD` will be going to be `garbage collected`

- cleaning up the `DaemonSet` will `clean up the POD` that `it spunned`

- if for example `we have a cluister 3 nodes` and we mentioned a `we define a POD is a part of the kubernetes DaemonSet workload` then `kubernetes` will make sure there will be `one instance of the POD running inside each node` hence there will be `3 instances of the POD`

- if we are considering the `fleetman application` then there will be no need for the `DaemonSet` as there are `no realuse for the same`

- for `microservice` its very rarely used with the `DaemonSet` as we don't `microservice POD to run on each Node`

- even the `webapp` which is a `critical part of the fleetman microservice` , if the `webapp goes down` then `user` will not be able to `use the fleetman microservice`

- but we can handle that with the `Deployment` as `increasing the number of replicas for the Deployment` which will `create another instance` of the `webapp` but that can be spunned in `any node of the kubernetes cluster`

- even though we have the `cluster with 10 nodes` and have `3 AZs` by creating the `3 replicas of the webapp` we can make it as `Highly avaialble`

- we can define the `workloads.yml` to define the increase in `number of replicas as below `  

    ```yaml
        workloads.yml
        =============
        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: Deployment # kubernetes object of type as Deployment
        metadata: # defining the name for the Deployment as webapp
            name: webapp
        spec: # specification for the Deployment 
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: webapp
            replicas: 3 # here the number of replicas as 3 in this case
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: webapp
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: webapp # name of the container
                          image: richardchesterwood/k8s-fleetman-webapp-angular:release2 # image of the container
                          env: # env variable needed for the container
                            - name: SPRING_PROFILE_ACTIVE # name of the env Variable
                              value: production-microservice # value of the env Variable

        ---

        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: Deployment # kubernetes object of type as Deployment
        metadata: # defining the name for the Deployment as queueapp
            name: queueapp
        spec:
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: queueapp
            replicas: 1 # here the number of replicas as 1 in this case
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: queueapp
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: queueapp # name of the container
                          image: richardchesterwood/k8s-fleetman-queue:release2 # image of the container
        
        ---

        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: Deployment # kubernetes object of type as Deployment
        metadata: # defining the name for the Deployment as position-simulator
            name: position-simulator
        spec: # specification for the Deployment 
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: position-simulator
            replicas: 1 # here the number of replicas as 1 in this case
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: position-simulator
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: position-simulator # name of the container
                          image: richardchesterwood/k8s-fleetman-position-simulator:release2 # image of the container
                          env: # env variable needed for the container
                            - name: SPRING_PROFILE_ACTIVE # name of the env Variable
                              value: production-microservice # value of the env Variable


        ---

        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: Deployment # kubernetes object of type as Deployment
        metadata: # defining the name for the Deployment as position-tracker
            name: position-tracker
        spec: # specification for the Deployment 
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: position-tracker
            replicas: 1 # here the number of replicas as 1 in this case
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: position-tracker
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: position-tracker # name of the container
                          image: richardchesterwood/k8s-fleetman-position-tracker:release3 # image of the container
                          env: # env variable needed for the container
                            - name: SPRING_PROFILE_ACTIVE # name of the env Variable
                              value: production-microservice # value of the env Variable


        ---

        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: Deployment # kubernetes object of type as Deployment
        metadata: # defining the name for the Deployment as api-gateway
            name: api-gateway
        spec: # specification for the Deployment 
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: api-gateway
            replicas: 1 # here the number of replicas as 1 in this case
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: api-gateway
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: api-gateway # name of the container
                          image: richardchesterwood/k8s-fleetman-api-gateway:release2 # image of the container
                          env: # env variable needed for the container
                            - name: SPRING_PROFILE_ACTIVE # name of the env Variable
                              value: production-microservice # value of the env Variable


    ```

- we can convert the `webapp` from `Deployment` to `DaemonSet` only by changing the type as below 

- here defining the `DaemonSet` we don't have to use the `replicas` as that depend/based on the `number of nodes` that we have in the `cluster`

- if we define the `DaemonSet` with the `replicas` then it will throw error   

- henece we can redefine the `workloads.yml` file as below with the `DaemonSet`

    ```yaml
        
        workloads.yml
        =============
        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: DaemonSet # kubernetes object of type as DaemonSet # THIS IS THE ONLY CHANGE we need to perform
        metadata: # defining the name for the Deployment as webapp
            name: webapp
        spec: # specification for the Deployment 
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: webapp
            # as based on the Node the PODs will be spuneed hence the there will be no need of the replicas
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: webapp
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: webapp # name of the container
                          image: richardchesterwood/k8s-fleetman-webapp-angular:release2 # image of the container
                          env: # env variable needed for the container
                            - name: SPRING_PROFILE_ACTIVE # name of the env Variable
                              value: production-microservice # value of the env Variable

        ---

        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: Deployment # kubernetes object of type as Deployment
        metadata: # defining the name for the Deployment as queueapp
            name: queueapp
        spec:
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: queueapp
            replicas: 1 # here the number of replicas as 1 in this case
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: queueapp
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: queueapp # name of the container
                          image: richardchesterwood/k8s-fleetman-queue:release2 # image of the container
        
        ---

        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: Deployment # kubernetes object of type as Deployment
        metadata: # defining the name for the Deployment as position-simulator
            name: position-simulator
        spec: # specification for the Deployment 
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: position-simulator
            replicas: 1 # here the number of replicas as 1 in this case
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: position-simulator
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: position-simulator # name of the container
                          image: richardchesterwood/k8s-fleetman-position-simulator:release2 # image of the container
                          env: # env variable needed for the container
                            - name: SPRING_PROFILE_ACTIVE # name of the env Variable
                              value: production-microservice # value of the env Variable


        ---

        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: Deployment # kubernetes object of type as Deployment
        metadata: # defining the name for the Deployment as position-tracker
            name: position-tracker
        spec: # specification for the Deployment 
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: position-tracker
            replicas: 1 # here the number of replicas as 1 in this case
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: position-tracker
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: position-tracker # name of the container
                          image: richardchesterwood/k8s-fleetman-position-tracker:release3 # image of the container
                          env: # env variable needed for the container
                            - name: SPRING_PROFILE_ACTIVE # name of the env Variable
                              value: production-microservice # value of the env Variable


        ---

        apiVersion: apps/v1 # apiVersion as apps/v1 as it belong to the apps Group
        kind: Deployment # kubernetes object of type as Deployment
        metadata: # defining the name for the Deployment as api-gateway
            name: api-gateway
        spec: # specification for the Deployment 
            selector: # selector to select the POD based on the label
                matchLabels: 
                    app: api-gateway
            replicas: 1 # here the number of replicas as 1 in this case
            template: # defining the template for the POD definition 
                metadata: # POD labels been defined in here
                    labels:
                        app: api-gateway
                spec: # specification for the POD been described in here
                    containers: # container details are in here
                        - name: api-gateway # name of the container
                          image: richardchesterwood/k8s-fleetman-api-gateway:release2 # image of the container
                          env: # env variable needed for the container
                            - name: SPRING_PROFILE_ACTIVE # name of the env Variable
                              value: production-microservice # value of the env Variable


    ```

- we can deploy to the `kubernetes eks cluster` by `applying the changes` as below 

    ```bash
        kubectl apply -f workloads.yml
        # here we are deploying the changes to the kubernetes cluster by making the changes 
        # the output will be as below in this case
        daemonset.apps/webapp created


        # as there are 3 nodes in the cluster hence we will be able to see 3 PODs running each running in different Nodes
        kubectl get pods -o wide
        # here we can see the Nodes where the POD been dispatched as below 
        NAME                                READY   STATUS    RESTARTS   AGE     IP               NODE                                              NOMINATED NODE   READINESS GATES
        api-gateway-7c996ff9db-k7kzb        1/1     Running   0          13d     192.168.57.84    ip-192-168-42-197.eu-central-1.compute.internal   <none>           <none>
        mongodb-65bd7dbbfb-pr9ct            1/1     Running   0          13d     192.168.79.188   ip-192-168-77-124.eu-central-1.compute.internal   <none>           <none>
        position-simulator-6f78798c-zqdv2   1/1     Running   0          13d     192.168.38.129   ip-192-168-42-197.eu-central-1.compute.internal   <none>           <none>
        position-tracker-7f5bfddf94-nbb4x   1/1     Running   0          13d     192.168.5.195    ip-192-168-20-235.eu-central-1.compute.internal   <none>           <none>
        queueapp-c679b7cdb-q886g            1/1     Running   0          13d     192.168.57.74    ip-192-168-42-197.eu-central-1.compute.internal   <none>           <none>
        webapp-5bdb5b4bd7-nnr2b             1/1     Running   0          13d     192.168.9.217    ip-192-168-20-235.eu-central-1.compute.internal   <none>           <none>
        webapp-h5l7j                        1/1     Running   0          2m28s   192.168.32.51    ip-192-168-42-197.eu-central-1.compute.internal   <none>           <none>
        webapp-hc45w                        1/1     Running   0          2m28s   192.168.13.159   ip-192-168-20-235.eu-central-1.compute.internal   <none>           <none>
        webapp-kswwn                        1/1     Running   0          2m28s   192.168.94.76    ip-192-168-77-124.eu-central-1.compute.internal   <none>           <none>


        # if we see here in every node we can see the DaemonSet in this case 
        # we can see the info about the daemon set as well by using the command as below 
        kubectl get daemonsets/ds
        # the output will be as below 
        # here we can see the desire and current count in this case
        NAME     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
        webapp   3         3         3       3            3           <none>          3m57s

        # we can see the description of DaemonSet as below 
        kubectl describe ds webapp
        # the poutput will show the DaemonSet description 
        Name:           webapp
        Selector:       app=webapp
        Node-Selector:  <none>
        Labels:         <none>
        Annotations:    deprecated.daemonset.template.generation: 1
        Desired Number of Nodes Scheduled: 3
        Current Number of Nodes Scheduled: 3
        Number of Nodes Scheduled with Up-to-date Pods: 3
        Number of Nodes Scheduled with Available Pods: 3
        Number of Nodes Misscheduled: 0
        Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
        Pod Template:
        Labels:  app=webapp
        Containers:
        webapp:
            Image:      richardchesterwood/k8s-fleetman-webapp-angular:release2
            Port:       <none>
            Host Port:  <none>
            Environment:
            SPRING_PROFILES_ACTIVE:  production-microservice
            Mounts:                    <none>
        Volumes:                     <none>
        Events:
        Type    Reason            Age    From                  Message
        ----    ------            ----   ----                  -------
        Normal  SuccessfulCreate  8m56s  daemonset-controller  Created pod: webapp-hc45w
        Normal  SuccessfulCreate  8m56s  daemonset-controller  Created pod: webapp-kswwn
        Normal  SuccessfulCreate  8m56s  daemonset-controller  Created pod: webapp-h5l7j

        # when we do the kubectl get all to see all the kubernetes object inside the default namespace we can also see the DaemonSet as below 
        kubectl get all
        # displaying all kubernetes object inside the default namespace 
        NAME                                    READY   STATUS    RESTARTS   AGE
        pod/api-gateway-7c996ff9db-k7kzb        1/1     Running   0          13d
        pod/mongodb-65bd7dbbfb-pr9ct            1/1     Running   0          13d
        pod/position-simulator-6f78798c-zqdv2   1/1     Running   0          13d
        pod/position-tracker-7f5bfddf94-nbb4x   1/1     Running   0          13d
        pod/queueapp-c679b7cdb-q886g            1/1     Running   0          13d
        pod/webapp-5bdb5b4bd7-nnr2b             1/1     Running   0          13d
        pod/webapp-h5l7j                        1/1     Running   0          10m
        pod/webapp-hc45w                        1/1     Running   0          10m
        pod/webapp-kswwn                        1/1     Running   0          10m
            TYPE           CLUSTER-IP       EXTERNAL-IP                                                                  PORT(S)              AGE
        service/fleetman-api-gateway        ClusterIP      10.100.93.38     <none>                                                                       8080/TCP             13d
        service/fleetman-mongodb            ClusterIP      10.100.63.51     <none>                                                                       27017/TCP            13d
        service/fleetman-position-tracker   ClusterIP      10.100.11.40     <none>                                                                       8080/TCP             13d
        service/fleetman-queue              ClusterIP      10.100.206.185   <none>                                                                       8161/TCP,61616/TCP   13d
        service/fleetman-webapp             LoadBalancer   10.100.103.87    af73ec9147f9b46dd9cda3b668a1373a-1503150957.eu-central-1.elb.amazonaws.com   80:31226/TCP         13d
        service/kubernetes                  ClusterIP      10.100.0.1       <none>                                                                       443/TCP              13d

        NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
        daemonset.apps/webapp   3         3         3       3            3           <none>          10m

        NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/api-gateway          1/1     1            1           13d
        deployment.apps/mongodb              1/1     1            1           13d
        deployment.apps/position-simulator   1/1     1            1           13d
        deployment.apps/position-tracker     1/1     1            1           13d
        deployment.apps/queueapp             1/1     1            1           13d
        deployment.apps/webapp               1/1     1            1           13d

        NAME                                          DESIRED   CURRENT   READY   AGE
        replicaset.apps/api-gateway-7c996ff9db        1         1         1       13d
        replicaset.apps/mongodb-65bd7dbbfb            1         1         1       13d
        replicaset.apps/position-simulator-6f78798c   1         1         1       13d
        replicaset.apps/position-tracker-7f5bfddf94   1         1         1       13d
        replicaset.apps/queueapp-c679b7cdb            1         1         1       13d
        replicaset.apps/webapp-5bdb5b4bd7             1         1         1       13d
 


    ```

- keep in mind that `we have separate the Deployment and Services` hence if we are running the `workloads.yml` which only contains the `Deployment` , then we will be end up getting the error for the webapp POD Deployment because the `webapp POD deployment` need the `Service` oparticularly the `api-gateway Service which is fleetman-ap-gatway`  
to fetch other `PODs` through the `Service`

- the `DaemonSet` is a `special kubernetes object` we will only use them `in special cases where we want the POD in each of the Node in cluster`

- such as `cluster storage daemon` such as `glusterd/ceph` then we need them as `each node`

- also if we want to `fetch the Logs from all the POD inside the Node` then we need to use the `fluend and Logstash as the DaemonSet` which we have used in the `logging with ELK` section

- here we need the `each instance of the fluentd running inside every Node inside the cluster` to `ingest the logs of all the POD into the flentd` hence we need to run as the `DaemonSet`

- `DaemonSets` are basically the `kubernetes Deployment` , but here `we will not mention` the `number of replicas we want to spin`