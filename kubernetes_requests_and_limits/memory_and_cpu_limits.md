# Memory and CPU Limits in kubernetes

- in the preveious topic we discussed about the `resources requests` 

- the `request` is really just a `Hint` that being used by the `kubernets cluster scheduler` to `make intelligent decession` on `which node the POD would be best deployed to` , `is there sufficient resource avaialble in the Node to deploy the POD`

- `resource request` is the `hint` for the `kubernetes cluster scheduler`

- its just a `guess by US the developer` what we think `the POD container` going to need `in term of resource` , here it does not `gurantee that POD need exactly requested amount of resource` or `resource requester will not also limit anything if something goes wrong`

- the `resource request` will be very important to understand, which will be used `when we are looking into the Horizonal POD autoscaling (HPA)`

- **Resource (Memory/CPU) limits in kubernetes** 

- although `resources limits` looks very similar `it will be very different concept` from the `resources requests`

- we can define the `limits` optionally inside the `resources` and exactly on the same label as `requests` in the definition yaml file inside the `container section`

- similar to the `resources request` , in `resources limits` we can define the `memory Limits` and `CPU limits`

- we can define `memory and CPU` same as we defined for the `resource requests` i.e `memory` will be in term of the `MegaBytes and GigaBytes` and `CPU` will be in term as `millicore or cores` 

- we can define the `workloads` definition as `resource-testing.yml` as below

    
    ```yaml
        resource-testing.yml
        ====================
        apiVersion: apps/v1 # here the apiVersion is of apps/v1 as Deployment belong to group belong to apps
        kind: Deployment # defining the kubernetes object as Deployment 
        metadata: # specifying the name for the Deployment
            name: queueapp
        spec: # defining the specification for the Deployment
            selector: # defining the selector for the POD labels
                matchLabels: 
                    app: queueapp
            replicas: 2 # defining the replicas as 2 for the Deployment for POD management
            template: # defining the template for the POD definition
                metadata: # defining the labels for the POD 
                    labels:
                        app: queueapp
                spec: # define the definition for the POD container
                    containers: # defining the container with the name
                        - name: queueapp # name of the container
                          image: richardchesterwood/k8s-fleetman-queue:release2 # image for the container
                          resources: # resources defined for the container for hinting the kubernetes cluster scheduler
                            requests: # requesting for the resources such as Memory and CPU
                                memory: 300Mi # requesting for the memory of 300MegaBytes allocated for the POD in the cluster 
                                cpu: 100m # requesting for the cpu of 100millicores allocated for the POD in the cluster
                            limits: # here defining the limits to limit the POD container usage
                                memory: 500Mi # requesting for the memory of 500MegaBytes to be max for the POD container 
                                cpu: 200m # requesting for the CPU of 200milliCores to be max for the POD container
    
    ```

- **What does resource limit means in Kubernetes**

- the `resources limits` such as `resources memory limit` and `resources cpu limit` work very differently

- **resources Memory limit**
  
  - if the `actual memory` used by `POD container` at the `run time` exceed the `resources memory limit` for any reason then `POD container` will be `killed and restarted`
  
  - the `POD` will remain be there , but the `container` will therefore `killed and restarted`
  
  - the `POD itself will not killed will be remain there` , but the `container` inside the `POD` will be `killed and restarted` by the `POD`
  
  - for the `memory` if we set the `resources memory limit` and the `limit` get exceeded for any reason then the `POD` will going to `reschedule` the corresponding `container to be killed and restarted`
  
- **resources CPU limit**

  - if the `actual CPU usage` used by the `POD container` at the `run time` exceed the `resources cpu limit` for any reason then the `POD container` will not be `killed` rather it will be `clamped` with the `CPU limit` that been defined for it
  
  - in this case the `POD` and `POD container` will `remain be there running` but `clamped/fixed` to the `defined CPU` that is defined in the `limits` will `not spill over`
  
  - in `kubernetes` this is called as `throttling` 
  
  - in this case the `POD` and `POD container` will `remain be there running` we `won't see a container restart` in this case
  

- the `difference` between the `resources requests` and `resources limits` that `resources limits`  have dramatic effect on the `POD container`

- the `limits` values are `actual used in practise` while we are using the `POD container` in this case


- **Example usage of Limits**

- an example usage of `limits` being , we are deploying a `POD container` to the `production cluster` and `you know through testing` that `there is a bug in the container` which is causing the `memory leak`

- this memory lick been happening lets suppose in `once in every six month`

