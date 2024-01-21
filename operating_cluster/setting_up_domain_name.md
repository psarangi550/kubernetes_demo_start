# Setting Up a Domain Name in AWS for fleetman Application

- the `DNS` i.e generated due to the `Amazon Elastic LoadBalancer` , we don't want to use that `publish the side to the customer`

- if you have a `regular registered domain name` then  we can configure that `registered domain name` to be act as an `alias` for the `Elastic Load Balancer Domain Name`

- we can register the `domain name in AWS` using the `AWS Route53 Service` , which is a `DNS mgmt Service`

- we can `register a domain name` using the  `AWS Route53 Service` , by going to the `AWS Mgmt Console` &rarr; `AWS Route53 Service` &rarr; `register a domain name`

- we can see the `proicing details of the registering the domain name service`  with [AWS Route 53 price](https://d32ze2gidvkk54.cloudfront.net/Amazon_Route_53_Domain_Registration_Pricing_20140731.pdf)

- if we have the `already a registered domain name` then that will be displayed on the `Hosted Zone` section 

- if we want to create the `domain name alias for the ELB(Eloastic Load Balancer)` then `goto the Hosted Zone` &rarr; `select the domain nname with which you have registered` 

- the `click the Hosted zone registered domain name ` then `select the create record set` which will help in `create the subdomain for the domain` that `you have registered`

- the provide the `Alias target name` as `<subdomain>.<domain>` and select the type as `A-IPV4 Addresses`

- select the `Alaias` as `yes`, which will display all the `static DNS name available inside the Amazon ELB(Elastic Load Balancer)` &rarr; `select the particular DNS name which been created for the fleetman-webapp kubernetes service` &rarr; `create`

- once done then we can perform the `http://<subdomain>.<domain>` which will redirtect to the ` DNS name available inside the Amazon ELB(Elastic Load Balancer)` and show the `fleetman-webapplication`

- if we have the `registered domain name` which is not with the `AWS then we need to ` goto the `admin section of the registered domain name provider`, the process is very similar

- if we are `deleting` the `kubernetes stack` and then recreate the `kubernetes stack` then it will create the `new AWS LoadBalancer for the new kubernetes stack` , hence we need to update the `register domain name` to redirect to the `new ELB DNS address`

- but in a `production env` we will `not be deleting` the `AWS Elastic Load Balancer` , ot will there for the `life time of the Application`
