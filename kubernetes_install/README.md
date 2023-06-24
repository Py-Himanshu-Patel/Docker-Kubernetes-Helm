## Setting up a Kubernetes cluster
It provides mechanisms for application deployment, scheduling, updating, maintenance, and scaling. Kubernetes' auto-placement, auto-restart, and auto-replication features make sure that the desired state of the application is maintained, which is defined by the user. Users define applications through YAML or JSON files.

- `Pods`: A pod, which consists of one or more containers, is the deployment unit of Kubernetes. Each container in a pod shares different namespaces with other containers in the same pod. For example, each container in a pod shares the same network namespace, which means they all can communicate through the localhost.
- `Node/minion`: A node, which was earlier called a minion, is a worker node in the Kubernetes cluster and is managed through the master. Pods are deployed on a node, which includes the following necessary services to run them:
	- `docker`, to run containers
	- `kubelet`, to interact with the master
	- `proxy` (kube-proxy), which connects the service to the corresponding pod.

- `Master`: Masters host cluster-level control services such as the following:
	- `API server`: This has RESTful APIs to interact with the master and nodes. This is the only component that talks to the etcd instance.
	- `Scheduler`: This schedules jobs in clusters, such as creating pods on nodes.
	- `ReplicaSet`: This ensures that the user-specified number of pod replicas is running at any given time. To manage replicas with ReplicaSet, we have to define a configuration file with a replica count for a pod.

Master also communicates with `etcd`, which is a distributed key-value pair. `etcd` is used to store configuration information, which is used by both the master and nodes. The watch unctionality of `etcd` is used to notify the changes in the cluster. `etcd` can be hosted on the master or on a different set of systems.

- `Services`: In Kubernetes, each pod gets its own IP address, and pods are created and destroyed every now and then based on the replication controller configuration. So, we cannot rely on the pod's IP address to cater an app. To overcome this problem, Kubernetes defines an abstraction, which defines logical set of pods and policies to access them. This abstraction is called a **service**. Labels are used to define the logical set, which our service manages.

- `Labels`: Labels are key-value pairs that can be attached to objects. Using these, we can select a subset of objects. For example, a service can select all pods with the label mysql.

- `Volumes`: A volume is a directory that is accessible to the containers in a pod. It is similar to Docker volumes, but not the same. Different types of volumes are supported in Kubernetes, some of which are `emptyDir` (ephemeral), `hostPath`, `gcePersistentDisk`, `awsElasticBlockStore`, and `NFS`. Active development is taking place to support more types of volumes. More details can be found at https:/​/kubernetes.​io/​docs/​concepts/​storage/​.

Kubernetes can be installed to VMs, physical machines, and the cloud. In this recipe, we'll see another way to install Kubernetes on your local host using **MiniKube** with **VirtualBox**.

Refer this video: https://www.youtube.com/watch?v=qG7FGVWijeM

1. Don't install VirtualBox rather go with `Docker` as container/virtual machine manager. [Refer Documentation](https://docs.docker.com/engine/install/ubuntu/).
2. VT-x or AMD-v virtualization must be enabled in your computer’s BIOS.
3. Install Kubectl on host system (Refer for more: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
)

	```bash
	curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
	```  

4. Install MiniKube on host system
	```bash
	curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
	sudo dpkg -i minikube_latest_amd64.deb
	```
	MiniKube will download the `MiniKube` ISO, create a new VM in VirtualBox, and then configure Kubernetes inside. Finally, it will set the cluster as the default for `kubectl`. If you need to run commands against the cluster, you'll d have to use the `kubectl` command.  

5. To create a new deployment, you would use the `kubctl run` command. Enter the following command to start an echo web service and refer the below screenshot:
	```bash
	$ minikube start --driver=docker

	$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
	
	$ kubectl expose pod hello-minikube --type NodePort --port=8080
	service/hello-minikube exposed
	
	$ kubectl get pod
	NAME             READY   STATUS    RESTARTS   AGE
	hello-minikube   1/1     Running   0          2m5s
	```
