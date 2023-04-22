# Mongo DB Project
Display services, deployments, pods, secrets in action

List
- 2 deployment / pod
- 2 services
- ConfigMap
- Secret

Web Browser  ->  
Mongo Express External Service ->  
Mongo Express Pod ->  
Mongo DB Internal Service ->  
Mongo DB Pod  

**Internal Service**: No external request is allowed only the services from same cluster can talk to it.

`Deployment.yaml` file use `ConfigMap.yaml` and `Secret.yaml` file.

Check if the cluster is empry
```bash
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   25h
```

Make the Deployment file [kubernetes_mongo_project/mongo-deployment.yaml](mongo-deployment.yaml)

## Secret File in MongoDB
Uses base 64 encoded data not plaintest for secrets
```bash
$ echo -n "username" | base64
dXNlcm5hbWU=
$ echo -n "password" | base64
cGFzc3dvcmQ=
```

mongo-secret.yaml
```yaml
apiVersion: v1
kind: Secret	# this tell it's a secret file
metadata:
  name: mongodb-secret	# name of secret
type: Opaque
data:
  mongo-root-username: dXNlcm5hbWU= # username
  mongo-root-password: cGFzc3dvcmQ=	# password
```

Apply the secret file
```bash
$ kubectl apply -f mongo-secret.yaml
secret/mongodb-secret created

$ kubectl get secret
NAME             TYPE     DATA   AGE
mongodb-secret   Opaque   2      5s
```

# Deploy MongoDB - Database
Run deploy script and create deployment
```bash
$  kubectl apply -f mongo-deployment.yaml 
deployment.apps/mongodb-deployment created
```

```bash
$ k get all
NAME                                      READY   STATUS    RESTARTS   AGE
pod/mongodb-deployment-5d966bd9d6-gc5nn   1/1     Running   0          63s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongodb-deployment   1/1     1            1           63s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/mongodb-deployment-5d966bd9d6   1         1         1       63s
```

Reapply the same deploy file after adding config of mongodb service.
```bash
$ kubectl apply -f mongo-deployment.yaml 
deployment.apps/mongodb-deployment unchanged
service/mongodb-service created
```

To see the service created
```bash
$ k get service
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP     28h
mongodb-service   ClusterIP   10.110.140.167   <none>        27017/TCP   107s
```

If we describe the service, then on of the endpoint of this service will point to the pod of mongodb. (IP of mongodb pod and port on which the pod is listening).

# Mongo Express - Service

Make the Deployment and Service file [kubernetes_mongo_project/mongo-express.yaml](mongo-express.yaml)

Also the config map file holds the URL
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  # the URL is nothing else but the name of mongodb service
  # note, mongodb-service not mongodb-deployment
  database_url: mongodb-service
```

After applying both the files - Configmap and then Mongo Express service we can see the IP's assigned to services
```bash
$ kubectl get services
NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes              ClusterIP      10.96.0.1       <none>        443/TCP          6d3h
mongo-express-service   LoadBalancer   10.97.205.192   <pending>     8081:30001/TCP   16m
mongodb-service         ClusterIP      10.96.187.178   <none>        27017/TCP        16m
```
Since we are in minikube, it's a little bit different than regular kubernetes. Thus we don't see a external IP assigned to Loadbalancer External service. To assign a public IP to the service use the below command in minikube.
```bash
$ minikube service mongo-express-service

|-----------|-----------------------|-------------|---------------------------|
| NAMESPACE |         NAME          | TARGET PORT |            URL            |
|-----------|-----------------------|-------------|---------------------------|
| default   | mongo-express-service |        8081 | http://192.168.49.2:30001 |
|-----------|-----------------------|-------------|---------------------------|
🎉  Opening service default/mongo-express-service in default browser...
```

Now the mongo-express-service is publicly accessable at http://192.168.49.2:30001