- in this case the `right thingto do` is to `fix the memory leak problem`

- but in a real world `fixing that memory leak` is `difficult` and `can take weeks and weeks` in order to fix , if the `POD gets down` then the `entire` cluster will go down as well

- rather we can go for the `providing extra memory limits` keeping the `memory leak case` in mind which the container `will not be using but it will  fail safe approach`

- when the `memory leak` happends and `memory exceed the specified extra limit` then `POD container` will be `killed` and `POD` will `reschedule` the `POD container` again as the `POD will remain be there` , which can `prevent the memory leak` for `next 6 month`\

- uif we are `not mentioning the limit` in that case `it will bring down` the `entire prodduction cluster`  which is not ok

- in a `microservice architecture` in the `kubernetes cluster` we `should not be worried` about `loosing container`  as it will be going to `reschedule and restart the container` by the `POD` with the `backed up data`

- there are other scenarion also we can use the `memory and CPU limits`

- **Goal of resources limits**

- the `main goal` of the `resource limit` being `protect the kubernetes cluster health` , if there is `misbehaving POD container` and the `POD container` have to `pay for it` if it is misbehaving , by `making the CPU limited and fixed or killing and restarting the POD container in case of memory leak`

- **Demo for the reource limits**
  
  - here we are actually not sure the `Actual memory and CPU` used by the `queueapp` `POD container`
  
  - the only rule for the `resources limits` are it should be atleast have `same values` as the `resources request`
  
  - i can't set the `resources memory limits` as `300Mi` where the `resource requests` will be for the `200Mi` , it does not make sense which will be silly
  
  - we can have `resources limits`  should be atleast have `same values` as the `resources request`  or we can have the `head room` for the `resources limits value` 
  
  - we can mention as `300Mi` for the `resource requests for memory` and if it reaches the `500Mi` which defined as the `resources limits` , then that means it is doing something wrong , hence need to be `killed and restarted`
  
  - on the `similar approach` we can derfine the `resources requests CPU` and `resources limits CPU` as well
  
  - now if we define like this as below then we can see the output as below 
    
    ```yaml
        resource-testing.yml
        ====================
        apiVersion: apps/v1 # here the apiVersion is of apps/v1 as Deployment belong to group belong to apps
        kind: Deployment # defining the kubernetes object as Deployment 
        metadata: # specifying the name for the Deployment
            name: queueapp
        spec: # defining the specification for the Deployment
            selector: # defining the selector for the POD labels
                matchLabels: 
                    app: queueapp
            replicas: 2 # defining the replicas as 2 for the Deployment for POD management
            template: # defining the template for the POD definition
                metadata: # defining the labels for the POD 
                    labels:
                        app: queueapp
                spec: # define the definition for the POD container
                    containers: # defining the container with the name
                        - name: queueapp # name of the container
                          image: richardchesterwood/k8s-fleetman-queue:release2 # image for the container
                          resources: # resources defined for the container for hinting the kubernetes cluster scheduler
                            requests: # requesting for the resources such as Memory and CPU
                                memory: 300Mi # requesting for the memory of 300MegaBytes allocated for the POD in the cluster 
                                cpu: 100m # requesting for the cpu of 100millicores allocated for the POD in the cluster
                            limits: # here defining the limits to limit the POD container usage
                                memory: 500Mi # requesting for the memory of 500MegaBytes to be max for the POD container 
                                cpu: 200m # requesting for the CPU of 200milliCores to be max for the POD container
    
    
    ```

- now when we can `deploy these changes` by applying the `definition file` as below

    
    ```bash
        kubectl apply -f resource-testing.yml
        # deploying the changes by applying the change in thisc ase out in here
        # the output will be as below 
        deployment.apps/queueapp created

    ```

