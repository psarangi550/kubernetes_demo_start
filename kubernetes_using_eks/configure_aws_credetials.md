# configuring AWS credetials for EKS operation 

- we need to set some `credetials` for the `aws command line`

- at present if we want to perform `any operation using the aws command line` it will show that `not logged into the aws account`

- we can do anything on the `User Interface` that we can do `using the aws command line in this case`

- we can run the `all the cluster that being set up` by using the command as below

    ```bash
        aws eks list-clusters
        # this will list out all the aws eks cluster that we have at a place
        # the output will be as below 
        unable to locate credetials. You can configure credetials by running `aws configure`

    ```

- we need to set a `user` with the `AWS account` and `login as the same user` in the `aws CLI command line`

- we can goto the `AWS Mgmt Console` &rarr; `Services` &rarr; `IAM`

- `IAM` stands for the `identity and access management` which let us to `set multiple users` for the `AWS Account`

- we need to `create a group which has a set of permission` &rarr; `then we can create user` &rarr; `put the user into that group` , which will provide the `user the permission that we have defined in the group` , because `we don't want every  user` to have the `entire access over the AWS account`

- we need to `create` the `group with restruicted access` which have `set off sufficient permission to create the EKS cluster`

- we can `click on the group` &rarr; `provide a group name` as `eksGroup` &rarr; `next` &rarr; `here we have the entire list of predefined policy/permission which is required for the AWS `

- if we want to provide the `permssion` with the `EC2 full Access` then we can select it as `AmazonEC2FullAccess` , then the `user` which is the part of the `eksGroup` will have the access the `EC2 instances`

- now for the `EKS` `we don't have the permission` as `EKS full Access` , but we do have the `reference manual of eksctl` which provide the `bare minimum access needed for operating with the AWS EKS cluster using eksctl reference docs`

