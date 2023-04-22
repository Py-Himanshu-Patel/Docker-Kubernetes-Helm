# Namespaces

Namespaces are a way to organize clusters into virtual sub-clusters — they can be helpful when different teams or projects share a Kubernetes cluster. Any number of namespaces are supported within a cluster, each logically separated from others but with the ability to communicate with each other.

In Kubernetes, namespaces provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).

Default namespaces which are present right from the beginning
```bash
$ kubectl get namespaces
NAME                   STATUS   AGE
default                Active   6d12h
kube-node-lease        Active   6d12h
kube-public            Active   6d12h
kube-system            Active   6d12h
kubernetes-dashboard   Active   6d1h
```

All the resources created initially are allocated in default namespace if we don't create a new namespace.

Create a new namespace
```bash
$ kubectl create namespace my-namespace
```

Benefits
- Resources grouped in namespaces - like databases, logging, monitoring, application, elastic stack.
- Conflict - Many teams creating pod/resources with same name create conflicts, latest override existing.
- Resource Sharing - Staging and Dev can be two different namespaces. Both may use one database residing in db namespace.
- Access and Resource Sharing - Give team access to specific namespace so they access and CRUD resource of their own mamespace.

To get a list of resource which are on cluster level and can't be assigned to any one namespace.
```bash
$ k api-resources --namespaced=false
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
componentstatuses                 cs           v1                                     false        ComponentStatus
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumes                 pv           v1                                     false        PersistentVolume
....
```
Resources which reside in a namespace
```bash
$ k api-resources --namespaced=true
NAME                        SHORTNAMES   APIVERSION                     NAMESPACED   KIND
bindings                                 v1                             true         Binding
configmaps                  cm           v1                             true         ConfigMap
endpoints                   ep           v1                             true         Endpoints
events                      ev           v1                             true         Event
limitranges                 limits       v1                             true         LimitRange
persistentvolumeclaims      pvc          v1                             true         PersistentVolumeClaim
pods                        po           v1                             true         Pod
.....
```

We can update the configmap files to have a namespace as below
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
  namespace: my-namespace
data:
  # the URL is nothing else but the name of mongodb service
  # note, mongodb-service not mongodb-deployment
  database_url: mongodb-service
```

To get resources of any namespace use the `-n` flag in the end of command
```bash
$ k get pod -n default
NAME                                  READY   STATUS    RESTARTS   AGE
mongo-express-5bcd46fcff-jttnz        1/1     Running   0          10h
mongodb-deployment-5d966bd9d6-mrr89   1/1     Running   0          10h
lenovo@workstation:~/dev/Docker-Kubernetes-Helm/kubernetes_mongo_project$ k get pod -n my-namespace
No resources found in my-namespace namespace.
```

No putting these `-n` after each command are annoying thus.

## Kubectx and Kubens
Install kubectx to manage context of kubernetes in minikube.

```bash
sudo apt-get update
sudo git clone https://github.com/ahmetb/kubectx /usr/local/kubectx
sudo ln -s /usr/local/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /usr/local/kubectx/kubens /usr/local/bin/kubens
# Enter kubens in your console and you should be able to see your namespaces
```

Get the current namespace
```bash
$ kubens
default # highlighted
kube-node-lease
kube-public
kube-system
kubernetes-dashboard
my-namespace
```

Change the namespace to desired namespace
```bash
$ kubens my-namespace
Context "minikube" modified.
Active namespace is "my-namespace".

$ kubens
default
kube-node-lease
kube-public
kube-system
kubernetes-dashboard
my-namespace  # highlighted
```
