# Deploy ActiveMQ as a Pod and Service

- here we want to `deploy another service` called as the `Queue Service` , later we will learn about the `full micro-service architecture` where we will discuss `why the queue service is needed` , but for now consider that developer of the system a `queue is needed `and we need to `deploy the queue`

- we can validate that `queue service` been working properly by visiting the `Admin console`

- here we need the image as `richardchesterwood/k8s-fleetman-queue` and the Tag should be off `release1` , we are using the `queue is fully released system`

- when we deploy the `POD` port `8161` is the `Admin console for the queue sercvice` , where the username and password for the queue being `admin` and `admin`

- but as we know already `we can't open` the `nodePort` as `on port 8161` here , we need to use the `30010` port the `nodePort Service Port`

- here we know that the `nodePort` is using the port between `30000-32767` is the defined port number

- we can define that as below

    
    ```yaml
        pod-queue.yml
        =============
        apiVersion: v1 # defining the apiVersion as v1 over here
        kind: Pod # defining the type of kubernetes object as POD in here
        metadata: # defining the metadata for the pod such as name , label  and namespace
            name: queueapp # name of the POD that we want to spun
            labels: # defining the Pod Label; in key value pair over here
                app: queueapp
                release: "1" 
        spec: # defining the specification for the POD
            containers: # defining the list of container running in the POD
                - name: queueapp # defining the name of the container as queueapp
                  image: richardchesterwood/k8s-fleetman-queue:release1 # defining the container image with the Tag
    
    ```

- we can define the service associating to the queue as below 

    
    ```yaml
        queue-service.yml
        =================
        apiVersion: v1 # defining the apiVersion as v1 in here
        kind: Service # defining the type of kubernetes object as Service
        metadata: # defining all the metadata for the service
            name: fleetman-queueapp # defining the name for the Service out in here
            namespace: default # defining the name space as default over here in the metadata section
        spec: #defining the specification for the kubernetes Service which will attach to the POD
            selector: # defining the selector for the POD as key value pair
                app: queueapp
                release: "1"
            ports: # defining the ports as which port to open
                - name: http # name of the port as  
                  protocol: TCP # defining the protocol as TCP
                  port: 8161 # accepting the traffic on port 8161
                  targetPort: 8161 # forwarding port 8181 to the container port
                  nodePort: 30010 # defining the nodePort as 30010 over here
            type: NodePort # the type of Service is a NodePort Serice which can help in accessing the service outside the kubernetes cluster

    ```

- we we start the `minikube kubernetes local cluster` then we don't have to start the `exsisting servcies and POD` when we start the `minikube cluster` it will detect the state of `PODs and Services` and `restore it to the same status` when we perform the `minikube start command`

- we can put `all the POD definition onto a single file` , there is no definition `How to staructure the yml file` , `we can slice them as we wanted `

-  

- we can run the command to apply the changes for above `yml` file as 

    ```bash
        kubectl apply -f pod-queue.yml
        #deploying the changes using the command as kubectl apply command
        # here we are applying for the POD yml file
        #below will be the outoput in here
        pod/queueapp created

        #now we can perform the same thing do the same thing for # Deploy ActiveMQ as a Pod and Service

- here we want to `deploy another service` called as the `Queue Service` , later we will learn about the `full micro-service architecture` where we will discuss `why the queue service is needed` , but for now consider that developer of the system a `queue is needed `and we need to `deploy the queue`

- we can validate that `queue service` been working properly by visiting the `Admin console`

- here we need the image as `richardchesterwood/k8s-fleetman-queue` and the Tag should be off `release1` , we are using the `queue is fully released system`

- when we deploy the `POD` port `8161` is the `Admin console for the queue sercvice` , where the username and password for the queue being `admin` and `admin`

- but as we know already `we can't open` the `nodePort` as `on port 8161` here , we need to use the `30010` port the `nodePort Service Port`

- here we know that the `nodePort` is using the port between `30000-32767` is the defined port number

