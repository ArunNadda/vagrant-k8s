#### what is deployment
- used to set desired state for RS. Deploymet Controller seeks to maintain the desired state by creating, deleting and replacing PODs with new config.

- Fields in Desired state:
  - Replicas
  - selector
  - template (pods)
  - 
  
- UseCases
  - to scale application up and down easiliy.
  - to perform rolling upgrades/updates.
  - rollback to previous version.


**Demo:**


```
cat 01_my-dep.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deploy
  template:
    metadata:
      labels:
        app: my-deploy
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
        ports:
        - containerPort: 80
```

- testing

```
kubectl create -f 01_my-dep.yml
kubectl get deploy
kubeclt get pods
kubectl delete pod my-deploy-559db4f57c-2d78m
kubectl get pods
```


#### scalling applications with Deployments
- what is scaling: increase or decrease resources 
- deployment scaling: horizontal scaling by increasing/decreasing number of pods
- how to scale a deployment: change nmber of replicas in deployment
  - edit deployment yaml, and use `kubectl apply -f <>`
  - use `kubectl edit` command.
  - use `kubectl scale` command. (`kubectl scale deployment.v1.apps/my-deploy --replicas=5`)


**Demo**

```
cat 02_my-dep.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: my-deploy
  template:
    metadata:
      labels:
        app: my-deploy
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
        ports:
        - containerPort: 80
```

```
kubectl get pods
kubeclt get deploy
kubectl apply -f 02_my-dep.yml
kubectl get pods
kubectl get deploy
kubectl edit deploy my-deploy
kubectl get deploy
kubectl scale deploy my-deploy --replicas=4
kubectl get deploy
kubectl scale deployment.v1.apps/my-deploy --replicas=3
kubectl get deploy
```

#### Deployment strategy
- recreate (non-default)
- Rolling Update (default, one pod at a time)






#### Rolling updates with Deployments
- What is Rolling update
  - changes to pods a controlled rate
  - 

- What are rollbacks:
  - can rollback to previous working state.

**Demo**

  - use `kubectl edit to change image in deployment template`
  - it will trigger rolling update.

```
kubectl edit deploy my-deploy
deployment.apps/my-deploy edited  
```

- kubectl rollout status deployment.v1.apps/my-deploy

```
$ kubectl rollout status deployment.v1.apps/my-deploy
deployment "my-deploy" successfully rolled out
vagrant@master-node:/vagrant/akn/05_deployments$ kubectl get deploy
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
my-deploy   3/3     3            3           20m
```



- kubectl set image deployment/my-deploy nginx=nginx:broken --record



```
kubectl edit deploy my-deploy
kubectl get deploy
kubectl rollout status deployment.v1.apps/my-deploy
kubectl get deploy
kubectl set image deployment/my-deploy nginx=nginx:broken --record
kubectl rollout status deployment.v1.apps/my-deploy
kubectl get deploy
kubectl get pods
```


```
kubectl rollout history  deploy my-deploy
deployment.apps/my-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl set image deployment/my-deploy nginx=nginx:broken --record=true

```


```
kubectl rollout history  deploy my-deploy
kubectl rollout undo deploy/my-deploy --to-revision=1
kubectl get po
```

####  Commands summary

```
kubectl create -f deploy.yml
kubectl get deploy
kubectl apply -f deploy1.yml
kubeclt set image deployment/my-deploy nginx=nginx:1.19.2
kubect rollout status deployment/my-deploy

kubectl rollout history  deploy my-deploy
kubectl rollout undo deploy/my-deploy --to-revision=1


kubectl get replicatsets
```
