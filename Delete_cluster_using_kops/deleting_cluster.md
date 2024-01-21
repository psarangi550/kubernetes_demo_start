# Deleting the kops kubernetes cluster

- we still have to do a couple of things with the `AWS Kops kubernetes cluster` such as `monitoring and logging the kubernetes cluster`

- if we `delete` the `kops cluster` , then it will be easy to `restart the kops kubernetes cluster` as well on the `later date`

- we can delete the cluster ` through the command line` we can don't have to do that using the `manual process of deleting`

- here currently we have `2 load balancer` , which will cost around `$18 per month` , which been billed hourly as `$0.025 per hour`

- if we go `manually` and then `delete the EC2 instances(4 node i.e 1 workernode/controlplain and 3 worker node)` then due to the `Auto Scaling group` these instances will be going to `recrested automatically`

- we can manually delete the `Auto Scaling group` and followed by `deleting the EC2 instances associated with it` , but `this is not a recomended approach` , there might be chances that we `leave something accidently`

- the correct way of `deleting the kops kubernetes cluster` `by the kops command line` as below 

- the `delete command` depends on `2 environment variable` i.e `NAME` and `KOPS_STATE_STORE` that we have set earlier 
    
    ```bash
        export NAME = fleetman.k8s.local
        export KOPS_STATE_STORE=s3://pratik-kops-12345
        # here we are setting the environment variable as NAME and KOPS_STATE_STORE
        # once the environment variable been set then we can issue the kops delete cluster command in here
        
        kops delete cluster ${NAME} --yes 
        # deleting the kops cluster using the kops delete cluster ${NAME} --name
        # here --yes will perform the command else it will perform the dry-run in that case
        # this script can take upto 2 mins to delete all the cluster
        # don't force yourself to delete the >EC2 instances manually else the script will not work properly and resources will be left behind

    
    
    
    
    ```

