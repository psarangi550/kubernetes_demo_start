# solving the ingressClassName problem in kind kubernetes cluster with metalLB

- when we use the `kubectl get nodes` it will show the `kubernetes version on the cluster` which will be the `response` from `kubelet service in the  masternode of cluster`

- there are `multiple type of configMaps` which been `avaialable` inside the `metalLB` such as below
  
  - `Layer2 configuration` :- here we will be using the `Layer2 configuration configMap` in this case
  
  - `BGP configuration`
  
  - `Advance Configuration`   

- we can also `configure` the `IPAddressPool` and `L2Advertisement` with the help of the `configMaps` as well

- here in this case we can define `configMap` as below in this case

    
    ```yaml
        metallb-configmap.yml
        =====================
        apiVersion : v1 # here the apiversion for the configMap as v1 as it belong to the core group
        kind: ConfigMap # here defining the kubernetes object will be of the type as configMap
        metadata: # defining the name of the configMap as config
            name: config
            namespace: metallb-system # here the namescpace been defined as metallb-system 
        data: # here defining the data where we can define the key and value as env variable or as file name and value 
            config: | # here defining the config as the name of the file that we will be using
                address-pools: # here defining the Details aboubt the IpAddressPool inside the metallb-system namespace
                    - name: default # name of the address pool i,e IPAddressPool as default
                      protocol: layer2 # here defining the protocol as layer2 in this case
                      addresses: # here the address will be based on the IP address kind kubernetes node by using the command as kubectl get node -o wide 
                        - 172.18.0.10-172.18.0.25 # from this IP range the mketalB will assign the external IP to the kubernetes Service as LoadBalancer

    ```

- we can visualize the `address` thaqt we will be using as below 

- we need to see `what is the IP Address of the kind node inside the kind kubernetes cluster`

- for that we can use the command as below in this case

    
    ```bash
        kubectl get nodes -o wide
        # fetching the wider details on the kubernetes kind cluster as below 
        # here we can see the outcome as below 
        NAME                      STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
        mycluster-control-plane   Ready    control-plane   16h   v1.26.0   172.18.0.4    <none>        Ubuntu 22.04.1 LTS   6.5.0-18-generic   containerd://1.6.12
        mycluster-worker          Ready    <none>          16h   v1.26.0   172.18.0.2    <none>        Ubuntu 22.04.1 LTS   6.5.0-18-generic   containerd://1.6.12
        mycluster-worker2         Ready    <none>          16h   v1.26.0   172.18.0.3    <none>        Ubuntu 22.04.1 LTS   6.5.0-18-generic   containerd://1.6.12


        # here we can see the IP of the  kind kubernetes cluster being as 172.18.0.0-172.18.255.255
        # here it is a /16 Network hence we can see the 172.18 Network in here
        # here we want to define for the few chunk from the above kind kubernetes node range
        # hence we can define that as below 
        # 172.18.0.10-172.18.0.25

    ```

- here we can `Deploy the changes to the kubernetes cluster` by `appling the changes onto it` as below 

    ```bash
        kubectl apply -f metallb-configmap.yml
        # here we can use the create command as well 
        kubectl create -f metallb-configmap.yml
        # here we are applying the changes to the definition file to deploy the configMap onto the metallb-system namespace
        # the output in this case will be as below 
        configmap/config created


        # now we can see the POD s and Services running inside the metallb-systenamespace as below
        kubectl get all -n metallb-system
        # here we are using the kubectl get all to fetch all the resource inside the metallb-system namespace
        # the output in this case will be as below 
        NAME                             READY   STATUS    RESTARTS      AGE
        pod/controller-7d678cf54-xvfx9   1/1     Running   2 (13h ago)   18h
        pod/speaker-4cd87                1/1     Running   2 (13h ago)   18h
        pod/speaker-4hws7                1/1     Running   2 (13h ago)   18h
        pod/speaker-qg6bz                1/1     Running   1 (13h ago)   18h

        NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
        service/webhook-service   ClusterIP   10.96.188.166   <none>        443/TCP   18h

        NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
        daemonset.apps/speaker   3         3         3       3            3           kubernetes.io/os=linux   18h

        NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/controller   1/1     1            1           18h

        NAME                                   DESIRED   CURRENT   READY   AGE
        replicaset.apps/controller-7d678cf54   1         1         1       18h


    ```

- we then need to `deploy the ingress controller` to the `kind kubernetes cluster` hence we can use it as below 

- we can use the `nginx kubernetes ingress controller` user manual and deploy it using `Helm` as below [nginx kubernetes ingress controller](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/)

- we can use it as below commands

    ```bash
        
        # this command been provided with --install command if the ingress-nginx namespace not there then it will going to install the chart
        # if the ingress-nginx namespace already present then we can simply upgrade using the helm chart
        # here we are using the helm chart with the option as --create-namespace if the namespace not there it will going to create the namespace
        # else it will reutilized if the namespace already exist
        # if the resources not there it will going to install the resources as the --install been provided
        # else it will going to upgrade the resources if the resources already exists 
        # here the chart name is ingress-nginx
        # the helm charts values.yml been present inside the ingress-nginx folder 
        helm upgrade --install ingress-nginx ingress-nginx \
        --repo https://kubernetes.github.io/ingress-nginx \
        --namespace ingress-nginx --create-namespace

        # here we can use the command as helm repo list  to list out all the repo in the kubernetes cluster
        helm repo list
        # here we can see the helm repo in this case as below 
        # the output will be as below 
        

        # we can also see the chart version using the command as helm list -a
        helm list -A
        # this will provide all the charts available as below from all the namespaces
        # here -A command styates that it belong to all the namespaces
        # here we can see the status of the helm deployment in this case
        NAME         	NAMESPACE    	REVISION	UPDATED                                	STATUS  	CHART              	 APP VERSION
        ingress-nginx	ingress-nginx	 1       	2024-02-23 10:53:03.022473689 +0530 IST	deployed	nginx-ingress-1.1.3	 3.4.3

        # we can see all the kubernetes workload that been defined under the ingress-nginx namespace
        # then here we can use the command as ingress-nginx
        kubectl -n ingress-nginx get all
        # fetching all the kubernetes resouce in ingress-nginx namespace
        NAME                                          READY   STATUS    RESTARTS      AGE
        pod/nginx-ingress-controller-548d4c58-9jz6n   1/1     Running   1 (13h ago)   15h

        NAME                               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
        service/nginx-ingress-controller   LoadBalancer   10.96.145.7   172.18.0.11   80:30502/TCP,443:30565/TCP   15h

        NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/nginx-ingress-controller   1/1     1            1           15h

        NAME                                                DESIRED   CURRENT   READY   AGE
        replicaset.apps/nginx-ingress-controller-548d4c58   1         1         1       15h

    

    ```

- here as we can see the `ingress-nginx` `service` been get the `Service as LoadBalancer` , where the `external IP` been `assinged` by the `metalLB` loadbalancer

- if we don't  have the `metalLB` before the `ingress controller` then we can see the `ingress nginx` service as the `NodePort` service hence we can `uninstall` the `helm chart` and `deploy it` again

- 