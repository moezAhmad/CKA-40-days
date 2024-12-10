# Node Affinity in Kubernetes

## Task 1: Schedule nginx Pod Using Node Affinity

1. Create a YAML file named `nginx.yml` with the following content to define the nginx pod with a node affinity rule that requires the node to have a `disktype` label with the value `ssd`.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent

```

2. To create the pod, execute the following command:

`kubectl apply -f .\nginx.yml`

3. The pod will initially be in the Pending state because it hasn't been scheduled to a node yet.

`kubectl get pods`

4. To ensure that the pod can be scheduled, label one of your worker nodes with `disktype=ssd`.

`kubectl label nodes cka-cluster-worker  disktype=ssd`

5. After the label has been applied, the pod will be scheduled to the `cka-cluster-worker` node. Check the pod’s status:

`kubectl get pods -o wide`

6. To confirm the node labels, use the following command:

`kubectl get nodes --show-labels`

## Task 2: Schedule redis Pod Using Node Affinity

1. Create a YAML file named `redis.yml` with the following content to define the `redis` pod with a node affinity rule that requires nodes to have the `disktype` label, regardless of its value.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: Exists
  containers:
  - name: redis
    image: redis
    imagePullPolicy: IfNotPresent
```

2. To create the `redis` pod, execute the following command:

`kubectl apply -f .\redis.yml`

3. The `redis` pod will now run on one of the worker nodes with the `disktype` label. Check its status:

`kubectl get pods -o wide`

4. Now, label a second worker node with `disktype=` to allow redis to be scheduled there.

`kubectl label nodes cka-cluster-worker2  disktype=`

5. Since two nodes now have the `disktype` label, the redis pod can be scheduled on either node. Verify that the redis pod has been scheduled on `cka-cluster-worker`.

To demonstrate how taints and tolerations can influence pod scheduling, let's configure a taint on `cka-cluster-worker` and add a toleration to the `nginx` pod.

6. Edit the `nginx.yml` file to add a toleration, allowing the nginx pod to tolerate the `disktype=true:NoExecute` taint.

Updated `nginx.yml`

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "disktype"
    operator: "Equal"
    value: "true"
    effect: "NoExecute"

```

7. Re-apply the `nginx.yml` configuration to update the pod:

`kubectl apply -f nginx.yml`

8. Apply a taint to the `cka-cluster-worker` node. This will prevent new pods from being scheduled there unless they have a corresponding toleration.

`kubectl taint node cka-cluster-worker disktype=true:NoExecute`

7. At this point, the `nginx` pod will continue to run, as it can tolerate the taint. However, the `redis` pod will be evicted from the `cka-cluster-worker` node due to the taint.

Check the pod status:

`kubectl get pods`

8. Rcreate the redis pod so that it can be scheduled on `cka-cluster-worker2` (which does not have the taint).

`kubectl apply -f redis.yml`

9. Now, the redis pod should be scheduled on `cka-cluster-worker2`