- `one more important note` we can also use the `history` command to fetch which `export command that we used` as below

    ```bash
        history | grep STATE
        # here from the set of history fetching where the value set to STATE
        # here we can get the output in this case as below 
        # the output will be as below
        1524  export KOPS_STATE_STORENAME=s3://pratik-kops-12345
        1527  export KOPS_STATE_STORE=s3://pratik-kops-12345
        1540  export KOPS_STATE_STORE=s3://pratik-kops-12345
        1619  export KOPS_STATE_STORE=s3://pratik-kops-12345
        1643  export KOPS_STATE_STORE=s3://pratik-kops-12345
        1753  export KOPS_STATE_STORE=s3://pratik-kops-12345
        1867  export KOPS_STATE_STORE=s3://pratik-kops-12345
        1946  export KOPS_STATE_STORE=s3://pratik-kops-12345
        1967  export KOPS_STATE_STORE=s3://pratik-kops-12345
        1987  export KOPS_STATE_STORE=s3://pratik-kops-12345
        1994  export KOPS_STATE_STORE=s3://pratik-kops-12345
        2009  export KOPS_STATE_STORE=s3://pratik-kops-12345
        2089  export KOPS_STATE_STORE=s3://pratik-kops-12345
        2098  export KOPS_STATE_STORE=s3://pratik-kops-12345
        2238  history | grep STATE

        # here we can reissue th command based on the line as below
        !<command number that we want to execute>
        !1524
        # this will bring back the command in that case
        export KOPS_STATE_STORENAME=s3://pratik-kops-12345
        # here executing the command line as 1524in this case 

        history | grep NAME
        # here from the set of history fetching where the value set to NAME
        # the output will be as below
        910  echo AKCpBrvkHKRiSn7LDz4aNfu2yzKJxHTv13FMCCy7RnX8esoYkigBh6nDh83E1mCkUqLLTd7Vy | jfrog rt docker-pull captain.rtf.siemens.net/siemens-docker-eagle-local/de_ds_tcp_common/dotnet:sdk-6.0 siemens-docker-eagle-local/de_ds_tcp_common/dotnet --url=https://captain.rtf.siemens.net/ --username=<YOUR_USERNAME> --password-stdin\n
        1521  kops create cluster \\n        --name=${NAME} \\n        --cloud=aws \\n        --zones=us-west-2a,us-west-2b,us-west-2c,us-west-2d \\n        --discovery-store=s3://pratik-kops-12345/${NAME}/discovery
        1523  export NAME=fleetman.k8s.local
        1524  export KOPS_STATE_STORENAME=s3://pratik-kops-12345
        1526  kops create cluster \\n        --name=${NAME} \\n        --cloud=aws \\n        --zones=us-west-2a,us-west-2b,us-west-2c,us-west-2d \\n        --discovery-store=s3://pratik-kops-12345/${NAME}/discovery
        1529  kops create cluster \\n        --name=${NAME} \\n        --cloud=aws \\n        --zones=us-west-2a,us-west-2b,us-west-2c,us-west-2d \\n        --discovery-store=s3://pratik-kops-12345/${NAME}/discovery
        1532  kops edit cluster --name ${NAME}
        1533  kops edit cluster --name ${NAME} | clipboard
        1536  kops edit cluster --name ${NAME}
        1538  kops edit cluster --name ${NAME}
        1539  export NAME=fleetman.k8s.local
        1542  kops edit cluster --name ${NAME}
        1543  kops edit cluster --name ${NAME} | clipboard
        1546  kops edit cluster --name ${NAME} | clipboard
        1547  kops edit cluster --name ${NAME}
        1548  kops edit cluster --name ${NAME} >> /dev/vull
        1549  sudo kops edit cluster --name ${NAME} >> /dev/vull
        1550  sudo kops edit cluster --name ${NAME} > /dev/vull
        1553  kops edit cluster --name ${NAME}
        1555  kops edit cluster --name ${NAME}
        1558  kops edit cluster --name ${NAME}
        1560  kops edit cluster --name ${NAME}
        1562  kops get ip --name ${NAME}
        1564  kops edit cluster --name ${NAME}
        1567  export NAME=fleetman.k8s.local
        1569  kops get ig --name ${NAME}
        1570  kops create cluster \\n        --name=${NAME} \\n        --cloud=aws \\n        --zones=us-west-2a,us-west-2b,us-west-2c \\n        --discovery-store=s3://pratik-kops-12345/${NAME}/discovery
        1571  kops edit cluster --name ${NAME}
        1573  kops edit cluster --name ${NAME}
        1576  kops edit cluster --name ${NAME}
        1578  kops get ig --name ${NAME}
        1579  kops delete cluster --name ${NAME} --yes\n
        1581  kops create cluster \\n        --name=${NAME} \\n        --cloud=aws \\n        --zones=us-west-2a,us-west-2b,us-west-2c \\n        --discovery-store=s3://pratik-kops-12345/${NAME}/discovery
        1583  kops get ig --name ${NAME}
        1595  echo $NAME
        1597  kops get ig --name ${NAME} | clipboard
        1598  kops edit ig --name ${NAME}
        1600  kops get ig --name ${NAME}
        1601  kops edit ig nodes-us-west-2a --name ${NAME}
        1618  export NAME=fleetman.k8s.local
        1621  kops edit cluster --name ${NAME}
        1623  kops update cluster ${NAME} 
        1625  kops update cluster ${NAME} --yes --admin
        1628  kops update cluster ${NAME} --yes --admin=87600h
        1633  aws iam get-role --role-name ROLE_NAME\n
        1638  kops export kubecfg --admin=87600h --name ${NAME}
        1640  export NAME=fleetman.k8s.local
        1642  kops export kubecfg --admin=87600h --name ${NAME}
        1645  kops export kubecfg --admin=87600h --name ${NAME}
        1651  kops validate cluster --name ${NAME}
        1653  kops validate cluster --name ${NAME}
        1655  kops validate cluster --name ${NAME}
        1659  kops validate cluster --name ${NAME}
        1660  kops export kubecfg ${NAME} --admin
        1662  kops validate cluster --name ${NAME}
        1670  kops validate cluster --name ${NAME}
        1673  kops delete cluster --name ${NAME} --yes
        1675  kops update cluster --name ${NAME} --yes --admin=87600h
        1677  kops create cluster \\n        -- name=${NAME}\n        -- zones=us-west-2a,us-west-2b,us-west-2c \\n        --discovery-store=S3://pratik-kops-12345/${NAME}/discovery
        1678  kops create cluster \\n        -- name=${NAME}\n        -- zones us-west-2a,us-west-2b,us-west-2c \\n        --discovery-store=S3://pratik-kops-12345/${NAME}/discovery
        1680  kops create cluster \\n    --name=${NAME} \\n    --zones=us-west-2a,us-west-2b,us-west-2c \\n    --discovery-store=s3://pratik-kops-12345/${NAME}/discovery\n
        1682  kops update cluster --name ${NAME} --yes --admin=87600h
        1689  kops validate cluster --name ${NAME}
        1691  kops export kubecfg ${NAME} --admin=87600h
        1693  kops validate cluster --name ${NAME}
        1695  kops validate cluster --name ${NAME}
        1717  kops validate cluster --name ${NAME}
        1722  kops validate cluster --name ${NAME}
        1727  kops validate cluster --name ${NAME}
        1747  kops delete cluster --name ${NAME} --yes
        1752  export NAME=fleetman.k8s.local
        1755  kops create cluster \\n        -- name=${NAME}\n        -- zones=us-west-2a,us-west-2b,us-west-2c \\n        --discovery-store=S3://pratik-kops-12345/${NAME}/discovery
        1756  kops create cluster \\n        -- name=${NAME} \\n        -- zones=us-west-2a,us-west-2b,us-west-2c \\n        --discovery-store=S3://pratik-kops-12345/${NAME}/discovery
        1758  kops create cluster \\n    --name=${NAME} \\n    --zones=us-west-2a,us-west-2b,us-west-2c \\n    --discovery-store=s3://pratik-kops-12345/${NAME}/discovery\n
        1760  kops create cluster \\n    --name=${NAME} \\n    --zones=us-west-2a,us-west-2b,us-west-2c \\n    --discovery-store=s3://pratik-kops-12345/${NAME}/discovery\n
        1762  kops delete cluster --name ${NAME} --yes
        1764  kops create cluster \\n    --name=${NAME} \\n    --zones=us-west-2a,us-west-2b,us-west-2c \\n    --discovery-store=s3://pratik-kops-12345/${NAME}/discovery\n
        1766  kops update cluster --name ${NAME} --yes --admin=87600h
        1768  kops export kubecfg ${NAME} --admin=87600h
        1770  kops validate cluster --name ${NAME}
        1772  kops validate cluster --name ${NAME}
        1774  kops validate cluster --name ${NAME}
        1776  kops validate cluster --name ${NAME}
        1849  kops validate cluster --name ${NAME}
        1853  kops validate cluster --name ${NAME} | clipboard
        1854  kops validate cluster --name ${NAME}
        1860  kops validate cluster --name ${NAME}
        1865  kops validate cluster --name ${NAME}
        1866  export NAME=fleetman.k8s.local
        1869  export NAME=fleetman.k8s.local
        1870  kops validate cluster --name ${NAME}
        1936  kops validate cluster --name ${NAME}
        1941  kops validate cluster --name ${NAME}
        1945  export NAME=fleetman.k8s.local
        1948  kops validate cluster --name ${NAME}
        1964  kops validate cluster --name ${NAME}
        1966  export NAME=fleetman.k8s.local
        1969  kops validate cluster --name ${NAME}
        1973  kops validate cluster --name ${NAME}
        1979  kops validate cluster --name ${NAME}
        1982  kops update cluster --name ${NAME} --yes --admin=87600h
        1983  kops export kubecfg ${NAME} --admin=87600h
        1985  kops validate cluster --name ${NAME}
        1990  kops validate cluster --name ${NAME}
        1991  kops get cluster ${NAME} -o yaml\n
        1993  export NAME=fleetman.k8s.local
        1998  kops validate cluster --name ${NAME}
        1999  kops export kubecfg ${NAME} --admin=87600h
        2000  kops validate cluster --name ${NAME}
        2002  kops validate cluster --name ${NAME}
        2005  kops delete cluster --name ${NAME} --yes
        2007  kops update cluster --name ${NAME} --yes --admin=87600h
        2008  export NAME=fleetman.k8s.local
        2010  kops update cluster --name ${NAME} --yes --admin=87600h
        2012  kops create cluster \\n    --name=${NAME} \\n    --zones=us-west-2a,us-west-2b,us-west-2c \\n    --discovery-store=s3://pratik-kops-12345/${NAME}/discovery\n
        2014  kops update cluster --name ${NAME} --yes --admin=87600h
        2016  kops validate cluster --name ${NAME}
        2019  kops validate cluster --name ${NAME}
        2021  kops export kubecfg ${NAME} --admin=87600h
        2022  kops validate cluster --name ${NAME}
        2023  kops get cluster ${NAME} -o json\n
        2030  kops get cluster ${NAME} -o json\n
        2031  kops get cluster ${NAME} -o json | grep api\n 
        2056  kubectl config set-cluster ${NAME} --insecure-skip-tls-verify=true
        2059  kubectl config set-cluster ${NAME} --insecure-skip-tls-verify=true
        2088  export NAME=fleetman.k8s.local
        2094  ecgho $NAME
        2095  echo $NAME
        2097  export NAME=fleetman.k8s.local
        2241  export KOPS_STATE_STORENAME=s3://pratik-kops-12345
        2242  history | grep NAME

        # then here we can use the command number as 29-097 to issue the command as below
        !2097
        # this will show the output as below
        export NAME=fleetman.k8s.local
        # if we hit enter then it will save the environment variable in this case


    ```

- once after the `kops delete` command been issued we can see that `AWS Mgmt console` &rarr; `EC2` &rarr; `running instances should be terminated`

- once after the `kops delete` command been issued we can see that `AWS Mgmt console` &rarr; `EC2` &rarr; `the Load Balancer been deleted`

- after the `kops delete` command been issued we can see that `AWS Mgmt console` &rarr; `EC2` &rarr; `Auto Scaling group also deleted`

- - after the `kops delete` command been issued we can see that `AWS Mgmt console` &rarr; `EC2` &rarr; `EBS Volume` and we can see the `EBS volume and the mongoDB volume which been set with the reclaimPoplicy as delete` will be `going to be get deleted`

- after the `kops delete` command been issued we can see that s2 bucket has no `resources` associated with it 

- One thing in `AWS` we need to keep in mind that `even though we have the AWS instance state sho9wing as stopped` , but as there is `EBS volume` associated with it hence it will be going to incur charges which will be `few cents per month`

- when we mark the `EC2 instance` as `terminated` then it will also going to `delete the volume` which will stop incuring charges even though its few cents month as the Associated Volume being deleted

- if we leave something off then `goto the mu account page` &rarr; `close Account` , if not closing the check the `billing from the amazon account`

- 