- now we can see the `minikube node` as for the `resources requests` and `requests limit` utlization then we can see as 

    
    ```bash   
        
        kubectl get nodes
        # fetching all the nodes inside the minikube cluster as below 
        # the output in this case will be as below
        NAME       STATUS   ROLES           AGE   VERSION
        minikube   Ready    control-plane   21h   v1.28.3

        # now we arew checking the nodesin detail by describing it then we can do as below 
        kubectl describe node minikube
        # describing the nodes int his case
        Name:               minikube
        Roles:              control-plane
        Labels:             beta.kubernetes.io/arch=amd64
                            beta.kubernetes.io/os=linux
                            kubernetes.io/arch=amd64
                            kubernetes.io/hostname=minikube
                            kubernetes.io/os=linux
                            minikube.k8s.io/commit=8220a6eb95f0a4d75f7f2d7b14cef975f050512d
                            minikube.k8s.io/name=minikube
                            minikube.k8s.io/primary=true
                            minikube.k8s.io/updated_at=2024_01_31T08_21_43_0700
                            minikube.k8s.io/version=v1.32.0
                            node-role.kubernetes.io/control-plane=
                            node.kubernetes.io/exclude-from-external-load-balancers=
        Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/cri-dockerd.sock
                            node.alpha.kubernetes.io/ttl: 0
                            volumes.kubernetes.io/controller-managed-attach-detach: true
        CreationTimestamp:  Wed, 31 Jan 2024 08:21:39 +0530
        Taints:             <none>
        Unschedulable:      false
        Lease:
        HolderIdentity:  minikube
        AcquireTime:     <unset>
        RenewTime:       Thu, 01 Feb 2024 06:17:30 +0530
        Conditions:
        Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
        ----             ------  -----------------                 ------------------                ------                       -------
        MemoryPressure   False   Thu, 01 Feb 2024 06:15:38 +0530   Wed, 31 Jan 2024 08:21:38 +0530   KubeletHasSufficientMemory   kubelet has sufficient memory available
        DiskPressure     False   Thu, 01 Feb 2024 06:15:38 +0530   Wed, 31 Jan 2024 08:21:38 +0530   KubeletHasNoDiskPressure     kubelet has no disk pressure
        PIDPressure      False   Thu, 01 Feb 2024 06:15:38 +0530   Wed, 31 Jan 2024 08:21:38 +0530   KubeletHasSufficientPID      kubelet has sufficient PID available
        Ready            True    Thu, 01 Feb 2024 06:15:38 +0530   Wed, 31 Jan 2024 08:21:40 +0530   KubeletReady                 kubelet is posting ready status
        Addresses:
        InternalIP:  192.168.49.2
        Hostname:    minikube
        Capacity:
        cpu:                16 # capacity of CPU there in the cluster
        ephemeral-storage:  153187824Ki 
        hugepages-1Gi:      0
        hugepages-2Mi:      0
        memory:             32821852Ki # capacity of the Memory that been there in cluster
        pods:               110
        Allocatable:
        cpu:                16
        ephemeral-storage:  153187824Ki # capacity of CPU that can be allocated to the cluster
        hugepages-1Gi:      0
        hugepages-2Mi:      0
        memory:             32821852Ki # capacity of the Memory that can be allocated to the cluster
        pods:               110
        System Info:
        Machine ID:                 29c397f9fbd44ab390fb3c41ca3a010c
        System UUID:                d1717e29-c121-4ba3-8bc2-f2f826b3b1ad
        Boot ID:                    c2edc6e7-9ac4-4439-8f55-3e6f2fe58a28
        Kernel Version:             6.5.0-15-generic
        OS Image:                   Ubuntu 22.04.3 LTS
        Operating System:           linux
        Architecture:               amd64
        Container Runtime Version:  docker://24.0.7
        Kubelet Version:            v1.28.3
        Kube-Proxy Version:         v1.28.3
        PodCIDR:                      10.244.0.0/24
        PodCIDRs:                     10.244.0.0/24
        Non-terminated Pods:          (8 in total)
        Namespace                   Name                                CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
        ---------                   ----                                ------------  ----------  ---------------  -------------  ---
        default                     queueapp-c85b85867-b8r5d            100m (0%)     200m (1%)   300Mi (0%)       500Mi (1%)     3m42s # here we can see the Memory/CPU Request and Memory/CPU Limit as well in this case for the POD
        kube-system                 coredns-5dd5756b68-sg2w2            100m (0%)     0 (0%)      70Mi (0%)        170Mi (0%)     21h
        kube-system                 etcd-minikube                       100m (0%)     0 (0%)      100Mi (0%)       0 (0%)         21h
        kube-system                 kube-apiserver-minikube             250m (1%)     0 (0%)      0 (0%)           0 (0%)         21h
        kube-system                 kube-controller-manager-minikube    200m (1%)     0 (0%)      0 (0%)           0 (0%)         21h
        kube-system                 kube-proxy-7gd5p                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         21h
        kube-system                 kube-scheduler-minikube             100m (0%)     0 (0%)      0 (0%)           0 (0%)         21h
        kube-system                 storage-provisioner                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         21h
        Allocated resources:
        (Total limits may be over 100 percent, i.e., overcommitted.)
        Resource           Requests    Limits
        --------           --------    ------
        cpu                850m (5%)   200m (1%)
        memory             470Mi (1%)  670Mi (2%)
        ephemeral-storage  0 (0%)      0 (0%)
        hugepages-1Gi      0 (0%)      0 (0%)
        hugepages-2Mi      0 (0%)      0 (0%)
        Events:              <none>

    
    
    ```

