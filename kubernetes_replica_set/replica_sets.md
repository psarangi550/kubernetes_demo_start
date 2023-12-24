# kubernetes Replica Set 

- with the `POD` and `Services` we know the `core element` of the `kubernetes` , but very `rarely` we deal with `POD` directly

- in the production environment we are more likely to work with `either deployments or replica set`

- `PODs` are `very basic and disposable object` in `kubernetes` , in a `full running production system` we can have `PODs which will going to die`

- there can be `multiple reason for POD to be dead` inside a `full running production system`

  - if a `node going to be get failed` in a `kubernetes cluster` then the `all the PODs running inside the node` will going to `die`

  - if a `POD` `consume too many resources such as CPU` then `kubernetes` will going to `kill that POD`
  
- `PODs` can be `short lived` for whatever reason

- if we are `deploying` the `POD` directly to the `kubernetes cluster` like how we are doing until now `by defining the kubernetes POD definition inside the yml file` and `applying those changes by using the kubectl apply -f <pod definition yml file>` , then `we are responsible` to `pods lifetime` in that case also the `wellfare for that POD`

- that means if `any of POD which been deployed directly to the cluster` then that `might not be able to come back as we are only responsible` for its `lifetime and wellfare for the POD` inside the `kubernetes cluster`

- we can see the `kubernetes Service` details using the below command 

  
  ```bash
      kubectl get all
      # fetching all object from kubenetes cluster as below
      pod/queueapp    1/1     Running   1 (40s ago)   8h
      pod/webapp      1/1     Running   6 (40s ago)   3d15h
      pod/webappnew   1/1     Running   6 (40s ago)   3d15h

      NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
      service/fleetman-queueapp   NodePort    10.100.177.109   <none>        8161:30010/TCP   8h
      service/fleetman-webapp     NodePort    10.106.68.128    <none>        80:30080/TCP     23h
      service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          3d17h
      
      # if we want to fetch the details about the fleetman-webapp then we can see that by using the command as below 
      kubectl describe svc fleetman-webapp
      # using the command as kubernetes describe <object> <object name> we can see the details in kubernetes
      # below will be the output in that case
      Name:                     fleetman-webapp
      Namespace:                default
      Labels:                   <none>
      Annotations:              <none>
      Selector:                 app=webappnew,release=0-5 # here we can see that its been referencing the POD with name as webappnew and release=0-5
      Type:                     NodePort
      IP Family Policy:         SingleStack
      IP Families:              IPv4
      IP:                       10.106.68.128
      IPs:                      10.106.68.128
      Port:                     http  80/TCP
      TargetPort:               80/TCP
      NodePort:                 http  30080/TCP
      Endpoints:                10.244.0.20:80
      Session Affinity:         None
      External Traffic Policy:  Cluster
      Events:                   <none>
  
  ```

- for some reason the `POD` with the name as `webappnew` got `removed` , in order to simulate that we can use the `kiubectl delete <object> <object name>` as below in that case

- when we use the `kubectl delete command` which `by default will gracefully try to shutdown` the `POD` i.e `going to close the container and close app file as well`

- we can also use the `--force` along with the `kubectl delete <object> <object name> --force` which will forceably going to `delete the kubernetes object` 

- we can do that as below 

  
  ```bash
      kubectl delete pod/po webappnew
      # removing the pod for simulating the POD stopped or die by any reason
      # then below will be the output
      pod "webappnew" deleted
  
  ```

- if we are a `docker swarm user` then we can expect the `POD will restarted` , but in case of `kubernetes that ain't going to happen`

- now if we are checking the `kubernetes object inside the minikube kubernetes cluster` then we can do that as below 

  
  ```bash
      kubectl get all
      # fetching all the kubernetes object inside the minikube kubernetes cluster
      # we can see the oputput as below 

      NAME           READY   STATUS    RESTARTS        AGE
      pod/queueapp   1/1     Running   1 (9m41s ago)   8h
      pod/webapp     1/1     Running   6 (9m41s ago)   3d15h

      NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
      service/fleetman-queueapp   NodePort    10.100.177.109   <none>        8161:30010/TCP   8h
      service/fleetman-webapp     NodePort    10.106.68.128    <none>        80:30080/TCP     23h
      service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          3d17h
  
  
  ```

- now when we try to access the `webapp` on the minikube cluster ip using the command as `minikube ip` and accessing the `nodePort` then we can see that `we are unable to access it` because due to the `selector of the kubernetes service fleetman-webapp` still been looking for the `app=webappnew ,release=0-5`

- we don't want that to happen as its not in the sprit of kubernetes 

