#### K8s Network Arch
- each pod has unique IP address even if these of on different nodes.


#### CNI plugins
- are type of k8s networking plugins.
- set as per k8s networking model
- e.g. calico
- k8s nodes will be unavailable until CNI plugin is installed.

#### k8s DNS
- runs as pod in kube-system
- kubeadm uses Core-Dns
- `pod-ip-address.namespace-name.pod.cluster.local`
- `192.168.1.20.default.pod.cluster.local`

#### Demo

```
$ cat 01_dnstest.yml
apiVersion: v1
kind: Pod
metadata:
  name: dnstest
spec:
  containers:
  - name: busyboxpod
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 3600; done']
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-dnstest
spec:
  containers:
  - name: nginx
    image: nginx:1.19.2
    ports:
    - containerPort: 80
```


#### commands

```
kubectl create -f 01_dnstest.yml
kubectl get pod -o wide
```

- check coredns pods

```
 kubectl get pod -n kube-system
 kubectl get service -n kube-system


 kubectl exec dnstest -- nslookup 192.168.158.25
 kubectl exec dnstest -- nslookup 192-168-158-25.default.pod.cluster.local
 kubectl exec dnstest -- curl http://192-168-158-25.default.pod.cluster.local
```





#### Network Policies
- what are Network policies
  - is an object that allows to control the flow of neetwork communication to and from Pods
  - allows to keep isolating pods as per requirements.

- pod selector
  - podSelector - determines which pods in NS the networkpolicy applies.
  - can select Pods using Pod levels.
  - By default Pods are considered non-isolated and are open to all communications.
  - if a NetworkPolicy selects a Pod, it is considered isolated and allows traffic open by NetworkPolicies only.

- ingress and egress
  - A NP can applied to Ingress, Egrees or both.
  - Ingress - network traffic coming into the pod
  - Egress - traffic leaving the pod

- from and to Selectors:
  - from selector - selects ingress traffic that is allowed.
  - to selector - select egress traffic that is allowed.
  - can use multiple selector types:
    - podSelector
    - namespaceSelector
    - ipBlock

- Ports:  
  - specified one or more ports that will be allosed traffic.

- traffic is only allowed if it matches both `allowed port` and `one of the from/to rules`.


#### Demo:

```
kubectl get namespace
kubectl create namespace np-test
kubectl label namespace np-test team=dev
kubectl create -f 02_np-nginx.yml
kubectl create -f 03_np-busybox.yml
kubectl get pods -n np-test -o wide
NGINX_IP=192.168.158.19 # ip from above command
kubectl exec dnstest -- curl http://$NGINX_IP
```

- nothing is blocking any communication coz there is no NP

- create a NP (no egress/ingress)

```
kubectl create -f 04_np-noallow.yml
kubectl exec -n np-test np-busybox -- curl http://$NGINX_IP

# -- timesout
```

```
$ kubectl apply -f 05-np-NPallow.yml
kubectl exec -n np-test np-busybox -- curl http://$NGINX_IP

```





#### K8s Services:
- What is a Service:
  - provides a way to expose an application running on set of Pods.
  - provide an abstract way for clients to access applications without needing to be aware of applications Pods.
  - routes traffic to underliying Pods in LB fashion.
  
- Endpoints
  - Endpoints are the backend entities to which Services route traffic.
  - with multiple pods, each pod will have an endpoint associated with the Service.
  - looks at service Endpoints to determine which pods it is routing traffic.

**Using K8s Services**

- Service Types:
  - deterimines how and where Serice will expose application.
  - ClusterIP
    - exposes applications inside the cluster nw. used when clients are inside the cluster itself.

  - NodePort
    - expose applications outside the cluster.

  - LoadBalancer
    - expose applications outside the cluster, but use external cloud loadbalancer.
    - is only works with cloud platforms.

  - ExternalName (outside of K8s) - used when vault is outside of K8s.


#### Demo ( iptables -L -t nat | grep svc)

- create deployment

```
kubectl create -f 06-depl-svc-ex.yml
kubectl get pods
```

- create service (clusterIP is default service type)

```
kubectl create -f 07_cluserip_svc.yml
kubectl get endpoints clusterip-svc
kubectl describe service clusterip-svc
```


- test service

```
kubectl create -f 08_svc-test.yml

kubectl exec  svc-test-busybox -- curl clusterip-svc.default
kubectl exec  svc-test-busybox -- curl clusterip-svc.default:80

```

- nodeport service 

  - (node port range 30000 - 32767)
  - TCP is default port

```
kubectl create -f 09_nodeport_svc.yml
kubectl describe service np-svc

# check on k8s node
curl localhost:30080
curl 10.0.0.10:30080
```

**using dns with services**


```
service-name.namespace-name.svc.cluster-domain-name (cluster.local)
```


**kubernetes Ingress** - L7 lb.
- What is ingress
  - is a k8s object that manages external access to services in the cluster
  - capable of providing more funtionality than NodePort - SSL termination, advanced load balaning or name based virtual hosting.

- ingress controller (no default ingress controller)
  - must install one/more ingress controllers.
  - check docs.


- Routing to a service
  - Defines a set of routing rules. A routing rule's properties determine to whech requests it applied.
  - Each rule has a set of paths, each with a backend. requests matching a path will be routed to its associated backend.
  - In this example, a request to http://<>/somepath would be routed to port 80 on the `my-service` service.


- Routing to a Service with a Name Port
  - If a service uses a named port, an ingress can also use the port's name to choose to which port it will route.
  - 

#### demo

```
kubectl create -f 10_ingress.yml
kubectl describe ingress my-ingr

kubectl apply -f 11_clusterip_svc.yml

```