- we can define that as below

    
    ```yaml
        pod-queue.yml
        =============
        apiVersion: v1 # defining the apiVersion as v1 over here
        kind: Pod # defining the type of kubernetes object as POD in here
        metadata: # defining the metadata for the pod such as name , label  and namespace
            name: queueapp # name of the POD that we want to spun
            labels: # defining the Pod Label; in key value pair over here
                app: queueapp
                release: "1" 
        spec: # defining the specification for the POD
            containers: # defining the list of container running in the POD
                - name: queueapp # defining the name of the container as queueapp
                  image: richardchesterwood/k8s-fleetman-queue:release1 # defining the container image with the Tag
    
    ```

- we can define the service associating to the queue as below 

    
    ```yaml
        queue-service.yml
        =================
        apiVersion: v1 # defining the apiVersion as v1 in here
        kind: Service # defining the type of kubernetes object as Service
        metadata: # defining all the metadata for the service
            name: fleetman-queueapp # defining the name for the Service out in here
            namespace: default # defining the name space as default over here in the metadata section
        spec: #defining the specification for the kubernetes Service which will attach to the POD
            selector: # defining the selector for the POD as key value pair
                app: queueapp
                release: "1"
            ports: # defining the ports as which port to open
                - name: http # name of the port as  
                  protocol: TCP # defining the protocol as TCP
                  port: 8161 # accepting the traffic on port 8161
                  targetPort: 8161 # forwarding port 8181 to the container port
                  nodePort: 30010 # defining the nodePort as 30010 over here
            type: NodePort # the type of Service is a NodePort Serice which can help in accessing the service outside the kubernetes cluster

    ```

- we we start the `minikube kubernetes local cluster` then we don't have to start the `exsisting servcies and POD` when we start the `minikube cluster` it will detect the state of `PODs and Services` and `restore it to the same status` when we perform the `minikube start command`

- we can put `all the POD definition onto a single file` , there is no definition `How to staructure the yml file` , `we can slice them as we wanted `

-  

- we can run the command to apply the changes for above `yml` file as 

    ```bash
        kubectl apply -f pod-queue.yml
        #deploying the changes using the command as kubectl apply command
        # here we are applying for the POD yml file
        #below will be the outoput in here
        pod/queueapp created

        #now we can perform the same thing do the same thing for the service filoe as well
        # for that we need to run the command as below to deploy the service by applying the changes in this case over here
        kubectl apply -f queue-service.yml
        #below will be the output for the same in this case
        service/fleetman-queueapp created

    
    ```

- if we have multiple `yml file` to `deploy by applying the changes` then `we can perform something as below which will be applied to all the yaml file inside the directory`

- ile` to `deploy by applying the changes` then `we can perform something as below which will be applied to all the yaml file inside the directory`

- we can do that by using the below command 

    ```bash
        kubectl apply -f .
        # this will apply changes to all the yaml file inside the current working directory (CWD)
        # this will show the kubernetes object mertadata name whether c reated/configured/unchanged

    ```

- now if we want to fetch the details all the kubernetes object inside the kubernetes cluster then we can use the command as below 

    ```bash
        kubectl get all
        # fetch the kubectl get all which will display all the kubenetes object status inside the cluster
        # below will be the output for the same
        NAME            READY   STATUS    RESTARTS     AGE
        pod/queueapp    1/1     Running   0            6h2m
        pod/webapp      1/1     Running   5 (8h ago)   3d13h
        pod/webappnew   1/1     Running   5 (8h ago)   3d12h

        NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-queueapp   NodePort    10.100.177.109   <none>        8161:30010/TCP   5h54m
        service/fleetman-webapp     NodePort    10.106.68.128    <none>        80:30080/TCP     21h
        service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          3d15h
    

    ```

- now if we are going to the `minikube ip` and with the port as `30010` we can access the `ActiveMQ admin page` , if we are going to the `manage admin` then by by putting the `username` as `admin` and `password` as `admin` then we can see the `queue` , which will helpful for the `microservice queue diagonostics`

- we can do that as below 

    ```bash
        minikube ip
        # this will provide the minikube cluster ip
        # using which we can get the ip address of the cluster as below
        192.168.49.2
        # then we can see that while accessing the below  address on browser we can visualize the admin console over here
        http://192.168.49.2:30010
        #here we can access the admin console of the minikube kubernetes cluster
        
    ```
