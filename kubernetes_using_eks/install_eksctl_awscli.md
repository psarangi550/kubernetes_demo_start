# installing eksctl and AWS CLI for amazon EKS cluster

- **install `eksctl` command line**
  
  - we can install `eksctl` command line args then we can do that using as below

    ```bash
        curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
        # here we will be usign the --silent along with the curl command hence even though to will install the eksctl command line nothing will be displayed
        
        # now we need to set the  eksctl to the /usr/local/bin to use it from any place
        # here we need to use the sudo access as below 
        sudo mv /tmp/eksctl /usr/local/bin

        # now we can run the command as below 
        eksctl 
        # the output in this case will be as below 
        The official CLI for Amazon EKS
        Usage: eksctl [command] [flags]

        Commands:
        eksctl anywhere                        EKS anywhere
        eksctl associate                       Associate resources with a cluster
        eksctl completion                      Generates shell completion scripts for bash, zsh or fish
        eksctl create                          Create resource(s)
        eksctl delete                          Delete resource(s)
        eksctl deregister                      Deregister a non-EKS cluster
        eksctl disassociate                    Disassociate resources from a cluster
        eksctl drain                           Drain resource(s)
        eksctl enable                          Enable features in a cluster
        eksctl get                             Get resource(s)
        eksctl help                            Help about any command
        eksctl info                            Output the version of eksctl, kubectl and OS info
        eksctl register                        Register a non-EKS cluster
        eksctl scale                           Scale resources(s)
        eksctl set                             Set values
        eksctl unset                           Unset values
        eksctl update                          Update resource(s)
        eksctl upgrade                         Upgrade resource(s)
        eksctl utils                           Various utils
        eksctl version                         Output the version of eksctl

        Common flags:
        -C, --color string   toggle colorized logs (valid options: true, false, fabulous) (default "true")
        -d, --dumpLogs       dump logs to disk on failure if set to true
        -h, --help           help for this command
        -v, --verbose int    set log level, use 0 to silence, 4 for debugging and 5 for debugging with AWS debug logging (default 3)

        Use 'eksctl [command] --help' for more information about a command.


        For detailed docs go to https://eksctl.io/


    ```

- `eksctl` `command line tool` will be issuing the `issue the command on our behalf` using the `AWS CLI`

- in `AWS` we can access `all of the AWS functionality` using the `command line tool` with the `aws` , which being `pre-installed` in the `Amazon Linux 2 instane` 

- the `eksctl` allow us to `issue the one simple command` , behind the scene `eksctl will issue dozens of aws cli command` in order to create the `Amazon EKS cluster`

- but there is a `problem relating to the aws cli version` with the `eksctl instance`

- here we can run the instance where the `aws cli` version is in `1.XX` then we need to update that to `AWS CLI version 2`

- we need to have the `aws cli version` as `1.18.186 or later or 2.0.25 version or later` installed  for the `eksctl to work`

- **updating the aws cli from version 1.XX to 2.XX**
  
  - we can update the `aws cli` to sync onto the `eksctl` command line args as below 
  
  - we can do that using the command as 
    
    ```bash
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        # installing the aws cli version2 as a zip file
        unzip awscliv2.zip
        # unzipping the zip file in here
        sudo ./aws/install
        # using the install.sh to install the awscliv2 with sudo permission

        # then we can check the aws --version which should c0ome with version2
        # if still showing as version 1 we need to exit and relogin to the bootstrap EC2 instance
        aws --version
        # here we can check the version as below 
        aws-cli/2.15.12 Python/3.11.6 Linux/6.5.0-14-generic exe/x86_64.ubuntu.22 prompt/off

    

    ```