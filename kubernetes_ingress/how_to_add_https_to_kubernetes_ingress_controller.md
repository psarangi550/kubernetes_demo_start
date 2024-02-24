# How to Add Https TLS and SSL cert for the kubernetes ingress in EKS kubernetes cluster

- there are `lot of different startergy` to put together for `setting` the `https` on the `kubernetes cluster`

- we `fiddle things` inside the `/etc/hosts` thats why we `don't have` to `registrer an domain name` 

- he hacked the `/etc/hosts` file to `fleetman.com` and `queue.fleetman.com` to the `/etc/hosts` file , where we `redirected` the `domain name` to the `IP address of the Network LoadBalancer`

- here we do need a `DNS Name` and it need to be `registered` with `amazon route53` which is the `amazon DNS Service`

- we need to `pay` for the `Domain Name` that we are `trying to register` [Amazon Route53 DNS](https://d32ze2gidvkk54.cloudfront.net/Amazon_Route_53_Domain_Registration_Pricing_20140731.pdf) per year

- here we will be performing `certain sterps` for the `HTTPS request`

  - there are actually `2 ways`we can setup the `https` connection to the `EKS cluster`
    
    -  **1st Approach**:-
      
      - we can `configure` the `Ingress Controller` to `terminate the SSL connection`
      
      - we can get an `certificate` from the `certificate Authority` and we need to `pay for that`
    
      - we will be `install` the `certificate provided by the certificate authority` onto the `ingress controller` running on `ingress-nginx` namespace
      
      - hence all the `traffic` coming from the `browser` to the `LoadBalancer` on port `443 rather than on port 80` would be `encrypted`
      
      - this `traffic` will be `encrypted` as far as the `ingress controller` which then `validate the certificate` and `domain name and url matched to the certificate installed` on the `ingress controller` , then the `traffic will be decrypted`
      
      - if we are doing the `SSL Termination` inside the `ingress controller` then the `traffic` from the `ingress controller` to the `corresponding kubernetes Service` will be `insecure` will be done by `http` only
      
      - how to `perform` that `ingress controller` SSL `termination then how to do that` 
      
      - but if we `install` the `tls certificate issued by the certificate authority` on the `ingress controller` then we need to manage the `tls certificate`
      
      - when the `certificate` `expired` then we need to `get the new certificate` and `plug that` into the `ingress controller` , which is a `management work` which `most often you don't want to do`
      
    
    - **2nd Approach** :-
      
      - we can `shift` the thing up a label  install the `certificate` on the `Load Balancer` , we can shift the `SSL Termination` on the `Load Balancer`
      
      - the `traffic` going from the `Load Balancer` to the `Ingress Controller` and rest of the Service will be done by using the `regular http`
      
      - the `other benefit` of that `amazon LoadBalancer` that `all that certification management` can be handle by the `amazon` automatically
      
      - here `amazon` will handle `rotating` the `certification management` on close to the `expiry`
      
      - if we have `already registered` for the `domain name` then we don't have to `pay for the certificate issued the certificate auithority`
      
      - we need to pay for the `domain name registered with Amazon Route 53` but we don't have to pay for the `certificate` which comes as extra , which makes the `process` smoother
      
      - 