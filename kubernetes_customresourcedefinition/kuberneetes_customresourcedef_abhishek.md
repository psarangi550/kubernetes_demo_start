# Custom resources in kubernetes 

- here we will learn about the `custom resource definition` and `custom resources` in `kubernetes`

- what is a `custom resource definition` or `CRD` ?

- what is a `custom resource` ?

- what is a `custom controller` ?

- **Overview of the `Custom Resouce Definition` and `Custom Resource` and `Custom Controler`**

- in a `kubernetes cluster` we have `different type of resource which comes out of the box` such as `Deployment, POD and Services` which been handled by the `kube controller`

- the `resouces which comes out of the box` are called as the `native resource in kubernetes`

- but `apart from the native kubernetes resources` , `kubernetes` says if we want to `extend the API of kubernetes` by `introducing new custom resource`

- **why we want to use the `custom resources` in kubernetes ?** 

- if we feel that `existing native resource` will not `support the functionality that we want to provide` then we can use the `custom resources`

- for example
  
  - if we `feel` that `kubernetes does not provide advance level security capability` such as `kubeHunter` and `kyverno` or `kube-bench` for the `kubernetes security prospect(CKS)` then we need to create the `custom  new resource for the same`
  
  - if we want to introduce `ArgoCD` or `flux` or `spinaker` for the `gitOps(CNCF)` approach  then we need to create the `custom new resource for the same`
  
- here there are `2 Actor` which will perform this action 
  
  - `DevOps engg` :- which will `manage and deploy` the `custom resource definition` and `custom controller`
  
  - `User` :- which will use the `custom resource` thats ben defined in the `custom resource definition`
  
- when we want to `create a new resource` by `extending the kubernetes API` then we can go for the `custom resource definition` or `CRD`


- **Deep Dive into Custom Resource and Custom Resource Definition and Custom Controller**

- 