6. Connect to the new service using `curl` to make sure it is working:
	```
	$ curl $(minikube service hello-minikube --url)
	CLIENT VALUES:
	client_address=172.17.0.1
	command=GET
	real path=/
	query=nil
	request_version=1.1
	request_uri=http://192.168.59.100:8080/

	SERVER VALUES:
	server_version=nginx: 1.10.0 - lua: 10001

	HEADERS RECEIVED:
	accept=*/*
	host=192.168.59.100:31282
	user-agent=curl/7.74.0
	BODY:
	-no body in request- $
	```

You can delete the service and the deployment by using the following code:
```
$ kubectl delete service hello-minikube
$ kubectl delete deployment hello-minikube
```
You can get a list of nodes with the following code:
```
$ kubectl get nodes
```
You can get a list of pods with the following code:
```
$ kubectl get pods
```
You can get a list of services with the following code:
```
$ kubectl get services
```
You can get a list of ReplicaSets with the following code:
```
$ kubectl get rs
```
You can stop a MiniKube cluster with the following code:
```
$ minikube stop
```
You can delete a MiniKube cluster with the following code:
```
$ minikube delete
```
You will see some pods, services, and replication controllers listed, as Kubernetes creates them for internal use. If you didn't see any, you can use the `-all-namespaces` command-line flag.

## Using secrets with Kubernetes
1. Add your secret to a file on your local machine:
```
$ echo -n "MyS3cRet123" > ./secret.txt
```
2. Add your secret to Kubernetes:
```
$ kubectl create secret generic my-secret --from-file=./secret.txt 
secret/my-secret created
```
3. View the secret to make sure it was added correctly:
```
 $ kubectl describe secrets/my-secret
Name:         my-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
secret.txt:  11 bytes
```
4. Use your secret in a pod using a volume. Create a file called `secret_pod.yml` and put the following inside it. We will use the file to create a pod that has a volume where we will mount our secret:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
containers:
- name: shell
  image: alpine
  command:
    - "bin/ash"
    - "-c"
    - "sleep 10000"
  volumeMounts:
    - name: secretvol
      mountPath: "/tmp/my-secret"
      readOnly: true
volumes:
- name: secretvol
  secret:
    secretName: my-secret
```
5. Create a pod using `secret_pod.yml`:
```
$ kubectl create -f ./secret_pod.yml
```
6. View the secret in the pod:
```
kubectl create -f ./secret_pod.yml
```
When you create a secret, Kubernetes will `base64` encode it, and store that secret in the data store that is backing the REST API, for example, `etcd`. If you create a pod that references a secret, when the pod is created, it will get permission to use that secret. When it is deployed, it will create and mount a secret volume, and the secret values are `base64` decoded and stored in this volume as files. Your application will need to reference these files in order to access the secrets.

## Scaling up and down in Kubernetes cluster
In the previous section, we mentioned that a ReplicaSet ensures that the user-specified number of pod replicas is running at any given time. To manage replicas with the ReplicaSet, we have to define a configuration file with a replica count for a pod. This configuration can be changed at runtime.

1. Start the nginx container with a replica count of 3:
```
$ kubectl run my-nginx --image=nginx --replicas=3 --port=80
```
2. This will start three replicas of the nginx container. List the pods to get the status as shown in the following screenshot:
```
$ kubectl get pods
```
As you can see, we have a `my-nginx` controller, which has a replica count of `3`.  
3. Scale down to a replica of 1 and update the ReplicaSet:
```
$ kubectl scale --replicas=1 deployment/my-nginx
$ kubectl get rs
```
4. Get the list of pods to verify that this has worked; you should see only one pod for nginx:
```
$ kubectl get pods
```
Use the following code to get the services:
```
$ kubectl get services
```
As you can see, we don't have any service defined for our nginx containers that we started earlier. This means that, although we have a container running, we cannot access from outside because the corresponding service is not defined.