- for the `EKS cluster using eksctl` we need the below permission while creating the `eksGroup`

  - we do need the `AmazonEC2FullAccess` permssion should be `ticked` 
  
  - we also need the `IAMFullAccess` permission should be `ticked` , which can be `broader permission` which can also be `restricted` which is listed in [eksctl minimum Access Policy](https://eksctl.io/usage/minimum-iam-policies/)
  
  - we also need the `AWSCloudFormationFullAccess` permission should be `ticked`  as well

- if we go to the `eksGroup` `user group` and select the `permission policy` and select the `inline policies access`

- we also have to provide the `inline policies` &rarr; `custom policy` &rarr; provide the below `JSON Policy` and name it as `EksAllAccess` as below 

    ```json
        EksAllAccess
        =============
        {
        "Version": "2012-10-17",
        "Statement": [
            {
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
            },
            {
            "Action": [
                "ssm:GetParameter",
                "ssm:GetParameters"
            ],
            "Resource": "*",
            "Effect": "Allow"
            }
        ]
        }
    
    ```
  
- we can also reference the `ekctl minimum IAM access` by going to the link as [EKS minimal IAM access ](https://eksctl.io/usage/minimum-iam-policies/) , if we are running a `professional EKS cluster` then we can want to be absolutely secure then we can use the link to `configure the minimum IAM access policy`

- if we are using the `EksAllAccess` policy as per the document  then we have to define as below

    ```json
        EksAllAccess
        ============
        {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "eks:*",
                "Resource": "*"
            },
            {
                "Action": [
                    "ssm:GetParameter",
                    "ssm:GetParameters"
                ],
                "Resource": [
                    "arn:aws:ssm:*:<account_id>:parameter/aws/*", // here we want to provide the <account_id> of the AWS Access
                    "arn:aws:ssm:*::parameter/aws/*"
                ],
                "Effect": "Allow"
            },
            {
                "Action": [
                "kms:CreateGrant",
                "kms:DescribeKey"
                ],
                "Resource": "*",
                "Effect": "Allow"
            },
            {
                "Action": [
                "logs:PutRetentionPolicy"
                ],
                "Resource": "*",
                "Effect": "Allow"
            }        
        ]
    }

    ```

- we can also use the `IAMLimited Access` as below 

    ```json
        IamLimitedAccess
        ================
            {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "iam:CreateInstanceProfile",
                    "iam:DeleteInstanceProfile",
                    "iam:GetInstanceProfile",
                    "iam:RemoveRoleFromInstanceProfile",
                    "iam:GetRole",
                    "iam:CreateRole",
                    "iam:DeleteRole",
                    "iam:AttachRolePolicy",
                    "iam:PutRolePolicy",
                    "iam:AddRoleToInstanceProfile",
                    "iam:ListInstanceProfilesForRole",
                    "iam:PassRole",
                    "iam:DetachRolePolicy",
                    "iam:DeleteRolePolicy",
                    "iam:GetRolePolicy",
                    "iam:GetOpenIDConnectProvider",
                    "iam:CreateOpenIDConnectProvider",
                    "iam:DeleteOpenIDConnectProvider",
                    "iam:TagOpenIDConnectProvider",
                    "iam:ListAttachedRolePolicies",
                    "iam:TagRole",
                    "iam:GetPolicy",
                    "iam:CreatePolicy",
                    "iam:DeletePolicy",
                    "iam:ListPolicyVersions"
                ],
                "Resource": [
                    "arn:aws:iam::<account_id>:instance-profile/eksctl-*", // here need to provide the <account_id> of AWS
                    "arn:aws:iam::<account_id>:role/eksctl-*", // here need to provide the <account_id> of AWS
                    "arn:aws:iam::<account_id>:policy/eksctl-*", // here need to provide the <account_id> of AWS
                    "arn:aws:iam::<account_id>:oidc-provider/*", // here need to provide the <account_id> of AWS
                    "arn:aws:iam::<account_id>:role/aws-service-role/eks-nodegroup.amazonaws.com/AWSServiceRoleForAmazonEKSNodegroup", // here need to provide the <account_id> of AWS
                    "arn:aws:iam::<account_id>:role/eksctl-managed-*" // here need to provide the <account_id> of AWS
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iam:GetRole"
                ],
                "Resource": [
                    "arn:aws:iam::<account_id>:role/*" // here need to provide the <account_id> of AWS
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iam:CreateServiceLinkedRole"
                ],
                "Resource": "*",
                "Condition": {
                    "StringEquals": {
                        "iam:AWSServiceName": [
                            "eks.amazonaws.com",
                            "eks-nodegroup.amazonaws.com",
                            "eks-fargate.amazonaws.com"
                        ]
                    }
                }
            }
        ]
    }
    
    ```

- but as we are using the `IAMFullAccess` to create the `EKS cluster` , which can be a security risk `as the EKS cluster administrator` have the access to the `IAM user, group and policy`

- then we can do is `create the IAM user and give it a name` &rarr; `attach the user to the UserGroup` as `programatic Access` , so that `user` have the permission to the `groups`

- this will provide the below things
  
  - `ACESS_KEY_ID` which behave as the `Username`
  
  - `SECRET_ACCESS_KEY` which will be `Secret Access Key` 
  
- we can perform the below action as below 

    ```bash
        aws configure
        # configure the aws cli terminal and below will be the output for the same
        AWS Access Key ID [****************PWF5]: 
        AWS Secret Access Key [****************J7lo]: 
        Default region name [us-west-2]: eu-central-1 # here we can find by going to the EC2 Dashboard instance
        Default output format [json]:
    
    ```

- now when we run the command as below to list out the `eks cluster` then we can get by using the command as below 

    ```bash
        aws eks list-clusters
        # this command will list out all the cluster that we have created in the AWS region
        # as we don't have any cluster created then we can see those info
        {
        "clusters": []
        }
 
    ```

- we can also create a `new cluster` using the `aws eks command` which can be `nighmare to do as we have to use the multiple low level command` , hence we will be using the `eksctl command line` in order to `create the cluster` which will call the `aws eks command` multiple times behind the scene

- but before to that we need to install the `kubectl` to use the `kubectl to manage the cluster` like `minikube local cluster`