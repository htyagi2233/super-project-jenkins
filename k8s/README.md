# Testing 
## 1. Verify on k8s Server
	
```
kubectl get all
NAME                                          READY   STATUS    RESTARTS   AGE
pod/project-k8s-deployment-547bd7fd8f-4mb7t   1/1     Running   0          5m31s
pod/project-k8s-deployment-547bd7fd8f-59qg8   1/1     Running   0          5m31s
pod/project-k8s-deployment-547bd7fd8f-jlnfv   1/1     Running   0          5m31s

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP        3d3h
service/project-k8s   NodePort    10.105.204.228   <none>        80:30080/TCP   5m31s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/project-k8s-deployment   3/3     3            3           5m31s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/project-k8s-deployment-547bd7fd8f   3         3         3       5m31s



root@k8s:~# kubectl get nodes -o wide
NAME         STATUS   ROLES           AGE    VERSION   INTERNAL-IP       EXTERNAL-IP         
k8s          Ready    control-plane   3d3h   v1.33.4   192.168.192.133   <none>     
k8s-worker   Ready    <none>          3d2h   v1.33.4   192.168.192.134   <none>       
```


## 2. Access the Application
- Use this pattern:
- http://<Node-IP>:<NodePort>
- http://<k8s-server-IP>:30080/addressbook-2.0/
- <http://192.168.192.134:30080/addressbook-2.0/><img width="514" height="20" alt="image" src="https://github.com/user-attachments/assets/9746ef42-14bd-46bb-bdca-915da39fe784" />



