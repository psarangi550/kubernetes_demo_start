# testing ingress rules in EKS kubernetes cluster

- we have `deployed the nginx ingress controller` onto the `kubernetes EKS cluster` by applying changes to `deploy.yaml` which we got it from `https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/aws/deploy.yaml`

- now we will be `testing the ingress configuration` 

- currently we are `running on the host named as fleetman.com` for the `ingress-public.yml` file , we should have `register that name` in the `route53` in order to `redirect the request to the ingress-nginx service LoadBalancer`

- we currently `donn't` have the `domain name` registered as `fleetman.com` yet , hence we will be doing the same hack as `hacking the /etc/hosts` file in this case

- here we can see the `ingress-nginx-controller` on the `ingress-nginx` namespace then we will get the reference as 

    ```bash
        kubectl get svc -n ingress-nginx
        # fetching the service from the ingress-nginx namespace
        # here we will get the output as below
        NAME                                 TYPE           CLUSTER-IP     EXTERNAL-IP                                                                        PORT(S)                      AGE
        ingress-nginx-controller             LoadBalancer   10.100.59.11   a13e087200a8848d1b8af3ce81e5c789-cbcfcb643bcbe84d.elb.eu-central-1.amazonaws.com   80:31555/TCP,443:31590/TCP   31h
        ingress-nginx-controller-admission   ClusterIP      10.100.11.71   <none>                                                                             443/TCP                      31h

        # we can also perform the describe on the ingress-nginx service to get the details about the loadBalancer
        # we can do the describe as below 
        kubectl describe svc ingress-nginx-controller -n ingress-nginx
        # here the fetching the details about the ingress-nginx service in this case
        # the output will be as below 
        Name:                     ingress-nginx-controller
        Namespace:                ingress-nginx
        Labels:                   app.kubernetes.io/component=controller
                                app.kubernetes.io/instance=ingress-nginx
                                app.kubernetes.io/name=ingress-nginx
                                app.kubernetes.io/part-of=ingress-nginx
                                app.kubernetes.io/version=1.8.1
        Annotations:              service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
                                service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: true
                                service.beta.kubernetes.io/aws-load-balancer-type: nlb
        Selector:                 app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx
        Type:                     LoadBalancer
        IP Family Policy:         SingleStack
        IP Families:              IPv4
        IP:                       10.100.59.11
        IPs:                      10.100.59.11
        LoadBalancer Ingress:     a13e087200a8848d1b8af3ce81e5c789-cbcfcb643bcbe84d.elb.eu-central-1.amazonaws.com
        Port:                     http  80/TCP
        TargetPort:               http/TCP
        NodePort:                 http  31555/TCP
        Endpoints:                192.168.77.163:80
        Port:                     https  443/TCP
        TargetPort:               https/TCP
        NodePort:                 https  31590/TCP
        Endpoints:                192.168.77.163:443
        Session Affinity:         None
        External Traffic Policy:  Local
        HealthCheck NodePort:     31637
        Events:                   <none>



    ```

- here  we can get the `Static DNS Name of the Network LoadBalancer` but we want the `IP Address of that Network LoadBalancer`

- by design on the `AWS` will `not publish` the `IP Address of that Network LoadBalancer`

- we know that `IP Address` of the `LoadBalancer` can change over time

- 