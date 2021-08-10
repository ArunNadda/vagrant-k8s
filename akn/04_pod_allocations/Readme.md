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