- if we want to demo `what happen` when we `go above the memory and CPU limit` , but which is not easy to demo

- here the `queueapp` been running on the `Apache Active MQ` , we are not certain `what sort of memory and CPU` it uses , we will looking into the `profiling of memory and CPU later` 

- here for the demo , we can demo, we can certainly `reduced the Memory and CPU resources requests and resource limits`  down to `6Mi` which is `value allowed now`

- for the `Memory limit` we can specify the `lowest value` as the `6Mi` in this case

- hence we can mention that in the `resource_testing.yml` as below 

    ```yaml
        
        resource_testing.yml
        ====================
        apiVersion: apps/v1 # here the apiVersion is of apps/v1 as Deployment belong to group belong to apps
        kind: Deployment # defining the kubernetes object as Deployment 
        metadata: # specifying the name for the Deployment
            name: queueapp
        spec: # defining the specification for the Deployment
            selector: # defining the selector for the POD labels
                matchLabels: 
                    app: queueapp
            replicas: 2 # defining the replicas as 2 for the Deployment for POD management
            template: # defining the template for the POD definition
                metadata: # defining the labels for the POD 
                    labels:
                        app: queueapp
                spec: # define the definition for the POD container
                    containers: # defining the container with the name
                        - name: queueapp # name of the container
                          image: richardchesterwood/k8s-fleetman-queue:release2 # image for the container
                          resources: # resources defined for the container for hinting the kubernetes cluster scheduler
                            requests: # requesting for the resources such as Memory and CPU
                                memory: 6Mi # requesting for the memory of 6MegaBytes allocated for the POD in the cluster which is the lowest
                                cpu: 100m # requesting for the cpu of 100millicores allocated for the POD in the cluster
                            limits: # here defining the limits to limit the POD container usage
                                memory: 6Mi # requesting for the memory of 6MegaBytes to be max for the POD container  which is the lowest
                                cpu: 100m # requesting for the CPU of 100milliCores to be max for the POD container
    
    
    ```

