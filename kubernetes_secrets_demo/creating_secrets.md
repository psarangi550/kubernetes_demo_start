# creating the secrets in kubernetes 

- we can define the `configMap` for `injecting the data inside multiple POD container image from configMap` , which can be done in `2 ways`
  
  - using the `environmental variable` as `reference variable` to the `value for the configMap`
  
  - using the `configMap` as `volume` which can be `mount to the POD container image as regular file / folder inside the image filesystem`  

- there is a `very closely related topic of configMap` known as `secrets` in `kubernetes`

- we have already used the `secrets` inside the `kubernetes` while using the `alertManager` in the `monitoring section`

- **What is a Kubernetes secrets**
  
  - we can find the reference docs for the `kubernetes secrets` in here as below 
  
  - [kubernetes secrets reference docs](https://kubernetes.io/docs/concepts/configuration/secret/) 
  
  - A `kubernetes secrets` allow us to `store and manages` the `sensitive info` such as 
    
    - `password`
    
    - `OAuth token`
    
    - `ssh keys`
    
  - `putting these info` inside the `secret` is `safer` and `more flexible` than `putting it directly inside the POD definition or container image`
  
  - it will `more flexible` to put the `data into secrets` in the `kubernetes secrets` rather than putting to the `POD container image` in `hardcoded way`
  
  - but this does not cut it for us , because we saw the `configMap` we have put the `data into the configMap` rather than `hardcoding the info inside the POD container image and deriving new image for each change` , hence i consider the `secret is a strange thing in kubernetes`
  

- **How to create a New Secret in kubernetes**     

- at the first glance , the `kubernetes secrets looks exactly same as the configMap`

- for example :- if we want to `store our AWS Credetials` as `kubernetes secrets` 

  - we can then define the `kubernetes secrets` as below with the name as `aws-credetials.yml`

    ```yaml
        aws-credetials.yml
        ==================
        apiVersion: v1 # here the secrets also belong to the core group hence mentioned as apiVersion: v1
        kind: Secret # here the type of object we will be using is of type Secret
        metadata: # defining the name of the kubernetes secrets and what namespace we want to put that in 
            name: aws-credetials # name of the kubernetes secrets
            namespace: default  # namespace where we wantr to put it in
        data: # inside the data block we can define the key and value pair here 
            accessKey: 1234567890
            sectretKey: SECRET1234


    ```

- now if we try to `deploy the changes onto the cluster` by `applying the changes` as below 

    
    ```bash
        kubectl apply -f aws-credetials.yml
        # deploy the changes onto the cluster by applying the changes
        # here we wil get the error in this case having the base64codec Issue
        # the output will be as below
        Error from server (BadRequest): error when creating "aws-configure.yml": Secret in version "v1" cannot be handled as a Secret: illegal base64 data at input byte 8
        
        # here we will get the error as secret can't be handled as secret as there is invalid input
        # here we can see the illegal base64 data at input byte 8

    ```

- we can `crate the secrets in a file first` in `JSON or in YAML`

- the `secret` contains `2 maps` i.e 
  
  - `data`
  
  - `stringData` 

- here as we have the `field` as `data` inside the `kubernetes secrets yml` i.e `aws-credetials.yml` , but the `requirement` being the `value of the key of the data inside the kubernetes secrets` should be in `base64 encoding format`

- **what is base64 encoding** :-
  
  - the `base64 encoding` convert `any binary data` into the `text data` using the `ASCII format`
  
  - this will be useful if we want to `transfer` the `image file` through a `protocol` which support only `text based format`
  
  - `base64` used most on the `world wide web` where it uses the `ability to embed the image file or other binary asset` inside the `textual asset` such as `HTML` and `CSS`
  
  - we can convert the `textual data` through `base64 encoding` as well , even though `its intended` for the `binary data`
  
  - we can convert that as below

    ```bash
        echo "pratik" | base64
        # this will provide the base64 encoded text of the textual data we provided
        # hence the output will be as below
        cHJhdGlrCg==

        # hence we can convert the accessKey into the base64 encoded format as below 
        echo 1234567890 | base64 
        # the output will be as below in this case
        MTIzNDU2Nzg5MAo=

        #similarly we can convert the secretKey into the base64 encoded format as below
        echo "SECRET1234" | base64 
        # the output will be as below 
        U0VDUkVUMTIzNAo=

        # base64 mostly ends with the = symbol at the end inthis case


    ```

- now we can mention the `aws-configure.yml` with the `base64 encoded value` as below

    ```yaml
        aws-credetials.yml
        ==================
        apiVersion: v1 # here the secrets also belong to the core group hence mentioned as apiVersion: v1
        kind: Secret # here the type of object we will be using is of type Secret
        metadata: # defining the name of the kubernetes secrets and what namespace we want to put that in 
            name: aws-credetials # name of the kubernetes secrets
            namespace: default  # namespace where we wantr to put it in
        data: # inside the data block we can define the key and value pair here 
            accessKey: "MTIzNDU2Nzg5MAo="
            sectretKey: "U0VDUkVUMTIzNAo="

    
    ```

- now if we try to `deploy the changes onto the cluster` by `applying the changes` as below 

    
    ```bash
        kubectl apply -f aws-credetials.yml
        # deploy the changes onto the cluster by applying the changes
        # here we wil get the error in this case having the base64codec Issue
        # the output will be as below
        secret/aws-credetials created

    ```

- if we are using the `stringData` field rather than `data` , this `field` will allow us to `put a non-base64 encoded string directly into the kubernetes secret`  , the `string data which is in non-base64 format for the value of the secret` will be `encoded for you` when the `secret been created or updated`

- now we can `remove/delete the secret` that we have created `earlier` as before

- we can do that by using the command as 

    ```bash
        kubectl delete -f aws-configure.yml\
        # deleting thr resource created by using the kubectl delete command in this case
        # the outpout will be as below 
        secret/aws-credetials deletd

    ```

- then we can replace the `data` field with `stringData` field as below 

- hence we can see the outcome as below for the same

    ```yaml
        aws-credetials.yml
        ==================
        apiVersion: v1 # here the secrets also belong to the core group hence mentioned as apiVersion: v1
        kind: Secret # here the type of object we will be using is of type Secret
        metadata: # defining the name of the kubernetes secrets and what namespace we want to put that in 
            name: aws-credetials # name of the kubernetes secrets
            namespace: default  # namespace where we wantr to put it in
        stringData: # inside the stringData block we can define the key and value pair here which will converted to dat field with base64 value at runtime
            accessKey: "1234567890" # defining the key and value with non-base64 format
            sectretKey: "SECRET1234" # defining the key and value with non-base64 format

    ```

- now if we try to `deploy the changes onto the cluster` by `applying the changes` as below 

    
    ```bash
        kubectl apply -f aws-credetials.yml
        # deploy the changes onto the cluster by applying the changes
        # here we wil get the error in this case having the base64codec Issue
        # the output will be as below
        secret/aws-credetials created

    ```