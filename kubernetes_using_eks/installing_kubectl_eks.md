# installing Kubectl for AWS EKS cluster

- installing `kubectl` will be exactly same as the `kuctl as before` , if we are installing on `linux`

- we need to `version of kubectl` match the `version of kubernetes we are using it in the EKS cluster` , normally `kubernetes version in Amazon EKS cluster will be little behind the kubectl current latest version` hence there mkight be chance of `difference in the kubectl version and kubernetes version` , we don't want that 

- we need to find the `default version of kubernetes in amazon EKS cluster` for that 
  
  - if we goto the `AWS Mgmt console` &rarr; `create cluster` &rarr; `the default version for that being 1.28`
  
- here we need to install the `kubectl` with the version as `1.28` which is the same exact version of the `kubernetes ion AWS EKS cluster`

- we can do that using the command as below 

    ```bash
        export RELEASE=1.28.0
        # here while exporting the RELEASE and setting it as the environment variable then we need to provide the minor version as well
        # hence if the version showing as 1,28 we need to mention as 1.28.0
        curl -LO https://storage.googleapis.com/kubernetes-release/release/v$RELEASE/bin/linux/amd64/kubectl
        # using the curl command in order to use the kubectl folder
        chmod +x ./kubectl
        # changing the kubectl folder permission in here
        sudo mv ./kubectl /usr/local/bin/kubectl
        # here we will be mv command to add the kubectl to the /usr/local/bin folder
        # so that after adding it onto the /usr/local/bin/kubectl we can use the kubectl command 
        kubectl version --client
        # checking the kubectl client version 
        # as kubectl only the client tool and kubernetes will be in AWS EKS Server hence we are using the command as --client
        # as we don't have the AWS EKS kubernetes cluster Server not running hence we  are using the --client command here 
    

    ```

- here i am using the `kubectl with the version as 1.26.0` hence the output will be as below 

    ```bash
        kubectl version --client
        # checking the kubectl client version 
        # as kubectl only the client tool and kubernetes will be in AWS EKS Server hence we are using the command as --client
        # as we don't have the AWS EKS kubernetes cluster Server not running hence we  are using the --client command here 
        # the output here is as below
        Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.1", GitCommit:"8f94681cd294aa8cfd3407b8191f6c70214973a4", GitTreeState:"clean", BuildDate:"2023-01-18T15:58:16Z", GoVersion:"go1.19.5", Compiler:"gc", Platform:"linux/amd64"}
        Kustomize Version: v4.5.7
    
    ```
