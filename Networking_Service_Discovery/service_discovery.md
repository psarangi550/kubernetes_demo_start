# kubernetes Service Discovery 

- here the `objective` is to `exec into the Webapp container POD terminal` and `network across` the `MYSQL DB POD service`

- if we are using the `cygwin terminal` we have `fix` these issue of `interactive teletype Terminal` using something called as `winpty`

- if we are using the `MacOS/Powershell/Linux` terminal then we can use `kubectl exec -it <POD> -- <command>` directly without using the `winpty`  , but if we are using the `cfygwin terminal` then we have to use the command as `winpty kubectl exec -it <POD> -- <command>`

### How the Networking mechanism work in Kubernetes 

- we want to `access` the `remote Database Service` i.e `MYSQL DB POD service` from the `Webapp container POD terminal`

- **How does `WebApp Container POD` communicate with the `kube-dns Service`  ? How does `WebApp Container POD` know where to find the `kube-dns service` ?**

  - the answer is `piece of automatic configuration` `kubernetes` been `doing` `behind the scene` for us with `all the container` inside the `kubernetes system`
  
  - currently we are inside the `webapp container POD` by using the below command but this `piece of automatic configuration` will be applied `all the container` inside the `kubernetes system`
    
    ```bash
        kubectl exec -it webapp-7f58455867-k2sbd -- sh
        # accessing the terminal of webapp-container POD using the kubectl exec command
        # hence how we have the access to the terminal
        $ # here is the terminal the been accessed 
    ``` 
  
  - `kubernetes` will `do` `automatically` the `management of all the container` and `automatically configure the DNS system` on the `container`
  
  - we can see that using by `kubectl exec -it webapp-7f58455867-k2sbd -- sh` by going to the `webapp-container POD` terminal and perform `cat /etc/resolve.conf`
  
  - we can do that as below

    ```bash
        kubectl exec -it webapp-7f58455867-k2sbd -- sh
        # accessing the terminal of webapp-container POD using the kubectl exec command
        # hence how we have the access to the terminal now
        $ # here the terminal being attached
        
        $ cat /etc/resolve.conf
        # the output will be as below 
        # imp :- 
        # this is the file that configure how the DNS name resolution going to happen or work which is been set by kubernetes for all the container by default
        nameserver 10.96.0.10
        search default.svc.cluster.local svc.cluster.local cluster.local localdomain
        options ndots:5

        #imp:-
        
        # here we are interested in the first line of the output of `cat /etc/resolve.conf`  i.e `nameserver 10.96.0.10`
        
        # when we make a reference to domain name from the Webapp Container POD terminal or  code then it will come to the /etc/resolve.conf fetch the nameserver as the kube-dns service and ask for the Particular Service based on the service name and kube-dns then will provide the corresponding IP address for the domain service we want to reach

        # now if we want to fetch all the kubernetes object inside the kube-system namespace then we can use the command as below 
        kubectl get all -n kube-system   
        # here we can see all the kubernetes object inside the kube-system namespace
        NAME                                   READY   STATUS    RESTARTS      AGE
        pod/coredns-5dd5756b68-zll6f           1/1     Running   1 (12h ago)   14h
        pod/etcd-minikube                      1/1     Running   3 (12h ago)   14h
        pod/kube-apiserver-minikube            1/1     Running   3 (12h ago)   14h
        pod/kube-controller-manager-minikube   1/1     Running   3 (12h ago)   14h
        pod/kube-proxy-ch2kf                   1/1     Running   1 (12h ago)   14h
        pod/kube-scheduler-minikube            1/1     Running   3 (12h ago)   14h
        pod/storage-provisioner                1/1     Running   3 (12h ago)   14h

        NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
        service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   14h # here we can see the kube-dns Service which has the same IP as the name server /etc/resolve.conf , hence when we provide the domain name to the Webapp container POD then it will contact the nameserver i.e kube-dns service in turn to fetch the IP Adreess of the Service

        NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
        daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   14h

        NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/coredns   1/1     1            1           14h

        NAME                                 DESIRED   CURRENT   READY   AGE
        replicaset.apps/coredns-5dd5756b68   1         1         1       14h

    
    ```

- here the `webapp container POD` will try to make a connection with the `remote MYSQL DB POD Service` by using the `Service Name as the domain inside the DB URI` and then the request will come to the `/etc/resolve.conf` as the `container does not know the domain name` , which will forward that request to the `kube-dns Service` , which will fetch the `Service IP` based on the `Service name which provided as key`

- we can use the `nslookup` command to see whrther we can access `One POD Service` from `another POD Terminal` as below 

    ```bash
        nslookup fleetman-database
        # if we are using the nslookup com mand to see the DNS resolution as below 
        # the output will be as below
        Name:      fleetman-database
        Address 1: 10.104.120.205 fleetman-database.default.svc.cluster.local

        # but now if wewant to see whether thats the IP of the mysql DB service or not then we can see that in the default namespace as 
        kubnectl get all 
        # fetching all the  kubernetes object inside the kubernetes cluster in default namespace
        NAME                          READY   STATUS    RESTARTS      AGE
        pod/mysql                     1/1     Running   0             11h
        pod/queueapp                  1/1     Running   1 (12h ago)   14h
        pod/webapp-7f58455867-k2sbd   1/1     Running   1 (12h ago)   14h
        pod/webapp-7f58455867-n4smg   1/1     Running   1 (12h ago)   14h

        NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-database   ClusterIP   10.104.120.205   <none>        3306/TCP         11h # here we can match that the IP address is exactly same what we  accessed inside the webapp container kubectl exec terminal
        service/fleetman-queueapp   NodePort    10.110.156.33    <none>        8161:30010/TCP   14h
        service/fleetman-webapp     NodePort    10.109.54.70     <none>        80:30080/TCP     14h
        service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          14h

        NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/webapp   2/2     2            2           14h

        NAME                                DESIRED   CURRENT   READY   AGE
        replicaset.apps/webapp-7f58455867   2         2         2       14h

    
    ```

- now if we wanr 
