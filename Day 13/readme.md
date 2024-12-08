# Kubernetes Manual Scheduling, Node Selectors, and Labels

## Task 1: Manually Scheduling a Pod Without the Scheduler

### Step 1: Disable the Kubernetes Scheduler

1. Access the control plane node

Use the following command to list all nodes in the cluster:

`kubectl get nodes`

Identify the control plane node. For a kind cluster, the nodes run as Docker containers.

2. Enter the control plane container

If using a kind cluster, execute into the control plane node with:

`docker exec -it cka-cluster-control-plane  sh`

3. Navigate to the Kubernetes manifest directory

Change to the `/etc/kubernetes/manifests` directory:

`cd /etc/kubernetes/manifests`

4. Move the scheduler manifest to a temporary location

Temporarily disable the Kubernetes scheduler by moving the `kube-scheduler.yaml` file:

`mv ./kube-scheduler.yaml /tmp`

5. Exit the container:

`exit`

6. Verify the scheduler is not running

Run the following command to confirm that the Kubernetes scheduler is no longer running:

`kubectl get pod -n kube-system`

### Step 2: Create and Apply a Pod Manually

1. Create a Pod manifest `pod.yml`

Create a YAML file `pod.yml` with the following content to manually schedule a pod:


```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  nodeName: cka-cluster-worker2
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  

```

Note: The key part of this configuration is the `nodeName: cka-cluster-worker2` field, which ensures that the pod is scheduled to a specific node manually.

2. Apply the Pod manifest

Create the pod by applying the `pod.yml` file:

`kubectl apply -f pod.yml`

3. Verify the Pod is scheduled on the desired node

Check that the pod is running on `cka-cluster-worker2` by running:

`kubectl get pod -o wide`

## Task 2: Restarting the Kubernetes Scheduler

### Step 1: Restore the Scheduler Manifest

1. Re-enter the control plane node

Access the control plane container again:

`docker exec -it cka-cluster-control-plane  sh` 

2. Move the scheduler manifest back to its original location

Restore the `kube-scheduler.yaml` file back into the `/etc/kubernetes/manifests` directory:

`mv /tmp/kube-scheduler.yaml /etc/kubernetes/manifests`

3. Exit the container:

`exit`

4. Verify the scheduler is running

Confirm that the scheduler pod is running again:

`kubectl get pod -n kube-system`

## Task 3: Creating pods with different labels

### Step 1: Create Pods with Specific Labels

1 .Create a pod with a `test` environment label

Create `pod1.yml` file using following content:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test
  labels:
    env: test
spec:
  nodeName: cka-cluster-worker2
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  
```
2. Create a pod with a dev environment label

Create `pod2.yml` file using following content:


```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-dev
  labels:
    env: dev
spec:
  nodeName: cka-cluster-worker2
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  
```

3. Create a pod with a prod environment label

Create `pod3.yml` file using following content:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-prod
  labels:
    env: prod
spec:
  nodeName: cka-cluster-worker2
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  
```

### Step 2: Apply the Pod Configurations

Create the three pods by running the following commands:
 `kubectl apply -f .\pod1.yml`
 `kubectl apply -f .\pod2.yml`
 `kubectl apply -f .\pod3.yml`

 ## Task 4: Filtering the pods

 ### Step 1: Use Label Selector to Filter Pods

To filter the pods based on specific labels `dev` and `prod`, use the following command:

 `kubectl get pods -l 'env in (dev, prod)'` 