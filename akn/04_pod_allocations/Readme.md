
#### scheduling

- what is scheduling: is a process of assigning pods to nodes.
- scheduler: a CP component handles and run scheduling
- Scheduling process: k8s scheduler selects node for each pod, by checking 
  - Resource requests vs available resources on node
  - various configurations that affect scheduling using node labels.

**nodeSeletor**:
- configure nodeSelector for pods to limit on which nodes it can run.
- Node Selectors using node labels to filter nodes

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    mylabel: myvalue
```


**nodeName**:
- allows  to bypass scheduling and assing nodes directly.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: worker1
```


#### Demo:

```
kubectl get nodes
kubectl label nodes worker-node01 dev=true
```

#### DaemonSets:
- What is DaemonSet: runs copy of pod on each node and any new node added in future.
- DaemonSet and Scheduling: DaemonSets respect scheduling rules around node labels, taints & tolerations.


#### Static Pod
- pod created directly by kubelet on the node
- created from manifest files. (/etc/kubernetes/manifest)

**mirrorPods**
- mirror pod is create in k8s API, to see status of static POD. but cannot be managed by API. (kind of ghost of staticpod)