- this is the reason why we don't deploy `POD` directly to the `kubernetes cluster` , instead we can deploy `replica-set` in that case

- All a `replica set` is `additional configuration` provided to `kubernetes`

- previously we can see the we have `multiple pods `containing `web-container` and `micro-service`

- ![Alt text](image.png)

- with the help of `replica-set` we are providing `additional piece of info to kubernetes` that `we can specify How many instances of the POD should run at a given point of time`

- here we are telling kubenetes `How many instances of the POD` should run `at a given point point of time` , we can start simple and simply say `we want one instance of the POD should be running at a given time`

- if the `POD` been `dead` for `any reason` the `kubernetes will spun a new POD instance and make it running in that place`

- we need to do that for all of the `POD` that we have defined inside the `kubernetes cluster`

- ![Alt text](image-1.png)

- we can keep the `replica-set` as `1` for the `POD` , but we can set to `any number` that we want

### How to define a Replica-Set in yml format 

- if we go to the [Kubernetes 1.28 API Docs](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#replicaset-v1-apps) then underneath the `Workload` section wwe can also seee the `ReplicaSet v1 apps` 

- here we can see the `group` for the `replica-set v1 apps` as `apps` in here hence we can use the `apiVersion` as `apiVersion: app/v1`

- where as `group` for the `PODs` and `Services` will be in the `core` we can see the `apiVersion` as `v1` as `apiVersion: v1`

- these `groups` are nnothing but the `folder` under which the `corresponding API folder been stored` 

- these `apiVersion` will actually allow us to add the `additional kubernetes feature` based on the `version` to the `use new feature definiation yml file` which are in `development`

- that `provide safety` if the new version have to be `changed` then `by changing the apiVersion to the newVersion` we can implement those changes 

- preveiously `replica-set` were used as the `extension` but that has added to `v1 Version` under the group of `apps` 

- but if we want to `replica-set` as below  

  
  ```yaml
      pod-replica-example.yml
      =======================
      apiVersion : apps/v1 # referecing the version as the app/v1 in this case out in here
      kind: ReplicaSet # using the kind as RreplicaSet in here
      metadata: # defining the metadat with name for the replica-set
        # Unique key of the ReplicaSet instance
        name: replicaset-example
      spec:
        # 3 Pods should exist at all times.
        replicas: 3 # defining that 3 POD should exist at once instance 
        selector: # defining the selector over here
          matchLabels: # defining the matchLabel as app:nginx over her which will select from the POD label
            app: nginx
        template: # here inside the template we will describe all the POD definition over here 
          metadata: # defining the metadata for the POD
            name: nginxapp # defining the POD name over here
            labels: # defining the pod label which will be selected from the POD using the selector
              app: nginx 
          spec: # defining the specifgication for the POD
            containers: # defining the container that need to run inside the POD
              - name: nginxapp # defining the docker container name
                image: nginx:1.14 # defining the image for the container to pull the image from
  
  ```

- we don't have to write the `separate yml file` for the `POD` and `replica-set`

- here the above `replica-set` yaml file kind of consider as `2 in 1` , where its the combination of `POD definition` + `Replica-set Definition`

- here on the `metadata` under the `template` inside the `spec` will define the `definition for the PODs`

# Writing the Replica-set

- if we want to put `we can put the entire POD Definition` into a `single yml file` for the `entire architecture`

- `kubernetes does not have any rules` regarding `what we should put onto the yml file`

- we can have `PODs` , `Services` and `Replica-seets` `mixed togethere` into a `single yml file ` or we can separate them into `separate yml file` as well , its entirely upto you

- if we have `1000 PODs` for the `micro-service architecture` then we don't want to all the `POD definition` into a `single file ` , we will try to `decide how to separate them or split them`

- but for this `micro-service` architecture in this course we can put all the `PODs` into `single yml file` and `Services` into `another yaml file` 

- we can update the `current POD definition file` into an `Replica-Set definition yml file`

- we can't have `POD` and `Replica-Set` exist together , we can have `Either the Replica-Set or the POD definition` i.e `one or other`

- when we define the `replica-set` then `we can provide the name of the replica-set as metadata` section  

- if we are defining the `replica set` we don't have to `provide each POD with a specific name` with the `replica-set` `POD` name will be `auto-generated` 

- we can define the `replica-set` as below

  
  
  ```yaml
      pod-replicas.yaml/workload.yml
      ==============================
      apiVersion: apps/v1 # for defining the replica set we need to define the apiVersion as apps/v1 in that case
      kind: ReplicaSet # defining the ReplicaSet in that case over here
      metadata: # providing the name for the replicaset with the name as webapp in this particular case
        name: webapp 
      spec: # defining the specification in that case 
        
        replicas: 1 # dsefining how many instances of the POD should be running on a given time 

        selector: # defining the Selector with the key value pair which will help in picking the PODs by the replica set
          matchLabels: # defining the matchLabel to provide which of the key value pair match with the POD label
            app: webapp # defining the name for the key value pair match with the POD label

        template: # here we can define the POD definition inside the template
          metadata: #defining the name for the POD as well as the POD label
            # but as we know while creating the replicaSet the it will automatically provide the name for the POD hence need not have to provide the same 
            labels: # defining the labels for the POD in this case out in here
              app: webapp # providing the label as the same key-value pair so that replica set can pick the specific POD in this particular case
          
          spec: # defining the specification for the POD in this particular case
            containers: # defining the name and image of the container in this case out in here
              - name: webapp # providing the container with the name as webapp
                image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
                # defining the image for the container in this particular case out in here
      
      --- # defining the document separator to define another POD for the queueapp
      
      apiVersion: v1 # defining the apiVersion as v1 in  here as we are defining the POD in this case 
      kind: Pod # defining the type of object in this case is of POD
      metadata: # defining the metadata for the POD in this case 
        name: queueapp # defining the name for the POD in here
        labels: # defining the label in the format as Key-value pair
          app: queueapp # defining the name as the key value pair in this case
      spec: # defining the specification for the POD in this case
        containers: # defining the container in this case which will be spunned because of containers# kubernetes Replica Set 

- with the `POD` and `Services` we know the `core element` of the `kubernetes` , but very `rarely` we deal with `POD` directly

- in the production environment we are more likely to work with `either deployments or replica set`

- `PODs` are `very basic and disposable object` in `kubernetes` , in a `full running production system` we can have `PODs which will going to die`

- there can be `multiple reason for POD to be dead` inside a `full running production system`

  - if a `node going to be get failed` in a `kubernetes cluster` then the `all the PODs running inside the node` will going to `die`

  - if a `POD` `consume too many resources such as CPU` then `kubernetes` will going to `kill that POD`
  
- `PODs` can be `short lived` for whatever reason

- if we are `deploying` the `POD` directly to the `kubernetes cluster` like how we are doing until now `by defining the kubernetes POD definition inside the yml file` and `applying those changes by using the kubectl apply -f <pod definition yml file>` , then `we are responsible` to `pods lifetime` in that case also the `wellfare for that POD`

- that means if `any of POD which been deployed directly to the cluster` then that `might not be able to come back as we are only responsible` for its `lifetime and wellfare for the POD` inside the `kubernetes cluster`

- we can see the `kubernetes Service` details using the below command 

  
  ```bash
      kubectl get all
      # fetching all object from kubenetes cluster as below
      pod/queueapp    1/1     Running   1 (40s ago)   8h
      pod/webapp      1/1     Running   6 (40s ago)   3d15h
      pod/webappnew   1/1     Running   6 (40s ago)   3d15h

      NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
      service/fleetman-queueapp   NodePort    10.100.177.109   <none>        8161:30010/TCP   8h
      service/fleetman-webapp     NodePort    10.106.68.128    <none>        80:30080/TCP     23h
      service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          3d17h
      
      # if we want to fetch the details about the fleetman-webapp then we can see that by using the command as below 
      kubectl describe svc fleetman-webapp
      # using the command as kubernetes describe <object> <object name> we can see the details in kubernetes
      # below will be the output in that case
      Name:                     fleetman-webapp
      Namespace:                default
      Labels:                   <none>
      Annotations:              <none>
      Selector:                 app=webappnew,release=0-5 # here we can see that its been referencing the POD with name as webappnew and release=0-5
      Type:                     NodePort
      IP Family Policy:         SingleStack
      IP Families:              IPv4
      IP:                       10.106.68.128
      IPs:                      10.106.68.128
      Port:                     http  80/TCP
      TargetPort:               80/TCP
      NodePort:                 http  30080/TCP
      Endpoints:                10.244.0.20:80
      Session Affinity:         None
      External Traffic Policy:  Cluster
      Events:                   <none>
  
  ```

- for some reason the `POD` with the name as `webappnew` got `removed` , in order to simulate that we can use the `kiubectl delete <object> <object name>` as below in that case

- when we use the `kubectl delete command` which `by default will gracefully try to shutdown` the `POD` i.e `going to close the container and close app file as well`

- we can also use the `--force` along with the `kubectl delete <object> <object name> --force` which will forceably going to `delete the kubernetes object` 

- we can do that as below 

  
  ```bash
      kubectl delete pod/po webappnew
      # removing the pod for simulating the POD stopped or die by any reason
      # then below will be the output
      pod "webappnew" deleted
  
  ```

- if we are a `docker swarm user` then we can expect the `POD will restarted` , but in case of `kubernetes that ain't going to happen`

- now if we are checking the `kubernetes object inside the minikube kubernetes cluster` then we can do that as below 

  
  ```bash
      kubectl get all
      # fetching all the kubernetes object inside the minikube kubernetes cluster
      # we can see the oputput as below 

      NAME           READY   STATUS    RESTARTS        AGE
      pod/queueapp   1/1     Running   1 (9m41s ago)   8h
      pod/webapp     1/1     Running   6 (9m41s ago)   3d15h

      NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
      service/fleetman-queueapp   NodePort    10.100.177.109   <none>        8161:30010/TCP   8h
      service/fleetman-webapp     NodePort    10.106.68.128    <none>        80:30080/TCP     23h
      service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          3d17h
  
  
  ```

- now when we try to access the `webapp` on the minikube cluster ip using the command as `minikube ip` and accessing the `nodePort` then we can see that `we are unable to access it` because due to the `selector of the kubernetes service fleetman-webapp` still been looking for the `app=webappnew ,release=0-5`

- we don't want that to happen as its not in the sprit of kubernetes 

- this is the reason why we don't deploy `POD` directly to the `kubernetes cluster` , instead we can deploy `replica-set` in that case

- All a `replica set` is `additional configuration` provided to `kubernetes`

- previously we can see the we have `multiple pods `containing `web-container` and `micro-service`

- ![Alt text](image.png)

- with the help of `replica-set` we are providing `additional piece of info to kubernetes` that `we can specify How many instances of the POD should run at a given point of time`

- here we are telling kubenetes `How many instances of the POD` should run `at a given point point of time` , we can start simple and simply say `we want one instance of the POD should be running at a given time`

- if the `POD` been `dead` for `any reason` the `kubernetes will spun a new POD instance and make it running in that place`

- we need to do that for all of the `POD` that we have defined inside the `kubernetes cluster`

- ![Alt text](image-1.png)

- we can keep the `replica-set` as `1` for the `POD` , but we can set to `any number` that we want

### How to define a Replica-Set in yml format 

- if we go to the [Kubernetes 1.28 API Docs](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#replicaset-v1-apps) then underneath the `Workload` section wwe can also seee the `ReplicaSet v1 apps` 

- here we can see the `group` for the `replica-set v1 apps` as `apps` in here hence we can use the `apiVersion` as `apiVersion: app/v1`

- where as `group` for the `PODs` and `Services` will be in the `core` we can see the `apiVersion` as `v1` as `apiVersion: v1`

- these `groups` are nnothing but the `folder` under which the `corresponding API folder been stored` 

- these `apiVersion` will actually allow us to add the `additional kubernetes feature` based on the `version` to the `use new feature definiation yml file` which are in `development`

- that `provide safety` if the new version have to be `changed` then `by changing the apiVersion to the newVersion` we can implement those changes 

- preveiously `replica-set` were used as the `extension` but that has added to `v1 Version` under the group of `apps` 

- but if we want to `replica-set` as below  

  
  ```yaml
      pod-replica-example.yml
      =======================
      apiVersion : apps/v1 # referecing the version as the app/v1 in this case out in here
      kind: ReplicaSet # using the kind as RreplicaSet in here
      metadata: # defining the metadat with name for the replica-set
        # Unique key of the ReplicaSet instance
        name: replicaset-example
      spec:
        # 3 Pods should exist at all times.
        replicas: 3 # defining that 3 POD should exist at once instance 
        selector: # defining the selector over here
          matchLabels: # defining the matchLabel as app:nginx over her which will select from the POD label
            app: nginx
        template: # here inside the template we will describe all the POD definition over here 
          metadata: # defining the metadata for the POD
            name: nginxapp # defining the POD name over here
            labels: # defining the pod label which will be selected from the POD using the selector
              app: nginx 
          spec: # defining the specifgication for the POD
            containers: # defining the container that need to run inside the POD
              - name: nginxapp # defining the docker container name
                image: nginx:1.14 # defining the image for the container to pull the image from
  
  ```

- we don't have to write the `separate yml file` for the `POD` and `replica-set`

- here the above `replica-set` yaml file kind of consider as `2 in 1` , where its the combination of `POD definition` + `Replica-set Definition`

- here on the `metadata` under the `template` inside the `spec` will define the `definition for the PODs`

# Writing the Replica-set

- if we want to put `we can put the entire POD Definition` into a `single yml file` for the `entire architecture`

- `kubernetes does not have any rules` regarding `what we should put onto the yml file`

- we can have `PODs` , `Services` and `Replica-seets` `mixed togethere` into a `single yml file ` or we can separate them into `separate yml file` as well , its entirely upto you

- if we have `1000 PODs` for the `micro-service architecture` then we don't want to all the `POD definition` into a `single file ` , we will try to `decide how to separate them or split them`

- but for this `micro-service` architecture in this course we can put all the `PODs` into `single yml file` and `Services` into `another yaml file` 

- we can update the `current POD definition file`  convert it into an `Replica-Set definition yml file`

- we can have `POD` and `Replica-Set` exist together , we can have `Either the Replica-Set or the POD definition` i.e `one or other`

- when we define the `replica-set` then `we can provide the name of the replica-set as metadata` section  

- if we are defining the `replica set` we don't have to `provide each POD with a specific name` with the `replica-set` `POD` name will be `auto-generated` 

- here also we will be `omitting the POD label as releaase:0-5` as `its no longer required` in this case out here

- As the `PODs` were being managed by the `Replica-Set` hence we don't have to `necessaryly provide name to the POD`

- as we are using the `POD labels` while `using the selector  inside the kubernetes Service` hence we need that info as well to be defined

- while defining the `Replica-Set` inside the `spec` section we need to define 3 things
  
  -  `replicas` :- `How many instances of the POD running at a given time`
  
  -  `Selector` :- which will behave exacly as the `Service Selector` in order to help `replica-set` to fetch the `particular POD` based on the `POD labels` such as `Prod/Dev`
                
                :- while selecting the `POD` we need to define the `matchLabels` which will help in fetch the `labels` for the `POD` from the `replicaset` 
  
  - `template` :- where we will have to define the `POD definition inside the Replica sset that we want to create the instance on a given point of time`  

- as we are putting `both PODs and replica-set` inside the `yml defination` then we can call the file as `workloads.yml` as both(POD and ReplicaSet) comes under `Workload API`

- when we are developing the `ReplicaSet` we don't have to bother about the `Selector` as we have achived it using the `Kubernetes Service Selector` , hence in the `ReplicaSet` we can define the `Selector` same as the `POD label` 

- we can define the `replica-set` as below

- ![Alt text](image-2.png)
  
  
  ```yaml
      pod-replicas.yaml/workload.yml
      ==============================
      apiVersion: apps/v1 # for defining the replica set we need to define the apiVersion as apps/v1 in that case
      kind: ReplicaSet # defining the ReplicaSet in that case over here
      metadata: # providing the name for the replicaset with the name as webapp in this particular case
        name: webapp # when we do a kubectl get all we will get the result as ReplicaSet/webapp listed out there
      spec: # defining the specification in that case 
        
        replicas: 1 # defining how many instances of the POD should be running on a given time 

        selector: # defining the Selector with the key value pair which will help in picking the PODs by the replica set
          matchLabels: # defining the matchLabel to provide which of the key value pair match with the POD label
            app: webapp # defining the name for the key value pair match with the POD label

        template: # here we can define the POD definition inside the template
          metadata: #defining the name for the POD as well as the POD label
            # but as we know while creating the replicaSet the it will automatically provide the name for the POD hence need not have to provide the same 
            labels: # defining the labels for the POD in this case out in here
              app: webapp # providing the label as the same key-value pair so that replica set can pick the specific POD in this particular case
          
          spec: # defining the specification for the POD in this particular case
            containers: # defining the name and image of the container in this case out in here
              - name: webapp # providing the container with the name as webapp
                image: richardchesterwood/k8s-fleetman-webapp-angular:release0-5
                # defining the image for the container in this particular case out in here
      
      --- # defining the document separator to define another POD for the queueapp
      
      apiVersion: v1 # defining the apiVersion as v1 in  here as we are defining the POD in this case 
      kind: Pod # defining the type of object in this case is of POD
      metadata: # defining the metadata for the POD in this case 
        name: queueapp # defining the name for the POD in here
        labels: # defining the label in the format as Key-value pair
          app: queueapp # defining the name as the key value pair in this case
      spec: # defining the specification for the POD in this case
        containers: # defining the container in this case which will be spunned because of containers
          - name: queueapp # defining the name for the container in this case 
            image: richardchesterwood/k8s-fleetman-queue:release1 
            # defining the image for the container out in here
            
  ```   

- if the `POD` defined under the `ReplicaSet` crashed then `ReplicaSet` will spun a `New POD` and make it running in that case here

# How to Apply the ReplicaSet to the Kubernetes Cluster

