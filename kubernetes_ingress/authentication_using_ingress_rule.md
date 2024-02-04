# Authentication using the Ingress rule in kubernetes

- in the `preveious session` we have `set the routing rules` to expose `multiple kubernetes services` in the `minikube kubernetes cluster` through `multiple host` using `domain and sub domain name`

- we can `provide various paths and patterns` using the `ingress-nginx controller` document as [Different Example of Ingress-Nginx](https://kubernetes.github.io/ingress-nginx/examples)

- we want the user to `access` the `fleetman web application frontend` , but we want to `protect the admin console for the queue` which is not bydefault available for the `users`

- `activeMQ` by default `provide security` , hence being secure `manage the ActiveMQ broker` , then it will `prompt for the username and password`

- ![alt text](image-7.png)

- but thats not good enough `but we want to make the homepage for the activeMQ` as well 

- here we will be doing a `basic Authentication` for the `admin console ingress controller`

- but we can also so as below with the `ingress controller`
    
    - `Client Certificate Authentication`
    
    - `External Basic Authenticatioon` 
    
    - `External OAUTH Authentication`


- all we need to do `kubernetes secrets` which will contain the `hashed version of the username and password` 

- once we setup the `kubernetes secret` we can reference that secret from the `ingress controller`