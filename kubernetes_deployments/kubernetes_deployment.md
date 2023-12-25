# Deployments in Kubernetes 

- last session we learnt that `How to handle POD using replica-set` , because mopst of the time we will not `Directly deploy the PODs onto the kubernetes cluster`

- as well as the `replica-set` we can use the `Deployment` in order to manage `PODs on the kubernetes cluster` which is also `prefered`

- in kubernetes `A Deployment is basically is more sophesticated form of replica-set` , which is `one more additional feature along with the replica-set`

- with the `Deployment` we got the `automatic rolling updates` with `Zero Down Time`

- we did the `rolling update` with the help of the `changing Service Selector block` based on the `changing the POD labels` which we have to do manually `for zero Downtime upgrade`

- but the `Deployments` are `more elegent` and we can `perform` also the `rollbacks` if `somethign goes wrong`

- if we go to the `Workload API` in [kubernetes 1.28 API one page guilde](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#deployment-v1-apps)

- we are going to see the `Deployment YAML Definition` for the `Deployment` is same as `Replica-Set`

- apart from the `kind: Deployment` rest all the `blocks of the Deployment yml file` is same as replica set

- we can define the `Deployment` as below 

    
    ```yaml
        deployment.yml
        ==============
        apiVersion: apps/v1 # defining the apiVersionb as apps/v1 as it present in the apps group here
        kind: Deployment # defining the kind of kubernetes object as Deployment in this case
        metadata: # defining the name for the deployment
            # Unique key of the Deployment instance
            name: deployment-example
        spec: # defining the specification for the Deployment over here
            # 3 Pods should exist at all times.
            replicas: 3 # defining the number of replicas for the POD being 3
            
            selector: # defining the selector for the PODs to be spunned on
                matchLabels: 
                    app: nginx # defing the key value pair to be matched to the POD label
            
            template: # here we can define  the POD definition over here
                metadata: # defining the POD label which need to be selected
                    labels: # defining the label for the POD in key-value pair
                    # Apply this label to pods and default
                    # the Deployment label selector to this value
                    app: nginx
                
                spec: # defining the spec for the POD here
                    containers: # defining the container name and image for the POD
                        - name: nginx # defining the name of the POD which need to be spunned on
                          image: nginx:1.14 #defining the image for the container inside the POD

    
    ```

- the `yaml definition structure of thye Deployment` is exactly same as the `Structure of the yml for the replica-set` with only difference as `kind: Deployment`

- we can add some `additional field onto the Deployment yml definition` if we want to tune the `rolling updates` for the `Deployment`

- we can resue the `replica-set` YAML definition  over here for the `Deployment`  here as below 

- here we want to deploy the `webapp-container` POD `prototype` with the `tag as release0` over here to the `kubernetes cluster` ,then we want to perform the `Zero Downtime rolling updates for the Deployment` in here

- if we now a `kubectl get all` to fetch all the `kubernetes object inside the kubernetes cluster` then we can see now the
  
  - `replica-set`
  
  - `PODs`
  
  - `Service`  

- if we are trying to `delete the PODs that been tied to the replica set` then it will come back up as it has the `replicas: <no>` defined in its definition 

- now we can remove the `replica-set` that we have created using the command as `kubectl delete rs <replica-set name>` which will terminate the `corresponding POD` that been created through  it  

- we can see the below output in this case out in here 

    ```bash
        kubectl get all
        # fetching all kkubernetes object inside the kubernetes cluster
        NAME                          READY   STATUS    RESTARTS       AGE
        pod/queueapp                  1/1     Running   1 (125m ago)   131m
        pod/webapp-fc94r              1/1     Running   0              123m
        pod/webapp-xwscm              1/1     Running   0              123m

        NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-queueapp   NodePort    10.97.37.84    <none>        8161:30010/TCP   124m
        service/fleetman-webapp     NodePort    10.103.74.42   <none>        80:30080/TCP     130m
        service/kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP          4d21h


        NAME                     DESIRED   CURRENT   READY   AGE
        replicaset.apps/webapp   2         2         2       131m
        

        # we can delete the replica-set over here by using the kubectl delete rs <replica-set name> which will term inate the PODs as well
        kubectl delete rs webapp
        # using this we can remove the replica-set in this case 
        replicaset.apps/webapp  deleted

        # now when we do the kubectl get all to fetch all the kubernetes object inside the cluster then we can see the details as below
        kubectl get all
        
        NAME                          READY   STATUS    RESTARTS       AGE
        pod/queueapp                  1/1     Running   1 (125m ago)   131m

        NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-queueapp   NodePort    10.97.37.84    <none>        8161:30010/TCP   124m
        service/fleetman-webapp     NodePort    10.103.74.42   <none>        80:30080/TCP     130m
        service/kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP          4d21h
    
    ```

- we can define the `Deployment yml definition` for the `webapp` with `release:0` `tag` as below 

    ```yaml
        workloads.yml
        =============
        apiVersion: apps/v1 # defining the apiVersion for the Deployment as it lies under the apps group
        kind: Deployment # defining the kind of object as Deployment over here
        metadata: # defining the metadata sectiuon for Deployment name over here
            name: webapp
        spec: # defining the specification for the Deployment over here 
            replicas: 2 # defining the replicas of the POD we want from the Deployment
            selector: # defining the Selector with key-value pair as app: webapp here
                matchLabels:
                    app: webapp
            template: # defining the POD definition over here with the label info
                metadata: # metadata for the POD defined here
                    labels: # labels of key-value pair to be selected
                        app: webapp

                spec: # specification for the Container defined here
                    contianers: #container for the POD defined here
                        - image: richardchesterwood/k8s-fleetman-webapp-angular:release0 # defining the container image over here
                           name: webapp # defining the container name over here
        
        --- # using the document separator defining the another POD with Deployment

        apiVersion: v1 # defining the POD apiVerison in this case
        kind: Pod # defining the kind of kubernetes object as POD here 
        metadata: # defining thw name and label for the queueapp POD
            name: queueapp
            labels:
                app: queueapp
        spec: # defining the specification for the POD in here
            containers: # defining the container that need to reside inside the POD in kubernetes
                - name: queueapp # defining the containers name in this particular case
                  image: richardchesterwood/k8s-fleetman-queue:release1 # images for the container POD


    ```

- we will also be upgrading the `pod/queueapp` to the `part of replica-set` as well as a part of `Microservice architecture`

- we can apply the changes using the command as `kubectl apply -f <definition file>` as below 


    ```bash
        kubectl apply -f workloads.yaml
        # here we are applying the changes and deployng the Deployment in this case out in here 
        # we can see the below output in this case
        deployment.apps "webapp" created
        pod "queueapp" unchanged

        #now when we try to access the web-browser in the cluster ip of minikube and do a hard refresh we can see the previous version of webapp where only map been showing up
        minikube ip
        # below will eb the output in that case
        192.168.49.2

        # now when we access the web-browser then we can see the output as below 
        http://192.168.49.2:30080
        # we can access the webappwith the map prototype because of the container version inside POD


        #if we want to see the all kubernetes object then we can see the output as below
        kubectl get all
        # fetching all the object inside the kubernetes cluster
        NAME                          READY   STATUS    RESTARTS       AGE
        pod/queueapp                  1/1     Running   1 (149m ago)   155m
        pod/webapp-7f58455867-fc94r   1/1     Running   0              147m
        pod/webapp-7f58455867-xwscm   1/1     Running   0              147m

        NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-queueapp   NodePort    10.97.37.84    <none>        8161:30010/TCP   148m
        service/fleetman-webapp     NodePort    10.103.74.42   <none>        80:30080/TCP     154m
        service/kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP          4d22h

        NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/webapp   2/2     2            2           155m

        NAME                                DESIRED   CURRENT   READY   AGE
        replicaset.apps/webapp-76db76b84f   2         2         2       155m 

    
    ```

- here we can see that `Deployment` will going to create a `Corresponding Replica Set` in this case out in here 

- we can consider the `Deployment` is an `entity` in `Kubernetes` which `manages` the `replica-set` for you

- hence we don't have to deal with the `Deployment` rather than `creating replica-set` directly on the `Kubernetes cluster` which will handle management of the `replica-set`

- when `Deployment` create the `Relica-Set` then it will create the `replica-set` with the `name` as `<Deployment Name>-<random alphanumeric string>` and this `replica-set` will handle the `PODs` hence the name of the became `<Deployment name>-<random alphanumeric string of the replica-set>-<random alphanumeric string>`

- when we are `updating the Deployment` then it will going to `create and destroy new replica-set` which in turn `create and destroy the PODs` and hence in order to manage the `replica-set` individually then we need to define the `replica-set` with `random string with prefix along with Deployment name`

- the `replica-set` also need to `create and destroy the PODs in order to manage` hence we need to `uniquely define that also with the random string as well in that case`

### How to Do a Rolling update in a Deployment with Fiddle around manually updating the POD label and Service Selector

- when we work with the `New container image` for updating the `Application` , when we make `changes` to the `image` inside the `Kubernetes Deployment yaml definition` and `redeploy` the changes then it will `create a new Replica Set` with the `updated image that been provided into it and spun the container as well`

- once the `replica-set` created those `PODs with new image and PODs are ready and avaiable` then `Deployment` will `shutdown the Existing the PODs inside the existing replica-set` and bring the `replicas count to 0 ` , but the `existing replica-set will be there but with no PODs or replicas as 0` , this will be helpful in case we want to `rollback the changes` if the `New PODs not working as expected inside the Deployment` then we can resurect the `old replica-set` with the `number of replicas as 2 back again`

- ![Alt text](image.png) &rarr; ![Alt text](image-1.png)

- but `here in this case if we want to update the webapp-container to the release0-5 tag` then `we can do that by changing the image of the container inside the deployment as below`  

- we can also add the `additional fields` onto the `Deployment Definition yml file` as well

- we can add something as `minReadySeconds`  which been present in `DeploymentSpec` i.e `spec` section inside the `kubernetes API overview under the workload section/Deployment V1`

- `minReadySeconds` is `Minimum number of seconds for which a newly created pod should be ready without any of its container crashing i.e make the PODs avialable from ready state`

- by default the `minReadySeconds` to `0 second` , but we can `make it to 30 seconds` if we want those changes `if we want to see the rolling update better way`

- if we are not setting the `minReadySeconds` then the `new-replica set will created very fastt also the old replica set will be scaled down as well quickly`

- we can visualize the `replica-set getting created underneath each POD will be created (Depending on the number of replicas) but with a minReadySeconds time`

- we can do that by as below 

    
    ```yaml
        workloads.yml
        =============
        apiVersion: apps/v1 # defining the apiVersion for the Deployment as it lies under the apps group
        kind: Deployment # defining the kind of object as Deployment over here
        metadata: # defining the metadata sectiuon for Deployment name over here
            name: webapp
        spec: # defining the specification for the Deployment over here 
            minReadySeconds: 30 # defining the minReadySeconds as 30 sec in this case out in here
            replicas: 2 # defining the replicas of the POD we want from the Deployment
            selector: # defining the Selector with key-value pair as app: webapp here
                matchLabels:
                    app: webapp
            template: # defining the POD definition over here with the label info
                metadata: # metadata for the POD defined here
                    labels: # labels of key-value pair to be selected
                        app: webapp

                spec: # specification for the Container defined here
                    contianers: #container for the POD defined here
                        - image: richardchesterwood/k8s-fleetman-webapp-angular:release0 # defining the container image over here
                           name: webapp # defining the container name over here
        
        --- # using the document separator defining the another POD with Deployment

        apiVersion: v1 # defining the POD apiVerison in this case
        kind: Pod # defining the kind of kubernetes object as POD here 
        metadata: # defining thw name and label for the queueapp POD
            name: queueapp
            labels:
                app: queueapp
        spec: # defining the specification for the POD in here
            containers: # defining the container that need to reside inside the POD in kubernetes
                - name: queueapp # defining the containers name in this particular case
                  image: richardchesterwood/k8s-fleetman-queue:release1 # images for the container POD
    
    ```

- we can apply those changes using the command as `kubectl apply -f <Definition file>` and we can also see the `behaviour` by using the command as `kubectl get all` command constantly and if we `do a hard refresh on the webpage then we can see without zero downtime the application been updated`

- we can do that as below 

    ```bash
        kubectl apply -f workloads.yml
        # applying the changes and Deploying the deployment for the rolling update in this case out in here
        # we can see the below changes
        deployment.apps "webapp" created
        pod "queueapp" unchanged    

        # we need no changes in the services.yml file     


        #now when we do the kubectl get all we can see the below output
        kubectl get all
        #checking the kubernetes object inside the kubernetes cluster
        NAME                          READY   STATUS    RESTARTS       AGE
        pod/queueapp                  1/1     Running   1 (149m ago)   155m
        pod/webapp-7f58455867-fc94r   1/1     Running   0              147m
        pod/webapp-7f58455867-xwscm   1/1     Running   0              147m

        NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-queueapp   NodePort    10.97.37.84    <none>        8161:30010/TCP   148m
        service/fleetman-webapp     NodePort    10.103.74.42   <none>        80:30080/TCP     154m
        service/kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP          4d22h

        NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/webapp   2/2     2            2           155m

        NAME                                DESIRED   CURRENT   READY   AGE
        replicaset.apps/webapp-76db76b84f   2         2         2       155m
        replicaset.apps/webapp-7f58455867   1         1         1       147m
            
        #after 30 seconds
        #now when we do the kubectl get all we can see the below output
        kubectl get all
        #checking the kubernetes object inside the kubernetes cluster
        NAME                          READY   STATUS    RESTARTS       AGE
        pod/queueapp                  1/1     Running   1 (149m ago)   155m
        pod/webapp-7f58455867-fc94r   1/1     Running   0              147m
        pod/webapp-7f58455867-xwscm   1/1     Running   0              147m

        NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-queueapp   NodePort    10.97.37.84    <none>        8161:30010/TCP   148m
        service/fleetman-webapp     NodePort    10.103.74.42   <none>        80:30080/TCP     154m
        service/kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP          4d22h

        NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/webapp   2/2     2            2           155m

        NAME                                DESIRED   CURRENT   READY   AGE
        replicaset.apps/webapp-76db76b84f   1         1         1       155m
        replicaset.apps/webapp-7f58455867   2         2         2       147m
    
        #after 30 seconds
        #now when we do the kubectl get all we can see the below output
        kubectl get all
        #checking the kubernetes object inside the kubernetes cluster
        NAME                          READY   STATUS    RESTARTS       AGE
        pod/queueapp                  1/1     Running   1 (149m ago)   155m
        pod/webapp-7f58455867-fc94r   1/1     Running   0              147m
        pod/webapp-7f58455867-xwscm   1/1     Running   0              147m

        NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-queueapp   NodePort    10.97.37.84    <none>        8161:30010/TCP   148m
        service/fleetman-webapp     NodePort    10.103.74.42   <none>        80:30080/TCP     154m
        service/kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP          4d22h

        NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/webapp   2/2     2            2           155m

        NAME                                DESIRED   CURRENT   READY   AGE
        replicaset.apps/webapp-76db76b84f   0         0         0       155m
        replicaset.apps/webapp-7f58455867   2         2         2       147m

        #NOW WHEN WE ACCESS THE MINIKUBE IP AS  
        minikube ip
        # showing the minikube ip
        # below will eb the output in that case
        192.168.49.2

        # now when we access the web-browser then we can see the output as below 
        http://192.168.49.2:30080
        # we can access the webappwith the map with app menu prototype because of the container version inside POD


    ```
