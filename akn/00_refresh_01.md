
# Basic k8s commands
kubectl cluster-info
kubectl version
kubectl config view


kubectl get nodes
kubectl get nodes -o wide

# get component status
kubectl get componentstatus
kubectl get cs


# check NS - namespace
kubectl get ns
kubectl get namespaces
kubectl get pods --namespace test
kubectl create namespace test



# runs on default NS
kubectl get pods

# queries all NS
kubectl get pods -A
kubectl get pods --all-namespaces
kubectl get pods --all-namespaces -o wide


## cluster management
- draining a k8s node

kubectl drain <node name>
kubectl drain <node name> --ignore-daemonsets

kubectl uncordon <node name> # node ready to run pods




### kubectl command options

$ kubectl get <object type> <object name> -o <output> --sort-by <JSONPath> --selector <selector>
$ kubectl describe <object type> <object name> 
$ kubectl create -f <file>
$ kubectl apply -f <file>
$ kubectl delete <>
$ kubectl exec  <podName> -c <containerName> -- <command>

