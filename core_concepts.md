## cluster_arch

### nodes
Master Node (coordinates scheduling and administration of worker nodes)
- master node controllers
	- node-controller: onboards new nodes to the cluster or removes
	- controller-manager:
	- replication-controller: desired number of containers are running on nodes at all times
- *ectd Node*: key value store of what events have occuring during the administration of the worker nodes from the communication with the master node
- the kube-scheduler is responsible getting pods spun up on worker nodes
- what coordinates communication between master and worker nodes? the kube-apiserver


Worker Node (where pods are deployed. managed by Master Node)
- every node needs a conatiner runtime in order to spin up pods. containerd / docker / etc need to be installed to make this happen (this applies for all node types)
- kubelet lives on each worker node and listens for instructions from the kube-apiserver in order to bring containers up/down 
- kube-proxy lives on each node so that pods on different nodes can communicate

### ectd server
- data store of information about pods, nodes, configs, secrets, accounts, roles, etc
- when you make a `kubectl get` command you are pulling from ectd server
- you can deploy the ectd server yourself to the master node or use kubeadm to set this up for you

### kube-apiserver
- coordinates requests between master and work nodes
- an incoming request will have this flow
	- authenticate
	- validate
	- retrieve data (etcd)
	- update etcd
	- scheduler
	- kubelet (work node)
- the kube-apiserver runs on the master node
```bash
❯ k get pods -n kube-system
NAME                                         READY   STATUS    RESTARTS      AGE
coredns-565d847f94-5szbd                     1/1     Running   0             16h
coredns-565d847f94-9djrn                     1/1     Running   0             16h
etcd-kind-control-plane                      1/1     Running   0             16h
kindnet-7cfxx                                1/1     Running   0             16h
kube-apiserver-kind-control-plane            1/1     Running   0             16h
kube-controller-manager-kind-control-plane   1/1     Running   3 (54m ago)   16h
kube-proxy-7mwtr                             1/1     Running   0             16h
kube-scheduler-kind-control-plane            1/1     Running   7 (93m ago)   16h
```

### controller-manager
controller - watches and manages the state of kubernetes obects
- the k8's controller-manager is where all std controllers live
- you'll see this service as a pod in the `kube-system` namespace

### kube-scheduler
- kube-scheduler determines where to assign a pod to a node in the cluster
- kublet is responsible for starting that pod on the node
- performs filtering based on resources, labels, taints, etc...

### kubelet (lives on worker nodes)
- registers worker node with master node
- starts up pods and monitors 

### replication-controller
- monitors pods on nodes
- scale up replicas
```bash
k scale --replicas=6 replicaset myapp-replicaset
```

### deployments
- create replicasets
- defines pod 
- can be used to scale pods up and down

### services
- use to communicate between pods or externally
- types:
	- NodePort: service connecting port on the node to service running in a pod
	- ClusterIP: virtual IP inside the cluster that can communicate to different servers
	- LoadBalancer: used to balanace traffic to our service


### declarative approach
```bash
kubectl create deployment --image=nginx nginx
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
kubectl scale deployment nginx --replicas=4
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --replicas=4
```