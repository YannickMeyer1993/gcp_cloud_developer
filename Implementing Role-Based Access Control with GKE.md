A manifest file called my-namespace.yaml has been created for you that creates a new namespace called production.

```yaml
apiVersion: v1
kind: Namespace
metadata:
name: production
```


List the current namespaces in the cluster using the following command:

`kubectl get namespaces`

In the Cloud Shell, execute the following command to create the new namespace:

`kubectl create -f ./my-namespace.yaml`

You can view details of an existing namespaces by executing:

`kubectl describe namespaces production`

Create a Resource in a Namespace

If you do not specify the namespace of a Pod it will use the namespace ‘default'. In this task you specify the location of our newly created namespace when creating a new Pod. A simple manifest file called my-pod.yaml that creates a Pod that contains an nginx container has been created for you.

```yaml
apiVersion: v1
kind: Pod
metadata:
name: nginx
labels:
name: nginx
spec:
containers:
- name: nginx
  image: nginx
  ports:
    - containerPort: 80
```

In the Cloud Shell, execute the following command to create the resource in the namespace called production:

Alternatively, you could have specified the namespace in the yaml file. This requires the namespace: production field in the metadata: section.

```yaml
apiVersion: v1
kind: Pod
metadata:
name: nginx
labels:
name: nginx
namespace: production
spec:
containers:
- name: nginx
  image: nginx
  ports:
    - containerPort: 80
```

Run the command again, but this time specify the new namespace:

`kubectl get pods --namespace=production`

n this task you will create a sample custom role, and then create a RoleBinding that grants Username 2 the editor role in the production namespace.

The role is defined in the pod-reader-role.yaml file that is provided for you. This manifest defines a role called pod-reader that provides create, get, list & watch permission for Pod objects in the production namespace. Note that this role cannot delete Pods.

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
namespace: production
name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "get", "list", "watch"]
```

Before you can create a Role, your account must have the permissions granted in the role being assigned. For cluster administrators this can be easily accomplished by creating the following RoleBinding to grant your own user account the cluster-admin role.

To grant the Username 1 account cluster-admin privileges, run the following command, replacing [USERNAME_1_EMAIL] with the email address of the Username 1 account:v

`kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user [USERNAME_1_EMAIL]
`
In the Cloud Shell, execute the following command to create the role:

`kubectl apply -f pod-reader-role.yaml
`
To list the roles to verify it was created, execute the following command:

`kubectl get roles --namespace production
`
Create a RoleBinding
The role is used to assign privileges, but by itself it does nothing. The role must be bound to a user and an object, which is done in the RoleBinding.

The username2-editor-binding.yaml manifest file creates a RoleBinding called username2-editor for the second lab user to the pod-reader role you created earlier. That role can create and view Pods but cannot delete them.

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
name: username2-editor
namespace: production
subjects:
- kind: User
  name: [USERNAME_2_EMAIL]
  apiGroup: rbac.authorization.k8s.io
  roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

In the Cloud Shell create an environment variable that contains the full email address of Username 2:

`export USER2=[USERNAME_2_EMAIL]`

In the Cloud Shell use sed to replace the placeholder in the file with the value of the environment variable.

`sed -i "s/\[USERNAME_2_EMAIL\]/${USER2}/" username2-editor-binding.yaml`

In the Cloud Shell for Username 1, execute the following command to create the RoleBinding that grants Username 2 the pod-reader role that includes the permission to create Pods in the production namespace:

`kubectl apply -f username2-editor-binding.yaml`

In the Cloud Shell for Username 1, execute the following command with the production namespace specified:

`kubectl get rolebinding --namespace production`

User2 kann nicht löschen:

`kubectl delete pod production-pod --namespace production`


