# introduction to Kops 

- we can choose any `availabilty zone` that we want to create the `AWS Resource`

- in order to start `we need to install kubernetes` onto `AWS cloud Account` , we can do that `manually` by spinning `multiple instances` and `create multiple resources` and `wire them together` , which can considered as `notoriously difficult process`

- there are various `projects` which can help in `automate the setting up` of the `kubernetes cluster on cloud without those hassle` 

- one of the popular option being `kops i.e kubernetes operation` which is present as `github repo` as [Kops Github Page](https://github.com/kubernetes/kops) which is the official repo of the `kubernetes` account

- its actually running the `production grade kubernetes cluster` using `kops kubernetes cloud cluster` , this is the easiest  way of `running production grade kubernetes cluster`

- the first implementation of `kops` builds around the `AWS` , but currently  kops provide support for 
  
  - `AWS`

  - `GCE`
  
  - `DigitalOcean` 
  
  - `Hetzner` 
  
  - `OpenStack` 
  
  - `Azure` 

- we can use the `kubernetes cluster` on these `mentioned platform` , but we need a `different tools` to `bootstrap these requirement`

- we can goto link as [kops installation page](https://kops.sigs.k8s.io/getting_started/install/) in order to install `kubernetes cluster using kops`

- now we will be building the `AWS Resource` which will be billed by `hours`

- we can `install kops CLI` `on linux` using the below commands 
  
- we can install `kops` onto the `local development machine` and `kops will then speak to the EC2 instance` to start the `starts the worker nodes and master nodes`

- but the `recomended way` to use the `bootstrap EC2 instance` rather than using `local development machine`

- here we can select the `Amazon Linux2 AMI (HVM)` volume type being mentioned , with the `t2.nano` instance type which comes with `1 vCPU` in this case with `0.5GB memory`

- we don't have to change much details which creating the `EC2 instance` for the `bootstrap mode` , we just have to add the `Tag` as `Name:Bootstrap` , with `8GB RAM and 30GB` storage 

- here by default when we are launching the instance the `port 22 going to be get open` , we can allow the `IP` as your `public Ip rather than allowing all the Ip` with the source as `MyIP with <ip will get populated>` rather than using the `custom 0.0.0.0/0`

- we will `generate a new keypair` for the `AWS EC2 instance` in order to `login/access to the bootstrap EC2 instance`

- but we need to keep in mind that `if these keypair will be displayed once hence we need to download the *.pem file`

- keep in mind that `if we loose the <*.pem keypair file>` then we `will not be able to generate the keypair again for the EC2 instance` 

- we need to see the `EC2 instance` instance state as `running` and the `public ip and public dns will be shown for the same`

- we need to set the `permission` as `chmod go-rwx <*.pem keypair file>` 

- for logging in we need to use the command as `ssh -i <*.pem keypair file> ec2-user@<public ip of the EC2 instance>`

- incase of cygwin we need to use the `cygdrive` rather than `mnt` for accessing the `any Drives` 

- here this is the `only EC2 instance` we need to `setup` , once `kops` installed then `kops will perform the heavy lifting for us`

- we can download the `latest version of the Kops CLI` using the option as below 
    
    ```bash
        # this will help in installing the kops binary which is for the latest version from github
        curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
        chmod +x ./kops
        sudo mv ./kops /usr/local/bin/
        # here moving it onto the /usr/local/bin directory in order to use the cli directly
    ```

- now if we want to validate the `kops` getting installed correctly then we can use the command as below 

    
    ```bash
        kops
        # here we will betting below output
        kOps is Kubernetes Operations.

        kOps is the easiest way to get a production grade Kubernetes cluster up and running. We like to think of it as kubectl for clusters.

        kOps helps you create, destroy, upgrade and maintain production-grade, highly available, Kubernetes clusters from the command line. AWS (Amazon Web Services) is currently officially supported, with Digital Ocean and OpenStack in beta support.

        Usage:
        kops [command]

        Available Commands:
        completion     Generate the autocompletion script for the specified shell
        create         Create a resource by command line, filename or stdin.
        delete         Delete clusters, instancegroups, instances, and secrets.
        distrust       Distrust keypairs.
        edit           Edit clusters and other resources.
        export         Export configuration.
        get            Get one or many resources.
        help           Help about any command
        promote        Promote a resource.
        replace        Replace cluster resources.
        rolling-update Rolling update a cluster.
        toolbox        Miscellaneous, experimental, or infrequently used commands.
        trust          Trust keypairs.
        update         Update a cluster.
        upgrade        Upgrade a kubernetes cluster.
        validate       Validate a kOps cluster.
        version        Print the kOps version information.

        Flags:
            --config string   yaml config file (default is $HOME/.kops.yaml)
        -h, --help            help for kops
            --name string     Name of cluster. Overrides KOPS_CLUSTER_NAME environment variable
            --state string    Location of state storage (kops 'config' file). Overrides KOPS_STATE_STORE environment variable
        -v, --v Level         number for the log level verbosity

        Use "kops [command] --help" for more information about a command.

    
    ```
- **Setting up Other Dependency**

- then we nee to install `kubectl` which will be used to fire the `kubernetes command against the kubernetes cluster` that we will be creating 

- we can install that by using the command as below `in other dependency`

    ```bash
        # here we will be using the kubectl exactly as we are using before for the kops kubernetes cluster on AWS
        curl -Lo kubectl https://dl.k8s.io/release/$(curl -s -L https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
        chmod +x ./kubectl
        sudo mv ./kubectl /usr/local/bin/kubectl
        # here moving it onto the /usr/local/bin/kubectl directory in order to use the kubectl cli directly    
    ```

- we can check the `kubectl` by using the command as below 

    ```bash
        kubectl version
        # fetching the kubernetes version in here
        WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
        Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-18T15:58:16Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
        Kustomize Version: v4.5.7
        The connection to the server localhost:8080 was refused - did you specify the right host or port?
        # the connection to the server being refused as we have not created the kubernetes cluster to which kubectl try to connect 
        # which is in case of minikube being service/kubernetes against which the API call supposed to be made
        # here we can also run the command as below inorder to validate 
        {
        "clientVersion": {
            "major": "1",
            "minor": "26",
            "gitVersion": "v1.26.1",
            "gitCommit": "8f94681cd294aa8cfd3407b8191f6c70214973a4",
            "gitTreeState": "clean",
            "buildDate": "2023-01-18T15:58:16Z",
            "goVersion": "go1.19.5",
            "compiler": "gc",
            "platform": "linux/amd64"
        },
        "kustomizeVersion": "v4.5.7"
        }
 
    ```

- **Setting up the Environment**
  
  - here we need to setup a `AWS User` called with name `kops` , which should have the `appropriate set off permission ` to perform its action 
  
  - for the `kops` we need the below `permission` such as 
    
      - `AmazonEC2FullAccess` :- `Access to EC2 with full Access`
    
      - `AmazonRoute53FullAccess` :-` Access to Route53 with Full Access`
    
      - `AmazonS3FullAccess` :- `Access to S3 bucket with full Access`
    
      - `IAMFullAccess` :- `Access to IAM with full Access`
    
      - `AmazonVPCFullAccess` :- `Access to VPC with full user  Access`
    
      

  - the `kops` provide the `script` to create the `user groups` and `permission` and `create and add the user` to the `usergroup having the permission`
  
  
  - there are 2 more `permission added here` which are as 
    
    -  `AmazonSQSFullAccess` :- `Amazon SQS service with full access`
    
    - `AmazonEventBridgeFullAccess` :- `Access to Amazon Event Bridge with full access `
  
  
  - if `we have only the below permission then we can create the cluster but we can'rt able to delete the cluster`
    
     - `AmazonEC2FullAccess` :- `Access to EC2 with full Access`
    
      - `AmazonRoute53FullAccess` :-` Access to Route53 with Full Access`
    
      - `AmazonS3FullAccess` :- `Access to S3 bucket with full Access`
    
      - `IAMFullAccess` :- `Access to IAM with full Access`
    
      - `AmazonVPCFullAccess` :- `Access to VPC with full user  Access`
  
- we can `set these permission ` using the  both `User Interface` as well as the `Script`

- For `User Interface`  :-
  
  - we need the `Identity and Access Management (IAM)` service in here as , which will be help in managing the `user and permision in AWS` in this case 
    
  - we can goto the `User Group`  &rarr;  `create Group` &rarr; `Name them as kops`
  
  - we then goto the `Attach Permission Policies` , Even though it show as `optional` , we need to provide the `permission policies over here`
  
  - we can search for the `permission policy` by searching through the same and `check the box` here for the `below permission` as below    
  
      - `AmazonEC2FullAccess` :- `Access to EC2 with full Access`
    
      - `AmazonRoute53FullAccess` :-` Access to Route53 with Full Access`
    
      - `AmazonS3FullAccess` :- `Access to S3 bucket with full Access`
    
      - `IAMFullAccess` :- `Access to IAM with full Access`
    
      - `AmazonVPCFullAccess` :- `Access to VPC with full user  Access`
      
      - `AmazonSQSFullAccess` :- `Amazon SQS service with full access`
      
      - `AmazonEventBridgeFullAccess` :- `Access to Amazon Event Bridge with full access ` 
  
  - then we can goto the `create group option shown down below` &rarr; `which will create the user group`
  
  - here also if we goto the `kops usergroup` then goto the `permission` then we can see all the `permission listed down below`
  
  - if we want to add `more permission` then we can select the `kops usergroup`  and goto `permission` and select `add permission`
  
  - once the `usergroup with the name kop been created` and `associated with the permission over in this case`
  
  - now we need to create the `AWS User` , hence click on the `User` and `Create User` &rarr; `Username being kops` &rarr; `Access Type :- Programmatic Access`
  
  -  now we can select the `Next Permission` &rarr; `Add User to group` &rarr;`select the User Group as kops that we have created` &rarr; `click on review` &rarr; `review that you have the correct usergroup selected for the User` &rarr; `create user`
  
  - this will provide the `AWS Access Key ID` and `AWS Security Access Group` in this case
  
  - we need to keep a not of that `AWS Access Key ID` and `AWS Security Access Group` , don't share them 
  
  - if we loose the `AWS Access Key ID` and `AWS Security Access Group` , then we can create the `New User and associate with the UserGroup & permission` and `we can use the new set of AWS Access Key ID and AWS Security Access Group`
  
- now we need to use the `aws cli tool` in order to `perform action against the AWS account for managing resource`
  
- we can create `EC2 instance` using the `aws cli` as well , but here we will just be `configuring the AWS CLI` so that `kops can use those credetional to create the EC2 instance node for kubernetes cluster`

-    