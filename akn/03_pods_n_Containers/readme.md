#### managing apps config and managing container resources
- what is application configuration, features in k8s to perform this.
- configMaps and Secrets
- env-variables and Config Volumes




#### Application management and configuration
- pass dynamic vaules to containers/apps at runtime., 2 ways to do it.
- create configmap and inject to pod (same with secrets)

- Configmaps (has data instead of spec)

```
cat 01_configmap.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  key1: value1
  key2: value2
  key4: |
    multiline data
    can also be
    added.
```

```
kubectl create -f 01_configmap.yml
kubectl get configmaps
kubectl describe configmap my-configmap
```


- Secrets

```
cat 02_secrets.yml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque #-- default
data:
  username: dGVzdAo=
  password: cGFzc3Rlc3QK
```

```
**echo passtest | base64**
**echo test | base64**

kubectl create -f 02_secrets.yml
kubectl describe secret my-secret
kubectl get secrets my-secret -o yaml
```
**how to pass it to containers**

- Environment Variable: (read configmap or secret as ENV)
- can be set directly or using configMap and secrets

```
env:
- name: myenv
  value: myvalue
```


```
spec:
  containers:
  - ...
  env:
  - name: ENVVAR
    valueFrom:
      configMapKeyRef:
      name: my-configmap
      key: mykey
```


```
# envs (can use all envs from configmap)
envFrom:
- configMapRef:
    name: 
```

- Configuration Volumes (supply data form configmap/serets as mounted volumes) and the mount these volumes in pods

```
...
volumes:
- name: secret-vol
  secret:
    secretName: my-secret
```


- commands:




```
# test env variable 

kubectl create -f 03_pod_env.yml
kubectl describe pod env-prod
kubectl logs env-prod
kubectl get pods
kubectl logs env-prod

## create volumes in yaml and the use these to `volumeMounts` in container.
kubectl create -f 04_pod_vol.yml
kubectl get pods
kubectl get vol-pod -- ls /etc/config/cm
kubectl exec vol-pod -- ls /etc/config/cm
kubectl exec vol-pod -- cat /etc/config/cm/key4
kubectl exec vol-pod -- cat /etc/config/secret/username
kubectl exec vol-pod -- ls /etc/config/sec
kubectl exec vol-pod -- cat /etc/config/sec/username
kubectl exec vol-pod -- cat /etc/config/sec/password
```




## managing container Resources
- ResourceRequests (affects scheduling)

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: busybox
    image: busybox
    resource:
      requests:
        cpu: "250m"
        memory: "128Mi"
```

```
kubectl create -f 05_big_pod.yml

kubectl get pods my-big-pod (##### this pod will be in pending...)
NAME         READY   STATUS    RESTARTS   AGE
my-big-pod   0/1     Pending   0          8s
```

```
cat 05_big_pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: my-big-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    resources:
      requests:
        cpu: "10000m"
        memory: "128Mi"
```



- Resource Limits (conteiner runtime is responsible to enforce these)

```
$ cat 06_nice_pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: my-nice-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    resources:
      requests:
        cpu: "250m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "256Mi"
```

## Monitoring container health with probes
- Container Health: check container (apps) health

- Liveness Probes: determine whether or not a container application is in a healthy state.

- Startup Probes: similar to Liveness Probes, runs only at startup.

- Readiness Probes: to determine when a container is ready to accept requests.


```
kubectl create -f 07_liveness_probe.yml
kubectl get pods

kubectl create -f 08_nginx_liveness.yml
kubectl get po

kubectl create -f 09_startup_nginx.yml

kubectl create -f 10_readiness_nginx.yml
## keep on checking pod status
```


#### Self-healing pods with Restart policies

- Restart policies
  - Always: default, containers will be restarted if they stop or unhealthy (or succeed and finishes). Should use for apps which should always be running
  - OnFailure: Restart container if it becomes unhealthy or process exit with an error code.
  - Never: pod will never be restarted (opposite of Always)

```
kubectl create -f 11_always_restart.yml
# it will restart after 10sec (even after successful complition)

kubectl create -f 12_onfailure.yml        #will not restart
kubectl create -f 13_onfailure_01.yml        #will restart



kubectl create -f 14_neverfail.yml
kubectl create -f 14_neverfail.yml
# none of pods will restart.

```


#### Creating Multi-Container pods
- What is multi-container pod: pod with more than one container. These containers share resources (network, storage)
- Cross-Container interaction: using shared resources, network (same network namespace), storage ( can use shared volumes to shared data in Pod)


- Example usecase (vault agent): 
  - sidecar 
  - read logs from an app where it send logs to static location


```
kubectl create -f 16_multicontainer.yml

kubectl create -f 17_sidecar.yml

```



#### Init Containers
- What is initContainer: run once to completions during the startof the pod, can be any number of init-containers.
- UseCase: (vault token retrieval)
  - initial startup tasks
  - fetch secrets before app start.
  - populate data to sahred volume at startup.
  - register pod to external services etc.

```
kubectl create -f 18_init.yml


kubectl logs po init-pod -c delay
```


