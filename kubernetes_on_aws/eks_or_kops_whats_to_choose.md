# EKS vs KOPS

- The `difference` between `Kops and EKS` being very `marginal or low`

- **Kops**
    
    - its the `reliable workhorse` to manage the `kubernetes clsuter on cloud` 

    - **Pros**
  
      - `Kops` being `well respected` and `heavily used` , it has a `long history` , `very stable` and  `doesnot have very rough edges `
  
      - it is very easy to use , well  used tool
  
    - **cons**
      
      - we will be responsible to handle the `master node` in this case in here
      
      - back to the `3AM Alert` , if the `masternode` being down on `3AM` we will not be able to `administer the cluster untill the masternode restarts`     
      
      - we still can get the `alert for those issue which can suggest that masternode getting high CPU usage for Kops` we need to `see that whats the problem with the masternode to fix it`
      
      - there will be `more to think about` if we are using the `Kops kubernetes cloud cluster as we are handling the master node`
      
      - by default if we are creating the `Kubernetes cluster using Kops` then we will be getting `one master node` , but we can easily `override the same`
      
      - if we have the `default single master Kops cluster` that went down `then we can't administer the kops cluster untill the master node back on`
      
      - but we absolutly can run the `more than one master node inside the Kops cluster` but we need to `red and configure those changes`
      
      - in case of `EKS` we don't have the access to see the `master node` , there will be `multiple masternode which is not visible as its managed by amazon` handling the `EKS cluster`
      

- **EKS**   
  
  - it is the `amazon hosted version of kubernetes cluster` 
  
  - **Pros**
    
    - it is the `amazon hosted version of kubernetes cluster`  which is going to handle the `masternode` for you
    
    - the `biggest advantage` by far its gaining more popularity , getting `more and more number of customer`
    
    - if we have the `yaml definition source code` on the `source control` moving from `Kops to EKS` will not be a problem , we simply can `create an alternate cluster` and `deploy or port` the `kubernetes resource` , which can seem easy in theory but take effort in time while doing it realworld pragmatically
    
    - we can use the `additional 3rd party tool` called `eksctl` in order to `handle the EKS Kubernetes cluster` , which can seems quite smooth
    
    - No management of the `master required` in case of the `EKS cluster`
    
    - `EKS` integrate `with another AWS Service called the AWS Fargate`
    
    - `AWS Fargate` is a `new service provided by AWS` which will allow you to `run the container with no service` which is called the `serverless service`
    
    - `actually there will be service` `but we can't see or manage or provision them AWS deoes it on the behalf`
    
    - if we choose the `AWS Fargate` we can run the `kubernetes cluster` without `services and EC2 instance as nodes`   
    
  - **Cons**
    
    - we need to use the `3rd party tool` such as `eksctl` in order to make use of the `EKS cluster` , becuase the `GUI` is not `user friendly`
    
    - we can't even `create the EKS cluster` using the `AWS Mgmt Gui`
    
    - might feel like `we are tied to AWS Infrastructure` , but which is not true as `EKS` under the hood `uses kubernetes to maintains its clsuter`
    
       