# configuring your first cluster using kops cli in AWS 

- if we are goin to the github documentation link as [Aws Kops GitHub](https://github.com/kubernetes/kops/blob/master/docs/getting_started/aws.md)

- **Cluster State store**
  
  - In order to `store` the `state of your cluster`, and the `representation of your cluster`, we need to `create a dedicated S3 bucket for kops to use`.
  
  - `AWS S3` is a `distributed file system` on `AWS` , this is where `Kops` store the `Working Data`
  
  - we can do that using the command line  with the `aws cli` as below 
    
    ```bash
        # this  command will help in create the s3 bucket on a particular region i.e us-east-1
        aws s3api create-bucket \
        --bucket prefix-example-com-state-store \
        --region us-east-1
        
        # Note: S3 requires --create-bucket-configuration LocationConstraint=<region> for regions other than us-east-1.
        # if we want to create the s3 bucket for any other region then we need to provide the --create-bucket-configuration LocationConstraint=<region> 
        # we can do that for the us-west-2 as below 
        
        aws s3api create-bucket --bucket <your-bucket-name> --region <your-region> --create-bucket-configuration LocationConstraint=<your-region>
        
        # for example the us-west-2 and unique bucket name can be written as 
        
        aws s3api create-bucket \
        --bucket pratik-kops-12345 \
        --region us-west-2 \
        --create-bucket-configuration LocationConstraint=us-west-2
        
        # here bucket name should be unique accross globally 
        # i.e the bucket name should be unique which has never been used in the history of Amazon s3
        # we will be getting error if we are using the name which is already been taken by the Amazon s3
        # the output will be as below 
        
        {
            "Location": "http://pratik-kops-12345.s3.amazonaws.com/"
        }

        # the bucket name should be unique and should have 
            - lowercase alphabet
            - number
            - dash/hypen(-)
            - it can also contain (.)

        # we also have to turn off the BlockPublicAcls
        # for that we can run the command as below 
        # which will make sure the public Access enable for the bucket
        aws s3api put-public-access-block \
        --bucket pratik-kops-12345 \
        --public-access-block-configuration "BlockPublicAcls=false, \
                                            IgnorePublicAcls=false, \
                                            BlockPublicPolicy=false, \
                                            RestrictPublicBuckets=false"
                            


    ```

- we can add `encryption on the s3 bucket that we have created` , if we are using the `more professional platform` using the command line 

- we can also configure the `s3 bucket to be shared` so that `multiple kops can put or write the data into the s3 bucket`

- **Creating your first cluster**

- we need to set 2 `env variable` before we can create the `Kops cluster` as below 

- here we will be using the `gossip-based cluster` hence the `NAME` i.e `name of the cluster` ends with `k8s.local` 
    
    ```bash
        # setting up the environment variable
        export NAME=<name of the cluster>.k8s.local
        export KOPS_STATE_STORE=s3://<unique bucket name we have created>

        # for this example we can set as below 
        export NAME=fleetman.k8s.local
        export KOPS_STATE_STORE=s3://pratik-kops-12345
        
        # here we are setting the name of the cluster and state store pointing to the s3 buicket
    
    ```

- **creation of cluster configuration**

- we need to know `availability zone` available to us based on the region we selected in order to se the `cluster configuration`

- An `availability zone` is nothing but the `physical data center within the region` , which is `owned by amazon` 

- if we go to the `AWS Mgmt console` &rarr; `EC2` &rarr; `we can see the instances over here and the list of avaiablity zone on the main page`

- in a `particular region` amazon operate on a `physically separated data center` , these `datacenter are separated by considerable distance`

- if a particular data center affected by fire then the rest of the data center should be working fine as they are ohysically separated and placed far distance

- depedning on the `region` , we can have `2 or 3 or 4 availability zone/ datacenter available inside a particular region`

- we can fetch the `internal name of the region` using the page reference as [Region Internal Name](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html) 

- we can fetch the `avilability zone` by using it as below 

    ```bash
        aws ec2 describe-availability-zones --region us-west-2
        # using this aws cli command we can fetch the availabilty zone inside the region 
        # the output will be as below 
        {
        "AvailabilityZones": [
        {
            "State": "available",
            "OptInStatus": "opt-in-not-required",
            "Messages": [],
            "RegionName": "us-west-2",
            "ZoneName": "us-west-2a",
            "ZoneId": "usw2-az2",
            "GroupName": "us-west-2",
            "NetworkBorderGroup": "us-west-2",
            "ZoneType": "availability-zone"
        },
        {
            "State": "available",
            "OptInStatus": "opt-in-not-required",
            "Messages": [],
            "RegionName": "us-west-2",
            "ZoneName": "us-west-2b",
            "ZoneId": "usw2-az1",
            "GroupName": "us-west-2",
            "NetworkBorderGroup": "us-west-2",
            "ZoneType": "availability-zone"
        },
        {
            "State": "available",
            "OptInStatus": "opt-in-not-required",
            "Messages": [],
            "RegionName": "us-west-2",
            "ZoneName": "us-west-2c",
            "ZoneId": "usw2-az3",
            "GroupName": "us-west-2",
            "NetworkBorderGroup": "us-west-2",
            "ZoneType": "availability-zone"
        },
        {
            "State": "available",
            "OptInStatus": "opt-in-not-required",
            "Messages": [],
            "RegionName": "us-west-2",
            "ZoneName": "us-west-2d",
            "ZoneId": "usw2-az4",
            "GroupName": "us-west-2",
            "NetworkBorderGroup": "us-west-2",
            "ZoneType": "availability-zone"
        }
    ]
    }

    
    ```

- this is particularly powerful because when we want to create `kubernetes kops cluster` where we will be `deploying the microservice system` then we want the `kubernetes PODs will be distributed accross the kubernetes nodes` , it would make more sense that these `kubernetes nodes(worker nodes)` will be disributed accross `multiple availability zones`

- it would make more sense `if each of the node inside the kubernetes cluster` belong to `different availability zone` , which can provide more `stronger resilience`

- if one of the `availblity zone` `were down for any reason `then `if out microservice system i.e kubernetes POD distributed accross the kubernetes nodes which are in different availabity zone` then it can `provide more stronger resilient to faiulure  `

- here while creating the `configuration for the cluster` we can provide the `set of availablity zone` , where we want to `create these kubernetes nodes in order to distribute the microservice system`

- we can create the `kops kubernetes lcuster configuration` as below 

    ```bash
        kops create cluster \
        --name=${NAME} \
        --cloud=aws \
        --zones=us-west-2a,us-west-2b,us-west-2c,us-west-2d \
        --discovery-store=s3://pratik-kops-12345/${NAME}/discovery
        # here using this command kops create cluster we are actually not creating the cluster rather we are creating the configuration file for the cluster
        # here are providing the flag as below 
        --name --> name of the kubernetes cluster
        -- cloud --> cloud name where we want to deploy the cluster
        --zones ---> list of  availablity zone separated by comman where the kubenetes nodes will be distributed accross
        --discovery-store --> s3 storage where we eant to store the kops working data 

        # based on the number of zones we have defined we can have those many number of kubernetes node
        # here as we have defined 4 zones hence 4 worker node will going to be created and also a master node with load balancer will going to be get created 
        # but as discussed we need only 3 workier node hence we can specify that as below 
        kops create cluster \
        --name=${NAME} \
        --cloud=aws \
        --zones=us-west-2a,us-west-2b,us-west-2c\
        --discovery-store=s3://pratik-kops-12345/${NAME}/discovery

    ```

- if while executing the command `if we got an error` as `SSH public key must be specified when running with AWS (create with "kops create secret --name <name of your kubernetes clsuter> sshpublickey admin -i ~/.ssh/id_rsa.pub")` then we need to perform the below steps of setting the `SSH public key to be used by kops for the cluster`

- here when we get the output error as above actually we do have the `cluster configuration created successfully` , as we don't have the `SSH Public key set  hence we are getting this error`

- in order to `solve this issue` we need to generate the `SSH key` , for which we can perform actiopn as below 

    ```bash
        ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
        # here generating the ssh keygen with the format as rsa and bytes as 2048
        # here also the we are creating the ~/.ssh/id_rsa which will actually create the private key as id_rsa and id_rsa.pub
        # we will then can execute the kops create secret command as below 

        kops create secret \
        --name ${NAME} \
        sshpublickey admin \
        -i ~/.ssh/id_rsa.pub

        # buy using this command we can set the public part of the key to kops which can help in creating the cluster configuration 
        # here we will be using the public part of the SSH Key with the kops command to create the secret which help in creating the kops cluster configuration
        # this will provide kops the priviledges to work with the cluster that we are going to create in AWS


    ```

- but as we have `only the cluster configuration file creeated rather than the kops kubernetes cluster` hence if we goto `AWS Mgmt console` &rarr; `EC2` &rarr; `running instances` &rarr; `we can't see any  over here rather than the bootstrap one that we created for the application`

- **Customize Cluster Configuration**

- here in this case `the default cluster configuration` works with the `kops kubernetes cluster` that we want to build 

- if we want we can leave that to default , but if we want to investigate then we can use th below command in this case 

- we can `customize the configuration file for the cluster` , which can be done using the below command 

    ```bash
        kops edit cluster --name ${NAME}
        # which will shouw the configuration file for the cluster as below 
    ```

- the result of `kops edit cluster --name ${NAME}` will display the result in the `default VI Editor` , if we want to change then we can perform the below action 

- we can change to `nano GNU editor` for displaying the output for the   `kops edit cluster --name ${NAME}` by setting a `env variable` as `EDITOR` below

- the `configuration that we see for the kops cluster` is `pretty advance as we seen below` , we can use the use the `default configuration for the production standard cluster as well` 

- we can do that below  

    
    ```bash
        export EDITOR=nano
        # setting the EDITOR as nano in here , but make sure nano editor installed on ubuntu
        # setting the EDITOR as nano which will open the kops kubernetes lcuster configuration in nano editor as below
        kops edit cluster --name ${NAME}
        # we can see the below output 
        # Please edit the object below. Lines beginning with a '#' will be ignored,
        # and an empty file will abort the edit. If an error occurs while saving this file will be
        # reopened with the relevant failures.
        #
        apiVersion: kops.k8s.io/v1alpha2
        kind: Cluster
        metadata:
        creationTimestamp: "2024-01-14T14:33:16Z"
        name: fleetman.k8s.local
        spec:
            api:
                loadBalancer:
                    class: Network
                    type: Public
            authorization:
                rbac: {}
            channel: stable
            cloudProvider: aws
            configBase: s3://pratik-kops-12345/fleetman.k8s.local
            etcdClusters:
            - cpuRequest: 200m
                etcdMembers:
                - encryptedVolume: true
                instanceGroup: control-plane-us-west-2a
                name: a
                manager:
                backupRetentionDays: 90
                memoryRequest: 100Mi
                name: main
            - cpuRequest: 100m
                etcdMembers:
                - encryptedVolume: true
                instanceGroup: control-plane-us-west-2a
                name: a
                manager:
                backupRetentionDays: 90
                memoryRequest: 100Mi
                name: events
            iam:
                allowContainerRegistry: true
                legacy: false
                useServiceAccountExternalPermissions: true
            kubeProxy:
                enabled: false
            kubelet:
                anonymousAuth: false
            kubernetesApiAccess:
            - 0.0.0.0/0
            - ::/0
            kubernetesVersion: 1.28.5
            networkCIDR: 172.20.0.0/16
            networking:
                cilium:
                enableNodePort: true
            nonMasqueradeCIDR: 100.64.0.0/10
            serviceAccountIssuerDiscovery:
                discoveryStore: s3://pratik-kops-12345/fleetman.k8s.local/discovery/fleetman.k8s.local 
                enableAWSOIDCProvider: true
            sshAccess:
            - 0.0.0.0/0
            - ::/0
            subnets: # here we can all the availability zone in this case 
            - cidr: 172.20.0.0/18
                name: us-west-2a
                type: Public
                zone: us-west-2a
            - cidr: 172.20.64.0/18
                name: us-west-2b
                type: Public
                zone: us-west-2b
            - cidr: 172.20.128.0/18
                name: us-west-2c
                type: Public
                zone: us-west-2c
            topology:
                dns:
                type: Private
        
    
    ```

- **how to define the number of kubernetes working noe and master node for kops cluster**

- here we have not define `How many working or masternode` that we want for this `kops kubernetes cluster`

- in order to `specify the number of master and worker node` we need not to define inside `kops cluster configuration` using `kops edit cluster --name ${NAME}` , rather `kops` has the concept of `instance group` in order to define the same 

- `kops` by default create few `instance group (ig)` for us by default , put `some default value in there `

- we can `look at those instance group created by kops by default` using the command as below 
    
    ```bash
        kops get ig --name <name of the cluster which is of gossip cluster type>
        # for example
        kops get ig --name ${NAME}
        # here ${NAME} comes from the env variable we set preveiously
        # here the output will be as below 
            NAME				ROLE		MACHINETYPE	MIN	MAX	ZONES
        control-plane-us-west-2a	ControlPlane	t3.medium	1	1	us-west-2a
        # here it telling the cluster that we need the "group of node with the role as ControlPlane" for the kops cluster
        nodes-us-west-2a		Node		t3.medium	1	1	us-west-2a
        nodes-us-west-2b		Node		t3.medium	1	1	us-west-2b
        nodes-us-west-2c		Node		t3.medium	1	1	us-west-2c
        
        # for the rest of them its telling that we need the "MINIMUM of 1 group of node with the Role as Node" in the kops cluster
        # here the MACHINETYPE for both the master node and worker node is t3.medium
        # based the region Kops will choose the default MACHINETYPE which should be ok to work for small and medium business

        # here Kops has created the 3 worker nodes in this case based on the number of zones we mentioned in kops create cluster command 
        # preveiously it used to create only one 1 worker node and set the MIN and MAX value set there 
        # but which not that great , based on the load those worker nodes get created 
        # but we are being more explicit in this case 

         # here also the MIN and MAX for the worker node is 1 
         # that means all the Worker nodes are in different Avaialbility zone which provide more resilience

        # here the MAX represent the MAX number of specific node that we want to spin up  which can be helpful in cluster scaling
        # cluster scaling is usally means to change the number of nodes based on the loads on the cluster decided by kops
        # cluster scaling is complecated , but we don't have to deal with the MAX Entry , MIN Entry where we bother about 


        # for every zone that we have mentioned in the below command those many number of worker node will going to be get created 
        # we run the command as below 
        kops create cluster \
        -- name=${NAME}
        -- zones=us-west-2a,us-west-2b,us-west-2c \
        --discovery-store=S3://pratik-kops-12345/${NAME}/discovery

        # here this will create the 3 working nodes in `us-west-2a,us-west-2b,us-west-2c` as shown above

        # if we do have only 2 availability zone then we can change the number of MIN and MAX for the Zone based node as below 
        kops edit ig <instance group name> --name <name of the cluster which is of gossip cluster type>
        
        # let suppose if we want to create 4 worker node for the us-west-2 region which has only 3 Availability zone(assume)
        # then in that case we can do as below 
        
        kops edit ig nodes-us-west-2a --name {NAME}
        # accessing the instance group named as nodes-us-west-2a
        # this will provide the ouput as below 
        
        # Please edit the object below. Lines beginning with a '#' will be ignored,
        # and an empty file will abort the edit. If an error occurs while saving this file will be
        # reopened with the relevant failures.
        #
        apiVersion: kops.k8s.io/v1alpha2
        kind: InstanceGroup
        metadata:
        creationTimestamp: "2024-01-14T16:09:58Z"
        labels:
            kops.k8s.io/cluster: fleetman.k8s.local
        name: nodes-us-west-2a
        spec:
        image: 099720109477/ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-20231207
        machineType: t3.medium
        maxSize: 1 # here we can change the maxSize to 2
        minSize: 1  # here we can change the minSize to 2
        role: Node
        subnets:
        - us-west-2a

        # consider always changing the maxSize and minSize simulteniously
        # we can also optionally change the machineType here but not required
        # then we can hit `ctrl+O`  and `ctrl+X`(nano) or `ESC+:wq`(vi) editor to save those changes 
        
        # now when we run the command as below we will get the output as below 
        kops get ig --name ${NAME}
        # here ${NAME} comes from the env variable we set preveiously
        # here the output will be as below 
            NAME				ROLE		MACHINETYPE	MIN	MAX	ZONES
        control-plane-us-west-2a	ControlPlane	t3.medium	1	1	us-west-2a
        # here it telling the cluster that we need the "group of node with the role as ControlPlane" for the kops cluster
        nodes-us-west-2a		Node		t3.medium	2	2	us-west-2a # this will create the 2 EC2 instance as node 
        nodes-us-west-2b		Node		t3.medium	1	1	us-west-2b
        nodes-us-west-2c		Node		t3.medium	1	1	us-west-2c

        

    ```
