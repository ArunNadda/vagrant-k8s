
# k8s control-plane

### kube-api-server
### etcd
### kube-scheduler
### kube-controller-manager
### cloud-controller-manager

# k8s nodes

### kubelet
### container runtime (docker)
### kube-proxy

# setup cluster
### kubeadm


# Basic k8s commands

```
kubectl cluster-info
kubectl version
kubectl config view


kubectl get nodes
kubectl get nodes -o wide
```
# get component status

```
kubectl get componentstatus
kubectl get cs
```

# check NS - namespace

```
kubectl get ns
kubectl get namespaces
kubectl get pods --namespace test
kubectl create namespace test
```


# runs on default NS

```
kubectl get pods
```

# queries all NS

```
kubectl get pods -A
kubectl get pods --all-namespaces
kubectl get pods --all-namespaces -o wide
```





## cluster management

- draining a k8s node

```
kubectl drain <node name>
kubectl drain <node name> --ignore-daemonsets

kubectl uncordon <node name> # node ready to run pods
```

# Demo:

- create a pod in default ns

```
cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
   name: nginxpod
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  restartPolicy: OnFailure

```

```
kubectl apply -f pod.yaml
kubectl get pods
```

- create a deployment 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```
kubectl get pods -o wide
```

#### drain the node

```
kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP               NODE            NOMINATED NODE   READINESS GATES
nginx-deployment-65587f96cc-68mqs   1/1     Running   0          23s     192.168.158.2    worker-node02   <none>           <none>
nginx-deployment-65587f96cc-7gjmb   1/1     Running   0          23s     192.168.87.194   worker-node01   <none>           <none>
nginxpod                            1/1     Running   0          5m14s   192.168.158.1    worker-node02   <none>           <none>

kubectl drain worker-node02

$ kubectl drain worker-node02
node/worker-node02 cordoned
error: unable to drain node "worker-node02", aborting command...

There are pending nodes to be drained:
 worker-node02
cannot delete Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet (use --force to override): default/nginxpod
cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-system/calico-node-mc2s6, kube-system/kube-proxy-dfhtk
```


```
 kubectl drain worker-node02 --force --ignore-daemonsets
node/worker-node02 already cordoned
WARNING: deleting Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet: default/nginxpod; ignoring DaemonSet-managed Pods: kube-system/calico-node-mc2s6, kube-system/kube-proxy-dfhtk
evicting pod default/nginxpod
evicting pod default/nginx-deployment-65587f96cc-68mqs
pod/nginxpod evicted
pod/nginx-deployment-65587f96cc-68mqs evicted
node/worker-node02 evicted
```

```
$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP               NODE            NOMINATED NODE   READINESS GATES
nginx-deployment-65587f96cc-7gjmb   1/1     Running   0          9m3s   192.168.87.194   worker-node01   <none>           <none>
nginx-deployment-65587f96cc-8whw7   1/1     Running   0          30s    192.168.87.195   worker-node01   <none>           <none>
```

```
$ kubectl get nodes
NAME            STATUS                     ROLES                  AGE   VERSION
master-node     Ready                      control-plane,master   12h   v1.21.3
worker-node01   Ready                      worker                 37m   v1.21.3
worker-node02   Ready,SchedulingDisabled   worker                 35m   v1.21.3
```

```
kubectl uncordon worker-node02

kubectl get nodes
NAME            STATUS   ROLES                  AGE   VERSION
master-node     Ready    control-plane,master   12h   v1.21.3
worker-node01   Ready    worker                 37m   v1.21.3
worker-node02   Ready    worker                 35m   v1.21.3
```



# upgrade k8s using kubeadm

### controlplane node
- upgrade kubeadm on cp node `sudo apt-get update && sudo apt-get install -y --allow-change-held-packages kubeadm=1.21.1-00`
- drain cp node
- plan the upgrade `kubeadm upgrade plan <version>`
- apply the upgrade `kubeadm upgrade apply <version>`
- upgrade kubelet and kubectl on cp node `sudo apt-get install -y --allow-change-held-packages kubelet=1.21.1-00 kubectl=1.21.1.00`
- uncordon cp node

### worker node
- drain node
- upgrade kubeadm
- upgrade kubelet config `kubeadm upgrade node`
- upgrade kubelet and kubectl
- uncordon node


# backup and restore etcd
- `etcdctl`
- `etcdctl snapshot save <>`
- `etcdctl restore snapshot <>`
