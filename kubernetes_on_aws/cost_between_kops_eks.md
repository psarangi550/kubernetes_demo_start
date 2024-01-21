# Cost Difference Between Kops and EKS

- for `every server` we will be going to spin up for the `kubernetes node(masternode and worker node)` we will have to  pay to `AWS`

- if we are running the `kubernetes cluster` in cloud we need to `delete those cluster` so that the charges are not `on going` , which will not cost you more than `10$ for the entire project`

- **with respect to `project` is there a vast cost difference between `EKS` or `kops` ?**

- regardless of the `EKS or Kops` we need to pay for the `amount` for the `kubernetes worker node` as `for the project it will be the same`

- it will the cost of the `running the masternode` that `will going to be different `

- but its `actually not only` the `masternode` only , but also need the `load balancer` in order to access the `masternode` 

- its not just the `masternode` , `EKS/Kops` manage `few other thing for you` , which is known as the `controlplane` in this case  

- here we will be discussing `How much it will cost to run the control plane using EKS or Kops`

- in case of both `EKS/Kops` we are in control of the `EKS cluster` that we will be using 

- **Kops**

  - `kops` will `create` the `loadbalncer` in order to access the `masternode` , while creating the `Kops kubernetes cluster`
  
  - while running the `control plane` using the `Kops` , we needc to decide `what type of node(EC2 instance) we want to use for the  masternode` and `loadbalancer`
  
  - if there are `thousand of PODs need to be run on the kubernetes cluster in cloud` and we also need the `monitoring the kubernetes clustyer created using the Kops` then we might need the `compute intensive kubernetes node in this case` that can `significantly affect the cost in this case`
  
  - if we don't have the `big cluster` then we can use the `kubernetes node` as the `m3.medium`  for the `masternode (EC2 instance)` , which is suitable for the `small and medium size cluster`
  
  - if we decide to use the `m3.medium`  for the `masternode (EC2 instance)`  then the running cost of that being `600 USD per year` , if we are running the `kubernetes cluster with m3.medium kubernetes masternode` then the `cost for that being significantly less`
  
  - one of the `downside of kops` that we need to decide `which type of kubernetes masternode we want to use` as the `masternode will be accessable on that node using the loadbalncer` 
  
  - we also need the `loadbalancer` to access the `masternode` , which cost can be upto `200 USD per year`   
  
  - which in total accumulate to `8000 USD per year` for the `Kops Kubernetes cluster` control plane
  
  - but if we are using `Kops` then we know that `Kops By default will going to create the single masternode` , which can be `sufficient to run most of the project`  
  
  - but the sot can vary if we are using `multiple masternode with m3.medium EC2 instance inside the kops cluster`
  
  - here we need to keep in mind that `we also have to pay for the workernode` in this case as well
  
  - here the `cost of the workernode depends what type of EC2 instance we will be using`  
  

- **EKS**
  
  - if we are creating the `EKS Kubernetes cluster` then `we don't see` the `loadbalancer for the masternode` , but rest assured that `loadbalncer is there which is hidden under the cover`
  
  - in case of `EKS` we need to pay a `set of fee for the entire controlplane(masternode + loadbalancer)`
  
  - we don't have to `choose the EC2 instance type for the masternode and can't even see that instance` in case of the `EKS cluster` , its just the `simple flat price`
  
  - the cost of `EKS control plane` in `2019` was `signigficantly highcost` but in `jan 2020` there is large cut down in the `price of control plane for the EKS cluster`
  
  - for the `EKS cluster control plan` the `flat price being $0.10 USD per hour` which comes around `876$ USD per month`
  
  - but we can have `multiple  masternodes` inside the `EKS kubernetes cluster` which being managed by `AWS` which make it very `affordable price if we are using multiple masternode inside the cluster along with the loadbalancer`
  
  - here also we need to keep in mind that `we also have to pay for the workernode` in this case as well    
  
  - here the `cost of the workernode depends what type of EC2 instance we will be using` 