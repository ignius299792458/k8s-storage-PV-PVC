# Overall view PV->PVC->Pod->Node/Worker->KIND Docker -> exact volume allocated place process

Example, illustrating
  -  Create PV (say: local-pv) and apply it (persistence-volume.yml config)
  -  Check if pv/local-pv is at `Available` state of status or not
  -  Claiming applied PV (local-pv) as local-pvc (persistent-volume-claim.yml config)
  -  Check if pvc/local-pvc is at `Bound` state of status or not
  -  Now, `claim the bounded pvc` to deployment.yml pods and apply it.
  -  check the status of pods, if `pending`, smth went wrong like in this no namespace maintainance in all resource pv, pvc and deployments.
  -  make sure the pod at `running` indicate pv and pvc are successfully joined with deployment's replicas
  -  then check pod under which node/worker, again check worker/node which container of KIND, 
      inside that container check assigned location in local-pv i.e `/mnt/data` would be created and persisting the data.

Hands-on

```
➜ vim persistence-volume.yml 

➜ kubectl apply -f persistence-volume.yml 
persistentvolume/local-pv created

➜ ls /mnt/data
ls: /mnt/data: No such file or directory

➜ kubectl get pv 
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    VOLUMEATTRIBUTESCLASS   REASON   AGE
local-pv   512Mi      RWO            Retain           Available           local-storage   <unset>                          36s

➜ vim persistent-volume-claim.yml

➜ kubectl apply -f persistent-volume-claim.yml 
persistentvolumeclaim/local-pvc created

➜ kubectl get pv     
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS    VOLUMEATTRIBUTESCLASS   REASON   AGE
local-pv   512Mi      RWO            Retain           Bound    default/local-pvc   local-storage   <unset>                          5m59s

➜ kubectl get pvc
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    VOLUMEATTRIBUTESCLASS   AGE
local-pvc   Bound    local-pv   512Mi      RWO            local-storage   <unset>                 44s

➜ ls /mnt/data
ls: /mnt/data: No such file or directory

➜ kubectl get pods -n nginx
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-f58b5c699-54ztl   1/1     Running   0          3d23h
nginx-deployment-f58b5c699-xmpnm   1/1     Running   0          3d23h

➜ vim deployment.yml 

➜ kubectl delete -f deployment.yml 
deployment.apps "nginx-deployment" deleted

➜ kubectl get pods -n nginx
No resources found in nginx namespace.

➜ vim deployment.yml

➜ kubectl apply -f deployment.yml 
deployment.apps/nginx-deployment created

➜ kubectl get pods -n nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77fb8d6bfc-2bw2g   0/1     Pending   0          14s
nginx-deployment-77fb8d6bfc-tjksv   0/1     Pending   0          14s

➜ kubectl get pods -n nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77fb8d6bfc-2bw2g   0/1     Pending   0          16s
nginx-deployment-77fb8d6bfc-tjksv   0/1     Pending   0          16s

➜ kubectl get pods -n nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77fb8d6bfc-2bw2g   0/1     Pending   0          18s
nginx-deployment-77fb8d6bfc-tjksv   0/1     Pending   0          18s

➜ kubectl describe pod/nginx-deployment-77fb8d6bfc-2bw2g -n nginx
Name:             nginx-deployment-77fb8d6bfc-2bw2g
Namespace:        nginx
Priority:         0
Service Account:  default
Node:             <none>
Labels:           app=nginx
                  pod-template-hash=77fb8d6bfc
Annotations:      <none>
Status:           Pending
IP:               
IPs:              <none>
Controlled By:    ReplicaSet/nginx-deployment-77fb8d6bfc
Containers:
  nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2jpbb (ro)
      /var/www/html from deployment-my-volume (rw)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  deployment-my-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  local-pvc
    ReadOnly:   false
  kube-api-access-2jpbb:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  73s   default-scheduler  0/4 nodes are available: persistentvolumeclaim "local-pvc" not found. preemption: 0/4 nodes are available: 4 Preemption is not helpful for scheduling.

➜ echo "local-pv, local-pvc aren't is same name space so, pods remain pending"
local-pv, local-pvc aren't is same name space so, pods remain pending

➜ vim persistence-volume.yml 

➜ vim persistent-volume-claim.yml 

➜ echo "re-apply all again within nginx namespace"
re-apply all again within nginx namespace

➜ kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS    VOLUMEATTRIBUTESCLASS   REASON   AGE
local-pv   512Mi      RWO            Retain           Bound    default/local-pvc   local-storage   <unset>                          41m

➜ kubectl delete -f persistence-volume.yml 
persistentvolume "local-pv" deleted
^C%                                                                              
➜ kubectl delete pv/local-pv
persistentvolume "local-pv" deleted
^C%                                                                              
➜ kubectl delete pvc/local-pvc
persistentvolumeclaim "local-pvc" deleted

➜ kubectl delete pv/local-pv 
Error from server (NotFound): persistentvolumes "local-pv" not found

➜ kubectl delete pv/local-pv
Error from server (NotFound): persistentvolumes "local-pv" not found

➜ kubectl apply -f persistence-volume.yml 
persistentvolume/local-pv created

➜ kubectl get pv -n nginx
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    VOLUMEATTRIBUTESCLASS   REASON   AGE
local-pv   512Mi      RWO            Retain           Available           local-storage   <unset>                          13s

➜ kubectl apply -f persistent-volume-claim.yml 
persistentvolumeclaim/local-pvc created

➜ kubectl get pvc -n nginx
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    VOLUMEATTRIBUTESCLASS   AGE
local-pvc   Bound    local-pv   512Mi      RWO            local-storage   <unset>                 17s

➜ kubectl get pods -n nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77fb8d6bfc-2bw2g   1/1     Running   0          8m21s
nginx-deployment-77fb8d6bfc-tjksv   1/1     Running   0          8m21s

➜ echo "Now, the delpoyment pods are running and bounded with PV through PVC"
Now, the delpoyment pods are running and bounded with PV through PVC

➜ kubectl describe pod/nginx-deployment-77fb8d6bfc-tjksv 
Error from server (NotFound): pods "nginx-deployment-77fb8d6bfc-tjksv" not found

➜ kubectl describe pod/nginx-deployment-77fb8d6bfc-tjksv -n nginx
Name:             nginx-deployment-77fb8d6bfc-tjksv
Namespace:        nginx
Priority:         0
Service Account:  default
Node:             ignius-k8s-cluster-worker2/172.18.0.3
Start Time:       Sun, 20 Apr 2025 00:28:44 +0545
Labels:           app=nginx
                  pod-template-hash=77fb8d6bfc
Annotations:      <none>
Status:           Running
IP:               10.244.2.68
IPs:
  IP:           10.244.2.68
Controlled By:  ReplicaSet/nginx-deployment-77fb8d6bfc
Containers:
  nginx:
    Container ID:   containerd://61e82e8a5c4c75a8558877d9b515d56737fcfa7d743a3d8e6a51c3a8e3c71a20
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:5ed8fcc66f4ed123c1b2560ed708dc148755b6e4cbd8b943fab094f2c6bfa91e
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 20 Apr 2025 00:28:47 +0545
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kctmh (ro)
      /var/www/html from deployment-my-volume (rw)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  deployment-my-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  local-pvc
    ReadOnly:   false
  kube-api-access-kctmh:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                   From               Message
  ----     ------            ----                  ----               -------
  Warning  FailedScheduling  4m7s (x2 over 9m31s)  default-scheduler  0/4 nodes are available: persistentvolumeclaim "local-pvc" not found. preemption: 0/4 nodes are available: 4 Preemption is not helpful for scheduling.
  Warning  FailedScheduling  103s                  default-scheduler  0/4 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/4 nodes are available: 4 Preemption is not helpful for scheduling.
  Normal   Scheduled         98s                   default-scheduler  Successfully assigned nginx/nginx-deployment-77fb8d6bfc-tjksv to ignius-k8s-cluster-worker2
  Normal   Pulling           98s                   kubelet            Pulling image "nginx:latest"
  Normal   Pulled            96s                   kubelet            Successfully pulled image "nginx:latest" in 2.202s (2.202s including waiting). Image size: 68844367 bytes.
  Normal   Created           96s                   kubelet            Created container: nginx
  Normal   Started           95s                   kubelet            Started container nginx

➜ kubectl get pods -n nginx -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE                         NOMINATED NODE   READINESS GATES
nginx-deployment-77fb8d6bfc-2bw2g   1/1     Running   0          11m   10.244.3.78   ignius-k8s-cluster-worker3   <none>           <none>
nginx-deployment-77fb8d6bfc-tjksv   1/1     Running   0          11m   10.244.2.68   ignius-k8s-cluster-worker2   <none>           <none>

➜ kubectl get nodes
NAME                               STATUS   ROLES           AGE    VERSION
ignius-k8s-cluster-control-plane   Ready    control-plane   4d4h   v1.32.2
ignius-k8s-cluster-worker          Ready    <none>          4d4h   v1.32.2
ignius-k8s-cluster-worker2         Ready    <none>          4d4h   v1.32.2
ignius-k8s-cluster-worker3         Ready    <none>          4d4h   v1.32.2

➜ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED      STATUS      PORTS                                      NAMES
286215680abd   kindest/node:v1.32.2   "/usr/local/bin/entr…"   4 days ago   Up 4 days                                              ignius-k8s-cluster-worker
5f791c93a241   kindest/node:v1.32.2   "/usr/local/bin/entr…"   4 days ago   Up 4 days   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   ignius-k8s-cluster-worker3
ac59468247c9   kindest/node:v1.32.2   "/usr/local/bin/entr…"   4 days ago   Up 4 days   127.0.0.1:54493->6443/tcp                  ignius-k8s-cluster-control-plane
e9a9b8407b12   kindest/node:v1.32.2   "/usr/local/bin/entr…"   4 days ago   Up 4 days                                              ignius-k8s-cluster-worker2
➜  docker exec -it 5f791c93a241 bash
root@ignius-k8s-cluster-worker3:/# ls
LICENSES  bin	demo-data  etc	 kind  media  opt   root  sbin	sys  usr
backups   boot	dev	   home  lib   mnt    proc  run   srv	tmp  var
root@ignius-k8s-cluster-worker3:/# ls -l /mnt/data 
total 0
root@ignius-k8s-cluster-worker3:/# cd /mnt/data/
root@ignius-k8s-cluster-worker3:/mnt/data# ls
root@ignius-k8s-cluster-worker3:/mnt/data# echo "This is where local-pv, data is kept as local-storage -> pv and pvc -> pod, pod under which node, node under which KIND worker, worker is docker container due to KIND setup of K8s, this is exactly where we allocate PV and PVC"
This is where local-pv, data is kept as local-storage -> pv and pvc -> pod, pod under which node, node under which KIND worker, worker is docker container due to KIND setup of K8s, this is exactly where we allocate PV and PVC
root@ignius-k8s-cluster-worker3:/mnt/data# 
```
