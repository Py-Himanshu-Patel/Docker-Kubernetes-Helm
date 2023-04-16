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

```mermaid
graph TD;
Web Browser  -->  Mongo Express External Service; 
Mongo Express External Service -->  Mongo Express Pod;
Mongo Express Pod -->  Mongo DB Internal Service;
Mongo DB Internal Service -->  Mongo DB Pod; 
```

**Internal Service**: No external request is allowed only the services from same cluster can talk to it.

`Deployment.yaml` file use `ConfigMap.yaml` and `Secret.yaml` file.

Check if the cluster is empry
```bash
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   25h
```

Make the Deployment file [kubernetes_mongo_project/mongo-deployment.yaml](mongo-deployment.yaml)
