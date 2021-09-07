### K8s Storage
- container FS
  - ephermal, 
  - data storage in CFS is lost as soon as docker is down/restarted.
  
- Volumes
  - provide persistent storage, external storage 
  - allow to store data outside the CFS, still allowing access to data at runtime.
  - simple way to provide storage.

- Persistent Volumes
  - advanced form of Volumes
  - allow to treat storage as an abstract resource and consume it using pod.
  - Persistent Volume Claim used by containers to claim PVs.

- Volume Types
  - determines how the storage is actually handled.
  - NFS
  - Cloud Storage Mechanisms
  - ConfigMaps and Secrets
  - A Simple Directory on K8s node.


### Using K8s Volumes

- volumes and volumesMounts: can setup easily within pod/container spec
  - Volumes: 
    - specified in Pod Spec (these are pod resources not Container Resources)
    - specify storage volumes available to the Pod
    - specify volume Type and other data which determines where and how the data is actually stored.
      
  - VolumeMounts:
    - specifed in Container Spec.
    - reference (name of) the `volumes` in Pod Spec and provide `mountPath` for container.


- Sharing Volumes between Containers.
  - Can use `volumeMounts` to mount same volume to multiple containers in same pod.
  - used to interact containers to each other.
  - used in sidecar/init containers.

- Common Volume Types
  - `hostPath`: stores data in a specified directory on the K8s node.
  - `emptyDir`: stores data in a dynamically created location on the node. it is temporary volume (not persistent).


### Demo

```
$ cat 01_pod-vol.yaml
apiVersion: v1
kind: Pod
metadata:
  name: vol-pod
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'echo Success! > /output/success.txt']
    volumeMounts:
    - name: my-vol
      mountPath: /output
  volumes:
  - name: my-vol
    hostPath:
      path: /var/voldata

kubectl create -f 01_pod-vol.yaml
```

- check which node pod was running

```
kubectl get pod -o wide
```


- on the worker node

```
cat /var/voldata/success.txt
```




###  emptydir volume

```
kubectl create -f 02_empty_dir_pod.yaml
kubectl logs -f empty-dir-pod -c busybox1
```



### persistent volumes:

- are k8s objects that allow to treat storage as an abstract resource to be consumed by Pods.
- uses a set of attributes to describe the underlying storage resrouce, which will be used to store data
- everything in spec are attributes for pv

**Storage Classes**
- allows to specify the types of storage services they offer on their platform.
- 

**allowVolumeExpansion**
- determinds wheather or not storageclass support resize after its created.
- if not ste to true, attempting to resize vol will result in an error

**reclaimPolicies**
- PVs persistentVolumeReclaimPOlicy determines how storage resources can be reused, when pv associated with PVC is deleted.
- `Recycle`- automatically deletes all data in the underlying storage resource., `Retain` - keeps all data, requires admins to manually cleanup data. , `Delete`- only works for cloud storage resources only.


### PersistentVolumeClaims
- A PVC represents request for storage resources.
- attributes similar to PVs
- a PVC will look at a PV which is able to meet requested criteria. if found, it will bound to that PV. (equal to or >than the storage requested by PVC)

- PVC can be mounted on Pods container like any other vol.
- pod will use PV bound to PVC.

**resizing**
- can expand PVC without interrupting apps using it. Edit `spec.resources.requests.storage` of existing PVC to higher value (StorageClass must support resizing volumes i.e. must have `allowVolumeExpantion = true`)

### Demo

- create Storage Class (so we can set expantion to true):

```
$ cat 03_storage_class_Local_Disk.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: localdisk
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
```

```
kubectl create -f 03_storage_class_LocalDisk.yaml
```

- create pv

```
$ cat 04_my_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mv-pv
spec:
  storageClassName: localdisk
  persistentVolumeReclaimPolicy: Recycle
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /var/output

$ kubectl create -f 04_my_pv.yaml
$ kubectl get pv
```


- create pvc


```
$ kubectl create -f 05_my_pvc.yaml
$ kubectl get pv
$ kubectl get pvc
```

- create pod

```
$ kubectl create -f 06-mypv-pod.yaml
$ kubectl get pod -o wide
```

- expand pvc

```
$ kubectl edit pvc my-pvc --record # change to 200Mi
```

- increase to 500 using `kubectl apply -f 07_my_pvc.yaml`
- check `kubectl describe pvc`
- increase to 1500 using `kubectl apply -f 07_my_pvc.yaml`
- check `kubectl describe pvc` 

**pvs are cluster resources, but pvcs are namespace bound**


**recycle**

```
$ kubectl delete pod mypv-pod

$ kubectl delete pvc my-pvc
persistentvolumeclaim "my-pvc" deleted


$ kubectl get pv
```



### StatefulSets
- stateless apps
- statefull apps

- deployment vs statefulsets
 - containers
 - volumes
 - pvs/pvcs

- why: 
  - coz service, pod share same storage
  - in case of apps like vault, need pods with same name/storage
  - same network identity (no persisted identity)
  - if pods are created/deleted with different names, no DNS names.
  - deployment create random nodes names and terminates these randomly
  - scaleup and down needed to be consistent.


**statdulsets**
- creates pods 1 by 1 in very predictable fashion (and is ready, before it created next pod)
- pods have their dns names.
- if pod0 is destroyed and recreated, it will have same DNS name (even though ip might change)
- everypod in statefulsets get their own PVs.
- scaledown (starts removing pods from highest number pod)
- 

```
$ kubectl create ns test
$ kubectl get storageclass


$ kubectl get statefulset -n vault
$ kubectl describe statefulset -n vault


$ kubectl get service -n vault
```
