# managing A cluster in cloud

- `A kubernetes lcuster in cloud` has the `collection of nodes` which is also known as `worker node`

- The job of the `Worker nodes` is to `run the Microservice or web container POD inside it` , we can have `as many number of worker node as we want depends on the size of the application`

- we have an `additional node called the master node` inside the `kubernetes cluster in cloud such as AWS`

- we can have `more than one master node inside the cluster` , but `one master node will be enough to run the entire kubernmetes cluster in cloud`

- the `master node` is responsible to run the `entire show for the kubernetes cluster`  , we will issue the `kubectl command against the master node`  and `master node decide on which node it will run the corresponding microservice POD`

### How to Set up the kubernetes cluster in cloud

- we can start `manually` 
  
  - where we need to `start a lot of instances` and declare them as `kubernetes Nodes` which can take a lot of effort
  
  - we also need to take care the `networking between these kubernetes nodes or instances`
  
  - this type of installation called as the `baremetal installation` , this will done when we are `deploying to own hardware instances`
  
- but here as we are `leveraging` the `AWS Hardware` hence we can use `collection of tools` to `bootstrap a kubernetes cluster` , using those tools we can 
  
  - we can create `kubernetes cluster` with `specific name` 
  
  - we can also specify the `number of worker node` we want for that `kubernetes cluster`
  
  - then the tool will create the `kubernetes clsuter with specific name and number of worker node mentioned with in it`
  
- there are `lot of tools we can choose with which we can create the kubernetes cluster in thew cloud with AWS Hardware` , but `2 of them are most popular`
  
  
  - `Kops`
    
    - `Kops` is the `older of the two choices` we have in this case 
    
    - we can see `Kops` github page on [Kops Github Web Page](https://github.com/kubernetes/kops)
    
    - this is the `official part of the kubernetes infrastructure` , not `some sort of third party software`
    
    - Tag line  is the `easiest way to get the production grade kubernetes cluster` and make it as `up and running`
    
    - `kops` will not only help you `create, destroy, upgrade and maintain` `production-grade, highly available, Kubernetes cluster`, but it will also `provision the necessary cloud infrastructure`
  
  
  - `EKS`  
    
    - `EKS` is part of the `AWS infrastructure itself`
    
    - if the `projects  are highly invested on AWS` then we might have to work with `EKS` most often
    
    - Tag line is `Most Trusted way to run kubernetes`
    

### PROS and CONS OF CREATING CLUSTER USING EKS OR KOPS

- the big difference `between` `EKS and KOPS` being `masternode`

- **Kops**

- if we are working with `kops` then we are responsible for `managing the master node`

- in case of `Kops` we can `handle or maintain and access` the `master node` which been running in the `EC2 instance node`  like other `worker nodes`

- we can access the `EC2 instance` and `turn the masternode` on or off , which will be `terrible idea` , but even though we `shutdown the masternode` it will be getting created a new masternode like the `kubernetes service in minikube cluster`

- even though the `masternode` been `terminated` the rest of the `worker node` will be `working just fine`

-  but the problem is that `until  the masternode comes backup` we can't `administer the kubernetes cluster` as the `kubectl command against the master node will not work` untill its been up and running 

- with `Kops` we are responsible to handle the `masternode` , hence there will be `duties and resposnsibiliity` comes along with it  


- **EKS**

- in case of `EKS` we can't even `visualize the master node`

- `masternode` in case of `EKS` will be running on `background` and `protected` by the `Amazon`

- `Amazon` will `handle and maintain the master node`

- here the `masternode` will not appeaar as the `EC2 instance like kops`

- we can't terminate the `kubernetes masternode` in case of the `EKS` , which can sound `terrible` as we are not in control

- but the benifit being `less responsibility as AWS will going to take care for that`
 