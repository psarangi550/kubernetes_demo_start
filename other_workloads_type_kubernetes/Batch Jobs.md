# other kubernetes workloads Types

- here we will be looking into the `miscellaneous workload types` that we havenot covered so far

- here we will be looking into `3 topics` as below 
  
  - `Jobs and Cron Jobs`
  
  - `Daemonsets` 
  
  - `StatefuleSets`   
  
- here for most of the `course` we will be looking into the `fleetman microservice application`

- if we goto the `kubernetes manual` on the page as [kubernetes manual](https://kubernetes.io/docs/concepts/workloads) under the `workloads` section we can see that below info inside the `workload management`

- insdie the `workloads` we have the below `kubernetes resources`
  
  - `PODs` :- 
    
    - which is the `center of everthing` which contains/wraps the `web container` or `microservice`
  
  - `Replicasets` :- 
  
    - `Replicasets` also `rarely used directly in kubernetes`  now a days , `Replicaset` allow us to `mention exact number PODs that we want to run of a particular type` that must be running on a `given instance`
                  
    - we can have 2 `webapp` POD running at any given instance of time
  
  - `Deployments` :- 
    
    - A `Deployment` is a `progression` of the `replica-sets` , it is a `replica-set` with `more functionality` 
  
    - A `Deployment` `does the same thing that` the `replica-set` do but with added `extra features`  such as `doing rolling Deployments`
    
  - `ReplicationController` :- 
    
    - `ReplicationController` are `pretty much` redundent now  , `kubernetes manual also say` that `Deployment  that configure the replicasets` are `recomended way` to setup the `Replication`
  
  - here there are `3 workloads` that we have not covered yet , we will be learning where these `workloads type` being useful
  
  - for most of the `kubernetes requirement` , `PODs` and `Deployments`  are the `core for most of the workloads` , below `3 workloads` are just `icing on the cake`
  
  - these `3 workload kubernetes resources` being used for `specific purpose` i.e `specific requirement`
    
  - `Jobs and Cron Jobs`
  
  - `DaemonSets`
  
  - `StatefulSets`      
  
- **Jobs in kubernetes** 
  
  - `Jobs` are very simple on `kubernetes`
  
  - The Idea of `kubernetes jobs` being `Job will create POD` , `kubernetes` will `sure to make sure` the `the POD created and successfully run to completion`
  
  - as we are running the `microservices` , where we are dealing with the `Services` which are mostly `long enough` which will `run forever`
  
  - but we can have `some kind of batch jobs` which will be running `infrequently` such as an `adhoc Report` or etc
  
  - A `Job` `creates` `one or more Pods` and `will continue to retry execution of the PODs until a specified number of them successfully terminate`
  
  - As `POD successfully complete, the Job tracks the successful completions.`
  
  - `When a specified number of successful completions is reached, the task (ie, Job) is complete.`
  
  - `Deleting a Job will clean up the Pods it created`
  
  - `Suspending a Job will delete its active Pods until the Job is resumed again.` 
  
  - we can have the `PODs` which run as below `configuration`

    
    ```yaml
        test-job.yml
        ============
        apiVersion: v1 # here using the apiVersion as v1 as the POD belong to the Core Group
        kind: Pod # kubernetes object of type POD in here
        metadata: # name of the jobs mentioned as test-jobs
            name: test-job
        spec: # defining the specification for the PODs
            containers: # here are the containers details mentioned here
                - name: long-job # name of the Jobs Defined here
                  image: python:rc-slim # image that we want to work on
                  command: ["python"] # here the command we want to use
                  args: ["-c","import time; print('starting'); time.sleep(30) ; print('done')"] # this command will use the args as below  

    ```

- here this `particular POD` will `not going to run` `for very long time` , it will going to `perform the job` and then `stop`

- when we create the `python container with python commandline` once the `image being spunned` and then we will be running the command as `python -c import time; print('starting'); time.sleep(30) ; print('done')` command here

- if we deploy this `raw POD onto the kubernetes cluster` by `applying the changes` then we will  get 

    ```bash
        kubectl apply -f test-job.yml
        # applying the changes onto the test-job.yml
        # here we will get the response as below 
        pod/test-job created

        # we can see the changes for the POD using the command as below 
        watch kubectl get pods
        # using the watch command we can see the POD changes on live
        # if we are watching that then we can see that PODs get completed and then started running again
        
        # imp
        # then we will be having the POD running again after getting completed
        # this is the part of the kubernetes POD that will running the container whether it completed or exited 
        # this might be very helpful for the microserive or webservice for long running task , so that there will be no downtime
        # but if it is batch job we want to stop the restrting of the PODs when the POD execution completed

    
        # Deployment means having multiple replicas and rolling out changes as rolling Deployment

    ```

- in a `POD` we can change the `POD lifecycle` [POD lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)

- if we are going to the `restart policy` , `POD` has the `restartPolicy` which can have the `values` as
  
  - `Always` :- which is the `default value` hence we are seeing the `POD` getting `running` after POD got `completed` or `exited` 
  
  - `onFailure` :- which will allow the `POD` to be `restart and running` after the `got exited due to some error`
  
  - `Never` :- which will allow the `POD` to be not to be `restart and running` after POD got `completed` or `exited` 
    
  
- hence we can set the `values` as below where the `restartPolicy` will be in the `same level` as the `container` inside the `POD definition`
    

    ```yaml
        test-job.yml
        ============
        apiVersion: v1 # here using the apiVersion as v1 as the POD belong to the Core Group
        kind: Pod # kubernetes object of type POD in here
        metadata: # name of the jobs mentioned as test-jobs
            name: test-job
        spec: # defining the specification for the PODs
            containers: # here are the containers details mentioned here
                - name: long-job # name of the Jobs Defined here
                  image: python:rc-slim # image that we want to work on
                  command: ["python"] # here the command we want to use
                  args: ["-c","import time; print('starting'); time.sleep(30) ; print('done')"] # this command will use the args as below 
            restartPolicy: Never # here the restartPolicy means what action kubernetes will take if the PODs successfully completed and exit or exited simply 
            # In Kubernetes, the restartPolicy field specifies what action should be taken by the Kubernetes system if a container in a Pod exits (either successfully or due to an error)


    ```

- if we deploy this `raw POD onto the kubernetes cluster` by `applying the changes` then we will  get 

    ```bash
        
        # here we can delete the old Version of the POD by using the command as below 
        kubectl delete po <pod name>
        # here for example we can mention as below 
        kubectl delete pod test-job
        # deleting the PODs in this case out in here


        kubectl apply -f test-job.yml
        # applying the changes onto the test-job.yml
        # here we will get the response as below 
        pod/test-job created

        # we can see the changes for the POD using the command as below 
        watch kubectl get pods
        # using the watch command we can see the POD changes on live
        # if we are watching that then we can see that PODs get completed and will not going to restart

        # we can also check the logs of the POD by using the command as below 
        kubectl logs test-job
        # fetching the logs of the kubernetes POD
        # here this will be the outcome in this case
        starting
        done

    ```

- when we have the `completed POD` that `POD` will remain be there in the `kubernetes cluster` , because as `system admin` we do want to see the `logs of the completed POD`

- the `default` behavioud of `kubernetes` being it will not going to clean that `POD` even though the `POD` get completed

- there is new feature in `kubernetes` called the `ttl controller` which stands for the `time to live controller` , which `does the garbage collection` , ehich we can configure to find the `finished POD`

- we can configure the `ttlSecondsAfterFinished` for the `cleaning up the finished/completed PODs` 

- but there is one `problem` that we have shown with the `short lived PODs` i.e  `what happens in the POD` `if the jobs goes wrong`

- we can ssimulate that `by deleting the existing POD` and then we will `deploy the changes to the cluster` by `applying changes to the POD definition`

- we need to `think of a way to fail the POD` , in this case we will be `stopping/killing the docker container which been used by minikube VM to spin the POD container` which will make the `POD as failed`

- we can do that by using the below command as 

    ```bash
        minikube docker-env
        # this command will give us the minikube DOCKER variable which been used as the env variable 
        # if we are using this command we can see the output as below
        export DOCKER_TLS_VERIFY="1"
        export DOCKER_HOST="tcp://192.168.49.2:2376"
        export DOCKER_CERT_PATH="/home/pratik/.minikube/certs"
        export MINIKUBE_ACTIVE_DOCKERD="minikube"

        # To point your shell to minikube's docker-daemon, run:
        # eval $(minikube -p minikube docker-env)
 

        # hence when we will be doing the normal dpocker ps we will be not able to see the kubernetes POD container in our system as they are running inside the minikube VM
        # but if we want to see the minikube docker container which is running under the  minikube VM on our host machine then we need to use command as below 
        eval $(minikube -p minikube docker-env)
        # this command will run the set the docker-env for the minikube VM in this case
        
        #now when we run the docker command we can see also the minikube VM docker images which been used for kubernetes as below 
        docker ps
        # checking all the running container here
        # the output will be as below 
        CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS          PORTS     NAMES
        0688e31ad047   psarangi58/ttnot8           "npm --no-update-not…"   43 minutes ago   Up 43 minutes             k8s_nodered_tt-notification-service-nodered-6d4d89fcf5-4h29r_default_bbab5257-9a20-4e78-a25e-b31b60f492dc_28
        afc6dbda65cb   6e38f40d628d                "/storage-provisioner"   6 hours ago      Up 6 hours                k8s_storage-provisioner_storage-provisioner_kube-system_8cb683ca-b509-45c0-837e-f5a4ed0351f5_20
        349c4782b3b7   5aa0bf4798fa                "/usr/bin/dumb-init …"   6 hours ago      Up 6 hours                k8s_controller_ingress-nginx-controller-7c6974c4d8-6tvlz_ingress-nginx_338a93d7-ce9d-4b7b-9609-dc4c11b8de75_11
        3789195fb8c3   422455829732                "docker-entrypoint.s…"   6 hours ago      Up 6 hours                k8s_postgres_tt-notification-service-postgres-6c894b59b9-9779l_default_f297c82d-9335-4ef6-b1ae-f61a0df5f847_3
        0d0c5999da20   e91b9306c922                "/bin/bash -c 'rabbi…"   6 hours ago      Up 6 hours                k8s_rabbitmq_tt-notification-service-rabbitmq-787c47b7b9-nlmq6_default_3d233aa0-226c-4c07-8112-1f04dd59796b_3
        d50223b78831   ead0a4a53df8                "/coredns -conf /etc…"   6 hours ago      Up 6 hours                k8s_coredns_coredns-5dd5756b68-nbwnx_kube-system_62ebd51c-2aa1-4e56-a8ec-046bfc12f5c6_12
        43c11e4f5998   bfc896cf80fb                "/usr/local/bin/kube…"   6 hours ago      Up 6 hours                k8s_kube-proxy_kube-proxy-mgwjp_kube-system_50a9ff37-d757-4e33-a5bd-d9d45a559d24_10
        022d087769d6   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_ingress-nginx-controller-7c6974c4d8-6tvlz_ingress-nginx_338a93d7-ce9d-4b7b-9609-dc4c11b8de75_11
        68de0f41e9a3   1499ed4fbd0a                "docker-entrypoint.s…"   6 hours ago      Up 6 hours                k8s_minikube-ingress-dns_kube-ingress-dns-minikube_kube-system_daf2d399-7574-4e87-b84d-08aed68bbf69_10
        f6db533c0192   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_kube-proxy-mgwjp_kube-system_50a9ff37-d757-4e33-a5bd-d9d45a559d24_10
        5f9f40a64c0e   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_storage-provisioner_kube-system_8cb683ca-b509-45c0-837e-f5a4ed0351f5_10
        61c63c1f1ce9   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_tt-notification-service-nodered-6d4d89fcf5-4h29r_default_bbab5257-9a20-4e78-a25e-b31b60f492dc_3
        baa2ddb46e09   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_tt-notification-service-postgres-6c894b59b9-9779l_default_f297c82d-9335-4ef6-b1ae-f61a0df5f847_3
        d8fdbd3116c2   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_tt-notification-service-rabbitmq-787c47b7b9-nlmq6_default_3d233aa0-226c-4c07-8112-1f04dd59796b_3
        9968bbe0f746   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_coredns-5dd5756b68-nbwnx_kube-system_62ebd51c-2aa1-4e56-a8ec-046bfc12f5c6_12
        e432802b3185   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_kube-ingress-dns-minikube_kube-system_daf2d399-7574-4e87-b84d-08aed68bbf69_10
        0a849392f45c   537434729123                "kube-apiserver --ad…"   6 hours ago      Up 6 hours                k8s_kube-apiserver_kube-apiserver-minikube_kube-system_55b4bbe24dac3803a7379f9ae169d6ba_10
        8304cbaeb39b   6d1b4fd1b182                "kube-scheduler --au…"   6 hours ago      Up 6 hours                k8s_kube-scheduler_kube-scheduler-minikube_kube-system_75ac196d3709dde303d8a81c035c2c28_10
        257b1c60eed4   73deb9a3f702                "etcd --advertise-cl…"   6 hours ago      Up 6 hours                k8s_etcd_etcd-minikube_kube-system_9aac5b5c8815def09a2ef9e37b89da55_10
        c062ffdb04f3   10baa1ca1706                "kube-controller-man…"   6 hours ago      Up 6 hours                k8s_kube-controller-manager_kube-controller-manager-minikube_kube-system_7da72fc2e2cfb27aacf6cffd1c72da00_10
        3d1726e42fb4   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_kube-scheduler-minikube_kube-system_75ac196d3709dde303d8a81c035c2c28_10
        26afb7eb21e0   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_kube-apiserver-minikube_kube-system_55b4bbe24dac3803a7379f9ae169d6ba_10
        6befed94048e   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_etcd-minikube_kube-system_9aac5b5c8815def09a2ef9e37b89da55_10
        44d1946eb7ec   registry.k8s.io/pause:3.9   "/pause"                 6 hours ago      Up 6 hours                k8s_POD_kube-controller-manager-minikube_kube-system_7da72fc2e2cfb27aacf6cffd1c72da00_10

    ```

- we can `deploy the changes by applying the changes to the POD definition` and then when that is `getting spunned` which `will create the container inside the minikube VM which we can now see on the Host System` we can access that `POD` and then we can `remove that Container` in order to `make the POD container` as failed 

- for which we need to perform the below steps 

    
    ```bash
        kubectl delete pod test-job
        # deleting the POD which is already Spunned from the default namespace of the container
        # this will provide the below response
        pod "test-job" deleted

        # now we need to deploy the changes again by using the command as below 
        kubectl apply -f test-job.yml
        # deploying the changes to the cluster by applying the changes
        # which wilol provide the response as below 
        pod/test-job created

        watch kubectl get pods
        # watching the pods as they change the status
        # here we can see the PODs will be running at first
        # but as soon as we stop the container by hooking the minikube docker as well we will see an error from running

        # we can use another terminal session to remove the docker container which the POD want to spin in order to make the POD failed 
        # here we can remove the docker container as below 
        docker container ls | grep python 
        # here we are trying the fetch python container which the test-job POD going to spin 
        # this will provide the ID of the container
        0a4c20354a40   python                      "python -c 'import t…"   7 seconds ago    Up 6 seconds              k8s_long-job_test-job_default_71996ddb-5dc0-4da2-a079-ef96172b5080_

        docker container kill <id of the python container>
        # for example in here
        docker container kill 0a4c20354a40
        # this will remove the python container subsecquently making the POD as failed 
        # here the output will be as below 
        0a4c20354a40

        kubectl get pods
        # fetching the PODs ionside the default namsespace 
        # the output will be as below in this case
        NAME                                                    READY   STATUS    RESTARTS         AGE
        pod/test-job                                            0/1     Error     0                2m2s
        pod/tt-notification-service-nodered-6d4d89fcf5-4h29r    1/1     Running   28 (3h50m ago)   2d22h
        pod/tt-notification-service-postgres-6c894b59b9-9779l   1/1     Running   3 (5h49m ago)    2d22h
        pod/tt-notification-service-rabbitmq-787c47b7b9-nlmq6   1/1     Running   3 (5h49m ago)    2d22h

        NAME                                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                       AGE
        service/kubernetes                         ClusterIP   10.96.0.1        <none>        443/TCP                       2d22h
        service/rabbitmq                           ClusterIP   10.109.184.121   <none>        5672/TCP,15672/TCP,1883/TCP   2d22h
        service/tt-notification-service-nodered    NodePort    10.99.224.32     <none>        1880:30030/TCP                2d22h
        service/tt-notification-service-postgres   ClusterIP   10.100.107.202   <none>        5432/TCP                      2d22h

        NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/tt-notification-service-nodered    1/1     1            1           2d22h
        deployment.apps/tt-notification-service-postgres   1/1     1            1           2d22h
        deployment.apps/tt-notification-service-rabbitmq   1/1     1            1           2d22h

        NAME                                                          DESIRED   CURRENT   READY   AGE
        replicaset.apps/tt-notification-service-nodered-6d4d89fcf5    1         1         1       2d22h
        replicaset.apps/tt-notification-service-postgres-6c894b59b9   1         1         1       2d22h
        replicaset.apps/tt-notification-service-rabbitmq-787c47b7b9   1         1         1       2d22h

        # here we can see the POD going to error state and never going to restart as the restartPolicy we defined as Never 

    ```

- here this way we can `simulate the unexpected reason such as node failure or aqny type of unexpected error` that can lead to `POD container` failure

- in microservice system `its very common` to `remove the nodes from the running cluster` and the `PODs` will be then `resheduled to surviving node`

- but `if we have the batchJobs` which will be `short running Jobs inside the POD container` then there might be the `scenario` that `POD will be failed and finished` will `never going to` be `run again` as we defined the `restartPolicy as Never`

- but we can easily address that using the `kubernetes JOBs`

- `kubernetes JObs` make sure or designed to run the `POD container` will run till `completion`

- we can see the outcome of that by using the command as [Kubernetes JOBS guide](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion) 

- we can write the `Kubernetes Jobs` as below for the `job.yml`

    ```yaml
        job.yml
        =========
        # we cansee the apiVersion for the jobs by using the command as kubectl api-resources -o wide | grep jobs
        # this will provide the jobs detail
        apiVersion: batch/v1
        kind: Job # defining the type as JOb in this case
        metadata: # here this is the name of the Job being defined in here 
            name:  test-job
        spec: # defining the specification for the Jobs in here 
            template: # defining the POD definition over here under the template section 
                spec: # this is the spec of the POD definition
                    containers: # container definition been defined here
                        - name: long-job # name of the container
                          image: python  # image of the container
                          command: ["python"] # command for the container
                          args: ["-c","import time; print('starting')"; time.sleep(30); print('done')"] # args for the command to run
                    restartPolicy: Never # MENTIONING THE pod-lifecycle with restartPolicy as Never
            backoffLimit: 2 # here the backoffLimit once the POD encounter error How many times kubernetes will try to restart and run the POD


    ```

- now we can `deploy the changes to the cluster` by `applying the changes as below` 

    
    ```bash
        
        kubectl delete pod test-job
        # deleting the existing POD in here
        pod "test-job" deleted

        # now we can deploy the changes to the cluster by making  the changes
        kubectl apply -f job.yml
        # deploy the changes to the cluster by applying the changes
        job.batch/test-job created

        kubectl get jobs
        # fetching the job defined inside the cluster 
        # the output will be as below 
        NAME       COMPLETIONS   DURATION   AGE
        test-job   0/1           37s        37s # here we can see the duration of the Kubernetes JOB in this case
        
        kubectl get pods 
        # fetching the POD created by the JOBs as below 
        # the output will be as below
        NAME                                                READY   STATUS    RESTARTS         AGE
        test-job-pklnl                                      1/1     Running   0                8s
        # after 30 sec
        NAME                                                READY   STATUS    RESTARTS         AGE
        test-job-pklnl                                      1/1     completed   0                38s
        
        # we can also see the description of the jobs by using the command as below 
        kubectl describe job test-job 
        # describing the hibs dfefined above as below 
        Name:             test-job
        Namespace:        default
        Selector:         batch.kubernetes.io/controller-uid=6f9a736c-0379-4e27-8534-042490b3f322
        Labels:           batch.kubernetes.io/controller-uid=6f9a736c-0379-4e27-8534-042490b3f322
                        batch.kubernetes.io/job-name=test-job
                        controller-uid=6f9a736c-0379-4e27-8534-042490b3f322
                        job-name=test-job
        Annotations:      <none>
        Parallelism:      1
        Completions:      1
        Completion Mode:  NonIndexed
        Start Time:       Sat, 10 Feb 2024 15:48:15 +0530
        Completed At:     Sat, 10 Feb 2024 15:48:51 +0530
        Duration:         36s
        Pods Statuses:    0 Active (0 Ready) / 1 Succeeded / 0 Failed
        Pod Template:
        Labels:  batch.kubernetes.io/controller-uid=6f9a736c-0379-4e27-8534-042490b3f322
                batch.kubernetes.io/job-name=test-job
                controller-uid=6f9a736c-0379-4e27-8534-042490b3f322
                job-name=test-job
        Containers:
        long-job:
            Image:      python
            Port:       <none>
            Host Port:  <none>
            Command:
            python
            Args:
            -c
            import time; print('starting'); time.sleep(30); print('done')
            Environment:  <none>
            Mounts:       <none>
        Volumes:        <none>
        Events:
        Type    Reason            Age    From            Message
        ----    ------            ----   ----            -------
        Normal  SuccessfulCreate  2m41s  job-controller  Created pod: test-job-pklnl # here the POD which created by the JOB
        Normal  Completed         2m5s   job-controller  Job completed


        # if we want to see the POD then we can see that as below 
        kubectl describe pod test-job-pklnl
        # the output will be as below 
        Name:             test-job-pklnl
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Sat, 10 Feb 2024 15:48:15 +0530
        Labels:           batch.kubernetes.io/controller-uid=6f9a736c-0379-4e27-8534-042490b3f322
                        batch.kubernetes.io/job-name=test-job
                        controller-uid=6f9a736c-0379-4e27-8534-042490b3f322
                        job-name=test-job
        Annotations:      <none>
        Status:           Succeeded
        IP:               10.244.0.162
        IPs:
        IP:           10.244.0.162
        Controlled By:  Job/test-job
        Containers:
        long-job:
            Container ID:  docker://3a9ef97053c68d68ee7100d43733b9cbf57811da7ac41a34263d10e864fd0ae1
            Image:         python
            Image ID:      docker-pullable://python@sha256:5eda1a3e78a90e7542c221cf525233f4b958a5778e61bcc350cc1e0d2bcf7ecf
            Port:          <none>
            Host Port:     <none>
            Command:
            python
            Args:
            -c
            import time; print('starting'); time.sleep(30); print('done')
            State:          Terminated
            Reason:       Completed
            Exit Code:    0
            Started:      Sat, 10 Feb 2024 15:48:19 +0530
            Finished:     Sat, 10 Feb 2024 15:48:49 +0530
            Ready:          False
            Restart Count:  0
            Environment:    <none>
            Mounts:
            /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-b2ggg (ro)
        Conditions:
        Type              Status
        Initialized       True 
        Ready             False 
        ContainersReady   False 
        PodScheduled      True 
        Volumes:
        kube-api-access-b2ggg:
            Type:                    Projected (a volume that contains injected data from multiple sources)
            TokenExpirationSeconds:  3607
            ConfigMapName:           kube-root-ca.crt
            ConfigMapOptional:       <nil>
            DownwardAPI:             true
        QoS Class:                   BestEffort
        Node-Selectors:              <none>
        Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                    node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
        Events:
        Type    Reason     Age    From               Message
        ----    ------     ----   ----               -------
        Normal  Scheduled  3m51s  default-scheduler  Successfully assigned default/test-job-pklnl to minikube
        Normal  Pulling    3m50s  kubelet            Pulling image "python"
        Normal  Pulled     3m47s  kubelet            Successfully pulled image "python" in 2.96s (2.96s including waiting)
        Normal  Created    3m47s  kubelet            Created container long-job
        Normal  Started    3m47s  kubelet            Started container long-job

        # if we want to checking the logs of the POD then wre can see as below
        kubectl logs test-job-pklnl
        # this will show the logs for the POD
        # the output will be as below 
        starting
        done


    ```

- we can also simulate `POD which been created By the JObs` that got failed by using the command as below 

- we can see that as with the below command 

    ```bash
        
        kubectl delete job test-job
        # deleting the existing JOB which will also delete the POD
        # the output will be as below in this case
        job.batch "test-job" deleted

        # or we can use the command as below 
        kubectl delete job --all
        # this will delete all the JOBS inside the default namspace

        # now we can reapply the JOB to deploy onto the cluster
        kubectl apply -f job.yml
        # deploying the changes to the cluster by applying the changes 


        # we can use another terminal session to remove the docker container which the POD want to spin in order to make the POD failed 
        # here we can remove the docker container as below 
        docker container ls | grep python 
        # here we are trying the fetch python container which the test-job POD going to spin 
        # this will provide the ID of the container
        da0fa2e9a98f   python                      "python -c 'import t…"   10 seconds ago   Up 10 seconds             k8s_long-job_test-job-5r5cp_default_ea8fab5f-336a-480e-a1ec-316e69f2dbc7_0

        docker container kill <id of the python container>
        # for example in here
        docker container kill da0fa2e9a98f
        # this will remove the python container subsecquently making the POD as failed 
        # here the output will be as below 
        da0fa2e9a98f

        watch kubectl get pods
        # fetching the PODs ionside the default namsespace 
        # the output will be as below in this case
        NAME                                                READY   STATUS    RESTARTS         AGE
        test-job-5r5cp                                      0/1     Error     0                3m34s

        # imp
        # as we have defined the backoffLimit as 2 hence the kubernetes JOb will create another POD 
        # if we also perform the same step as above

        # we can use another terminal session to remove the docker container which the POD want to spin in order to make the POD failed 
        # here we can remove the docker container as below 
        docker container ls | grep python 
        # the output in here as below 
        8e76c916af42   python                      "python -c 'import t…"   10 seconds ago   Up 9 seconds              k8s_long-job_test-job-ckhvc_default_7b77eb81-01b6-42fc-a661-2c2a033255d4_0


        docker container kill <id of the python container>
        # for example in here
        docker container kill 8e76c916af42
        # this will remove the python container subsecquently making the POD as failed 
        # here the output will be as below 
        8e76c916af42

        watch kubectl get pods
        # fetching the PODs ionside the default namsespace 
        # the output will be as below in this case
        NAME                                                READY   STATUS    RESTARTS         AGE
        test-job-5r5cp                                      0/1     Error     0                3m34s
        test-job-ckhvc                                      0/1     Error     0                2m52s

        # imp
        # as we have defined the backoffLimit as 2 hence the kubernetes JOb will create another POD 
        # if we also perform the same step as above

        # we can use another terminal session to remove the docker container which the POD want to spin in order to make the POD failed 
        # here we can remove the docker container as below 
        docker container ls | grep python 
        # the output in here as below 
        1187e2523412   python                      "python -c 'import t…"   7 seconds ago    Up 6 seconds             k8s_long-job_test-job-xtrdg_default_48fa5420-4346-4ff8-bb66-7ef40687bbb3_0


        docker container kill <id of the python container>
        # for example in here
        docker container kill 1187e2523412
        # this will remove the python container subsecquently making the POD as failed 
        # here the output will be as below 
        1187e2523412

        watch kubectl get pods
        # fetching the PODs ionside the default namsespace 
        # the output will be as below in this case
        NAME                                                READY   STATUS    RESTARTS         AGE
        test-job-5r5cp                                      0/1     Error     0                3m34s
        test-job-ckhvc                                      0/1     Error     0                2m52s
        test-job-xtrdg                                      0/1     Error     0                2m7s

        # as the backoffLimit now being ,matched hence the Job will not create another POD
        # we can see that in the Job description as well
        # we can see that as below 
        kubectl describe job test-job
        # we can see the below output in that case
        Name:             test-job
        Namespace:        default
        Selector:         batch.kubernetes.io/controller-uid=e3f7b94a-c97f-4bbd-aeb8-5232db661c47
        Labels:           batch.kubernetes.io/controller-uid=e3f7b94a-c97f-4bbd-aeb8-5232db661c47
                        batch.kubernetes.io/job-name=test-job
                        controller-uid=e3f7b94a-c97f-4bbd-aeb8-5232db661c47
                        job-name=test-job
        Annotations:      <none>
        Parallelism:      1
        Completions:      1
        Completion Mode:  NonIndexed
        Start Time:       Sat, 10 Feb 2024 15:59:19 +0530
        Pods Statuses:    0 Active (0 Ready) / 0 Succeeded / 3 Failed
        Pod Template:
        Labels:  batch.kubernetes.io/controller-uid=e3f7b94a-c97f-4bbd-aeb8-5232db661c47
                batch.kubernetes.io/job-name=test-job
                controller-uid=e3f7b94a-c97f-4bbd-aeb8-5232db661c47
                job-name=test-job
        Containers:
        long-job:
            Image:      python
            Port:       <none>
            Host Port:  <none>
            Command:
            python
            Args:
            -c
            import time; print('starting'); time.sleep(30); print('done')
            Environment:  <none>
            Mounts:       <none>
        Volumes:        <none>
        Events:
        Type     Reason                Age    From            Message
        ----     ------                ----   ----            -------
        Normal   SuccessfulCreate      8m41s  job-controller  Created pod: test-job-5r5cp # here the POD which created by the JOB
        Normal   SuccessfulCreate      7m59s  job-controller  Created pod: test-job-ckhvc # here the POD which created by the JOB
        Normal   SuccessfulCreate      7m14s  job-controller  Created pod: test-job-xtrdg # here the POD which created by the JOB
        Warning  BackoffLimitExceeded  6m54s  job-controller  Job has reached the specified backoff limit # here we can see that Job limit has reached

        # now here we do a kubectl get jobs we can see the 0/1 completed
        kubectl get jobs
        # fetching all the JObs inside the default namspace
        NAME       COMPLETIONS   DURATION   AGE
        test-job   0/1           18m        18m
 


    ```

- the `backoffLimit` being optional in this case as `4` which is optional 

- if we don't mention then `default value for the backoffLimit` is `6` in this case

- the `kubernetes JOb` will look for the `intermittent failure` i.e the `failure` which is not `expected to happen` but` might happen` such as `node failure`

- this is a `in a way, saying to kubernetes that we want to run  a special type of POD` , if `POD running fine then its ok` else if the `POD failed` try of until the `backoffLimit` being reached , making sure the `POD completed`

-  