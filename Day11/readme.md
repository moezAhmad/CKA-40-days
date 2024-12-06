#  Environment Variables and Multi-Container Pods in Kubernetes

This document explains how to work with environment variables and multi-container pods in Kubernetes. Follow the steps below to create and manage your Kubernetes pods and services. 

## Step 1: Create a Multi-Container Pod

1. Create a file named `Pod.yml` and paste the following code into it:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    env: demo
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.default.svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.default.svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```
2. Run the following command to start the pod:
`kubectl apply -f .\pod.yml`

The pod will have two init containers and one main container running inside. The main container will not start until both init containers have finished their tasks.

 ## Step 2: Create Services for the Init Containers

To ensure the init containers can complete their tasks, we need to create services for them. Follow the steps below:

1. To view the logs of the `init-myservice` container, run:
`kubectl logs pod/myapp -c init-myservice`

2. To view the logs of the init-mydb container, run:
`kubectl logs pod/myapp -c init-mydb`

Both containers will be looking for services, which we will create next. First, we will create deployments to expose them with services.

3. To create the `myservice` deployment, run:
`kubectl create deployment myservice --image=nginx --replicas=1 --port=80`

4. To expose the myservice deployment, run:
`kubectl expose deploy myservice --port=80 --target-port=80 --name=myservice`

5. To monitor the pod status run:
`kubectl get pods -w`

It may take some time, but eventually, the `init-myservice` container will complete.

6. Similarly, to create the `mydb` deployment, run:
`kubectl create deployment mydb --image=redis --replicas=1 --port=6379`

7. To expose the mydb deployment, run:
`kubectl expose deploy mydb --port=6379 --target-port=6379 --name=mydb`

8. Run the following command again to check the status of the pods:
`kubectl get pods -w`

After some time, the init-mydb pod will complete, and the main container will change to the "Running" state.
