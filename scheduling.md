### scheduler
- scheduler-controller looks for pods without a nodeName field set and attempts to assign them to a node
- once a canidate node is found for the pod, a Binding object is created and the nodeName field is assigned on the pod
- if you want to schedule the pod yourself, assign a nodeName at create time of the pod
- after creation you can specify which node to assign the pod to by creating a new Binding object and making a POST request to the pods binding-api

### labels and selectors
- applied at the metadata level and used to group objects together
- to get a pod by label use
```bash
k get pods -n kube-system --selector k8s-app=kube-dns
```
- get all objects (in default namespace) using a label selector
```bash
k get all --selector env=prod
```
- get object matching on multiple labels
```bash
k get pods --selector env=prod,bu=finance,tier=frontend
```

### taints and tolerations
- taints are set on node and tolerations on pods
	- if a taint is set on a node the only way to schedule a pod there is with a cooresponding toleration on that pod
```bash
k taint nodes node-name app=blue:NoSchedule
```

pod toleration 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

resource limits
- kubernetes gives a standard amount of CPU, RAM, and Storage space to all pods
- you can specify more or less then the standard given in the pods spec.template
- the CPU limit for a standard pod is 1 vCPU and 512 Mi of RAM. You can increase these limits as well
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```
- pods that try to exceed their CPU usage become throttled
- pods that continously use more memory then allocated and given the memory but are evenutally terminated
- resource limits can not be changed while pod is running

### Node Selectors
- `nodeSelector` can be set at the pod definition level and match a node by label
  ```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
  nodeSelector:
	size: Large # label on a node in the cluster 
```
- if you want to add more criteria to where a pod will schedule then `nodeAffinity` will be better suited since the `nodeSelector` is confined to just using labels

### Node Affinity
- `nodeAffinity` allows for OR / AND clauses to be added to pod scheduling
- `requiredDuringSchedulingIgnoredDuringExecution`: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
- `preferredDuringSchedulingIgnoredDuringExecution`: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.
- the `IgnoreDuringExecution` portion deals with potential label changes to the node the pod is scheduled on changing which would no longer match the nodeAffinity the pod started with. In that case, that pod would still stay running on this node because we are ignoring this requirement during execution
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0
```