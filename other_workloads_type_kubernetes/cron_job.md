# cron Jobs in kubernetes 

- A `JOb` is a `kubernetes object` which create the `PODs` and `kubernetes` does it very best to make sure the `completion of the POD for the specific Task or job`

- the `main tuning element for the kubernetes Job` being as `backoffLimit` , which is the `number of times` `kubernetes attempt to reschedule` the `POD` in case of `POD task failure or does not complete`

- the `prime purpose` of the `kubernetes JOb` to make sure the `POD complete its job or Task` even if in case of `unpredictable failure such as node failure which can lead to POD does not complete/finished its task`

- **CronJob**

- A `CronJob` is a special type of `Job` in `kubernetes`

- A `Cron` is a `unix process` which will help in `schedule jobs` on the `Unix or Linux OS Platform`

- A `Cron` is a `linux or unix process/tool` which will allow to schedule the `jobs` on a `regular basis`

- the `Cron` been driven by the `syntax` called as `Crontab` which we can also visit by [crontab web page](htts://crontab.guru/)

- in `crontab syntax` we can `specify 5 values` to `schedule` when the `Job or Task` will run 

- where the `* * * * *` means that `run this Job every minute`

- where the `<minutye field> <hour field> <day of the month> <month> <day of the week>` 

- here we will be seeing the `Kubernetes Job that we have written on the preveious video` and `upgrade that` to a `scheduled job` or `kubernetes cron job`

- we can add the `schedule` field onto the `existing kubernetes Jobs` to make it as a `cronjob`

- here we will have the `existing JOB YAML definition` we need to add the `wrapper around that` to put the `CronJob` Definition

- we can see the `reference` on the `kubernetes manual` &rarr; `API reference` &rarr; `CronJob` 

- the `status` of the `cronjob` will be put on the `runtime` , we don't have to `put those details in the definition here`

- we can define the `cronjob` definition as below 

    
    ```yaml
        cron.yml    
        =========
        apiVersion: batch/v1 # here the apiVersion which can also been seen using the command as kubectl api-resources -o wide | grep cron
        kind: CronJob # type of kubernetes object is of type Cronjob
        metadata: # name of the Cronjob
            name: cron-job
        spec: # specification for the Cronjob
            schedule: "* * * * *" # here we will settting the crontab expression which will try to run the job at every minute
            jobTemplate: # here we will define the kubernetes JOb definition over here
                spec: # specification for the kubernetes Job
                    template: # template for the kubernetes POd definition defined in here
                        spec: # specification for the POD definition 
                            containers: # container definition defined over here
                                - name: cron-python-job # name of the container
                                  image: python:rc-slim # image for the container
                                  command: ["python"] # command that will run inside the container
                                  args: ["-c", "import time ; print('starting') ; time.sleep(30) ; print('done')"] # args that will be p[assed to the container
                            restartPolicy: Never # here defining the restartPolicy as Neverwhich make sure the POd will not restart and run the container even it got failed or completed
                    backoffLimit: 2 # defining the backoffLimit as 2 if the Job been find error due to unexpected reason then it will auto schedule another POD for the Deployment 


    ```

- we can  define the `metadata(optional) and spec` inside the `jobTemplate field` which will define the `specification for the JOb`

- 

- we can `deploy this changes` `onto the cluster` by applying the changes as below 

    ```bash

        kubectl delete job test-job
        # deleting the existing job that we have deployed to the cluster in the default namespace
        # the output will be as below 
        job.batch "test-job" deleted

        kubectl apply -f cron.yml
        # we can deploy the changes onto the cluster by applying the chnages
        # here we will get the below response
        cronjob.batch/cron-job created

        #now if we put the watch to see the pods we can see as below 
        watch kubectl get pods
        # we can see the output as below in this case
        NAME                                                    READY   STATUS      RESTARTS      AGE
        pod/cron-job-28459582-4px64                             0/1     Completed   0             67s
        pod/cron-job-28459583-4wjn5                             1/1     Running     0             7s
         
        # here we can see that for every minute the cro job make sure the POD get ran and completed successfully
        # for every minute it will create the POD and make sure POD completed its task successfuloly
        # as its running the task in case of failure the JOb will schedule another POD to fix the same

        # we can see the details about the cronjob using the command as below
        kubectl get cronjobs
        # this will fetch the cronjob inside the default namespace
        NAME       SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
        cron-job   * * * * *   False     1        13s             22s 

        # we will also be able to see the details about the cron job by describing it as below 
        kubectl describe cronjob cron-job
        # fetching the details about the cronjob in here
        Name:                          cron-job
        Namespace:                     default
        Labels:                        <none>
        Annotations:                   <none>
        Schedule:                      * * * * *
        Concurrency Policy:            Allow
        Suspend:                       False
        Successful Job History Limit:  3
        Failed Job History Limit:      1
        Starting Deadline Seconds:     <unset>
        Selector:                      <unset>
        Parallelism:                   <unset>
        Completions:                   <unset>
        Pod Template:
        Labels:  <none>
        Containers:
        cron-python-job:
            Image:      python
            Port:       <none>
            Host Port:  <none>
            Command:
            python
            Args:
            -c
            import time ; print('starting') ; time.sleep(30) ; print('done')
            Environment:     <none>
            Mounts:          <none>
        Volumes:           <none>
        Last Schedule Time:  Sat, 10 Feb 2024 20:46:00 +0530
        Active Jobs:         cron-job-28459636
        Events:
        Type    Reason            Age                 From                Message
        ----    ------            ----                ----                -------
        Normal  SuccessfulCreate  50m                 cronjob-controller  Created job cron-job-28459586
        Normal  SawCompletedJob   49m                 cronjob-controller  Saw completed job: cron-job-28459586, status: Complete
        Normal  SuccessfulCreate  49m                 cronjob-controller  Created job cron-job-28459587
        Normal  SawCompletedJob   48m                 cronjob-controller  Saw completed job: cron-job-28459587, status: Complete
        Normal  SuccessfulCreate  48m                 cronjob-controller  Created job cron-job-28459588
        Normal  SawCompletedJob   47m                 cronjob-controller  Saw completed job: cron-job-28459588, status: Complete
        Normal  SuccessfulCreate  47m                 cronjob-controller  Created job cron-job-28459589
        Normal  SuccessfulDelete  46m                 cronjob-controller  Deleted job cron-job-28459586
        Normal  SawCompletedJob   46m                 cronjob-controller  Saw completed job: cron-job-28459589, status: Complete
        Normal  SuccessfulCreate  46m                 cronjob-controller  Created job cron-job-28459590
        Normal  SuccessfulDelete  45m                 cronjob-controller  Deleted job cron-job-28459587
        Normal  SawCompletedJob   45m (x2 over 45m)   cronjob-controller  Saw completed job: cron-job-28459590, status: Complete
        Normal  SuccessfulCreate  45m                 cronjob-controller  Created job cron-job-28459591
        Normal  SuccessfulDelete  44m                 cronjob-controller  Deleted job cron-job-28459588
        Normal  SawCompletedJob   44m                 cronjob-controller  Saw completed job: cron-job-28459591, status: Complete
        Normal  SuccessfulCreate  44m                 cronjob-controller  Created job cron-job-28459592
        Normal  SuccessfulDelete  43m                 cronjob-controller  Deleted job cron-job-28459589
        Normal  SawCompletedJob   43m                 cronjob-controller  Saw completed job: cron-job-28459592, status: Complete
        Normal  SuccessfulCreate  43m                 cronjob-controller  Created job cron-job-28459593
        Normal  SuccessfulDelete  42m                 cronjob-controller  Deleted job cron-job-28459590
        Normal  SawCompletedJob   42m (x2 over 42m)   cronjob-controller  Saw completed job: cron-job-28459593, status: Complete
        Normal  SuccessfulCreate  42m                 cronjob-controller  Created job cron-job-28459594
        Normal  SuccessfulDelete  41m                 cronjob-controller  Deleted job cron-job-28459591
        Normal  SawCompletedJob   41m                 cronjob-controller  Saw completed job: cron-job-28459594, status: Complete
        Normal  SuccessfulCreate  15s (x42 over 41m)  cronjob-controller  (combined from similar events): Created job cron-job-28459636


    ```

- if we have a `process` which need to `run on everyDay at midnight everyday` then we can do that by using the `Cronjob in kubernetes`

- as its a `wrapper` around the `normal JOb` hence if the `JOB does not complete because of POD failure` the `Job will schedule another POD untill the backoffLimit` being reached , hence there will be `some level of gurantee the Job wil;l be completed`
