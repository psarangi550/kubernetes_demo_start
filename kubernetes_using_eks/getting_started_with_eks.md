# Getting Started with Amazon EKS cluster

- here we want to use the `Amazon EKS` which stands for `Amazon Elastic Kubernetes Service` , which been `built into AWS` in order to `bootstrap` a `kubernetes cloud cluster`

- the `AWS EKS cluster` `not avaialable` in `all the region of AWS` , here we will be using the `Frankfruit(eu-central-1)` region as there are `no resources creaqted in the past for that region` , if the `EKS not available for the region` then use the `nearest region to spin up the AWS EKS cluster` as there will be little `network latency`

- here we will be choosing a `region such as eu-=central-1(FrankFurt)` which is never been used because `currently no resources in the past` available or lingering in that region 

- as like other `AWS Services` we have the `graphical user interface` , we might think `which will be easy to use`

  - we can put the `Cluster Name for the EKS Cluster`
  
  - we can put the `kubernetes version` all those things
  
- but the problem with the `Amazon EKS graphical user interface` , whi9ch is not a `easy process to doit using the Amazon EKS GUI` , we might have to `go several steps back and forward creating other AWS Service required for the EKS Process` in order to create the `EKS kubernetes cluster `

- but now the `AWS EKS cluster` has improved dramatically `while creating the Amazon EKS cluster` which is `usable but have couple of rough edges which need to be sorted out`

- here we will be using the command line tool called the `eksctl` in order to create the `Amazon EKS cluster` , the `userguide` for the command line tool such as `eksctl` given as below [eksctl AWS userguide](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)

- with the help of `eksctl command line tool` we can use the `amazon EKS cluster` with `joy`

- `eksctl` has also have the `github page` as [eksctl HOME page](https://github.com/eksctl-io/eksctl)  

- `eksctl ` is a `simple CLI tool` for `creating clusters on EKS - Amazon's new managed Kubernetes service for EC2`. `It is written in Go, and uses CloudFormation`

- we need this `eksctl` on top of the `AWS` command line to ease out the `work with AWS >EKS cluster` , `eksctl` offcially linked with the `Amazon EKS docs` which make it `authentic/official to use as the command line to bootstrap the AWS EKS cluster`

- we can create a `bootstrap EC2 instance which is not part of the Amazon EKS cluster` in order to `install the AWS EKS command line tool eksctl` on the `Bootstrap EC2 instance`

- here we can use the `EC2 instance as(t2.micro) for "Amazon Linux 2 instance (where AWS CLI auto installed) on FrankFurt region " ` & `providing a Tag with the name as Name:Bootstrap`  

- here the `security group` is the `firewall` , we need to `tell that we want to access the port 22 for the AWS EC2 instance` , hence we need to open the `port 22` &rarr; `myip` &rarr; `it will select the local ip address of the local computer` , but we need to be careful , if we comeback on the next Day and `ISP(internet service provider)` provide a `new local IP address` we need to update that `security group` else we will not be able to `login to the EC2 instance`

- then once we goto the `review+launch on EC2 instance we are building ` &rarr; `which will create the Key Pair` , if we are creating for the `firsttime` then we can select `create a new key pair` &rarr; `provide an name to the new key pair` &rarr; `Download the key pair` , please don't share this `key pair PEM` with anyone or `push to the repo as well` , this key can't be generated again hence make sure keep it somewhere safe , else we need to `spin the new EC2 instance and terminate the existing instance`

- we need to access the `EC2 instance` using the `ssh command line` as below 

    ```bash
        chmod go-rwx <path to the pem file>
        # here go stands for group and other
        # here - means we are removing the read write and exexute permission here
        # here we need to provide the appropriate permission for the PEM file 
        # if we are not [providing the appropriate access label then we can get error there
        
        # if running on windows we can solve this problem with as below link
        # https://superuser.com/questions/1296024/windows-ssh-permissions-for-private-key-are-too-open
        
        ssh -i <path to the pem file> ec2-user@<public ip of the EC2 instance>
        # once the EC2 instance running state we can use the public ip of the EC2 instance
        # this will ask for the fingerprint if we are usingit for the first time 

    
    ```