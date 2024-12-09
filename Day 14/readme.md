# Taints and Tolerations in Kubernetes

This guide demonstrates how to manage pod scheduling on tainted nodes using Taints and Tolerations in Kubernetes.

## Task 1: Running a Pod on a Tainted Node

1. First, list all the nodes in the Kubernetes cluster using the following command:
`kubectl get nodes`

2. Next, taint all the nodes (assuming you have 3 nodes in your cluster) to prevent pods from being scheduled unless they tolerate the taint. Run the following commands:

```sh
kubectl taint nodes cka-cluster-worker  gpu=true:NoSchedule  
kubectl taint nodes cka-cluster-worker2  gpu=false:NoSchedule 
kubectl taint nodes cka-cluster-worker3  gpu=false:NoSchedule 
```

3. Create a `pod.yml` file with the following YAML content for the pod you want to schedule:

```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```
Apply the pod definition:

`kubectl apply pod.yml`

4. At this point, the pod will be in a `Pending` state since it does not tolerate the taints on the nodes. To check the status of the pod, run:

`kubectl get pod -o wide`

5. Update the `pod.yml` file to add the necessary toleration that matches the taint on the node where you want to run the pod. Add the following `tolerations` section under `spec`:

```yml
tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

The updated `pod.yml` should now look like this:


```yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

Apply the updated `pod.yml` file to Kubernetes:

 `kubectl appy -f pod.yml`

6. After applying the updated definition, the pod should now be scheduled and running on the node `cka-cluster-worker` that tolerates the taint. Verify by running:

`kubectl get pod -o wide`

## Task 2: Schedule a Pod on the Control Plane Node

1. To check the taint on the control plane node, run the following command:

`kubectl describe  node cka-cluster-control-plane`

2. To allow pods to be scheduled on the control plane node, you need to remove the taint using the following command:

`kubectl taint  node cka-cluster-control-plane node-role.kubernetes.io/control-plane:NoSchedule-`

3. Create a `redis-pod.yml` file to define the Redis pod. This pod will be scheduled specifically on the control plane node by specifying `nodeName`.

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: redis
  name: redis
spec:
  nodeName: "cka-cluster-control-plane"
  containers: 
  - image: redis
    name: redis

```

4. Apply the `redis-pod.yml` file to create the Redis pod:

`kubectl apply -f .\redis-pod.yml`

5. To confirm the pod is running on the control plane node, execute the following command:

`kubectl get pod -o wide`

6. Once you've verified the pod is running on the control plane, you can reapply the taint to prevent further scheduling of non-control-plane pods on the node:

`kubectl taint  node cka-cluster-control-plane node-role.kubernetes.io/control-plane:NoSchedule`

7. Remove taints from worker nodes:

```sh
kubectl taint nodes cka-cluster-worker  gpu=true:NoSchedule-
kubectl taint nodes cka-cluster-worker2  gpu=false:NoSchedule-
kubectl taint nodes cka-cluster-worker3  gpu=false:NoSchedule-
```