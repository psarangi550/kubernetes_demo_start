# How to Delete EKS cluster in AWS

- we must have to `delete the EKS cluster` in order `avoid incurring ongoing cost`

- we haver to run the `simple eksctl command` as below in order to remove the `EKS cluster` that we created using the `eksctl command` 

    ```bash
        eksctl delete cluster <name of the cluster>
        # we can use the eksctl command which will delete the cluster
        # for example we have the cluster with name as fleetman hence we can use the command as below 
        eksctl delete cluster fleetman
        # here we are tryuing to delete the fleetman cluster in this case

    ```

- this command will take around `5 mins to run` on the `eu-central-1` region 

- when we will issue the `eksctl delete cluster <cluster name>` command this `will going to delete` the `loadbalancer` that got created because of the `fleetman-webapp` Service in this case

- we can goto the `AWS Mgmt console` &rarr; `EC2 instance` &rarr; `Load Balancers` &rarr; `we should not see any Load Balancer` in this case

- if we  goto the `AWS Mgmt console` &rarr; `EC2 instance` &rarr; `running instances` , we must see all the `worker node instances` must be stopped in this case

- we can see the `logs` as `deleting the nodegroup` which is the `set of instances` that `got deleted`

- once the command executed we can see all the `worker nodes` now `being deleted `

- finally we will see the logs as below `all cluster resource` were `now deleted `

- if we now goto the `AWS EKS` by going into the `AWS Mgmt console` &rarr; `Services` &rarr; `EKS` &rarr; `Clusters` , then we can see `no cluster should be listed in here`

- when we see the `AWS EKS` doesnot `list out the cluster that we created in the region` that means that `control plane (masternode+corresponding Load Balancer)` now being `deleted as well`

- `one thing which will not be deleted` is the `EBS volume` with which `we use the persistentVolumeClaim with EBS-Volume for the mongodb POD`

- if we now goto the `AWS Mgmt console` &rarr; `EC2 instance` &rarr; `Elastic Block Store` &rarr; `Volumes` &rarr; `Then we can see the volume listed in here`

- if we are using `kops` and we use the command as `kops delete cluster --name {NAME} --yes` then it will going to `delete the EBS Volume` which we have used as the `persistentVolumeClaim` with the `StorageClass` which will be converted to `persistentVolume` at runtime as `EBS Volume`

- we can see the `Status of the EBS volume` as `available state` in this case , but the `eksctl` will leave the `virtual SSD on the available state without deleting the same` for `recovering the info from the virtual SSD`

- if we leave those `virtual SSD created for the persistentVolume for the mongodb using the storageClass and persistentVolumeClaim` will be not that high but it will be as `$0.1 per GB per Month` so if we have the `$0.7 per month if we have the 7GB of Storage`

- we can `delete that manually` by `selecting the available Volume` &rarr; `Action` &rarr;`Delete Volume`

- if we have `bootstrap instance` which you will using for the `EKS cluster` then `you need to terminate the instance` and then only yopu can `remove the EBS volume associated with that`

