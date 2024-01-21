# Note for Kops users

- If you're running Kops, here's a new feature which you need to be aware of...

- When you change a service to type LoadBalancer (all explained in the next video),  you'll see me find out the name of the load balancer by doing

    ```bash
        kubectl get svc
    
    ```
- On the video, the name of LoadBalancer appears immediately, although when I visit the address of the LoadBalancer, it is actually unavailable for several minutes

- A great new feature of kops, added after I recorded the video, is that "kubectl get svc" will continue to show "pending" for the LoadBalancer name until it becomes fully functional.

-  This avoids frustration when you're refreshing the page wondering if something is wrong.

- So: if you're on Kops, don't panic if you keep running "kubectl get svc" and you're getting "pending". It can take 5 or 10 minutes for the loadbalancer to work - just be patient.

- we can use the below command for this as 
    
    ```bash
        watch kubectl get svc

    ```
- EKS continues to work in exactly the same way portrayed on the video.