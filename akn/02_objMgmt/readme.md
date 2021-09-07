### refresh 
#### Basic k8s commands

```
kubectl cluster-info
kubectl version
kubectl config view


kubectl get nodes
kubectl get nodes -o wide
```
#### get component status

```
kubectl get componentstatus
kubectl get cs
```

#### check NS - namespace

```
kubectl get ns
kubectl get namespaces
kubectl get pods --namespace test
kubectl create namespace test
```


#### runs on default NS

```
kubectl get pods
```

#### queries all NS

```
kubectl get pods -A
kubectl get pods --all-namespaces
kubectl get pods --all-namespaces -o wide
```

#### set namespace context

kubectl config set-context --current --namespace=test



####  cluster management
- draining a k8s node

```
kubectl drain <node name>
kubectl drain <node name> --ignore-daemonsets

kubectl uncordon <node name> # node ready to run pods
```



#### kubectl command options

```
$ kubectl get <object type> <object name> -o <output> --sort-by <JSONPath> --selector <selector>
$ kubectl describe <object type> <object name> 
$ kubectl create -f <file>
$ kubectl apply -f <file>
$ kubectl delete <>
$ kubectl exec  <podName> -c <containerName> -- <command>
```

#### about kubectl

```
$ kubectl get <object type> <object name> -o <output> --sort-by <JSONPath> --selector <selector>
$ kubectl describe <object type> <object name> 
$ kubectl create -f <file>
$ kubectl apply -f <file>
$ kubectl delete <>
$ kubectl exec  <podName> -c <containerName> -- <command>
```

#### demo

```
cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
   name: busybox
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 3600; done']
```

```
$ kubectl apply -f pod.yaml

$ kubectl api-resources

$ kubectl get po
$ kubectl get pods
$ kubectl get pods -o wide
$ kubectl get pods -o json
$ kubectl get pods -o yaml
$ kubectl get pods --sort-by .spec.nodeName
$ kubectl get pods -n kube-system
$ kubectl get pods -n kube-system --selector k8s-app=calico-node
$ kubectl describe po busybox
$ kubectl create -f pod.yaml
$ kubectl exec busybox -c busybox -- echo "Hello, world!"
$ kubectl delete pod busybox
```

#### kubectl tips (imperative command)

```
kubectl create deployment my-depl --image=nginx
kubectl create deployment my-depl --image=nginx --dry-run -o yaml >my-depl.yaml
cat my-depl.yaml
kubectl scale deployment my-depl --replicas=3 --record
```


#### roles and role binding RBAC (k8s objects)

- role
- clusterRole
- roleBinding
- clusterRoleBinding

```
$ cat role.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "watch", "list"]


cat rolebinding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader
  namespace: default
subjects:
- kind: User
  name: dev
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

```

```
kubectl apply -f role.yml
kubectl apply -f rolebinding.yml
```

#### service accounts and rbac (used in vault integration)
- what is SA (account used by container processes within pod to access k8s API)
- can use RBAC to control access to SAs. (bind to Role or clusterRole)

```
kubectl create -f my-sa.yml
kubectl create sa my-sa2 -n default

kubectl get sa
kubectl create -f sa-pod-reader.yml
kubectl describe sa my-sa
```

```
$ cat my-sa.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-sa
  namespace: default ### not necesary as default is assumed.


$ cat sa-pod-reader.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-pod-reader
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```



## check sa details in pod

```
cd /var/run/secrets/kubernetes.io/
$ ls
serviceaccount
$
$ ls -l
total 0
lrwxrwxrwx 1 root root 13 Jul 27 22:54 ca.crt -> ..data/ca.crt
lrwxrwxrwx 1 root root 16 Jul 27 22:54 namespace -> ..data/namespace
lrwxrwxrwx 1 root root 12 Jul 27 22:54 token -> ..data/token
$ pwd
/var/run/secrets/kubernetes.io/serviceaccount
$
```

