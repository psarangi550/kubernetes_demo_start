# Kubernetes Fully Qualified Domain Name

- in the `Service Discovery` chapter , we get to know that `we can discover any particular service` by the `POD Service name in the kubernetes cluster`

- if we are going to the `WebApp container POD terminal` as below 

    
    ```bash
        kubectl get all 
        # fetching all the kubernetes object in the default namespace
        # the output will be as below
        NAME                          READY   STATUS    RESTARTS   AGE
        pod/mysql                     1/1     Running   0          25m
        pod/queueapp                  1/1     Running   0          25m
        pod/webapp-6d696dd977-s4r2g   1/1     Running   0          25m
        pod/webapp-6d696dd977-t2d2j   1/1     Running   0          25m

        NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-database   ClusterIP   10.100.139.182   <none>        3306/TCP         25m
        service/fleetman-queueapp   NodePort    10.109.57.24     <none>        8161:30010/TCP   25m
        service/fleetman-webapp     NodePort    10.99.238.98     <none>        80:30080/TCP     25m
        service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          27m

        NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/webapp   2/2     2            2           25m

        NAME                                DESIRED   CURRENT   READY   AGE
        replicaset.apps/webapp-6d696dd977   2         2         2       25m
    
        # now if we access one of the Web-container POD terminal by using the kubectl exec command as below 
        kubectl exec -it webapp-6d696dd977-s4r2g -- sh
        # accessing the sh terminal of the WebApp container POD
        # it will associate it with a TTY (Interactive Teletype terminal)
        $ # attacheing the terminal 

        $ nslookup fleetman-database
        # using the nslookup command accessing the Remote kubernetes MYSQL DB POD service 
        # below will be the output 
        nslookup: can\'t resolve '(null)': Name does not resolve # this does not means the error as we are getting the IP address on return 

        Name:      fleetman-database
        Address 1: 10.100.139.182 fleetman-database.default.svc.cluster.local # if we check the kubectl get all this will be same internal IP for the kubernetes service as fleetman-database
        

    ```

- here the `Kubernetes POD service fleetman-database` is `not register` as `fleetman-database` under the `kube-dns Service` under `kube-system namespace`

- it is register on the `fleetman-database.default.svc.cluster.local` as the `fully qualified domain name(FQDN)` under the `kube-dns Service` under `kube-system namespace`

- when we does the `nslookup fleetman-database` under the `Kubernetes webapp container POD terminal` it would not able to fetch the `fleetman-database` inside the  `kube-dns Service` under `kube-system namespace` , hence we are getting the error as `nslookup: can\'t resolve '(null)': Name does not resolve` intially , as its been saved under the `fully qualified name`

- we also see the `/etc/resolve.conf` file which is `responsible file for domain name resolution` , we can see that inside the `Kubernetes webapp container POD terminal` as below 

    ```bash
        # now if we access one of the Web-container POD terminal by using the kubectl exec command as below 
        kubectl exec -it webapp-6d696dd977-s4r2g -- sh
        # accessing the sh terminal of the WebApp container POD
        # it will associate it with a TTY (Interactive Teletype terminal)
        $ # attacheing the terminal 

        #now if we want to fetch the content of the domain resolution file then we can see that as beloiw 
        cat /etc/resolve.conf
        # the output should be as below without the localdomain 
        nameserver 10.96.0.10
        search default.svc.cluster.local svc.cluster.local cluster.local
        options ndots:5

    ```

