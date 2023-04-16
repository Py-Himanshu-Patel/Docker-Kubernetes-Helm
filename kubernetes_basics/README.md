# Basics of Kubernetes

Deployment -> (manage) -> ReplicaSet -> (manage) -> Pod -> (manage) -> Docker Container

- Create Deployment
```bash
$ kubectl create deployment --image=nginx
$ kubectl get deployment
```

- Edit the config of deployment
```bash
$ kubectl edit deployment <deployment-name>
```

- Describe pod to get info
```bash
$ kubectl describe pod <pod-id>
$ kubectl describe deployment <deploy-name>
$ kubectl describe service <service-name>
```

- Get IP of pods
```bash
$ kubectl get pods -o wide
```

- Move inside a pod
```bash
$ kubectl exec -it <pod-id> -- bin/bash
```

- Delete deployment or pod or service
```bash
$ kubectl delete deployment <deployment-name>
$ kubectl delete service <service-name>
$ kubectl delete pod <pod-id>
```

- Create or delete deployment or services using `.yaml` scripts

Refer Files:
- [kubernetes_basics/nginx-deployment.yaml](kubernetes_basics/nginx-deployment.yaml)
- [kubernetes_basics/nginx-service.yaml](kubernetes_basics/nginx-service.yaml)


```bash
# create
$ kubectl apply -f nginx-deployment.yaml
$ kubectl apply -f nginx-service.yaml

# delete
$ kubectl delete -f nginx-deployment.yaml
$ kubectl delete -f nginx-service.yaml
```

- Get result of deployment in yaml
```bash
$ kubectl describe deployment nginx-deployment -o yaml > nginx-deployment-result.yaml
```

- Get all resources - deploy/service/pods
```bash
$ kubectl get all
```