- here now we are `deploying to the cluster` by `applying the changes` then we can do as below 

    
    ```bash
        kubectl apply -f resource_testing.yml
        # deploying the changes by applying the change in thisc ase out in here
        # the output will be as below 
        deployment.apps/queueapp configured
    
        # now here we can see the old POD getting terminated and new POD getting created by replica-set as it is a Deployment
        # but the old replica set will not be remove but sitting there idle
        # if we want we can make the old replica-set work as well using the command as below
        # kubectl rollout history deployment <deployment-name>
        # kubectl rollout undo deployment <deployment-name> --to-revision=<revision-number>
        # here in this case only the deployment will be rollback but the definition yaml will be having the mis match hence make sure change that as well

        # now when we are doing the kubectl get all then we can see that POD getting restarted
        # if we goto the describe the POD then we can see some important points
        kubectl get all
        # checking all the kubernetes object inside the default namespace
        NAME                           READY   STATUS             RESTARTS      AGE
        pod/queueapp-78c5f5b94-x4gn2   0/1     CrashLoopBackOff   3 (28s ago)   73s

        NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
        service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   22h

        NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/queueapp   0/1     1            0           42m

        NAME                                  DESIRED   CURRENT   READY   AGE
        replicaset.apps/queueapp-6bf5b4bb64   0         0         0       39m
        replicaset.apps/queueapp-78c5f5b94    1         1         0       73s
        replicaset.apps/queueapp-c85b85867    0         0         0       27m
        replicaset.apps/queueapp-f8ddcf484    0         0         0       42m

        # we might sometimes see thye POD status as OOMKilled which stands for Out of memory killed

        # here we can see the POD status as CrashLoopBackOff
        # CrashLoopBackOff means the container inside the POD having some problem not at the happy state
        # when the containerget crashed then the POD will not immediately restart it , but rather than providing time to restart it 
        # hence we are seeig the status as CrashLoopBackOff

        # sometime we can see the status as Running as well
        # but after it being in Running state and then become CrashLoopBackOff
        # but immediately after running we can see the status as OOMKilled/CrashLoopBackOff when we do a kubectl get all next 
        # this is because the POD might get start with 6Mi memory later immediately realises that it has less memory hence restart it 


        # here we can also see the RESTART as 3 in this case that means POS tried 3 times setting the POD container

        # if we see the PODs description then we can see the below info
        kubectl describe pod/queueapp-78c5f5b94-x4gn2
        # the output will be as below
        Name:             queueapp-78c5f5b94-x4gn2
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Thu, 01 Feb 2024 06:40:01 +0530
        Labels:           app=queueapp
                        pod-template-hash=78c5f5b94
        Annotations:      <none>
        Status:           Running
        IP:               10.244.0.6
        IPs:
        IP:           10.244.0.6
        Controlled By:  ReplicaSet/queueapp-78c5f5b94
        Containers:
        queueapp:
            Container ID:   docker://b6b46185e1273b90e605cb8503abbc75f8fd9e8d394606ecda7a80de6c287db4
            Image:          richardchesterwood/k8s-fleetman-queue:release2
            Image ID:       docker-pullable://richardchesterwood/k8s-fleetman-queue@sha256:f7778362d6144a3c58bf2e34eb56c9c5bb038f150eb8c9599ca61862e4e64937
            Port:           <none>
            Host Port:      <none>
            State:          Waiting # here we can see the current status of the POD been waiting not been terminated or Killed as discussed
            Reason:       CrashLoopBackOff # the reason for the POD on waiting status
            Last State:     Terminated # here defining the last state of the POD
            Reason:       OOMKilled # here we can see the output as OOMKilled which means that Outof memory issue
            Exit Code:    137
            Started:      Thu, 01 Feb 2024 06:45:49 +0530
            Finished:     Thu, 01 Feb 2024 06:45:49 +0530
            Ready:          False
            Restart Count:  6
            Limits:
            cpu:     200m
            memory:  6Mi
            Requests:
            cpu:        100m
            memory:     6Mi
            Environment:  <none>
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-hfbvj (ro)
        Conditions:
        Type              Status
        Initialized       True 
        Ready             False 
        ContainersReady   False 
        PodScheduled      True 
        Volumes:
        kube-api-access-hfbvj:
            Type:                    Projected (a volume that contains injected data from multiple sources)
            TokenExpirationSeconds:  3607
            ConfigMapName:           kube-root-ca.crt
            ConfigMapOptional:       <nil>
            DownwardAPI:             true
        QoS Class:                   Burstable
        Node-Selectors:              <none>
        Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                    node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
        Events:
        Type     Reason     Age                     From               Message
        ----     ------     ----                    ----               -------
        Normal   Scheduled  8m50s                   default-scheduler  Successfully assigned default/queueapp-78c5f5b94-x4gn2 to minikube
        Normal   Pulled     7m23s (x5 over 8m49s)   kubelet            Container image "richardchesterwood/k8s-fleetman-queue:release2" already present on machine
        Normal   Created    7m23s (x5 over 8m49s)   kubelet            Created container queueapp
        Normal   Started    7m23s (x5 over 8m49s)   kubelet            Started container queueapp 
        Warning  BackOff    3m40s (x26 over 8m47s)  kubelet            Back-off restarting failed container queueapp in pod  #not providing much info
        queueapp-78c5f5b94-x4gn2_default(dbadc559-e3fe-41df-b85f-df3c0d150a36)
    
    ```

- we can see the `POD will remain be there` even though the `container inside it being killed and restarted`

- in recomendation , it is `optional` to `put resource limits` , but we must put the `resource request` in order to instruct the `kubernetes scheduler` to schedule `POD intelligently onto the nodes` for a `good cluster` 


- **How kubernetes manage to kill the container if resource memory exceed and CPU limit if resource CPU limit exceed**
  
  - its not actually the `feature of kubernetes` to `kill the container if resource memory exceed `and `limit or clamp CPU`  if `resource CPU limit exceed`
  
  - not its the `feature of docker` which works `behind the scene` to `kill the container if resource memory exceed `and `limit CPU` if `resource CPU limit exceed`
  
  - `docker` behind the scene use a `linux native feature` called the `cgroups` which will `restrict the CPU/memory/disk/network` for the `docker process`
  
  - this `cgroups` been defined in `linux` in `2006` , but get into the `linux kernal` on `2008`  developed by `google` 
  
  - `docker` preveiosuly using the `cgroups` , but not sure what it `use at latest` 

- we will be using the `limit` while using the `Quality of Service or qos`