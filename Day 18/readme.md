# Health probes in Kubernetes

To generate a basic YAML file for a busybox pod, run the following command:

`kubectl run pod --image=registry.k8s.io/busybox --dry-run=client  -o yaml > '.\pod.yml'`

This command generates a YAML file named `pod.yml` that defines a simple `busybox` pod but does not actually deploy it yet.


Update the `pod.yml` to include a shell command that will create a file, wait, remove it, and then continue sleeping:

Updated `pod.yaml`

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
  name: busybox
spec:
  containers:
  - image: registry.k8s.io/busybox
    name: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
```
This configuration ensures that the `busybox` container will execute a script that simulates an application lifecycle, including creating a file and removing it, with pauses in between.

Next, update the `pod.yml` to include a liveness probe that checks the presence of the `/tmp/healthy` file:

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
  name: busybox
spec:
  containers:
  - image: registry.k8s.io/busybox
    name: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
This `livenessProbe` executes a `cat` command on the `/tmp/healthy` file to check if it exists. If the file is missing (indicating the container is not functioning properly), Kubernetes will restart the container.

To deploy the updated `busybox` pod with the liveness probe, run the following command:

`kubectl apply -f pod.yml`


Next, you can create another pod using the `agnhost` image, which includes health probe functionality. Create a new YAML file, `agnhost.yml`, with the following content:

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
  name: agnhost
spec:
  containers:
  - image: registry.k8s.io/e2e-test-images/agnhost:2.40
    name: agnhost
    args:
    - liveness
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
```

This configuration sets up both a liveness probe and a readiness probe for the `agnhost` container. Both probes check the `/healthz` endpoint on port `8080` to determine the health of the container.

To deploy the `agnhost` pod with the defined health probes, use the following command:

`kubectl apply -f agnhost.yml`