# RBAC and creating role and role binding objects.


Assuming a `learner` role has been created with the following configuration:

```yml
kind: Role
metadata:
  creationTimestamp: "2024-12-12T14:00:08Z"
  name: learner
  namespace: default
  resourceVersion: "339359"
  uid: 6a4008e2-e4a9-4d3b-aa55-1cc106d51468
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - create
  - get
  - list
  - update
  - delete
```

Run the following command to switch the context to the `learner` user:

`kubectl config use-context learner`

Create a pod using the `nginx` image:

`kubectl run nginx --image=nginx`

`learner` has permission to create pod so pod will be created


To return to the original context, use:

`kubectl config use-context kind-cka-cluster`

To view the existing roles, use the following command:

`kubectl get role`

To view the existing role bindings, use the following command:

`kubectl get rolebinding`

To delete the `learner` role, run the following command:

`kubectl delete role learner`

To delete the role binding associated with the `learner` role, execute:

`kubectl delete rolebinding learner-binding-learner`


Define a new role `pod-reader` with the ability to `get`, `watch`, and `list` pods:

`kubectl create role pod-reader --verb=get --verb=watch --verb=list --resource=pods`

Bind the `learner` user to the pod-`reader` role by creating a `RoleBinding`:

`kubectl create rolebinding read-pods --role=pod-reader --user=learner  `

To confirm the objects were created successfully, run:

```sh
kubectl get role

kubectl get rolebinding
```

Set the context back to `learner`:

`kubectl config use-context learner`


Try creating a new `nginx` pod with the following command:

`kubectl run nginx --image=nginx`

The `learner` user will not have permission to create pods, as this action requires higher privileges

List the pods in the `default` namespace:

`kubectl get pods -n default`

Finally, attempt to create a new deployment:

`kubectl create deployment nginx-deploy --image=nginx` 

The `learner` user will not be authorized to create deployments.