- when `nslookup fleetman-database` intially not able to fetch the `fleetman-database` inside the `kube-dns Service` , then it will try to `append the search` onto the `service that we are looking for` such as 
  
  - `fleetman-database` # not able to fetch the service as its saved under the fully qualified domain name
  
  - `fleetman-database.default.svc.cluster.local` # here it will try to append the `search` i.e `default.svc.cluster.local` to the `Service` as `fleetman-database.default.svc.cluster.local`  as this `kubernetes service fleetman-database available in defualt namespace` hence we will be getting the output as `fleetman-database.default.svc.cluster.local` which is the `fully qualified domain name` hence it able to fetch the `Service`
  
  - if not fetched from the `kube-dns service ,  as the FQDN does not exist` inside the `default namespace` , then it will try to add the `next search` i.e `svc.cluster.local` and making it as `fleetman-database.svc.cluster.local`
  
  - similarly if  `fleetman-database.svc.cluster.local` does not founf in the `kube-dns service ,  as the FQDN does not exist` , then it will try to append the last one  `search` as `cluster.local` to the `service` making it as `fleetman-database.cluster.local`
  
  - if we have a `kubernetes service` whiuch reside under the `particular namespace` rather then the `default name space` then
    
    - in that case first it will look for the `Service` inside the `kube-dns` service as it does not exist hence `it will throw an error as` `nslookup: can\'t resolve '(null)': Name does not resolve`
    
    - in that case `default.svc.cluster.local` will append to `service` , and as the `service` does not exist in the `default namespace` hence the `<service-name>.default.svc.cluster.local` will also endup getiing the error message as `nslookup: can\'t resolve '(null)': Name does not resolve`
    
    - in the case then the `svc.cluster.local` try to append to the `service` as `<service-name>.svc.cluster.local` which is also not valid hence also we will be getting the error message `nslookup: can\'t resolve '(null)': Name does not resolve`
    
    - but if we provide the `service` as `<sevice-name>.<namespace>` , in that case `<service-name>.<namespace>.svc.cluster.local`  will able tom fetch the `FQDN` inside the `kube-dns service` and we will get the response
    
    - as it fetched from the `FQDN fetched on the last attempt` it will not try to append the `search` to the `<service name>.<namespace>`

    ```bash
        kubectl get all 
        # fetching all the kubernetes object in the default namespace
        # the output will be as below
        NAME                          READY   STATUS    RESTARTS   AGE
        pod/mysql                     1/1     Running   0          25m
        pod/queueapp                  1/1     Running   0          25m
        pod/webapp-6d696dd977-s4r2g   1/1     Running   0          25m
        pod/webapp-6d696dd977-t2d2j   1/1     Running   0          25m

        NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
        service/fleetman-database   ClusterIP   10.100.139.182   <none>        3306/TCP         25m
        service/fleetman-queueapp   NodePort    10.109.57.24     <none>        8161:30010/TCP   25m
        service/fleetman-webapp     NodePort    10.99.238.98     <none>        80:30080/TCP     25m
        service/kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP          27m

        NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/webapp   2/2     2            2           25m

        NAME                                DESIRED   CURRENT   READY   AGE
        replicaset.apps/webapp-6d696dd977   2         2         2       25m
    
        # now if we access one of the Web-container POD terminal by using the kubectl exec command as below 
        kubectl exec -it webapp-6d696dd977-s4r2g -- sh
        # accessing the sh terminal of the WebApp container POD
        # it will associate it with a TTY (Interactive Teletype terminal)
        $ # attacheing the terminal 

        $ nslookup fleetman-database
        # using the nslookup command accessing the Remote kubernetes MYSQL DB POD service 
        # below will be the output 
        nslookup: can\'t resolve '(null)': Name does not resolve # this does not means the error as we are getting the IP address on return 
        # here it first look for the only-service name inside the kube-dns service not able to fetch hence getting the Name does not resolve
        # here on the second attempt fleetman-database service appended with default.svc.cluster.local as `fleetman-database`+ `default.svc.cluster.local` which is the fully qualified name as the service exist in the default namespace
        # hence it got the full FQDN hence it will be able to fetch the service name as key label and corresponding IP for the same 

        Name:      fleetman-database
        Address 1: 10.100.139.182 fleetman-database.default.svc.cluster.local # if we check the kubectl get all this will be same internal IP for the kubernetes service as fleetman-database
        

    ```

- in the `microservice architecture` we have provided `fully qualified name` for the `all kubernetes Service` that we will be using 